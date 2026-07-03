# Cross-Language Testing Strategy

Researched: 2026-07-03

## Goal

Before every pull request, every supported language package must run the same shared conformance tests plus its own native unit, lint, type, build, and packaging checks.

The definitive local command should be:

```sh
./tools/test/pr
```

Until this wrapper exists, run the shared checks and the package-specific commands listed below.

## Core Rule

Correctness must come from shared fixtures, not from each language inventing its own expectations.

Every language must use the same:

- Fixture IDs.
- Expected values.
- Error categories.
- Numeric tolerances.
- Parsing and serialization rules.
- Feature gates.
- Conformance report format.

## Correctness Oracle Strategy

Use layered testing instead of relying on one proof technique.

For a conversion such as RGB to HSL, the normal standard should be:

1. Hand-written reference vectors for canonical and edge-case colors.
2. Differential tests against at least one established library.
3. Property tests over generated inputs.
4. Round-trip tests where the inverse conversion exists.
5. Optional exhaustive tests for bounded integer spaces.

Do not treat another library as absolute truth. Treat it as an oracle that can catch implementation mistakes, then combine it with formulas from the relevant specification, explicit fixtures, and mathematical invariants.

### Reference Vectors

Each conversion should include a small fixed set of obvious and edge-case values:

| RGB | Expected HSL case |
| --- | --- |
| `rgb(255, 0, 0)` | Red primary, hue `0`, full saturation, middle lightness. |
| `rgb(0, 255, 0)` | Green primary, hue `120`, full saturation, middle lightness. |
| `rgb(0, 0, 255)` | Blue primary, hue `240`, full saturation, middle lightness. |
| `rgb(0, 0, 0)` | Black, zero saturation, zero lightness, powerless hue behavior. |
| `rgb(255, 255, 255)` | White, zero saturation, full lightness, powerless hue behavior. |
| `rgb(128, 128, 128)` | Gray, zero saturation, midpoint lightness, powerless hue behavior. |
| `rgb(12, 34, 56)` | Uneven channel order and non-round values. |
| `rgb(250, 128, 114)` | Named-color style sample with non-primary hue. |

These examples should live in shared fixtures so every language package checks the same expected behavior.

### Differential Tests

The TypeScript implementation should compare generated samples against established color libraries such as `color`, `culori`, or `colorjs.io`.

For example, a TypeScript conformance test can:

1. Generate or load RGB samples.
2. Convert each sample with this library.
3. Convert the same sample with a pinned reference-library version.
4. Compare numeric channels with the named tolerance from `spec/v1/tolerances.yaml`.
5. Report failures using the shared fixture ID or generated sample seed.

Pin the reference dependency version. A reference library upgrade should be reviewed as a test-behavior change, not accepted silently.

### Generated Sample Coverage

Normal CI does not need every possible RGB value. It should run a deterministic sample set that covers:

- Channel minimums and maximums.
- Equal-channel grayscale values.
- Near-equal channels.
- Primary and secondary colors.
- Values around branch boundaries in the formula.
- Random samples from a fixed seed.
- Known historical bug cases.

A PR-sized budget of `1_000` to `10_000` generated RGB samples is usually enough to catch implementation mistakes quickly while keeping CI fast.

### Exhaustive RGB Checks

For 8-bit RGB, exhaustive testing means:

```text
256 * 256 * 256 = 16,777,216 colors
```

This is feasible as a local, nightly, or release-gate check for simple conversions such as RGB to HSL, but it should not be the default test path for every pull request.

Exhaustive testing becomes impractical for:

- Float coordinate spaces.
- Alpha channels with many representations.
- Wide-gamut spaces.
- Multi-step conversion graphs.
- Parsing and serialization inputs.
- Cross-language package matrices.

Use exhaustive tests only where the input domain is small, discrete, and performance is acceptable.

## Required PR Gate

`./tools/test/pr` should run:

1. Validate shared spec files.
2. Regenerate constants in check mode, if generated constants exist.
3. Regenerate fixtures in check mode, if fixture generators exist.
4. Run the TypeScript reference package tests.
5. Run every language package against shared fixtures.
6. Run package-native unit tests.
7. Run property and fuzz tests with a PR-sized budget.
8. Run format, lint, type, and static-analysis checks.
9. Build each package.
10. Run package install/import smoke tests.
11. Execute documentation examples.
12. Produce `reports/conformance-summary.json`.

A pull request should not merge unless this gate passes locally and in CI.

## Shared Test Categories

Every language must support these common test categories:

| Category | Required coverage |
| --- | --- |
| Spec validation | Color spaces, transfer functions, matrices, white points, channels, errors, tolerances. |
| Generated output | Constants and fixtures reproduce without diffs. |
| Parsing | Valid CSS, structured values, arrays, objects, and design-token values. |
| Invalid parsing | Invalid inputs fail with the exact shared error category. |
| Conversion | Coordinates match shared fixtures within named tolerances. |
| Round trip | Reversible routes return within documented tolerance. |
| Serialization | Strings and structured outputs match exactly. |
| Precision | Rounding, quantization, negative zero, and decimal precision. |
| Alpha | Preserve, reject, clamp, format, and compose alpha consistently. |
| Missing values | CSS `none`, powerless hue, and undefined hue behavior. |
| Gamut | In-gamut checks, clipping, and mapping behavior. |
| Interpolation | Space, hue method, alpha handling, and gamut timing. |
| Contrast | Relative luminance and WCAG contrast. |
| Delta E | CIE76, CIE94, CIEDE2000, and CMC where supported. |
| Profiles | ICC/profile behavior only for packages that claim support. |
| Properties | Parse-format-parse, round-trip, alpha preservation, and error stability. |
| Packaging | Build, install, import, and run a minimal example. |
| Docs | README and guide examples execute successfully. |

## Fixture Contract

Fixtures should live under:

```text
fixtures/v1/
  manifest.json
  parse/
  invalid/
  convert/
  roundtrip/
  serialize/
  gamut/
  interpolate/
  contrast/
  delta-e/
  profiles/
```

Each fixture must include:

- `id`
- `operation`
- `feature`
- `input`
- `expected`
- `tolerance`, when numeric
- `source`
- `required`

Skipped fixtures are allowed only when the fixture is feature-gated and the package does not claim that feature.

## Language Adapter

Every package should expose the same adapter command:

```sh
./tools/test/package <language> --fixtures fixtures/v1/manifest.json --report reports/<language>.json
```

The adapter must:

1. Load the shared fixture manifest.
2. Run all required fixtures for claimed features.
3. Compare strings and errors exactly.
4. Compare numbers using `spec/v1/tolerances.yaml`.
5. Preserve fixture IDs in failures.
6. Emit a JSON report even when tests fail.

## Package Commands

| Language | Required checks |
| --- | --- |
| JavaScript / TypeScript | `npm ci`, `npm run format:check`, `npm run lint`, `npm run typecheck`, `npm test`, `npm run test:conformance`, `npm run build`, `npm pack --dry-run` |
| Python | `python -m pip install -e ".[dev]"`, `python -m ruff format --check .`, `python -m ruff check .`, `python -m mypy .`, `python -m pytest`, `python -m pytest tests/conformance`, `python -m build` |
| Rust | `cargo fetch`, `cargo fmt --check`, `cargo clippy --all-targets --all-features -- -D warnings`, `cargo test --all-features`, `cargo test --test conformance --all-features`, `cargo package --allow-dirty` |
| Go | `go mod download`, `gofmt -l .`, `go vet ./...`, `go test ./...`, `go test ./... -run Conformance`, `go list ./...` |
| .NET / C# | `dotnet restore`, `dotnet format --verify-no-changes`, `dotnet build -warnaserror`, `dotnet test`, `dotnet test --filter Conformance`, `dotnet pack --no-build` |
| JVM / Kotlin | `./gradlew dependencies`, `./gradlew spotlessCheck`, `./gradlew test`, `./gradlew conformanceTest`, `./gradlew check`, `./gradlew assemble` |
| Swift | `swift package resolve`, `swift format lint --recursive .`, `swift build`, `swift test`, `swift test --filter Conformance` |
| Ruby | `bundle install`, `bundle exec rubocop`, `bundle exec rake test`, `bundle exec rake test:conformance`, `gem build` |
| PHP | `composer install`, `composer format-check`, `composer lint`, `composer analyse`, `composer test`, `composer test:conformance`, `composer validate` |
| Dart | `dart pub get`, `dart format --set-exit-if-changed .`, `dart analyze`, `dart test`, `dart test test/conformance`, `dart pub publish --dry-run` |
| C / C++ | `cmake --preset dev`, `cmake --build --preset dev`, `ctest --preset dev`, `ctest --preset conformance`, `clang-format` check |

## PR Checklist

Before opening a pull request:

1. Run `./tools/test/pr`.
2. Confirm generated files have no unexpected diffs, if generated files exist.
3. Review `reports/conformance-summary.json`.
4. Confirm every claimed feature passes in every language.
5. Confirm skipped tests are only unsupported feature-gated fixtures.
6. Add or update shared fixtures for behavior changes.
7. Do not change fixtures only to make one package pass.

## Release Rule

A language package is releasable only when it passes all shared fixtures for every feature it claims to support, its native checks pass, its package smoke test works, and its documentation examples run successfully.
