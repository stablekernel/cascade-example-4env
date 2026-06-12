# cascade-example-4env

A live validation example for [cascade](https://github.com/stablekernel/cascade),
exercising a four-environment promotion pipeline: `dev -> test -> staging -> prod`.

![Scenario Suite](https://github.com/stablekernel/cascade-example-4env/actions/workflows/scenario-suite.yaml/badge.svg)

## What this repo demonstrates

This repository carries a complete cascade-generated pipeline and a scenario suite
that drives it end to end. It covers:

- **Four-environment promotion** from `dev` through `test`, `staging`, and `prod`,
  with the release moving from draft, to prerelease at `staging`, to a published
  `latest` release at `prod`.
- **SHA action pinning** (`pin_mode: sha`) so every referenced action is pinned to
  an immutable commit rather than a moving tag.
- **Rollback** of `prod` to a prior version while the published release stays intact.
- **Dry-run promotion** that previews a promotion with no state, tag, or release
  mutation.
- **Breaking-change gate** that blocks a `feat!:` change from promoting past the
  prerelease boundary unless promotion is explicitly allowed.
- **Extra triggers**: scheduled runs, `repository_dispatch`, `workflow_run`, and a
  merge queue lane.
- **Status and reset** verbs for reading deployed state and resetting the repo
  between runs.

## Layout

- `.github/manifest.yaml` - the cascade manifest describing the four environments,
  builds, deploy, and triggers.
- `.github/workflows/orchestrate.yaml` - CI/CD on trunk merges; mints the `dev` draft.
- `.github/workflows/promote.yaml` - operator-driven environment promotions.
- `.github/workflows/cascade-validate.yaml` - manifest validation as a pull-request check.
- `.github/workflows/cascade-merge-queue.yaml` - read-only merge queue validation lane.
- `.github/workflows/cascade-hotfix.yaml` - applies a trunk commit onto a pinned base.
- `.github/workflows/build-app.yaml`, `build-docs.yaml`, `deploy-app.yaml` - callback
  workflows invoked by the pipeline.
- `.github/workflows/scenario-suite.yaml` - the end-to-end driver that walks the full
  stage table and asserts state, tags, and releases after each step.

## Running the scenario suite

The scenario suite runs on a daily schedule, on `repository_dispatch`
(`cascade-revalidate`), and on manual dispatch. It uses the `CASCADE_STATE_TOKEN`
secret for all trunk and release operations.
