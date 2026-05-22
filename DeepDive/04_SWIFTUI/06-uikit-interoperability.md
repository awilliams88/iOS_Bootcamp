# SwiftUI — UIKit Interoperability

---

## Why Interoperability Matters

SwiftUI and UIKit coexist in production apps because:
- Legacy codebases are incrementally migrated to SwiftUI
- Some UIKit controls have no SwiftUI equivalent (UIActivityViewController, MKMapView before iOS 17, etc.)
- UIKit gives pixel-perfect control for complex custom views
- SwiftUI can be adopted screen-by-screen without a full rewrite

---

## UIViewRepresentable — UIKit View in SwiftUI

Wrap any `UIView` for use in SwiftUI:

```swift
// Wrapping UIActivityIndicatorView
struct ActivityIndicator: UIViewRepresentable {
    let isAnimating: Bool
    
    func makeUIView(context: Context) -> UIActivityIndicatorView {
        let indicator = UIActivityIndicatorView(style: .large)
        indicator.hidesWhenStopped = true
        return indicator
    }
    
    func updateUIView(_ uiView: UIActivityIndicatorView, context: Context) {
        isAnimating ? uiView.startAnimating() : uiView.stopAnimating()
    }
}

// Usage
ActivityIndicator(isAnimating: isLoading)
```

### Full Example — UITextView with Character Count

```swift
struct RichTextEditor: UIViewRepresentable {
    @Binding var text: String
    var onCommit: () -> Void = {}
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    func makeUIView(context: Context) -> UITextView {
        let textView = UITextView()
        textView.delegate = context.coordinator
        textView.font = .systemFont(ofSize: 16)
        textView.backgroundColor = .systemBackground
        textView.layer.borderColor = UIColor.separator.cgColor
        textView.layer.borderWidth = 1
        textView.layer.cornerRadius = 8
        return textView
    }
    
    func updateUIView(_ uiView: UITextView, context: Context) {
        // Only update if different (avoid cursor jumping)
        if uiView.text != text {
            uiView.text = text
        }
    }
    
    // Coordinator bridges UIKit delegates to SwiftUI
    class Coordinator: NSObject, UITextViewDelegate {
        var parent: RichTextEditor
        
        init(_ parent: RichTextEditor) {
            self.parent = parent
        }
        
        func textViewDidChange(_ textView: UITextView) {
            parent.text = textView.text
        }
        
        func textViewDidEndEditing(_ textView: UITextView) {
            parent.onCommit()
        }
    }
}

// Usage
@State private var bodyText = ""
RichTextEditor(text: $bodyText, onCommit: { savePost() })
    .frame(height: 200)
```

### The Coordinator Pattern

The `Coordinator` is a helper object that:
- Acts as UIKit delegate/data source
- Bridges UIKit callbacks → SwiftUI state changes
- Persists for the lifetime of the represented view

```swift
// Key lifecycle:
// 1. makeCoordinator() — called first, creates coordinator
// 2. makeUIView(context:) — creates the UIView, coordinator available via context.coordinator
// 3. updateUIView(_:context:) — called whenever SwiftUI state changes
// 4. dismantleUIView(_:coordinator:) — optional cleanup
```

> 💡 **Tip:** Store references to the parent in Coordinator as a `var`, not `let` — SwiftUI updates it on each re-render cycle.

> ⚠️ **Pitfall:** Don't call `updateUIView` with expensive operations. It's called on every SwiftUI re-render. Check if value actually changed before updating UIKit.

---

## UIViewControllerRepresentable — UIViewController in SwiftUI

```swift
struct ImagePickerView: UIViewControllerRepresentable {
    @Binding var selectedImage: UIImage?
    @Environment(\.dismiss) var dismiss
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    func makeUIViewController(context: Context) -> UIImagePickerController {
        let picker = UIImagePickerController()
        picker.delegate = context.coordinator
        picker.sourceType = .photoLibrary
        picker.allowsEditing = true
        return picker
    }
    
    func updateUIViewController(_ uiViewController: UIImagePickerController, context: Context) {
        // Usually nothing here for VCs
    }
    
    class Coordinator: NSObject, UIImagePickerControllerDelegate, UINavigationControllerDelegate {
        let parent: ImagePickerView
        
        init(_ parent: ImagePickerView) {
            self.parent = parent
        }
        
        func imagePickerController(
            _ picker: UIImagePickerController,
            didFinishPickingMediaWithInfo info: [UIImagePickerController.InfoKey: Any]
        ) {
            if let image = info[.editedImage] as? UIImage {
                parent.selectedImage = image
            }
            parent.dismiss()
        }
        
        func imagePickerControllerDidCancel(_ picker: UIImagePickerController) {
            parent.dismiss()
        }
    }
}

// Usage in SwiftUI
@State private var showPicker = false
@State private var selectedImage: UIImage?

Button("Pick Image") { showPicker = true }
    .sheet(isPresented: $showPicker) {
        ImagePickerView(selectedImage: $selectedImage)
    }
```

---

## UIHostingController — SwiftUI View in UIKit

Embed SwiftUI views inside UIKit:

```swift
// Basic usage
class ProfileViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let profileView = ProfileSwiftUIView(user: currentUser)
        let hostingVC = UIHostingController(rootView: profileView)
        
        addChild(hostingVC)
        view.addSubview(hostingVC.view)
        hostingVC.view.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            hostingVC.view.topAnchor.constraint(equalTo: view.topAnchor),
            hostingVC.view.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            hostingVC.view.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            hostingVC.view.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
        hostingVC.didMove(toParent: self)
    }
}
```

### Sizing UIHostingController

```swift
// Self-sizing (intrinsic content size)
let hostingVC = UIHostingController(rootView: BadgeView())
hostingVC.sizingOptions = .intrinsicContentSize  // iOS 16+

// Manual sizing
let size = hostingVC.view.systemLayoutSizeFitting(UIView.layoutFittingCompressedSize)
hostingVC.view.frame = CGRect(origin: .zero, size: size)
```

### Updating SwiftUI State from UIKit

```swift
class MyUIKitVC: UIViewController {
    @ObservedObject var viewModel = SharedViewModel()  // Shared state object
    private var hostingVC: UIHostingController<SwiftUIView>!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        hostingVC = UIHostingController(rootView: SwiftUIView(vm: viewModel))
        // ... embed hostingVC
    }
    
    @IBAction func buttonTapped(_ sender: Any) {
        viewModel.count += 1  // SwiftUI view updates automatically
    }
}
```

---

## Data Sharing Between UIKit and SwiftUI

```swift
// Shared ViewModel using ObservableObject
class AppViewModel: ObservableObject {
    @Published var user: User?
    @Published var isLoggedIn = false
}

// UIKit reads
class UIKitViewController: UIViewController {
    var viewModel: AppViewModel!
    private var cancellables = Set<AnyCancellable>()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        viewModel.$user
            .receive(on: DispatchQueue.main)
            .sink { [weak self] user in
                self?.updateUI(for: user)
            }
            .store(in: &cancellables)
    }
}

// SwiftUI reads
struct UserView: View {
    @ObservedObject var viewModel: AppViewModel
    var body: some View {
        Text(viewModel.user?.name ?? "Guest")
    }
}
```

---

## Migration Strategy

### Incremental Migration Approach

```
Phase 1: Leaf Views First
├── Replace simple, stateless UIKit views with SwiftUI
├── Wrap SwiftUI in UIHostingController
└── Keep UIKit as coordinator/container

Phase 2: Screen-by-Screen
├── Replace full UIViewControllers with SwiftUI views
├── Keep UIKit navigation (UINavigationController)
└── Use UIHostingController to embed full screens

Phase 3: Navigation
├── Migrate to SwiftUI NavigationStack
├── Replace UITabBarController with TabView
└── Full SwiftUI app entry point
```

> 💡 **Tip:** Start migration from leaf views (cells, sub-views, simple screens). These have fewer dependencies and lower risk. Leave navigation until last.

---

## Interview Q&A

**Q: What is the Coordinator in UIViewRepresentable and why is it needed?**  
A: SwiftUI views are value types (structs) so they can't be delegates — delegates require reference types. The Coordinator is a class (reference type) that acts as the UIKit delegate, bridging UIKit callbacks (delegate methods) back to SwiftUI state mutations via `@Binding`.

**Q: What's the difference between makeUIView and updateUIView?**  
A: `makeUIView` is called once to create the UIView. `updateUIView` is called every time SwiftUI state changes that could affect this view. Always check if the value actually changed before updating UIKit to avoid redundant work (e.g., avoid setting text if it's already the same, which would cause cursor to jump in UITextView).

**Q: How do you embed a SwiftUI view in a UIKit view controller?**  
A: Use `UIHostingController`. Create it with `UIHostingController(rootView: mySwiftUIView)`, add it as a child view controller (`addChild`, `addSubview`, `didMove(toParent:)`), and set up Auto Layout constraints for its view.

**Q: How do you pass data between UIKit and SwiftUI?**  
A: Use a shared `ObservableObject` (or `@Observable` class) as the source of truth. UIKit subscribes via Combine (`$property.sink`); SwiftUI uses `@ObservedObject` or `@StateObject`. Mutating the shared model updates both sides.

**Q: What's your strategy for migrating a UIKit app to SwiftUI?**  
A: Incremental migration — start with leaf views (cells, small components) using `UIViewRepresentable` in reverse (actually, wrap SwiftUI in `UIHostingController` for UIKit). Replace full screens over time. Migrate navigation last. Avoid a big-bang rewrite — it's risky and you'll miss edge cases.

---

## Quick Revision

- `UIViewRepresentable` — wrap UIKit view for use in SwiftUI
- `makeUIView` — called once (create), `updateUIView` — called on state change
- `Coordinator` — class that acts as UIKit delegate, bridges to SwiftUI
- `UIViewControllerRepresentable` — wrap UIViewController for SwiftUI
- `UIHostingController` — embed SwiftUI view in UIKit
- Share state via `ObservableObject` / `@Observable` between both worlds
- Migration: leaf views first → screens → navigation
