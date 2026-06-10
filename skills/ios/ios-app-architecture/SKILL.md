---
name: ios-app-architecture
description: >-
  Structure a SwiftUI iOS app — the App/Scene lifecycle with @main and
  WindowGroup, MVVM with @Observable view models on the @MainActor, dependency
  injection via the environment, loading data with .task and reacting to lifecycle
  with scenePhase/onChange, folder/module organization, and the App Intents
  surface for Siri/Shortcuts. Use when starting a new app, organizing code,
  wiring up view models and dependencies, or deciding where logic belongs on
  iOS 26+ (forward to iOS 27).
---

# iOS App Architecture (SwiftUI)

A pragmatic, modern structure for SwiftUI apps: thin views, `@Observable` models on the main actor, dependencies
injected through the environment, and async work driven by `.task`. Builds on `observation-framework`,
`swiftui-state-and-data-flow`, and `swift-concurrency`.

> **Versions.** iOS 26 baseline; forward to iOS 27. Note: **App Intents is now the required integration surface
> for Siri**, and SiriKit is deprecated — model user-facing actions as App Intents for Siri/Shortcuts/Spotlight.

## App entry point & lifecycle

```swift
@main
struct TasksApp: App {
    @State private var store = TaskStore()          // app-wide model, created once
    @Environment(\.scenePhase) private var scenePhase

    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(store)                  // inject into the whole tree
        }
        .modelContainer(for: Task.self)              // SwiftData store (if used)
        .onChange(of: scenePhase) { _, phase in
            if phase == .background { store.persist() }
        }
    }
}
```

- `App` describes the app; `Scene` (`WindowGroup`, `DocumentGroup`, `Settings`, `WindowGroup(for:)`) describes windows.
- Create app-wide models with `@State` here and inject via `.environment(_:)`.
- React to foreground/background via `@Environment(\.scenePhase)`.
- Use `.modelContainer` once at the top for SwiftData.

## MVVM with @Observable

Keep views declarative; put state + logic in an `@Observable` model pinned to the main actor.

```swift
@MainActor
@Observable
final class TaskListModel {
    private(set) var tasks: [Task] = []
    private(set) var isLoading = false
    var error: String?

    private let service: TaskService
    init(service: TaskService) { self.service = service }

    func load() async {
        isLoading = true; defer { isLoading = false }
        do { tasks = try await service.fetchTasks() }
        catch { self.error = error.localizedDescription }
    }

    func add(_ title: String) async { /* call service, update tasks */ }
}
```

```swift
struct TaskList: View {
    @State private var model: TaskListModel
    init(service: TaskService) { _model = State(initialValue: TaskListModel(service: service)) }

    var body: some View {
        List(model.tasks) { TaskRow(task: $0) }
            .overlay { if model.isLoading { ProgressView() } }
            .task { await model.load() }                 // runs on appear, cancels on disappear
            .alert("Error", isPresented: .constant(model.error != nil)) {
                Button("OK") { model.error = nil }
            } message: { Text(model.error ?? "") }
    }
}
```

Rules of thumb:
- The view reads model state and forwards user intent (`await model.add(...)`); it contains no business logic.
- The model owns state, talks to services, and stays `@MainActor` so UI updates are safe.
- Heavy/concurrent work hops to actors or task groups inside the model (see `swift-concurrency`).

## Dependency injection

Define services behind protocols so you can swap real/mock implementations:

```swift
protocol TaskService: Sendable {
    func fetchTasks() async throws -> [Task]
}

struct LiveTaskService: TaskService { /* URLSession-backed */ }
struct MockTaskService: TaskService { func fetchTasks() async -> [Task] { Task.samples } }
```

Inject either by constructor (as above) or through the environment for app-wide singletons:

```swift
extension EnvironmentValues {
    @Entry var taskService: any TaskService = LiveTaskService()   // @Entry macro defines a key
}
// Provide: .environment(\.taskService, MockTaskService())  (e.g. in previews/tests)
// Consume: @Environment(\.taskService) private var service
```

Prefer constructor injection for view-model dependencies; use the environment for cross-cutting singletons
(session, theme, analytics).

## Loading data & reacting to changes

- `.task { await model.load() }` — start async work tied to the view's lifetime (auto-cancels).
- `.task(id: selectedID) { … }` — re-run when `id` changes.
- `.refreshable { await model.reload() }` — pull-to-refresh.
- `.onChange(of:) { old, new in … }` — react to value changes.
- `.onScenePhaseChange` / `@Environment(\.scenePhase)` — app foreground/background.

## Project organization

Group by feature, not by type, once the app grows:

```
App/                 TasksApp.swift, RootView.swift
Features/
  TaskList/          TaskList.swift, TaskListModel.swift, TaskRow.swift
  TaskDetail/        …
Models/              Task.swift (domain + @Model)
Services/            TaskService.swift, LiveTaskService.swift
Shared/              extensions, reusable views, design system
```

For larger apps, split features into Swift Package modules to enforce boundaries and speed builds.

## App Intents (Siri / Shortcuts / Spotlight)

Expose key actions as App Intents (replaces SiriKit). This makes them available to Siri, Shortcuts, Spotlight,
and widgets:

```swift
struct AddTaskIntent: AppIntent {
    static let title: LocalizedStringResource = "Add Task"
    @Parameter(title: "Title") var taskTitle: String
    func perform() async throws -> some IntentResult {
        await TaskStore.shared.add(taskTitle)
        return .result()
    }
}
```

## Pitfalls

- **Business logic in views** — move it into the `@Observable` model.
- **Model not on `@MainActor`** — UI-facing models should be; do off-main work explicitly inside.
- **Creating models in `body`** — own them with `@State`/environment so they persist across re-renders.
- **Singletons everywhere** — inject dependencies; it keeps code testable and previewable.
- **Using SiriKit for new voice features** — use App Intents.

## Quick reference

| Concern | Approach |
| --- | --- |
| App entry | `@main struct: App` + `WindowGroup` |
| App-wide model | `@State` in `App` + `.environment(_:)` |
| Foreground/background | `@Environment(\.scenePhase)` + `.onChange` |
| Screen logic | `@MainActor @Observable` view model |
| Load on appear | `.task { await model.load() }` |
| Dependencies | protocol + constructor or `@Entry` environment key |
| Persistence | `.modelContainer` + SwiftData |
| Code layout | group by feature; modularize with SPM when large |
| Siri/Shortcuts | App Intents (`AppIntent`) |
