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
| `.github/workflows/docker-publish.yml` | Build a container image and push it to the OneLiteFeather Harbor registry using **chunked blob uploads** (via [`regctl`](https://regclient.org/)) so no single request exceeds the proxy body limit; optionally **keyless-signs** the image with cosign (GitHub OIDC). |
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

### Publish a Docker image (chunked upload)

Pushes the image one blob at a time in chunks below the proxy body limit, so
large layers no longer fail with `413 Request Entity Too Large` / `504 Gateway
Timeout` behind a proxy such as Cloudflare (100 MB limit). Plain `docker push`
/ `buildx` cannot chunk a blob; this workflow builds the image to an OCI archive
and pushes it with [`regctl`](https://regclient.org/) (`--blob-chunk` /
`--blob-max`) instead.

Typical use is from a release job, gated on `release_created`:

Typical use is from a release job, gated on `release_created`. Grant
`id-token: write` on the calling job so cosign can sign keyless:

```yaml
jobs:
  docker:
    needs: release-please
    if: needs.release-please.outputs.release_created == 'true'
    permissions:
      contents: read
      id-token: write                  # required for keyless cosign signing
    uses: OneLiteFeatherNET/workflows/.github/workflows/docker-publish.yml@v2
    with:
      image-name: "otis/otis"          # registry host comes from HARBOR_REGISTRY
      version: ${{ needs.release-please.outputs.version }}
      # Build the container context with Gradle first ($VERSION is exported):
      setup-java: true
      build-command: "./gradlew jar optimizedBuildLayers optimizedDockerfile -Pversion=$VERSION"
      context: "./backend/build/docker/optimized"
    secrets: inherit
```

For a project with a plain `Dockerfile` checked into the repo, drop the Gradle
inputs (and the `permissions` block if you set `sign: false`):

```yaml
jobs:
  docker:
    uses: OneLiteFeatherNET/workflows/.github/workflows/docker-publish.yml@v2
    with:
      image-name: "myteam/myapp"
      version: "1.2.3"
      context: "."
      sign: false                      # skip signing (no id-token needed)
    secrets: inherit
```

Default tags are `{{version}}`, `{{major}}.{{minor}}`, `{{major}}` and a
`sha-` tag; add more via `extra-tags`. Tune chunking with `blob-chunk` (bytes,
default 50 MiB) and parallel layer uploads with `req-concurrent`. Signing is
keyless via GitHub OIDC — no signing key/secret to manage; verify with the
workflow identity (`--certificate-identity-regexp` + `--certificate-oidc-issuer
https://token.actions.githubusercontent.com`). The pushed manifest `digest` and
full `image` reference are exposed as workflow outputs.

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
    uses: OneLiteFeatherNET/workflows/.github/workflows/markdown-lint.yml@v2
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

`docker-publish` pushes to the Harbor registry, so it expects:

- `HARBOR_REGISTRY` — registry host (no scheme), e.g. `harbor.onelitefeather.dev`
- `HARBOR_USERNAME`
- `HARBOR_PASSWORD`

Signing is keyless (cosign + GitHub OIDC) — no signing secrets. The calling job
just needs `permissions: id-token: write` when `sign: true` (the default).

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
