# Project name

TODO: Explain who this project helps, what it does, and its current limits.

This starter is tailored for solo Python development with AI assistance.
For its first use, follow [TEMPLATE_SETUP.md](TEMPLATE_SETUP.md).

## Start working

Install [uv](https://docs.astral.sh/uv/getting-started/installation/), then:

```sh
uv run --locked poe setup
uv run --locked poe check
```

`setup` installs the locked environment and turns on the repository's commit
hook. `check` runs lint, the format check, and the unit tests. GitHub Actions
runs the same three checks as separate jobs; see
[Development](docs/DEVELOPMENT.md) for the commands behind them. The starter's
tests cover the package import and the Claude Code hooks; none validate
application behavior. Add behavior tests with your first feature.

## Use the project

TODO: Add one real example with expected output before the first release.

## Find your place

- [Intent](intent/intent.md): problem, priorities, scope, and success criteria.
- [Development](docs/DEVELOPMENT.md): commands, tests, review, and releases.
- [Decisions](docs/DECISIONS.md): why important choices were made.
- [Next steps](docs/NEXT.md): the current goal and the next useful action.
- [Agent instructions](AGENTS.md): shared rules for coding assistants.

## License

Choose and add the project's license before inviting public reuse.
