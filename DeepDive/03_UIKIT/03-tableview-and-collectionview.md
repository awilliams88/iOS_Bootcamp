# 03 — Table Views, Collection Views, Reuse, and Modern Data Sources

List rendering is one of the most practical UIKit interview topics.
It touches architecture, layout, performance, data flow, and user experience.
Senior candidates should be able to explain both classic delegate/data source patterns and modern diffable and compositional APIs.

## Learning goals

By the end of this file you should be able to:

- Explain table view data source and delegate responsibilities.
- Describe cell reuse and why it matters.
- Explain self-sizing cells and their tradeoffs.
- Use `UITableViewDiffableDataSource`.
- Explain `UICollectionViewCompositionalLayout`.
- Use `UICollectionViewDiffableDataSource` and snapshots.
- Describe cell registration APIs.
- Discuss prefetching, image loading, and performance pitfalls.

## UITableView mental model

A table view is a scrolling list optimized for vertical presentation of rows.
It asks its data source for counts and cells.
It asks its delegate about selection, heights, display events, and interaction details.

### Data source responsibilities

Typically:

- Number of sections.
- Number of rows per section.
- Cell configuration.
- Optional editing and moving support.

### Delegate responsibilities

Typically:

- Responding to selection.
- Display customization hooks.
- Height estimation or layout-related callbacks in older styles.
- Swipe actions and other behavior.

```swift
import UIKit

final class MessagesViewController: UIViewController {
    private let tableView = UITableView(frame: .zero, style: .plain)
    private var messages: [Message] = []

    override func loadView() {
        view = UIView()
        view.backgroundColor = .systemBackground
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        tableView.translatesAutoresizingMaskIntoConstraints = false
        tableView.dataSource = self
        tableView.delegate = self
        tableView.register(MessageCell.self, forCellReuseIdentifier: MessageCell.reuseIdentifier)

        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
}

extension MessagesViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        messages.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        guard let cell = tableView.dequeueReusableCell(
            withIdentifier: MessageCell.reuseIdentifier,
            for: indexPath
        ) as? MessageCell else {
            fatalError("Unexpected cell type")
        }

        cell.configure(with: messages[indexPath.row])
        return cell
    }
}

extension MessagesViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
    }
}
```

## Cell reuse

List views do not create a brand-new cell for every visible row forever.
That would be wasteful.
Instead, they reuse cell instances as rows scroll on and off screen.

> 🎯 **Interview Answer:** “Cell reuse is a recycling mechanism that keeps scrolling fast and memory usage controlled. The key engineering responsibility is making cell configuration cheap, idempotent, and safe under reuse.”

### Why reuse matters

- Reduces allocation cost.
- Lowers memory usage.
- Improves scrolling performance.
- Makes large lists feasible.

### The lifecycle of a reusable cell

1. The list asks for a cell.
2. The table or collection view dequeues an existing reusable cell or creates one.
3. You configure the cell with the current item.
4. The cell scrolls off screen.
5. The cell is recycled for another item later.

### `dequeueReusableCell`

This API is central.
The list view owns the reuse pool.
Your job is to register the cell type and configure the dequeued cell for the current item every time.

```swift
let cell = tableView.dequeueReusableCell(withIdentifier: MessageCell.reuseIdentifier, for: indexPath)
```

### Why stale content bugs happen

Cells are reused objects.
If you do not reset transient state, old state may appear in the wrong row.
Common examples include:

- Old images.
- Old accessory views.
- Old loading spinners.
- Old selection-dependent styling.
- Old async task results.

## `prepareForReuse()`

Override this in a custom cell to reset transient UI state before reuse.
Do not use it to rebuild the whole cell hierarchy.
That belongs in initialization.

```swift
import UIKit

final class MessageCell: UITableViewCell {
    static let reuseIdentifier = "MessageCell"

    private let avatarView = UIImageView()
    private let titleLabel = UILabel()
    private let subtitleLabel = UILabel()
    private var imageLoadTask: Task<Void, Never>?

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        selectionStyle = .none

        let stack = UIStackView(arrangedSubviews: [titleLabel, subtitleLabel])
        stack.axis = .vertical
        stack.spacing = 4

        let root = UIStackView(arrangedSubviews: [avatarView, stack])
        root.axis = .horizontal
        root.alignment = .center
        root.spacing = 12
        root.translatesAutoresizingMaskIntoConstraints = false

        contentView.addSubview(root)

        avatarView.translatesAutoresizingMaskIntoConstraints = false
        avatarView.widthAnchor.constraint(equalToConstant: 40).isActive = true
        avatarView.heightAnchor.constraint(equalToConstant: 40).isActive = true
        avatarView.layer.cornerRadius = 20
        avatarView.clipsToBounds = true

        NSLayoutConstraint.activate([
            root.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 12),
            root.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            root.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),
            root.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -12)
        ])
    }

    func configure(with message: Message) {
        titleLabel.text = message.senderName
        subtitleLabel.text = message.preview
        avatarView.image = UIImage(systemName: "person.crop.circle")

        imageLoadTask?.cancel()
        imageLoadTask = Task { [weak self] in
            guard let self else { return }
            let image = try? await AvatarService.shared.image(for: message.avatarURL)
            guard !Task.isCancelled else { return }
            self.avatarView.image = image
        }
    }

    override func prepareForReuse() {
        super.prepareForReuse()
        titleLabel.text = nil
        subtitleLabel.text = nil
        avatarView.image = UIImage(systemName: "person.crop.circle")
        imageLoadTask?.cancel()
        imageLoadTask = nil
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}
```

This is a strong interview example because it addresses both reuse and async image loading correctness.

> ⚠️ **Pitfall:** If you launch async image loading in a cell and do not cancel or verify identity on reuse, images can appear in the wrong row after scrolling.

## Self-sizing cells

Self-sizing cells let Auto Layout determine cell height from content.
This is very useful for dynamic text and flexible layouts.
It also has cost.

### Benefits

- Natural support for dynamic content.
- Easier adaptation to localization and Dynamic Type.
- Less manual height math.

### Tradeoffs

- More Auto Layout work during scrolling.
- Poorly constrained cells can become expensive or unstable.
- Estimated heights matter for performance.

### Basic setup

```swift
import UIKit

tableView.rowHeight = UITableView.automaticDimension
tableView.estimatedRowHeight = 88
```

### Good self-sizing cell practices

- Use a stable subview hierarchy.
- Fully constrain content top to bottom and leading to trailing.
- Avoid frequent constraint churn.
- Avoid expensive work in `layoutSubviews`.

## `UITableViewDiffableDataSource`

Diffable data sources let you model list state as snapshots instead of manually coordinating insertions and deletions.
This reduces invalid update crashes and makes state transitions clearer.

### Why they matter

Traditional `performBatchUpdates` or `beginUpdates` code can be fragile.
Diffable data sources centralize list state in a snapshot.
The system computes the differences.

```swift
import UIKit

enum InboxSection: Hashable {
    case main
}

struct InboxItem: Hashable {
    let id: UUID
    let title: String
    let subtitle: String
}

final class InboxViewController: UIViewController {
    private let tableView = UITableView(frame: .zero, style: .plain)
    private lazy var dataSource = makeDataSource()

    override func loadView() {
        view = UIView()
        view.backgroundColor = .systemBackground
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    private func makeDataSource() -> UITableViewDiffableDataSource<InboxSection, InboxItem> {
        UITableViewDiffableDataSource(tableView: tableView) { tableView, indexPath, item in
            let cell = tableView.dequeueReusableCell(withIdentifier: BasicCell.reuseIdentifier)
                as? BasicCell ?? BasicCell(style: .subtitle, reuseIdentifier: BasicCell.reuseIdentifier)
            cell.textLabel?.text = item.title
            cell.detailTextLabel?.text = item.subtitle
            return cell
        }
    }

    func apply(items: [InboxItem], animatingDifferences: Bool = true) {
        var snapshot = NSDiffableDataSourceSnapshot<InboxSection, InboxItem>()
        snapshot.appendSections([.main])
        snapshot.appendItems(items, toSection: .main)
        dataSource.apply(snapshot, animatingDifferences: animatingDifferences)
    }
}
```

### Snapshot mental model

A snapshot is the complete or partial desired state of sections and items.
You mutate the snapshot, then apply it.
The data source computes the differences.

## UICollectionView

Collection views are more general than table views.
They support grids, carousels, complex compositions, and highly customized layouts.
Modern collection view APIs are especially important in senior UIKit interviews.

## `UICollectionViewCompositionalLayout`

Compositional layout lets you build complex layouts declaratively using items, groups, and sections.
This replaced a lot of custom layout subclassing for common patterns.

### Key building blocks

- Item.
- Group.
- Section.
- Supplementary items.
- Decoration items.

### Example — two-column grid

```swift
import UIKit

func makeGridLayout() -> UICollectionViewLayout {
    let itemSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(0.5),
        heightDimension: .estimated(180)
    )
    let item = NSCollectionLayoutItem(layoutSize: itemSize)
    item.contentInsets = NSDirectionalEdgeInsets(top: 8, leading: 8, bottom: 8, trailing: 8)

    let groupSize = NSCollectionLayoutSize(
        widthDimension: .fractionalWidth(1.0),
        heightDimension: .estimated(180)
    )
    let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item, item])

    let section = NSCollectionLayoutSection(group: group)
    section.contentInsets = NSDirectionalEdgeInsets(top: 16, leading: 16, bottom: 16, trailing: 16)

    return UICollectionViewCompositionalLayout(section: section)
}
```

### Why interviewers like compositional layout

It shows whether you can think in layout composition rather than one-off frame math.
It also reflects modern UIKit adoption.

## `UICollectionViewDiffableDataSource`

The collection view counterpart to table diffable data sources uses the same snapshot concept.
It is often paired with compositional layout.

```swift
import UIKit

enum GallerySection: Hashable {
    case featured
    case grid
}

struct GalleryItem: Hashable {
    let id: UUID
    let title: String
}

final class GalleryViewController: UIViewController {
    private lazy var collectionView = UICollectionView(frame: .zero, collectionViewLayout: makeLayout())
    private lazy var dataSource = makeDataSource()

    override func loadView() {
        view = UIView()
        view.backgroundColor = .systemBackground
    }

    override func viewDidLoad() {
        super.viewDidLoad()
        collectionView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(collectionView)

        NSLayoutConstraint.activate([
            collectionView.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            collectionView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            collectionView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            collectionView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    private func makeLayout() -> UICollectionViewLayout {
        UICollectionViewCompositionalLayout { _, _ in
            let itemSize = NSCollectionLayoutSize(
                widthDimension: .fractionalWidth(1.0),
                heightDimension: .estimated(120)
            )
            let item = NSCollectionLayoutItem(layoutSize: itemSize)
            item.contentInsets = NSDirectionalEdgeInsets(top: 8, leading: 8, bottom: 8, trailing: 8)

            let groupSize = NSCollectionLayoutSize(
                widthDimension: .fractionalWidth(0.5),
                heightDimension: .estimated(120)
            )
            let group = NSCollectionLayoutGroup.horizontal(layoutSize: groupSize, subitems: [item])

            let section = NSCollectionLayoutSection(group: group)
            return section
        }
    }

    private func makeDataSource() -> UICollectionViewDiffableDataSource<GallerySection, GalleryItem> {
        UICollectionViewDiffableDataSource(collectionView: collectionView) { collectionView, indexPath, item in
            let cell = collectionView.dequeueReusableCell(withReuseIdentifier: GalleryCell.reuseIdentifier, for: indexPath) as! GalleryCell
            cell.configure(title: item.title)
            return cell
        }
    }

    func apply(_ items: [GalleryItem]) {
        var snapshot = NSDiffableDataSourceSnapshot<GallerySection, GalleryItem>()
        snapshot.appendSections([.grid])
        snapshot.appendItems(items, toSection: .grid)
        dataSource.apply(snapshot, animatingDifferences: true)
    }
}
```

## `NSDiffableDataSourceSnapshot`

The snapshot is the state container you mutate and apply.
You can append, delete, reload, or reconfigure items depending on API needs.
It is worth being able to explain this type by name in interviews.

### Good phrasing

“Snapshots describe the desired list state. Applying one lets UIKit compute the diff and update the UI safely.”

## Cell registration API

Modern UIKit offers registration APIs that reduce identifier string handling and centralize cell configuration.
These are a good sign of up-to-date UIKit knowledge.

```swift
import UIKit

let registration = UICollectionView.CellRegistration<UICollectionViewListCell, GalleryItem> { cell, _, item in
    var content = UIListContentConfiguration.cell()
    content.text = item.title
    cell.contentConfiguration = content
}

let cell = collectionView.dequeueConfiguredReusableCell(using: registration, for: indexPath, item: item)
```

### Why registration is nice

- Less manual identifier management.
- Configuration is colocated with registration.
- Stronger typing.
- Cleaner code in many cases.

## Prefetching

Prefetching lets the list view notify you about content likely to be needed soon.
This is useful for image loading, lightweight data warming, and other ahead-of-scroll work.

### Why it helps

- Improves perceived performance.
- Hides network or decoding latency.
- Reduces visible loading flashes.

### Be careful

- Do not overfetch aggressively.
- Cancel prefetch work when rows are no longer likely to appear.
- Coordinate with reuse and caching.

## Avoid heavy work in `cellForRowAt`

`cellForRowAt` and collection view cell providers are on the scrolling path.
Keep them cheap.

### Avoid doing

- Synchronous network calls.
- Heavy image decoding.
- Rebuilding expensive attributed strings repeatedly if avoidable.
- Layout recalculation tricks that can be precomputed.
- Large database fetches.

### Prefer doing

- Cheap state application.
- Using precomputed view models.
- Using cached images or async pipelines.
- Starting lightweight async fetches that are identity-safe and cancelable.

> 💡 **Tip:** “Keep cell configuration idempotent and cheap” is an excellent sentence to use in list-view interviews.

## Image loading patterns

Image loading is where reuse bugs and performance issues often collide.
A good production pattern usually includes:

- Request identity.
- In-memory cache.
- Optional disk cache.
- Cancellation on reuse.
- Placeholder state.
- Optional prefetching.

### Good flow

1. Configure placeholder synchronously.
2. Check cache.
3. Start async load if needed.
4. Cancel on reuse.
5. Verify identity before applying result.

### Avoid

- Letting old tasks update reused cells.
- Decoding large images on the main thread.
- Starting duplicate requests for the same visible content without deduplication.

## Performance checklist for list screens

- Use reuse properly.
- Keep cell hierarchies stable.
- Prefer precomputed formatting when possible.
- Use estimated heights thoughtfully.
- Avoid synchronous work on the main thread.
- Use prefetching for likely-nearby content.
- Cancel irrelevant work.
- Measure with Instruments when scrolling performance matters.

## Senior-level discussion

Senior engineers think about list views as pipelines.
There is data state, diff application, cell configuration, async resource loading, and scrolling performance.
Each part has an owner.
The UI should not be doing business logic.
Cells should not become mini-view-controllers.
Controllers or view models should not push imperative updates row by row when a snapshot expresses the full state more clearly.

A few strong senior takeaways:

- Prefer stable view models over ad hoc formatting inside cells.
- Prefer diffable data sources for state-driven updates.
- Treat reuse as an identity problem as much as a memory problem.
- Pair self-sizing with disciplined layout.
- Use collection view for richer layouts rather than fighting table view into becoming a grid.

## Interview Q&A

### 1. What is the difference between a table view data source and delegate?

The data source provides the content structure and cells.
The delegate handles behavior, interaction, and certain display-related decisions.

### 2. Why does cell reuse exist?

To avoid creating a new cell for every row as you scroll, which would waste memory and hurt performance.
Reusable cells are recycled and reconfigured for new items.

### 3. What does `dequeueReusableCell` do?

It returns a reusable cell from the list’s reuse pool or creates one if needed, so you can configure it for the current item.

### 4. Why is `prepareForReuse()` important?

It resets transient state before the cell is reused so stale content, async tasks, or visual state do not bleed into another item.

### 5. What are self-sizing cells?

Cells whose size is determined by Auto Layout and content rather than fixed manual heights.
They are flexible but can cost more during scrolling.

### 6. What problem does diffable data source solve?

It reduces manual bookkeeping for inserts, deletes, and moves by letting you apply snapshots that represent desired state.
UIKit computes the diff safely.

### 7. What is `NSDiffableDataSourceSnapshot`?

It is the type used to describe sections and items in a diffable data source before applying that state to the UI.

### 8. Why would you use compositional layout?

To build complex collection view layouts declaratively using items, groups, and sections without writing a fully custom layout for many common cases.

### 9. What is the cell registration API?

A modern strongly typed way to register and configure cells without relying heavily on reuse identifier strings and scattered configuration logic.

### 10. Why should `cellForRowAt` stay cheap?

Because it is on the scrolling path.
Heavy work there directly harms scroll performance and responsiveness.

### 11. How do you avoid wrong images in reused cells?

Use placeholders, caching, cancel async loads on reuse, and verify the result still belongs to the current item before applying it.

### 12. When would you use prefetching?

When warming nearby content such as images or lightweight data can improve perceived performance without causing overfetch or wasted work.

### 13. When would you choose collection view over table view?

When the layout is more general than a simple vertical list, especially for grids, carousels, dashboards, or mixed sections with different layouts.

### 14. What is the senior-level summary of UIKit lists?

Treat them as state-driven rendering pipelines: stable data, safe diff application, disciplined reuse, cheap configuration, and carefully managed async resource loading.
