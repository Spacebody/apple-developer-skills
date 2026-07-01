---
name: swiftdata-persistence
description: >-
  Persist app data with SwiftData — define models with @Model, set up
  ModelContainer/ModelContext, query and auto-update SwiftUI views with @Query,
  filter with #Predicate, sort with SortDescriptor, model relationships and
  delete rules with @Relationship, configure storage with @Attribute, and handle
  schema migrations. Use when storing local app data, building CRUD features, or
  choosing/replacing Core Data on any Apple platform (iOS 26+, macOS 26+,
  watchOS 26+). SwiftData is the default persistence framework for new apps.
---

# SwiftData Persistence

SwiftData is the modern, Swift-native persistence framework (built on Core Data). Use it instead of raw
Core Data for new apps. It integrates directly with SwiftUI via `@Query`.

> **Versions.** Targets iOS 26 and forward to iOS 27. Prefer SwiftData for new code; only drop to Core Data for
> features SwiftData still lacks (e.g. certain advanced fetched-results or NSPersistentCloudKit edge cases).

## 1. Define models with @Model

```swift
import SwiftData

@Model
final class Trip {
    var name: String
    var startDate: Date
    @Attribute(.unique) var code: String          // unique constraint
    @Relationship(deleteRule: .cascade, inverse: \Activity.trip)
    var activities: [Activity] = []

    init(name: String, startDate: Date, code: String) {
        self.name = name
        self.startDate = startDate
        self.code = code
    }
}

@Model
final class Activity {
    var title: String
    var trip: Trip?
    init(title: String) { self.title = title }
}
```

- A `@Model` class gets persistence + observation automatically (it's also observable for SwiftUI).
- `@Attribute(.unique)` enforces uniqueness; `.externalStorage` stores large `Data`/blobs as files.
- `@Relationship(deleteRule:inverse:)` — `.cascade` deletes children, `.nullify` (default) clears the link.
- Mark a property `@Transient` to exclude it from storage.

## 2. Create the container at app launch

```swift
@main
struct TripsApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
        .modelContainer(for: [Trip.self, Activity.self])   // inject container into the environment
    }
}
```

For custom config (in-memory tests, CloudKit, file location):

```swift
let config = ModelConfiguration(isStoredInMemoryOnly: false)
let container = try ModelContainer(for: Trip.self, configurations: config)
```

## 3. Query in SwiftUI with @Query (auto-updating)

```swift
struct TripList: View {
    @Environment(\.modelContext) private var context
    @Query(sort: \Trip.startDate, order: .reverse) private var trips: [Trip]

    var body: some View {
        List(trips) { trip in Text(trip.name) }
    }
}
```

`@Query` re-runs and refreshes the view automatically when matching data changes. Filter and sort inline:

```swift
@Query(filter: #Predicate<Trip> { $0.name.contains("Paris") },
       sort: [SortDescriptor(\.startDate)])
private var parisTrips: [Trip]
```

## 4. Create / update / delete via ModelContext

```swift
@Environment(\.modelContext) private var context

func add() {
    let trip = Trip(name: "Tokyo", startDate: .now, code: "TYO")
    context.insert(trip)            // autosave is on by default; explicit save rarely needed
}

func remove(_ trip: Trip) {
    context.delete(trip)
}

// Mutating a fetched model's property persists automatically:
trip.name = "Tokyo 2026"
```

Call `try context.save()` explicitly only if you disabled autosave or need a checkpoint before a risky op.

## 5. Programmatic fetches (outside a view)

```swift
let descriptor = FetchDescriptor<Trip>(
    predicate: #Predicate { $0.startDate > .now },
    sortBy: [SortDescriptor(\.startDate)]
)
let upcoming = try context.fetch(descriptor)
```

`#Predicate` is type-safe and compiled — supports comparisons, `contains`, `starts(with:)`, boolean logic, and
relationship traversal. Avoid unsupported closures inside it (no arbitrary Swift function calls).

## 6. Concurrency

`ModelContext` is **not** Sendable — don't pass one across actors. For background work, use a `ModelActor`:

```swift
@ModelActor
actor ImportActor {
    func importTrips(_ payloads: [TripPayload]) throws {
        for p in payloads { modelContext.insert(Trip(name: p.name, startDate: p.date, code: p.code)) }
        try modelContext.save()
    }
}
```

The main-thread context backing `@Query`/views lives on the `@MainActor`.

## 7. Migrations

For additive changes (new optional properties, new models) SwiftData migrates automatically. For breaking
changes, define a `VersionedSchema` per version and a `SchemaMigrationPlan` with `.lightweight` or `.custom`
migration stages, then pass it to the `ModelContainer`.

```swift
enum SchemaV1: VersionedSchema { static var models: [any PersistentModel.Type] { [Trip.self] } /* … */ }
enum SchemaV2: VersionedSchema { /* … */ }

enum MigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] { [SchemaV1.self, SchemaV2.self] }
    static var stages: [MigrationStage] { [.lightweight(fromVersion: SchemaV1.self, toVersion: SchemaV2.self)] }
}
```

## Pitfalls

- **No `init` on a `@Model`** → required stored properties must be initialized; add an `init`.
- **Passing `ModelContext`/models across actors** → use `@ModelActor`; fetch by `PersistentIdentifier` instead of sharing instances.
- **Unsupported expressions in `#Predicate`** → keep predicates to comparisons/string ops/logic.
- **Forgetting `.modelContainer`** → `@Query`/`modelContext` are nil/crash without it in the environment.

## Quick reference

| Need | API |
| --- | --- |
| Define entity | `@Model final class` |
| Unique / large blob | `@Attribute(.unique)` / `.externalStorage` |
| Relationship + delete rule | `@Relationship(deleteRule:inverse:)` |
| Inject store | `.modelContainer(for:)` |
| Auto-updating view query | `@Query(filter:sort:)` |
| Filter | `#Predicate<T> { … }` |
| Sort | `SortDescriptor(\.key)` |
| Insert/delete | `context.insert(_:)` / `context.delete(_:)` |
| Manual fetch | `FetchDescriptor` + `context.fetch(_:)` |
| Background writes | `@ModelActor` |
