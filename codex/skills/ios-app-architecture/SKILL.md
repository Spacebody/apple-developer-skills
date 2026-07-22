---
name: ios-app-architecture
description: >-
  Structure the root of a SwiftUI iOS app: App/Scene lifecycle, WindowGroup,
  dependency ownership and injection, feature/module boundaries, lifecycle-driven
  work, and the placement of domain services and shared state. Use when starting
  an app, designing its root dependency graph, reorganizing feature boundaries,
  or deciding where app-level logic belongs. Preserve an existing project's
  architecture; do not use this as a generic SwiftUI view-refactoring skill and
  do not introduce MVVM or a view model by default.
---

# iOS App Architecture (SwiftUI)

A pragmatic, architecture-neutral structure for SwiftUI apps: preserve local conventions, keep view-local state
in the view, place domain behavior in services/models, own shared state at a stable root, and add a dedicated
screen model only when coordination complexity justifies it. Builds on `observation-framework`,
`swiftui-state-and-data-flow`, and `swift-concurrency`.

> **Boundary.** Use `build-ios-apps:swiftui-view-refactor` for view-file restructuring and
> `build-ios-apps:ios-app-intents` for Siri, Shortcuts, Spotlight, widgets, or controls.

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

## Place state and logic deliberately

Default to SwiftUI's MV style rather than adding a view model automatically:

- Keep transient presentation state in the owning view with `@State`.
- Put domain operations and reusable business rules in services or domain models.
- Own shared `@Observable` state once at the app or feature root and pass it explicitly or through the environment.
- Add a screen-specific `@MainActor @Observable` model only for substantial, long-lived coordination such as
  multiple async operations, derived screen state, or an existing MVVM convention. Do not create one merely to
  mirror local state or wrap environment dependencies.

When a dedicated screen model is justified, use this shape:

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
- The optional screen model coordinates UI-facing state and services; domain rules remain in domain types/services.
- UI-facing models stay `@MainActor` so state updates are safe.
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

Prefer explicit initializer injection for feature-local dependencies. Use the environment for shared cross-cutting
dependencies or state with clear root ownership (session, theme, analytics). If an existing project has a stronger
local convention, preserve it.

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
  TaskList/          TaskList.swift, TaskRow.swift, optional TaskListModel.swift
  TaskDetail/        …
Models/              Task.swift (domain + @Model)
Services/            TaskService.swift, LiveTaskService.swift
Shared/              extensions, reusable views, design system
```

For larger apps, split features into Swift Package modules to enforce boundaries and speed builds.

## System surfaces

Use `build-ios-apps:ios-app-intents` for App Intents and related Siri, Shortcuts, Spotlight, widget, and control
surfaces. Keep this skill focused on the app's root architecture and dependency boundaries.

## Pitfalls

- **Business logic in views** — move reusable domain behavior into a service or domain model; introduce a screen model only when coordination warrants it.
- **View models by reflex** — prefer local SwiftUI state and explicit dependencies until a dedicated coordination object has a clear responsibility.
- **UI-facing model not on `@MainActor`** — isolate UI state on the main actor; move heavy work to an actor or service.
- **Creating models in `body`** — own them with `@State`/environment so they persist across re-renders.
- **Singletons everywhere** — inject dependencies; it keeps code testable and previewable.
- **Using SiriKit for new voice features** — use App Intents.

## Quick reference

| Concern | Approach |
| --- | --- |
| App entry | `@main struct: App` + `WindowGroup` |
| App-wide model | `@State` in `App` + `.environment(_:)` |
| Foreground/background | `@Environment(\.scenePhase)` + `.onChange` |
| Local screen state | `@State`, `@Binding`, `@Environment`, `@Query` as appropriate |
| Domain behavior | Service or domain model |
| Complex screen coordination | Optional `@MainActor @Observable` model |
| Load on appear | `.task { await model.load() }` |
| Dependencies | protocol + constructor or `@Entry` environment key |
| Persistence | `.modelContainer` + SwiftData |
| Code layout | group by feature; modularize with SPM when large |
| Siri/Shortcuts | `build-ios-apps:ios-app-intents` |
