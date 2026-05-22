# ARC & Memory Management

## ARC

Automatic Reference Counting manages memory for reference types.

Applies to:
- classes
- actors

Not applied to:
- structs
- enums

## Ownership Mental Model

Strong references own objects.
When strong ownership reaches zero, deallocation occurs.

## Retain Cycle

```text
A ----strong----> B
B ----strong----> A
```

ARC cannot resolve cyclic ownership automatically.

## weak
- non-owning
- optional
- auto nil
- safe when lifetime uncertain

## unowned
- non-owning
- non-optional
- crashes if invalid access occurs
- use when lifetime guaranteed

## Common Sources
- closure captures
- delegates
- parent-child object graphs

## Tools
Detect leaks with:
- Memory Graph Debugger
- Instruments Leaks

## Interview Guidance
Most leaks in iOS are ownership graph mistakes, especially closures and delegates.
