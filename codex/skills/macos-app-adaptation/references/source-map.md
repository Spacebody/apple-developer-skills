# Mac adaptation source map

Use current Apple documentation and verify APIs against the installed SDK.

## Choosing and adapting the runtime

- [Running your iOS apps in macOS](https://developer.apple.com/documentation/apple-silicon/running-your-ios-apps-in-macos):
  eligibility, Apple-silicon runtime, fixed/resizable windows, environment assumptions, testing, and App Store
  availability.
- [Adapting iOS code to run in the macOS environment](https://developer.apple.com/documentation/apple-silicon/adapting-ios-code-to-run-in-the-macos-environment):
  hardware checks, UIKit idiom behavior, scenes, iPad multitasking, keyboard, filesystem, system UI, and deprecated
  framework migration.
- [Mac Catalyst](https://developer.apple.com/documentation/uikit/mac-catalyst): Catalyst target, desktop UIKit
  adaptations, menus, windows, controls, and platform customization.
- [Building and improving your app with Mac Catalyst](https://developer.apple.com/documentation/uikit/building-and-improving-your-app-with-mac-catalyst):
  native controls, multiple windows, sharing, printing, menus, and shortcuts.

## Safe environment and window APIs

- Swift compile condition: `#if targetEnvironment(macCatalyst)` for the Catalyst build only.
- Runtime property: `ProcessInfo.processInfo.isiOSAppOnMac` for the unmodified iOS-on-Mac runtime; prefer capability
  checks and shared behavior before using it.
- [`ProcessInfo.isMacCatalystApp`](https://developer.apple.com/documentation/foundation/processinfo/ismaccatalystapp)
  is not a Catalyst-only runtime discriminator; distinguish compilation with `targetEnvironment(macCatalyst)`.
- [Choosing a user interface idiom](https://developer.apple.com/documentation/uikit/choosing-a-user-interface-idiom-for-your-mac-app):
  scaled-iPad versus optimized-for-Mac Catalyst idioms and why `.mac` does not mean every Mac runtime.
- [`UITitlebar`](https://developer.apple.com/documentation/uikit/uititlebar) and
  [`UIWindowScene.titlebar`](https://developer.apple.com/documentation/uikit/uiwindowscene/titlebar): Catalyst
  title and toolbar integration.
- [Catalyst toolbar APIs](https://developer.apple.com/documentation/uikit/mac-catalyst/toolbar): `NSToolbar`,
  toolbar items, validation, and UIKit-hosting toolbar items exposed to Catalyst.
- Scene key: `UIApplicationSupportsMultipleScenes`. Full-screen-on-Mac keys:
  `UILaunchToFullScreenByDefaultOnMac` and `UISupportsTrueScreenSizeOnMac`. Verify the target product model before
  adding them and avoid `UIRequiresFullScreen` for a resizable iPad app.

## Commands and input

- [Menus and shortcuts](https://developer.apple.com/documentation/uikit/menus-and-shortcuts): `UIMenu`,
  `UICommand`, `UIKeyCommand`, menu systems, and Mac Catalyst command surfaces.
- [`UIMenuBuilder`](https://developer.apple.com/documentation/uikit/uimenubuilder): customize the main menu from
  the app delegate or context menus from a view controller.
- [Adding menus and shortcuts](https://developer.apple.com/documentation/uikit/adding-menus-and-shortcuts-to-the-menu-bar-and-user-interface):
  complete Catalyst sample and standard menu placement.
- [Providing touch gesture equivalents using Touch Alternatives](https://developer.apple.com/documentation/apple-silicon/providing-touch-gesture-equivalents-using-touch-alternatives):
  compatibility fallback for an iOS app on Mac; prefer native keyboard/pointer paths when code changes are possible.
- [Pointer interactions](https://developer.apple.com/documentation/uikit/pointer-interactions) and
  [`UIHoverGestureRecognizer`](https://developer.apple.com/documentation/uikit/uihovergesturerecognizer): pointer
  affordances and explicit hover state for custom UIKit controls.

## Adjacent boundaries

- [Designing for macOS](https://developer.apple.com/design/human-interface-guidelines/designing-for-macos/):
  desktop windows, menus, keyboard, precision input, and personalization for the HIG audit.
- [Porting macOS apps to Apple silicon](https://developer.apple.com/documentation/apple-silicon/porting-your-macos-apps-to-apple-silicon):
  universal native Mac binaries and Rosetta, which are outside this iPad/iOS adaptation skill.
