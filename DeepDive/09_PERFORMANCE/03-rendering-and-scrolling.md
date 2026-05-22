# Performance — Rendering & Scrolling

---

## The Rendering Pipeline

```
Layout → Display → Compositing

Layout: calculateFrames, layoutSubviews
Display: drawRect:, CoreGraphics drawing → bitmap
Compositing: GPU composites layers → screen
```

All three must complete in **16ms** for 60fps (8ms for 120fps ProMotion).

---

## Off-Screen Rendering

Off-screen rendering forces the GPU to render a layer in a separate pass before compositing. It's expensive.

### What Triggers Off-Screen Rendering

```swift
// ❌ Triggers off-screen rendering
layer.cornerRadius = 12
layer.masksToBounds = true     // Clips sublayers — forces off-screen pass

layer.shadowColor = UIColor.black.cgColor
layer.shadowOpacity = 0.5
layer.shadowOffset = CGSize(width: 0, height: 4)
// Shadow without shadowPath forces off-screen rendering

layer.shouldRasterize = true   // Rasterizes to bitmap — can be good or bad

// Effects that trigger: masks, group opacity, filters, shadows without path
```

### Fixes

```swift
// ✅ Corner radius — use CAShapeLayer mask or pre-clipped image
// Option 1: Rasterize (cache the off-screen result)
layer.cornerRadius = 12
layer.masksToBounds = true
layer.shouldRasterize = true       // Cache the clipped layer as bitmap
layer.rasterizationScale = UIScreen.main.scale

// Option 2: Use shadow path (prevents off-screen pass for shadows)
let shadowPath = UIBezierPath(roundedRect: bounds, cornerRadius: 12)
layer.shadowPath = shadowPath.cgPath

// Option 3: For images — clip at load time
extension UIImage {
    func rounded(radius: CGFloat) -> UIImage {
        let renderer = UIGraphicsImageRenderer(size: size)
        return renderer.image { ctx in
            let rect = CGRect(origin: .zero, size: size)
            UIBezierPath(roundedRect: rect, cornerRadius: radius).addClip()
            draw(in: rect)
        }
    }
}
```

> 💡 **Tip:** Use Instruments → Core Animation to detect off-screen rendering. Enable "Color Off-screen Rendered" in the Debug menu while profiling.

---

## Scroll Performance

### Cell Layout

```swift
// ❌ Do layout work in cellForRowAt
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath) as! UserCell
    cell.configure(with: users[indexPath.row])
    
    // ❌ Calculating height during cell setup
    let height = textLayout(for: users[indexPath.row].bio).height
    cell.bioHeightConstraint.constant = height
    return cell
}

// ✅ Pre-calculate heights, use self-sizing cells
// Self-sizing (preferred)
tableView.estimatedRowHeight = 80
tableView.rowHeight = UITableView.automaticDimension
// Cells use Auto Layout to size themselves

// ✅ Pre-calculate layout on background thread if needed
func preCalculateLayouts(for users: [User]) -> [NSAttributedString] {
    return users.map { buildAttributedString(for: $0) }
}
```

### Image Loading in Cells

```swift
// ❌ Synchronous image loading in cellForRowAt
cell.avatarImageView.image = UIImage(contentsOfFile: imagePath)  // NEVER

// ✅ Async with cancellation on reuse
class UserCell: UITableViewCell {
    private var loadTask: Task<Void, Never>?
    
    override func prepareForReuse() {
        super.prepareForReuse()
        loadTask?.cancel()  // Cancel previous load
        avatarImageView.image = nil
    }
    
    func configure(with user: User) {
        loadTask = Task {
            guard let url = user.avatarURL else { return }
            if let image = await ImageCache.shared.image(for: url) {
                guard !Task.isCancelled else { return }
                await MainActor.run { avatarImageView.image = image }
            }
        }
    }
}
```

### Prefetching

```swift
// Prefetch data before cells appear
extension UserListViewController: UITableViewDataSourcePrefetching {
    func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
        let users = indexPaths.map { users[$0.row] }
        ImagePrefetcher.shared.startPrefetching(urls: users.compactMap(\.avatarURL))
    }
    
    func tableView(_ tableView: UITableView, cancelPrefetchingForRowsAt indexPaths: [IndexPath]) {
        let users = indexPaths.map { users[$0.row] }
        ImagePrefetcher.shared.stopPrefetching(urls: users.compactMap(\.avatarURL))
    }
}
```

---

## Image Optimization

```swift
// ❌ Loading a 4000×3000 photo for a 50×50 thumbnail
imageView.image = UIImage(named: "large_photo")

// ✅ Downsample at load time
func downsample(imageAt url: URL, to pointSize: CGSize, scale: CGFloat = UIScreen.main.scale) -> UIImage? {
    let imageSourceOptions = [kCGImageSourceShouldCache: false] as CFDictionary
    guard let imageSource = CGImageSourceCreateWithURL(url as CFURL, imageSourceOptions) else { return nil }
    
    let maxDimensionInPixels = max(pointSize.width, pointSize.height) * scale
    let downsampleOptions: [CFString: Any] = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceShouldCacheImmediately: true,
        kCGImageSourceCreateThumbnailWithTransform: true,
        kCGImageSourceThumbnailMaxPixelSize: maxDimensionInPixels
    ]
    
    guard let cgImage = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, downsampleOptions as CFDictionary) else { return nil }
    return UIImage(cgImage: cgImage)
}
```

---

## Main Thread Checklist

```swift
// Always on main thread:
// - UIView/UIViewController updates
// - tableView.reloadData()
// - Any UIKit property changes

// Never on main thread:
// - JSONDecoding large responses
// - Image loading from disk
// - Database reads/writes
// - Heavy computation

// Verification tool: MainActor annotation
@MainActor
func updateUI() {
    // Compiler ensures this runs on main thread
    tableView.reloadData()
}
```

---

## Interview Q&A

**Q: What causes janky scrolling and how do you diagnose it?**  
A: Janky scrolling (< 60fps) is caused by work blocking the main thread — image loading on main thread, expensive cell layout, off-screen rendering, or large memory pressure causing page outs. Diagnose with Core Animation instrument (FPS drops), Time Profiler (what's blocking main thread), and enabling "Color Off-screen Rendered" in Core Animation debug overlay.

**Q: What is off-screen rendering and why is it bad?**  
A: Off-screen rendering happens when the GPU must render a layer in a separate offscreen buffer before compositing it on screen. Common triggers: `cornerRadius + masksToBounds`, shadows without `shadowPath`, complex masks. It's bad because it requires an additional GPU pass, doubles memory bandwidth, and can cause dropped frames during scrolling.

**Q: How do you efficiently load images in a UITableView cell?**  
A: Never load images synchronously. Use async loading with task cancellation in `prepareForReuse`. Cache images (NSCache for memory, disk for persistence). Downsample large images to thumbnail size before display. Implement `UITableViewDataSourcePrefetching` to start loads before cells appear on screen.

**Q: What is `shadowPath` and why does it help performance?**  
A: Without `shadowPath`, Core Animation must determine the shadow shape by analyzing all the layer's contents — which requires an offscreen pass. Setting `shadowPath` explicitly tells Core Animation the exact shape of the shadow, allowing it to render on-screen without an offscreen pass.

---

## Quick Revision

- 60fps = 16ms per frame; 120fps (ProMotion) = 8ms
- Off-screen rendering: triggered by `cornerRadius + masksToBounds`, shadows, masks
- Fix shadows: set `shadowPath` explicitly
- Fix corner radius: rasterize with `shouldRasterize = true` if static
- Scroll performance: no image loading on main thread, cancel tasks in `prepareForReuse`
- Prefetching: `UITableViewDataSourcePrefetching` to load before cell appears
- Image downsampling: use `CGImageSourceCreateThumbnailAtIndex` for large images
- Main thread: only UI updates; background: I/O, decoding, computation
