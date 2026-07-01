# Apple Developer Skills

A curated collection of Apple developer skills distilled from
[Apple's official developer documentation](https://developer.apple.com/documentation), focused on building
modern Apple-platform apps with Swift and SwiftUI.

The repository ships two installable distributions:

- `skills/`: Claude Code Agent Skills.
- `codex/skills/`: Codex Skills adapted for Codex's skill-loading model.

Each skill is a self-contained `SKILL.md` that the target agent can load on demand to write idiomatic,
up-to-date code (Swift 6 concurrency, the Observation framework, SwiftData, async `URLSession`, and more).

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
│   ├── apple-dev-router/           # Claude router: maps a task to the right specialized skill(s)
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
├── codex/
│   └── skills/
│       ├── apple-dev-router/        # Codex router adapted for Codex skill trigger rules
│       ├── swift-concurrency/
│       ├── foundation-essentials/
│       ├── networking-urlsession/
│       ├── swiftdata-persistence/
│       ├── observation-framework/
│       ├── swiftui-fundamentals/
│       ├── swiftui-state-and-data-flow/
│       ├── ios-app-architecture/
│       ├── combine-essentials/
│       ├── app-store-review/
│       └── apple-hig-review/
├── LICENSE
└── README.md
```

Every leaf folder contains a `SKILL.md` with YAML frontmatter (`name`, `description`) and a progressive-disclosure body.

## Claude and Codex isolation

The Claude and Codex distributions are intentionally separate directories, not symlinks:

- Install Claude Code skills from `skills/` into `~/.claude/skills/`.
- Install Codex skills from `codex/skills/` into `~/.codex/skills/`.
- Keep the skill folder names the same inside each agent's own skill directory so prompts such as
  `swift-concurrency` work consistently in both tools.
- Do not install both distributions into the same target directory. They share skill names by design and would
  overwrite each other there.

Most skill bodies are identical across the two distributions. The Codex copy may contain small agent-specific
wording changes where Claude Code concepts such as the Skill tool do not apply.

## Skills

The catalog below applies to both distributions. Claude keeps the source grouped by platform under `skills/`;
Codex keeps install-ready flat folders under `codex/skills/`.

### Router (`skills/apple-dev-router/`, `codex/skills/apple-dev-router/`)

| Skill | What it covers |
| --- | --- |
| **apple-dev-router** | Entry point for broad/ambiguous Apple-dev tasks. Maps the task to the right specialized skill(s) so only the relevant ones load. Documents how skill loading works (progressive disclosure — descriptions are always in context, bodies load only on invocation, no duplicate loading). |

> **On "auto-loading" skills:** Claude Code and Codex load skills on demand from their installed skill
> directories. Each skill's `description` is the trigger, and `apple-dev-router` acts as a router/index for broad
> requests by pointing the agent at the right specialized skill(s) for the current step.

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

### Claude Code

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
