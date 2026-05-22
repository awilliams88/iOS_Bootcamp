# Module 04 — SwiftUI

SwiftUI is Apple's declarative UI framework introduced in 2019 and now the primary way to build new iOS, macOS, watchOS, and tvOS applications. This module covers everything from the foundational `View` protocol to performance optimization and UIKit interoperability.

---

## Topics

| # | File | Description |
|---|------|-------------|
| 01 | [View Protocol & State Management](01-view-protocol-and-state.md) | View protocol, @State, @Binding, @ObservedObject, @StateObject, @EnvironmentObject, @Environment, @AppStorage |
| 02 | [Navigation & Lists](02-navigation-and-lists.md) | NavigationStack, programmatic navigation, List, ForEach, sheets, alerts |
| 03 | [Animations & Transitions](03-animations-and-transitions.md) | withAnimation, matchedGeometryEffect, transitions, PhaseAnimator |
| 04 | [Modifiers & Geometry](04-modifiers-and-geometry.md) | Modifier ordering, custom ViewModifier, GeometryReader, PreferenceKey |
| 05 | [Performance & Observation](05-performance-and-observation.md) | SwiftUI diffing, @Observable macro, Equatable views, Instruments |
| 06 | [UIKit Interoperability](06-uikit-interoperability.md) | UIViewRepresentable, UIHostingController, Coordinator, migration |

---

## Key Interview Themes

- **@StateObject vs @ObservedObject** — the most common pitfall question
- **Modifier ordering** — affects rendering, semantics change order matters
- **SwiftUI diffing** — struct identity vs view identity
- **@Observable (iOS 17)** — replaces ObservableObject, know the differences
- **NavigationStack** — replaces NavigationView in iOS 16+
- **Performance** — lazy stacks, Equatable, avoiding body recomputation

---

## Prerequisites

- Swift generics & protocols (Module 01)
- ARC & memory management (Module 01)
- UIKit fundamentals (Module 03) — helpful for interop section
