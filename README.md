# OneLiteFeatherNET Reusable Workflows

Central collection of reusable GitHub Actions workflows shared across
OneLiteFeatherNET repositories (Butterfly, Aonyx-bom, ...).

Each workflow is exposed via `workflow_call` and consumed from downstream
repositories by referencing a tagged release of this repo.

## Available workflows

| Workflow | Purpose |
| --- | --- |
| `.github/workflows/gradle-build-pr.yml` | Build & test a Gradle project on pull requests across a runner matrix. Skips when no Gradle-relevant files changed; aggregates JUnit results across the matrix; auto-enables verbose logging on debug re-runs. |
| `.github/workflows/gradle-publish.yml` | Build & publish a Gradle project to the OneLiteFeather Maven repository on tag pushes. |
| `.github/workflows/release-please.yml` | Run [release-please](https://github.com/googleapis/release-please) for a repository. |
| `.github/workflows/close-invalid-prs.yml` | Close PRs opened from a fork's default branch with a configurable message. |

## Defaults at a glance

- Java **25** on Temurin.
- Three-OS matrix on PRs: `ubuntu-latest`, `windows-latest`, `macos-latest`.
- `gradle-build-pr` runs `build test` (no `clean`, to keep incremental caches).
- Path filter: build only runs when files under `src/`, `*.gradle*`, `buildSrc/`, JVM sources, or `.github/workflows/**` changed.
- Debug re-runs (`Re-run with debug logging`) automatically activate `--info --stacktrace`.
- Test reports are uploaded as artifacts on every run and aggregated into a unified check + PR comment.
- Concurrency cancels superseded PR runs; publish runs are never cancelled mid-flight.

## Versioning

This repository is released via [release-please](https://github.com/googleapis/release-please).
Pin consumers to a tag (e.g. `@v2.0.0`) or a major (e.g. `@v2`) rather than
`main` for reproducible builds.

Use [Renovate](https://docs.renovatebot.com/modules/manager/github-actions/) in
your consumer repository to auto-bump the version pin. The `github-actions`
manager picks up `uses: OneLiteFeatherNET/workflows/.github/workflows/foo.yml@vX`
out of the box. A sample `renovate.json` is shipped at the root of this repo;
copy it as a starting point.

## Usage examples

### Build a PR (Gradle)

```yaml
name: Build PR
on: [pull_request]

jobs:
  build:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-build-pr.yml@v2
    secrets: inherit
```

Single-OS, custom JDK, force-build:

```yaml
jobs:
  build:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-build-pr.yml@v2
    with:
      java-version: "21"
      runs-on: '["ubuntu-latest"]'
      force-build: true
    secrets: inherit
```

Custom path filter (must define a `code:` key):

```yaml
jobs:
  build:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-build-pr.yml@v2
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
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-publish.yml@v2
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
    uses: OneLiteFeatherNET/workflows/.github/workflows/release-please.yml@v2
```

### Close invalid PRs

```yaml
name: Close invalid PRs
on:
  pull_request_target:
    types: [opened]

jobs:
  close:
    uses: OneLiteFeatherNET/workflows/.github/workflows/close-invalid-prs.yml@v2
    with:
      protected-branch: main
```

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
to post a unified check and PR comment.

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

## License

MIT — see [LICENSE](LICENSE).
