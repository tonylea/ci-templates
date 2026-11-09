# Changelog

All notable changes to this project will be documented in this file. See [commit-and-tag-version](https://github.com/absolute-version/commit-and-tag-version) for commit guidelines.

## [1.2.0](https://github.com/tonylea/ci-templates/compare/v1.1.1...v1.2.0) (2026-11-09)

### Features

* report whether a pull request touched an Ansible role's code, tests or CI ([40067e9](https://github.com/tonylea/ci-templates/commit/40067e9ad11308f83f07d95d79adbbe818b355eb))

### Documentation

* add a contributing guide ([1f702ff](https://github.com/tonylea/ci-templates/commit/1f702ff0ddc4856e56bb914d9a23c87e084b3567))
* put each sentence of the README on its own line ([f3679f1](https://github.com/tonylea/ci-templates/commit/f3679f1dbe631dd01464a46808c74803ee0e5e77))

## [1.1.1](https://github.com/tonylea/ci-templates/compare/v1.1.0...v1.1.1) (2026-11-08)

### Bug Fixes

* install collections without ansible-core's Galaxy response cache, which fails at random ([d565b99](https://github.com/tonylea/ci-templates/commit/d565b99529d791dcd56825a1d3487fb6cc96a233))

## [1.1.0](https://github.com/tonylea/ci-templates/compare/v1.0.0...v1.1.0) (2026-11-08)

### Features

* turn the runner's swap off before Molecule, as the nodes run without it ([ccdfcb8](https://github.com/tonylea/ci-templates/commit/ccdfcb8c9115a224167bccec31552983829ebd84))

### Documentation

* versioning runs on main only and isn't a required check ([0727600](https://github.com/tonylea/ci-templates/commit/072760063cc75d9e239de339f08891ae0719414a))

## 1.0.0 (2026-11-08)

### Features

* add linting, testing and versioning templates for common and Ansible repos ([a92b2ee](https://github.com/tonylea/ci-templates/commit/a92b2ee26d528663d35b941bca7167de9ce61ba7))
