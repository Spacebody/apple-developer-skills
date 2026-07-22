---
name: macos-app-adaptation
description: Adapt an existing iPadOS or iOS app to run well on Mac through Mac Catalyst or the unmodified iOS Apps on Mac runtime. Use when choosing between those paths, enabling a Catalyst target, removing mobile-only assumptions, adding Mac menus and keyboard shortcuts, adapting windows and resizing, checking hardware/framework availability, using Catalyst titlebar or toolbar APIs, supporting pointer and non-touch input, or testing and distributing the Mac variant. Do not use for a new native AppKit/SwiftUI macOS app, a HIG-only audit, ordinary iOS UI work, Apple-silicon porting of an existing native Mac binary, or signing/notarization failures.
---

# macOS App Adaptation

Prefer shared, capability-driven code. Add platform branches only where the product behavior truly differs, and
test the actual Mac runtime rather than treating Simulator or an iPad window as proof.

## Boundaries

- Use `macos-hig-review` to audit desktop conventions; this skill implements the adaptation.
- Use `build-macos-apps:*` for a native macOS SwiftUI/AppKit product, build/debug, window internals, signing,
  packaging, or notarization. Do not force a Catalyst app into native AppKit architecture by accident.
- Use `app-store-review` for App Store availability, policy, privacy, and submission decisions.
- Use `apple-localization` for menu titles, device variations such as “Tap” versus “Click,” and localized shortcuts.

Read [references/source-map.md](references/source-map.md) before relying on platform detection, Catalyst-only
UIKit/AppKit integration, Info.plist keys, or App Store behavior.

## Choose the product path first

| Path | Use when | Key constraints |
| --- | --- | --- |
| iOS app on Mac | The existing compatible iPhone/iPad binary already works and needs small capability/input fixes | Apple silicon only, no Mac-specific compilation, UIKit idiom remains phone/pad |
| Mac Catalyst | An iPad app needs a separately built Mac variant with desktop menus, titlebar/toolbar customization, and Mac-specific code | Maintain a Catalyst target and test both shared and conditional branches |
| Native macOS | The product needs deep AppKit/macOS behavior or a desktop-first architecture | Route to `build-macos-apps:*`; do not use this skill as a porting shortcut |

Document why the selected path fits the feature set, supported hardware, distribution plan, and long-term code
ownership. Do not select Catalyst merely to gain one AppKit API or keep iOS-on-Mac enabled for an unusable app.

## Detect capabilities, not labels

1. Check the specific sensor, camera, framework, API, input, and window capability at runtime and provide a useful
   fallback. Macs may lack motion sensors, GPS precision, particular cameras, or iOS-only frameworks.
2. Use `#if targetEnvironment(macCatalyst)` only for code compiled into the Catalyst variant.
3. Use `ProcessInfo.processInfo.isiOSAppOnMac` only when behavior genuinely must distinguish the unmodified Mac
   runtime. Apple recommends this as a last choice; shared capability-driven behavior is more robust.
4. Do not detect iOS-on-Mac with `UIDevice.current.userInterfaceIdiom == .mac`: an iPad-deployable app reports
   `.pad`, and an iPhone-only app reports `.phone` in that runtime. Catalyst itself may report `.pad` or `.mac`
   depending on its interface-optimization setting.
5. Use `#available` and framework capability APIs independently from environment detection. A platform label does
   not prove an API or hardware feature is usable.

Do not use `ProcessInfo.processInfo.isMacCatalystApp` as a Catalyst-only discriminator; it can also be true for an
iOS app running on Mac. Do not substitute `#if os(macOS)` for Catalyst detection.

## Adapt windows, layout, and system UI

- Adopt scenes and enable multiple scenes when independent documents or workspaces benefit from multiple windows.
  Keep state scene-scoped rather than assuming one global window.
- Support iPad multitasking, compact/regular size classes, flexible constraints, all applicable orientations, and
  live resizing. Avoid `UIRequiresFullScreen` when resizable Mac windows are expected. iPhone-only apps remain in a
  fixed-size window.
- Do not inspect or reposition private internals of system pickers, alerts, share sheets, or file interfaces.
  Present them through public UIKit APIs and let macOS choose its native panel/presentation.
- Resolve app/container/standard-directory URLs through Foundation. Users can move apps, and filesystem locations
  and visibility differ from iOS.
- Use the documented full-screen-on-Mac Info.plist keys only for a deliberately full-screen iPad experience, not
  as a workaround for broken resizing.

## Add desktop input and command surfaces

1. Route touch, keyboard, menu, toolbar, and pointer actions to the same command/state layer.
2. Add `UIKeyCommand`, `UICommand`, and `UIMenu` entries for frequent actions. In Catalyst, customize the main menu
   with `buildMenu(with:)` / `UIMenuBuilder` and preserve conventional Mac command placement and shortcuts.
3. Keep context menus, hover, and secondary click supplementary. Provide visible/menu/keyboard paths for core work.
4. Prefer standard gesture recognizers that macOS maps to trackpad/mouse input. Provide keyboard or visible
   alternatives for custom multifinger gestures; use Touch Alternatives only as a compatibility fallback.
5. Use pointer interactions to communicate actionable regions, not to hide information until hover.

## Catalyst-specific customization

- Access a Catalyst window's `UIWindowScene.titlebar` and `UITitlebar` only inside the Catalyst build and on a
  supported OS. Attach an `NSToolbar` when native desktop command density warrants it; keep commands available in
  the menu bar too.
- Prefer UIKit APIs that adapt automatically. Use AppKit classes exposed to Catalyst narrowly and isolate them so
  the iOS target never imports or references unavailable symbols. `canImport(AppKit)` alone is not permission to
  call arbitrary AppKit; use only APIs documented as Mac Catalyst-available.
- Keep title, represented document URL, toolbar visibility/style, and window state synchronized with the current
  scene/document rather than one application-global singleton.
- Treat `UIWindowScene.sizeRestrictions` as optional and change it only when the product has a justified minimum
  or maximum size.

## Test and distribute

1. Run the iOS binary with the “My Mac (Designed for iPad/iPhone)” destination on Apple silicon; this is a real Mac
   runtime, not Simulator. Separately build and run the Catalyst target.
2. If the Catalyst product evaluates both scaled-iPad and optimized-for-Mac interface idioms, test both; do not
   assume one run covers the other.
3. Exercise resizing, multiple windows, menu validation, shortcuts, pointer/scroll/trackpad, drag and drop, file
   pickers, camera/sensor fallbacks, open URLs/documents, background/foreground transitions, and accessibility.
4. Test shared behavior on iPhone/iPad again so Mac branches do not regress the primary product.
5. Validate every supported architecture and distribution route. Treat Mac App Store availability for an iOS app
   as a product decision in App Store Connect, not a compile-time assumption.

## Output

State the chosen path and why, platform/capability branches introduced, Mac command/window/input behavior, hardware
fallbacks, actual destinations and architectures tested, App Store availability implications, and any native-macOS
work intentionally deferred.
