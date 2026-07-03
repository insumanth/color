# Color

**Color** is a planned, cross-language color conversion library for parsing, converting, comparing, formatting, and reasoning about color with a consistent API across popular programming languages.

The long-term goal is to build a definitive color library with broad format support, strong color-science foundations, shared conformance tests, and first-class packages for major language ecosystems.

> Project status: early design / pre-release. No runtime package is published yet, and no public API should be considered stable.

---

## Project Vision

Color conversion is often treated as a small utility problem, but production-grade color handling touches many areas:

- Web color syntax and modern CSS color functions.
- Wide-gamut display color spaces.
- Perceptual color models.
- Print and device color workflows.
- HDR and video color systems.
- Accessibility contrast checks.
- Color difference metrics.
- Gamut handling.
- Cross-platform consistency.

This project aims to provide one reliable foundation for those needs across Python, JavaScript / TypeScript, Ruby, Rust, and other popular languages.

---

## Goals

### Broad Format Coverage

Support the color formats developers encounter across the web, design tools, graphics systems, print workflows, and platform APIs.

### Correct Color Science

Conversions should be based on documented formulas, explicit white points, transfer functions, chromatic adaptation methods, and published numerical tolerances.

### Cross-Language Consistency

Each language package should feel native while matching the same behavior, fixtures, edge cases, and numerical expectations.

### Clean API Design

The final API should be easy for common use cases, but flexible enough for advanced users who need control over precision, gamut mapping, adaptation, serialization, and error handling.

### Explicit Failure Modes

Invalid syntax, unsupported color spaces, out-of-gamut conversions, unknown profiles, and lossy operations should be surfaced deliberately instead of being hidden behind silent guesses.

### Long-Term Maintainability

The project should be built around shared specifications, reusable fixtures, automated conformance tests, and carefully versioned package releases.

---

## Planned Language Packages

The project is intended to ship as a coordinated family of packages.

| Language | Package Manager | Package Name | Status |
| --- | --- | --- | --- |
| Python | PyPI | To be finalized | Planned |
| JavaScript / TypeScript | npm | To be finalized | First implementation / reference |
| Ruby | RubyGems | To be finalized | Planned |
| Rust | crates.io | To be finalized | Planned |
| Go | Go modules | To be finalized | Planned |
| Swift | Swift Package Manager | To be finalized | Planned |
| Kotlin / JVM | Maven Central | To be finalized | Planned |
| PHP | Packagist | To be finalized | Planned |
| .NET | NuGet | To be finalized | Planned |

Package names will be chosen after checking availability, ecosystem conventions, and naming conflicts in each package registry.

The JavaScript / TypeScript npm package will be the first library developed. It will serve as the reference implementation for behavior, fixtures, API decisions, and release practices that all other language packages depend on.

---

## Planned Library Capabilities

### Parsing

The library should parse common color inputs from CSS, design systems, platform APIs, and structured data formats.

Planned parsing areas include:

- CSS named colors.
- Hex color notation.
- RGB and alpha RGB notation.
- HSL and alpha HSL notation.
- HWB notation.
- CSS Color Level 4 and later syntax.
- Lab and LCh notation.
- OKLab and OKLCh notation.
- Device and profile-backed color declarations.
- Structured object and array inputs.
- Design token color values.

### Conversion

The library should convert between common, modern, and specialist color spaces with documented behavior.

Planned conversion areas include:

- sRGB and linear sRGB.
- Display P3.
- Rec. 2020 / BT.2020.
- Rec. 709 / BT.709.
- Adobe RGB.
- ProPhoto RGB.
- ACES-oriented RGB spaces.
- CIE XYZ.
- CIE xyY.
- CIE Lab.
- CIE LCh.
- CIE Luv.
- OKLab.
- OKLCh.
- Jzazbz.
- ICtCp.
- CAM16 and CAM16-UCS.
- HSL, HSV, HSB, HWB, and HSI.
- CMY and CMYK.
- Device gray.
- YCbCr, YUV, YIQ, and YPbPr families.
- HDR-oriented color spaces and transfer functions.

### Formatting

The library should serialize colors into practical output formats for browsers, design tools, native platforms, data files, and rendering systems.

Planned formatting areas include:

- CSS strings.
- Hex strings.
- RGB-family strings.
- HSL-family strings.
- Lab, LCh, OKLab, and OKLCh strings.
- JSON-compatible objects.
- Numeric arrays.
- Design token values.
- Platform-friendly color representations.
- ICC/profile-aware representations where supported.

### Color Analysis

Beyond conversion, the library should support analysis operations that are common in design systems, visualization, accessibility, and graphics workflows.

Planned analysis areas include:

- Color difference metrics.
- Relative luminance.
- Accessibility contrast.
- Perceptual contrast.
- Gamut checks.
- Channel inspection.
- Color equality within tolerance.
- Nearest named color.
- Temperature and chromaticity utilities.

### Color Operations

The library should support predictable color manipulation while making the chosen color space explicit.

Planned operation areas include:

- Mixing.
- Interpolation.
- Palette generation.
- Scale generation.
- Lightness adjustment.
- Chroma adjustment.
- Hue rotation.
- Saturation adjustment.
- Alpha composition.
- Gamut clipping.
- Perceptual gamut mapping.

---

## Planned Color Families

### Web and CSS

| Family | Target Support |
| --- | --- |
| Named colors | Planned |
| Hex notation | Planned |
| RGB / RGBA | Planned |
| HSL / HSLA | Planned |
| HWB | Planned |
| CSS color function | Planned |
| CSS Lab / LCh | Planned |
| CSS OKLab / OKLCh | Planned |
| Relative color syntax | Planned |

### RGB and Display Spaces

| Space | Target Support |
| --- | --- |
| sRGB | Planned |
| Linear sRGB | Planned |
| Display P3 | Planned |
| Rec. 2020 / BT.2020 | Planned |
| Rec. 709 / BT.709 | Planned |
| Adobe RGB | Planned |
| ProPhoto RGB | Planned |
| A98 RGB | Planned |
| ACEScg | Planned |
| ACEScc / ACEScct | Research |

### CIE and Perceptual Spaces

| Space | Target Support |
| --- | --- |
| CIE XYZ | Planned |
| CIE xyY | Planned |
| CIE Lab | Planned |
| CIE LCh | Planned |
| CIE Luv | Planned |
| CIE LChuv | Planned |
| OKLab | Planned |
| OKLCh | Planned |
| Jzazbz | Research |
| ICtCp | Research |
| CAM16 | Research |
| CAM16-UCS | Research |

### Artist-Friendly Models

| Model | Target Support |
| --- | --- |
| HSL | Planned |
| HSV / HSB | Planned |
| HWB | Planned |
| HSI | Planned |
| HCL-style models | Research |
| Munsell | Research |
| NCS | Research |

### Print, Device, and Profile Spaces

| Space | Target Support |
| --- | --- |
| CMY | Planned |
| CMYK | Planned |
| DeviceGray | Planned |
| ICC profile workflows | Planned |
| Spot color workflows | Research |

### Video and HDR

| Area | Target Support |
| --- | --- |
| YCbCr | Planned |
| YUV | Planned |
| YIQ | Planned |
| YPbPr | Planned |
| PQ transfer function | Planned |
| HLG transfer function | Planned |
| Rec. 2100 workflows | Research |
| Full-range and limited-range video values | Planned |

---

## Quality Standards

### Accuracy

Every supported conversion should have documented formulas, references, fixtures, and expected tolerances.

### Determinism

The same input should produce equivalent output across all officially supported languages, subject only to documented floating-point tolerances.

### Compatibility

The library should follow relevant specifications where practical, including modern CSS color behavior, common color-science conventions, and platform expectations.

### Transparency

Color-space metadata should be explicit: white point, transfer function, matrix transforms, coordinate ranges, and adaptation behavior should not be hidden.

### Stability

Stable releases should avoid unnecessary breaking changes. Behavior changes that affect numerical output should be clearly documented.

---

## Conformance Strategy

The project should use shared fixtures and compliance tests so every implementation can be verified against the same expectations.

Planned fixture categories include:

- Parsing fixtures.
- Formatting fixtures.
- Conversion fixtures.
- Gamut fixtures.
- Interpolation fixtures.
- Color difference fixtures.
- Contrast fixtures.
- Error handling fixtures.
- Cross-language compatibility fixtures.

Each fixture should define:

- Input color.
- Source color space.
- Target color space or operation.
- Expected output.
- Allowed tolerance.
- Relevant options.
- Reference notes.

---

## Repository Direction

The repository is expected to evolve into a monorepo containing the shared specification, test fixtures, documentation, and language implementations.

Planned areas include:

- Core color-space specifications.
- Transfer function definitions.
- Chromatic adaptation definitions.
- Named color data.
- CSS parsing rules.
- Shared conformance fixtures.
- Language-specific packages.
- Documentation.
- Release automation.
- Package publishing workflows.

---

## Roadmap

### Phase 1: Specification

- Define the first supported color spaces.
- Define conversion formulas and references.
- Define shared fixture format.
- Define public API principles.
- Define package naming strategy.

### Phase 2: Core Implementation

- Implement the JavaScript / TypeScript npm package as the first production-grade language package.
- Add RGB, XYZ, Lab, LCh, OKLab, OKLCh, HSL, HSV, HWB, and CMYK.
- Add CSS parsing and formatting.
- Add conformance tests.
- Add documentation for accuracy and limitations.

### Phase 3: Multi-Language Support

- Port the core behavior from the JavaScript / TypeScript reference implementation to additional languages.
- Share fixtures across all implementations.
- Add package-specific documentation.
- Verify numerical consistency between languages.
- Prepare package publishing workflows.

### Phase 4: Package Releases

- Publish the first package manager releases.
- Add release notes and versioning policy.
- Add CI validation for all supported packages.
- Add compatibility badges and conformance status.

### Phase 5: Advanced Color Workflows

- Add ICC/profile support.
- Add HDR and video workflows.
- Add advanced perceptual color spaces.
- Add advanced gamut mapping.
- Add design-token integrations.
- Add command-line tooling.

---

## Versioning Plan

The project should use semantic versioning for each package.

| Version Range | Meaning |
| --- | --- |
| 0.x | Early development, unstable API, active design. |
| 1.x | Stable core API and tested conversion behavior. |
| 2.x and later | Breaking changes only when required for correctness, standards alignment, or major API improvements. |

The conformance fixture version should be tracked separately from package versions so implementations can state exactly which behavior set they support.

---

## Contribution Principles

Contributions should improve one or more of these areas:

- Accuracy.
- Format coverage.
- Language support.
- Documentation.
- Test coverage.
- Performance.
- API design.
- Package reliability.

New color-space or conversion contributions should include references, formulas, fixtures, edge cases, and tolerance notes.

---

## Non-Goals

Color should stay focused on color values, conversions, formatting, and analysis.

This project is not intended to be:

- A full image-processing library.
- A painting or drawing application.
- A complete design system.
- A replacement for professional color-management systems in every print workflow.
- A library that silently guesses ambiguous color meaning.

---

## License

MIT License. See [LICENSE](./LICENSE).

---

## Maintainer

Created and maintained by Sumanth.
