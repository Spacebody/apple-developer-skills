# macOS HIG source map

Use these primary Apple sources to support a finding or refresh the checklist. Prefer the most specific topic and
recheck pages when Apple changes the HIG or the target SDK.

## Platform foundations

- [macOS Pathway](https://developer.apple.com/macos/get-started/): Apple's current platform overview spanning
  tools, native design, windows and window controllers, menus, Dock integration, privacy, security,
  accessibility, localization, testing, performance, Catalyst, iOS apps on Mac, and distribution.
- [Designing for macOS](https://developer.apple.com/design/human-interface-guidelines/designing-for-macos/):
  large and multiple displays, resizable and full-screen windows, menu-bar commands, keyboard workflows,
  precision input, and personalization.
- [Windows](https://developer.apple.com/design/human-interface-guidelines/windows): window anatomy, main/key
  window state, sizing, placement, controls, and specialized panels.
- [Accessibility](https://developer.apple.com/design/human-interface-guidelines/accessibility/): semantic and
  multimodal access, larger text, Full Keyboard Access, VoiceOver, contrast, motion, and control sizing. The
  published macOS defaults are 13 pt text and 28 x 28 pt controls; the published minimums are 10 pt and 20 x 20 pt.
- [Accessibility for AppKit](https://developer.apple.com/documentation/appkit/accessibility-for-appkit): standard
  AppKit controls' built-in accessibility and requirements for custom `NSView` or `NSAccessibilityElement` types.

## Desktop navigation and commands

- [Menus](https://developer.apple.com/design/human-interface-guidelines/menus): command organization, menu item
  labels and states, shortcuts, submenus, and menu-bar expectations.
- [Keyboards](https://developer.apple.com/design/human-interface-guidelines/keyboards): established shortcuts,
  keyboard-only workflows, focus, and Full Keyboard Access.
- [Toolbars](https://developer.apple.com/design/human-interface-guidelines/toolbars): frequent commands, item
  grouping, customization and overflow. On macOS, make toolbar commands available in the menu bar too.
- [Sidebars](https://developer.apple.com/design/human-interface-guidelines/sidebars): hierarchy depth, visibility,
  accent color, adaptive collapse, and macOS sidebar sizing.
- [Context menus](https://developer.apple.com/design/human-interface-guidelines/context-menus): concise contextual
  commands, one submenu level, availability rules, and a non-contextual path to every action.
- [Dock menus](https://developer.apple.com/design/human-interface-guidelines/dock-menus): high-value commands and
  window shortcuts that remain available from the menu bar or primary UI too.

## Preferences, modality, and direct manipulation

- [Settings](https://developer.apple.com/design/human-interface-guidelines/settings): App-menu placement,
  Command-Comma, settings-window behavior, pane organization, and avoiding duplicate system preferences.
- [Alerts](https://developer.apple.com/design/human-interface-guidelines/alerts): actionable interruptions,
  destructive/default/cancel actions, Escape and Command-Period, suppression, and accessory content.
- [Sheets](https://developer.apple.com/design/human-interface-guidelines/sheets): window-attached modal work and
  the boundary between a sheet, alert, and independent window.
- [Drag and drop](https://developer.apple.com/design/human-interface-guidelines/drag-and-drop): multi-item drags,
  move/copy behavior, undo, cross-app exchange, Finder exports, and accessible alternatives.
- [Pointing devices](https://developer.apple.com/design/human-interface-guidelines/pointing-devices): standard
  pointer shapes, hover feedback, precision input, and drag-result communication.

## SwiftUI and AppKit implementation entry points

- [SwiftUI](https://developer.apple.com/documentation/swiftui): scenes, windows, commands, toolbars, settings,
  inspectors, AppKit integration, accessibility, and localization.
- [AppKit](https://developer.apple.com/documentation/appkit): macOS windows, views, controls, menus, events,
  documents, pasteboard, accessibility, and application lifecycle.
- [SwiftUI `Settings`](https://developer.apple.com/documentation/swiftui/settings): declaring a standard settings
  scene and enabling the App-menu Settings command.

## iPad and iOS apps on Mac

- [Running your iOS apps in macOS](https://developer.apple.com/documentation/apple-silicon/running-your-ios-apps-in-macos):
  compatibility constraints, unavailable mobile hardware, touch-dependent interactions, testing on the actual
  Mac runtime, and the boundary between an unmodified app and Mac Catalyst.
- [Mac Catalyst](https://developer.apple.com/documentation/uikit/mac-catalyst): building a Mac-specific variant of
  an iPad app and customizing its desktop behavior.
- [Menus and shortcuts](https://developer.apple.com/documentation/uikit/menus-and-shortcuts): menu-bar commands,
  contextual actions, and keyboard shortcuts for Mac Catalyst apps.
