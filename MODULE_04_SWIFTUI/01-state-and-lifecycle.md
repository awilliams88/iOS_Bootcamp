# SwiftUI State Management and Lifecycle

## Declarative UI

SwiftUI describes UI as a function of state.

```text
State changes -> body recompute -> diff -> render update
```

## Property Wrappers

### @State
Local owned state.

### @Binding
Two-way reference to external state.

### @ObservedObject
Observe externally owned reference model.

### @StateObject
Own reference model lifecycle.

### @EnvironmentObject
Dependency from environment.

## Observation Framework
Modern Swift observation reduces boilerplate.

## Interview Answer

SwiftUI renders from state. Choosing the correct ownership wrapper is critical to correctness and performance.
