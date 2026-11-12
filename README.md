# ci-templates

Reusable GitHub Actions workflows.
Every repo runs the same three stages, linting, testing and versioning, built from these templates for whichever stacks it holds.

## How it fits together

- **Templates** live here, one per stage and stack, each a single job called with `workflow_call`.
- **Stage files** live in each repo: `linting.yml`, `testing.yml` and `versioning.yml`, plus any extra stages such as `building.yml`.
  Each calls the templates for the stacks that repo holds.
- **A gate job** ends each pull request stage file and is named after the stage.
  It is the required check, so the checks on `main` stay `linting` and `testing` whatever stacks a repo holds.
  Skipped jobs pass the gate; failed or cancelled ones don't.
- **Versioning** runs on pushes to `main` only.
  It checks nothing on a pull request, so it isn't a required check and needs no gate.

## Naming

GitHub only finds reusable workflows directly in `.github/workflows/`, so the name carries the structure:

```text
<stage>-<stack>[-<kind>].yml
```

`common` is the stack every repo uses.
Stage files have no stack: `linting.yml`, `testing.yml`, `versioning.yml`.

## Templates

| Template | Does | Inputs |
| --- | --- | --- |
| `linting-common.yml` | Commit messages, workflows (actionlint), Markdown, spelling, secrets. Pull requests only. | none |
| `linting-ansible.yml` | ansible-lint, installing collections from `requirements.yml` if there is one | `working-directory` |
| `testing-ansible-molecule.yml` | Turns the runner's swap off, as on the nodes, then `molecule test` | `working-directory`, `runs-on` (ARM by default) |
| `changes-ansible-role.yml` | Reports (output `role`) whether a pull request touched Ansible's standard role folders, `molecule/`, the requirements files, `.ansible-lint` or the workflows. Not a stage: a role's stage files run their Ansible jobs only when it's `'true'`. | none |
| `linting-helm.yml` | `helm lint --strict`, then `helm template` checked by kubeconform (`-strict`) against the cluster's Kubernetes version, custom resources against `datreeio/CRDs-catalog`. Once per `ci/*-values.yaml`, or once with the defaults if there are none. | `working-directory`, `kubernetes-version` |
| `testing-helm-unittest.yml` | helm-unittest, from the chart's `tests/*_test.yaml` | `working-directory` |
| `testing-helm-kind.yml` | Applies the chart's setup kustomization (CRDs, Secrets) into the namespace `ct` of a kind cluster on ARM, then `ct install --upgrade` there, once per `ci/*-values.yaml`. Pull requests only. | `working-directory`, `setup-path` (`ci/kind` in the chart by default) |
| `changes-helm-chart.yml` | Reports (output `chart`) whether a pull request touched anything but Markdown, `LICENSE` and the repo's own tooling. Not a stage: a chart's stage files run their Helm jobs only when it's `'true'`. | none |
| `versioning-common.yml` | commit-and-tag-version, then pushes the release commit and tag and creates the GitHub release. Pushes to `main` only. | secret `release-deploy-key`; output `version`, the tag released or empty |
| `versioning-helm.yml` | Packages the released chart, attaches it to the GitHub release, and merges it into the Helm repository's `index.yaml` on the repo's GitHub Pages. Call it after `versioning-common.yml`, only when that released. | `version`, `working-directory` |

The tools come from each repo's own `package.json` and `requirements-*.txt`, the same versions its local hooks use; the Helm tools are pinned in the templates.
A repo names the files to bump in its `.versionrc.json`; a chart repo lists `Chart.yaml` there, so the chart's version is the repo's release.

## Using them

Pin an exact release, never `main`:

```yaml
# .github/workflows/linting.yml
name: Linting

on:
  pull_request:

jobs:
  changes:
    uses: tonylea/ci-templates/.github/workflows/changes-ansible-role.yml@v1.3.0

  common:
    uses: tonylea/ci-templates/.github/workflows/linting-common.yml@v1.3.0

  ansible:
    needs: [changes]
    if: needs.changes.outputs.role == 'true'
    uses: tonylea/ci-templates/.github/workflows/linting-ansible.yml@v1.3.0

  linting:
    needs: [changes, common, ansible]
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
    uses: tonylea/ci-templates/.github/workflows/versioning-common.yml@v1.3.0
    permissions:
      contents: write
    secrets:
      release-deploy-key: ${{ secrets.RELEASE_DEPLOY_KEY }}
```

A chart repo publishes each release to the Helm repository on its GitHub Pages, which must deploy from GitHub Actions:

```yaml
# .github/workflows/versioning.yml
name: Versioning

on:
  push:
    branches: [main]

jobs:
  release:
    uses: tonylea/ci-templates/.github/workflows/versioning-common.yml@v1.3.0
    permissions:
      contents: write
    secrets:
      release-deploy-key: ${{ secrets.RELEASE_DEPLOY_KEY }}

  publish:
    needs: [release]
    if: needs.release.outputs.version != ''
    uses: tonylea/ci-templates/.github/workflows/versioning-helm.yml@v1.3.0
    permissions:
      contents: write
      pages: write
      id-token: write
    with:
      version: ${{ needs.release.outputs.version }}
```

## Changing a template

Callers pin releases, so a change only reaches a repo when its pin is bumped.
Changing a template's inputs or what it needs from callers breaks them: commit it as a breaking change (`feat!:`) so it gets a major version.

This repo's own stage files call the templates from the same commit, and `tests/` holds what they run against, so a pull request tests its template changes for real.
