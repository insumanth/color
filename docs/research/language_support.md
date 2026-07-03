# Language Support Research

Researched: 2026-07-03

## Recommendation

Build one shared specification and conformance fixture suite, then publish native-feeling packages in the ecosystems below. Prioritize languages that combine developer usage, open-source activity, color-domain fit, and a mature package repository.

## Priority Targets

| Tier | Language target | Library repository | Why it belongs |
| --- | --- | --- | --- |
| 1 | JavaScript / TypeScript | [npm](https://www.npmjs.com/) | Highest Stack Overflow professional usage: JavaScript 68.8%, TypeScript 48.8%; top GitHub repository creation; primary CSS/web color audience. |
| 1 | Python | [PyPI](https://pypi.org/) | Stack Overflow professional usage 54.8%; TIOBE #1; GitHub's most-used language in 2024; strong data, design, and scientific workflows. |
| 1 | JVM: Java + Kotlin | [Maven Central](https://central.sonatype.com/) | Java remains top-tier across Stack Overflow, TIOBE, and GitHub; Kotlin adds Android/UI reach from the same ecosystem. |
| 1 | .NET: C# first | [NuGet](https://www.nuget.org/) | C# has 29.9% professional usage and TIOBE #5; one package can serve C#, F#, and Visual Basic .NET users. |
| 1 | Rust | [crates.io](https://crates.io/) | Strong correctness/performance fit; TIOBE #12; GitHub and Stack Overflow show rising adoption. |
| 1 | Go | [Go modules / pkg.go.dev](https://pkg.go.dev/) | 17.4% professional usage; TIOBE #13; useful for CLIs, services, and infrastructure tooling. |
| 1 | PHP | [Packagist](https://packagist.org/) | 19.1% professional usage; still important for web and CMS ecosystems where color utilities are common. |
| 1 | Swift | [Swift Package Manager / Swift Package Index](https://swiftpackageindex.com/) | Apple UI, graphics, and display workflows make color support strategically important. |
| 1 | Ruby | [RubyGems](https://rubygems.org/) | Smaller but still relevant web/design ecosystem; already named in the project README. |
| 2 | C / C++ | [Conan Center](https://conan.io/center/) + [vcpkg](https://vcpkg.io/) | C and C++ rank #2/#3 in TIOBE; useful for native graphics, bindings, and a possible shared core. |
| 2 | Dart / Flutter | [pub.dev](https://pub.dev/) | Mobile/UI-heavy ecosystem; GitHub lists Dart among 2024's fastest-growing languages. |
| 2 | R | [CRAN](https://cran.r-project.org/) | TIOBE #9; strong fit for visualization, palettes, scales, and scientific color workflows. |
| 2 | Julia | [General registry](https://github.com/JuliaRegistries/General) | Scientific-computing fit for color science, despite smaller general usage. |

## Release Order

1. Start with JavaScript/TypeScript as the first implementation because it is the primary CSS/web color audience and can ship quickly to npm.
2. Add Python next after the fixture suite is stable enough to exercise both CSS and data/scientific workflows.
3. Add Rust as a native port or performance-oriented engine only when the shared contract is mature enough to justify the extra implementation cost.
4. Add JVM, .NET, Go, PHP, Swift, and Ruby after the fixture suite is stable.
5. Add Dart, R, Julia, and C/C++ when Tier 1 packages are passing the same conformance tests or when maintainers appear.

## Correctness Oracle Strategy

Do not rely on one internal implementation as the source of truth. Use authoritative specs plus trusted external libraries to generate language-neutral fixtures, then require every package to pass the same fixtures.

| Conversion area | Primary oracle | Language | Role |
| --- | --- | --- | --- |
| CSS Color 4/5 parsing, formatting, Lab/LCh, OKLab/OKLCh, Display P3, Rec.2020 | [Color.js](https://colorjs.io/) | JavaScript | Strong CSS-focused oracle; maintained by CSS Color spec editors. |
| Modern CSS plus broad color-space coverage | [ColorAide](https://facelessuser.github.io/coloraide/) | Python | Independent CSS/color-space cross-check. |
| CIE XYZ, xyY, Lab, Luv, chromatic adaptation, CAM16, Jzazbz, ICtCp, HDR transfer functions | [Colour Science](https://www.colour-science.org/) | Python | Primary scientific-color oracle. |
| ICC profiles, profile-backed RGB/Gray/Lab/CMYK, real device conversion | [LittleCMS](https://www.littlecms.com/) | C | Primary color-management engine oracle. |
| YCbCr, YUV, YIQ, YPbPr, video matrix/range behavior | [Colour Science](https://www.colour-science.org/) + [libyuv](https://chromium.googlesource.com/libyuv/libyuv/) | Python / C++ | Scientific and practical video cross-checks. |
| ACES and VFX transforms | [ACES documentation](https://docs.acescentral.com/) + [OpenColorIO](https://opencolorio.org/) | C++ / Python | VFX pipeline oracle. |
| Material HCT, dynamic color, design palettes | [Material Color Utilities](https://github.com/material-foundation/material-color-utilities) | Multi-language | Authoritative for Material behavior only. |
| WCAG relative luminance and contrast | [WCAG 2.2](https://www.w3.org/TR/WCAG22/) | Spec formula | Implement directly and fixture from known values. |

Fixture flow:

```text
external oracle libraries
  -> fixture generator
  -> fixtures/v1/
  -> all language package tests
```

TypeScript should be the initial reference package because it is the first implementation. Rust can become a performance-oriented implementation later, but external specs and oracle libraries should remain the audit source.

## CLI

Defer the CLI until the TypeScript package and shared fixtures are stable. If a Rust implementation later exists, it is the best candidate for cross-platform CLI binaries; otherwise a Node-based CLI can cover early npm users.

## Defer For Now

Do not create first-class packages yet for SQL, HTML/CSS, shell, or PowerShell; support those users through docs, CLI output, or generated values instead. Scala, Groovy, F#, and Visual Basic .NET should be covered by the JVM or .NET packages first. Lua, Perl, Elixir, Haskell, Zig, MATLAB, Delphi, Ada, and Fortran are better handled later if community demand or maintainers appear.

## Sources

- [Stack Overflow Developer Survey 2025](https://survey.stackoverflow.co/2025/technology)
- [TIOBE Index for June 2026](https://www.tiobe.com/tiobe-index/)
- [GitHub Octoverse 2024](https://github.blog/news-insights/octoverse/octoverse-2024/)
