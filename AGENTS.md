# Agent instructions

This is the canonical source of project instructions for coding assistants.
Tool-specific entry points should reference it instead of copying its rules.

## Get oriented

1. Read `README.md` and `docs/NEXT.md` for purpose and current work, and
   `intent/intent.md` for goals, scope, and success criteria. For a change
   with its own `intent.md`, `spec.md`, or `plan.md`, read those too (see
   "From intent to merge" in `docs/DEVELOPMENT.md`).
2. Read `docs/DEVELOPMENT.md` for commands and verification boundaries.
3. Before a design change, search `docs/DECISIONS.md` by mechanism or concept.
   Preserve rejected approaches and explain any decision you supersede.

## Work on the requested change

- Prefer the simplest correct implementation that makes the reasoning clear.
- Keep domain logic separate from file access, presentation, and integrations.
- State mathematical conventions, assumptions, units, and tolerances where
  relevant. Cite a source when implementing a nontrivial published method.
- Use type hints on public Python functions. Explain non-obvious behavior in
  docstrings; use NumPy-style sections where useful.
- Keep changes focused. Avoid speculative frameworks, unrelated cleanup, or
  dependencies without a concrete need.
- Preserve existing uncommitted work and report any conflict with it.
- Implement from the approved `plan.md` when one exists. When the work
  departs from it, update `plan.md` in the same commit.
- Propose changes to an `intent.md` to the owner, since it records the
  owner's own statement of the work.

## Verify what changed

- Before reporting a code change complete, run each check below and report
  each result on its own. If a test fails, fix the code, not the test.

  | Check | Command | Healthy output ends with |
  | --- | --- | --- |
  | Lint | `uv run --locked ruff check .` | `All checks passed!` |
  | Format | `uv run --locked ruff format --check .` | `N files already formatted` |
  | Tests | `uv run --locked pytest -m "not property"` | `N passed` and no failures |

  These are the same commands CI runs as the Lint, Format, and Tests jobs.
  `poe check` is the owner's shortcut for them; report the separate results.
- Edit a committed test only when the test itself is wrong. Such an edit
  asks for the owner's approval.
- Add tests for changed behavior and regression risks. Prefer invariants,
  small explicit cases, and independent expected results to implementation
  snapshots. Formatting-only or prose edits do not need new tests.
- For generated deliverables, rebuild first and inspect the newly generated
  output. A check of an old artifact is not evidence about the new code.
- Report exactly what ran, what passed, and what could not run. Mocks and
  structural checks do not establish integration or numerical correctness.
- Keep repeatable verification commands in `pyproject.toml`; CI calls them.

## Leave useful context

- Explain the result and its reason, with the relevant validation evidence.
- Record consequential tradeoffs in `docs/DECISIONS.md`. Routine edits do not
  need decision entries.
- Update `docs/NEXT.md` when stopping with unfinished work. Include a specific
  next action and any blocker; do not append a session transcript.
- Deliver changes on a branch as a PR; the owner reviews and merges.
- Publishing, merging, or sending messages follows the owner's instructions.
  These repository notes are not blanket authorization for those actions.

## Respond to PR review

- Monitor each PR you open and respond to every review comment, from people
  and from bots.
- Respond with either a fix or an explanation of why the comment is incorrect
  or irrelevant, and reply on the comment's thread. When you applied a fix,
  name the commit or PR in the reply.
- Land each fix on the reviewed PR's own branch while that PR is open: a
  commit for a quick fix, or a PR into that branch for a significant change.
  The owner then squash-merges the reviewed PR into main with every fix in it.
- Check a bot's finding against the code before acting on it, and reproduce a
  reported bug before fixing it.
- Resolve a conversation once its fix is pushed, or once your reply explains
  why you are leaving the code as it is.
- Bring judgment calls to the owner and leave those conversations open. These
  include design or scope changes, security findings, and disagreements with
  the owner's own comments.
