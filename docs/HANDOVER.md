---
title: Handover to real-data work
status: draft
author: Proposed for Stephen's review
created: 2026-11-26
---

# Handover to real-data work

This repository develops and validates the method on synthetic data.
Applying it to real project data happens later, in a separate repository.
The aim is that the real-data repository only has to add one data adapter
and its configuration, and inherits everything else unchanged.

These patterns are proposals until the spec accepts them. Each is cheap to
set up now and expensive to retrofit after the model code exists.

## Data contract as code

The model reads one canonical shape, versioned, with a data dictionary:

| Table | Grain | Contents |
| --- | --- | --- |
| Periods | project × leaf element × period | `project_id`, `wbs_id`, `period` (1-based), `period_end`, incremental `PV`, `EV`, `AC` |
| Elements | project × element | `BAC`, `wbs_parent` (nullable), optional `wbs_type` |
| Projects | project | `BAC`, `SAC` in periods, `start_date`, `status_date`, `schema_version` |
| Outcomes | project | final cost and completion period, when known |

Validation splits in two:

- **Structural checks** hold for any data: types, unique keys, one row per
  leaf per period, parents never in the period table, BAC totals reconcile.
- **Complete-history invariants** (EV sums to BAC, completion reached,
  non-negative increments after preprocessing) apply only where a history
  is finished or synthetic, because real in-progress data breaks them by
  design.

A failed check raises a named `ContractError(ValueError)`, so callers can
catch it.

The contract module also owns the helpers every adapter needs:
`snapshot(data, as_of)` to truncate a history, `first_crossing(data, pct)`
to find evaluation points, and the earned schedule generalized inverse,
ES = inf{t : PV(t) ≥ EV}, which stays well defined where PV is flat.

## Adapters, not branches

Model code consumes only the contract. Each source is an adapter into it:

- the model's own simulator → contract,
- `synthetic_evm` → contract,
- later, the real source → contract, in the real-data repository.

Input that arrives cumulative-to-date passes through one normalization
step into increments, and records which form each row came from.

## Preprocessing as named, logged rules

Each rule (negative adjustments, flat PV segments, eligibility from the
baseline) is a named function that reports how many records it touched.
Those counts are the disclosure-safe summaries a real-data paper reports.

## Backtest protocol as configuration

The holdout scheme, evaluation points, metrics and normalizations live in
one configuration file, so the real-data repository reruns the identical
protocol. Experiment settings (scenarios, number of projects, seed,
misspecification knobs) are TOML files too.

## Disclosure-safe outputs by default

Every report and figure function emits only dimensionless or normalized
quantities: ratios, percent complete, error / BAC, error / SAC. Absolute
dollars and dates appear only when a caller asks for them explicitly.

## Run records

Every dataset and fit writes a manifest: package versions and git SHAs,
seed, configuration hash, schema version, row counts, and a checksum of
the input data. Figures and tables in the paper cite a manifest, which also
shows a reviewer that a result came from synthetic inputs only.

## Seeding

One `numpy.random.SeedSequence` tree keyed by experiment, then project, then
mechanism. Each mechanism draws from its own child stream, so adding a
mechanism does not reshuffle the others.

## Dependencies that travel

- `synthetic_evm` is a development dependency, pinned to a release tag, not
  a runtime dependency. The real-data repository can drop it or vendor a
  tagged copy.
- Dependency floors are set to what the code needs, and CI tests the lowest
  allowed versions, since an internal package mirror may lag.
- Nothing calibrated from real data (for example, priors or scenario
  parameters tuned to look like real projects) is committed here.

## Masking real data

Real data will need masking before it can be used, and claims about it
will be backtesting results in aggregate only. The preferred masks are
transformations the model is invariant to, so a masked fit gives the same
forecasts, in masked units, as an unmasked one. This is a design
consideration, not a driving factor; the masking practices themselves
still need research.

Candidates to keep possible while designing the model:

- **Rescaling dollars per project.** Divide PV, EV, AC and BAC by a
  project constant (BAC itself, or a secret factor). The Gamma family is
  closed under scaling, and ρ is dimensionless, so this works if the scale
  of μ and h is carried by project-level effects rather than by fixed
  priors in absolute dollars.
- **Shifting and coarsening time.** The model uses period indices, not
  calendar dates, so start dates can be shifted or dropped.
- **Relabeling identifiers.** Project and WBS identifiers become arbitrary
  keys; only the tree structure is kept.

Each choice to check against: priors stated in absolute units, covariates
that reveal identity (size, location, contractor), and whether the tree
shape itself identifies a project.

## No real data in this repository

Real cost and performance data never enter this repository, its history,
its issues or its CI logs. Test fixtures are synthetic.

## Planned package layout

A starting point for the spec, not yet created:

| Module | Responsibility |
| --- | --- |
| `hbm_evm.data` | Contract schemas, validation, `snapshot`, `first_crossing`, earned schedule inverse |
| `hbm_evm.simulate` | Forward simulator of the generative model (data generation and forecasting) |
| `hbm_evm.model` | Likelihood and priors in the probabilistic programming library |
| `hbm_evm.aggregate` | WBS roll-ups and derived κ, ξ |
| `hbm_evm.score` | CRPS, interval score, coverage, benchmark forecasts |
| `hbm_evm.backtest` | Holdout runner driven by the protocol configuration |
| `figures/` | Paper figures, ported in style from the earlier primer |
