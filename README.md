# OneLiteFeatherNET Reusable Workflows

Central collection of reusable GitHub Actions workflows shared across
OneLiteFeatherNET repositories (Butterfly, Aonyx-bom, Titan, ...).

Each workflow is exposed via `workflow_call` and consumed from downstream
repositories by referencing a tagged release of this repo.

## Available workflows

| Workflow | Purpose |
| --- | --- |
| `.github/workflows/gradle-build-pr.yml` | Build & test a Gradle project on pull requests across a runner matrix. Skips when no Gradle-relevant files changed; aggregates JUnit results across the matrix; auto-enables verbose logging on debug re-runs. |
| `.github/workflows/gradle-publish.yml` | Build & publish a Gradle project to the OneLiteFeather Maven repository on tag pushes. |
| `.github/workflows/release-please.yml` | Run [release-please](https://github.com/googleapis/release-please) for a repository. |
| `.github/workflows/close-invalid-prs.yml` | Close PRs opened from a fork's default branch with a configurable message. |
| `.github/workflows/markdown-lint.yml` | Lint Markdown files with [`markdownlint-cli2`](https://github.com/DavidAnson/markdownlint-cli2-action) and check links with [`lychee`](https://github.com/lycheeverse/lychee-action). |

## Defaults at a glance

- Java **25** on Temurin.
- Three-OS matrix on PRs: `ubuntu-latest`, `windows-latest`, `macos-latest`.
- `gradle-build-pr` runs `build test` by default (no `clean`, to keep incremental caches). Set `run-tests: false` for BOM-only / test-less projects: the default task drops to `build` and the JUnit aggregation step is skipped.
- Path filter: build only runs when files under `src/`, `*.gradle*`, `buildSrc/`, JVM sources, or `.github/workflows/**` changed.
- Debug re-runs (`Re-run with debug logging`) automatically activate `--info --stacktrace`.
- Test reports are uploaded as artifacts on every run and aggregated into a unified check + PR comment.
- Concurrency cancels superseded PR runs; publish runs are never cancelled mid-flight.
- `markdown-lint` filters on `.md`/`.markdownlint*`/`.lycheeignore` paths and runs markdownlint + lychee in parallel.

## Versioning

This repository is released via [release-please](https://github.com/googleapis/release-please).
Pin consumers to a concrete tag (e.g. `@v2.0.1`) rather than `main` for
reproducible builds. There is intentionally no floating major alias such as
`@v2` — consumers pin to a full `vX.Y.Z` and let Renovate bump them.

Use [Renovate](https://docs.renovatebot.com/modules/manager/github-actions/) in
your consumer repository to auto-bump the version pin. The `github-actions`
manager picks up `uses: OneLiteFeatherNET/workflows/.github/workflows/foo.yml@vX.Y.Z`
out of the box. A sample `renovate.json` is shipped at the root of this repo;
copy it as a starting point.

## Usage examples

> All examples use the current latest tag `v2.0.1`. Substitute the freshest
> release (see [Releases](https://github.com/OneLiteFeatherNET/workflows/releases))
> or rely on Renovate to bump.

### Build a PR (Gradle)

```yaml
name: Build PR
on: [pull_request]

jobs:
  build:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-build-pr.yml@v2.0.1
    secrets: inherit
```

Single-OS, custom JDK, force-build:

```yaml
jobs:
  build:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-build-pr.yml@v2.0.1
    with:
      java-version: "21"
      runs-on: '["ubuntu-latest"]'
      force-build: true
    secrets: inherit
```

BOM-only or test-less project:

```yaml
jobs:
  build:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-build-pr.yml@v2
    with:
      run-tests: false
    secrets: inherit
```

Custom path filter (must define a `code:` key):

```yaml
jobs:
  build:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-build-pr.yml@v2.0.1
    with:
      paths-filters: |
        code:
          - 'src/**'
          - 'build.gradle.kts'
          - 'gradle/**'
    secrets: inherit
```

### Publish on tag (Gradle)

```yaml
name: Publish JAR
on:
  push:
    tags: ["v*"]

jobs:
  publish:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-publish.yml@v2.0.1
    secrets: inherit
```

### release-please

```yaml
name: release-please
on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    uses: OneLiteFeatherNET/workflows/.github/workflows/release-please.yml@v2.0.1
```

### Close invalid PRs

```yaml
name: Close invalid PRs
on:
  pull_request_target:
    types: [opened]

jobs:
  close:
    uses: OneLiteFeatherNET/workflows/.github/workflows/close-invalid-prs.yml@v2.0.1
    with:
      protected-branch: main
```

### Markdown lint

```yaml
name: Lint docs
on:
  pull_request:
    paths:
      - '**/*.md'
      - '.markdownlint.json'
      - '.lycheeignore'

jobs:
  lint:
    uses: OneLiteFeatherNET/workflows/.github/workflows/markdown-lint.yml@v2.0.1
    with:
      force-lint: true
```

A `.markdownlint.json` and optional `.lycheeignore` (regex per line) at the
repo root configure rules and skip-lists.

## Required secrets

Workflows that publish or read from the OneLiteFeather Maven repository expect
these secrets to be available in the caller repository (and forwarded via
`secrets: inherit`):

- `ONELITEFEATHER_MAVEN_USERNAME`
- `ONELITEFEATHER_MAVEN_PASSWORD`

## Test results

For `gradle-build-pr`, JUnit XML from every matrix job is uploaded as
`test-reports-<os>-jdk<version>` and merged by a downstream `test-report` job
that uses [`EnricoMi/publish-unit-test-result-action`](https://github.com/marketplace/actions/publish-test-results)
to post a unified check and PR comment. Each matrix runner's artifact extracts
into its own subdirectory so identically-named JUnit XML files from different
OSes never collide during aggregation.

## Debugging

When a workflow run fails, press **Re-run with debug logging** in the GitHub UI.
The reusable workflows detect `RUNNER_DEBUG=1` automatically and switch Gradle
to `--info --stacktrace`. Build summaries (test counts, build scan link) are
posted to the run summary and as PR comments via
[`gradle/actions/setup-gradle`](https://github.com/gradle/actions/blob/main/docs/setup-gradle.md).

## Contributing

- Conventional Commits are required (`feat:`, `fix:`, `chore:`, ...).
- Breaking changes use `feat!:` or `BREAKING CHANGE:` to trigger a major bump.
- Releases are produced automatically by release-please on merge to `main`.
- **PR titles must also be conventional** — GitHub uses the PR title as the
  squash-merge commit subject, and release-please only picks up commits with a
  proper Conventional prefix. A non-conventional squash title (e.g.
  `Remove merge-multiple option`) silently skips the release.

## License

MIT — see [LICENSE](LICENSE).
