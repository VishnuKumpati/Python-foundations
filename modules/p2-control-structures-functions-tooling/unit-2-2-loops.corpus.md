---
topic_id: 0ef5b1c5-43be-4f7a-8e00-ac4ab29451e5
title: Unit 2.2 - Loops
position_in_module: 2
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 2.2 - Loops — Topic Corpus

## 2. Prerequisites

- Unit 2.1 - Conditionals (`c9feb41c-1aee-404d-b359-d468df334892`) — loops reuse the same indentation-as-grammar rule and boolean expressions that conditionals introduced; this unit assumes you can already write and read an `if`/`elif`/`else` block.
- Unit 1.3 - Operators & Expressions (`1792db9e-4427-4ba3-b2b3-619b7a559105`) — comparison operators (`<=`, `==`, etc.) and truthy/falsy values are used directly inside loop conditions.

## 3. Learning Objectives

- **Explain** how a `while` loop repeats a block based on a condition, and how a `for` loop repeats a block once for each item in a sequence.
- **Apply** `range()` to generate numeric sequences using `start`, `stop`, and `step`, and use it to drive a `for` loop a fixed number of times.
- **Apply** `enumerate()` to track a position while looping and `zip()` to walk two sequences together in step.
- **Differentiate** between `break` and `continue`, and predict exactly how each one changes a loop's execution.
- **Identify** the classic infinite-loop mistake and the off-by-one errors that come from misreading `range()`.
- **Construct** nested loops to solve grid-shaped problems such as tables, seat charts, and reports.

## 4. Introduction

Imagine a bank clerk who has to check every single transaction in today's statement for fraud. There might be ten transactions, or there might be ten thousand — the clerk cannot know in advance, and writing a separate line of code for each one is not a real option. Every program you have written up to this point in the course runs each line exactly once, from top to bottom. That works for small, fixed tasks, but it falls apart the moment you need to repeat an action — print every row of a report, ask for a password until it's correct, walk through every seat on a train looking for a free one.

Python solves this with a **loop**: an instruction that says "repeat this block of code, either while some condition stays true, or once for every item in a group of items." This unit covers the two loop tools Python gives you — `while` and `for` — plus the small toolkit that makes them useful in real programs: `range()`, `enumerate()`, `zip()`, and the `break`/`continue` keywords. By the end, you will be able to write a program that repeats itself correctly, stops when it should, and does not get stuck running forever.

## 5. Core Concepts

**Loop, loop body, and iteration.** A **loop** is a control structure that repeats a block of code multiple times, either until a condition becomes `False` or until a group of items is used up [1]. The block of code that gets repeated is called the **loop body** — it is the indented block underneath the loop's first line, using the same indentation-as-grammar rule you learned for `if` blocks in Unit 2.1. Each single run through the loop body is called an **iteration**. If a loop body runs five times, that loop has completed five iterations.

**The `while` loop.** A `while` loop repeats its body for as long as a given condition stays `True` [1]. Its shape is:

```python
while condition:
    # loop body
```

`condition` is any expression that evaluates to `True` or `False` — the same kind of boolean expression used in an `if` statement. Python checks `condition` *before* every single iteration, including the very first one. If `condition` is `False` from the start, the body never runs even once. For example:

```python
coach = 1
while coach <= 3:
    print("Checking Coach", coach)
    coach = coach + 1
```

This prints `Checking Coach 1`, `Checking Coach 2`, `Checking Coach 3`, then stops, because after the third iteration `coach` becomes `4`, and `4 <= 3` is `False` [1]. The line `coach = coach + 1` is essential: it is what moves the condition toward `False`. A **while** loop that never changes anything the condition depends on will keep running forever — this is called an **infinite loop**, a loop whose condition never becomes `False` so it never stops on its own [1].

**The `for` loop and iterables.** A `for` loop repeats its body once for each item in a sequence, with no condition to check yourself [1]. Its shape is:

```python
for item in sequence:
    # loop body
```

`item` is the **loop variable** — a name you choose, which takes on each item's value in turn, one per iteration. `sequence` is anything Python can walk through one item at a time; this general category is called an **iterable** (or just a **sequence** for the kinds you'll use here) — a string, the output of `range()`, `enumerate()`, or `zip()` are all examples [1]. For instance, `for letter in "hi":` runs the body twice, once with `letter` set to `"h"` and once with `letter` set to `"i"` [1]. Unlike a `while` loop, a `for` loop's number of iterations is decided entirely by how many items are in `sequence` — there is no separate stop condition to write or maintain, and no risk of an infinite loop, because the loop ends the moment the sequence runs out of items.

Choose between the two based on what you know before the loop starts: use a `for` loop whenever you already know what you are iterating over (a fixed count, a string, a ready-made group of values); use a `while` loop only when you are repeating until some condition changes and you cannot know the number of repetitions in advance (for example, "keep asking for a password until it's correct") [1].

**`range()`.** `range()` is a built-in function that produces a sequence of whole numbers (integers), controlled by up to three values: `start`, `stop`, and `step` [1]. `range(stop)` produces integers from `0` up to, but *not including*, `stop` — so `range(5)` produces `0, 1, 2, 3, 4`, never `5`. This "stop is exclusive" rule is one of the most common sources of confusion for beginners, and misreading it produces an **off-by-one error** — a bug where a loop runs one time too many or one time too few [1]. `range(start, stop)` starts counting from `start` instead of `0`: `range(2, 6)` produces `2, 3, 4, 5`. `range(start, stop, step)` counts by `step` instead of by ones: `range(0, 10, 2)` produces `0, 2, 4, 6, 8` [1]. `range()` is most often paired with a `for` loop when you need to repeat something a fixed number of times, or when you need actual numbers (like seat numbers or a position counter) rather than the items of an existing sequence.

**`enumerate()`.** `enumerate()` is a built-in function that pairs each item in a sequence with an index (a position number), starting at `0` by default [1]. `enumerate("hi")` produces the pairs `(0, 'h')` and `(1, 'i')`. You can change where the counting starts with a second argument: `enumerate("hi", 1)` produces `(1, 'h')` and `(2, 'i')` [1]. Inside a `for` loop, you write this as `for position, letter in enumerate("hi", 1):` — Python automatically splits each pair into the two names you provide. `enumerate()` is what you reach for whenever you need to know *where* you are in a sequence, not just *what* the current item is — for example, printing a roll number alongside a student's attendance status.

**`zip()`.** `zip()` is a built-in function that walks two (or more) sequences together, producing one item from each sequence per pass [1]. `zip("ab", "12")` produces the pairs `('a', '1')` and `('b', '2')`. A `for` loop can unpack each pair directly: `for letter, number in zip("ab", "12"):`. An important rule: `zip()` stops the moment the *shorter* of its sequences runs out — there is no error or warning, the extra items in the longer sequence are simply never produced [1]. Forgetting this is a common mistake: pairing two sequences of different lengths and being surprised that some items from the longer one never show up in the loop at all.

**`break` and `continue`.** These two keywords redirect a loop's flow from inside its own body, and both are only valid *inside* a loop body — using either one outside a loop raises a `SyntaxError` [1]. `break` ends the loop immediately: no further iterations run, not even the rest of the current one [1]. `continue` skips only the rest of the *current* iteration and jumps straight to the next one — it does not stop the loop the way `break` does [1]. Both are normally written directly behind an `if` check, since you only want to break or skip under a specific condition, not on every iteration.

**Nested loops.** A **nested loop** is a loop placed inside the body of another loop [1]. Nested loops are the standard tool for anything shaped like a grid — a seat chart with rows and columns, a report with sections and line items. A crucial rule governs their behavior: in a nested loop, `break` and `continue` only affect the *innermost* loop they are written in — they never reach out to stop or skip an outer loop [1]. If you want to stop both loops when something is found in the inner one, you need a separate signal (commonly a flag variable set to `True`) that the outer loop checks with its own `break` right after the inner loop finishes.

## 6. Implementation

The two loop types execute by different internal procedures, useful to trace step by step when debugging.

**How a `while` loop runs:**
1. Evaluate `condition`.
2. If `condition` is `False`, exit the loop — the body never runs (or never runs again).
3. If `condition` is `True`, run the loop body once (one iteration).
4. Go back to step 1.

**How a `for` loop runs:**
1. Ask the sequence for its next item. If there isn't one, exit the loop.
2. Assign that item's value to the loop variable.
3. Run the loop body once (one iteration).
4. Go back to step 1.

**How a nested loop runs:** for every single iteration of the outer loop, the *entire* inner loop runs start to finish before the outer loop moves to its next iteration. This means the total number of iterations across both loops is (outer iterations) × (inner iterations) — a calculation worth doing by hand whenever you write or read a nested loop, to sanity-check that it behaves the way you expect [1].

## 7. Real-World Patterns

Loops are not a classroom exercise — they are the mechanism behind everyday repeated behavior in software [1]:

- **Retry prompts:** an ATM PIN screen uses a `while` loop that keeps asking for a PIN until it is correct or a maximum number of attempts is reached, `break`-ing the instant the correct PIN is entered.
- **Polling:** a payment status checker uses `while True:` paired with `break` to keep asking "is this transaction done yet?" and stop the moment it succeeds or fails.
- **Totals over a collection:** an e-commerce cart total is calculated by a `for` loop that adds every item's price to a running total, one item per iteration — the "repeat for every item" pattern `for` loops exist for.
- **Skip-without-stopping:** a monitoring dashboard uses `continue` to skip a single bad sensor reading without ending the whole monitoring loop.
- **Grid-shaped search:** searching a train for a free seat across multiple coaches is a textbook nested loop — the outer loop walks coaches, the inner loop walks seats within a coach, and `break` fires the moment a seat is found.

## 8. Best Practices

- Default to a `for` loop when the thing you're repeating over is already known (a fixed range, a string, a ready-made group of values); reach for a `while` loop only when repetition depends on a condition that might change unpredictably [1].
- Before running a `while` loop, read its body back to yourself and confirm there is a line that moves the condition toward `False` — this is the single most effective way to catch an infinite loop before it happens [1].
- Prefer `range()` to manually tracking a counter with a `while` loop when you just need to repeat something a fixed number of times [1].
- Name loop variables for what they represent (`for student in ...`, not `for x in ...`) — it makes the loop body far easier to read [1].
- Keep nested loops to two levels where possible; a third level is usually a sign the logic should be broken down first [1].
- Use `break` and `continue` sparingly, and always pair each one with a clear `if` — scattering them through a loop body makes the flow hard to trace [1].
- Double-check any nested-loop `break`: remember it only exits the innermost loop, so stopping an outer loop requires its own separate exit check [1].

## 9. Hands-On Exercise

A cinema has 4 rows, numbered `1` to `4`, with 3 seats each, numbered `1` to `3`. Write nested `for` loops using two `range()` calls to print every seat in the format `"Row 1, Seat 1"` through `"Row 4, Seat 3"`. Then modify it: use `continue` to silently skip Row `2` entirely, and use a flag variable with `break` in both loops to stop the whole printout as soon as `"Row 3, Seat 2"` is reached.

## 10. Key Takeaways

- A **`while`** loop repeats as long as its condition stays `True` and must contain something that eventually makes the condition `False`, or it becomes an infinite loop.
- A **`for`** loop repeats once per item in a sequence and ends automatically when the sequence is exhausted — no manual counter or stop condition needed.
- `range(start, stop, step)` generates integers with an exclusive `stop` value, and `enumerate()`/`zip()` add position-tracking and side-by-side iteration on top of a basic `for` loop.
- `break` ends a loop immediately; `continue` skips only the current iteration — and in a nested loop, both only affect the innermost loop they are written inside.
- The two most common loop bugs are the infinite loop (a `while` condition that never turns `False`) and the off-by-one error (misreading `range()`'s exclusive stop).

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
