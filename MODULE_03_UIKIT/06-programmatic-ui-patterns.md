# 06 — Programmatic UI Patterns, XIBs, Storyboards, and Reusable View Code

Many iOS teams still use Interface Builder somewhere in their app.
Many others prefer programmatic UI for most new screens.
Senior candidates should be able to discuss the tradeoffs honestly and demonstrate clean code-based UI patterns.
This file covers those tradeoffs and practical programmatic UIKit organization.

## Learning goals

By the end of this file you should be able to:

- Compare XIBs, storyboards, and programmatic UI.
- Explain when to override `loadView()`.
- Use `UINib` appropriately.
- Create constraints programmatically.
- Organize view code with clean setup patterns.
- Build reusable view components.
- Explain why many teams avoid storyboards at scale.

## Three major UIKit construction styles

UIKit historically supports several ways to build interfaces.
Each has valid use cases.
The senior answer is a tradeoff answer, not a purity argument.

### Storyboards

Storyboards represent multiple scenes and transitions visually.
They can speed up simple app construction and be approachable for teams that like visual assembly.

### XIBs

XIBs are nib files that usually represent a single view or component.
They are more modular than storyboards and can be useful for reusable cells or view fragments.

### Programmatic UI

Programmatic UI creates and configures views directly in code.
It is popular on larger teams because code review, reuse, and merge workflows tend to be clearer.

> 🎯 **Interview Answer:** “I choose between storyboard, XIB, and programmatic UI based on team scale, complexity, and reuse needs. For large feature teams, programmatic UI often wins because ownership and diffs are clearer, but XIBs can still be a good fit for isolated reusable components.”

## Storyboards — strengths and weaknesses

### Strengths

- Fast for simple prototypes.
- Easy to visualize scene relationships.
- Good for onboarding developers who like visual tools.
- Can be convenient for straightforward segue-based apps.

### Weaknesses

- Merge conflicts become painful on larger teams.
- Navigation wiring can become implicit and harder to reason about.
- Large storyboard files become fragile.
- Runtime crashes from identifier or outlet mismatches can still happen.
- Reusability across features can be weaker than code-based composition.

### When storyboards are still reasonable

- Small apps.
- Small teams.
- Simple flows.
- Internal tools where long-term scale pressure is low.

## XIBs — strengths and weaknesses

### Strengths

- More modular than storyboards.
- Nice for reusable cells or custom components.
- Lets designers or developers preview parts visually.
- Can strike a middle ground between visual editing and modularity.

### Weaknesses

- Still involves Interface Builder tooling and outlet wiring.
- Not as transparent in code review as fully programmatic UI.
- Can still produce merge pain, though often less than large storyboards.

### When XIBs are a good fit

- Reusable table or collection cells.
- Standalone component views.
- Teams that want visual editing for isolated pieces rather than whole app flows.

## Programmatic UI — strengths and weaknesses

### Strengths

- Full control in code.
- Better merge behavior in many teams.
- Easier to review diffs in pull requests.
- Easier to create reusable patterns and abstractions.
- Navigation and hierarchy stay explicit.
- Often easier to support large-scale modularization.

### Weaknesses

- More verbose initially.
- Requires stronger discipline in code organization.
- Harder for some designers or less code-oriented contributors to tweak visually.
- Without conventions, code can become messy.

### When programmatic UI is a strong default

- Medium to large teams.
- Modular feature architecture.
- Complex dynamic screens.
- Apps with many reusable components.
- Teams that value explicit ownership and code-review clarity.

## Overriding `loadView()`

When building a controller programmatically, overriding `loadView()` is often the cleanest way to assign a custom root view.
This keeps the root view strongly typed and separates view construction from controller lifecycle.

```swift
import UIKit

final class SettingsView: UIView {
    let titleLabel = UILabel()
    let stackView = UIStackView()

    override init(frame: CGRect) {
        super.init(frame: frame)
        backgroundColor = .systemBackground

        titleLabel.text = "Settings"
        titleLabel.font = .preferredFont(forTextStyle: .largeTitle)

        stackView.axis = .vertical
        stackView.spacing = 16

        let root = UIStackView(arrangedSubviews: [titleLabel, stackView])
        root.axis = .vertical
        root.spacing = 24
        root.translatesAutoresizingMaskIntoConstraints = false

        addSubview(root)

        NSLayoutConstraint.activate([
            root.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 24),
            root.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 20),
            root.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -20)
        ])
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

final class SettingsViewController: UIViewController {
    private let rootView = SettingsView()

    override func loadView() {
        view = rootView
    }
}
```

### Why this pattern is clean

- The controller owns one root view.
- The view encapsulates subview structure.
- The controller interacts with the typed root view.
- Setup code is localized.

## `UINib`

`UINib` is useful when you want to load nib-based views or cells programmatically.
It is common in codebases that use nib-backed reusable components.

### Example — registering a nib-backed cell

```swift
import UIKit

final class CatalogViewController: UIViewController {
    private let tableView = UITableView(frame: .zero, style: .plain)

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.register(UINib(nibName: "CatalogCell", bundle: nil), forCellReuseIdentifier: CatalogCell.reuseIdentifier)
    }
}
```

### When nib-backed components still work well

- Reusable cells designed visually.
- Isolated component teams comfortable with nibs.
- Mature codebases with stable nib infrastructure.

## Creating constraints programmatically

Programmatic constraints are explicit and reviewable.
The main pattern is:

1. Create views.
2. Add them to the hierarchy.
3. Set `translatesAutoresizingMaskIntoConstraints = false`.
4. Activate constraints.

```swift
import UIKit

final class LoginFormView: UIView {
    let emailField = UITextField()
    let passwordField = UITextField()
    let submitButton = UIButton(type: .system)

    override init(frame: CGRect) {
        super.init(frame: frame)

        backgroundColor = .systemBackground
        emailField.borderStyle = .roundedRect
        passwordField.borderStyle = .roundedRect
        submitButton.setTitle("Sign In", for: .normal)

        let stack = UIStackView(arrangedSubviews: [emailField, passwordField, submitButton])
        stack.axis = .vertical
        stack.spacing = 12
        stack.translatesAutoresizingMaskIntoConstraints = false

        addSubview(stack)

        NSLayoutConstraint.activate([
            stack.centerYAnchor.constraint(equalTo: centerYAnchor),
            stack.leadingAnchor.constraint(equalTo: safeAreaLayoutGuide.leadingAnchor, constant: 20),
            stack.trailingAnchor.constraint(equalTo: safeAreaLayoutGuide.trailingAnchor, constant: -20)
        ])
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

This style is simple and reliable.
It also scales well when paired with consistent naming and organization.

## Common organization patterns

Programmatic UI can become messy if every controller mixes lifecycle, event handling, and view construction in one block.
Many teams standardize setup methods.

### Example pattern

- `loadUI()`
- `setupHierarchy()`
- `setupConstraints()`
- `setupAppearance()`
- `bindViewModel()`
- `setupActions()`

```swift
import UIKit

final class ProfileEditorViewController: UIViewController {
    private let rootView = ProfileEditorView()

    override func loadView() {
        view = rootView
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        loadUI()
        bindViewModel()
    }

    private func loadUI() {
        setupAppearance()
        setupActions()
    }

    private func setupAppearance() {
        title = "Edit Profile"
    }

    private func setupActions() {
        rootView.saveButton.addTarget(self, action: #selector(saveTapped), for: .touchUpInside)
    }

    private func bindViewModel() {
        // Bind outputs to UI.
    }

    @objc private func saveTapped() {
        // Forward intent.
    }
}
```

### Why these patterns help

- They reduce giant lifecycle methods.
- They make intent obvious.
- They standardize code review expectations.
- They help new team members navigate screen code quickly.

## Reusable view components

Programmatic UI shines when building reusable components.
Instead of copying layout logic across screens, you create focused views with clear configuration APIs.

### Example — reusable status banner

```swift
import UIKit

final class StatusBannerView: UIView {
    enum Style {
        case success
        case warning
        case error
    }

    private let iconView = UIImageView()
    private let messageLabel = UILabel()

    override init(frame: CGRect) {
        super.init(frame: frame)
        layer.cornerRadius = 12

        let stack = UIStackView(arrangedSubviews: [iconView, messageLabel])
        stack.axis = .horizontal
        stack.spacing = 8
        stack.alignment = .center
        stack.translatesAutoresizingMaskIntoConstraints = false

        addSubview(stack)

        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: topAnchor, constant: 12),
            stack.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 12),
            stack.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -12),
            stack.bottomAnchor.constraint(equalTo: bottomAnchor, constant: -12)
        ])
    }

    func configure(message: String, style: Style) {
        messageLabel.text = message

        switch style {
        case .success:
            backgroundColor = .systemGreen.withAlphaComponent(0.15)
            iconView.image = UIImage(systemName: "checkmark.circle.fill")
        case .warning:
            backgroundColor = .systemYellow.withAlphaComponent(0.15)
            iconView.image = UIImage(systemName: "exclamationmark.triangle.fill")
        case .error:
            backgroundColor = .systemRed.withAlphaComponent(0.15)
            iconView.image = UIImage(systemName: "xmark.octagon.fill")
        }
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

### Good reusable component traits

- One clear responsibility.
- Small public configuration surface.
- Internal layout hidden from the caller.
- Works in multiple screens.
- Easy to preview or test in isolation.

## View code organization in custom views

Many teams use a pattern inside custom views such as:

- `setupHierarchy()`
- `setupConstraints()`
- `setupStyle()`

```swift
import UIKit

final class EmptyStateView: UIView {
    private let imageView = UIImageView()
    private let titleLabel = UILabel()
    private let subtitleLabel = UILabel()
    private let stackView = UIStackView()

    override init(frame: CGRect) {
        super.init(frame: frame)
        setupHierarchy()
        setupConstraints()
        setupStyle()
    }

    private func setupHierarchy() {
        addSubview(stackView)
        stackView.addArrangedSubview(imageView)
        stackView.addArrangedSubview(titleLabel)
        stackView.addArrangedSubview(subtitleLabel)
    }

    private func setupConstraints() {
        stackView.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            stackView.centerYAnchor.constraint(equalTo: centerYAnchor),
            stackView.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 24),
            stackView.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -24)
        ])
    }

    private func setupStyle() {
        stackView.axis = .vertical
        stackView.alignment = .center
        stackView.spacing = 12
        titleLabel.font = .preferredFont(forTextStyle: .title3)
        subtitleLabel.font = .preferredFont(forTextStyle: .body)
        subtitleLabel.textColor = .secondaryLabel
        subtitleLabel.numberOfLines = 0
        subtitleLabel.textAlignment = .center
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

This approach keeps the initializer short and the structure readable.

## Dependency injection and programmatic screens

Programmatic UI pairs well with dependency injection.
Controllers can receive services or view models through `init`, while views stay focused on presentation.
This makes the dependency graph explicit.
That is another reason many teams prefer code-based screen construction.

## Testing implications

Programmatic UI can simplify certain tests because hierarchy and dependencies are visible in code.
You can instantiate views directly in unit tests, inspect configuration, and avoid some Interface Builder wiring assumptions.
That does not mean storyboards or nibs are untestable.
It means code-based construction often feels more direct.

## Why many teams avoid storyboards

This is a frequent senior interview question.
A strong answer includes more than “merge conflicts.”

### Reasons teams avoid storyboards at scale

- Merge conflicts are harder to resolve.
- Large storyboard files obscure ownership.
- Navigation wiring becomes implicit.
- Refactoring across modules is harder.
- Code review is less precise.
- Reusable component composition is often clearer in code.
- Feature teams prefer each screen to live near its implementation.

### A balanced answer

Do not say storyboards are “bad.”
Say they are often less scalable for larger teams and more complex products.
That sounds measured and credible.

> 💡 **Tip:** In interviews, replace “we never use storyboards because they’re terrible” with “many teams move away from them as app scale and team size increase because code ownership, modularity, and merge behavior become more important.”

## When XIBs are still a great compromise

Many teams that avoid storyboards still use XIBs for isolated reusable components.
That can be a pragmatic middle ground.
You get some visual design convenience without centralizing whole navigation flows in Interface Builder.

## Senior discussion — view/controller separation in programmatic UI

Programmatic UI can still be poorly architected.
Putting every subview, constraint, style, action, and business rule directly in the controller is not “better” just because it is code.
A strong programmatic UIKit architecture usually separates:

- Reusable views.
- Controller lifecycle coordination.
- View models or presenters.
- Navigation ownership.
- Styling/theme decisions.

That separation scales better than either a giant storyboard or a giant controller.

## Example — screen built from reusable parts

```swift
import UIKit

final class AccountSummaryView: UIView {
    private let nameLabel = UILabel()
    private let emailLabel = UILabel()
    private let bannerView = StatusBannerView()

    override init(frame: CGRect) {
        super.init(frame: frame)

        let stack = UIStackView(arrangedSubviews: [bannerView, nameLabel, emailLabel])
        stack.axis = .vertical
        stack.spacing = 12
        stack.translatesAutoresizingMaskIntoConstraints = false

        addSubview(stack)

        NSLayoutConstraint.activate([
            stack.topAnchor.constraint(equalTo: safeAreaLayoutGuide.topAnchor, constant: 24),
            stack.leadingAnchor.constraint(equalTo: leadingAnchor, constant: 20),
            stack.trailingAnchor.constraint(equalTo: trailingAnchor, constant: -20)
        ])
    }

    func configure(with viewState: AccountSummaryViewState) {
        bannerView.configure(message: viewState.statusMessage, style: viewState.statusStyle)
        nameLabel.text = viewState.name
        emailLabel.text = viewState.email
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

@MainActor
final class AccountSummaryViewController: UIViewController {
    private let rootView = AccountSummaryView()
    private let viewModel: AccountSummaryViewModel

    init(viewModel: AccountSummaryViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }

    override func loadView() {
        view = rootView
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Account"
        rootView.configure(with: viewModel.viewState)
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

This pattern keeps the view focused on layout and rendering while the controller coordinates lifecycle.
That is generally the kind of code style larger teams appreciate.

## Common pitfalls

- Massive controllers doing all view construction and logic.
- Copy-pasted constraint code across many screens.
- Reusable components with unclear public APIs.
- Storyboards used for flows that changed faster than the tooling could support cleanly.
- Over-abstracting tiny views into unnecessary frameworks.
- Forgetting accessibility and Dynamic Type because code is more manual.

> ⚠️ **Pitfall:** Programmatic UI is not automatically maintainable. It becomes maintainable when naming, organization, reuse boundaries, and ownership are deliberate.

## Senior-level discussion

The strongest programmatic UI teams tend to have consistent conventions.
That is the real differentiator.
Not whether the code is generated from a DSL, written with raw Auto Layout, or built using stack views.
Consistency is what makes screens easy to review, debug, and extend.

A good senior answer often includes:

- Why a team chose code-based UI.
- What conventions keep it readable.
- How reusable components are organized.
- How root views, controllers, and navigation responsibilities are separated.
- When a nib-backed component is still a valid choice.

## Interview Q&A

### 1. What are the tradeoffs between storyboards, XIBs, and programmatic UI?

Storyboards are visual and convenient for simple flows but can scale poorly for large teams.
XIBs are more modular and useful for reusable components.
Programmatic UI is more explicit and reviewable but requires disciplined organization.

### 2. When should you override `loadView()`?

When building a screen programmatically and you want the controller to provide a custom root view instead of relying on nib or storyboard loading.

### 3. What is `UINib` used for?

It loads nib-based views or reusable components programmatically, such as nib-backed table or collection view cells.

### 4. Why do many teams prefer programmatic UI?

Because ownership is clearer, diffs are easier to review, merge conflicts are often better, and reusable patterns can be expressed directly in code.

### 5. Why do many teams avoid storyboards?

As apps and teams grow, storyboards can become harder to merge, harder to modularize, and less explicit in navigation and screen ownership.

### 6. When are XIBs still a good choice?

For isolated reusable components or cells where visual editing is helpful but full storyboard-based flow management is unnecessary.

### 7. What is a good way to organize programmatic view code?

Use clear setup methods such as hierarchy, constraints, style, bindings, and actions so lifecycle methods stay readable.

### 8. What makes a reusable view component good?

Single responsibility, a small configuration API, internal layout encapsulation, and ease of reuse across screens.

### 9. Is programmatic UI always better?

No.
It often scales better for larger apps and teams, but the right choice depends on product complexity, team workflow, and existing codebase constraints.

### 10. What is the senior-level summary of programmatic UIKit?

Choose the construction style that fits the team, keep ownership explicit, organize view code consistently, and build reusable components that make screens simpler rather than more abstract for their own sake.
