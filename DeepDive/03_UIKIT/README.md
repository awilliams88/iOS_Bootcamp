# Module 03 — UIKit

UIKit still matters in iOS interviews.
Even in teams adopting SwiftUI, many production apps remain hybrid.
Interviewers often probe UIKit because it exposes how well you understand app lifecycle, layout, rendering, reuse, navigation, and event handling.
This module is designed to help you explain UIKit with precision and senior-level tradeoff language.

## Why this module matters

UIKit questions often sound simple.
“What happens in `viewDidLoad`?”
“What is cell reuse?”
“What is Auto Layout?”
But strong answers require more than definitions.
They require you to explain ordering, responsibilities, performance consequences, and architectural tradeoffs.

This module focuses on that deeper layer.
It aims to help you sound like someone who has debugged real rendering issues, screen lifecycle bugs, navigation complexity, and scrolling performance problems.

> 💡 **Tip:** In UIKit interviews, a senior answer usually includes lifecycle timing, performance implications, and one production example.

## How to use this module

Read these files in order the first time.
UIKit concepts build on one another.
Lifecycle informs layout.
Layout informs cell sizing.
Navigation affects ownership and event flow.
Responder chain knowledge helps explain gestures, focus, and event routing.

Recommended routine:

- Read one file at a time.
- Sketch the lifecycle or hierarchy by hand.
- Explain the code samples aloud.
- Relate each concept to a bug you have seen or might plausibly debug.
- Practice the Q&A sections in timer-based speaking drills.

## Topic map

- [01-view-and-viewcontroller-lifecycle.md](./01-view-and-viewcontroller-lifecycle.md) — UIView hierarchy, UIViewController lifecycle, layout callbacks, deinit, and lifecycle pitfalls.
- [02-auto-layout-and-layout-cycle.md](./02-auto-layout-and-layout-cycle.md) — Constraints, priorities, hugging, compression resistance, intrinsic content size, stack views, safe area, and layout timing.
- [03-tableview-and-collectionview.md](./03-tableview-and-collectionview.md) — Data source/delegate, cell reuse, self-sizing, diffable data sources, compositional layout, and scrolling performance.
- [04-navigation-and-presentation.md](./04-navigation-and-presentation.md) — Navigation stacks, tabs, modal presentation, custom transitions, and coordinator patterns.
- [05-responder-chain-gestures-animations.md](./05-responder-chain-gestures-animations.md) — Event delivery, first responder, hit testing, gestures, UIView animation, property animators, and Core Animation.
- [06-programmatic-ui-patterns.md](./06-programmatic-ui-patterns.md) — Storyboards versus XIBs versus programmatic UI, `loadView`, UINib, code-based constraints, and reusable view composition.

## Learning outcomes

By the end of this module you should be able to:

- Explain the UIView tree and view controller containment.
- Walk through the view controller lifecycle in correct order.
- Explain Auto Layout’s constraint solving model at a practical level.
- Distinguish layout passes from rendering passes.
- Explain table and collection view reuse clearly.
- Discuss diffable data sources and compositional layout.
- Explain navigation stack behavior and modal presentation choices.
- Describe responder chain event routing.
- Compare UIView animation APIs with Core Animation.
- Explain when teams choose programmatic UI over Interface Builder.

## What interviewers usually probe

- Do you know the lifecycle order accurately?
- Can you explain where to put setup work?
- Do you understand `viewDidLoad` versus `viewWillAppear`?
- Can you explain why constraints break or become ambiguous?
- Do you know why cell reuse exists?
- Can you explain why not to do heavy work in `cellForRowAt`?
- Can you compare `UIView.animate` with `UIViewPropertyAnimator`?
- Can you describe modal presentation tradeoffs?
- Can you explain why many teams avoid storyboards?
- Can you talk about performance instead of only APIs?

## UIKit mindset

UIKit is stateful and lifecycle-driven.
That means many bugs are not “wrong syntax” bugs.
They are timing bugs.
They happen because a view was not loaded yet.
Or because constraints were not fully established.
Or because a cell reused stale state.
Or because the wrong controller owns navigation.

When answering UIKit questions, keep four dimensions in mind:

1. Lifecycle timing.
2. View hierarchy ownership.
3. Performance cost.
4. Architectural maintainability.

> 🎯 **Interview Answer:** “With UIKit, I think in terms of lifecycle, hierarchy, and rendering cost. Most bugs come from putting the right work in the wrong phase or from letting ownership boundaries get fuzzy.”

## Suggested reading sequence

### Pass 1 — Foundation

1. Learn lifecycle order.
2. Learn Auto Layout and the layout cycle.
3. Learn list views and reuse.
4. Learn navigation and presentation.
5. Learn responder chain and animation.
6. Learn programmatic UI patterns.

### Pass 2 — Performance lens

1. Re-read lifecycle and layout with performance in mind.
2. Re-read list views with scrolling cost in mind.
3. Re-read animations with layer-backed rendering in mind.
4. Re-read navigation with ownership and memory in mind.

### Pass 3 — Senior polish

1. Practice tradeoff answers.
2. Compare UIKit and SwiftUI honestly.
3. Explain when a screen should be fully programmatic.
4. Rehearse lifecycle and layout diagrams from memory.

## Fast glossary

### UIView

A rectangular region in the interface that participates in hierarchy, layout, drawing, and event handling.

### UIViewController

A controller object that manages a view hierarchy and participates in lifecycle transitions, presentation, and containment.

### Auto Layout

A constraint-based system for describing relationships between views so the system can compute frames.

### Safe area

The region of a view that avoids system bars, notches, and other interface obstructions.

### Responder chain

The ordered path UIKit uses to route events and actions through views, controllers, windows, and application objects.

### Cell reuse

The recycling mechanism list views use to avoid constantly creating new cells during scrolling.

### Diffable data source

A state-driven data source API that applies snapshots and computes insertions, deletions, and moves safely.

## Study checklist by file

### 01-view-and-viewcontroller-lifecycle.md

- I can list key lifecycle callbacks in order.
- I can explain what belongs in `loadView`.
- I can explain what belongs in `viewDidLoad`.
- I can explain why `viewWillAppear` can run many times.
- I can explain why `deinit` matters for leak detection.

### 02-auto-layout-and-layout-cycle.md

- I can explain `translatesAutoresizingMaskIntoConstraints`.
- I can define hugging and compression resistance.
- I can explain intrinsic content size.
- I can compare `setNeedsLayout` and `layoutIfNeeded`.
- I can explain the layout cycle order.

### 03-tableview-and-collectionview.md

- I can explain cell reuse.
- I can explain self-sizing cells.
- I can describe diffable data sources.
- I can discuss prefetching and image loading.
- I can explain why layout work in `cellForRowAt` is dangerous.

### 04-navigation-and-presentation.md

- I can explain push versus modal.
- I can describe tab and navigation controller roles.
- I can name common modal presentation styles.
- I can explain custom transitions at a high level.
- I can describe the coordinator pattern honestly.

### 05-responder-chain-gestures-animations.md

- I can explain hit testing.
- I can explain first responder.
- I can compare gesture recognizers.
- I can compare UIView animations with Core Animation.
- I can explain `UIViewPropertyAnimator` at a practical level.

### 06-programmatic-ui-patterns.md

- I can compare storyboard, XIB, and programmatic UI tradeoffs.
- I can explain when to override `loadView()`.
- I can set up constraints in code cleanly.
- I can describe reusable view component patterns.
- I can explain why some teams avoid storyboards.

## Whiteboard drills

Use these in mock interviews.

1. Draw a basic UIView hierarchy.
2. Draw the main view controller lifecycle order.
3. Draw the Auto Layout pass and layout pass.
4. Draw a table view reuse flow.
5. Draw a navigation stack push and pop.
6. Draw modal presentation over a presenting controller.
7. Draw the responder chain for a button tap.
8. Draw an animation stack from UIView to CALayer.
9. Draw a coordinator object above tab and navigation controllers.
10. Draw a programmatic view setup flow from `loadView` to constraints.

## Speaking drills

Practice each in 60 seconds, 2 minutes, and 5 minutes.

1. Explain `viewDidLoad` versus `viewWillAppear`.
2. Explain why `loadView` exists.
3. Explain Auto Layout in practical terms.
4. Explain hugging versus compression resistance.
5. Explain why cell reuse matters for performance.
6. Explain diffable data sources.
7. Explain push versus modal navigation.
8. Explain hit testing.
9. Explain why some animations should use Core Animation instead of `UIView.animate`.
10. Explain why many teams prefer programmatic UI.

## Production scenarios this module prepares you for

- Building screens with dynamic content.
- Debugging ambiguous or unsatisfiable constraints.
- Fixing table view jank and reuse bugs.
- Coordinating navigation across deep feature flows.
- Handling gestures that interact with scrolling.
- Migrating legacy storyboard-heavy screens to code.
- Debugging “view is nil” or “layout is wrong on first appearance” issues.
- Designing reusable view components for larger teams.

## Common misconceptions to remove early

### “`viewDidLoad` runs every time the screen appears.”

No.
It usually runs once per controller lifetime, after the view is loaded.
Appearance callbacks can run many times.

### “Auto Layout sets frames immediately when I activate constraints.”

Not necessarily.
Constraint changes typically participate in the next layout pass unless you force layout.

### “Collection views are just table views with more layout options.”

They overlap, but collection views are more general and have grown substantially with compositional layout and modern registration APIs.

### “Cell reuse means old content disappears automatically.”

No.
Cells are reused objects.
You must reset transient state properly.

### “Programmatic UI is always better than storyboards.”

No.
Many teams prefer it for scalability and merge safety, but there are valid contexts for storyboards and XIBs.
The senior answer is a tradeoff answer.

## Mock interview prep by theme

### Lifecycle

- When is the view hierarchy created?
- When is it safe to access outlets?
- When do you start or stop observers tied to visibility?
- When do you trigger analytics impressions?
- When do you cancel screen-specific work?

### Layout

- Why are my constraints ambiguous?
- Why does a label truncate instead of expanding?
- Why did the first frame look wrong until after animation?
- When should I call `layoutIfNeeded`?
- When should I avoid touching frames directly?

### Lists

- Why is scrolling janky?
- Why do reused cells show stale images?
- Why are self-sizing cells expensive?
- How do I apply large updates safely?
- How do I prefetch without overfetching?

### Navigation

- Who owns navigation decisions?
- When should a flow be modal instead of pushed?
- How do I coordinate nested flows?
- How do I pass data backward on dismissal?
- How do I implement an interactive custom transition?

### Events and animation

- Why is a touch not reaching my view?
- When does a gesture recognizer cancel touches?
- When should I use a property animator?
- What is the difference between model layer and presentation layer?
- Why does animation state snap back?

### Programmatic UI

- Why override `loadView()`?
- How should I organize view setup methods?
- How do I make reusable views easy to test?
- Why do teams worry about storyboard merge conflicts?
- When is a XIB still a good compromise?

## Final-week review plan

### Day 1

- Rehearse lifecycle from memory.
- Explain where setup belongs in each callback.
- Review common lifecycle pitfalls.

### Day 2

- Rehearse layout cycle and constraint priorities.
- Build one small screen programmatically.
- Explain safe area and stack view tradeoffs.

### Day 3

- Rehearse table and collection view reuse.
- Explain diffable data source snapshots.
- Review scrolling performance patterns.

### Day 4

- Rehearse navigation stack and modal styles.
- Explain custom transitions at a high level.
- Review coordinator pattern tradeoffs.

### Day 5

- Rehearse responder chain and gesture flow.
- Explain animation APIs.
- Review layer-backed animation concepts.

### Day 6

- Rehearse programmatic UI tradeoffs.
- Explain why teams choose code versus Interface Builder.
- Build a small reusable component.

### Day 7

- Run rapid-fire Q&A across the full module.
- Whiteboard two diagrams.
- Revisit weak areas only.

## Senior signal checklist

- I know the controller lifecycle in order.
- I can explain layout timing rather than guessing.
- I connect reuse to scrolling performance.
- I can discuss navigation ownership instead of pushing logic into random controllers.
- I can explain responder chain and hit testing precisely.
- I can compare animation APIs by capabilities and cost.
- I can explain why programmatic UI scales well for teams.
- I can give at least one production bug example per topic.
- I use tradeoff language instead of one-size-fits-all claims.
- I connect UIKit mechanics to maintainability.

## Rapid-fire review prompts

- What is the difference between `loadView` and `viewDidLoad`?
- Why can `viewWillAppear` run many times?
- What does `translatesAutoresizingMaskIntoConstraints` do?
- What is intrinsic content size?
- Why do content hugging and compression resistance matter together?
- Why do cells need `prepareForReuse()`?
- What problem does diffable data source solve?
- When do you push versus present modally?
- What is the responder chain?
- What is hit testing?
- When should I use `UIViewPropertyAnimator`?
- Why might a layer animation not update the final model state?
- When is `loadView()` override appropriate?
- Why do teams dislike storyboard merge conflicts?

## Senior-level discussion

UIKit is not just a legacy framework.
It is still an important way to evaluate engineering judgment.
The strongest candidates do three things especially well.

First, they explain timing accurately.
They know when views exist, when frames are reliable, and when appearance callbacks fire.

Second, they explain performance consequences.
They know that list rendering cost, constraint solving cost, and animation choices can affect the user experience directly.

Third, they explain architectural ownership.
They know that navigation, view configuration, and reuse boundaries should be deliberate rather than accidental.

A strong UIKit answer should usually include:

- The lifecycle phase.
- The object that owns the work.
- The performance implication.
- The tradeoff or alternative.

## Final advice

If Swift language questions test precision, UIKit questions test operational maturity.
Describe what happens first.
Describe what depends on that step.
Describe where bugs usually come from.
Describe how you would structure the code so the next engineer can reason about it.
That is what makes UIKit answers sound senior.

## Interview Q&A

### 1. Why does UIKit still matter in interviews?

Because many production apps are hybrid or still primarily UIKit-based, and UIKit questions reveal practical knowledge of lifecycle, layout, rendering, navigation, and performance.

### 2. What makes a UIKit answer sound senior?

Accurate timing, clear ownership, performance awareness, and honest tradeoff language rather than just naming APIs.

### 3. What are the highest-yield UIKit topics to review?

Lifecycle, Auto Layout, list view reuse, navigation ownership, event routing, animation choices, and code organization.

### 4. What is the best way to practice this module?

Draw the lifecycles and hierarchies from memory, explain code samples aloud, and rehearse tradeoffs with one real production example per topic.
