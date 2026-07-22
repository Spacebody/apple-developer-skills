---
name: apple-dev-router
description: >-
  Entry point and router for Apple-platform development with this skill
  collection. Use at the START of any iOS/macOS/watchOS app task, when a request
  is broad or spans several areas (e.g. "build an app", "add a feature", "set up
  the project"), or whenever it's unclear which specific skill applies. Maps the
  task to the right specialized skill(s) — swift-concurrency, foundation-essentials,
  networking-urlsession, swiftdata-persistence, observation-framework,
  swiftui-fundamentals, swiftui-state-and-data-flow, ios-app-architecture,
  combine-essentials, app-store-review, apple-hig-review — so only the relevant
  ones are loaded, not all of them.
---

# Apple Development Router

This is a **dispatcher**, not a content skill. It helps pick the *right* specialized skill for the task at hand
so you load only what's needed. The specialized skills carry the actual guidance and code.

> **How skill loading actually works (read this first).** Skills use progressive disclosure: only each skill's
> `name` + `description` sits in context at all times (cheap). A skill's full body is loaded **only when it's
> invoked**, and once loaded in a session it is **not** reloaded. So you never pay for "all skills at once," and
> there is no duplicate loading. This router's job is purely to *route* — to tell you which skill(s) to invoke
> next — not to load anything itself. A skill cannot force-load another skill; you invoke the chosen one via the
> Skill tool.

## How to use this router

1. Identify what the current step actually needs (UI? data? networking? concurrency? app setup?).
2. Look it up in the routing table below.
3. Invoke **only** the matching skill(s). Skip this router for narrowly-scoped requests — those trigger their
   specialized skill directly.
4. Don't re-invoke a skill already loaded this session.

## Routing table — task → skill

| If the task is about… | Use skill |
| --- | --- |
| Laying out screens, views, navigation, lists, controls, Liquid Glass styling, animations | `swiftui-fundamentals` |
| Which property wrapper to use (`@State`/`@Binding`/`@Bindable`/`@Environment`), parent↔child data flow, view not updating | `swiftui-state-and-data-flow` |
| Making a model class observable, `@Observable`, migrating off `ObservableObject`/`@Published` | `observation-framework` |
| App/Scene lifecycle, `@main`, MVVM structure, dependency injection, project layout, App Intents | `ios-app-architecture` |
| `async`/`await`, `Task`, actors, `@MainActor`, `Sendable`, data-race errors, off-main work | `swift-concurrency` |
| Local persistence, `@Model`, `@Query`, predicates, relationships, migrations | `swiftdata-persistence` |
| Calling a REST/HTTP API, `URLSession`, decoding JSON responses, uploads/downloads | `networking-urlsession` |
| `Codable`, date/number/currency formatting, files, `UserDefaults`, units | `foundation-essentials` |
| Existing Combine code, debounced/throttled pipelines, merging event streams | `combine-essentials` |
| Pre-submission App Store compliance: Info.plist usage strings, background modes, private API, account deletion, ATT, privacy manifest ("会不会被拒"/"审核合规") | `app-store-review` |
| UI review against Apple HIG in code: Dynamic Type, semantic colors/Dark Mode, 44pt hit targets, standard navigation/components, accessibility ("HIG 审查"/"界面是否符合苹果规范") | `apple-hig-review` |

## Common multi-skill workflows

Most real features touch a few skills. Typical bundles — load these together, not the whole collection:

- **New app from scratch** → `ios-app-architecture` → `swiftui-fundamentals` → `swiftui-state-and-data-flow`
  (+ `observation-framework` for the view models).
- **Screen backed by a remote API** → `networking-urlsession` + `foundation-essentials` (Codable) +
  `swift-concurrency` + `swiftui-fundamentals`.
- **Screen backed by local storage** → `swiftdata-persistence` + `swiftui-fundamentals`
  (+ `observation-framework` for non-`@Query` state).
- **Fixing concurrency/data-race build errors** → `swift-concurrency` (alone is usually enough).
- **Reactive search / live-updating input** → `combine-essentials` (+ `swiftui-state-and-data-flow`).
- **Pre-submission / release review** → `app-store-review` (guideline compliance in code/config) +
  `apple-hig-review` (UI against HIG). Run both before submitting to App Store.

## Disambiguation hints

- "My view doesn't update" → start with `swiftui-state-and-data-flow`; if it's a model class, also
  `observation-framework`.
- "Where do I put this logic?" → `ios-app-architecture`.
- "Sendable / actor-isolated / main actor" compiler errors → `swift-concurrency`.
- "Parse/format this" (JSON, dates, currency) → `foundation-essentials`; if it comes from the network,
  `networking-urlsession` too.

## When NOT to use this router

Skip it when the request already points at one area (e.g. "decode this JSON", "fix this data race",
"add a NavigationStack"). In those cases the specialized skill triggers on its own description — going through the
router just adds a hop.

## Versions

This collection targets **iOS 26** (current shipping OS) with forward notes for **iOS 27** (developer beta,
public release fall 2026). The `shared/` skills apply to macOS and watchOS too; platform-specific skills for
those are planned.
