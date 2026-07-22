# iOS 27 and Xcode 27 forward notes

Read this reference only when the request targets iOS 27, the 2027 OS generation, or Xcode 27. Keep iOS 26-compatible code unchanged unless the user asks to adopt newer behavior. Confirm availability against the installed SDK and add deployment guards where required.

## Apple-published changes

Apple's June 2026 SwiftUI update materials document these changes:

- Building with Xcode 27 or later changes `@State` to use the `State()` macro. For class values, initialization and storage occur once. Recheck ownership assumptions when migrating code that relied on eager property-wrapper initialization.
- `ContentBuilder` provides type-agnostic content construction and replaces several type-specific builder patterns.
- `reorderable()` and `reorderContainer(for:isEnabled:move:)` add drag reordering to lists, stacks, grids, and custom layouts.
- `swipeActions(edge:allowsFullSwipe:content:onPresentationChanged:)` and `swipeActionsContainer()` add swipe actions beyond `List` rows.
- `AsyncImage` gains standard HTTP caching behavior and additional request/session configuration in the 2027 platform generation.
- Apps built with Xcode 27 adopt the refreshed Liquid Glass appearance on 2027 OS releases. Standard controls receive the updated appearance automatically; use `build-ios-apps:swiftui-liquid-glass` before changing custom glass code.

Primary sources:

- [SwiftUI updates](https://developer.apple.com/documentation/updates/swiftui)
- [`State` documentation](https://developer.apple.com/documentation/swiftui/state)
- [What's new in SwiftUI, WWDC26](https://developer.apple.com/videos/play/wwdc2026/269/)
- [Adopting Liquid Glass](https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass)

## Retained watchlist

Keep these ideas for release-time verification, but do not present them as shipping APIs or products until Apple publishes matching SDK documentation:

- **"Liquid Glass 2"** was shorthand in an earlier draft, not an Apple-confirmed product name. Recheck whether new transparency controls or variants exist in the shipping SDK, and then replace this note with exact API names.
- **Foldable or hinge-aware iPhone layouts** remain a watch item. Do not claim an `iPhone Fold`, hinge-state environment value, or foldable-specific SwiftUI API exists until it appears in Apple's hardware and SDK documentation.

When iOS 27 ships, refresh this file from the final Xcode 27 SDK diff and Apple release notes. Move confirmed watchlist items into the Apple-published section, delete disproven items, and keep availability annotations exact.
