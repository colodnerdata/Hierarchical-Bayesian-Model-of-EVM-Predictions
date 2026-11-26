---
title: Problem statement
status: draft
author: Proposed for Stephen's review
created: 2026-11-26
---

# Problem statement

This page states the research problem the repository exists to answer, in
plain terms, before any code. It is the raw material for `intent/intent.md`,
which remains the owner's own statement of the work.

## In one paragraph

Earned value management (EVM) reports how much work a project has earned
(EV) and what it has cost (AC) against a time-phased plan (PV). The standard
forecasts of final cost and duration are deterministic index formulas such
as EAC = BAC / CPI. They give a single number, no honest statement of
uncertainty, and no way to borrow evidence from similar work elsewhere in
the work breakdown structure (WBS). This project builds a hierarchical
Bayesian model of how cost and progress accrue at the lowest reported WBS
level, fits it to the incomplete history of a project that is still under
way, and forecasts final cost and completion date as full predictive
distributions. The method is developed and validated on synthetic data.

## Research question

> Given the periodic EVM history of an unfinished project, does a
> hierarchical stochastic model of leaf-level cost and progress produce
> better-calibrated forecasts of final cost and completion date than
> index-based EVM methods and non-hierarchical alternatives?

Sub-questions the paper answers:

1. **Calibration.** Do the model's 50, 80 and 90 percent predictive
   intervals for final cost and completion date contain the realized
   values at those rates, at 25, 50 and 75 percent complete?
2. **Accuracy.** Is the forecast distribution closer to the outcome than
   the benchmarks, as measured by a proper scoring rule?
3. **Recoverability.** Can the model state the probability that a project
   finishes within budget, P(C_final ≤ BAC), the probability that it
   finishes late, P(T > SAC), and the probability of both?
4. **Pooling.** Does letting WBS elements borrow strength from their parent
   help, and when does it mislead?

## Why the earlier formulation was replaced

The earlier work (the sequential hierarchical Bayesian EVM primer and its
repository) treated two efficiency ratios as the random quantities to
model: periodic cost efficiency κ = ΔAC / ΔEV and schedule efficiency
ξ = Δt / ΔES. Each received a right-skewed likelihood, and each period's
posterior became the next period's prior. A critical review of the paper
draft found five problems with that premise:

1. **The ratios are undefined when they matter most.** A period in which
   money is spent but no value is earned has ΔEV = 0, so κ does not exist.
   Stalls that burn money are exactly the behavior a forecast must capture,
   and a likelihood on κ cannot represent them.
2. **The "probabilistic" recoverability test was deterministic.** The
   proposed probabilities reduced algebraically to facts already observed
   (for example, whether cumulative κ exceeds 1).
3. **The intervals were on the wrong quantity.** A credible interval for a
   mean efficiency is not a prediction interval for final cost. It leaves
   out the noise of all the periods still to come, so it is too narrow.
4. **The ratios are not new.** κ and ξ are the reciprocals of CPI and of
   the earned schedule index SPI(t). They remain useful for presenting
   results, but are not a contribution.
5. **Parents were counted twice.** Fitting parent and child observations
   double-counts the same dollars, and summing independently sampled
   children ignores their covariance.

The research question survived the review; the formulation did not. The
replacement models the increments that the ratios are computed from.

## The replacement model in plain terms

The organizing idea: accounting identities define which histories are
possible, two stochastic processes move each work element through that
space, a hierarchy says how elements resemble each other, and EVM ratios
are measurements of the resulting paths.

At each leaf element j of project i, in each reporting period t:

1. **Progress.** A coin flip decides whether any value is earned (with
   probability π, none is: a stall). If some is, a potential amount Q is
   drawn from a Gamma distribution with mean μ. Earned value for the
   period is the smaller of Q and the work remaining, so EV never exceeds
   BAC and an element can actually finish.
2. **Cost.** Expected spend is h·Δt + ρ·ΔEV: a time-driven part h
   (standing labor, overhead, holding costs, which accrue even in a stall)
   plus a progress-driven part ρ (cost per dollar of earned value, the
   descendant of κ). Actual spend is Gamma around that mean.

The four element parameters (π, μ, h, ρ), on logit or log scales, combine
a population baseline, observed covariates, a persistent element effect
nested in the WBS (each child varies around its parent), and a shared
project state that drifts over time. The drifting state answers the
objection that a fixed efficiency cannot track a project whose performance
changes, and it makes sibling elements move together in a bad month.

Everything else is computed, not modeled: cumulative EV and AC by summing,
parent values by summing children, earned schedule by inverting the
parent's own PV curve at its EV, and κ and ξ from those.

**Forecasting** is posterior predictive simulation. For each posterior
draw, the leaf processes are rolled forward to completion. One set of
simulated paths gives the distribution of final cost and completion date,
the recoverability probabilities, and intervals at any WBS node. Parameter
uncertainty and future-period noise both enter, which fixes problem 3.

Three properties make the model practical to fit:

- **Stalls are observed, not latent.** For an unfinished element, ΔEV = 0
  exactly when the coin flip says stall, so the progress likelihood has a
  closed form and gradient-based samplers (NUTS) work.
- **Stalls identify the time-driven cost.** In a period with no progress,
  expected spend is h·Δt alone. The periods that broke the ratio model are
  the ones that pin down h.
- **Eligibility comes from the baseline.** A zero only counts as a stall
  when the element was planned to be active. Not-yet-started and finished
  periods are left out of the progress likelihood.

Symbols follow the owner's variable glossary for the generative model. The
ones used most often:

| Symbol | Meaning |
| --- | --- |
| i, j, t | Project, leaf WBS element, reporting period |
| ΔEV, ΔAC | Earned value and actual cost accrued in a period |
| R | Remaining budgeted work at the start of a period, BAC − EV |
| Z, π | Progress indicator and probability of a zero-progress period |
| Q, μ, a_q | Potential progress, its mean and Gamma shape |
| h, ρ, a_c | Time-driven cost rate, cost per unit of EV, Gamma shape of cost |
| u_ij, pa(j) | Persistent element effect and the element's WBS parent |
| G_i,t | Shared, time-varying project state (AR(1)) |
| T_i, C_i^final | Realized completion time and final cost |

## Scope of the paper's version

In scope:

- The joint model written as an explicit factorization, with the
  accounting identities as constraints.
- Leaf-level hurdle-Gamma progress, censored at completion; Gamma cost
  with time-driven and progress-driven terms.
- Elements within project (and one WBS parent level if the synthetic data
  carries one), diagonal covariance first; a scalar shared project shock
  before the full shared state.
- A posterior predictive simulator to completion, with P(C_final ≤ BAC),
  P(T > SAC), their joint probability and intervals at each node.
- A synthetic study: parameter recovery and a light simulation-based
  calibration check, then well-specified data plus three or four
  deliberately misspecified scenarios.
- Benchmarks: CPI-based EAC, IEAC(t), complete pooling, no pooling, and the
  same model without the shared state.
- A synthetic backtest that runs exactly the protocol later real-data work
  will use (see [Handover](HANDOVER.md)).

## Non-goals

These are out of scope for this repository's first version and for the
paper:

- **Feeding unfinished projects into parametric estimating relationships.**
  The submitted abstract proposed entering each incomplete project into a
  cost estimating relationship as one row with its posterior mean and
  standard deviation. Done naively, that recycles prior information as if
  it were data. It is left for a future paper; the coherent route is to put
  project-level covariates into the model's root effects.
- **Real project data.** No real EVM data enters this repository. Real-data
  application happens later, in a separate repository, against the same
  data contract and protocol.
- **Cancelled projects.** Their completion cost is a counterfactual that EVM
  data alone does not identify. They would need a competing-event model.
- **Re-baselining.** A rebaseline may change what a parameter means. It is
  a candidate structural break, discussed but not modeled.
- **Schedule network logic.** Earned schedule is not a critical-path model.
  The claim is EVM-based statistical duration forecasting, nothing more.

## Data

This repository uses synthetic data only, from two sources with different
jobs:

- **The model's own simulator** generates data from the hurdle-Gamma model
  with known parameters. It is the ground truth for parameter recovery and
  calibration checks.
- **`synthetic_evm`** (planned dependency, the owner's separate synthetic
  EVM data package) generates histories from a mechanistic world of work
  packages, EV techniques and time warps that is not the model being
  fitted. It supplies the deliberately misspecified data the review asked
  for. It stays separate from the model and exports no model parameters,
  so its data is never generated by the model being tested. What it still
  needs before it can serve as a dependency is reviewed separately.

Both feed the model through one data contract: tables of leaf-period
increments, baselines, the WBS tree and outcomes. That contract is what a
later real-data adapter must produce. See [Handover](HANDOVER.md).

## How success is judged

- Proper scores: CRPS and interval score for final cost and completion.
- Coverage at 50, 80 and 90 percent, clustered by project, with the small
  number of independent projects stated plainly.
- Errors normalized by BAC for cost and by SAC for duration.
- Evaluation points chosen by a first-crossing rule: the first reporting
  period at or past 25, 50 and 75 percent complete.
- Leave-one-project-out fits for every stage, so no forecast sees its own
  outcome.

## Constraints

- **Timeline.** The paper is due at the end of January 2027 after two
  internal reviews, so the method, synthetic study and figures need to be
  complete by mid-January.
- **Main technical risk.** Identifiability with a few dozen projects, in
  particular separating h from ρ when progress is steady. Parameter
  recovery is the early warning; the fallback is to fix shape parameters or
  tighten priors and say so.
- **Conventions.** Python 3.12, uv, Ruff, numpydoc docstrings, EAFP style.

## Open questions

1. Does the paper use one WBS parent level, or elements directly within
   project?
2. What preprocessing rule handles negative adjustments and zero-cost
   periods with positive expected cost (net into the prior period, a point
   mass, or an observation layer)?
3. Which probabilistic programming library and sampler? The plan so far
   assumes PyMC with NUTS.
4. Which license, given that the repository is public?
