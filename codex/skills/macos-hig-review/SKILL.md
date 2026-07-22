---
name: macos-hig-review
description: Review SwiftUI and AppKit macOS interfaces against code-detectable Apple Human Interface Guidelines requirements. Use when the user explicitly requests a macOS HIG review, Mac interface-compliance audit, desktop accessibility or keyboard-navigation review, or asks whether a Mac app follows Apple conventions. Check windows, menus and commands, keyboard access, toolbars, sidebars and inspectors, settings, modality, pointer and drag interactions, accessibility, localization, and adaptive desktop layout. Do not use for iOS/iPadOS HIG reviews, generic visual polish, branding, build failures, signing, packaging, or implementation work that is not asking for HIG compliance.
---

# macOS HIG Review

Audit only issues that source, configuration, or an observed runtime state can support. Keep visual taste, brand
preference, and speculative screenshot criticism out of a code-only review.

## Boundaries

- Use `apple-hig-review` for iOS or iPadOS. Do not apply iOS's 44 pt target rule to macOS controls.
- Use `build-macos-apps:swiftui-patterns` for implementing scenes, commands, settings, split views, and inspectors.
- Use `build-macos-apps:window-management` for window chrome, placement, restoration, and specialized windows.
- Use `build-macos-apps:appkit-interop` for panels, responder-chain behavior, and narrow AppKit bridges.
- Use `build-macos-apps:liquid-glass` for macOS Liquid Glass implementation or review.
- Use `macos-app-adaptation` to implement Mac Catalyst or iOS/iPadOS-on-Mac changes; this skill audits the result.
- Use `macos-performance-profiling` for runtime responsiveness, CPU, memory, disk, or energy claims.
- Use `apple-localization` for String Catalog, extraction, plural/device variation, and RTL implementation work.
- Use `app-store-review` for submission, privacy, entitlement, and App Store policy compliance.

Read [references/source-map.md](references/source-map.md) when a finding needs an exact Apple design topic,
a platform-specific measurement, or a current primary-source link.

## Review workflow

1. Confirm the target is macOS and identify SwiftUI, AppKit, Catalyst, or hybrid UI. State the reviewed scope.
2. Read behavior-driving files before localization or styling churn: the app entry point, scenes/windows, root
   views/controllers, commands and menus, settings, toolbars, selection model, alerts/sheets, and accessibility.
3. Trace each important action through all discoverable paths: content UI, menu bar, toolbar, context menu, and
   keyboard. Verify the action reaches the same enabled-state and undo/destructive behavior.
4. Apply the checklist below. Report only evidence-backed findings with a concrete user impact and smallest fix.
5. Route implementation to the closest specialized skill. Do not broaden a HIG audit into an unsolicited redesign.

## Code-detectable checklist

### Desktop structure and windows

- Model independent workspaces, documents, settings, utilities, and menu bar extras as deliberate scenes or
  windows instead of hiding the entire app inside one phone-like navigation stack.
- Let ordinary document and content windows resize, move, restore, and enter full screen when the product model
  supports it. Flag rigid fixed frames that make core content unusable at plausible window sizes.
- Preserve standard window controls and title/toolbar behavior unless a specialized window has a documented need.
- Prefer fewer nested navigation levels and less modality on a large display. Keep selection stable across
  sidebar-detail-inspector layouts and across window activation changes.
- For Mac Catalyst or an iOS app running on Apple silicon, test the actual Mac runtime. Flag core flows that
  require touch-only gestures, unavailable iPhone/iPad hardware, or fixed mobile geometry without a keyboard,
  pointer, menu, window-resizing, or full-screen equivalent.

### Menu bar, commands, and keyboard

- Expose the app's complete command set through conventional menu-bar locations. Use standard names, ordering,
  roles, and shortcuts for New/Open/Close, Save, Undo/Redo, Cut/Copy/Paste, Find, Settings, Window, and Help.
- Do not override system-reserved or widely established shortcuts with unrelated actions.
- Give frequent desktop actions keyboard shortcuts and a keyboard-only path. Verify focus order, default action,
  cancellation with Escape where appropriate, and disabled state from the same source of truth as visible controls.
- Never make a toolbar, context menu, hover affordance, gesture, or secondary click the only path to a core action.
  Every toolbar command and contextual command should also be discoverable from the menu bar or primary UI.
- Treat custom Dock menu commands as shortcuts to a few high-value actions or windows, not a complete command
  surface. Provide the same actions in the menu bar or primary interface; use badges only for meaningful state.

### Toolbars, sidebars, inspectors, and dense content

- Keep toolbars focused on frequent commands and navigation. Let macOS manage overflow; do not add a redundant
  custom overflow control or assume every menu command deserves toolbar space.
- Let people hide and show a sidebar through a standard control or View menu command. Avoid more than two visible
  hierarchy levels, fixed icon colors that ignore the accent color, or critical actions anchored only at the
  bottom of a potentially clipped sidebar.
- Use inspectors for secondary properties and editing controls that should remain available beside primary
  content. Keep tables and lists selection-driven, sortable or resizable where the task benefits from it, and
  usable with keyboard navigation.

### Settings, modality, and feedback

- Put infrequently changed app-wide preferences in a standard Settings scene/window reachable from the App menu
  with Command-Comma. Do not consume a main-window toolbar item for ordinary app settings or duplicate systemwide
  accessibility and appearance preferences.
- Attach a sheet to the window or document whose task it blocks. Keep independent work in a separate window.
- Use alerts only for essential, actionable, and interruptive information. Prefer inline status for recoverable
  errors; prefer Undo over confirmation for routine reversible destruction.
- Give destructive or irreversible choices explicit labels and a safe cancellation path. Keep progress, empty,
  offline, permission-denied, and failure states understandable without opening logs.

### Pointer, drag and drop, and contextual interaction

- Treat hover as enhancement, not required discovery. Use standard pointer shapes and visible state changes only
  when they communicate an interaction or drag result.
- Support multi-item drag and drop when the model permits it, provide correct move-versus-copy semantics, and make
  accidental operations undoable where practical. If content can leave the app, offer a durable format the app
  can open again.
- Keep context menus concise and relevant. Do not hide a unique action there; expose the same command through the
  menu bar or visible interface.

### Appearance, controls, and accessibility

- Prefer semantic colors, system materials, standard controls, SF Symbols, and native selection appearances so
  Light/Dark Mode, accent color, Increased Contrast, and Reduced Transparency remain effective.
- Flag custom fixed controls below the macOS HIG's 20 x 20 pt minimum or custom text below its 10 pt minimum when
  there is no justified dense-data exception. Treat 28 x 28 pt controls and 13 pt text as platform defaults, not
  mandatory dimensions for every component.
- Ensure icon-only and custom controls expose meaningful accessibility labels, values, roles, enabled state, and
  keyboard focus. Preserve standard AppKit accessibility instead of replacing it with inaccessible drawing.
- Respect Reduce Motion and avoid conveying state through color, animation, sound, hover, or position alone.
- Test zoomed/larger text, long localization, RTL where supported, and keyboard-only/VoiceOver navigation without
  clipping commands, labels, table content, or critical state.

## Severity

- **Blocker** — core workflow is unusable or destructive behavior creates a credible data-loss/accessibility trap.
- **High** — a primary action is undiscoverable or inaccessible, established Mac behavior is materially broken,
  or a critical window/menu/keyboard path fails.
- **Medium** — a concrete convention, adaptability, feedback, or accessibility issue degrades a secondary path.
- **Info** — evidence-backed polish with low user impact; omit subjective preference.

## Output

Return findings first, ordered by severity. For every finding include:

1. `file:line` or the observed runtime surface.
2. The affected macOS HIG topic.
3. The concrete user impact and why the evidence proves it.
4. The smallest practical fix and the specialized implementation skill, if relevant.

If no code-detectable issues remain, say so explicitly and state what was not verified, such as runtime layout,
VoiceOver behavior, localization expansion, or visual hierarchy without screenshots.
