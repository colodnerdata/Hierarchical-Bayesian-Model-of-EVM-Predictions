# Hierarchical Bayesian Model of EVM Predictions

Forecasts of final cost and completion date for projects still under way,
built from their earned value management (EVM) histories. Instead of a
single index-based estimate such as EAC = BAC / CPI, the model describes how
cost and progress accrue at each lowest-level work breakdown structure
(WBS) element, lets elements borrow strength from their parents, and
simulates the rest of the project to give full predictive distributions:
intervals for estimate and duration at completion at any WBS level, and the
probability of finishing within budget or on time.

It is written for cost estimators and project controls analysts, and it
supports a planned AACE International conference paper.

**Current limits.** No model code exists yet; the repository is at the
problem-statement stage. All data is synthetic. Real project data never
enters this repository; applying the method to real data happens later in a
separate repository. See the [problem statement](docs/PROBLEM.md) for the
research question, scope and non-goals.

## Start working

Install [uv](https://docs.astral.sh/uv/getting-started/installation/), then:

```sh
uv run --locked poe setup
uv run --locked poe check
```

`setup` installs the locked environment and turns on the repository's commit
hook. `check` runs lint, the format check, and the unit tests. GitHub Actions
runs the same three checks as separate jobs; see
[Development](docs/DEVELOPMENT.md) for the commands behind them. The current
tests cover the package import and the Claude Code hooks; none validate
model behavior yet.

## Use the project

An example forecast with expected output will be added with the first
model release.

## Find your place

- [Problem statement](docs/PROBLEM.md): the research question, why the
  earlier ratio-based model was replaced, scope and non-goals.
- [Intent](intent/intent.md): the owner's statement of priorities, scope,
  and success criteria.
- [Handover](docs/HANDOVER.md): patterns that let later real-data work
  reuse this repository unchanged.
- [Development](docs/DEVELOPMENT.md): commands, tests, review, and releases.
- [Decisions](docs/DECISIONS.md): why important choices were made.
- [Next steps](docs/NEXT.md): the current goal and the next useful action.
- [Agent instructions](AGENTS.md): shared rules for coding assistants.

## Related work

- The earlier sequential hierarchical Bayesian EVM model, whose ratio-based
  formulation this project replaces (see
  [Decisions](docs/DECISIONS.md)).
- `synthetic_evm`, the owner's synthetic EVM data generator, planned as a
  development dependency.

## License

No license has been chosen yet. Until one is added, all rights are
reserved.
