# macOS performance source map

Use current Apple documentation and the installed Xcode help because instrument availability and CLI options change.

## Measurement workflow

- [Testing and performance](https://developer.apple.com/documentation/technologyoverviews/testing-and-performance):
  continuous measurement across launch, memory, SwiftUI updates, CPU stalls, blocked work, hangs, graphics, energy,
  and concurrency.
- [Instruments tutorials](https://developer.apple.com/tutorials/instruments): guided hang analysis and busy-main-
  thread diagnosis.
- [Writing and running performance tests](https://developer.apple.com/documentation/xcode/writing-and-running-performance-tests):
  XCTest metrics, baselines, variance, and profiling a failing performance test.
- [Profile, fix, and verify](https://developer.apple.com/videos/play/wwdc2026/268/): current Instruments workflow
  using Swift Concurrency, Time Profiler, System Trace, comparison, and verification.

## Responsiveness and CPU

- [Improving app responsiveness](https://developer.apple.com/documentation/xcode/improving-app-responsiveness):
  Hangs, Time Profiler, CPU Profiler, Hitches, thread state, field diagnostics, and Xcode Organizer.
- [Understanding hangs in your app](https://developer.apple.com/documentation/xcode/understanding-hangs-in-your-app):
  busy versus blocked main-thread hangs and main-run-loop behavior.
- [Analyzing CPU usage with Processor Trace](https://developer.apple.com/documentation/xcode/analyzing-cpu-usage-with-processor-trace):
  low-overhead branch-level traces on supported recent Apple silicon and Xcode versions.
- [Reducing launch time](https://developer.apple.com/documentation/xcode/reducing-your-app-s-launch-time):
  App Launch, time profiles, thread-state traces, dyld work, and Points of Interest.

## Memory, storage, and correctness tools

- [Gathering information about memory use](https://developer.apple.com/documentation/xcode/gathering-information-about-memory-use):
  Xcode memory report/graph, allocation stack traces, Allocations, and generation marks.
- [Making changes to reduce memory use](https://developer.apple.com/documentation/xcode/making-changes-to-reduce-memory-use):
  unreachable leaks, retain cycles, unused live memory, and the limits of leak detection.
- [Diagnosing memory, thread, and crash issues early](https://developer.apple.com/documentation/xcode/diagnosing-memory-thread-and-crash-issues-early):
  Address/Thread/Undefined Behavior sanitizers, Main Thread Checker, and their measurement overhead.
- [Reducing disk writes](https://developer.apple.com/documentation/xcode/reducing-disk-writes): File Activity,
  filesystem events, physical I/O latency, Organizer reports, and performance tests.

## Energy and production evidence

- [Energy Efficiency Guide for Mac Apps](https://developer.apple.com/library/archive/documentation/Performance/Conceptual/power_efficiency_guidelines_osx/):
  macOS-specific idle behavior, App Nap, timers, CPU, I/O, Activity Monitor, and Instruments. Treat this archived
  guide as stable background and verify API details in current documentation.
- [MetricKit](https://developer.apple.com/documentation/metrickit): performance and diagnostic reports from real
  macOS and Mac Catalyst users, including the macOS 27 asynchronous report APIs.
- [`OSLog.Category.pointsOfInterest`](https://developer.apple.com/documentation/os/oslog/category/pointsofinterest):
  signpost/Points of Interest categorization for app-defined performance intervals.
