# Coding Challenges

Swift-focused live coding problems with solutions. Practice these until you can write them fluently.

---

## Pattern: Two-Sum (Warm-Up)

```swift
// Problem: Given an array of integers and a target, return indices of two numbers that sum to target.
// Assume exactly one solution exists.

func twoSum(_ nums: [Int], _ target: Int) -> [Int] {
    var seen: [Int: Int] = [:]  // value → index
    
    for (index, num) in nums.enumerated() {
        let complement = target - num
        if let complementIndex = seen[complement] {
            return [complementIndex, index]
        }
        seen[num] = index
    }
    return []  // Should not reach here per problem guarantee
}

// Time: O(n), Space: O(n)
// Test: twoSum([2, 7, 11, 15], 9) → [0, 1]
```

---

## Problem: Reverse a Linked List

```swift
class ListNode {
    var val: Int
    var next: ListNode?
    init(_ val: Int, _ next: ListNode? = nil) {
        self.val = val
        self.next = next
    }
}

func reverseList(_ head: ListNode?) -> ListNode? {
    var prev: ListNode? = nil
    var current = head
    
    while let node = current {
        let next = node.next  // Save next
        node.next = prev      // Reverse pointer
        prev = node           // Advance prev
        current = next        // Advance current
    }
    
    return prev  // New head
}

// Recursive version
func reverseListRecursive(_ head: ListNode?) -> ListNode? {
    guard let head = head, head.next != nil else { return head }
    let newHead = reverseListRecursive(head.next)
    head.next?.next = head
    head.next = nil
    return newHead
}
// Time: O(n), Space: O(1) iterative, O(n) recursive
```

---

## Problem: Flatten Nested Array

```swift
// Implement flatten for a nested array (any depth)

indirect enum NestedArray {
    case value(Int)
    case array([NestedArray])
}

func flatten(_ nested: [NestedArray]) -> [Int] {
    var result: [Int] = []
    for element in nested {
        switch element {
        case .value(let v):       result.append(v)
        case .array(let subArray): result.append(contentsOf: flatten(subArray))
        }
    }
    return result
}

// Swift Array built-in
let nested = [[1, 2], [3, [4, 5]], 6] as [Any]
// Use: array.flatMap { ... }  or reduce(into:)
```

---

## Problem: LRU Cache

```swift
// Implement an LRU cache with O(1) get and put.

class LRUCache {
    private class Node {
        let key: Int
        var value: Int
        var prev: Node?
        var next: Node?
        init(_ key: Int, _ value: Int) { self.key = key; self.value = value }
    }
    
    private let capacity: Int
    private var cache: [Int: Node] = [:]
    // Doubly-linked list: head = LRU, tail = MRU
    private let head = Node(0, 0)
    private let tail = Node(0, 0)
    
    init(_ capacity: Int) {
        self.capacity = capacity
        head.next = tail
        tail.prev = head
    }
    
    func get(_ key: Int) -> Int {
        guard let node = cache[key] else { return -1 }
        remove(node)
        insertAtTail(node)
        return node.value
    }
    
    func put(_ key: Int, _ value: Int) {
        if let existing = cache[key] {
            existing.value = value
            remove(existing)
            insertAtTail(existing)
        } else {
            if cache.count == capacity {
                // Remove LRU (head.next)
                if let lru = head.next, lru !== tail {
                    remove(lru)
                    cache.removeValue(forKey: lru.key)
                }
            }
            let node = Node(key, value)
            cache[key] = node
            insertAtTail(node)
        }
    }
    
    private func remove(_ node: Node) {
        node.prev?.next = node.next
        node.next?.prev = node.prev
    }
    
    private func insertAtTail(_ node: Node) {
        node.prev = tail.prev
        node.next = tail
        tail.prev?.next = node
        tail.prev = node
    }
}
// Time: O(1) get and put. Space: O(capacity)
```

---

## Problem: Decode String (Coding Interview Favorite)

```swift
// Input: "3[a2[c]]" → Output: "accaccacc"

func decodeString(_ s: String) -> String {
    var countStack: [Int] = []
    var stringStack: [String] = []
    var current = ""
    var k = 0
    
    for char in s {
        if char.isNumber {
            k = k * 10 + Int(String(char))!
        } else if char == "[" {
            countStack.append(k)
            stringStack.append(current)
            current = ""
            k = 0
        } else if char == "]" {
            let count = countStack.removeLast()
            let prefix = stringStack.removeLast()
            current = prefix + String(repeating: current, count: count)
        } else {
            current.append(char)
        }
    }
    return current
}
// Time: O(n * max_repetition), Space: O(n)
```

---

## Problem: Merge Intervals

```swift
// Input: [[1,3],[2,6],[8,10],[15,18]]
// Output: [[1,6],[8,10],[15,18]]

func merge(_ intervals: [[Int]]) -> [[Int]] {
    let sorted = intervals.sorted { $0[0] < $1[0] }
    var result: [[Int]] = []
    
    for interval in sorted {
        if let last = result.last, interval[0] <= last[1] {
            // Overlapping — extend
            result[result.count - 1][1] = max(last[1], interval[1])
        } else {
            result.append(interval)
        }
    }
    return result
}
// Time: O(n log n) for sort, Space: O(n)
```

---

## Problem: Binary Search

```swift
func binarySearch<T: Comparable>(_ array: [T], target: T) -> Int? {
    var left = 0
    var right = array.count - 1
    
    while left <= right {
        let mid = left + (right - left) / 2  // Avoid overflow
        if array[mid] == target {
            return mid
        } else if array[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return nil
}
// Time: O(log n), Space: O(1)
```

---

## Problem: iOS Specific — Thread-Safe Property

```swift
// Implement a thread-safe property wrapper using actors

actor SafeValue<T> {
    private var value: T
    
    init(_ value: T) { self.value = value }
    
    func get() -> T { value }
    func set(_ newValue: T) { value = newValue }
    func update(_ transform: (inout T) -> Void) { transform(&value) }
}

// Usage
let counter = SafeValue(0)
await counter.update { $0 += 1 }
let value = await counter.get()

// Alternative: using concurrent queue + barrier (GCD approach)
final class ThreadSafeArray<T> {
    private var array: [T] = []
    private let queue = DispatchQueue(label: "thread-safe-array", attributes: .concurrent)
    
    func append(_ element: T) {
        queue.async(flags: .barrier) { self.array.append(element) }
    }
    
    var count: Int {
        queue.sync { array.count }
    }
    
    subscript(index: Int) -> T {
        queue.sync { array[index] }
    }
}
```

---

## Problem: Async Debounce / Search

```swift
// Implement a debounced search (common in Swift UI interviews)

@MainActor
final class SearchViewModel: ObservableObject {
    @Published var query = ""
    @Published private(set) var results: [String] = []
    
    private var searchTask: Task<Void, Never>?
    
    func onQueryChanged(_ newQuery: String) {
        searchTask?.cancel()
        
        guard !newQuery.isEmpty else {
            results = []
            return
        }
        
        searchTask = Task {
            try? await Task.sleep(for: .milliseconds(300))  // Debounce
            guard !Task.isCancelled else { return }
            
            results = await performSearch(query: newQuery)
        }
    }
    
    private func performSearch(query: String) async -> [String] {
        // Network call or local filter
        return try? await searchService.search(query: query)
    }
}
```

---

## Problem: Observable Pattern (Without Combine)

```swift
// Implement a lightweight observable/event emitter

final class Observable<T> {
    private var observers: [(T) -> Void] = []
    
    var value: T {
        didSet { observers.forEach { $0(value) } }
    }
    
    init(_ value: T) { self.value = value }
    
    @discardableResult
    func observe(_ handler: @escaping (T) -> Void) -> () -> Void {
        observers.append(handler)
        handler(value)  // Emit current value immediately
        
        // Return unsubscribe function
        let index = observers.count - 1
        return { [weak self] in self?.observers.remove(at: index) }
    }
}

// Usage
let counter = Observable(0)
let unsubscribe = counter.observe { print("Count: \($0)") }
counter.value = 1  // Prints "Count: 1"
counter.value = 2  // Prints "Count: 2"
unsubscribe()      // Remove observer
counter.value = 3  // No print
```

---

## Interview Tips for Live Coding

1. **Clarify first** — "Can I assume the array is sorted?" "What should I return for empty input?"
2. **Talk through approach** — "I'm thinking a hash map for O(1) lookups..."
3. **Write the signature first** — function name, parameters, return type
4. **Handle edge cases explicitly** — empty arrays, nil, negative numbers
5. **Start simple** — brute force first, then optimize
6. **Complexity analysis** — always state time and space at the end
7. **Test with example** — walk through your code with a sample input

```swift
// Template for any coding problem
func solve(_ input: SomeType) -> ReturnType {
    // 1. Handle edge cases
    guard !input.isEmpty else { return defaultValue }
    
    // 2. Initialize data structures
    var result: ReturnType = initial
    
    // 3. Core algorithm
    for element in input {
        // ...
    }
    
    // 4. Return
    return result
}
```
