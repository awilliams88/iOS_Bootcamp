# 04 — Navigation, Presentation, Custom Transitions, and Coordinators

Navigation is not just about moving from one screen to another.
It is about ownership, information flow, stack behavior, presentation semantics, and maintainable feature coordination.
This file covers the major UIKit navigation patterns you need for interviews and real apps.

## Learning goals

By the end of this file you should be able to:

- Explain `UINavigationController` stack behavior.
- Describe push and pop operations.
- Explain `UITabBarController` and tab-based app structure.
- Compare modal presentation styles.
- Describe custom transition APIs at a practical level.
- Explain `UIViewControllerTransitioningDelegate`.
- Discuss the coordinator pattern for navigation ownership.

## Navigation mental model

UIKit navigation is a combination of containment and presentation.
A navigation controller contains a stack of controllers.
A tab bar controller contains sibling root flows.
Modal presentation creates a presenting-presented relationship.
Strong candidates understand these roles separately rather than treating “navigation” as one undifferentiated thing.

## `UINavigationController`

A navigation controller manages a stack of view controllers.
The top controller is visible.
Pushing adds a new controller to the top.
Popping removes the top and reveals the previous one.

### Why the stack matters

The stack expresses drill-down hierarchy.
It also implies back navigation behavior and ownership boundaries.
A detail screen pushed from a list usually belongs to the same flow.

### Basic push

```swift
import UIKit

final class ProductsViewController: UIViewController {
    private let coordinator: ProductsCoordinating

    init(coordinator: ProductsCoordinating) {
        self.coordinator = coordinator
        super.init(nibName: nil, bundle: nil)
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    func didSelectProduct(_ product: Product) {
        coordinator.showProductDetail(product)
    }
}

protocol ProductsCoordinating {
    func showProductDetail(_ product: Product)
}

final class ProductsCoordinator: ProductsCoordinating {
    let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func showProductDetail(_ product: Product) {
        let detailController = ProductDetailViewController(product: product)
        navigationController.pushViewController(detailController, animated: true)
    }
}
```

### Pop operations

- `popViewController(animated:)` removes the top controller.
- `popToRootViewController(animated:)` returns to the root of the stack.
- `popToViewController(_:animated:)` returns to a specific earlier controller.

## Common navigation stack interview points

- A push is typically used for hierarchical drill-down.
- The navigation controller owns the stack, not each child controller individually.
- You can customize navigation bar items from the visible controller.
- Interactive back gestures are part of the navigation controller’s behavior.

> 🎯 **Interview Answer:** “I use push when the next screen is part of the current navigation hierarchy and back navigation should return the user to the previous step naturally.”

## `UITabBarController`

A tab bar controller manages multiple sibling root flows.
Each tab typically hosts its own root controller, often a navigation controller.
This lets each tab preserve its own navigation stack independently.

### Common app structure

- Tab bar controller at the root.
- Each tab contains a navigation controller.
- Each navigation controller owns its feature flow.

```swift
import UIKit

final class AppTabBuilder {
    func build() -> UITabBarController {
        let homeNav = UINavigationController(rootViewController: HomeViewController())
        homeNav.tabBarItem = UITabBarItem(title: "Home", image: UIImage(systemName: "house"), selectedImage: nil)

        let searchNav = UINavigationController(rootViewController: SearchViewController())
        searchNav.tabBarItem = UITabBarItem(title: "Search", image: UIImage(systemName: "magnifyingglass"), selectedImage: nil)

        let settingsNav = UINavigationController(rootViewController: SettingsViewController())
        settingsNav.tabBarItem = UITabBarItem(title: "Settings", image: UIImage(systemName: "gearshape"), selectedImage: nil)

        let tabController = UITabBarController()
        tabController.viewControllers = [homeNav, searchNav, settingsNav]
        return tabController
    }
}
```

### Why this is important

It shows you understand containment hierarchy.
It also matters for restoration, deep linking, and independent flow ownership.

## Modal presentation

Not every navigation transition belongs on a push stack.
Modals represent a different semantic relationship.
They are often used for temporary flows, creation flows, authentication, confirmation, or context that should sit above the current hierarchy.

### Common modal presentation styles

- `fullScreen`
- `pageSheet`
- `formSheet`
- `custom`

### `fullScreen`

Takes over the full interface.
Historically common.
Still appropriate when the presented flow should feel separate or when the UI requires full takeover.

### `pageSheet`

Common on larger devices and modern presentations.
Feels more contextual than full screen.
Often used for detail or editing flows that should preserve awareness of the underlying screen.

### `formSheet`

Useful on iPad-like presentations where a centered form is appropriate.

### `custom`

Used when you provide custom transition and/or presentation behavior.

```swift
import UIKit

let composeController = ComposeViewController()
composeController.modalPresentationStyle = .pageSheet
present(composeController, animated: true)
```

### Push versus modal

Use push when:

- The destination is part of the same hierarchy.
- Back navigation should move up one level.
- The user is drilling into content.

Use modal when:

- The flow is temporary or self-contained.
- The user is creating or editing something in a distinct context.
- You need an overlay or independent dismissal model.

> 💡 **Tip:** In interviews, talk about user semantics, not just animations. “Push vs modal” is really a product-flow decision as much as a technical one.

## Dismissing modals and passing results back

Presented controllers typically dismiss themselves or ask a coordinator to dismiss them.
Passing information back often uses delegation, closures, a shared store, or coordinator mediation.
The right approach depends on ownership.

### A closure-based example

```swift
import UIKit

final class FilterViewController: UIViewController {
    var onApply: ((SearchFilters) -> Void)?

    private var currentFilters = SearchFilters()

    func applyTapped() {
        onApply?(currentFilters)
        dismiss(animated: true)
    }
}
```

This is fine for small local flows.
For bigger flows, a coordinator can keep navigation and result handling clearer.

## Custom transitions

Custom transitions let you control how controllers animate between states.
The main APIs to know are:

- `UIViewControllerAnimatedTransitioning`
- `UIPercentDrivenInteractiveTransition`
- `UIViewControllerTransitioningDelegate`

You do not need to memorize every method signature perfectly in an interview.
You should understand the responsibilities.

## `UIViewControllerAnimatedTransitioning`

This protocol provides the animation controller object.
It tells UIKit how long the animation lasts and performs the transition.

### High-level responsibilities

- Return transition duration.
- Perform animations inside the provided transition context.
- Signal completion.

```swift
import UIKit

final class FadeInAnimator: NSObject, UIViewControllerAnimatedTransitioning {
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        0.3
    }

    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        guard let toView = transitionContext.view(forKey: .to) else {
            transitionContext.completeTransition(false)
            return
        }

        let container = transitionContext.containerView
        toView.alpha = 0
        toView.frame = transitionContext.finalFrame(for: transitionContext.viewController(forKey: .to)!)
        container.addSubview(toView)

        UIView.animate(withDuration: transitionDuration(using: transitionContext), animations: {
            toView.alpha = 1
        }, completion: { finished in
            transitionContext.completeTransition(finished)
        })
    }
}
```

### Common mistakes

- Forgetting to add the destination view to the container.
- Using incorrect final frames.
- Forgetting to complete the transition.
- Not handling cancellation correctly in interactive flows.

## `UIPercentDrivenInteractiveTransition`

This class helps manage interactive transitions, such as swipe-to-dismiss or drag-driven presentations.
It lets user input drive transition progress.

### Why it matters

Interactive transitions feel more natural in many gesture-driven experiences.
Interviewers may ask at a high level how you would build one.

### Conceptual flow

- Start the transition.
- Update percent complete as the gesture changes.
- Finish or cancel based on gesture outcome.

## `UIViewControllerTransitioningDelegate`

The transitioning delegate provides the animation controller and optionally the interaction controller.
This is the bridge between the presented controller and your custom transition objects.

```swift
import UIKit

final class CardTransitioningDelegate: NSObject, UIViewControllerTransitioningDelegate {
    let animator = FadeInAnimator()

    func animationController(
        forPresented presented: UIViewController,
        presenting: UIViewController,
        source: UIViewController
    ) -> UIViewControllerAnimatedTransitioning? {
        animator
    }

    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        animator
    }
}
```

### When this is used

Set a controller’s `transitioningDelegate` and use `.custom` modal presentation style.
UIKit then asks your delegate for transition objects.

## Navigation architecture and the coordinator pattern

A coordinator is an object that owns navigation decisions and flow composition.
This helps keep view controllers focused on view logic rather than routing.

### Why teams use coordinators

- Reduce navigation logic inside controllers.
- Centralize deep-link handling.
- Simplify flow orchestration.
- Improve testability of navigation decisions.
- Make feature composition clearer.

### A simple coordinator protocol

```swift
import UIKit

protocol Coordinator: AnyObject {
    func start()
}

final class CheckoutCoordinator: Coordinator {
    private let navigationController: UINavigationController
    private let cart: Cart

    init(navigationController: UINavigationController, cart: Cart) {
        self.navigationController = navigationController
        self.cart = cart
    }

    func start() {
        let summary = CheckoutSummaryViewController(cart: cart)
        summary.onContinue = { [weak self] in
            self?.showPayment()
        }
        navigationController.pushViewController(summary, animated: true)
    }

    private func showPayment() {
        let payment = PaymentViewController(cart: cart)
        payment.onFinished = { [weak self] result in
            self?.handlePayment(result)
        }
        navigationController.pushViewController(payment, animated: true)
    }

    private func handlePayment(_ result: PaymentResult) {
        let confirmation = ConfirmationViewController(result: result)
        navigationController.pushViewController(confirmation, animated: true)
    }
}
```

### Coordinator tradeoffs

Benefits:

- Clear navigation ownership.
- Better flow reuse.
- Better separation of UI rendering from routing.

Costs:

- More types and boilerplate.
- Potential retain-cycle or lifecycle complexity if poorly designed.
- Can become over-engineered if used blindly for tiny apps.

> ⚠️ **Pitfall:** Coordinators help when they simplify ownership. They hurt when they become another layer of indirection without clear benefits.

## Common navigation interview scenarios

### “Should this be push or modal?”

A strong answer starts with user flow semantics.
If the user is drilling deeper into current content, push is natural.
If the user is entering a temporary or separate flow, modal is often better.

### “Who should own navigation?”

A strong answer says controllers can initiate user intent, but navigation ownership is often cleaner in a coordinator or routing layer for larger apps.

### “How would you deep-link into a tab-based app?”

A good answer often includes:

- Select the correct tab.
- Ensure the correct navigation controller is active.
- Rebuild or adjust the stack as needed.
- Push or present the destination.
- Avoid duplicated screens if the flow is already partially active.

## Memory and navigation

Navigation decisions affect object lifetime.
Pushed controllers stay on the stack until popped.
Presented controllers stay retained by presentation relationships until dismissed.
Coordinators may need explicit child coordinator cleanup when flows end.

### Common retain-cycle risk

- Controller strongly retains coordinator.
- Coordinator strongly retains controller.
- Closures capture `self` strongly across the flow.

These are very real production issues.
Mentioning them is a strong senior signal.

## State restoration and navigation stacks

Even if not deeply probed, it is useful to understand that navigation stacks represent app state.
Tab selection, stack composition, and presented flows all matter if you restore state or handle deep links.
This is another reason centralized navigation ownership can help.

## Senior-level discussion

Navigation is often where architecture quality becomes visible.
Small apps can survive with controllers pushing controllers directly.
Larger apps often benefit from more explicit ownership.
The right choice depends on scale, not ideology.

Senior engineers usually discuss navigation in terms of:

- Flow ownership.
- User semantics.
- State restoration or deep linking.
- Testability.
- Coupling between features.

A strong senior answer might sound like this:

- I use navigation controllers for hierarchical drill-down.
- I use modals for temporary or separate flows.
- I embed navigation controllers in tab roots for independent stacks.
- I move complex routing into coordinators when the app’s flow graph becomes nontrivial.
- I treat custom transitions as a UI capability that should remain isolated from business logic.

## Interview Q&A

### 1. What does `UINavigationController` do?

It manages a stack of view controllers and provides push/pop navigation behavior for hierarchical flows.

### 2. When would you push a view controller?

When the next screen is part of the current hierarchical flow and back navigation should return to the previous step naturally.

### 3. When would you present modally instead of push?

When the new screen is a temporary, separate, or overlay-style flow that should not simply become the next level of the current stack.

### 4. What does `UITabBarController` do?

It manages multiple sibling root flows, often each with its own navigation controller and preserved stack.

### 5. Why do many apps embed navigation controllers inside tabs?

So each tab can own its own hierarchical stack independently while the tab bar manages top-level switching.

### 6. What are common modal presentation styles?

`fullScreen`, `pageSheet`, `formSheet`, and `custom` are common ones to know.

### 7. What is `UIViewControllerAnimatedTransitioning` for?

It provides the animation logic for a custom transition, including duration and the actual animations performed using the transition context.

### 8. What is `UIPercentDrivenInteractiveTransition` for?

It helps drive a transition interactively based on gesture progress, such as a swipe-to-dismiss flow.

### 9. What does `UIViewControllerTransitioningDelegate` do?

It supplies the animator and optional interaction controller objects used for custom presentations and dismissals.

### 10. What is the coordinator pattern?

It is an architectural pattern where a separate object owns navigation and flow composition so view controllers stay more focused on view logic.

### 11. Why do teams use coordinators?

To reduce controller coupling, centralize routing, support deep links more cleanly, and make complex flows easier to reason about.

### 12. What is a downside of coordinators?

They can add boilerplate and indirection, especially if applied too aggressively in a small app.

### 13. What is a strong answer to push vs modal?

Base the answer on user semantics and flow ownership, not just animation style.
Push is for hierarchical drill-down; modal is for separate or temporary flows.

### 14. What is the senior-level summary of navigation?

Use containment and presentation deliberately, keep flow ownership clear, and choose push, tab, modal, coordinator, and custom transitions based on user semantics and app scale.
