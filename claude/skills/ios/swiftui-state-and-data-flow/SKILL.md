---
name: swiftui-state-and-data-flow
description: >-
  Manage SwiftUI state and data flow correctly — choose between @State, @Binding,
  @Bindable, @Environment, @AppStorage, and @Observable models; establish a
  single source of truth; pass data down and actions up; create and use Bindings
  with $; and decide what state a view should own vs. receive. Use when a view
  doesn't update, updates too much, when deciding which property wrapper to use,
  or when structuring data flow between parent and child views on iOS 26+.
---

# SwiftUI State & Data Flow

SwiftUI re-renders a view when its observed state changes. Getting data flow right means having a **single source
of truth** for each piece of state and choosing the right wrapper based on **who owns the data**. Pairs with
`observation-framework` (for `@Observable` model classes).

> **Versions.** iOS 26+ baseline. With `@Observable` (iOS 17+) the old `@StateObject`/`@ObservedObject`/
> `@EnvironmentObject` trio is no longer used for model classes — see the migration table in `observation-framework`.

## The decision: who owns this state?

| Wrapper | Owns? | Use for |
| --- | --- | --- |
| `@State` | View owns it | Local value state (`Bool`, `Int`, `String`, enums) **and** view-owned `@Observable` model instances |
| `@Binding` | Parent owns it | A read-write reference to a parent's value state |
| `@Bindable` | Passed in | Getting `$` bindings to a passed-in `@Observable` model's properties |
| `@Environment` | App/ancestor owns it | Injected models or system values (color scheme, dismiss, etc.) |
| `@AppStorage` | UserDefaults | Small persisted prefs (flags, last tab) |

## @State — local source of truth

```swift
struct Counter: View {
    @State private var count = 0                 // this view owns `count`
    var body: some View {
        Stepper("Count: \(count)", value: $count)  // $count is a Binding<Int>
    }
}
```

`@State` should be `private`. For value types it holds the value; for an `@Observable` class it holds the single
instance across re-renders (replacing `@StateObject`).

## @Binding — share write access with a child

Pass `$value` down so the child can read **and** write the parent's state:

```swift
struct Parent: View {
    @State private var name = ""
    var body: some View { NameField(name: $name) }   // pass a Binding
}

struct NameField: View {
    @Binding var name: String                        // two-way connection
    var body: some View { TextField("Name", text: $name) }
}
```

The child doesn't own the data — it edits the parent's single source of truth.

## @Bindable — bindings into an @Observable model

When a child receives an `@Observable` model and needs `$` bindings to its properties:

```swift
struct EditTrip: View {
    @Bindable var trip: Trip               // passed-in @Observable / @Model
    var body: some View {
        TextField("Name", text: $trip.name)   // binding into the model
    }
}
```

If the child only *reads* the model, use a plain `let` — no wrapper needed (SwiftUI still tracks the properties it reads).

## @Environment — inject across many layers

Avoid threading a dependency through every initializer. Inject once, read anywhere below:

```swift
@main struct MyApp: App {
    @State private var session = SessionModel()
    var body: some Scene {
        WindowGroup { RootView().environment(session) }
    }
}

struct ProfileView: View {
    @Environment(SessionModel.self) private var session
    @Environment(\.dismiss) private var dismiss        // built-in environment values
    var body: some View { /* … */ }
}
```

## @AppStorage — small persisted settings

```swift
struct SettingsView: View {
    @AppStorage("isDarkMode") private var isDarkMode = false
    var body: some View { Toggle("Dark Mode", isOn: $isDarkMode) }   // persists to UserDefaults
}
```

For real app data use SwiftData (`swiftdata-persistence`), not `@AppStorage`.

## Data down, actions up

Keep state as high as the lowest common ancestor that needs it. Pass **data down** (plain values or models) and
send **actions up** (closures or method calls on a shared model):

```swift
struct TaskRow: View {
    let task: Task
    var onToggle: () -> Void                 // action goes up to the owner
    var body: some View {
        Button { onToggle() } label: { Label(task.title, systemImage: task.done ? "checkmark.circle.fill" : "circle") }
    }
}
```

For anything beyond trivial local state, put the logic in an `@Observable` model (see `ios-app-architecture`) and
pass the model down rather than scattering many `@State`/`@Binding` values.

## Creating Bindings manually

```swift
// From a model property:
Toggle("On", isOn: $model.isOn)

// Derived/custom binding:
let binding = Binding(get: { model.value }, set: { model.value = $0 })

// Optional with a default:
TextField("Note", text: $model.note ?? "")   // with a Binding optional-unwrap helper
```

## Pitfalls

- **Duplicated state** (same data in two `@State`s) → bugs; keep one source of truth and pass `@Binding`.
- **`@State` for passed-in data** → it won't update when the parent changes; use `@Binding`/plain/`@Bindable`.
- **Reaching for `@StateObject`/`@ObservedObject`** with `@Observable` types → use `@State`/plain/`@Bindable`.
- **State too low** → if a sibling also needs it, lift it to the common parent.
- **Non-private `@State`** → make it `private`; it's view-internal.

## Quick reference

| Situation | Wrapper |
| --- | --- |
| Local value this view owns | `@State private` |
| View owns an `@Observable` model | `@State private var m = Model()` |
| Child edits parent's value | `@Binding` (`$value`) |
| Child needs `$` on a passed-in model | `@Bindable` |
| Child only reads a passed-in model | plain `let` |
| Cross-cutting dependency | `@Environment` + `.environment(_:)` |
| Small persisted setting | `@AppStorage` |
