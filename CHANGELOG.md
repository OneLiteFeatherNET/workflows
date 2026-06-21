# Changelog

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
