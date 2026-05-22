# Advanced — Swift Package Manager

SPM is Apple's first-party dependency manager and the tool for modularizing large iOS codebases.

---

## Package.swift

```swift
// swift-tools-version: 5.9
import PackageDescription

let package = Package(
    name: "NetworkKit",
    platforms: [.iOS(.v16), .macOS(.v13)],
    
    products: [
        // Library that other packages can import
        .library(name: "NetworkKit", targets: ["NetworkKit"]),
        // Executable target
        // .executable(name: "MyTool", targets: ["MyTool"])
    ],
    
    dependencies: [
        // Remote package dependency
        .package(url: "https://github.com/apple/swift-argument-parser", from: "1.0.0"),
        .package(url: "https://github.com/apple/swift-log", exact: "1.5.3"),
        // Range
        .package(url: "https://github.com/SomeOrg/SomePackage", "1.0.0"..<"2.0.0"),
        // Local package dependency
        .package(path: "../SharedModels"),
    ],
    
    targets: [
        .target(
            name: "NetworkKit",
            dependencies: [
                .product(name: "Logging", package: "swift-log"),
                "SharedModels"  // Local package target
            ],
            path: "Sources/NetworkKit",
            resources: [
                .process("Resources")  // Processes resources (images, etc.)
                // .copy("RawFiles")   // Copies files as-is
            ],
            swiftSettings: [
                .enableExperimentalFeature("StrictConcurrency")
            ]
        ),
        .testTarget(
            name: "NetworkKitTests",
            dependencies: ["NetworkKit"],
            path: "Tests/NetworkKitTests"
        )
    ]
)
```

---

## Local Packages for Modularization

```
MyApp/
├── MyApp.xcodeproj
├── Sources/
│   └── ... (app source)
└── Packages/
    ├── DesignSystem/
    │   ├── Package.swift
    │   └── Sources/DesignSystem/
    ├── NetworkKit/
    │   ├── Package.swift
    │   └── Sources/NetworkKit/
    └── SharedModels/
        ├── Package.swift
        └── Sources/SharedModels/
```

```swift
// App's Package.swift or Xcode project
// Add local packages via File → Add Package Dependencies → Add Local

// NetworkKit/Package.swift
let package = Package(
    name: "NetworkKit",
    products: [.library(name: "NetworkKit", targets: ["NetworkKit"])],
    dependencies: [
        .package(path: "../SharedModels")
    ],
    targets: [
        .target(name: "NetworkKit", dependencies: ["SharedModels"])
    ]
)
```

---

## Version Resolution

```swift
// Semantic versioning strategies
.package(url: url, from: "1.2.3")          // >= 1.2.3 and < 2.0.0 (minor + patch)
.package(url: url, exact: "1.2.3")         // Exactly 1.2.3
.package(url: url, "1.0.0"..<"2.0.0")     // Range
.package(url: url, .upToNextMinor(from: "1.2.0"))  // >= 1.2.0 and < 1.3.0
.package(url: url, .branch("main"))        // Track branch (not recommended for release)
.package(url: url, .revision("abc123"))    // Specific commit
```

---

## Binary Targets (XCFramework)

```swift
// For closed-source dependencies
.binaryTarget(
    name: "AnalyticsSDK",
    url: "https://cdn.example.com/AnalyticsSDK-1.0.0.xcframework.zip",
    checksum: "abc123..."  // SHA256 of the zip
),
// Or local
.binaryTarget(
    name: "LocalSDK",
    path: "Frameworks/LocalSDK.xcframework"
)
```

---

## Resources in Swift Packages

```swift
// In Package.swift target
.target(
    name: "DesignSystem",
    resources: [
        .process("Assets.xcassets"),  // Processes like Xcode (compresses images)
        .process("Fonts"),            // Processes font files
        .copy("Templates")            // Copies directory as-is
    ]
)

// Accessing resources in code
// SPM generates Bundle.module for packages
let image = UIImage(named: "logo", in: .module, compatibleWith: nil)
let font = Bundle.module.url(forResource: "Inter-Regular", withExtension: "ttf")
let template = Bundle.module.url(forResource: "email_template", withExtension: "html")
```

---

## SPM Plugins

```swift
// Build tool plugin — runs during build
.plugin(
    name: "GenerateStrings",
    capability: .buildTool(),
    dependencies: ["StringsGenerator"]
)

// Command plugin — run on demand
.plugin(
    name: "FormatCode",
    capability: .command(
        intent: .custom(verb: "format", description: "Format source code"),
        permissions: [.writeToPackageDirectory(reason: "Format files")]
    )
)

// Run: swift package plugin format
```

---

## Interview Q&A

**Q: SPM vs CocoaPods vs Carthage — tradeoffs?**  
A: SPM: first-party, integrated in Xcode, fastest compile times (source-based), no separate install step, best for new projects. CocoaPods: mature ecosystem, widest library support, generates Xcode workspace, `pod install` required. Carthage: pre-builds frameworks (faster CI), less Xcode integration, less popular now. SPM is the modern default; CocoaPods still needed for some libraries without SPM support.

**Q: How do you share resources (fonts, images) across local packages?**  
A: Define them in a dedicated package (e.g., `DesignSystem`). Use `.process()` resource declaration in `Package.swift`. Access via `Bundle.module` in the package code. The app that imports the package automatically bundles these resources.

**Q: How do you handle a dependency that doesn't support SPM?**  
A: Options: (1) Use CocoaPods alongside SPM (they can coexist). (2) Create a wrapper Swift package with a binary target using the framework's XCFramework. (3) Fork the library and add SPM support. (4) Find an alternative library that supports SPM.

---

## Quick Revision

- `Package.swift`: defines name, platforms, products, dependencies, targets
- `.library`: consumable by other packages; `.executable`: standalone binary
- `.target` + `.testTarget`: source + test targets
- Version resolution: `from:` (up to next major), `exact:`, range, branch
- Local packages: `path: "../PackageName"` for monorepo modularization
- Binary targets: `url:` + `checksum:` for XCFramework distribution
- Resources: `.process()` (optimized) vs `.copy()` (raw); access via `Bundle.module`
- SPM plugins: build tool (every build) or command (on demand)
