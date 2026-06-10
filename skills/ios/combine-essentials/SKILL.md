---
name: combine-essentials
description: >-
  Use Apple's Combine framework for reactive streams — Publishers and
  Subscribers, common operators (map/filter/debounce/removeDuplicates/combineLatest),
  @Published, sink and assign, AnyCancellable lifetime management, and bridging
  Combine with async/await. Also explains when NOT to use Combine and to prefer
  async/await + Observation instead. Use when working with existing Combine code,
  debouncing/throttling inputs, observing NotificationCenter/Timer publishers, or
  reactive pipelines on iOS 26+.
---

# Combine Essentials

Combine is Apple's reactive framework for processing values over time. In modern apps its role has **narrowed**:
`async`/`await` + `AsyncSequence` (see `swift-concurrency`) and the `@Observable` Observation framework now cover
most cases Combine used to. Reach for Combine for **event-stream pipelines** (debounced search, merging multiple
sources) and for maintaining existing Combine code.

> **When NOT to use Combine.** For view-model state use `@Observable`, not `ObservableObject`/`@Published`. For
> one-shot async work use `async`/`await`. For async iteration prefer `AsyncSequence`/`AsyncStream`. Don't
> introduce Combine into a new codebase that has no reactive-pipeline need.

## Core concepts

- **Publisher** — emits values + a completion (finished/failure). `Output`/`Failure` types.
- **Operator** — transforms a publisher (returns a new publisher).
- **Subscriber** — receives values; `sink` and `assign` are the built-ins.
- **AnyCancellable** — the subscription's lifetime token; storing it keeps the stream alive.

## A typical pipeline: debounced search

```swift
import Combine

@MainActor
final class SearchModel: ObservableObject {        // Combine pairs with ObservableObject here
    @Published var query = ""
    @Published private(set) var results: [Result] = []
    private var cancellables = Set<AnyCancellable>()

    init(api: SearchAPI) {
        $query
            .debounce(for: .milliseconds(300), scheduler: RunLoop.main)  // wait for typing to settle
            .removeDuplicates()
            .filter { $0.count >= 2 }
            .map { api.searchPublisher(for: $0).replaceError(with: []) } // -> Publisher of Publishers
            .switchToLatest()                                            // cancel stale in-flight searches
            .receive(on: RunLoop.main)
            .assign(to: &$results)                                       // drive @Published output
    }
}
```

This is Combine's sweet spot: debounce + dedupe + cancel-previous + merge — concise as a pipeline.

## sink & assign

```swift
publisher
    .sink(
        receiveCompletion: { completion in /* .finished or .failure(error) */ },
        receiveValue: { value in /* handle */ }
    )
    .store(in: &cancellables)            // ← REQUIRED, or the subscription is deallocated immediately

// assign output straight to a property:
publisher.assign(to: \.title, on: self).store(in: &cancellables)
```

**The #1 Combine bug**: forgetting `.store(in:)` (or `assign(to: &$x)`), so the `AnyCancellable` is released and
the stream never delivers. Always retain cancellables.

## Common operators

| Operator | Purpose |
| --- | --- |
| `map` / `tryMap` | Transform each value |
| `filter` | Drop values failing a predicate |
| `removeDuplicates` | Skip consecutive equal values |
| `debounce` / `throttle` | Rate-limit (settle vs. sample) |
| `combineLatest` / `merge` / `zip` | Combine multiple publishers |
| `flatMap` / `switchToLatest` | Map to publishers; flatten / keep-latest |
| `replaceError` / `catch` | Handle/recover from failures |
| `receive(on:)` / `subscribe(on:)` | Choose scheduler (e.g. main thread for UI) |

## Built-in publishers worth knowing

```swift
NotificationCenter.default.publisher(for: UIApplication.didBecomeActiveNotification)
Timer.publish(every: 1, on: .main, in: .common).autoconnect()
URLSession.shared.dataTaskPublisher(for: url)        // (prefer async data(for:) for one-shot fetches)
Just(value)                                          // emits once
$someProperty                                         // @Published projected value
```

## Bridging Combine ↔ async/await

```swift
// Publisher → async value (first value):
let value = try await publisher.values.first { _ in true }

// AsyncSequence over a publisher:
for await v in publisher.values { handle(v) }

// async work → Combine: wrap in Future
func loadFuture() -> Future<Data, Error> {
    Future { promise in Task { do { promise(.success(try await load())) } catch { promise(.failure(error)) } } }
}
```

`.values` gives you an `AsyncSequence` view of any publisher — the cleanest bridge when consuming a publisher in
async code.

## Pitfalls

- **Not storing cancellables** → stream dies silently. Use `.store(in:)` / `assign(to: &$x)`.
- **Retain cycles** in `sink` closures → use `[weak self]`.
- **Updating UI off-main** → insert `.receive(on: RunLoop.main)` before UI sinks/assigns.
- **Choosing Combine for plain state** → use `@Observable`. Combine is for *streams/pipelines*, not general state.

## Quick reference

| Need | Use |
| --- | --- |
| Observe a property as a stream | `@Published` + `$property` |
| Debounced search input | `debounce` + `removeDuplicates` + `switchToLatest` |
| Combine multiple sources | `combineLatest` / `merge` / `zip` |
| Receive values | `.sink { }` / `.assign(to:on:)` (always `.store(in:)`) |
| Keep subscription alive | `Set<AnyCancellable>` + `.store(in:)` |
| UI-thread delivery | `.receive(on: RunLoop.main)` |
| Bridge to async | `publisher.values` (AsyncSequence) |
| General view state | **Not Combine** — use `@Observable` |
