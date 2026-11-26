# Decisions

Record the choices future-you or a coding assistant might otherwise repeat.
Add an entry when a decision affects interfaces, architecture, dependencies,
data formats, verification, or a meaningful constraint. Keep entries short.

This file holds decisions that outlast a single change. A decision that only
matters within one change stays in that change's `spec.md`. When a spec
decision will constrain later changes, add a short entry here that links to
the spec.

Preserve the original reasoning when a decision changes. Mark the old entry
superseded and link to the new one. Record rejected alternatives when the
reason is likely to be useful again. Search by mechanism before proposing a
design change.

## Entry pattern

- **Date and title:** YYYY-MM-DD — the decision in a few words.
- **Status:** accepted, rejected, deferred, or superseded (with a link).
- **Context:** the problem and constraint.
- **Decision:** what we chose.
- **Why:** the tradeoff and important alternative.
- **Revisit when:** evidence or a changed condition that would alter the choice.
- **Source:** the spec or PR where the decision was made, if any.

## 2026-11-26 — Model cost and progress increments, not efficiency ratios

- **Status:** accepted. Supersedes the ratio-based formulation of the
  earlier sequential hierarchical Bayesian EVM work.
- **Context:** The earlier model treated periodic κ = ΔAC / ΔEV and
  ξ = Δt / ΔES as the random quantities. A critical review showed they are
  undefined in stall periods, that the recoverability test reduced to
  observed facts, and that intervals on mean efficiency omit future noise.
- **Decision:** Model leaf-level ΔEV (hurdle-Gamma, capped at remaining
  work) and ΔAC (Gamma with a time-driven and a progress-driven term) with
  WBS random effects and a shared project state. κ and ξ are derived
  summaries. Forecasts are posterior predictive simulations to completion.
- **Why:** Stalls become data that identify time-driven cost instead of
  undefined ratios, and predictive intervals include future-period noise.
  Rejected: keeping the ratios and patching the zero-EV case, which leaves
  the interval and recoverability problems in place.
- **Revisit when:** parameter recovery shows h and ρ cannot be separated
  with the data available.
- **Source:** the owner's variable glossary for the generative model;
  [Problem statement](PROBLEM.md).

## 2026-11-26 — Synthetic data only in this repository

- **Status:** accepted.
- **Context:** The mechanism has to be built and validated on a tight
  timeline, and real project data needs separate handling.
- **Decision:** This repository and the paper use synthetic data only. Real
  data is applied later in a separate repository.
- **Why:** Lets the method be developed in the open, keeps the paper free of
  program data, and makes the data contract the handover point.
- **Revisit when:** not for this repository; real data goes to the other
  one. See [Handover](HANDOVER.md).
- **Source:** owner's decision.

## 2026-11-26 — Two synthetic sources with separate jobs

- **Status:** accepted.
- **Context:** Parameter recovery needs data with known model parameters;
  the review also asked for test data that the fitted model did not
  generate.
- **Decision:** This repository simulates its own ground-truth data from
  the generative model. `synthetic_evm` stays a separate, independent
  "realistic world" with no mode that simulates this model.
- **Why:** Keeps the misspecification study honest. Rejected: a model mode
  in `synthetic_evm` that exports true parameters, which would blur the
  two roles.
- **Revisit when:** not expected.
- **Source:** owner's decision.

## 2026-11-26 — Parametric integration of unfinished projects deferred

- **Status:** deferred.
- **Context:** The submitted abstract proposed entering each incomplete
  project into a cost estimating relationship as one row with its
  posterior mean and standard deviation.
- **Decision:** Out of scope for this paper and this repository's first
  version; a future paper.
- **Why:** Naively done, it recycles prior information as data. The
  coherent route, project covariates in the root effects, needs its own
  validation.
- **Revisit when:** the forecasting model is validated.
- **Source:** owner's decision.
