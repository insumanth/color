# TypeScript MVP Roadmap

Purpose: move Color from research and planning into a first usable TypeScript package with a shared behavior contract, shared fixtures, and a repeatable test gate.

This roadmap is only for the MVP and getting started. It is intentionally limited to the TypeScript package. Other language packages, native shared runtimes, ICC/profile modules, HDR/video modules, CLI work, and advanced design-system tooling should wait until the TypeScript MVP is stable.

## Roadmap Inputs

This plan is based on the current documentation under `docs/`:

- `docs/layout.md`
- `docs/research/current_landscape.md`
- `docs/research/cross_language_architecture.md`
- `docs/research/language_support.md`
- `docs/research/testing.md`

## MVP Definition

The MVP is complete when the repository contains:

- A versioned shared contract under `spec/v1`.
- A versioned fixture corpus under `fixtures/v1`.
- A TypeScript package under `packages/typescript`.
- A TypeScript implementation that passes the shared fixtures for the MVP feature set.
- A package-local test, lint, typecheck, build, and npm packaging workflow.
- A first local PR gate that validates spec files, fixtures, and the TypeScript package.
- Getting-started documentation for early npm users.

The MVP does not need to prove every future package strategy. It needs to prove that the project can define behavior once and implement it correctly once in TypeScript.

## MVP Scope

### Language Scope

In scope:

- TypeScript package for npm users.
- JavaScript runtime output from the TypeScript package.
- TypeScript declaration output.
- Browser-compatible and Node-compatible package shape, if this can be done without delaying the MVP.

Out of scope until after MVP:

- Python, Rust, Go, Ruby, Swift, JVM, .NET, PHP, Dart, C, or C++ packages.
- Foreign-function interfaces.
- Shared Rust, C, or WebAssembly runtime cores.
- Cross-language release automation beyond a fixture contract that future packages can consume.

### Feature Scope

In scope for the TypeScript MVP:

- Parse CSS named colors.
- Parse hex colors: `#rgb`, `#rgba`, `#rrggbb`, and `#rrggbbaa`.
- Parse `rgb()`, `rgba()`, `hsl()`, `hsla()`, `hwb()`, `lab()`, `lch()`, `oklab()`, and `oklch()`.
- Convert between sRGB, linear sRGB, HSL, HWB, XYZ D65, XYZ D50, Lab, LCh, OKLab, and OKLCh.
- Format hex where representable.
- Format legacy CSS where representable.
- Format modern CSS for supported MVP spaces.
- Format JSON-compatible objects.
- Format numeric arrays.
- Compute WCAG 2.x relative luminance.
- Compute WCAG 2.x contrast ratio.
- Compare colors using documented numeric tolerances.
- Preserve alpha consistently across parse, convert, and format operations.
- Preserve out-of-gamut coordinates internally where possible.
- Require explicit clipping or gamut decisions for bounded outputs such as hex and legacy `rgb()`.

Feature-gated or deferred:

- CSS `color()` predefined spaces.
- Display P3, A98 RGB, ProPhoto RGB, Rec. 2020, Rec. 709, ACES, Luv, LChuv, HSV, HSB, HSI, CMY, and CMYK.
- CSS Color 5 features such as `color-mix()`, relative color syntax, `contrast-color()`, `light-dark()`, custom color spaces, and `device-cmyk()`.
- ICC profiles and profile-backed CMYK.
- HDR, video, YCbCr, ICtCp, PQ, HLG, and Rec. 2100 behavior.
- Delta E metrics.
- Gamut mapping beyond explicit clipping and in-gamut checks needed for serialization.
- Interpolation, mixing, palette generation, and scale generation.
- APCA and other experimental perceptual contrast models.
- Proprietary color catalogs.
- Design-token batch conversion.
- CLI commands.

## MVP Repository Shape

The MVP repository should grow only the directories that contain real content:

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
      package
      pr

  packages/
    typescript/
      README.md
      package.json
      tsconfig.json
      vitest.config.ts
      src/
      test/
      examples/
```

Do not add placeholder directories for future language packages. Add `reports/` only as gitignored generated output when the test tooling writes conformance reports.

## Build Principles

- The shared spec and fixtures are the behavior source of truth.
- TypeScript is the first implementation and initial reference package.
- The TypeScript API can be idiomatic, but behavior must match `spec/v1` and `fixtures/v1`.
- Numeric constants should live in machine-readable spec files where practical.
- Markdown should explain behavior, tradeoffs, and standards decisions. It should not become a second source of numeric truth.
- Unsupported future work should be represented as feature-gated or explicitly unsupported behavior, not partially implemented APIs.
- Out-of-gamut values should be representable internally.
- Lossy output should be explicit.
- Error behavior should be part of the public contract.

## Milestone 0: Foundation Decisions

Goal: remove ambiguity before implementation starts.

Deliverables:

- Decide the npm package name or temporary internal package name.
- Decide the initial package version, likely `0.1.0`.
- Decide supported Node.js versions.
- Decide whether the package is ESM-only or dual ESM/CommonJS.
- Decide which package manager will be used in the repository.
- Decide formatting and linting tools.
- Decide whether generated constants will be committed during MVP or loaded directly from `spec/v1`.
- Decide the default chromatic adaptation method for D50/D65 routes, likely Bradford.
- Decide default serializer precision for CSS and object outputs.
- Decide how strict parsing options differ from default parsing options.

Acceptance criteria:

- Decisions are captured in `spec/v1/README.md`, `packages/typescript/README.md`, or a short decision section in this roadmap.
- No TypeScript API work starts while these basics are unresolved.

## Milestone 1: Shared Spec Skeleton

Goal: create the shared contract that the TypeScript package will consume and test against.

Deliverables:

- `spec/v1/README.md`
- `spec/v1/features.yaml`
- `spec/v1/errors.yaml`
- `spec/v1/tolerances.yaml`
- `spec/v1/css-color.md`
- `spec/v1/serialization.md`
- `spec/v1/named-colors.yaml`
- `spec/v1/color-spaces/README.md`
- `spec/v1/color-spaces/registry.yaml`
- `spec/v1/transforms/transfer-functions.yaml`
- `spec/v1/transforms/chromatic-adaptation.yaml`

`features.yaml` should define stable feature IDs and status:

- `mvp`: required for the TypeScript MVP.
- `planned`: expected later, not implemented now.
- `experimental`: tracked but unstable.
- `unsupported`: intentionally rejected.

Initial MVP feature IDs should cover:

- named color parsing
- hex parsing
- RGB and RGBA parsing
- HSL and HSLA parsing
- HWB parsing
- Lab and LCh parsing
- OKLab and OKLCh parsing
- sRGB conversion
- linear sRGB conversion
- HSL conversion
- HWB conversion
- XYZ D65 conversion
- XYZ D50 conversion
- Lab conversion
- LCh conversion
- OKLab conversion
- OKLCh conversion
- hex serialization
- legacy CSS serialization
- modern CSS serialization
- object serialization
- array serialization
- relative luminance
- WCAG contrast ratio
- tolerance equality

`errors.yaml` should define public error categories:

- `invalid-syntax`
- `unsupported-color-space`
- `invalid-channel`
- `missing-component`
- `out-of-gamut-for-output`
- `lossy-serialization`
- `ambiguous-color`
- `conformance-error`

`tolerances.yaml` should define named tolerances:

- parser normalization tolerance
- transfer function tolerance
- matrix conversion tolerance
- chromatic adaptation tolerance
- polar coordinate tolerance
- round-trip tolerance
- contrast tolerance
- equality default tolerance

Acceptance criteria:

- Spec files are parseable by a simple validator.
- Every MVP feature has an ID.
- Every public error the TypeScript package can throw has a shared category.
- Every numeric fixture can refer to a named tolerance.

## Milestone 2: Color Space And Transform Spec

Goal: define the MVP spaces and conversion metadata before writing package code.

MVP spaces:

- `srgb`
- `srgb-linear`
- `hsl`
- `hwb`
- `xyz-d65`
- `xyz-d50`
- `lab`
- `lch`
- `oklab`
- `oklch`

Each color-space YAML file should define:

- stable ID
- aliases
- feature gate
- channel names
- channel units
- channel ranges
- channel bounds
- missing-value rules
- alpha behavior
- white point where applicable
- observer where applicable
- transfer function where applicable
- connection space
- transform metadata
- CSS syntax support
- serialization support
- fixture coverage expectations
- standards or oracle references

Transform files should define:

- sRGB companding and inverse companding.
- sRGB to XYZ D65 matrix.
- XYZ D65 to sRGB matrix.
- Bradford D65 to D50 adaptation.
- Bradford D50 to D65 adaptation.
- XYZ D50 to Lab behavior.
- Lab to XYZ D50 behavior.
- Lab to LCh and LCh to Lab behavior.
- sRGB to OKLab behavior.
- OKLab to sRGB behavior.
- OKLab to OKLCh and OKLCh to OKLab behavior.
- HSL and HWB algorithms against sRGB.

Acceptance criteria:

- The registry lists only supported MVP spaces plus explicitly feature-gated future spaces if needed for documentation.
- The registry does not duplicate every constant from individual space files.
- Every transform used by TypeScript has a spec entry.
- White-point assumptions are explicit for XYZ, Lab, and LCh.

## Milestone 3: Shared Fixtures

Goal: define expected behavior before the TypeScript implementation is written against it.

Deliverables:

- `fixtures/v1/manifest.json`
- valid parse fixtures under `fixtures/v1/parse/`
- invalid parse fixtures under `fixtures/v1/invalid/`
- conversion fixtures under `fixtures/v1/convert/`
- round-trip fixtures under `fixtures/v1/roundtrip/`
- serialization fixtures under `fixtures/v1/serialize/`
- contrast fixtures under `fixtures/v1/contrast/`

Every fixture should include:

- `id`
- `feature`
- `operation`
- `input`
- `expected`
- `tolerance`, when numeric comparison is required
- `source`
- `required`

Valid parse fixtures should cover:

- all CSS named color aliases from the MVP named color table
- transparent handling if included in MVP parsing
- three, four, six, and eight digit hex
- legacy comma RGB and RGBA syntax
- modern space-separated RGB syntax
- RGB percentages
- alpha as number and percentage
- HSL angles in degrees
- HSL hue wrapping
- HSL powerless hue behavior
- HWB percentages
- Lab percentages and numeric channels
- LCh hue angles and powerless hue behavior
- OKLab numeric channels
- OKLCh hue angles
- whitespace and case normalization

Invalid parse fixtures should cover:

- malformed hex lengths
- invalid channel counts
- invalid separators
- unsupported color spaces
- unsupported CSS Color 5 functions
- unsupported `color()` syntax during MVP
- alpha outside accepted strict bounds
- invalid hue tokens
- missing required channels
- unknown named colors
- ambiguous structured input

Conversion fixtures should cover:

- black, white, red, green, blue, cyan, magenta, and yellow
- mid gray
- non-round colors such as `rgb(12 34 56)`
- alpha preservation
- sRGB to linear sRGB
- linear sRGB to sRGB
- sRGB to XYZ D65
- XYZ D65 to sRGB
- XYZ D65 to XYZ D50
- XYZ D50 to Lab
- Lab to LCh
- sRGB to OKLab
- OKLab to OKLCh
- HSL to sRGB
- HWB to sRGB
- out-of-gamut coordinate preservation where the target model allows it

Serialization fixtures should cover:

- hex lower-case output
- hex with alpha when representable
- legacy `rgb()` and `rgba()` output where representable
- modern CSS output for supported spaces
- object output with stable keys
- array output with documented order
- exact rounding behavior
- negative zero handling
- lossy serialization errors
- out-of-gamut-for-output errors

Contrast fixtures should cover:

- relative luminance for black, white, gray, and primary colors
- WCAG contrast for black on white
- WCAG contrast for identical colors
- alpha handling policy for contrast inputs
- tolerance around floating-point output

Acceptance criteria:

- TypeScript can run all MVP fixtures.
- Every skipped fixture is tied to an explicit non-MVP feature gate.
- Fixture IDs are stable and useful in failure messages.
- Reference values are traceable to specs, documented formulas, or pinned oracle tooling.

## Milestone 4: Validation And Test Tooling

Goal: make spec and fixture correctness repeatable before package work grows.

Deliverables:

- `tools/spec/validate`
- `tools/fixtures/validate`
- `tools/test/package`
- first version of `tools/test/pr`
- gitignored `reports/` output once reports are generated

`tools/spec/validate` should:

- validate YAML and JSON syntax
- require unique feature IDs
- require unique error categories
- require unique color-space IDs and aliases
- verify registry paths exist
- verify every MVP color space has required metadata
- verify transform IDs referenced by color spaces exist

`tools/fixtures/validate` should:

- validate fixture shape
- verify fixture IDs are unique
- verify fixture feature IDs exist
- verify fixture operations are known
- verify named tolerances exist
- verify required fields exist for each operation
- verify skipped fixtures are feature-gated

`tools/test/package typescript` should:

- run TypeScript package checks
- run TypeScript conformance tests against `fixtures/v1/manifest.json`
- write a JSON conformance report
- preserve fixture IDs in failures

`tools/test/pr` should eventually run:

- spec validation
- fixture validation
- TypeScript format check
- TypeScript lint
- TypeScript typecheck
- TypeScript unit tests
- TypeScript conformance tests
- TypeScript build
- package install/import smoke test
- documentation example tests
- conformance summary report

Acceptance criteria:

- A local contributor can run one command before opening a pull request.
- CI can call the same command or the same underlying steps.
- Test failures identify the fixture or spec entry that failed.

## Milestone 5: TypeScript Package Scaffold

Goal: create a minimal npm package that can compile, test, and consume the shared contract.

Deliverables:

- `packages/typescript/package.json`
- `packages/typescript/README.md`
- `packages/typescript/tsconfig.json`
- `packages/typescript/vitest.config.ts`
- source directory structure
- unit test directory
- conformance test directory
- examples directory

Suggested source layout:

```text
packages/typescript/src/
  index.ts
  color.ts
  parse/
  format/
  convert/
  spaces/
  contrast/
  errors/
  internal/
```

Suggested test layout:

```text
packages/typescript/test/
  unit/
  conformance/
  smoke/
```

Package setup should include:

- strict TypeScript settings
- explicit package exports
- declaration output
- source maps if useful for debugging
- deterministic formatting
- linting
- Vitest unit and conformance tests
- build script
- conformance script
- npm pack dry-run script

Acceptance criteria:

- The package installs locally.
- `npm test` runs at least placeholder tests.
- `npm run build` emits JavaScript and declarations.
- The package can import spec and fixture files from the repository.

## Milestone 6: Public API Draft

Goal: define the TypeScript surface area before the implementation spreads.

Initial API concepts:

```ts
export type ColorSpaceId =
  | "srgb"
  | "srgb-linear"
  | "hsl"
  | "hwb"
  | "xyz-d65"
  | "xyz-d50"
  | "lab"
  | "lch"
  | "oklab"
  | "oklch";

export type ChannelValue = number | "none";

export interface Color {
  readonly space: ColorSpaceId;
  readonly coords: readonly ChannelValue[];
  readonly alpha: ChannelValue;
  readonly source?: ParsedSource;
  readonly flags?: ColorFlags;
}

export function parse(input: string, options?: ParseOptions): Color;
export function convert(color: Color, target: ColorSpaceId, options?: ConvertOptions): Color;
export function format(color: Color, mode: FormatMode, options?: FormatOptions): string | ColorObject | readonly number[];
export function equals(left: Color, right: Color, options?: EqualsOptions): boolean;
export function luminance(color: Color, options?: LuminanceOptions): number;
export function contrast(foreground: Color, background: Color, options?: ContrastOptions): number;
```

API rules:

- Keep values immutable.
- Keep alpha explicit and default to `1` when omitted.
- Keep color-space IDs stable and lower-case.
- Avoid browser-dependent behavior.
- Avoid implicit gamut clipping during conversion.
- Require explicit policy for lossy serialization.
- Throw typed errors that map to `spec/v1/errors.yaml`.
- Return numbers in normalized canonical coordinate ranges unless a documented format says otherwise.

Acceptance criteria:

- API examples cover parse, convert, format, equality, luminance, and contrast.
- Public types are documented.
- Errors are predictable enough for callers to catch by category.
- The API does not expose future package or advanced module concepts prematurely.

## Milestone 7: Core Color Model And Errors

Goal: implement the foundation used by parse, convert, format, and analysis.

Tasks:

- Implement immutable `Color` construction helpers.
- Validate known color-space IDs.
- Validate coordinate counts.
- Normalize alpha behavior.
- Represent missing components only where MVP syntax requires it.
- Preserve source metadata when parsing strings.
- Implement public error classes.
- Map every public error to a shared error category.
- Add helper functions for checking finite numbers, percentages, angles, and bounded outputs.

Unit tests should cover:

- valid color construction
- invalid coordinate counts
- invalid space IDs
- alpha defaults
- alpha validation
- missing component behavior
- error category stability

Acceptance criteria:

- Internal modules do not pass untyped tuples without space metadata.
- Invalid states are rejected at the package boundary.
- Error snapshots or exact category tests prevent accidental error drift.

## Milestone 8: Conversion Engine

Goal: implement MVP conversions with explicit routes and tolerances.

Tasks:

- Implement sRGB transfer functions.
- Implement sRGB to linear sRGB.
- Implement linear sRGB to sRGB.
- Implement sRGB to XYZ D65.
- Implement XYZ D65 to sRGB.
- Implement XYZ D65 to XYZ D50.
- Implement XYZ D50 to XYZ D65.
- Implement XYZ D50 to Lab.
- Implement Lab to XYZ D50.
- Implement Lab to LCh.
- Implement LCh to Lab.
- Implement sRGB to OKLab.
- Implement OKLab to sRGB.
- Implement OKLab to OKLCh.
- Implement OKLCh to OKLab.
- Implement sRGB to HSL.
- Implement HSL to sRGB.
- Implement sRGB to HWB.
- Implement HWB to sRGB.
- Preserve alpha through every route.
- Preserve out-of-gamut numeric coordinates internally.
- Attach route metadata internally if needed for debugging and conformance reports.

Recommended route policy:

- Use connection spaces rather than hand-writing every pairwise conversion.
- Keep D50/D65 adaptation explicit.
- Keep polar and rectangular space conversions separate.
- Do not silently clamp conversion output.
- Clamp only in explicitly bounded serialization or explicit gamut policy helpers.

Unit tests should cover:

- formula branch boundaries
- primary colors
- grayscale colors
- hue wrapping
- powerless hue handling
- alpha preservation
- out-of-gamut preservation
- round-trip tolerances

Acceptance criteria:

- Conversion fixtures pass for every MVP space.
- Route behavior is documented.
- Numeric drift stays inside `spec/v1/tolerances.yaml`.

## Milestone 9: CSS Parser

Goal: parse the MVP CSS input surface with clear error behavior.

Tasks:

- Build a small tokenizer or parser structure that can distinguish functions, idents, numbers, percentages, angles, separators, slash alpha, and whitespace.
- Parse named colors from `spec/v1/named-colors.yaml`.
- Parse `transparent` if included in the named-color contract.
- Reject `currentColor` for MVP unless explicitly feature-gated and represented.
- Parse hex forms.
- Parse legacy comma `rgb()` and `rgba()`.
- Parse modern space-separated `rgb()`.
- Parse RGB percentages and numeric channels.
- Parse alpha values as numbers and percentages.
- Parse legacy comma `hsl()` and `hsla()`.
- Parse modern space-separated `hsl()`.
- Parse hue angles.
- Parse `hwb()`.
- Parse `lab()`.
- Parse `lch()`.
- Parse `oklab()`.
- Parse `oklch()`.
- Normalize case and whitespace.
- Reject unsupported CSS Color 5 functions with stable errors.
- Reject `color()` during MVP unless the feature gate changes.

Parser edge cases:

- channel count mismatches
- mixed comma and space syntax
- missing slash before modern alpha
- invalid percentage placement
- hue units and unitless hue decisions
- negative channels where standards allow them
- values outside legacy display bounds
- `none` handling in modern syntax
- nested functions are unsupported in MVP

Acceptance criteria:

- Valid parse fixtures pass exactly.
- Invalid parse fixtures fail with exact shared categories.
- Parser behavior is independent of browser APIs.
- Parser implementation is maintainable enough to extend for `color()` later.

## Milestone 10: Serialization And Formatting

Goal: provide predictable output modes without hiding lossy behavior.

MVP format modes:

- `hex`
- `css-legacy`
- `css-modern`
- `object`
- `array`

Tasks:

- Define output precision in `spec/v1/serialization.md`.
- Implement lower-case hex serialization.
- Implement alpha hex serialization when alpha is representable.
- Implement legacy `rgb()` and `rgba()` output only for bounded sRGB values.
- Implement legacy `hsl()` and `hsla()` output only where representable and requested.
- Implement modern `rgb()`, `hsl()`, `hwb()`, `lab()`, `lch()`, `oklab()`, and `oklch()` output.
- Implement object output with stable keys.
- Implement array output with documented coordinate order.
- Normalize negative zero.
- Surface `lossy-serialization` when requested output cannot preserve the value under the chosen policy.
- Surface `out-of-gamut-for-output` when bounded output requires clipping and no policy was provided.

Serializer policy options should cover:

- precision
- compact versus readable CSS
- alpha omission when alpha is `1`
- gamut policy for bounded formats
- lossy behavior: throw, warn metadata, or explicit allow

Acceptance criteria:

- Serialization fixtures pass exactly.
- Hex and legacy CSS never silently hide out-of-gamut values.
- Object and array formats are stable enough for tests and documentation examples.

## Milestone 11: Analysis Helpers

Goal: support the basic analysis operations promised by the MVP.

Tasks:

- Implement WCAG 2.x relative luminance from sRGB.
- Implement WCAG 2.x contrast ratio.
- Implement tolerance-based equality.
- Define behavior when inputs are not already sRGB.
- Define behavior when inputs include alpha.
- Define behavior when inputs contain missing components.

Recommended MVP behavior:

- Convert inputs to sRGB for luminance and contrast.
- Require callers to composite alpha before contrast unless a clear background option is provided.
- Throw `missing-component` when analysis requires a numeric channel that is missing.
- Use named tolerances for equality defaults.

Acceptance criteria:

- Contrast fixtures pass.
- Analysis errors use shared categories.
- Documentation warns that WCAG contrast ratio is not APCA and is not WCAG 3.

## Milestone 12: Conformance Test Adapter

Goal: prove the TypeScript package against the shared fixtures.

Tasks:

- Load `fixtures/v1/manifest.json`.
- Select required MVP fixtures.
- Skip only fixtures behind unsupported feature gates.
- Execute parse fixtures.
- Execute invalid parse fixtures.
- Execute conversion fixtures.
- Execute round-trip fixtures.
- Execute serialization fixtures.
- Execute contrast fixtures.
- Compare strings exactly.
- Compare error categories exactly.
- Compare numbers with named tolerances.
- Preserve fixture IDs in failures.
- Emit `reports/typescript.json`.

Acceptance criteria:

- `npm run test:conformance` fails with useful fixture IDs.
- The TypeScript package cannot claim an MVP feature unless it passes that feature's required fixtures.
- The report can feed `reports/conformance-summary.json` later.

## Milestone 13: Package Documentation

Goal: make the first npm package usable by early adopters.

Deliverables:

- `packages/typescript/README.md`
- getting started examples
- parse guide
- conversion guide
- formatting guide
- contrast guide
- supported color spaces list
- supported CSS syntax list
- error behavior section
- gamut and lossy output section
- conformance section
- migration notes for future breaking changes during `0.x`

Examples should cover:

```ts
import { contrast, convert, format, parse } from "PACKAGE_NAME";

const color = parse("oklch(62% 0.18 28)");
const srgb = convert(color, "srgb");
const css = format(srgb, "css-modern");
const ratio = contrast(parse("#000"), parse("#fff"));
```

Documentation rules:

- Do not imply support for deferred spaces.
- Do not imply print-accurate CMYK support.
- Do not imply ICC support.
- Do not imply APCA support.
- Do not imply other language packages exist.
- State that TypeScript is the first MVP implementation.

Acceptance criteria:

- Every documented example is covered by a test or smoke check.
- The README explains what is supported now and what is not.
- The package docs link behavior back to `spec/v1` and `fixtures/v1`.

## Milestone 14: CI And Local PR Gate

Goal: make the MVP reliable enough to iterate safely.

Minimum local gate:

```sh
./tools/test/pr
```

The gate should run:

- spec validation
- fixture validation
- TypeScript dependency install check
- TypeScript format check
- TypeScript lint
- TypeScript typecheck
- TypeScript unit tests
- TypeScript conformance tests
- TypeScript build
- package import smoke test
- documentation example tests
- npm pack dry run

Acceptance criteria:

- CI and local development use the same checks where practical.
- Generated reports are written under `reports/` and ignored by git.
- The PR gate is documented in the root README or TypeScript README.

## Milestone 15: MVP Release Preparation

Goal: prepare a cautious `0.1.0` release.

Tasks:

- Confirm package name availability.
- Confirm package metadata.
- Confirm license metadata.
- Confirm included files in npm package.
- Confirm package exports.
- Confirm TypeScript declaration output.
- Confirm source maps if included.
- Run npm pack dry run.
- Install the packed tarball in a temporary project.
- Import the package in Node.
- Run a small browser bundling smoke test if browser support is claimed.
- Confirm README examples work from the packed package.
- Write `CHANGELOG.md` entry.
- Add release checklist.

Acceptance criteria:

- The package can be installed from the packed tarball.
- Public API examples work from the packed tarball.
- The package passes all MVP fixtures.
- Unsupported features fail clearly.
- Release notes state that this is a TypeScript MVP and that other packages are not published yet.

## Recommended Implementation Order

1. Lock foundation decisions.
2. Create `spec/v1`.
3. Create `fixtures/v1`.
4. Create validation tools.
5. Scaffold `packages/typescript`.
6. Implement errors and core color model.
7. Implement conversion engine.
8. Implement parser.
9. Implement serialization.
10. Implement luminance, contrast, and equality.
11. Implement conformance adapter.
12. Write TypeScript package documentation.
13. Build local PR gate.
14. Prepare `0.1.0` release.

This order keeps behavior decisions ahead of implementation. It also prevents the TypeScript package from becoming the only undocumented source of truth.

## Decisions To Revisit Before Release

These are not blockers for starting, but they must be decided before publishing:

- Final npm package name.
- ESM-only versus dual ESM/CommonJS package output.
- Minimum supported Node.js version.
- Whether `transparent` is represented as a named color, a keyword, or normalized `srgb` with alpha `0`.
- Whether `currentColor` is rejected or represented as an unresolved keyword.
- Whether CSS `none` is supported in MVP parsing or rejected until a fuller missing-component model exists.
- Default serializer precision.
- Default equality tolerance.
- Default chromatic adaptation route.
- Default handling of alpha in contrast calculations.
- Whether conformance reports are JSON only or also summarized in text.

## MVP Exit Criteria

The MVP is done when all of the following are true:

- `spec/v1` defines every MVP feature, space, error, tolerance, and serialization rule used by TypeScript.
- `fixtures/v1` contains required fixtures for parse, invalid parse, convert, round-trip, serialize, and contrast behavior.
- `packages/typescript` exposes `Color`, `ColorSpaceId`, `parse`, `convert`, `format`, `equals`, `luminance`, and `contrast`, or documented equivalents.
- The TypeScript package passes all required MVP fixtures.
- The TypeScript package has native unit tests for formulas, parser behavior, serialization, errors, and analysis helpers.
- The TypeScript package builds JavaScript and TypeScript declarations.
- The package install/import smoke test passes.
- Documentation examples run successfully.
- `./tools/test/pr` passes locally.
- The repository does not contain placeholder future language packages.
- Deferred features are documented as planned, experimental, or unsupported.

## Post-MVP Hold Line

After the TypeScript MVP, the next work should be chosen from real evidence: fixture stability, user feedback, package adoption, and implementation pain. Do not start another language package until the shared spec and fixture contract have proven useful in TypeScript.
