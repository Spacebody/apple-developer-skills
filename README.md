# Apple Developer Skills

A curated collection of Apple developer skills distilled from
[Apple's official developer documentation](https://developer.apple.com/documentation), focused on building
modern Apple-platform apps with Swift and SwiftUI.

The repository ships two installable distributions:

- `claude/skills/`: Claude Code Agent Skills.
- `codex/skills/`: Codex Skills adapted for Codex's skill-loading model.

Each skill is a self-contained `SKILL.md` that the target agent can load on demand to write idiomatic,
up-to-date code (Swift 6 concurrency, the Observation framework, SwiftData, async `URLSession`, and more).

## Status

| Platform | Status |
| --- | --- |
| iOS | ✅ Available |
| Cross-platform (Swift, Foundation, SwiftData, networking) | ✅ Available |
| Keychain/passkeys and localization workflows (Codex) | ✅ Available |
| macOS HIG, Catalyst/iOS-on-Mac adaptation, and performance profiling (Codex) | ✅ Available |
| Repository-owned native macOS SwiftUI/AppKit implementation skills | 🚧 Planned |
| watchOS | 🚧 Planned |

The status table tracks skills maintained in this repository, not additional skills that may be installed by the
Codex environment.

> This release targets **modern iOS core** development and includes Codex-specific macOS review, adaptation, and
> performance workflows. Native macOS SwiftUI/AppKit implementation skills maintained by this repository and
> watchOS skill sets remain on the roadmap.
> The `claude/skills/shared/` skills already apply to every Apple platform, so adding macOS/watchOS only requires
> new platform-specific folders — the shared foundations are reused, not duplicated.

### macOS Pathway coverage

The coverage map below follows Apple's current [macOS Pathway](https://developer.apple.com/macos/get-started/)
without pretending that one HIG skill owns unrelated implementation work:

| Pathway area | Current owner |
| --- | --- |
| Native SwiftUI/AppKit UI, scenes/windows, menus, build/debug, tests, signing, packaging | Installed `build-macos-apps:*` skills; not vendored here |
| HIG, desktop conventions, accessibility, localization layout, Dock discoverability, Catalyst/iOS-on-Mac UX | `macos-hig-review` |
| Privacy declarations and App Store submission compliance | `app-store-review` |
| Keychain, Passkeys, and directly related Security/AuthenticationServices implementation | `apple-keychain-passkeys` |
| String Catalog localization workflow, plural/device variants, testing, and RTL | `apple-localization`; locale-aware value formatting remains in `foundation-essentials` |
| Mac Catalyst and iOS-on-Mac implementation beyond a UX audit | `macos-app-adaptation` |
| Instruments profiling, performance, energy, hangs, and memory analysis | `macos-performance-profiling` |

Keeping these areas separate prevents high-risk security guidance and evidence-driven performance work from
being reduced to checklist items inside a UI review.

## Repository layout

```
apple-developer-skills/
├── claude/
│   └── skills/
│       ├── apple-dev-router/           # Claude router: maps a task to specialized skills
│       ├── shared/                     # Cross-platform Apple frameworks
│       │   ├── swift-concurrency/
│       │   ├── foundation-essentials/
│       │   ├── networking-urlsession/
│       │   ├── swiftdata-persistence/
│       │   └── observation-framework/
│       └── ios/                        # iOS app development
│           ├── swiftui-fundamentals/
│           ├── swiftui-state-and-data-flow/
│           ├── ios-app-architecture/
│           ├── combine-essentials/
│           ├── app-store-review/
│           └── apple-hig-review/
├── codex/
│   └── skills/
│       ├── apple-dev-router/        # Codex router adapted for Codex skill trigger rules
│       ├── apple-keychain-passkeys/ # Keychain, LocalAuthentication, and passkey client flows
│       ├── apple-localization/      # String Catalogs, localization APIs, testing, and RTL
│       ├── swift-concurrency/
│       ├── foundation-essentials/
│       ├── networking-urlsession/
│       ├── swiftdata-persistence/
│       ├── observation-framework/
│       ├── swiftui-fundamentals/  # Includes references/ios-27.md for forward notes
│       ├── swiftui-state-and-data-flow/
│       ├── ios-app-architecture/
│       ├── combine-essentials/
│       ├── app-store-review/
│       ├── macos-app-adaptation/    # Mac Catalyst and iOS/iPadOS apps running on Mac
│       ├── macos-hig-review/       # Codex-only macOS HIG and desktop-convention review
│       ├── macos-performance-profiling/ # Instruments-based macOS performance diagnosis
│       └── apple-hig-review/
├── .claude/
│   └── settings.local.json          # Repository-local Claude permissions, not a skill distribution
├── LICENSE
└── README.md
```

Every leaf folder contains a `SKILL.md` with YAML frontmatter (`name`, `description`) and a progressive-disclosure body.
The hidden `.claude/` directory is project configuration only; installable Claude skills live exclusively under
`claude/skills/`.

## Claude and Codex isolation

The Claude and Codex distributions are intentionally separate directories, not symlinks:

- Install Claude Code skills from `claude/skills/` into `~/.claude/skills/`.
- Install Codex skills from `codex/skills/` into `~/.codex/skills/`.
- Keep the skill folder names the same inside each agent's own skill directory so prompts such as
  `swift-concurrency` work consistently in both tools.
- Do not install both distributions into the same target directory. They share skill names by design and would
  overwrite each other there.

The two distributions cover the same local Apple topics, but their instructions may intentionally diverge. The
Codex copy is adapted to coexist with installed `build-ios-apps:*` and `build-macos-apps:*` skills, so it delegates
platform-specific implementation, runtime debugging, Liquid Glass, view refactoring, App Intents, and macOS
work when a closer specialized skill is available. Claude source files remain independent and are not rewritten
when Codex routing changes.

## Skills

The shared catalog below describes topics present in both distributions. Claude keeps the source grouped by
platform under `claude/skills/`; Codex keeps install-ready flat folders under `codex/skills/`. Codex-only additions and
their narrower ownership boundaries are listed separately.

### Router (`claude/skills/apple-dev-router/`, `codex/skills/apple-dev-router/`)

| Skill | What it covers |
| --- | --- |
| **apple-dev-router** | Entry point for broad/ambiguous Apple-dev tasks. Maps the task to the right specialized skill(s) so only the relevant ones load. Documents how skill loading works (progressive disclosure — descriptions are always in context, bodies load only on invocation, no duplicate loading). |

> **On "auto-loading" skills:** Claude Code and Codex load skills on demand from their installed skill
> directories. Each skill's `description` is the trigger, and `apple-dev-router` acts as a router/index for broad
> requests by pointing the agent at the right specialized skill(s) for the current step.

### Cross-platform (`claude/skills/shared/`)

| Skill | What it covers |
| --- | --- |
| **swift-concurrency** | `async`/`await`, `Task`, `TaskGroup`, actors, `@MainActor`, `Sendable`, `AsyncSequence`/`AsyncStream`, Swift 6 data-race safety |
| **foundation-essentials** | `Codable`, `JSONEncoder`/`JSONDecoder`, dates & `FormatStyle`, `FileManager`, `URL`, `Measurement`, `UserDefaults` |
| **networking-urlsession** | async `URLSession`, `URLRequest`, status/error handling, uploads/downloads, decoding into `Codable` models |
| **swiftdata-persistence** | `@Model`, `@Query`, `ModelContainer`, `ModelContext`, `#Predicate`, relationships, migrations |
| **observation-framework** | `@Observable`, observation tracking, migrating off `ObservableObject`/`@Published`, SwiftUI integration |

### Additional cross-platform (`codex/skills/`)

| Skill | What it covers |
| --- | --- |
| **apple-keychain-passkeys** | Small-secret Keychain CRUD and protection, access groups/synchronization, LocalAuthentication-gated access, passkey registration/assertion, `webcredentials` AASA, and the client/server verification boundary. Excludes general cryptography, OAuth, Sign in with Apple, and policy-only review. |
| **apple-localization** | String Catalog creation/migration, `LocalizedStringResource`, `String(localized:)`, generated symbols, plural/device variants, package/table ownership, pseudolanguage tests, language/region coverage, and RTL implementation. |

### iOS (`claude/skills/ios/`)

| Skill | What it covers |
| --- | --- |
| **swiftui-fundamentals** | Views, layout (stacks/grids/`Layout`), modifiers, lists, `NavigationStack`, controls, animations, and stable Liquid Glass adoption principles. *Building* UI — for *auditing* it, see `apple-hig-review` |
| **swiftui-state-and-data-flow** | `@State`, `@Binding`, `@Environment`, `@Bindable`, passing data, single source of truth |
| **ios-app-architecture** | `App`/`Scene` lifecycle, `@main`, root dependency ownership and injection, `.task`/lifecycle hooks, feature organization |
| **combine-essentials** | Publishers, subscribers, operators, `@Published`, when to use Combine vs. `async`/`await` |
| **app-store-review** | Pre-submission **App Store Review Guideline** compliance detectable in code/config: Info.plist usage strings, `UIBackgroundModes`, private/deprecated API, account deletion, ATT, `PrivacyInfo.xcprivacy`, export compliance. Ships with `guidelines-full.txt` |
| **apple-hig-review** | **Human Interface Guidelines** UI audit in code: Dynamic Type, semantic colors/Dark Mode, SF Symbols, 44pt hit targets, navigation/components, feedback/empty states, accessibility, i18n. Severity-graded review checklist |

### macOS (`codex/skills/`)

| Skill | What it covers |
| --- | --- |
| **macos-hig-review** | Code-detectable macOS HIG audit for windows, menus and commands, keyboard access, toolbars, sidebars and inspectors, Settings, Dock shortcuts, modality, pointer/drag behavior, accessibility, localization, Mac Catalyst, and iOS apps running on Mac. Routes implementation to the matching `build-macos-apps:*` skill instead of duplicating it. |
| **macos-app-adaptation** | Chooses and implements Mac Catalyst versus an unmodified iOS/iPadOS app on Apple silicon; covers compile/runtime detection, capability fallbacks, resizable scenes, menus, shortcuts, pointer/touch alternatives, Catalyst titlebars/toolbars, testing, and distribution boundaries. |
| **macos-performance-profiling** | Evidence-first Xcode/Instruments workflow for launch, CPU, hangs, concurrency, memory/leaks, disk I/O, energy impact, SwiftUI hitches, MetricKit, and repeatable before/after verification. |

### Codex-specific ownership

The `build-ios-apps:*` and `build-macos-apps:*` skills referenced here are specialized skills supplied by the
Codex environment; this repository does not copy or vendor them. Those handoffs apply only when the matching
skill is installed. The repository's own cross-platform and foundational skills remain usable on their own.

- `apple-dev-router` triggers only for broad, ambiguous, or cross-cutting Apple tasks. A narrowly scoped request
  should use its matching specialized skill directly.
- `swiftui-fundamentals` is the stable fundamentals/fallback reference. Production component patterns, view
  refactoring, Liquid Glass implementation, performance, and Simulator debugging route to the matching
  `build-ios-apps:*` skill when installed.
- `ios-app-architecture` owns app/scene lifecycle, root dependency ownership, and feature boundaries. It preserves
  the project's existing architecture and does not introduce MVVM or a view model by default. App Intents route to
  `build-ios-apps:ios-app-intents` when installed.
- `apple-hig-review` triggers for explicit code-detectable HIG/compliance audits, not generic visual polish or UX
  research. Optional independent review uses the current Codex runtime's built-in read-only collaboration rather
  than starting a nested `codex exec` session.
- `macos-hig-review` is the corresponding Mac-specific audit. It uses desktop measurements and conventions rather
  than applying iOS's 44 pt control rule, and includes Mac Catalyst and iOS-on-Mac adaptation checks.
- `macos-app-adaptation` owns Catalyst and iOS-on-Mac implementation; `macos-hig-review` owns the resulting UX audit,
  while installed `build-macos-apps:*` skills continue to own native AppKit/SwiftUI implementation.
- `macos-performance-profiling` requires a reproducible runtime symptom and trace evidence. It does not replace
  build/debug, test triage, telemetry, or code-only SwiftUI performance skills.
- `apple-keychain-passkeys` owns small-secret storage and platform-passkey client flows, not general cryptography,
  OAuth, signing, App Store policy, or server-side WebAuthn.
- `apple-localization` owns catalogs, extraction, plural/device variants, RTL implementation, and localization test
  coverage; `foundation-essentials` keeps focused locale-aware value formatting.
- `codex/skills/swiftui-fundamentals/references/ios-27.md` keeps Apple-published Xcode 27 changes separate from unverified
  forward-looking watch items. Load it only for iOS 27, the 2027 OS generation, or Xcode 27 work.

## Installation

### Claude Code

Skills live in `~/.claude/skills/` (user scope) or `<project>/.claude/skills/` (project scope).
Copy the skill folders you want:

```bash
git clone https://github.com/Spacebody/apple-developer-skills.git
cd apple-developer-skills

# Install every skill at the user level
mkdir -p ~/.claude/skills
cp -R claude/skills/apple-dev-router claude/skills/shared/* claude/skills/ios/* ~/.claude/skills/

# …or install a single skill
cp -R claude/skills/shared/swift-concurrency ~/.claude/skills/
```

Claude automatically discovers any `SKILL.md` under those directories and loads it when a task matches its
`description`. You can also invoke one explicitly, e.g. `Use the swift-concurrency skill to refactor this code`.

### Codex

Codex user skills live in `~/.codex/skills/`. Copy the Codex distribution from `codex/skills/`:

```bash
git clone https://github.com/Spacebody/apple-developer-skills.git
cd apple-developer-skills

# Install every Codex skill at the user level
mkdir -p ~/.codex/skills
cp -R codex/skills/* ~/.codex/skills/

# …or install a single Codex skill
cp -R codex/skills/swift-concurrency ~/.codex/skills/
```

Codex discovers installed `SKILL.md` files under `~/.codex/skills/` and applies the matching skill instructions
when a task triggers the skill. If you also use Claude Code on the same machine, keep the Claude and Codex install
commands separate so the two clients can evolve independently.

## Contributing

Contributions that extend coverage (macOS, watchOS, tvOS, additional frameworks) or keep existing skills
current with the latest Apple releases are welcome. Please keep each `SKILL.md`:

- **Accurate** — grounded in the official Apple documentation.
- **Concise** — load only what's needed; push deep detail into reference files.
- **Idiomatic** — modern Swift (Swift 6, `async`/`await`, Observation, SwiftData) over legacy patterns.

## License

[MIT](LICENSE). Skill content is original prose distilled from publicly available Apple documentation;
"Apple", "iOS", "macOS", "watchOS", "Swift", and "SwiftUI" are trademarks of Apple Inc. This project is not
affiliated with or endorsed by Apple Inc.
