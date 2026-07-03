# Cross-Language Architecture

Researched: 2026-07-03

## Recommendation

Build Color as a conformance-first family of native packages. The shared center of the project should be a versioned specification, shared fixtures, and common conformance tests. Generated constants can become part of that center once generators exist and their output is reproducible. Each language package should expose a native-feeling API while proving that it matches the same behavior, numerical tolerances, parsing rules, serialization rules, and failure modes.

Do not start with one shared runtime core for every language. That can look efficient early, but across ten or more language ecosystems it usually creates packaging, foreign-function-interface, platform, build, and API ergonomics problems. A shared Rust, C, or WebAssembly core can still be added later for specific packages where it provides clear value.

## Core Principle

The spec and fixtures should be the product's center of gravity, not any single implementation.

This keeps the project consistent across languages without forcing every ecosystem to depend on the same binary runtime. Python can feel Pythonic, Rust can expose idiomatic Rust types, Swift can fit Apple platform expectations, and JavaScript can work cleanly in browser and server environments, while all packages still pass the same conformance suite.

## Suggested Repository Structure

```text
/color
  /spec
    /v1
      README.md
      features.yaml
      errors.yaml
      tolerances.yaml
      css-color.md
      serialization.md
      named-colors.yaml
      /color-spaces
        registry.yaml
      /transforms
        transfer-functions.yaml
        chromatic-adaptation.yaml

  /fixtures
    /v1
      manifest.json
      parse/
      invalid/
      convert/
      roundtrip/
      serialize/
      contrast/

  /tools
    spec/
    fixtures/
    test/
      pr
      package

  /packages
    typescript/
```

## Shared Specification

Keep behavior that must match across languages in machine-readable files wherever practical:

- Color spaces, channel names, channel ranges, white points, primaries, conversion matrices, and transfer functions.
- Chromatic adaptation methods and the exact constants used by each method.
- CSS named colors and aliases.
- Parsing grammar decisions that affect accepted and rejected inputs.
- Serialization rules, rounding rules, clamping rules, and missing-component behavior.
- Error categories for invalid syntax, unsupported spaces, out-of-gamut handling, unknown profiles, and lossy operations.
- Numerical tolerances per operation and color space.

Use Markdown for explanation and YAML, JSON, or TOML for data that must be consumed by package builds and test harnesses.

## Reference Behavior

Use the first TypeScript package as the initial reference implementation. Its purpose is correctness, clarity, and fixture verification, not maximum speed.

The reference implementation path should:

- Encode the canonical algorithms from the project spec.
- Verify expected conversion, parsing, formatting, round-trip, serialization, and contrast fixtures.
- Make rounding and tolerance choices explicit.
- Act as a debugging oracle when language ports disagree.

If fixture generation grows beyond simple checked-in data, put language-neutral generators under `tools/fixtures/`. Do not create a separate `/reference` package unless it solves a real maintenance problem.

## Shared Fixtures

All language packages should run the same conformance fixtures. A typical conversion fixture can look like this:

```json
{
  "id": "srgb-to-oklab-red",
  "operation": "convert",
  "input": {
    "space": "srgb",
    "coords": [1, 0, 0],
    "alpha": 1
  },
  "output": {
    "space": "oklab",
    "coords": [0.627955, 0.224863, 0.125846],
    "alpha": 1
  },
  "tolerance": 0.000001,
  "source": "css-color-4"
}
```

Fixture groups should include:

- Valid CSS and structured parsing inputs.
- Invalid inputs and expected error categories.
- Direct conversions between canonical spaces.
- Round-trip conversions with defined tolerances.
- Serialization output for CSS, hex, arrays, objects, and design-token-friendly values.
- Gamut checks, clipping, and mapping behavior once gamut features are in scope.
- Interpolation and mixing behavior once interpolation features are in scope.
- Contrast and relative luminance behavior.
- Edge cases such as negative components, values above one, powerless hues, missing components, alpha handling, and out-of-gamut colors.

## Generated Constants

Generate repeated constants instead of hand-copying them into every package:

- Named color tables.
- Color space definitions.
- Matrix constants.
- Transfer function metadata.
- Channel metadata.
- Error identifiers.
- Fixture manifests.
- Documentation tables.

Generated files should be checked for reproducibility in CI. If generated outputs are committed, CI should verify that running the generator does not change them.

## Native Package Strategy

Implement package APIs natively for each ecosystem, but keep semantics shared.

Recommended first-class package targets:

1. JavaScript / TypeScript
2. Python
3. Rust
4. Go
5. .NET / C#
6. JVM: Java + Kotlin
7. Swift
8. Ruby
9. PHP
10. Dart
11. C / C++ later if native graphics, bindings, or shared-core use cases justify it

Each package should:

- Consume shared spec data and generated constants where generated constants exist.
- Run the shared fixtures.
- Provide idiomatic types, errors, and package conventions for its ecosystem.
- Document any intentional API differences while preserving semantic equivalence.
- Avoid silently diverging in parsing, rounding, gamut handling, or serialization.

## Continuous Integration

Every pull request should run a conformance matrix:

- Validate the shared spec files.
- Regenerate constants and fixtures when generators exist.
- Run the TypeScript reference package tests.
- Run every implemented language package against the shared fixture suite.
- Report conformance coverage by package and feature area.

A package should not be considered releasable until it passes the required shared fixtures for the feature set it claims to support.

## Standards Anchors

Anchor the first stable project version around CSS Color Module Level 4. As of this research note, the W3C page identifies CSS Color Module Level 4 as a Candidate Recommendation Draft dated 18 June 2026. It covers modern CSS color syntax, Lab and LCH, Oklab and OkLCh, predefined spaces such as sRGB, Display P3, A98 RGB, ProPhoto RGB, Rec.2020, XYZ D50 and D65, interpolation, gamut mapping, serialization, and sample conversion code.

Treat CSS Color Module Level 5 as experimental until the relevant behavior stabilizes. The Level 5 Editor's Draft dated 18 June 2026 adds features such as `color-mix()`, relative colors, custom color spaces, `device-cmyk()`, `light-dark()`, and `contrast-color()`, with some features marked at risk.

Also track Web Platform Tests for CSS color behavior. WPT should inform parsing and browser-facing behavior, but the project should still maintain its own language-neutral fixtures because this library covers more than browser rendering.

## Suggested README Positioning

The project can describe its architecture like this:

> Color uses a shared, versioned conformance specification. Each language package is implemented natively, but all packages must pass the same fixture suite before release.

## Sources

- [CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/)
- [CSS Color Module Level 5 Editor's Draft](https://drafts.csswg.org/css-color-5/)
- [Web Platform Tests CSS color results](https://wpt.fyi/results/css/css-color)
