---
name: macos-performance-profiling
description: Measure, diagnose, and verify native macOS and Mac Catalyst performance with Xcode, Instruments, Activity Monitor, MetricKit, and repeatable performance tests. Use for slow launch, high CPU, hangs or beachballs, memory growth or leaks, excessive disk I/O, energy impact, Swift concurrency contention, SwiftUI update churn, or regressions that require runtime evidence. Do not use for ordinary build failures, test assertion triage, crash-only debugging, iOS Simulator profiling, or speculative code-only optimization without a reproducible performance symptom.
---

# macOS Performance Profiling

Measure before changing code. Treat a trace as evidence of where time or resources went, then confirm causality
with the smallest change and a comparable second capture.

## Boundaries

- Use `build-macos-apps:build-run-debug` when the app does not build, launch, or attach normally.
- Use `build-macos-apps:test-triage` for functional test failures and `build-macos-apps:telemetry` for durable
  `Logger` events or signposts. This skill owns performance experiments and trace interpretation.
- Use `swift-concurrency` after evidence identifies actor contention, priority inversion, or task-structure issues.
- Use `build-macos-apps:liquid-glass` or `build-macos-apps:view-refactor` only after a UI trace points to those areas.

Read [references/source-map.md](references/source-map.md) for current Apple tools, platform constraints, and
primary-source links.

## Establish a measurement contract

1. Define one user-visible symptom and a short deterministic reproduction: launch, open document, resize window,
   search, import, idle background period, or another named transaction.
2. Record the app version/commit, macOS and Xcode versions, Mac model, build configuration, input data, and whether
   the process was cold or warm. Use the same conditions after the fix.
3. Prefer an optimized, properly symbolicated build when evaluating shipping performance. Do not compare a
   sanitizer-enabled run with a normal run; instrumentation overhead changes timing and memory.
4. Capture a baseline before editing. Repeat noisy experiments and compare distributions or representative runs,
   not one unusually good sample.
5. Save the baseline and verification traces when possible, and note the exact transaction range in each trace.

## Choose the narrowest tool

| Symptom | Start with | Escalate when needed |
| --- | --- | --- |
| Slow launch | App Launch template, Xcode launch metrics | Time Profiler, dyld activity, Points of Interest |
| Sustained CPU or slow action | Time Profiler | CPU Profiler, System Trace, Processor Trace on supported Apple silicon |
| Beachball, freeze, delayed input | Hangs with Time Profiler/CPU Profiler | System Trace; distinguish busy main thread from blocked main thread |
| Swift task stalls or contention | Swift Concurrency instrument + Time Profiler | System Trace, signposts, actor/task redesign |
| Memory growth | Xcode memory report/graph + Allocations | Generations, VM Tracker, `vmmap`; use Leaks only for unreachable allocations |
| Suspected retain cycle or leak | Debug memory graph + Leaks | Allocation stack traces and object ownership audit |
| Disk churn or slow file work | File Activity / Filesystem Activity | Disk I/O latency, signposts around the transaction |
| High idle energy impact | Activity Monitor Energy and wake/CPU/I/O evidence | Time Profiler, File/Network activity, timer and App Nap audit |
| SwiftUI redraw or interaction hitch | SwiftUI/Hitches instruments when available | Time Profiler, view invalidation and model ownership audit |
| Metal-heavy rendering | Metal debugger or Game Performance template | GPU, VM Tracker, Metal resource tools |

Do not select an instrument merely because its name sounds related. Confirm it supports the target macOS/Xcode
combination and that it records the resource involved in the reproduction.

## Diagnose from evidence

- In sampling profiles, narrow the time range to the slow transaction, hide system libraries only after confirming
  symbols are present, and inspect both heavy stacks and callers. A hot leaf may be a symptom of repeated calls.
- For hangs, determine whether the main thread is running expensive work or blocked on a lock, file, IPC, network,
  or another thread. Moving blocked work into an unstructured task does not remove the dependency.
- Do not default to `Task.detached`. It drops structured cancellation and actor inheritance, can change priority and
  task-local behavior, requires safe `Sendable` boundaries, and may increase peak memory by keeping input and output
  alive together. Use concurrency only after a trace identifies independent work that can leave the main actor.
- For memory, distinguish live footprint, dirty/resident VM, allocation rate, caches, and unreachable leaks.
  Leaks reporting zero does not prove memory growth is harmless.
- For energy, look for work while the app should be idle: polling, tight timers, repeated redraw, small disk writes,
  prevent-sleep assertions, unnecessary network requests, or inappropriate quality of service.
- Add a small Points of Interest interval only when the system trace cannot identify the app transaction. Keep
  sensitive values out of signpost names and metadata.

## Repeatable memory-growth protocol

1. Mark Allocations generations at idle, after open/import, after close, and after a fixed quiescence interval;
   repeat the same cycle enough times to distinguish delayed release from a stepwise live-object increase.
2. Use Allocations and generations to answer what remains live and where it was allocated. Check for duplicated
   input data, mapped files, decompression buffers, and parsed output existing simultaneously.
3. Use Debug Memory Graph to answer who still owns reachable objects, including tasks, closures, delegates,
   observations, windows, and caches. Use Leaks for unreachable malloc allocations; zero leaks does not clear
   reachable retention.
4. If live allocations fall but process footprint does not, inspect VM Tracker or `vmmap` for mappings, dirty or
   compressed pages, graphics resources, and allocator reservations before calling it a leak.

## Fix and verify

1. State the trace-backed hypothesis and the exact evidence range, stack, allocation category, or blocked resource.
2. Make one bounded change. Preserve behavior and avoid broad architectural rewrites during diagnosis.
3. Rebuild with the same configuration and repeat the same scenario on the same class of hardware.
4. Compare both the target metric and guardrail metrics; reducing latency by greatly increasing memory or energy is
   a tradeoff, not an unqualified win.
5. Add an XCTest performance test or project benchmark when the transaction is deterministic enough to catch a
   regression. Set a baseline from repeated representative runs, not from an instrumented outlier.

For command-line capture, inspect the installed tool first with `xcrun xctrace help` and
`xcrun xctrace list templates`. Template names and recording options vary by Xcode version; do not invent a CLI
command or reuse an old option without checking the local help.

## Output

Report:

1. Symptom and reproduction.
2. Environment and measurement conditions.
3. Trace evidence with the relevant time range, stack, allocation, thread state, or metric.
4. Root-cause confidence and alternatives not yet ruled out.
5. Change made and before/after result under comparable conditions.
6. Remaining coverage gaps and the saved trace/test artifact when one exists.

If no capture was performed, label conclusions as hypotheses and give the shortest next measurement. Never report
a performance improvement from code inspection alone.
