---
name: foundation-essentials
description: >-
  Use Foundation correctly in modern Swift — Codable encoding/decoding with
  JSONEncoder/JSONDecoder, dates with Date/Calendar and the FormatStyle API,
  files and paths with URL/FileManager, UserDefaults, Measurement/units, and
  Data. Use when parsing or producing JSON, formatting dates/numbers/currency,
  reading or writing files, persisting small key-value settings, or working with
  URLs on any Apple platform (iOS 26+, macOS 26+, watchOS 26+).
---

# Foundation Essentials

Foundation provides the building blocks beneath SwiftUI/UIKit: serialization, dates, files, and formatting.
Prefer the modern Swift APIs (`Codable`, `FormatStyle`, `URL`-based paths) over legacy ones
(`NSDateFormatter` strings, `NSKeyedArchiver`, raw path strings).

## Codable — JSON in and out

```swift
struct Article: Codable, Identifiable {
    let id: UUID
    let title: String
    let publishedAt: Date
    let tags: [String]
}

// Decode
let decoder = JSONDecoder()
decoder.dateDecodingStrategy = .iso8601
decoder.keyDecodingStrategy = .convertFromSnakeCase   // published_at -> publishedAt
let article = try decoder.decode(Article.self, from: data)

// Encode
let encoder = JSONEncoder()
encoder.dateEncodingStrategy = .iso8601
encoder.outputFormatting = [.prettyPrinted, .sortedKeys]
let data = try encoder.encode(article)
```

### Custom keys & nested/odd JSON

```swift
struct User: Codable {
    let name: String
    let avatar: URL
    enum CodingKeys: String, CodingKey {
        case name = "full_name"
        case avatar = "avatar_url"
    }
}
```

For irregular shapes, implement `init(from:)`/`encode(to:)` with `container(keyedBy:)`. Keep custom logic minimal —
most APIs are handled by strategies above.

### Tips

- Decode dates with a matching `dateDecodingStrategy` (`.iso8601`, `.secondsSince1970`, or `.custom`).
- Make optionals truly optional in JSON; a missing non-optional key throws.
- Decode errors are descriptive — print the `DecodingError` to see the exact key path that failed.

## Dates, calendars & formatting

Use `Date` for instants, `Calendar` for components, and `FormatStyle` for display (not `DateFormatter` strings).

```swift
let now = Date.now

// Display (locale-aware, no format strings):
now.formatted(date: .abbreviated, time: .shortened)        // "Jun 11, 2026 at 3:04 PM"
now.formatted(.dateTime.year().month(.wide).day())          // "June 11, 2026"

// Numbers / currency / percent:
1234.5.formatted(.number.precision(.fractionLength(2)))     // "1,234.50"
9.99.formatted(.currency(code: "USD"))                      // "$9.99"
0.42.formatted(.percent)                                    // "42%"

// Relative:
RelativeDateTimeFormatter().localizedString(for: past, relativeTo: .now)  // "2 hours ago"
```

Date math via `Calendar`:

```swift
let cal = Calendar.current
let tomorrow = cal.date(byAdding: .day, value: 1, to: .now)!
let comps = cal.dateComponents([.year, .month, .day], from: .now)
```

Parse a known string format with `Date.ParseStrategy` / ISO8601:

```swift
let date = try Date("2026-06-11T15:00:00Z", strategy: .iso8601)
```

## Files & paths — use URL, not String paths

```swift
let fm = FileManager.default
let docs = URL.documentsDirectory                      // app's Documents dir
let file = docs.appending(path: "notes.json")

try data.write(to: file, options: .atomic)
let loaded = try Data(contentsOf: file)
let exists = fm.fileExists(atPath: file.path())
try fm.createDirectory(at: docs.appending(path: "cache"), withIntermediateDirectories: true)
```

Common directories: `.documentsDirectory`, `.cachesDirectory`, `.applicationSupportDirectory`, `.temporaryDirectory`.
Use Documents for user data, Caches/Temporary for regenerable data.

## UserDefaults — small key-value settings

For lightweight preferences only (flags, last-selected tab). Not for large or sensitive data.

```swift
UserDefaults.standard.set(true, forKey: "didOnboard")
let didOnboard = UserDefaults.standard.bool(forKey: "didOnboard")
```

In SwiftUI, bind directly with `@AppStorage("didOnboard") var didOnboard = false`.
For secrets/tokens use the Keychain, not UserDefaults; use `apple-keychain-passkeys` for implementation. For app
data use SwiftData (see `swiftdata-persistence`).

## Measurement & units

```swift
let distance = Measurement(value: 5, unit: UnitLength.kilometers)
distance.converted(to: .miles)                 // 3.107 mi
distance.formatted()                            // locale-aware "5 km"
```

## Quick reference

| Task | API |
| --- | --- |
| JSON ↔ model | `Codable` + `JSONDecoder`/`JSONEncoder` (set `dateDecodingStrategy`, `keyDecodingStrategy`) |
| Display date/number/currency | `value.formatted(…)` / `FormatStyle` |
| Parse date string | `Date(_, strategy:)` |
| Date math | `Calendar.current.date(byAdding:)`, `dateComponents` |
| Read/write files | `URL` + `Data(contentsOf:)` / `write(to:)`, `FileManager` |
| Standard dirs | `URL.documentsDirectory`, `.cachesDirectory`, `.temporaryDirectory` |
| Small settings | `UserDefaults` / `@AppStorage` |
| Units | `Measurement` + `Unit*` |
