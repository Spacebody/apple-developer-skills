---
name: apple-dev-router
description: >-
  Route broad, ambiguous, or cross-cutting Apple-platform app tasks to the
  smallest useful set of installed Codex skills. Use when a request spans
  multiple areas or no specialized skill clearly owns it. Do not use for a
  narrowly scoped Swift, SwiftUI, build, debug, review, or platform task; invoke
  the matching specialized skill directly. Prefer build-ios-apps:* and
  build-macos-apps:* for platform-specific implementation and runtime work, and
  use the local Apple skills for cross-cutting Swift, Foundation, data, and
  review guidance.
---

# Apple Development Router

This is a **dispatcher**, not a content skill. It helps pick the *right* specialized skill for the task at hand
so you load only what's needed. The specialized skills carry the actual guidance and code.

> **How skill loading actually works (read this first).** Skills use progressive disclosure: only each skill's
> `name` + `description` sits in context at all times (cheap). A skill's full body is loaded **only when it's
> invoked**, and once loaded in a session it is **not** reloaded. So you never pay for "all skills at once," and
> there is no duplicate loading. This router's job is purely to *route* — to tell you which skill(s) to invoke
> next — not to load anything itself. A skill cannot force-load another skill; in Codex, select the matching
> skill by name and read that skill's `SKILL.md` according to the normal skill trigger rules.

## How to use this router

1. Follow repository and user instructions before any routing preference in this skill.
2. If one specialized skill clearly owns the request, stop routing and use that skill directly.
3. Otherwise identify the task's platform and independent concerns (UI, data, networking, concurrency, app setup, review).
4. Use the smallest set of matching skills. Do not load a broad local skill when an installed platform-specific skill is a closer match.

## Routing table — task → skill

| If the task is about… | Use skill |
| --- | --- |
| iOS screens, navigation, controls, or component composition | `build-ios-apps:swiftui-ui-patterns` (fallback: `swiftui-fundamentals`) |
| Splitting or restructuring a SwiftUI view, tightening data flow, Observation ownership | `build-ios-apps:swiftui-view-refactor` |
| iOS Liquid Glass implementation or review | `build-ios-apps:swiftui-liquid-glass` |
| iOS Simulator launch/debug, performance traces, or memory leaks | the matching `build-ios-apps:ios-*` or `build-ios-apps:swiftui-performance-audit` skill |
| App Intents, Siri, Shortcuts, Spotlight, widgets, or controls | `build-ios-apps:ios-app-intents` |
| macOS SwiftUI/AppKit UI, windows, build/debug, tests, signing, packaging, or notarization | the matching `build-macos-apps:*` skill |
| macOS HIG compliance, desktop conventions, menus/keyboard discoverability, or Mac accessibility audit | `macos-hig-review` |
| Implementing Mac Catalyst or adapting an iOS/iPadOS app to run on Mac | `macos-app-adaptation` |
| Auditing Mac Catalyst or an iOS app on Mac against desktop conventions | `macos-hig-review` |
| Profiling native macOS/Mac Catalyst launch, CPU, hangs, memory, disk, energy, or runtime regressions | `macos-performance-profiling` |
| Keychain storage, `SecItem`, biometric/passcode-gated secrets, passkeys, or `webcredentials` AASA | `apple-keychain-passkeys` |
| String Catalogs, `.xcstrings`, localization APIs, plural/device variants, pseudolanguages, or RTL implementation | `apple-localization` |
| Which property wrapper to use (`@State`/`@Binding`/`@Bindable`/`@Environment`), parent↔child data flow, view not updating | `swiftui-state-and-data-flow` |
| Making a model class observable, `@Observable`, migrating off `ObservableObject`/`@Published` | `observation-framework` |
| iOS App/Scene lifecycle, root dependency graph, feature boundaries, or project layout | `ios-app-architecture` |
| `async`/`await`, `Task`, actors, `@MainActor`, `Sendable`, data-race errors, off-main work | `swift-concurrency` |
| Local persistence, `@Model`, `@Query`, predicates, relationships, migrations | `swiftdata-persistence` |
| Calling a REST/HTTP API, `URLSession`, decoding JSON responses, uploads/downloads | `networking-urlsession` |
| `Codable`, date/number/currency formatting, files, `UserDefaults`, units | `foundation-essentials` |
| Existing Combine code, debounced/throttled pipelines, merging event streams | `combine-essentials` |
| Pre-submission App Store compliance: Info.plist usage strings, background modes, private API, account deletion, ATT, privacy manifest ("会不会被拒"/"审核合规") | `app-store-review` |
| iOS/iPadOS UI review against Apple HIG in code: Dynamic Type, semantic colors/Dark Mode, 44pt hit targets, standard navigation/components, accessibility | `apple-hig-review` |

## Common multi-skill workflows

Most real features touch a few skills. Typical bundles — load these together, not the whole collection:

- **New iOS app from scratch** → `ios-app-architecture` + `build-ios-apps:swiftui-ui-patterns`
  (+ `swiftui-state-and-data-flow` when state ownership needs design).
- **Screen backed by a remote API** → `networking-urlsession` + `foundation-essentials` (Codable) +
  `swift-concurrency` + `build-ios-apps:swiftui-ui-patterns`.
- **Screen backed by local storage** → `swiftdata-persistence` + `build-ios-apps:swiftui-ui-patterns`.
- **macOS feature** → choose the matching `build-macos-apps:*` UI/build/window skill, then add only the cross-cutting local Swift/data skill it actually needs.
- **New native macOS app** → `build-macos-apps:swiftui-patterns` + the smallest cross-cutting local Swift/data skill; add `macos-hig-review` for an explicit compliance pass.
- **Adapt an iPad app for Mac** → `macos-app-adaptation` for implementation + `macos-hig-review` for the desktop-experience audit.
- **Secure login with passkeys** → `apple-keychain-passkeys`; add `app-store-review` only for policy/privacy/submission questions.
- **Localize a feature** → `apple-localization` + `foundation-essentials` only when dates, numbers, currency, measurements, or lists need formatting.
- **Diagnose a slow Mac app** → `macos-performance-profiling`; add `build-macos-apps:telemetry` only when durable logs or signposts are needed.
- **Fixing concurrency/data-race build errors** → `swift-concurrency` (alone is usually enough).
- **Reactive search / live-updating input** → `combine-essentials` (+ `swiftui-state-and-data-flow`).
- **Pre-submission / release review** → `app-store-review` (guideline compliance in code/config) + the platform
  HIG skill: `apple-hig-review` for iOS/iPadOS or `macos-hig-review` for macOS.

## Disambiguation hints

- "My view doesn't update" → start with `swiftui-state-and-data-flow`; if it's a model class, also
  `observation-framework`.
- "Where do I put this logic?" → `ios-app-architecture`.
- "Does this follow Apple HIG?" → identify the target first: `apple-hig-review` for iOS/iPadOS,
  `macos-hig-review` for macOS or Mac Catalyst.
- "Store this token securely" or "add passkeys" → `apple-keychain-passkeys`; not `foundation-essentials`.
- "Translate/localize this UI" → `apple-localization`; a pure locale-formatting question remains `foundation-essentials`.
- "This app is slow on Mac" → collect runtime evidence with `macos-performance-profiling`; do not infer a fix from code alone.
- "Sendable / actor-isolated / main actor" compiler errors → `swift-concurrency`.
- "Parse/format this" (JSON, dates, currency) → `foundation-essentials`; if it comes from the network,
  `networking-urlsession` too.

## When NOT to use this router

Skip it when the request already points at one area (e.g. "decode this JSON", "fix this data race",
"add a NavigationStack"). In those cases the specialized skill triggers on its own description — going through the
router just adds a hop.

## Versions

The local collection targets **iOS 26** with forward notes for **iOS 27** and includes Codex-specific macOS HIG,
adaptation, performance, localization, and secure-credential workflows. Installed
`build-ios-apps:*` and `build-macos-apps:*` skills own platform-specific implementation,
runtime, build, and distribution workflows; the local cross-cutting Swift skills remain usable
across Apple platforms when their descriptions match.
