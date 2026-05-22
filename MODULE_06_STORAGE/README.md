# Module 06 — Storage & Persistence

Data persistence is critical for iOS apps. This module covers every storage layer from lightweight preferences to full relational databases.

---

## Topics

| # | File | Description |
|---|------|-------------|
| 01 | [UserDefaults & Keychain](01-userdefaults-and-keychain.md) | Lightweight storage, secure storage, when to use each |
| 02 | [File System & Data](02-file-system.md) | FileManager, app directories, reading/writing files, iCloud |
| 03 | [Core Data](03-core-data.md) | NSManagedObject, NSPersistentContainer, relationships, migration, threading |
| 04 | [SwiftData](04-swiftdata.md) | @Model macro, ModelContainer, queries, migration from Core Data |

---

## Key Interview Themes

- **Keychain vs UserDefaults** — security tradeoffs
- **Core Data threading** — always use `performAndWait` / `perform`
- **Core Data migration** — lightweight vs heavyweight
- **SwiftData** — iOS 17+ replacement, @Model macro
- **iCloud sync** — NSPersistentCloudKitContainer
