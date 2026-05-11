# OneLiteFeatherNET Reusable Workflows

Central collection of reusable GitHub Actions workflows shared across
OneLiteFeatherNET repositories (Butterfly, Aonyx-bom, ...).

Each workflow is exposed via `workflow_call` and consumed from downstream
repositories by referencing a tagged release of this repo.

## Available workflows

| Workflow | Purpose |
| --- | --- |
| `.github/workflows/gradle-build-pr.yml` | Build & test a Gradle project on pull requests (matrix OS, configurable JDK). |
| `.github/workflows/gradle-publish.yml` | Build & publish a Gradle project to the OneLiteFeather Maven repository on tag pushes. |
| `.github/workflows/release-please.yml` | Run [release-please](https://github.com/googleapis/release-please) for a repository. |
| `.github/workflows/close-invalid-prs.yml` | Close PRs opened from a fork's default branch with a configurable message. |

## Versioning

This repository is released via [release-please](https://github.com/googleapis/release-please).
Pin consumers to a tag (e.g. `@v1.0.0`) or a major (e.g. `@v1`) rather than
`main` for reproducible builds.

## Usage examples

### Build a PR (Gradle)

```yaml
name: Build PR
on: [pull_request]

jobs:
  build:
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-build-pr.yml@v1
    with:
      java-version: "25"
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
    uses: OneLiteFeatherNET/workflows/.github/workflows/gradle-publish.yml@v1
    with:
      java-version: "21"
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
    uses: OneLiteFeatherNET/workflows/.github/workflows/release-please.yml@v1
```

### Close invalid PRs

```yaml
name: Close invalid PRs
on:
  pull_request_target:
    types: [opened]

jobs:
  close:
    uses: OneLiteFeatherNET/workflows/.github/workflows/close-invalid-prs.yml@v1
    with:
      protected-branch: main
```

## Required secrets

Workflows that publish or read from the OneLiteFeather Maven repository expect
these secrets to be available in the caller repository (and forwarded via
`secrets: inherit`):

- `ONELITEFEATHER_MAVEN_USERNAME`
- `ONELITEFEATHER_MAVEN_PASSWORD`

## Contributing

- Conventional Commits are required (`feat:`, `fix:`, `chore:`, ...).
- Releases are produced automatically by release-please on merge to `main`.

## License

MIT — see [LICENSE](LICENSE).
