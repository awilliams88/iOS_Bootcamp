# 05 — Responder Chain, Gestures, and Animations

UIKit event handling and animation are foundational interview topics because they connect directly to how screens feel and behave.
This file covers responder chain mechanics, hit testing, first responder behavior, gesture recognizers, and the major UIKit/Core Animation APIs used to animate interfaces.

## Learning goals

By the end of this file you should be able to:

- Explain the responder chain.
- Describe hit testing with `hitTest(_:with:)` and `point(inside:with:)`.
- Explain first responder behavior.
- Compare common gesture recognizers.
- Use gesture delegate methods thoughtfully.
- Compare `UIView.animate`, `UIViewPropertyAnimator`, and Core Animation APIs.
- Explain layer-backed animation concepts.

## Responder chain mental model

The responder chain is the ordered path UIKit uses to route events and actions.
Views are responders.
View controllers can participate.
Windows and application-level objects are in the chain too.
If one responder does not handle an event or action, UIKit can pass it along.

### Why it matters

The responder chain explains:

- How touches travel through the view tree.
- How action messages can bubble upward.
- Why keyboard and text input behavior works through first responder status.
- Why some custom interaction bugs happen in unexpected places.

### Practical chain intuition

A touch is first localized to the relevant window and view hierarchy.
Then the system determines which view should receive it.
Actions like `sendAction` can move up through target and responder relationships.
Keyboard input goes to the current first responder.

> 🎯 **Interview Answer:** “The responder chain is UIKit’s routing path for events and actions across views, controllers, windows, and application objects. It lets the framework find the right receiver without every object knowing every other object directly.”

## Hit testing

Hit testing determines which view should receive a touch.
UIKit starts from the window and traverses the hierarchy.
A view is a candidate only if it is interactive, visible enough, and contains the touch point.

## `point(inside:with:)`

This method answers whether the touch point lies within the view’s interactive bounds.
It is a geometric gate.
You can override it to enlarge or shrink the interactive area.

```swift
import UIKit

final class LargeTapButton: UIButton {
    override func point(inside point: CGPoint, with event: UIEvent?) -> Bool {
        let expandedBounds = bounds.insetBy(dx: -10, dy: -10)
        return expandedBounds.contains(point)
    }
}
```

This is useful when the visible control is small but you want a more forgiving tap target.

## `hitTest(_:with:)`

This method returns the specific view that should receive the touch.
It considers subviews recursively.
You can override it for custom routing behavior, but do so carefully.

```swift
import UIKit

final class PassthroughContainerView: UIView {
    override func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView? {
        let hitView = super.hitTest(point, with: event)
        return hitView === self ? nil : hitView
    }
}
```

This pattern allows touches to pass through the container itself while still allowing subviews to receive touches.
It is common in overlay UIs.

### Hit-testing rules to remember

- Hidden views do not receive touches.
- Views with `alpha` below the interactive threshold do not receive touches.
- Views with `isUserInteractionEnabled == false` do not receive touches.
- Subviews are considered from frontmost to backmost.

> ⚠️ **Pitfall:** Overriding `hitTest` or `point(inside:)` can easily create hard-to-debug touch routing bugs. Only do it when gesture recognizers or simpler hierarchy changes are insufficient.

## First responder

The first responder is the object currently best positioned to receive certain events, especially keyboard and editing input.
Text fields and text views often become first responder when editing begins.

### Why first responder matters

- Keyboard input goes there.
- Editing commands route there.
- Menus and some action handling depend on it.
- Focus-like behavior in UIKit often starts with it.

### Common methods

- `becomeFirstResponder()`
- `resignFirstResponder()`
- `canBecomeFirstResponder`

```swift
import UIKit

final class LoginViewController: UIViewController {
    private let emailField = UITextField()

    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        emailField.becomeFirstResponder()
    }
}
```

This is a small but practical example of lifecycle tied to responder behavior.

## Gesture recognizers

Gesture recognizers interpret touch sequences into higher-level intents.
Instead of manually processing raw touches for many common interactions, you attach recognizers and respond to state changes.

### Common recognizers

- `UITapGestureRecognizer`
- `UILongPressGestureRecognizer`
- `UIPanGestureRecognizer`
- `UIPinchGestureRecognizer`
- `UIRotationGestureRecognizer`
- `UIScreenEdgePanGestureRecognizer`
- `UISwipeGestureRecognizer`

## `UITapGestureRecognizer`

Good for simple taps.
Often used for dismissing overlays, selecting large custom regions, or handling taps outside standard controls.

```swift
import UIKit

final class CardView: UIView {
    var onTap: (() -> Void)?

    override init(frame: CGRect) {
        super.init(frame: frame)
        let recognizer = UITapGestureRecognizer(target: self, action: #selector(handleTap))
        addGestureRecognizer(recognizer)
    }

    @objc private func handleTap() {
        onTap?()
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

## `UIPanGestureRecognizer`

Used for drag interactions, drawers, swipe-to-dismiss patterns, and many interactive transitions.
The recognizer provides translation and velocity.

```swift
import UIKit

final class DraggablePanelView: UIView {
    private lazy var panRecognizer = UIPanGestureRecognizer(target: self, action: #selector(handlePan))

    override init(frame: CGRect) {
        super.init(frame: frame)
        addGestureRecognizer(panRecognizer)
    }

    @objc private func handlePan(_ recognizer: UIPanGestureRecognizer) {
        let translation = recognizer.translation(in: superview)
        switch recognizer.state {
        case .began, .changed:
            transform = CGAffineTransform(translationX: 0, y: max(0, translation.y))
        case .ended, .cancelled:
            UIView.animate(withDuration: 0.25) {
                self.transform = .identity
            }
        default:
            break
        }
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

## Gesture delegate

`UIGestureRecognizerDelegate` lets you control recognizer behavior.
This is important when gestures interact with each other or with scrolling.

### Common delegate decisions

- Whether a recognizer should begin.
- Whether two recognizers can recognize simultaneously.
- Whether one recognizer should require another to fail.
- Whether touches should be received at all.

### Simultaneous recognition example

```swift
import UIKit

final class ZoomingImageViewController: UIViewController, UIGestureRecognizerDelegate {
    private let imageView = UIImageView()

    override func viewDidLoad() {
        super.viewDidLoad()

        let pinch = UIPinchGestureRecognizer(target: self, action: #selector(handlePinch))
        let rotation = UIRotationGestureRecognizer(target: self, action: #selector(handleRotation))
        pinch.delegate = self
        rotation.delegate = self

        imageView.isUserInteractionEnabled = true
        imageView.addGestureRecognizer(pinch)
        imageView.addGestureRecognizer(rotation)
    }

    func gestureRecognizer(
        _ gestureRecognizer: UIGestureRecognizer,
        shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer
    ) -> Bool {
        true
    }

    @objc private func handlePinch(_ recognizer: UIPinchGestureRecognizer) {}
    @objc private func handleRotation(_ recognizer: UIRotationGestureRecognizer) {}
}
```

### Gesture conflict example

A horizontal carousel inside a vertical scroll view is a classic problem.
The right answer often involves delegate logic about gesture direction and simultaneous recognition policy.
Being able to explain that is a good senior signal.

> ⚠️ **Pitfall:** Gesture conflicts are often ownership problems, not just API problems. If you cannot explain which recognizer should win and why, the UI will feel inconsistent even if every recognizer compiles.

## `UIView.animate`

This is the classic high-level UIKit animation API.
It is great for common view property animations.

### Good for

- Alpha changes.
- Frame or constraint-driven layout changes.
- Transform changes.
- Simple springy motion with convenience overloads.

```swift
import UIKit

UIView.animate(withDuration: 0.25) {
    self.bannerView.alpha = 1
    self.bannerView.transform = .identity
}
```

### What it animates well

It animates view-backed animatable properties by coordinating with the underlying layer system.
For many UI transitions, it is all you need.

## Spring animations

UIKit provides spring-based animation convenience methods that simulate more natural motion.
These are useful for cards, modals, playful controls, and interactions that benefit from softness or bounce.

```swift
import UIKit

UIView.animate(
    withDuration: 0.6,
    delay: 0,
    usingSpringWithDamping: 0.78,
    initialSpringVelocity: 0.6,
    options: [.curveEaseInOut]
) {
    self.cardView.transform = .identity
}
```

### Senior note

Use spring parameters intentionally.
Do not copy random values from old snippets without understanding the feel you want.

## `UIViewPropertyAnimator`

`UIViewPropertyAnimator` gives you more control than the classic block API.
It supports interactive control, pausing, scrubbing, reversing, and more explicit state management.

### When to use it

- Gesture-driven animations.
- Interruptible transitions.
- Animations that need pause/resume or manual progress updates.
- More advanced coordination than simple fire-and-forget view animations.

```swift
import UIKit

final class ExpandableCardController: UIViewController {
    private let cardView = UIView()
    private lazy var animator = UIViewPropertyAnimator(duration: 0.4, dampingRatio: 0.85) {
        self.cardView.transform = CGAffineTransform(scaleX: 1.0, y: 1.0)
        self.cardView.alpha = 1.0
    }

    func presentCard() {
        cardView.transform = CGAffineTransform(scaleX: 0.92, y: 0.92)
        cardView.alpha = 0
        animator.startAnimation()
    }
}
```

### Why interviewers like this API

It reveals whether you understand animation as state over time rather than just “run this block.”

## Core Animation

UIKit animations sit on top of Core Animation in many cases.
Core Animation works more directly with layers.
It is powerful and important for lower-level or more specialized animations.

### Why layers matter

Every view has a backing `CALayer`.
Layer properties control many visual aspects such as corner radius, shadow, opacity, and transform.
Some animations are easier or more efficient at the layer level.

## `CABasicAnimation`

This animates one property from one value to another.
It is simple and useful for straightforward layer animations.

```swift
import QuartzCore
import UIKit

let animation = CABasicAnimation(keyPath: "opacity")
animation.fromValue = 0
animation.toValue = 1
animation.duration = 0.3
layer.add(animation, forKey: "fade")
```

### Important caveat

Core Animation often animates the presentation layer.
If you want the final model state to persist, you should also set the layer’s actual property.

## `CAKeyframeAnimation`

This animates through multiple values or along a path.
It is useful for shake effects, orbiting motion, or custom timing shapes.

```swift
import QuartzCore

let shake = CAKeyframeAnimation(keyPath: "transform.translation.x")
shake.values = [-8, 8, -6, 6, -3, 3, 0]
shake.duration = 0.4
layer.add(shake, forKey: "shake")
```

## `CAAnimation` versus UIView animations

A good interview comparison:

- `UIView.animate` is high-level and great for common view property changes.
- `UIViewPropertyAnimator` adds interactive and stateful control.
- Core Animation is lower-level and stronger for advanced layer animation scenarios.

## Layer properties worth knowing

Common animatable layer properties include:

- `opacity`
- `transform`
- `position`
- `bounds`
- `cornerRadius`
- `backgroundColor`
- `shadowOpacity`
- `shadowRadius`

Knowing these helps you discuss performance and animation choices.

## Model layer versus presentation layer

This is a classic senior interview topic.

### Model layer

Represents the target state of the layer in your app’s data model.
This is the state you read and set directly.

### Presentation layer

Represents the in-flight animated state currently being shown on screen.
This is useful when you need the current animated position during an active animation.

### Why it matters

If you add a Core Animation without updating the model layer, the object may visually animate and then snap back because the real target state never changed.

> ⚠️ **Pitfall:** Developers sometimes think Core Animation “changed the layer permanently” when they only animated the presentation. Always reason about final model state explicitly.

## Performance notes for animation

- Animate cheap properties when possible, especially transforms and opacity.
- Be careful with shadows, masks, and offscreen rendering on large moving views.
- Avoid forcing repeated layout inside high-frequency animation loops when not necessary.
- Measure if a complex animation stutters.
- Keep gesture-driven animations responsive and interruptible.

## Gesture + animation example

```swift
import UIKit

final class BottomSheetController: UIViewController {
    private let sheetView = UIView()
    private var animator: UIViewPropertyAnimator?

    override func viewDidLoad() {
        super.viewDidLoad()

        let pan = UIPanGestureRecognizer(target: self, action: #selector(handlePan))
        sheetView.addGestureRecognizer(pan)
    }

    @objc private func handlePan(_ recognizer: UIPanGestureRecognizer) {
        let translation = recognizer.translation(in: view).y
        let progress = min(max(translation / 300, 0), 1)

        switch recognizer.state {
        case .began:
            animator = UIViewPropertyAnimator(duration: 0.35, curve: .easeOut) {
                self.sheetView.transform = CGAffineTransform(translationX: 0, y: 300)
            }
            animator?.pauseAnimation()
        case .changed:
            animator?.fractionComplete = progress
        case .ended, .cancelled:
            if progress > 0.4 {
                animator?.continueAnimation(withTimingParameters: nil, durationFactor: 1)
            } else {
                animator?.isReversed = true
                animator?.continueAnimation(withTimingParameters: nil, durationFactor: 1)
            }
        default:
            break
        }
    }
}
```

This example combines gesture progress with a property animator.
That is a modern UIKit pattern worth understanding.

## Common interview bugs and reasoning

### “My tap isn’t being recognized.”

Check:

- Is the view interactive?
- Is another view covering it?
- Is hit testing being overridden?
- Is a gesture recognizer cancelling or intercepting touches?
- Is the view’s frame actually where you think it is?

### “My scroll view and custom pan are fighting.”

Check gesture delegate policies.
Ask which recognizer should win and under what direction or threshold.

### “My layer animation snaps back.”

You probably animated the presentation layer but did not update the model layer’s final property value.

### “My animation feels janky.”

Check for layout thrash, main-thread work during animation, expensive visual effects, or too many simultaneous updates.

## Senior-level discussion

Senior UIKit engineers understand that input handling and animation are both state machines.
Touches become gestures.
Gestures move through recognizer states.
Animations move views and layers through visual states.
The best solutions make those state transitions explicit.

A few useful heuristics:

- Use standard controls before custom gesture handling when possible.
- Use gesture recognizers for high-level interactions instead of manual touch tracking unless custom behavior truly requires it.
- Use `UIView.animate` for common one-shot UI changes.
- Use `UIViewPropertyAnimator` for interactive or interruptible motion.
- Use Core Animation when layer-level or advanced timing/path control is the better fit.
- Keep an eye on hit testing, not just visible layout, when debugging touch issues.

## Interview Q&A

### 1. What is the responder chain?

It is UIKit’s routing path for events and actions through responders such as views, view controllers, windows, and application objects.

### 2. What does `point(inside:with:)` do?

It determines whether a touch point is within the view’s interactive area.
It is often overridden to adjust tappable bounds.

### 3. What does `hitTest(_:with:)` do?

It returns the specific view that should receive the touch by evaluating the hierarchy and subviews.

### 4. What is the first responder?

It is the object currently designated to receive certain events, especially keyboard and editing input.

### 5. Why use gesture recognizers instead of raw touch handling?

They provide higher-level interpretation of touch sequences, simpler APIs for common gestures, and better integration with UIKit’s gesture system.

### 6. What is `UIGestureRecognizerDelegate` for?

It lets you control recognizer behavior, including whether gestures begin, recognize simultaneously, or require other recognizers to fail.

### 7. When would you use `UIView.animate`?

For common high-level view property animations such as alpha, transforms, and layout-driven changes.

### 8. When would you use `UIViewPropertyAnimator`?

For interactive, interruptible, or stateful animations where you need more control than the classic block API provides.

### 9. What is `CABasicAnimation`?

A Core Animation class that animates a single layer property from one value to another over time.

### 10. What is `CAKeyframeAnimation`?

A Core Animation class that animates through multiple values or along a path.
It is useful for more complex motion.

### 11. What is the difference between model layer and presentation layer?

The model layer is the actual target state you set.
The presentation layer is the in-flight animated visual state currently shown on screen.

### 12. Why might an animation snap back at the end?

Because the final model state was not updated, so only the temporary presentation changed.

### 13. How do you debug a touch not reaching a view?

Check view visibility, interaction enabled state, frames, overlapping views, hit-testing overrides, and gesture recognizer interactions.

### 14. What is the senior-level summary of UIKit interaction and animation?

Understand event routing, keep gesture ownership deliberate, choose the right animation abstraction for the interaction, and always reason about both visible behavior and underlying state.
