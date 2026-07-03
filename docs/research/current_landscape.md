# Current Color Conversion Landscape

Researched: 2026-07-03

Scope: color value parsing, conversion, formatting, comparison, accessibility, gamut handling, profile-aware workflows, HDR/video color, design-token interoperability, and cross-language library strategy for the planned Color product.

## Executive Summary

The color conversion ecosystem is broad but fragmented. Web standards have moved past `hex`/`rgb()`/`hsl()` into CSS Color 4 and 5 syntax, wide-gamut RGB spaces, Lab/LCh, OKLab/OKLCh, relative color syntax, `color-mix()`, profile-backed `color()`, and early `device-cmyk()` work. Design tokens are converging on structured color values that need explicit color spaces and alpha handling. Accessibility guidance still centers on WCAG 2.x contrast ratios, while perceptual contrast work remains unsettled. Print and device workflows require ICC profile support and rendering intents. Video and HDR workflows add transfer functions, range conventions, scene/display references, and spaces such as Rec. 2020, Rec. 2100, ICtCp, PQ, and HLG.

No widely adopted library appears to cover the whole space with strong color-science rigor, modern CSS parsing, profile/HDR awareness, explicit failure modes, and cross-language conformance. Existing tools cluster into four categories:

- Web color libraries: good ergonomics and CSS familiarity, uneven standards depth, usually JavaScript-only.
- Scientific color libraries: broad formulas and appearance models, usually heavy and Python/R/Rust-specific.
- Color-management engines: correct profile/device conversion, usually low-level and not ergonomic for app developers.
- Platform APIs: useful native primitives, but inconsistent behavior, limited portability, and weak shared fixtures.

The product opportunity is to become the standard conversion contract across languages: one specification, one fixture corpus, native-feeling packages, explicit options for lossy choices, and documented numerical tolerances.

## What "Definitive" Requires

A definitive color conversion library must cover more than pairwise transforms. It needs to define:

| Area | Required product stance |
| --- | --- |
| Syntax | Parse and serialize modern CSS, legacy CSS, hex variants, structured objects, arrays, design-token values, and platform-oriented forms. |
| Color-space metadata | Every space must define coordinate ranges, white point, transfer function, matrices, reference viewing assumptions where applicable, and alpha semantics. |
| Conversion graph | Routes must be explicit, versioned, tested, and documented. Hidden clipping, hidden chromatic adaptation, and hidden profile assumptions should be avoided. |
| Gamut behavior | Out-of-gamut values must be representable. Clipping, chroma reduction, perceptual mapping, and "preserve coordinates" should be explicit modes. |
| Missing and powerless components | CSS `none`, powerless hue, undefined hue, NaN-like channel states, and alpha absence must round-trip where standards allow. |
| Precision | Floating-point precision, rounding, serialization precision, integer channel quantization, and tolerance bands must be documented. |
| Interpolation | Interpolation space, hue interpolation method, premultiplied alpha handling, and gamut mapping timing must be explicit. |
| Accessibility | WCAG 2.x relative luminance and contrast should be supported, with perceptual contrast clearly labeled as experimental or non-normative until standardized. |
| Profiles | ICC/profile-backed conversion should be optional but first-class, with rendering intent, black point compensation, PCS, and missing-profile failure modes. |
| Cross-language behavior | Fixtures must be shared across packages; package APIs can be idiomatic, but behavior must be conformance-tested. |

## Standards And Specs Map

### Web And CSS

CSS is the dominant interchange layer for application-facing color. CSS Color 4 and CSS Color 5 are the baseline research targets.

| Spec area | Current importance | Product implication |
| --- | --- | --- |
| CSS Color 4 | Defines modern CSS color syntax, `lab()`, `lch()`, `oklab()`, `oklch()`, `color()`, predefined RGB spaces, interpolation behavior, hue handling, gamut mapping guidance, serialization, and named colors. | Treat as v1 web compatibility target. Track Web Platform Tests and spec sample code. |
| CSS Color 5 | Adds and evolves `color-mix()`, relative color syntax, `contrast-color()`, `light-dark()`, custom ICC profile hooks, and `device-cmyk()`. | Implement stable portions behind capability flags or versioned behavior. Keep draft features separable from stable CSS Color 4 support. |
| CSSOM/browser serialization | Browsers can serialize colors differently across legacy and modern syntax. | Formatter APIs need target modes: "authoring", "browser-compatible", "legacy", "lossless where possible". |
| `color()` function | Enables predefined spaces such as `display-p3`, `a98-rgb`, `prophoto-rgb`, `rec2020`, `srgb`, `srgb-linear`, `xyz`, `xyz-d50`, and `xyz-d65`. | Internal registry must support named spaces plus future custom profiles. |
| Relative colors | Lets channels be derived from another color in a chosen space. | Parser must preserve unresolved expressions or expose an evaluation API with an environment. |
| `none` channels | CSS can represent missing components. | Data model cannot be a simple tuple of numbers only. |

Primary standard: [CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/) and [CSS Color Module Level 5](https://www.w3.org/TR/css-color-5/).

### Design Tokens

Design tokens are becoming a key structured interchange format for design systems. The W3C Design Tokens Community Group format aligns color tokens with typed values rather than raw strings.

| Requirement | Product implication |
| --- | --- |
| Parse `$type: "color"` values and preserve alpha. | Treat design-token values as first-class parse/format targets. |
| Preserve source color space and component names. | Avoid normalizing tokens to hex by default. |
| Support modern color spaces, not only sRGB hex. | Token serialization should be able to emit CSS-compatible strings and structured token objects. |
| Round-trip metadata. | Keep original syntax, normalized coordinates, and target serialization separate. |

Primary standard: [Design Tokens Community Group Format Module](https://design-tokens.github.io/community-group/format/).

### Accessibility And Contrast

WCAG 2.2 remains the normative accessibility baseline for contrast in most product workflows. It uses relative luminance and contrast ratios for text and non-text contrast. APCA and other perceptual contrast approaches are important research inputs, but they should not be presented as a WCAG 2.x replacement unless a consuming product explicitly chooses them.

| Method | Status | Product stance |
| --- | --- | --- |
| WCAG 2.x contrast ratio | Normative and widely required. | Implement exactly, including relative luminance definition and threshold helpers. |
| Non-text contrast | WCAG 2.x requirement for UI components and graphical objects. | Provide helpers, but do not infer semantic UI roles from colors alone. |
| APCA/perceptual contrast | Useful and influential but still standards-moving. | Provide only with explicit labeling, versioning, and references. |
| CSS `contrast-color()` | Draft CSS feature; selection algorithm may be UA-defined. | Do not conflate with WCAG compliance. Expose as CSS feature separately. |

Primary standard: [WCAG 2.2](https://www.w3.org/TR/WCAG22/).

### CIE And Color Science

CIE spaces and measurements remain the mathematical substrate for many conversions and metrics.

| Area | Product implication |
| --- | --- |
| CIE XYZ | Core profile connection and conversion hub; must distinguish D50, D65, 2 degree observer, and other assumptions. |
| CIE Lab/LCh | Required for CSS, print-adjacent workflows, and legacy perceptual operations; white point and adaptation must be explicit. |
| CIE Luv/LChuv | Less common in modern CSS but important for legacy and scientific compatibility. |
| Delta E metrics | Support CIE76, CIE94, CIEDE2000, and CMC l:c with documented parameter defaults. |
| Chromatic adaptation | Bradford is common, but CAT02/CAT16 and identity routes matter. Adaptation method must be an option or embedded route metadata. |
| Appearance models | CAM16/CAM16-UCS and Jzazbz are valuable for advanced use, but require careful viewing-condition semantics. |

Practical note: CIE publications are authoritative but often paywalled. Implementation docs should cite the exact CIE standard where used and include independently testable fixtures.

### ICC, Print, And Device Color

ICC profile workflows are unavoidable for serious print, PDF, asset pipelines, and device-specific conversion.

| Area | Product implication |
| --- | --- |
| ICC v4 profiles | Support profile metadata, profile connection space, rendering intents, PCS Lab/XYZ, and D50 behavior. |
| iccMAX | Research target for spectral and advanced profile workflows. |
| CMYK | Device CMYK is not one universal color space. Treat generic formula CMYK separately from profile-backed CMYK. |
| `device-cmyk()` | CSS draft feature with fallback behavior. Should be implemented as syntax plus explicit print/profile semantics, not generic CMYK magic. |
| Spot colors | Often proprietary or workflow-specific. Treat as metadata/research unless licensed datasets and output workflows are defined. |
| CMM integration | LittleCMS is the pragmatic open-source engine for ICC conversion; bindings may be preferable to reimplementing CMM behavior. |

Primary sources: [ICC.1:2022 profile specification](https://www.color.org/specification/ICC.1-2022-05.pdf), [ICC specifications](https://www.color.org/specification/), and [LittleCMS](https://www.littlecms.com/).

### Video, HDR, And Wide Gamut

Video color spaces are not just RGB primaries. They combine primaries, transfer functions, matrix coefficients, quantization ranges, reference levels, and scene/display assumptions.

| Area | Product implication |
| --- | --- |
| Rec. 709 | Common SDR video baseline; related to but not identical with generic sRGB workflows. |
| Rec. 2020 | Wide-gamut RGB primaries used in UHD/HDR contexts. |
| Rec. 2100 | HDR system definition using PQ or HLG with wide-gamut primaries. |
| PQ / ST 2084 | Absolute display-referred HDR transfer function. |
| HLG | Relative scene/display system for HDR broadcast. |
| YCbCr/YPbPr/YUV/YIQ | Must define matrix coefficients, legal/full range, bit depth, chroma siting where image/video use is in scope. |
| ICtCp | HDR-oriented color representation used with BT.2100. |

Product boundary: support color-value conversion and metadata-aware transforms; avoid becoming a full video/image-processing library unless future scope explicitly expands.

Primary standards: [ITU-R BT.709](https://www.itu.int/rec/R-REC-BT.709), [ITU-R BT.2020](https://www.itu.int/rec/R-REC-BT.2020), and [ITU-R BT.2100](https://www.itu.int/rec/R-REC-BT.2100).

### Film, VFX, And Scene-Referred Color

VFX pipelines rely on scene-referred interchange and configuration-based color management.

| Area | Product implication |
| --- | --- |
| ACES2065-1 | Archival/interchange scene-linear color space. |
| ACEScg | Common computer-graphics working space. |
| ACEScc / ACEScct | Log spaces used for grading-style operations. |
| OpenColorIO | Configuration-driven color management used in VFX and animation pipelines. |

Product stance: support ACES spaces as named spaces after the v1 web/CIE core is stable. Treat OCIO interop as advanced integration, not core conversion graph policy.

Primary sources: [ACES documentation](https://docs.acescentral.com/) and [OpenColorIO](https://opencolorio.org/).

### Native Platform APIs

Platform color APIs are useful but not portable enough to define product behavior.

| Platform | Current role | Gap for this product |
| --- | --- | --- |
| Web browsers | Primary runtime target for CSS parsing, wide-gamut color, Canvas, and UI output. | Browser support and serialization vary; standards and WPT should define expected behavior. |
| Android `ColorSpace` | Strong native model with named RGB spaces and connectors. | Android-only, not a cross-language spec. |
| Apple CoreGraphics/ColorSync | Mature ICC and display-profile ecosystem. | Powerful but platform-specific and bridged through OS behavior. |
| Skia | Important rendering engine with color-space primitives. | Rendering-focused; not a standalone app-level parsing/conversion contract. |
| .NET/System.Drawing/WinUI | Useful color structs and platform integration. | Usually sRGB-oriented or platform-bound; weak modern CSS coverage. |

References: [Android ColorSpace](https://developer.android.com/reference/android/graphics/ColorSpace), [Apple CGColorSpace](https://developer.apple.com/documentation/coregraphics/cgcolorspace), and [Skia color spaces](https://skia.org/docs/user/api/skcolorspace_overview/).

## Library Landscape

### JavaScript And TypeScript

JavaScript is the most mature ecosystem for modern CSS-facing color libraries, but it is still fragmented.

| Library | Strengths | Limits / product opening |
| --- | --- | --- |
| [Color.js](https://colorjs.io/) | Deep CSS Color 4 alignment, modern spaces, interpolation, gamut mapping, Delta E, authored by CSS color experts. | JS-focused; API and package stability are narrower than a cross-language conformance program. Strong reference competitor. |
| [Culori](https://culorijs.org/) | Functional API, broad color spaces, good modern CSS support, composable converters. | JS-only; less positioned as a formal multi-language standard. |
| [chroma.js](https://gka.github.io/chroma.js/) | Popular manipulation/interpolation library; approachable API. | Older model focus; modern CSS/profile/HDR coverage is not the core promise. |
| [d3-color](https://d3js.org/d3-color) | Well-known visualization integration; RGB/HSL/Lab/HCL/Cubehelix. | Visualization utility scope, not comprehensive parsing/profile/HDR. |
| [color-convert](https://github.com/Qix-/color-convert) / [color](https://github.com/Qix-/color) | Widely used dependency layer for common conversions. | Legacy/common spaces; not a rigorous standards-level product. |
| [tinycolor](https://github.com/bgrins/TinyColor) / forks | Familiar small utility for web colors. | Legacy CSS and manipulation focus. |
| [polished](https://polished.js.org/) | CSS-in-JS design utility helpers. | Not a color-science foundation. |
| [Material Color Utilities](https://github.com/material-foundation/material-color-utilities) | HCT/CAM16-derived design-system color generation. | Material-specific purpose; useful as a contrast/palette reference, not general conversion coverage. |

Strategic read: JS has the strongest single-language competitors. The product must either interoperate with or exceed Color.js/Culori quality for CSS Color 4/5 while differentiating through cross-language guarantees, fixture governance, strict error modes, and profile/HDR extension paths.

### Python

Python has the best scientific color tooling and a growing modern CSS-capable layer.

| Library | Strengths | Limits / product opening |
| --- | --- | --- |
| [Colour Science](https://www.colour-science.org/) | Very broad color-science library: CIE, CAMs, spectral, HDR, plotting, datasets. | Heavy scientific API; not optimized for simple product parsing/formatting or cross-language app SDK parity. |
| [ColorAide](https://facelessuser.github.io/coloraide/) | Modern CSS-oriented parsing/conversion, many spaces, Delta E, gamut mapping. | Python-only; stronger as a library than as a cross-language standard. |
| [colorspacious](https://colorspacious.readthedocs.io/) | CIECAM02/CAM02-UCS and perceptual transforms. | Narrower and older; not full modern CSS/profile coverage. |
| [python-colormath](https://python-colormath.readthedocs.io/) | Familiar CIE and Delta E formulas. | Maintenance and modern CSS coverage are weak relative to current needs. |
| [Pillow ImageCms](https://pillow.readthedocs.io/en/stable/reference/ImageCms.html) | LittleCMS-backed ICC workflows in image contexts. | Image/profile operations, not a general color-value conversion API. |
| [matplotlib.colors](https://matplotlib.org/stable/api/colors_api.html) | Colormap and plotting color parsing. | Visualization-scoped. |

Strategic read: Python can borrow the scientific breadth of Colour Science and the CSS practicality of ColorAide, but the product should not inherit their complexity. A lightweight, conformance-driven Python package can coexist with scientific dependencies.

### Rust

Rust has strong type-safety potential for color spaces, but fewer end-user parsing/formatting batteries.

| Library | Strengths | Limits / product opening |
| --- | --- | --- |
| [palette](https://docs.rs/palette/latest/palette/) | Type-safe color spaces, conversions, gradients, no-std possibilities. | Not a complete modern CSS/design-token/profile product. |
| [csscolorparser](https://docs.rs/csscolorparser/latest/csscolorparser/) | CSS color parsing utility. | Parsing scope, not comprehensive conversion landscape. |
| [colorgrad](https://docs.rs/colorgrad/latest/colorgrad/) | Gradients and interpolation. | Focused on gradients, not standards coverage. |

Strategic read: Rust could become a later performance-oriented implementation or shared core because type-level space metadata fits the problem. It should not be the initial reference path unless the project explicitly chooses a native shared core. If used as a shared core, keep FFI/WASM numerics deterministic and avoid forcing Rust idioms onto higher-level languages.

### Go

| Library | Strengths | Limits / product opening |
| --- | --- | --- |
| [go-colorful](https://github.com/lucasb-eyer/go-colorful) | Common RGB/HSV/HSL/Lab/Luv/HCL conversions and blending. | Older scope; no full modern CSS/profile/HDR contract. |
| Standard `image/color` | Simple RGBA-oriented primitives. | Not a conversion library. |

Strategic read: Go has room for a disciplined package if the API stays small, explicit, and dependency-light.

### Ruby

Ruby color libraries are generally small utilities for RGB/hex/HSL manipulation, terminal colors, or image integration. There is no obvious dominant modern CSS Color 4/5, ICC-aware, conformance-tested package.

Strategic read: Ruby is a good candidate for a later port once shared fixtures and API vocabulary are mature.

### JVM / Kotlin / Android

| Tooling | Strengths | Limits / product opening |
| --- | --- | --- |
| Android `ColorSpace` | Strong native named-space model and connectors. | Android API dependency; not general JVM/Kotlin. |
| Java `Color`, `ColorSpace`, `ICC_ColorSpace` | Built-in primitives and ICC integration. | Legacy ergonomics and limited modern CSS support. |
| Material Color Utilities | Modern design-system color algorithms. | Palette/theme generation focus. |

Strategic read: Kotlin/JVM support should wrap platform strengths where available but keep the product behavior governed by shared fixtures.

### Swift / Apple

Apple platforms have strong ColorSync, CoreGraphics, UIKit/AppKit, and SwiftUI color primitives, especially for display/profile workflows. However, they do not provide a portable modern CSS parsing/conversion package with cross-platform fixture parity.

Strategic read: Swift should expose idiomatic bridges to `CGColorSpace`, `Color`, `UIColor`, and `NSColor`, but the library's normalized behavior should remain independent of OS display settings unless explicitly requested.

### .NET

.NET color tooling is split among platform structs, image libraries, and community packages such as ColorMine. ImageSharp and SkiaSharp provide rendering/image color primitives, but app-level modern CSS parsing plus CIE/OKLab/profile semantics is not a unified mainstream package.

Strategic read: .NET support should target `System.Numerics`-friendly numeric operations, clear interop with platform colors, and strict separation between color values and image processing.

### PHP

PHP packages mostly cover hex/RGB/HSL manipulation, CSS parsing, or image-adjacent utilities. There is little evidence of a comprehensive modern color-science package for app developers.

Strategic read: PHP should be a later package after the core spec stabilizes; the likely user need is reliable parsing/formatting and design-token conversion.

### R

R has mature color-science and visualization tooling.

| Library | Strengths | Limits / product opening |
| --- | --- | --- |
| [colorspace](https://colorspace.r-forge.r-project.org/) | Perceptual color spaces, palettes, visualization-oriented workflows. | R/statistics ecosystem focus. |
| [farver](https://farver.data-imaginist.com/) | Fast color conversion and comparison. | R-specific; not a multi-language product contract. |

Strategic read: R is not in the README package plan, but its libraries are useful references for palette, Delta E, and performance testing.

## Capability Gaps In The Market

| Gap | Why it matters | Product requirement |
| --- | --- | --- |
| Cross-language conformance | Existing libraries disagree on parsing, white points, rounding, hue handling, gamut mapping, and errors. | One fixture corpus with per-language CI and versioned conformance levels. |
| Modern CSS completeness | Many packages support only legacy CSS; CSS Color 4/5 has spaces, `none`, relative syntax, `color-mix()`, and draft features. | CSS parser and formatter should be spec-tracked and test-backed. |
| Profile-aware app ergonomics | CMMs are powerful but low-level; high-level libraries often ignore ICC. | Optional profile module with clear CMM backend strategy. |
| CMYK ambiguity | Generic CMYK formulas are often mistaken for print correctness. | Split "formula CMYK" from "profile CMYK" in names, docs, and types. |
| Gamut mapping inconsistency | Clipping versus perceptual mapping changes output materially. | Require explicit gamut policy and expose diagnostics. |
| Appearance-model complexity | CAM16, Jzazbz, ICtCp, and HCT need context and assumptions. | Treat advanced spaces as opt-in modules with required viewing/context parameters where applicable. |
| Accessibility confusion | WCAG ratios, APCA, browser contrast choices, and design-tool scores are often conflated. | Separate normative WCAG helpers from experimental perceptual contrast APIs. |
| Serialization loss | Normalizing everything to hex loses gamut, alpha precision, missing channels, and source syntax. | Preserve parsed representation and expose target-specific serializers. |
| Proprietary systems | Pantone, Munsell books, NCS, and spot-color datasets can be licensed or restricted. | Do not ship proprietary named datasets without rights; allow user-supplied catalogs. |
| Round-trip guarantees | App developers need stable "parse -> convert -> format" behavior. | Document which operations are lossless, lossy, or syntax-preserving only. |

## Recommended Product Scope

### V1 Core

V1 should be narrow enough to ship, but broad enough to credibly support "definitive" positioning.

The first TypeScript MVP can be a smaller feature-gated subset of this table. Do not pull every V1 target into `0.1.0` if doing so delays the shared spec, fixtures, and core package.

| Area | Include in V1 |
| --- | --- |
| Parsing | CSS named colors, hex, rgb/rgba, hsl/hsla, hwb, lab, lch, oklab, oklch, `color()` predefined spaces, alpha, percentages, angles, `none` where supported. |
| Formatting | CSS modern, CSS legacy where representable, hex, numeric arrays, structured JSON objects, design-token color values. |
| Spaces | sRGB, linear sRGB, Display P3, A98 RGB, ProPhoto RGB, Rec. 2020, XYZ D50/D65, Lab, LCh, OKLab, OKLCh, Luv, LChuv, HSL, HSV/HSB, HWB, CMY, formula CMYK. |
| Operations | Convert, compare within tolerance, Delta E 76/94/2000/CMC, relative luminance, WCAG contrast, gamut check, clamp, explicit clip, mix/interpolate with hue methods. |
| Metadata | White point, transfer function, primaries, matrices, channel ranges, alpha, source syntax, conversion route. |
| Errors | Invalid syntax, unsupported space, unsupported profile, out-of-range strict mode, lossy serialization, out-of-gamut policy required. |
| Tests | WPT-aligned CSS fixtures, spec sample values, cross-language golden fixtures, fuzz/property tests for round-trip and monotonicity where applicable. |

### V1 Should Avoid

- Full ICC CMM reimplementation.
- Proprietary named color catalogs.
- Full video frame/image conversion.
- Full OCIO pipeline execution.
- Claiming APCA/WCAG 3 compliance as a stable standard.
- Treating CMYK as print-accurate without a profile.

### V2 / Advanced Modules

| Module | Scope |
| --- | --- |
| ICC/profile | LittleCMS-backed conversion, profile loading, rendering intents, black point compensation, profile metadata, CSS custom profile interop. |
| HDR/video | Rec. 709, Rec. 2020, Rec. 2100, PQ, HLG, ICtCp, full/limited range helpers, YCbCr matrix families. |
| Appearance models | CAM16, CAM16-UCS, Jzazbz, HCT, viewing-condition-aware APIs. |
| ACES/VFX | ACES2065-1, ACEScg, ACEScc, ACEScct, optional OCIO config interop. |
| Design systems | Token batch conversion, palette generation, contrast audits, design-tool import/export adapters. |
| CLI | Parse/convert/format/diff colors, validate tokens, generate fixtures. |

## API And Architecture Implications

### Data Model

Use a value object with explicit metadata:

```text
Color {
  space: ColorSpaceId
  coordinates: ChannelValue[]
  alpha: ChannelValue
  source?: ParsedSource
  profile?: ProfileRef
  flags?: ColorFlags
}
```

`ChannelValue` must be able to represent numeric values, percentages normalized to numbers, missing components, powerless hue, and possibly unevaluated relative expressions before resolution.

### Space Registry

Each registered space should define:

- Stable id and aliases.
- Channel names, units, ranges, and optional bounds.
- White point and observer where applicable.
- Transfer function or encoding.
- To/from connection-space transforms.
- Chromatic adaptation requirements.
- Gamut definition.
- Serialization support.
- Test fixture coverage.

### Conversion Policy

Every conversion should report or accept:

- Source and target spaces.
- Route version.
- Chromatic adaptation method.
- Gamut policy.
- Precision and rounding policy.
- Whether the output is in gamut.
- Whether the operation was lossy.

### Serializer Modes

| Mode | Purpose |
| --- | --- |
| `lossless` | Preserve original syntax where possible; fail if impossible. |
| `css-modern` | Emit modern CSS syntax with target space preserved. |
| `css-legacy` | Emit hex/rgb/hsl only when representable in sRGB; otherwise require gamut policy. |
| `token` | Emit structured design-token object. |
| `array` | Emit numeric tuple with documented order. |
| `platform` | Emit values suitable for a target platform adapter. |

### Error Taxonomy

Recommended public errors:

- `ColorParseError`
- `UnsupportedColorSpaceError`
- `UnsupportedProfileError`
- `OutOfGamutError`
- `LossyConversionError`
- `LossySerializationError`
- `InvalidChannelError`
- `MissingComponentError`
- `AmbiguousColorError`
- `ConformanceError`

## Conformance Strategy

The conformance suite is the product moat.

| Fixture group | Examples |
| --- | --- |
| Parsing | Valid/invalid CSS, legacy commas, modern spaces, `none`, percentages, angles, relative syntax, design tokens. |
| Serialization | Precision, minified/pretty, legacy compatibility, gamut-limited outputs, source-preserving outputs. |
| Conversion | Known values across sRGB, P3, Rec. 2020, XYZ, Lab, LCh, OKLab, OKLCh, CMYK formula. |
| Adaptation | D50/D65 round trips, Bradford/CAT02/CAT16 where supported. |
| Gamut | In-gamut, out-of-gamut, clipping, chroma reduction, no-clip preservation. |
| Interpolation | Premultiplied alpha, hue routes, rectangular/polar spaces, missing hue. |
| Difference | CIE76, CIE94, CIEDE2000, CMC l:c. |
| Accessibility | WCAG relative luminance, contrast ratios, threshold classification. |
| Error behavior | Unsupported spaces, invalid channels, lossy serialization, unknown profiles. |
| Cross-language | Same input/options/output/tolerance across all official packages. |

Fixture sources should include:

- Web Platform Tests for CSS color behavior.
- CSS spec sample code and examples.
- ICC reference profiles and public profile test cases.
- Independently generated high-precision reference values.
- Regression fixtures for bugs found in downstream integrations.

## Competitive Positioning

| Competitor type | How Color should differentiate |
| --- | --- |
| Color.js / Culori | Match modern CSS competence; exceed through cross-language conformance, stricter error semantics, broader roadmap, and stable fixture governance. |
| Colour Science | Provide a smaller app-developer API with enough scientific rigor and references; interop rather than compete on every scientific dataset. |
| LittleCMS / platform CMMs | Use as optional profile engines; wrap them with ergonomic value-level APIs and explicit failure modes. |
| Material Color Utilities | Support design-system color generation as an advanced layer, but keep general conversion independent from one design language. |
| Native platform APIs | Bridge to native values without making OS-specific behavior the source of truth. |
| Legacy web utilities | Offer an easy path for hex/RGB/HSL users while preserving modern wide-gamut and perceptual correctness. |

## Naming And Packaging Notes

- The bare package name `color` is likely unavailable or occupied in major registries; research package-name availability before committing README names.
- Use one shared spec/fixture version across packages, separate from package semver.
- Package names should avoid implying proprietary catalog coverage, such as Pantone/NCS/Munsell, unless licensed support exists.
- Consider a neutral namespace if available, for example `@color/...` on npm only if the scope can be acquired, or a project-specific namespace across ecosystems.
- Keep optional advanced modules separately installable where native dependencies are likely, especially ICC and OCIO support.

## Roadmap Implications

1. Write the shared color-space registry before writing language APIs.
2. Use TypeScript as the initial reference package, with future native ports governed by shared fixtures.
3. Build the CSS parser and formatter against CSS Color 4 first; isolate CSS Color 5 drafts behind feature flags.
4. Define loss, gamut, and error semantics before publishing any package.
5. Create fixture versioning from day one.
6. Delay profile/HDR/ACES support until the core conversion contract is stable.
7. Publish conformance badges per language and per capability group.

## Open Decisions

| Decision | Recommendation |
| --- | --- |
| Reference implementation language | Use TypeScript first because it is the initial package. Revisit Rust, C, or WebAssembly only after the API and fixture contract are stable. |
| ICC backend | Use LittleCMS through optional bindings instead of reimplementing ICC CMM behavior. |
| CSS Color 5 timing | Track now, ship stable CSS Color 4 first, mark Level 5 features experimental until spec/browser behavior settles. |
| APCA support | Research and optionally expose as experimental/versioned; keep WCAG 2.x as the default compliance helper. |
| Gamut mapping default | Preserve out-of-gamut coordinates by default for conversion; require explicit clipping/mapping for bounded outputs such as hex or legacy RGB. |
| Chromatic adaptation default | Choose a documented default per route, likely Bradford for common D50/D65 Lab workflows, and expose options. |
| CMYK default | Keep generic CMYK formula separate from profile-backed CMYK. Never imply print accuracy without a profile. |
| Proprietary named colors | Support user-provided catalogs; avoid shipping restricted datasets. |

## Source Index

### Standards And Primary References

- [CSS Color Module Level 4](https://www.w3.org/TR/css-color-4/)
- [CSS Color Module Level 5](https://www.w3.org/TR/css-color-5/)
- [WCAG 2.2](https://www.w3.org/TR/WCAG22/)
- [Design Tokens Community Group Format Module](https://design-tokens.github.io/community-group/format/)
- [ICC specifications](https://www.color.org/specification/)
- [ICC.1:2022 profile specification](https://www.color.org/specification/ICC.1-2022-05.pdf)
- [ITU-R BT.709](https://www.itu.int/rec/R-REC-BT.709)
- [ITU-R BT.2020](https://www.itu.int/rec/R-REC-BT.2020)
- [ITU-R BT.2100](https://www.itu.int/rec/R-REC-BT.2100)
- [ACES documentation](https://docs.acescentral.com/)
- [OpenColorIO](https://opencolorio.org/)
- [LittleCMS](https://www.littlecms.com/)
- [Web Platform Tests: CSS color](https://github.com/web-platform-tests/wpt/tree/master/css/css-color)

### Platform References

- [Android ColorSpace](https://developer.android.com/reference/android/graphics/ColorSpace)
- [Apple CGColorSpace](https://developer.apple.com/documentation/coregraphics/cgcolorspace)
- [Skia color spaces](https://skia.org/docs/user/api/skcolorspace_overview/)

### Library References

- [Color.js](https://colorjs.io/)
- [Culori](https://culorijs.org/)
- [chroma.js](https://gka.github.io/chroma.js/)
- [d3-color](https://d3js.org/d3-color)
- [color-convert](https://github.com/Qix-/color-convert)
- [Material Color Utilities](https://github.com/material-foundation/material-color-utilities)
- [Colour Science](https://www.colour-science.org/)
- [ColorAide](https://facelessuser.github.io/coloraide/)
- [colorspacious](https://colorspacious.readthedocs.io/)
- [python-colormath](https://python-colormath.readthedocs.io/)
- [Pillow ImageCms](https://pillow.readthedocs.io/en/stable/reference/ImageCms.html)
- [Rust palette](https://docs.rs/palette/latest/palette/)
- [csscolorparser](https://docs.rs/csscolorparser/latest/csscolorparser/)
- [go-colorful](https://github.com/lucasb-eyer/go-colorful)
- [R colorspace](https://colorspace.r-forge.r-project.org/)
- [farver](https://farver.data-imaginist.com/)
