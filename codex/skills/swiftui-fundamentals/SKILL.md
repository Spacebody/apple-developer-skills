---
name: swiftui-fundamentals
description: >-
  Provide a foundational SwiftUI reference for view composition, stacks and
  grids, modifiers, List and ForEach, NavigationStack, sheets and alerts,
  standard controls, and basic animations. Use for learning SwiftUI fundamentals,
  answering a focused API question, or as a fallback when no installed
  build-ios-apps:* skill is a closer match. Do not auto-use for production screen
  architecture, view refactoring, Liquid Glass, performance, Simulator debugging,
  or other work owned by a specialized build-ios-apps:* skill.
---

# SwiftUI Fundamentals (iOS)

SwiftUI is the declarative UI framework for all Apple platforms. You describe *what* the UI should be for a given
state; the framework renders and updates it. For state/data flow see `swiftui-state-and-data-flow` and
`observation-framework`; for app structure see `ios-app-architecture`.

> **Scope:** this skill is for *building* UI. To *audit* finished UI against Apple's Human Interface Guidelines
> (Dynamic Type, semantic colors/Dark Mode, 44pt hit targets, accessibility, standard navigation/components), use
> the `apple-hig-review` skill instead — it's a severity-graded review checklist, not a how-to-build reference.

> **Specialized work.** Prefer `build-ios-apps:swiftui-ui-patterns` for production component patterns,
> `build-ios-apps:swiftui-view-refactor` for restructuring, and
> `build-ios-apps:swiftui-liquid-glass` for Liquid Glass implementation or review. Confirm SDK availability
> from the installed toolchain instead of assuming unreleased APIs.

> **iOS 27 / Xcode 27.** When a task targets the 2027 platform generation, read
> [references/ios-27.md](references/ios-27.md). It separates Apple-published changes from a retained watchlist
> that must be rechecked against the shipping SDK.

## Views & composition

A view is a `struct` conforming to `View` with a `body`:

```swift
struct ProfileCard: View {
    let user: User
    var body: some View {
        HStack(spacing: 12) {
            AsyncImage(url: user.avatar) { $0.resizable() } placeholder: { ProgressView() }
                .frame(width: 48, height: 48)
                .clipShape(.circle)
            VStack(alignment: .leading) {
                Text(user.name).font(.headline)
                Text(user.handle).font(.subheadline).foregroundStyle(.secondary)
            }
        }
        .padding()
    }
}
```

Build UIs by composing small views. Extract subviews freely — it's cheap and improves update granularity.

## Layout

- **Stacks**: `HStack`, `VStack`, `ZStack` (+ `spacing:`, `alignment:`).
- **Spacer / padding / frame**: `Spacer()`, `.padding(_:)`, `.frame(width:height:alignment:)`,
  `.frame(maxWidth: .infinity)` to fill.
- **Grids**: `Grid`/`GridRow` for static grids; `LazyVGrid`/`LazyHGrid` with `GridItem` for scrollable collections.
- **ScrollView**: wrap content; use `LazyVStack` inside for large lists of non-row content.
- **Custom layout**: conform to the `Layout` protocol for bespoke arrangements.

```swift
ScrollView {
    LazyVGrid(columns: [GridItem(.adaptive(minimum: 120))], spacing: 12) {
        ForEach(photos) { PhotoCell(photo: $0) }
    }.padding()
}
```

### Adaptive layout

Drive layout from available space and size class rather than hard-coded device checks. Use
`@Environment(\.horizontalSizeClass)` and `NavigationSplitView` for iPad/large widths, and prefer fluid reflow
over fixed frames.

## Modifiers

Order matters — modifiers wrap outward:

```swift
Text("Hi")
    .padding()                       // pad first
    .background(.blue, in: .capsule) // then background around the padded text
    .foregroundStyle(.white)
```

Common: `.font`, `.foregroundStyle`, `.background`, `.overlay`, `.clipShape`, `.cornerRadius`/`.clipShape(.rect(cornerRadius:))`,
`.shadow`, `.opacity`, `.disabled`, `.tint`.

## Lists

```swift
List {
    Section("Today") {
        ForEach(tasks) { task in
            TaskRow(task: task)
        }
        .onDelete { tasks.remove(atOffsets: $0) }
    }
}
.listStyle(.insetGrouped)
```

Models in `ForEach` must be `Identifiable` (or pass `id:`). For SwiftData-backed lists use `@Query` (see
`swiftdata-persistence`).

## Navigation

Use `NavigationStack` (not the deprecated `NavigationView`):

```swift
NavigationStack {
    List(articles) { article in
        NavigationLink(article.title, value: article)
    }
    .navigationTitle("Articles")
    .navigationDestination(for: Article.self) { article in
        ArticleDetail(article: article)
    }
}
```

For programmatic control, bind a `path`: `NavigationStack(path: $path) { … }` where `path` is a
`[Route]` or `NavigationPath`. For multi-column (iPad/Mac) use `NavigationSplitView`.

Modals:

```swift
.sheet(isPresented: $showSheet) { EditView() }
.alert("Delete?", isPresented: $confirming) {
    Button("Delete", role: .destructive) { delete() }
    Button("Cancel", role: .cancel) { }
}
```

## Controls

```swift
Button("Save") { save() }
Button("Delete", role: .destructive) { }
TextField("Email", text: $email).textFieldStyle(.roundedBorder)
Toggle("Notifications", isOn: $on)
Picker("Theme", selection: $theme) { ForEach(Theme.allCases) { Text($0.name).tag($0) } }
Stepper("Qty: \(qty)", value: $qty, in: 1...99)
```

Wrap form-like UIs in `Form { … }` for platform-correct styling.

## Liquid Glass styling (iOS 26+)

Prefer standard system navigation bars, toolbars, sheets, and controls; they receive the platform's
Liquid Glass treatment automatically. Reach for custom glass APIs only when building custom app chrome.

Use `build-ios-apps:swiftui-liquid-glass` for Liquid Glass APIs, availability handling, modifier ordering,
container placement, performance, and design fit. Keep this fundamentals skill focused on stable SwiftUI basics.

## Animations & transitions

```swift
withAnimation(.spring) { isExpanded.toggle() }      // animate a state change
view.animation(.easeInOut, value: isExpanded)        // animate when value changes
view.transition(.move(edge: .bottom))                // insertion/removal transition
```

Prefer `.spring`/`.smooth`/`.snappy` presets. Use `matchedGeometryEffect` (or `.matchedTransitionSource` for
zoom navigation transitions) for shared-element animations.

## Pitfalls

- **Massive `body`** — extract subviews; SwiftUI diffs and updates them independently.
- **`NavigationView`** — deprecated; use `NavigationStack`/`NavigationSplitView`.
- **Hard-coded device sizes** — drive layout from size classes / adaptive APIs.
- **Glass on everything** — reserve Liquid Glass for chrome, not content.
- **Heavy work in `body`** — `body` runs often; compute in the model, not the view.

## Quick reference

| Need | API |
| --- | --- |
| Linear/overlay layout | `VStack`/`HStack`/`ZStack` |
| Scrollable grid | `ScrollView` + `LazyVGrid` |
| List with delete | `List` + `ForEach` + `.onDelete` |
| Push navigation | `NavigationStack` + `NavigationLink(value:)` + `.navigationDestination` |
| Multi-column | `NavigationSplitView` |
| Modal | `.sheet` / `.alert` / `.confirmationDialog` |
| Async image | `AsyncImage` |
| Liquid Glass | `build-ios-apps:swiftui-liquid-glass` |
| iOS 27 / Xcode 27 | [references/ios-27.md](references/ios-27.md) |
| Animate | `withAnimation { }` / `.animation(_:value:)` / `.transition` |
