# ARC & Memory Management

## What is ARC?

Automatic Reference Counting manages memory for reference types by tracking strong references.

Applies to:
- classes
- actors

Does not apply to:
- structs
- enums

## Mental Model

Each strong reference increments ownership.
When count reaches zero, object deallocates.

## Retain Cycle

```text
A ----strong----> B
B ----strong----> A
```

ARC cannot break cyclic ownership automatically.

## weak vs unowned

### weak
- non-owning
- auto nil
- optional
- safe for uncertain lifetime

### unowned
- non-owning
- non-optional
- crashes if invalid
- use when lifetime guaranteed

## Interview Answer

Weak is used when referenced objects may disappear. Unowned is used when lifetime is guaranteed.
