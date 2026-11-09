# Contributing

Thanks for your interest in improving this project.

## Reporting a problem

Open an issue using the **Bug report** template.
Say what you expected, what happened, and how to reproduce it.

## Suggesting a change

1. For anything bigger than a small fix, open an issue using the **Feature request** template first, so the approach is agreed before you spend time on it.
2. Fork the repository and create a branch from `main`.
3. Run `npm install` once.
   It installs the Git hooks that check commit messages, Markdown, spelling and secrets before each commit.
4. Make your change, with tests where it changes behaviour.
5. Open a pull request against `main`.

## Commit messages

Commits follow [Conventional Commits](https://www.conventionalcommits.org/), for example `fix: keep the locale link systemd creates`.
The type says what changed:

- `feat`: new behaviour
- `fix`: a bug fix
- `docs`, `test`, `ci`, `chore`, `refactor`: changes that don't alter behaviour

Releases and the changelog are generated from commit messages.
Pull requests that change version numbers or the changelog will not be merged.

## Checks

Every pull request runs linting and testing.
Both must pass before it can be merged.

## Licence

Contributions are licensed under the licence in `LICENSE`.
