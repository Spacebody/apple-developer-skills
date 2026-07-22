---
name: apple-localization
description: Implement, migrate, audit, and test Apple-platform localization with Xcode String Catalogs, LocalizedStringResource, String(localized:), generated symbols, plural and device variations, locale-aware formatting, pseudolanguages, and right-to-left layout. Use for .xcstrings work, missing or unextractable UI text, localization API selection, translation tables and bundles, pluralization, language/region test coverage, or RTL implementation. Do not use for translation quality or terminology review, App Store metadata and screenshots, pure date/number formatting with no catalog work, or a general HIG/accessibility audit.
---

# Apple Localization

Keep translatable UI copy, user-generated content, and locale-formatted values distinct. Build and inspect every
relevant target before claiming extraction or coverage is complete.

## Boundaries

- Use `foundation-essentials` for a focused date, number, currency, measurement, duration, or list-formatting task.
- Use `apple-hig-review` or `macos-hig-review` for broader layout, icon mirroring, accessibility, and platform-
  convention review. This skill owns localization resources, APIs, extraction, and tests.
- Treat translation accuracy, tone, and terminology as language-review work. Pseudolanguages prove layout and
  plumbing, not native-language quality.

Read [references/source-map.md](references/source-map.md) for API details, Xcode workflows, and primary sources.

## Inventory before editing

1. Identify deployment targets, development language, supported language-region-script combinations, all
   `.xcstrings` catalogs/tables, owned bundles or packages, and test-plan localization settings.
2. Classify visible text as:
   - fixed translatable UI copy;
   - user or remote content that must remain verbatim;
   - a complete interpolated message;
   - a locale-formatted value;
   - a deferred localizable resource passed through an API;
   - plural or device-specific wording.
3. Search behavior-driving source and resources. Do not infer full coverage from the main app target if extensions,
   widgets, packages, or tests own additional strings.

## Choose the right API

| Need | Use |
| --- | --- |
| SwiftUI literal label | A localizable initializer such as `Text("Title")`, `Button("Continue")`, or `Label` |
| Immediate localized `String` outside SwiftUI | `String(localized:table:bundle:locale:comment:)` |
| Localized attributed text | `AttributedString(localized:)` |
| Defer resolution across a model/API/process or locale | `LocalizedStringResource`, then resolve at the boundary |
| Stable machine key distinct from source-language value | Key/default-value initializer plus translator comment |
| User-provided or remote text | Plain `String`; do not turn it into a localization key |
| Dates/numbers/currency/measurements/lists | Foundation `FormatStyle` / `.formatted(...)` |

A `String` variable passed to `Text` is displayed as content; do not assume it performs catalog lookup. For a
reusable UI component that accepts fixed product copy, prefer `LocalizedStringResource` over resolving a `String`
before the component knows its locale and bundle.

## String Catalog workflow

1. Add or select the catalog and table owned by the target or package. Supply the correct `bundle` (including
   `#bundle` where appropriate), table, default value, and concise translator comment.
2. Replace hand-built lookup or concatenation with extractable localization APIs. Keep a whole sentence in one
   resource so translators can reorder interpolations.
3. Build every relevant target so Xcode extracts source strings. Inspect source locations, placeholders, new/stale
   state, table ownership, and generated symbols rather than assuming the compiler found everything.
4. For new work, use String Catalog plural variations instead of manual singular/plural branches. Interpolate the
   typed count in one complete resource, then add every plural category the target language exposes.
5. Use device variations only when interaction wording genuinely differs, such as “Tap” versus “Click.” Do not use
   string variants to compensate for a layout that should adapt with Dynamic Type and flexible sizing.
6. Generated catalog symbols are optional. When enabled, keep generated typed functions for placeholder resources
   and static properties for non-parameterized resources consistent with the owning catalog.

## Locale formatting and RTL

- Format data semantically; do not embed hardcoded date order, decimal separators, currency symbols, unit
  conversions, or list conjunctions inside localized prose. Use `Decimal` for monetary values.
- Prefer leading/trailing, semantic alignment, standard SwiftUI/AppKit/UIKit controls, and environment layout
  direction. Do not globally force RTL or blindly mirror logos, media controls, maps, graphs, clocks, or physical-
  direction symbols.
- Keep locale and layout direction separate in previews/tests. A right-to-left language can reveal both string and
  geometry issues, while an RTL pseudolanguage can isolate layout behavior.

## Test coverage

1. Enable Xcode's nonlocalized-string diagnostics and run relevant pseudolanguages: double-length, accented,
   bounded, tall, RTL, and RTL with RTL strings.
2. Preview actual localizations with `.environment(\.locale, ...)`; set
   `.environment(\.layoutDirection, .rightToLeft)` for a deliberate RTL layout preview.
3. Exercise supported App Language and App Region combinations. Put representative combinations in test-plan
   configurations for repeatable CI coverage.
4. Verify every locale-relevant plural category, placeholder order/type, device wording, formatted values,
   truncation, resizing, mirroring, and package/extension bundle lookup.
5. Finish with simulator/device/Mac runs and native-speaker review for shipped translations.

## Guardrails

- Never localize secrets, identifiers, filenames, URLs, user content, or server-controlled prose as catalog keys.
- Never concatenate translated fragments or implement English-only plural logic.
- Do not delete stale catalog entries until confirming no other target, package, extension, or dynamic lookup owns
  them. Keep source data and translator work intact during migration.
- Do not claim full localization from a successful build; state which targets, languages, regions, plural forms,
  pseudolanguages, and runtime surfaces were actually exercised.

## Output

Report changed catalogs/tables, API migrations, extracted or unresolved strings, plural/device variants, bundles,
and exact test coverage. Separate code-verifiable defects from translation-quality items requiring a language
reviewer.
