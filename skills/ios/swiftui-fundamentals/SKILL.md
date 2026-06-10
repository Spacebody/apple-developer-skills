---
name: swiftui-fundamentals
description: >-
  Build iOS user interfaces with SwiftUI — views and view composition, layout
  with HStack/VStack/ZStack/Grid and the Layout protocol, view modifiers, List
  and ForEach, NavigationStack and navigationDestination, sheets/alerts,
  controls (Button/TextField/Toggle/Picker), animations and transitions, and the
  Liquid Glass design system (glassEffect, .glass styles, adaptive/foldable
  layouts). Use when creating SwiftUI screens, laying out views, building
  navigation, styling, or adopting the current iOS 26/27 look and feel.
---

# SwiftUI Fundamentals (iOS)

SwiftUI is the declarative UI framework for all Apple platforms. You describe *what* the UI should be for a given
state; the framework renders and updates it. For state/data flow see `swiftui-state-and-data-flow` and
`observation-framework`; for app structure see `ios-app-architecture`.

> **Versions.** Baseline iOS 26; forward notes for iOS 27 (developer beta, public release fall 2026). iOS 26
> introduced the **Liquid Glass** design; iOS 27 refines it ("Liquid Glass 2": new transparency controls) and adds
> **adaptive/foldable layout** APIs (hinge-state) for devices like the iPhone Fold.

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

### Adaptive & foldable layout (iOS 27)

Drive layout from size class and the new hinge/adaptive APIs rather than hard-coding device checks. Continue to
use `@Environment(\.horizontalSizeClass)` and `NavigationSplitView` for iPad/large widths; on foldables prefer
fluid reflow over fixed frames so content adapts to the unfolded canvas instead of letterboxing.

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

The system applies Liquid Glass to standard navigation/toolbars/sheets automatically — adopt standard components
and you get it for free. Apply glass to custom surfaces:

```swift
// Custom view on a glass surface:
MyControlBar()
    .glassEffect()                       // Liquid Glass material
    .glassEffect(in: .capsule)           // shaped

// Glass-styled button:
Button("Filter") { }.buttonStyle(.glass)

// Group multiple glass elements so they morph together coherently:
GlassEffectContainer {
    HStack { GlassButton(); GlassButton() }
}
```

iOS 27 ("Liquid Glass 2") adds finer transparency controls; if your app mixes SwiftUI and UIKit, audit
`GlassEffectContainer` boundaries so adjacent glass elements morph consistently. Don't over-apply glass to content
areas — it's for chrome (controls, bars, floating surfaces), not body content.

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
| Liquid Glass surface | `.glassEffect()` / `.buttonStyle(.glass)` / `GlassEffectContainer` |
| Animate | `withAnimation { }` / `.animation(_:value:)` / `.transition` |
