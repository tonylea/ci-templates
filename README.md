# ci-templates

Reusable GitHub Actions workflows. Every repo runs the same three stages, linting, testing and versioning, built from these templates for whichever stacks it holds.

## How it fits together

- **Templates** live here, one per stage and stack, each a single job called with `workflow_call`.
- **Stage files** live in each repo: `linting.yml`, `testing.yml` and `versioning.yml`, plus any extra stages such as `building.yml`. Each calls the templates for the stacks that repo holds.
- **A gate job** ends each pull request stage file and is named after the stage. It is the required check, so the checks on `main` stay `linting` and `testing` whatever stacks a repo holds. Skipped jobs pass the gate; failed or cancelled ones don't.
- **Versioning** runs on pushes to `main` only. It checks nothing on a pull request, so it isn't a required check and needs no gate.

## Naming

GitHub only finds reusable workflows directly in `.github/workflows/`, so the name carries the structure:

```text
<stage>-<stack>[-<kind>].yml
```

`common` is the stack every repo uses. Stage files have no stack: `linting.yml`, `testing.yml`, `versioning.yml`.

## Templates

| Template | Does | Inputs |
| --- | --- | --- |
| `linting-common.yml` | Commit messages, workflows (actionlint), Markdown, spelling, secrets. Pull requests only. | none |
| `linting-ansible.yml` | ansible-lint, installing collections from `requirements.yml` if there is one | `working-directory` |
| `testing-ansible-molecule.yml` | `molecule test` | `working-directory`, `runs-on` (ARM by default) |
| `versioning-common.yml` | commit-and-tag-version, then pushes the release commit and tag and creates the GitHub release. Pushes to `main` only. | secret `release-deploy-key` |

The tools come from each repo's own `package.json` and `requirements-*.txt`, the same versions its local hooks use. A repo names the files to bump in its `.versionrc.json`.

## Using them

Pin an exact release, never `main`:

```yaml
# .github/workflows/linting.yml
name: Linting

on:
  pull_request:

jobs:
  common:
    uses: tonylea/ci-templates/.github/workflows/linting-common.yml@v1.0.0

  ansible:
    uses: tonylea/ci-templates/.github/workflows/linting-ansible.yml@v1.0.0

  linting:
    needs: [common, ansible]
    if: always()
    runs-on: ubuntu-24.04
    steps:
      - name: Every linting job passed
        if: contains(needs.*.result, 'failure') || contains(needs.*.result, 'cancelled')
        run: exit 1
```

`versioning.yml` runs on pushes to `main` only and grants what the template needs:

```yaml
# .github/workflows/versioning.yml
name: Versioning

on:
  push:
    branches: [main]

jobs:
  common:
    uses: tonylea/ci-templates/.github/workflows/versioning-common.yml@v1.0.0
    permissions:
      contents: write
    secrets:
      release-deploy-key: ${{ secrets.RELEASE_DEPLOY_KEY }}
```

## Changing a template

Callers pin releases, so a change only reaches a repo when its pin is bumped. Changing a template's inputs or what it needs from callers breaks them: commit it as a breaking change (`feat!:`) so it gets a major version.

This repo's own stage files call the templates from the same commit, and `tests/` holds what they run against, so a pull request tests its template changes for real.
