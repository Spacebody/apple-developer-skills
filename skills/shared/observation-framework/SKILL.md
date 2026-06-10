---
name: observation-framework
description: >-
  Use the Observation framework for SwiftUI state — mark model classes with the
  @Observable macro, let SwiftUI track only the properties a view reads, and
  migrate off the legacy ObservableObject/@Published/@StateObject pattern. Use
  when building observable model/view-model classes, deciding between @State,
  @Observable, @Bindable, and @Environment, fixing views that don't update or
  update too often, or modernizing pre-iOS-17 code on any Apple platform
  (iOS 26+, macOS 26+, watchOS 26+).
---

# Observation Framework

The Observation framework (`@Observable`) is the modern way to make reference-type models drive SwiftUI.
It replaces `ObservableObject` + `@Published`. SwiftUI tracks exactly the properties a view *reads* in `body`,
so views update only when those change — more correct and more efficient than the old pattern.

> **Versions.** `@Observable` is the default for iOS 17+ and remains the standard through iOS 26/27. Use
> `ObservableObject` only when deploying to iOS 16 or earlier.

## Define an observable model

```swift
import Observation

@Observable
final class CartModel {
    var items: [Item] = []
    var coupon: String?
    var total: Decimal { items.reduce(0) { $0 + $1.price } }   // computed; tracked when read
}
```

No `@Published`. No `ObservableObject` conformance. Mark properties you don't want tracked with
`@ObservationIgnored`.

## Use it in SwiftUI

Pick the property wrapper by **ownership**, not by type:

| Wrapper | Use when |
| --- | --- |
| `@State` | The view **owns/creates** the model instance (its lifetime = the view's) |
| *(plain `let`/`var`)* | The model is **passed in** and the view only reads it |
| `@Bindable` | The view needs **two-way bindings** (`$model.prop`) to a passed-in model |
| `@Environment` | The model is injected via the environment |

```swift
struct CartScreen: View {
    @State private var cart = CartModel()          // view owns it

    var body: some View {
        VStack {
            CartTotals(cart: cart)                  // pass by reference; child reads only
            CouponField(cart: cart)
        }
    }
}

struct CartTotals: View {
    let cart: CartModel                             // just reads → no wrapper needed
    var body: some View { Text(cart.total, format: .currency(code: "USD")) }
}

struct CouponField: View {
    @Bindable var cart: CartModel                   // needs a Binding
    var body: some View { TextField("Coupon", text: $cart.coupon.bound) }
}
```

> With `@Observable`, passing a model to a child as a plain property is fine — the child still re-renders when the
> specific properties it reads change. You do **not** need `@ObservedObject`.

## Inject through the environment

```swift
// Provide:
ContentView().environment(cartModel)

// Consume:
struct SomeView: View {
    @Environment(CartModel.self) private var cart
    var body: some View { Text("\(cart.items.count) items") }
}
```

For bindings from an environment model, re-declare locally with `@Bindable`:

```swift
@Environment(CartModel.self) private var cart
var body: some View {
    @Bindable var cart = cart
    Toggle("Gift wrap", isOn: $cart.giftWrap)
}
```

## Migrating from ObservableObject

| Old | New |
| --- | --- |
| `class M: ObservableObject` | `@Observable final class M` |
| `@Published var x` | `var x` (plain) |
| `@StateObject private var m = M()` | `@State private var m = M()` |
| `@ObservedObject var m` | plain `let m` (or `@Bindable var m` for bindings) |
| `@EnvironmentObject var m` | `@Environment(M.self) private var m` |
| `.environmentObject(m)` | `.environment(m)` |

## Why views update more precisely

The old `objectWillChange` fired on *any* `@Published` change, re-rendering every observing view. `@Observable`
tracks per-property reads inside `body`, so a view that reads only `cart.coupon` won't re-render when
`cart.items` changes. Net effect: fewer redundant updates, fewer bugs.

## Pitfalls

- **Using `@StateObject`/`@ObservedObject` with an `@Observable` type** — they don't apply; use `@State`/plain/`@Bindable`.
- **Wanting a binding from a passed-in model** — switch the property to `@Bindable`.
- **Mutating from a background thread** — keep UI-facing `@Observable` models on the `@MainActor` (see `swift-concurrency`).
- **Forgetting it's a reference type** — `@State var m = Model()` keeps one instance across re-renders; that's intended.

## Quick reference

| Need | Do |
| --- | --- |
| Make a model observable | `@Observable final class` |
| Exclude a property | `@ObservationIgnored` |
| View owns the model | `@State private var m = Model()` |
| View only reads passed-in model | plain `let m` |
| View needs `$` bindings | `@Bindable var m` |
| Inject / read via environment | `.environment(m)` / `@Environment(Model.self)` |
