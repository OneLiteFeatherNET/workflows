# Changelog

## [2.9.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.8.2...v2.9.0) (2026-09-11)


### Features

* **release-please:** let callers open the release PR as a GitHub App ([#35](https://github.com/OneLiteFeatherNET/workflows/issues/35)) ([70e7c0f](https://github.com/OneLiteFeatherNET/workflows/commit/70e7c0ff4a80ec62b65773a109fcb89c6dba2b48))

## [2.8.2](https://github.com/OneLiteFeatherNET/workflows/compare/v2.8.1...v2.8.2) (2026-09-06)


### Bug Fixes

* **pr-lint:** stop setup-node reaching for pnpm it never installed ([#32](https://github.com/OneLiteFeatherNET/workflows/issues/32)) ([60a1533](https://github.com/OneLiteFeatherNET/workflows/commit/60a1533602f188399e129800fb8981495c978f3c))

## [2.8.1](https://github.com/OneLiteFeatherNET/workflows/compare/v2.8.0...v2.8.1) (2026-08-15)


### Bug Fixes

* **docker-publish:** scope the concurrency group to the image ([#29](https://github.com/OneLiteFeatherNET/workflows/issues/29)) ([372e8da](https://github.com/OneLiteFeatherNET/workflows/commit/372e8da2086a961ba542e1624e513e8ba717e05e))

## [2.8.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.7.0...v2.8.0) (2026-08-10)


### Features

* **resourcepack:** publish SHA-1 and a JSON manifest alongside SHA256 ([#27](https://github.com/OneLiteFeatherNET/workflows/issues/27)) ([01215ee](https://github.com/OneLiteFeatherNET/workflows/commit/01215eeaa1c061b09e2f5991a627681a4cea8b66))

## [2.7.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.6.0...v2.7.0) (2026-08-10)


### Features

* add reusable resource pack publish workflow ([#25](https://github.com/OneLiteFeatherNET/workflows/issues/25)) ([03e0355](https://github.com/OneLiteFeatherNET/workflows/commit/03e035560bb2aecdfd48d44aefc6ba58965815db))

## [2.6.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.5.0...v2.6.0) (2026-08-08)


### Features

* add reusable SBOM publish and Trivy security scan workflows ([#23](https://github.com/OneLiteFeatherNET/workflows/issues/23)) ([e5c73b3](https://github.com/OneLiteFeatherNET/workflows/commit/e5c73b3d0944ffd403de934bbfff36bcbb160798))

## [2.5.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.4.0...v2.5.0) (2026-08-04)


### Features

* expose release-please outputs to calling workflows ([#21](https://github.com/OneLiteFeatherNET/workflows/issues/21)) ([339e8a1](https://github.com/OneLiteFeatherNET/workflows/commit/339e8a121935c89fbb796d537482d15c46215ef1))

## [2.4.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.3.0...v2.4.0) (2026-07-08)


### Features

* **pr-lint:** add reusable Conventional Commits PR/commit lint workflow ([#18](https://github.com/OneLiteFeatherNET/workflows/issues/18)) ([66f4ba4](https://github.com/OneLiteFeatherNET/workflows/commit/66f4ba45e63e1f14b00eb741b82336abe45b0986))

## [2.3.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.2.0...v2.3.0) (2026-06-21)


### Features

* **docker-publish:** pure Docker build + gradle-docker-context producer ([#15](https://github.com/OneLiteFeatherNET/workflows/issues/15)) ([7c9f397](https://github.com/OneLiteFeatherNET/workflows/commit/7c9f3975bdf64b1ba0d32e6232f6c04c72d84137))

## [2.2.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.1.0...v2.2.0) (2026-06-19)


### Features

* **docker-publish:** reusable chunked docker publish workflow ([#13](https://github.com/OneLiteFeatherNET/workflows/issues/13)) ([887181d](https://github.com/OneLiteFeatherNET/workflows/commit/887181df56e19ef51a8b675a12bc695bf3ad9f20))

## [2.1.0](https://github.com/OneLiteFeatherNET/workflows/compare/v2.0.1...v2.1.0) (2026-05-13)


### Features

* **gradle-build-pr:** add run-tests flag for BOM and test-less projects ([#10](https://github.com/OneLiteFeatherNET/workflows/issues/10)) ([3508e38](https://github.com/OneLiteFeatherNET/workflows/commit/3508e382105bc7e5dcf551cf1fe57b684358e816))

## [2.0.1](https://github.com/OneLiteFeatherNET/workflows/compare/v2.0.0...v2.0.1) (2026-05-11)


### Bug Fixes

* **gradle-build-pr:** deduplicate junit_files glob to avoid double-counted results ([#7](https://github.com/OneLiteFeatherNET/workflows/issues/7)) ([1f2b60d](https://github.com/OneLiteFeatherNET/workflows/commit/1f2b60d62bf35dd84bb129285c27089deb72fb7d))

## [2.0.0](https://github.com/OneLiteFeatherNET/workflows/compare/v1.0.0...v2.0.0) (2026-05-11)


### ⚠ BREAKING CHANGES

* default `java-version` changed from `21` to `25`, default `runs-on` changed from triple-OS matrix to `["ubuntu-latest"]`, default `gradle-task` changed from `clean build test` to `build test`.

### Features

* path-based change detection, Java 25 default, speed & debug tu… ([#3](https://github.com/OneLiteFeatherNET/workflows/issues/3)) ([5400a36](https://github.com/OneLiteFeatherNET/workflows/commit/5400a36a6ad341831c22220cc7bda680df59edef))

## 1.0.0 (2026-05-11)


### Features

* initial reusable workflows, docs and release-please setup ([e5a9178](https://github.com/OneLiteFeatherNET/workflows/commit/e5a917855e3387f170bdba1ede1185ea6cd65333))

## Changelog

All notable changes to this project will be documented in this file. The format
is maintained by [release-please](https://github.com/googleapis/release-please).
