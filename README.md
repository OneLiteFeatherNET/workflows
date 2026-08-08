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
| `.github/workflows/sbom-publish.yml` | Publish a CycloneDX SBOM to the OneLiteFeather [Dependency-Track](https://dependencytrack.org/) instance, so the shipped dependency inventory keeps being matched against CVEs published later. Takes the project's own SBOM via an artifact, or generates one with [Trivy](https://trivy.dev/) when the project has none. |
| `.github/workflows/security-scan.yml` | Scan a filesystem or container image with [Trivy](https://trivy.dev/) and surface the findings in GitHub code scanning. Report-only by default, optionally gating. |

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
  release-please:
    uses: OneLiteFeatherNET/workflows/.github/workflows/release-please.yml@v2.5.0
```

The workflow forwards the action's outputs, so a follow-up job can be gated on whether a
release was actually cut:

```yaml
jobs:
  release-please:
    uses: OneLiteFeatherNET/workflows/.github/workflows/release-please.yml@v2.5.0

  publish:
    needs: release-please
    if: needs.release-please.outputs.release_created == 'true'
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-publish.yml@v2.5.0
    secrets: inherit
```

| Output | Description |
|---|---|
| `release_created` | `'true'` when the root package was released. |
| `releases_created` | `'true'` when at least one release was created - use this on a multi-package manifest. |
| `tag_name` | Tag of the root package's release, e.g. `v1.2.3`. Empty when it was not released. |
| `version` | Version of the root package's release, e.g. `1.2.3`. Empty when it was not released. |
| `sha` | Commit the root package's release was cut from. |
| `paths_released` | JSON array of released package paths. |
| `prs` | JSON array of the release pull requests opened or updated. |

On a multi-package manifest the action exposes per-package values as `<path>--release_created`
and friends. Those names are not fixed, so a reusable workflow cannot declare them - gate on
`releases_created` and read `paths_released` instead.

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

### Publish an SBOM to Dependency-Track

Two shapes, depending on whether the project already generates an SBOM.

**The project generates its own** (preferred — a build tool resolves the
dependency graph better than any external scanner). Upload it as an artifact,
then hand the artifact name over:

```yaml
jobs:
  publish:
    # ... your existing build/publish job, ending with:
    #   - uses: actions/upload-artifact@v4
    #     with:
    #       name: sbom
    #       path: build/reports/cyclonedx/bom.xml

  sbom:
    needs: publish
    uses: OneLiteFeatherNET/workflows/.github/workflows/sbom-publish.yml@v2.6.0
    with:
      project-name: "MyProject"
      project-version: "1.2.3"
      artifact-name: "sbom"
      sbom-path: "bom.xml"
    secrets: inherit
```

**The project generates nothing** — leave `artifact-name` empty and Trivy
produces a CycloneDX SBOM from the checked-out repository:

```yaml
jobs:
  sbom:
    uses: OneLiteFeatherNET/workflows/.github/workflows/sbom-publish.yml@v2.6.0
    with:
      project-name: "MyProject"
      project-version: "1.2.3"
    secrets: inherit
```

Run it as its own job, not as a step inside the publish job: Dependency-Track
being unreachable should never take down the release that produced the
artifact.

`autocreate` defaults to `true`, which needs the API key's team to hold
**`PROJECT_CREATION_UPLOAD`** on top of `BOM_UPLOAD`. Without it the server
answers `403` the first time any new version is uploaded.

### Scan for vulnerabilities (Trivy)

```yaml
name: Security
on:
  pull_request:
  schedule:
    - cron: '0 6 * * 1'   # new CVEs land against unchanged code

jobs:
  scan:
    permissions:
      contents: read
      security-events: write   # SARIF upload to code scanning
    uses: OneLiteFeatherNET/workflows/.github/workflows/security-scan.yml@v2.6.0
```

Report-only by default: adopting it makes findings visible in code scanning
without turning a repository's CI red on day one. Set `fail-on-findings: true`
once a repo is clean enough to keep it that way.

On **private** repositories the SARIF upload needs GitHub Advanced Security.
Without it, set `upload-sarif: false` and rely on the job summary plus
`fail-on-findings`.

#### Gate a release before anything goes public

To stop a vulnerable build from ever reaching a registry, put the scan in its
own job between the build and everything that publishes. Have the build upload
the artifact, gate on it, and let the publishing job depend on the gate:

```yaml
jobs:
  build:            # produces the artifact, publishes nothing
    # - uses: actions/upload-artifact@v4
    #   with: { name: plugin-jar, path: build/libs/*.jar }

  security-gate:
    needs: build
    permissions:
      contents: read
      security-events: write
    uses: OneLiteFeatherNET/workflows/.github/workflows/security-scan.yml@v2.6.0
    with:
      scan-type: rootfs        # see the warning below
      artifact-name: plugin-jar
      fail-on-findings: true
    secrets: inherit

  publish:
    needs: security-gate       # nothing public happens until the gate is green
    # ...
```

> **Use `rootfs`, not `fs`, for built JVM artifacts.** Trivy's `fs` scanner
> ignores JAR contents. Measured on AntiRedstoneClock-Remastered's shaded jar:
> `fs` reported 0 packages and 0 findings, `rootfs` reported 12 packages and a
> HIGH finding. `fs` on the source tree of a Gradle project without a
> `gradle.lockfile` also finds nothing — a gate on it looks green because it
> checked nothing at all.

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

`sbom-publish` talks to Dependency-Track, so it expects:

- `DEPENDENCYTRACK_HOSTNAME` — host only, no scheme, e.g. `dependency-track.onelitefeather.dev`
- `DEPENDENCYTRACK_APIKEY` — the key's team needs `BOM_UPLOAD`, plus `PROJECT_CREATION_UPLOAD` while `autocreate` is on

`security-scan` needs no secrets at all.

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
