---
name: swift-concurrency
description: >-
  Write modern Swift concurrency with async/await, Task, TaskGroup, actors,
  @MainActor, Sendable, AsyncSequence/AsyncStream, and Swift 6 strict data-race
  safety. Use when writing or refactoring asynchronous Swift code, fixing
  "data race"/"non-Sendable"/"actor-isolated" compiler errors, replacing
  completion handlers, running work off the main thread, or coordinating
  concurrent tasks on any Apple platform (iOS 26+, macOS 26+, watchOS 26+).
---

# Swift Concurrency

Modern Swift uses structured concurrency. Prefer `async`/`await` and actors over completion handlers,
`DispatchQueue`, and locks. Swift 6 enforces **data-race safety at compile time**.

> **Versions.** Targets iOS 26 (current shipping OS) and forward to iOS 27 / Xcode 27 (in beta, public release
> fall 2026). Swift 6's strict concurrency originally required heavy annotation; the Xcode 27 toolchain tightens
> data-isolation guarantees while *reducing* that annotation burden. Turn on **"Approachable Concurrency"** and
> **default main-actor isolation** (build settings / `defaultIsolation(MainActor.self)`) so app code is
> main-actor by default and you only annotate the code that intentionally runs off the main actor — the inverse
> of the older "annotate everything `@MainActor`" approach.

## When to use this skill

- Writing `async` functions or calling `await` APIs (e.g. `URLSession.data(for:)`).
- Refactoring callback/`completionHandler` code into `async`/`await`.
- Fixing compiler diagnostics: "Sending ... risks causing data races", "non-Sendable type", "main actor-isolated", "Reference to captured var in concurrently-executing code".
- Coordinating multiple concurrent operations (parallel fetches, fan-out/fan-in).
- Protecting shared mutable state.

## Core model

### async / await

```swift
func loadProfile(id: String) async throws -> Profile {
    let (data, _) = try await URLSession.shared.data(from: profileURL(id))
    return try JSONDecoder().decode(Profile.self, from: data)
}
```

`await` marks a suspension point — the function can pause and the thread is freed. Call `async` functions from
another `async` context, or from a `Task`.

### Task — entering an async context

```swift
Task {
    do { self.profile = try await loadProfile(id: "42") }
    catch { self.error = error }
}
```

- `Task { }` inherits the current actor (so a `Task` started in a `@MainActor` view runs its body on the main actor).
- `Task.detached { }` does **not** inherit context — avoid unless you specifically need to escape actor isolation.
- Store the handle (`let t = Task { … }`) to `await t.value` or `t.cancel()`.

### Structured concurrency: run work in parallel

Use `async let` for a fixed number of concurrent children:

```swift
async let user = loadUser()
async let posts = loadPosts()
let (u, p) = try await (user, posts)   // both run concurrently; awaited together
```

Use `TaskGroup` for a dynamic number:

```swift
func loadAll(ids: [String]) async throws -> [Profile] {
    try await withThrowingTaskGroup(of: Profile.self) { group in
        for id in ids { group.addTask { try await loadProfile(id: id) } }
        var result: [Profile] = []
        for try await profile in group { result.append(profile) }
        return result
    }
}
```

Children are scoped to the parent: when the scope exits (or throws), remaining children are cancelled. This is
the key advantage over unstructured `Task`s.

## Actors — protecting shared mutable state

An `actor` serializes access to its mutable state; cross-actor access is `await`-ed.

```swift
actor ImageCache {
    private var cache: [URL: Image] = [:]
    func image(for url: URL) -> Image? { cache[url] }
    func store(_ image: Image, for url: URL) { cache[url] = image }
}

let cache = ImageCache()
await cache.store(img, for: url)        // await: crossing into the actor
```

### @MainActor — UI and main-thread work

UI must run on the main actor. Annotate types or members that touch UI:

```swift
@MainActor
final class FeedModel {
    var items: [Item] = []           // mutated only on the main actor
    func refresh() async {
        let new = try? await loadAll(ids: ids)   // hops off to do async work
        items = new ?? []                         // back on main actor — safe UI update
    }
}
```

`@Observable` view models that feed SwiftUI are typically `@MainActor`. You rarely need `DispatchQueue.main.async`
anymore — annotate with `@MainActor` instead.

## Sendable — safe across isolation boundaries

A `Sendable` type is safe to pass between actors/tasks. Value types of `Sendable` members are `Sendable`
automatically; `final` classes can conform if immutable or internally synchronized.

```swift
struct Message: Sendable { let id: Int; let text: String }
```

- Prefer immutable value types crossing boundaries.
- Avoid `@unchecked Sendable` — it disables the safety check. Use only with a manually-synchronized class and a comment justifying it.

## AsyncSequence / AsyncStream — streams of values

```swift
for try await line in url.lines { print(line) }   // built-in AsyncSequence

// Bridge a callback/delegate API into an async stream:
func locations() -> AsyncStream<CLLocation> {
    AsyncStream { continuation in
        let delegate = LocationDelegate { continuation.yield($0) }
        continuation.onTermination = { _ in delegate.stop() }
        delegate.start()
    }
}
```

## Cancellation

Cancellation is cooperative — check it in long work:

```swift
for item in items {
    try Task.checkCancellation()    // throws CancellationError if cancelled
    await process(item)
}
```

In SwiftUI, prefer `.task { }` over `Task { }` for view-tied work — it auto-cancels when the view disappears.

## Pitfalls & best practices

- **Don't** capture and mutate a `var` from inside a concurrent closure — pass values in, return results out.
- **Don't** block an actor with synchronous long work; it stalls everything queued on it.
- **Don't** reach for `Task.detached` to "fix" an isolation error — fix the isolation instead.
- **Do** enable the Swift 6 language mode / strict concurrency checking and resolve warnings early.
- **Do** replace `DispatchQueue.main.async` with `@MainActor`, and GCD work queues with actors or `Task`s.
- **Do** keep async functions free of side effects on shared state unless that state is actor-isolated.

## Quick reference

| Need | Use |
| --- | --- |
| Call async API | `await` inside an `async` func or `Task` |
| Enter async from sync | `Task { }` (inherits context) / `.task { }` in SwiftUI |
| Fixed parallel work | `async let` |
| Dynamic parallel work | `withTaskGroup` / `withThrowingTaskGroup` |
| Protect mutable state | `actor` |
| Run on main thread | `@MainActor` |
| Cross-boundary value | `Sendable` type |
| Stream of async values | `AsyncSequence` / `AsyncStream` |
| Cancel | `task.cancel()`, `Task.checkCancellation()` |
