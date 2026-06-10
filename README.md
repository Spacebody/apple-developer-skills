# Apple Developer Skills

A curated collection of [Claude Code](https://claude.com/claude-code) **Agent Skills** distilled from
[Apple's official developer documentation](https://developer.apple.com/documentation), focused on building
modern Apple-platform apps with Swift and SwiftUI.

Each skill is a self-contained `SKILL.md` that Claude can load on demand to write idiomatic, up-to-date code
(Swift 6 concurrency, the Observation framework, SwiftData, async `URLSession`, and more).

## Status

| Platform | Status |
| --- | --- |
| iOS | ✅ Available |
| Cross-platform (Swift, Foundation, SwiftData, networking) | ✅ Available |
| macOS | 🚧 Planned |
| watchOS | 🚧 Planned |

> This first release targets **modern iOS core** development. macOS and watchOS skill sets are on the roadmap.
> The `skills/shared/` skills already apply to every Apple platform, so adding macOS/watchOS only requires
> new platform-specific folders — the shared foundations are reused, not duplicated.

## Repository layout

```
apple-developer-skills/
├── skills/
│   ├── apple-dev-router/           # Router: maps a task to the right specialized skill(s)
│   ├── shared/                     # Cross-platform Apple frameworks (iOS, macOS, watchOS, tvOS)
│   │   ├── swift-concurrency/
│   │   ├── foundation-essentials/
│   │   ├── networking-urlsession/
│   │   ├── swiftdata-persistence/
│   │   └── observation-framework/
│   └── ios/                        # iOS app development
│       ├── swiftui-fundamentals/
│       ├── swiftui-state-and-data-flow/
│       ├── ios-app-architecture/
│       ├── combine-essentials/
│       ├── app-store-review/        # App Store guideline compliance review (code/config level)
│       └── apple-hig-review/        # Human Interface Guidelines UI review (code level)
├── LICENSE
└── README.md
```

Every leaf folder contains a `SKILL.md` with YAML frontmatter (`name`, `description`) and a progressive-disclosure body.

## Skills

### Router (`skills/apple-dev-router/`)

| Skill | What it covers |
| --- | --- |
| **apple-dev-router** | Entry point for broad/ambiguous Apple-dev tasks. Maps the task to the right specialized skill(s) so only the relevant ones load. Documents how skill loading works (progressive disclosure — descriptions are always in context, bodies load only on invocation, no duplicate loading). |

> **On "auto-loading" skills:** Claude Code already loads skills on demand — each skill's `description` is the
> trigger, the body loads only when invoked, and an already-loaded skill is never reloaded. You don't need a skill
> to load other skills (it can't anyway). `apple-dev-router` instead acts as a *router/index* for broad requests,
> pointing Claude at the right specialized skill(s) for the current step.

### Cross-platform (`skills/shared/`)

| Skill | What it covers |
| --- | --- |
| **swift-concurrency** | `async`/`await`, `Task`, `TaskGroup`, actors, `@MainActor`, `Sendable`, `AsyncSequence`/`AsyncStream`, Swift 6 data-race safety |
| **foundation-essentials** | `Codable`, `JSONEncoder`/`JSONDecoder`, dates & `FormatStyle`, `FileManager`, `URL`, `Measurement`, `UserDefaults` |
| **networking-urlsession** | async `URLSession`, `URLRequest`, status/error handling, uploads/downloads, decoding into `Codable` models |
| **swiftdata-persistence** | `@Model`, `@Query`, `ModelContainer`, `ModelContext`, `#Predicate`, relationships, migrations |
| **observation-framework** | `@Observable`, observation tracking, migrating off `ObservableObject`/`@Published`, SwiftUI integration |

### iOS (`skills/ios/`)

| Skill | What it covers |
| --- | --- |
| **swiftui-fundamentals** | Views, layout (stacks/grids/`Layout`), modifiers, lists, `NavigationStack`, controls, animations, Liquid Glass. *Building* UI — for *auditing* it, see `apple-hig-review` |
| **swiftui-state-and-data-flow** | `@State`, `@Binding`, `@Environment`, `@Bindable`, passing data, single source of truth |
| **ios-app-architecture** | `App`/`Scene` lifecycle, `@main`, MVVM with `@Observable`, dependency injection, `.task`/lifecycle hooks, App Intents |
| **combine-essentials** | Publishers, subscribers, operators, `@Published`, when to use Combine vs. `async`/`await` |
| **app-store-review** | Pre-submission **App Store Review Guideline** compliance detectable in code/config: Info.plist usage strings, `UIBackgroundModes`, private/deprecated API, account deletion, ATT, `PrivacyInfo.xcprivacy`, export compliance. Ships with `guidelines-full.txt` |
| **apple-hig-review** | **Human Interface Guidelines** UI audit in code: Dynamic Type, semantic colors/Dark Mode, SF Symbols, 44pt hit targets, navigation/components, feedback/empty states, accessibility, i18n. Severity-graded review checklist |

## Installation

Skills live in `~/.claude/skills/` (user scope) or `<project>/.claude/skills/` (project scope).
Copy the skill folders you want:

```bash
git clone https://github.com/Spacebody/apple-developer-skills.git
cd apple-developer-skills

# Install every skill at the user level
mkdir -p ~/.claude/skills
cp -R skills/apple-dev-router skills/shared/* skills/ios/* ~/.claude/skills/

# …or install a single skill
cp -R skills/shared/swift-concurrency ~/.claude/skills/
```

Claude automatically discovers any `SKILL.md` under those directories and loads it when a task matches its
`description`. You can also invoke one explicitly, e.g. `Use the swift-concurrency skill to refactor this code`.

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
