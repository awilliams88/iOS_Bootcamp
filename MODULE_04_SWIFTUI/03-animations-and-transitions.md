# SwiftUI — Animations & Transitions

---

## withAnimation

The simplest way to animate state changes:

```swift
@State private var isExpanded = false

Button("Toggle") {
    withAnimation(.spring(response: 0.3, dampingFraction: 0.7)) {
        isExpanded.toggle()
    }
}

// View responds declaratively
RoundedRectangle(cornerRadius: 12)
    .frame(height: isExpanded ? 200 : 60)
```

---

## Implicit vs Explicit Animations

```swift
// Implicit — .animation modifier animates any change to watched value
Text("Hello")
    .scaleEffect(isLarge ? 2 : 1)
    .animation(.easeInOut, value: isLarge)  // Only animates when isLarge changes

// Explicit — withAnimation wraps the state mutation
withAnimation(.easeInOut) { isLarge.toggle() }
```

> ⚠️ **Pitfall:** `.animation()` without a `value:` parameter animates ALL changes, including ones you don't intend. Always use `value:` parameter (iOS 15+).

---

## Transitions

```swift
if isVisible {
    CardView()
        .transition(.move(edge: .bottom))

    // Combined transitions
    CardView()
        .transition(.asymmetric(
            insertion: .move(edge: .trailing).combined(with: .opacity),
            removal: .move(edge: .leading).combined(with: .opacity)
        ))
}
```

### Custom Transitions (iOS 17+)

```swift
struct ScaleAndFadeTransition: Transition {
    func body(content: Content, phase: TransitionPhase) -> some View {
        content
            .scaleEffect(phase.isIdentity ? 1 : 0.5)
            .opacity(phase.isIdentity ? 1 : 0)
    }
}

// Usage
view.transition(ScaleAndFadeTransition())
```

---

## matchedGeometryEffect

Creates hero animations between views in different parts of the hierarchy:

```swift
@Namespace private var heroNamespace
@State private var isExpanded = false

if isExpanded {
    ExpandedCard()
        .matchedGeometryEffect(id: "card", in: heroNamespace)
} else {
    ThumbnailCard()
        .matchedGeometryEffect(id: "card", in: heroNamespace)
}
```

> 💡 **Tip:** Both views must share the same `@Namespace` and the same `id`. SwiftUI animates between the two frames automatically.

> ⚠️ **Pitfall:** Only one view with a given `id` should be visible at a time, or layout becomes undefined.

---

## PhaseAnimator & KeyframeAnimator (iOS 17)

```swift
// PhaseAnimator — cycles through phases
PhaseAnimator([false, true]) { phase in
    Circle()
        .scaleEffect(phase ? 1.5 : 1.0)
        .opacity(phase ? 0.5 : 1.0)
} animation: { phase in
    .spring(duration: 0.5)
}

// KeyframeAnimator — precise keyframe control
KeyframeAnimator(initialValue: AnimationState()) { value in
    Circle()
        .scaleEffect(value.scale)
        .rotationEffect(.degrees(value.rotation))
} keyframes: { _ in
    KeyframeTrack(\.scale) {
        SpringKeyframe(1.5, duration: 0.3)
        CubicKeyframe(1.0, duration: 0.3)
    }
    KeyframeTrack(\.rotation) {
        LinearKeyframe(360, duration: 0.6)
    }
}
```

---

## Interview Q&A

**Q: What's the difference between implicit and explicit animations in SwiftUI?**  
A: Implicit animations use the `.animation(_:value:)` modifier and automatically animate changes to the specified value wherever it's used in that view's rendering. Explicit animations wrap state mutations in `withAnimation {}` and animate all resulting view changes. Explicit gives more control over timing; implicit is simpler but can animate unintended changes.

**Q: How does matchedGeometryEffect work?**  
A: It tells SwiftUI that two views in different positions represent the same conceptual element. SwiftUI computes the frame difference and animates the transition between them. The `@Namespace` provides a shared identity scope.

**Q: When would you use PhaseAnimator vs KeyframeAnimator?**  
A: `PhaseAnimator` for simple looping animations through discrete states. `KeyframeAnimator` for precise, complex multi-property animations with specific timing curves — like a character animation or a loading indicator.
