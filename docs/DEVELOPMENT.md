# Development

## Commands

The project uses uv for its environment and lockfile, Ruff for lint and
formatting, and pytest for tests. The development Python version is in
`.python-version`.

Each check is one uv command. CI, the git hook, and `AGENTS.md` call these
commands directly, written the same way everywhere, so a local pass means the
same as a CI pass. Poe tasks in `pyproject.toml` are shortcuts for the owner;
nothing automated calls them.

| Purpose | Command | Poe shortcut |
| --- | --- | --- |
| Set up a new clone | `uv sync --locked`, then `git config core.hooksPath .githooks` | `poe setup` |
| Lint | `uv run --locked ruff check .` | `poe lint` |
| Check formatting | `uv run --locked ruff format --check .` | `poe format-check` |
| Apply formatting | `uv run --locked ruff format .` | `poe format` |
| Unit tests | `uv run --locked pytest -m "not property"` | `poe test` |
| All three checks | | `poe check` |
| Add a runtime dependency | `uv add PACKAGE` | |
| Add a development dependency | `uv add --dev PACKAGE` | |
| Deliberately refresh dependencies | `uv lock --upgrade` | |

Run Poe as `uv run --locked poe TASK`. Commit dependency metadata and
`uv.lock` together, and rerun the checks after updates. `--locked` makes a
stale `uv.lock` fail the run instead of silently re-resolving it. Review the
dependency diff before merging it.

Property-based tests get the `property` pytest marker and their own CI job
once the first ones exist, since they tend to slow down as the code grows.
Until then there is no property job: `pytest -m property` with nothing to
collect exits non-zero.

## Where each check runs

Only the edit-time hook changes files. Every later layer only checks, so a
commit or CI run never rewrites code that was already reviewed.

| When | What runs | Where it is defined |
| --- | --- | --- |
| Claude edits a `.py` file | `ruff format` on that file, then `ruff check` on it, reported without `--fix` | `.claude/hooks/format_on_edit.py` |
| Claude edits a committed test | Asks the owner first; new tests stay free to edit | `.claude/hooks/ask_before_test_edit.py` |
| Claude edits `uv.lock` or an `intent.md` | Denied, or asks the owner first | `permissions` in `.claude/settings.json` |
| Any commit | Format check and lint, in seconds | `.githooks/pre-commit` |
| Claude runs `git commit` | Blocks `--no-verify`, `-n`, and `core.hooksPath` overrides | `.claude/hooks/block_no_verify.py` |
| Before work is reported done | Lint, format check, and unit tests | `AGENTS.md` |
| Every PR and push to main | The Lint, Format, and Tests jobs | `.github/workflows/ci.yml` |
| Merge to main | The three jobs must pass | The `main` ruleset (below) |

Lint stays report-only at edit time because Claude often adds an import in one
edit and uses it in the next; an autofix in between would delete it as unused.

Git does not enable hooks from a clone by itself, so `core.hooksPath` is set
per clone. A Claude Code `SessionStart` hook sets it for every Claude session,
local or cloud, and `poe setup` sets it for the owner. Other tools depend on
`poe setup` having run, with CI as the backstop. `core.hooksPath` replaces
`.git/hooks`, so a tool that installs hooks there (Git LFS, for example) would
need its hook added to `.githooks/`.

The Claude Code hooks do not see edits made through shell commands. The
ruleset on main is the backstop.

## GitHub settings

These live in the repository settings, not in files, so they are recorded
here.

- **Merges:** squash only, with the PR title and description as the default
  commit message. Merged branches are deleted automatically.
- **Ruleset `main`:** targets the default branch. It requires a pull request
  with zero approvals, requires the Lint, Format, and Tests checks, blocks
  force pushes and deletion, and requires linear history. The bypass list is
  empty.

Zero approvals is deliberate: GitHub does not let an author approve their own
PR, so the owner's merge is the approval. The empty bypass list matters
because agent sessions act with the owner's GitHub permissions; a bypass for
the owner would be one for every agent too. When CI gains a job, merge the
workflow change before adding the job to the ruleset, or every PR waits on a
check that never reports.

## From intent to merge

This project follows the stages of the AI-native SDLC playbook. Each stage ends
by committing an artifact, and the next stage starts by reading it.

| Stage | Artifact | Decision point |
| --- | --- | --- |
| Plan | `intent.md`: problem, outcome, scope, constraints, open questions | Owner sets `status: accepted` |
| Design | `spec.md`: requirements and design, with flagged concerns | Owner accepts; open questions answered or carried forward |
| Build | `plan.md`: files that change, order of work, risks, and how each step is proven | Owner approves before code is written |
| Test | Tests, check output, and eval results where they exist | Output shown in the PR |
| Release | The PR, with any review findings | Owner merges |

Keep each change's `intent.md`, `spec.md`, and `plan.md` together, so the set
records what was asked, what was decided, and how it was built. The project's
founding set lives directly in `intent/`. Later changes get their own folder,
such as `intent/YYYY-MM-short-name/`. Start each document from its skeleton in
`docs/templates/`.

- **`intent.md`** is the owner's own statement of what and why. Agents
  propose changes to it rather than make them.
- **`spec.md`** says what the software does and the decisions that shape it.
  Keep it to a few pages, with exact contracts and acceptance criteria.
- **`plan.md`** is written per PR, detailed enough for someone who never saw
  the conversation to carry it out.

During Build and Test, each change follows the same loop:

1. Work on a short-lived branch.
2. Implement the change with a test that could detect a real mistake. When
   the work departs from `plan.md`, update the plan in the same commit.
3. Run the relevant focused test, then the checks listed in `AGENTS.md`.
4. Read the diff yourself. Check assumptions, boundary cases, error behavior,
   dependencies, and anything an AI reviewer missed.
5. Open a PR; the owner merges it and GitHub deletes the merged branch. A PR
   is a useful checkpoint even when you are its only human reviewer.

Small fixes can skip the intent, spec, and plan and go straight to a PR. The
`main` ruleset requires a PR for every change, prose fixes included. An issue
is useful for a reproducible bug or a larger chunk of work; it need not
precede every edit.

## Tests that earn their maintenance cost

- Test observable behavior and mathematical invariants.
- Use hand-computable cases or an independent reference implementation for
  numerical results. Record tolerances and their reason.
- Include invalid inputs, degenerate cases, and past failure cases.
- A bug fix should normally have a test that fails without the fix.
- Use mocks to isolate boundaries, then test the real boundary when needed.
- Add property-based tests, type checking, coverage reporting, and additional
  Python/OS CI jobs when they address a specific risk. A coverage percentage
  is a diagnostic, not proof of correctness.

The CI jobs use Python 3.12 on Linux. The project permits Python 3.12
and later, but makes no claim to have tested every permitted version. Add a
supported-version matrix when this becomes a library used by other people.
If Windows or another platform is part of the product contract, test it too.

## Deliverables and integrations

For a workbook, UI, native dependency, GPU, or external service, add a specific
verification command and document its environment requirement here. Keep it
separate from headless checks. A missing required environment is a reported
limitation, not a passing integration check.

For generated files, use the order: build, verify that output, then publish.
If a check cannot run in CI, retain concise evidence tied to the commit,
command, environment, and artifact. Save useful failures as well as successes;
keep routine terminal logs out of the repository.

Keep source and generated output ownership clear. Prefer release assets for
large distributables; commit a generated artifact only when it serves a clear
purpose. Do not hand-edit a generated file whose source can be rebuilt.

## Resume without rereading a conversation

Three places hold the project's state between sessions:

- **The change documents** (`intent.md`, `spec.md`, `plan.md`) hold the detail
  for each change.
- **`docs/NEXT.md`** points to the active change, its stage, and the next
  action. It links to the change documents instead of restating them.
- **`docs/DECISIONS.md`** holds decisions that outlast a single change.

Put detailed work in issues if needed, and link to them instead of maintaining
two backlogs. Add architecture documentation when actual boundaries become
worth explaining.

## Release when there is something to release

Choose a license, document an example, verify the actual deliverable, and tag
the release. Keep a short changelog once users rely on versioned behavior.
Mark incompatible changes explicitly. Add publishing automation when the
manual sequence is stable and repeated often enough to justify it.

Dependabot is configured for monthly grouped uv and Actions update PRs, with
one open version-update PR per ecosystem. Review and test them; nothing here
auto-merges or publishes a release. Configure GitHub security alerts separately
where available so important notices do not wait for a monthly maintenance pass.
