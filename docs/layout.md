# Project Layout

Color should start as a small conformance-first monorepo. The project needs a shared behavior contract before it needs many packages, but it should not create empty future directories or a shared native runtime before those choices are justified.

The first implementation should be TypeScript for npm users. Future language packages should be native ports that prove compatibility against the same spec and fixtures.

## Layout Principles

- Keep the spec and fixtures as the source of truth for behavior.
- Build one package first, then add languages only after the shared contract is useful.
- Use feature gates for unsupported future work instead of partially implemented APIs.
- Keep generated output out of the repository until it is reproducible and shared by more than one consumer.
- Do not add empty package, fixture, or tool directories just to match a future plan.
- Treat CSS Color Module Level 4 as the first standards anchor. Keep CSS Color 5, ICC, HDR, video, advanced appearance models, and proprietary color catalogs explicitly feature-gated.

`spec/v1` and `fixtures/v1` are contract versions, not package versions. The TypeScript package can ship `0.x` releases while conforming to `spec/v1` fixture subsets.

## Target Repository Shape

```text
color/
  README.md
  LICENSE

  docs/
    layout.md
    roadmap.md
    research/
      current_landscape.md
      cross_language_architecture.md
      language_support.md
      testing.md

  spec/
    v1/
      README.md
      features.yaml
      errors.yaml
      tolerances.yaml
      css-color.md
      serialization.md
      named-colors.yaml
      color-spaces/
        README.md
        registry.yaml
        srgb.yaml
        srgb-linear.yaml
        hsl.yaml
        hwb.yaml
        xyz-d65.yaml
        xyz-d50.yaml
        lab.yaml
        lch.yaml
        oklab.yaml
        oklch.yaml
      transforms/
        transfer-functions.yaml
        chromatic-adaptation.yaml

  fixtures/
    v1/
      README.md
      manifest.json
      parse/
      invalid/
      convert/
      roundtrip/
      serialize/
      contrast/

  tools/
    README.md
    spec/
    fixtures/
    test/
      pr
      package

  packages/
    typescript/
      README.md
      package.json
      tsconfig.json
      vitest.config.ts
      src/
      test/
      examples/

  reports/
```

`reports/` should be gitignored output for local and CI conformance reports. Add a placeholder only if tooling requires the directory to exist.

Add these directories later, when they have real content:

```text
generated/
  v1/
    constants/
    fixture-index/
    schemas/

fixtures/
  v1/
    gamut/
    interpolate/
    delta-e/
    profiles/
```

## Directory Responsibilities

### `docs/`

Human-facing planning, research, and architecture notes. These documents explain product direction and tradeoffs, but package builds and conformance tests should consume `spec/` and `fixtures/`, not prose from `docs/`.

### `spec/v1/`

The shared behavior contract for official packages.

- `README.md`: overview of the contract, standards anchors, oracle strategy, feature gates, and conformance expectations.
- `features.yaml`: stable feature IDs and whether each feature is MVP, planned, experimental, or unsupported.
- `errors.yaml`: shared public error categories.
- `tolerances.yaml`: named numeric tolerances by operation and color space.
- `css-color.md`: CSS parsing, missing-component, relative syntax, serialization, and standards decisions that need prose.
- `serialization.md`: output modes, rounding, precision, lossy-output, and gamut-policy behavior.
- `named-colors.yaml`: CSS named colors and aliases from the standards-backed data source.
- `color-spaces/`: one machine-readable YAML file per supported space or coordinate model.
- `transforms/`: shared transfer-function and chromatic-adaptation definitions.

Keep numeric constants in machine-readable files. Markdown can cite standards, explain choices, and show small examples, but it should not become a second source of truth.

### `spec/v1/color-spaces/`

Each color-space YAML file should define:

- stable ID and aliases
- family, status, and feature gate
- channel names, units, ranges, bounds, and missing-value rules
- white point and observer where applicable
- transfer function or linear-light status
- connection space and transform metadata
- CSS syntax support where applicable
- fixture coverage expectations
- source references

Use `registry.yaml` to list official IDs, aliases, status, and file paths. Do not put every constant in the registry.

The MVP registry should include sRGB, linear sRGB, HSL, HWB, XYZ D50, XYZ D65, Lab, LCh, OKLab, and OKLCh. Display P3, A98 RGB, ProPhoto RGB, Rec. 2020, CMYK, Delta E spaces, ICC-backed spaces, HDR spaces, and video spaces should stay feature-gated until the core package is solid.

### `fixtures/v1/`

The shared conformance corpus. Every official package must run the fixtures for the features it claims to support.

- `manifest.json`: fixture files, fixture groups, feature gates, required status, and report metadata.
- `parse/`: valid CSS strings and supported structured inputs.
- `invalid/`: rejected inputs and exact shared error categories.
- `convert/`: known-value conversions.
- `roundtrip/`: reversible-route expectations and tolerances.
- `serialize/`: exact CSS, hex, object, and array outputs.
- `contrast/`: relative luminance and WCAG contrast expectations.

Future fixture groups such as `gamut/`, `interpolate/`, `delta-e/`, and `profiles/` should be added only when those features are implemented or actively specified.

Each fixture should include:

- `id`
- `feature`
- `operation`
- `input`
- `expected`
- `tolerance`, when numeric
- `source`
- `required`

Skipped fixtures should be allowed only through explicit feature gates.

### `tools/`

Language-neutral automation.

- `tools/spec/`: validate `spec/v1`.
- `tools/fixtures/`: build and validate fixture files.
- `tools/test/package`: run one package against `fixtures/v1/manifest.json`.
- `tools/test/pr`: eventual one-command local PR gate.

The target local gate should be:

```sh
./tools/test/pr
```

Until it exists, run the TypeScript package commands and any available spec or fixture validators directly.

### `packages/typescript/`

The first package and first reference implementation.

```text
packages/typescript/
  README.md
  package.json
  tsconfig.json
  vitest.config.ts

  src/
    index.ts
    color.ts
    parse/
    format/
    convert/
    spaces/
    contrast/
    errors/
    internal/

  test/
    unit/
    conformance/

  examples/
```

The TypeScript package should:

- expose a small public API around `Color`, `ColorSpace`, `parse`, `format`, `convert`, `equals`, and `contrast`
- consume shared spec data rather than hand-copying tables into unrelated modules
- run native unit tests for formulas, parser behavior, and API behavior
- run conformance tests against `fixtures/v1/manifest.json`
- publish JavaScript output and TypeScript declarations for npm users

Keep the implementation idiomatic to TypeScript, but do not let parsing, conversion, serialization, error categories, or tolerances diverge from the shared contract.

## Initial MVP Scope

The first TypeScript release should target a small, useful CSS and perceptual-color core:

- Parse: CSS named colors, hex, `rgb()`, `rgba()`, `hsl()`, `hsla()`, `hwb()`, `lab()`, `lch()`, `oklab()`, and `oklch()`.
- Convert: sRGB, linear sRGB, HSL, HWB, XYZ D50/D65, Lab, LCh, OKLab, and OKLCh.
- Format: hex, legacy CSS where representable, modern CSS, JSON-compatible objects, and numeric arrays.
- Analyze: relative luminance, WCAG contrast ratio, and tolerance-based equality.
- Errors: invalid syntax, unsupported color space, invalid channel, missing component, out-of-gamut-for-output, and lossy serialization.

Out-of-gamut coordinates should be representable internally. Bounded outputs such as hex or legacy `rgb()` should require an explicit clipping or gamut-mapping decision.

## Future Package Contract

Future packages should follow this convention when they become real work:

```text
packages/
  typescript/
  python/
  rust/
  go/
  dotnet/
  jvm/
  swift/
  ruby/
  php/
  dart/
```

Do not add these directories until implementation starts.

Every future package should provide:

- a native public API for its ecosystem
- a fixture adapter that can run `fixtures/v1/manifest.json`
- exact string and error comparisons
- tolerance-based numeric comparisons using `spec/v1/tolerances.yaml`
- a JSON conformance report under `reports/<language>.json`
- package-native unit, lint, build, and packaging checks

The API can vary by language, but behavior must not silently diverge from the shared conformance contract.

## Build Order

1. Write `spec/v1/features.yaml`, `errors.yaml`, `tolerances.yaml`, `css-color.md`, and `serialization.md`.
2. Add the MVP color-space and transform YAML files.
3. Add `fixtures/v1/manifest.json` and initial parse, invalid, convert, roundtrip, serialize, and contrast fixtures.
4. Add TypeScript package scaffolding under `packages/typescript`.
5. Add minimal spec and fixture validation tools.
6. Implement TypeScript against the shared fixtures.
7. Add `tools/test/package typescript`.
8. Add `tools/test/pr` once the shared and package-specific checks are stable.

## CI Shape

The first PR gate should run:

1. validate shared spec files
2. validate fixture manifests and fixture schemas
3. run TypeScript format, lint, typecheck, unit tests, conformance tests, and build
4. run package install/import smoke tests
5. execute documentation examples
6. write `reports/conformance-summary.json`

As new packages are added, they should plug into the same fixture contract without changing the shared behavior model.
