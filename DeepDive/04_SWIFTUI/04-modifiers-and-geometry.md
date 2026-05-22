# SwiftUI — Modifiers, Geometry & Preferences

---

## Modifier Ordering — The Critical Pitfall

In SwiftUI, **modifier order changes semantics**. Each modifier wraps the view in a new view — order matters.

```swift
// These produce DIFFERENT results:

// 1. Background behind the padded area
Text("Hello")
    .background(Color.red)
    .padding()

// 2. Background fills the padded area
Text("Hello")
    .padding()
    .background(Color.red)
```

```swift
// Tap area difference:
Button("Tap") { }
    .padding()
    .contentShape(Rectangle())  // ✅ Large tap area

Button("Tap") { }
    .contentShape(Rectangle())  // ❌ Small tap area — contentShape applied before padding
    .padding()
```

> ⚠️ **Pitfall:** `.clipped()`, `.clipShape()`, `.background()`, `.border()`, `.shadow()` — all behave differently depending on position relative to `.padding()`. Always think "each modifier wraps the previous result."

```swift
// Frame + background order
Text("Hi")
    .frame(width: 100, height: 50)
    .background(Color.blue)   // Blue fills the 100×50 frame

Text("Hi")
    .background(Color.blue)   // Blue only behind the text intrinsic size
    .frame(width: 100, height: 50)
```

### Mental Model

Think of modifiers as nesting Russian dolls:

```
padding(
  background(
    Text("Hello")
  )
)
```

Each modifier creates a new view that wraps the inner view.

> 🎯 **Interview Answer:** "SwiftUI modifier order matters because each modifier creates a new wrapper view. `.padding().background()` gives you a background that covers the padded area, while `.background().padding()` gives a background only behind the content. The outermost modifier sees the final frame of what's inside it."

---

## Custom ViewModifier

Extract reusable styling into a `ViewModifier`:

```swift
struct CardStyle: ViewModifier {
    var cornerRadius: CGFloat = 12
    
    func body(content: Content) -> some View {
        content
            .padding()
            .background(Color(.systemBackground))
            .cornerRadius(cornerRadius)
            .shadow(color: .black.opacity(0.1), radius: 8, x: 0, y: 4)
    }
}

extension View {
    func cardStyle(cornerRadius: CGFloat = 12) -> some View {
        modifier(CardStyle(cornerRadius: cornerRadius))
    }
}

// Usage
Text("Card Content")
    .cardStyle()

VStack { ... }
    .cardStyle(cornerRadius: 20)
```

### When to Use ViewModifier

- Reusable styling across many views
- Conditional modifiers (apply modifier only in certain conditions)
- Animated modifiers

```swift
// Conditional modifier with ViewModifier
struct ConditionalModifier<M: ViewModifier>: ViewModifier {
    let condition: Bool
    let modifier: M
    
    func body(content: Content) -> some View {
        Group {
            if condition {
                content.modifier(modifier)
            } else {
                content
            }
        }
    }
}

extension View {
    @ViewBuilder
    func `if`<Transform: View>(_ condition: Bool, transform: (Self) -> Transform) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
}

// Usage
Text("Hello")
    .if(isHighlighted) { $0.background(Color.yellow) }
```

> ⚠️ **Pitfall:** Avoid `if` modifiers that change view identity — SwiftUI will destroy and recreate the view tree. Prefer opacity or other non-structural changes for toggling.

---

## GeometryReader

`GeometryReader` provides the size and coordinate space of the parent container:

```swift
struct ProportionalView: View {
    var body: some View {
        GeometryReader { geometry in
            HStack(spacing: 0) {
                Rectangle()
                    .fill(.blue)
                    .frame(width: geometry.size.width * 0.6)
                Rectangle()
                    .fill(.red)
                    .frame(width: geometry.size.width * 0.4)
            }
        }
    }
}
```

### Coordinate Spaces

```swift
GeometryReader { geometry in
    // Local coordinate space (within this view)
    let localFrame = geometry.frame(in: .local)
    
    // Global coordinate space (full screen)
    let globalFrame = geometry.frame(in: .global)
    
    // Named coordinate space
    let namedFrame = geometry.frame(in: .named("scrollView"))
    
    Color.blue
        .onAppear {
            print("Size: \(geometry.size)")
            print("Global origin: \(globalFrame.origin)")
        }
}
```

### Safe Area

```swift
GeometryReader { geometry in
    VStack {
        Text("Top inset: \(geometry.safeAreaInsets.top)")
        Text("Bottom inset: \(geometry.safeAreaInsets.bottom)")
    }
}
```

> ⚠️ **Pitfall:** `GeometryReader` takes all available space — it expands to fill its container. Don't use it just to read size; it will break layouts. Use `background(GeometryReader {...})` to read size without affecting layout.

```swift
// ✅ Read size without affecting layout
struct SizeReader: View {
    @Binding var size: CGSize
    
    var body: some View {
        Color.clear
            .background(
                GeometryReader { geo in
                    Color.clear
                        .preference(key: SizePreferenceKey.self, value: geo.size)
                }
            )
            .onPreferenceChange(SizePreferenceKey.self) { size = $0 }
    }
}
```

---

## PreferenceKey — Communicating Up the Hierarchy

SwiftUI data flows down (parent → child). `PreferenceKey` flows data **up** (child → parent):

```swift
// Define the key
struct HeightPreferenceKey: PreferenceKey {
    static var defaultValue: CGFloat = 0
    
    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        value = max(value, nextValue())  // Keep the maximum height
    }
}

// Child sets the preference
struct ChildView: View {
    var body: some View {
        Text("Hello")
            .background(
                GeometryReader { geo in
                    Color.clear.preference(
                        key: HeightPreferenceKey.self,
                        value: geo.size.height
                    )
                }
            )
    }
}

// Parent reads the preference
struct ParentView: View {
    @State private var childHeight: CGFloat = 0
    
    var body: some View {
        VStack {
            ChildView()
            Text("Child height: \(childHeight)")
        }
        .onPreferenceChange(HeightPreferenceKey.self) { height in
            childHeight = height
        }
    }
}
```

### Scroll Offset via PreferenceKey

A classic pattern to read scroll position:

```swift
struct ScrollOffsetKey: PreferenceKey {
    static var defaultValue: CGFloat = 0
    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) {
        value = nextValue()
    }
}

struct ScrollOffsetView: View {
    @State private var offset: CGFloat = 0
    
    var body: some View {
        ScrollView {
            VStack {
                GeometryReader { geo in
                    Color.clear
                        .preference(
                            key: ScrollOffsetKey.self,
                            value: geo.frame(in: .named("scroll")).minY
                        )
                }
                .frame(height: 0)
                
                ForEach(0..<50) { i in
                    Text("Item \(i)").frame(maxWidth: .infinity).padding()
                }
            }
        }
        .coordinateSpace(name: "scroll")
        .onPreferenceChange(ScrollOffsetKey.self) { offset = $0 }
        .overlay(alignment: .top) {
            Text("Offset: \(Int(offset))")
                .padding()
                .background(.ultraThinMaterial)
        }
    }
}
```

---

## alignmentGuide

Customize alignment beyond built-in options:

```swift
// Align text baselines across different-sized views
HStack(alignment: .firstTextBaseline) {
    Text("Large")
        .font(.largeTitle)
    Text("Small")
        .font(.caption)
}

// Custom alignment
extension VerticalAlignment {
    struct CustomCenter: AlignmentID {
        static func defaultValue(in d: ViewDimensions) -> CGFloat { d.height / 2 }
    }
    static let customCenter = VerticalAlignment(CustomCenter.self)
}

// Override alignment for specific views
HStack(alignment: .customCenter) {
    Rectangle()
        .frame(width: 50, height: 100)
    
    Text("Aligned to custom point")
        .alignmentGuide(.customCenter) { d in d[.top] } // Align top to custom center
}
```

---

## ViewThatFits (iOS 16+)

Picks the first view that fits in the available space:

```swift
ViewThatFits {
    // Tries each in order, uses first that fits
    HStack {
        Image(systemName: "star.fill")
        Text("Favorite this item")
    }
    // Fallback if horizontal layout doesn't fit
    VStack {
        Image(systemName: "star.fill")
        Text("Favorite")
    }
    // Minimal fallback
    Image(systemName: "star.fill")
}
```

> 💡 **Tip:** `ViewThatFits` is perfect for adaptive layouts that work across iPhone/iPad/Mac without conditional code based on size class.

---

## Interview Q&A

**Q: Why does modifier order matter in SwiftUI?**  
A: Each modifier creates a new wrapper view. The modifier only sees the view it wraps — its size, position, and hit area. So `.padding().background()` produces a background that covers the padded area (the outer wrapper sees the padded size), while `.background().padding()` shows a background only behind the content's intrinsic size.

**Q: When should you use a custom ViewModifier vs a plain view extension?**  
A: `ViewModifier` is better when you need to maintain state, use `@Environment`, or compose multiple modifiers that need to share context. View extensions are fine for simple modifier chains. `ViewModifier` also enables `modifier()` call which allows the modifier itself to be passed around as a value.

**Q: How does PreferenceKey flow data up the view hierarchy?**  
A: Child views set preferences with `.preference(key:value:)`. SwiftUI collects all preference values of the same key using the key's `reduce` function, then delivers the final value to parent views via `.onPreferenceChange`. This is the sanctioned way to communicate child measurements upward.

**Q: What's the problem with overusing GeometryReader?**  
A: `GeometryReader` expands to fill all available space, which disrupts natural SwiftUI layout. It also causes layout passes since it reads geometry after layout. Prefer `background(GeometryReader{})` to measure without affecting layout, or use PreferenceKey patterns.

---

## Quick Revision

- Modifier order = wrapping order — outer modifier sees inner result's frame
- `.padding().background()` → background covers padding area
- `.background().padding()` → background only behind content
- `ViewModifier` for reusable, stateful modifier bundles
- `GeometryReader` expands to fill available space — use carefully
- `PreferenceKey` = data flowing UP the hierarchy (child → parent)
- `alignmentGuide` for custom alignment positions
- `ViewThatFits` for adaptive layouts (iOS 16+)
