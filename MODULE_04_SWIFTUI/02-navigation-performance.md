# Navigation and Performance

## NavigationStack

Modern SwiftUI navigation API.

## task modifier

Run async work tied to view lifecycle.

```swift
.task {
    await loadData()
}
```

## Performance Pitfalls

- unstable identity
- heavy body computation
- incorrect state ownership
- excessive view recomputation

## UIKit Interoperability

Use UIViewRepresentable / UIViewControllerRepresentable.

## Interview Answer

SwiftUI performance depends heavily on correct state ownership, stable identity, and minimizing unnecessary recomputation.
