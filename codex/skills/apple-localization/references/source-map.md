# Apple localization source map

Use current Apple documentation because Xcode extraction and catalog tooling evolve with releases.

## APIs and extraction

- [Preparing app text for translation](https://developer.apple.com/documentation/xcode/preparing-your-apps-text-for-translation):
  extractable APIs, translator comments, tables, bundles, deferred resources, and reusable UI inputs.
- [`LocalizedStringResource`](https://developer.apple.com/documentation/foundation/localizedstringresource):
  deferred localizable content carrying key, default value, table, bundle, locale, and comment.
- [`String(localized:)`](https://developer.apple.com/documentation/swift/string/init(localized:table:bundle:locale:comment:)):
  immediate localized strings outside SwiftUI localizable initializers.
- [`LocalizedStringKey`](https://developer.apple.com/documentation/swiftui/localizedstringkey): SwiftUI localized
  key behavior and interpolation.
- [Generated localizable symbols](https://developer.apple.com/documentation/xcode/using-generated-localizable-symbols-in-your-code):
  typed catalog properties and functions.

## Catalogs, variations, and regions

- [Localizing and varying text with a String Catalog](https://developer.apple.com/documentation/xcode/localizing-and-varying-text-with-a-string-catalog):
  catalog workflow and plural/device variations.
- [Localizing strings that contain plurals](https://developer.apple.com/documentation/xcode/localizing-strings-that-contain-plurals):
  locale-specific plural categories and migration background.
- [Creating width and device variants](https://developer.apple.com/documentation/xcode/creating-width-and-device-variants-of-strings):
  current device variations and the boundary with adaptable UI.
- [Choosing localization regions and scripts](https://developer.apple.com/documentation/xcode/choosing-localization-regions-and-scripts):
  language-region-script specificity.
- [Preparing dates and numbers for translation](https://developer.apple.com/documentation/xcode/preparing-dates-numbers-with-formatters):
  typed locale formatting and interpolation.

## RTL and testing

- [Right to left](https://developer.apple.com/design/human-interface-guidelines/right-to-left): semantic mirroring
  and content that should retain physical direction.
- [`LayoutDirection`](https://developer.apple.com/documentation/swiftui/layoutdirection): SwiftUI environment
  direction.
- [`UIView.semanticContentAttribute`](https://developer.apple.com/documentation/uikit/uiview/semanticcontentattribute):
  UIKit semantic mirroring behavior.
- [Preparing your interface for localization](https://developer.apple.com/documentation/xcode/preparing-your-interface-for-localization):
  adaptable layout and resource preparation.
- [Previewing localizations](https://developer.apple.com/documentation/xcode/previewing-localizations): previews and
  pseudolanguages.
- [Testing localizations when running your app](https://developer.apple.com/documentation/xcode/testing-localizations-when-running-your-app):
  scheme language, region, and diagnostic options.
- [Organizing tests to improve feedback](https://developer.apple.com/documentation/xcode/organizing-tests-to-improve-feedback):
  localization settings in test-plan configurations.
