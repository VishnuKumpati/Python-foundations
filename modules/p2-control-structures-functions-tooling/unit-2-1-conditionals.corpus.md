---
topic_id: c9feb41c-1aee-404d-b359-d468df334892
title: Unit 2.1 - Conditionals
position_in_module: 1
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 2.1 - Conditionals — Topic Corpus

## 2. Prerequisites

- `1792db9e-4427-4ba3-b2b3-619b7a559105` — Unit 1.3, Operators & Expressions: this unit's entire premise depends on the comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`), the logical operators (`and`, `or`, `not`), chained comparisons, and the ideas of truthy/falsy values that were introduced there. A conditional is simply a place where those boolean expressions get *used* to steer a program.
- `3c04d4c0-e9ca-4a4a-9d5e-368effd5d162` — Unit 1.4, Statements, Conversion & Output: conditionals are built from statements, and this unit's worked example reuses type conversion (`int(...)`) and f-strings from that unit.

## 3. Learning Objectives

By the end of this topic, a learner will be able to:

- **Explain** how an `if` statement uses a boolean expression to decide whether a block of code runs.
- **Implement** multi-branch decisions using `if`/`elif`/`else` so that exactly one branch executes.
- **Describe** how indentation defines the boundaries of a conditional block in Python.
- **Apply** compound boolean expressions (`and`, `or`, `not`, chained comparisons) inside a condition.
- **Differentiate** a nested conditional from a flattened compound condition and judge which is more readable in a given case.
- **Construct** a compact conditional (ternary) expression for a simple two-value choice.

## 4. Introduction

Imagine a food delivery app. If your cart is over a certain amount, delivery is free; otherwise, you pay a fee. Every program you have written up to this point in the course runs the same way no matter what: line one, then line two, then line three, always in that order, regardless of what data it's given. That kind of program cannot decide anything — it cannot tell a big cart from a small one, or a correct password from a wrong one. Real software needs to *branch*: to look at a piece of data and choose one path over another.

That branching ability is called a **conditional**, and Python's tool for it is the `if` statement. You already know how to build the yes/no questions a conditional asks — comparison operators like `>=` and logical operators like `and` were covered in Unit 1.3 [1]. What was missing was a way to say "and if the answer is yes, do *this*; if no, do *that* instead." This unit supplies exactly that missing piece.

By the end of this topic you will be able to write a program that reacts differently depending on its input: an `if`/`elif`/`else` chain that sorts a value into one of several categories, a compound condition that checks two facts at once, a nested conditional for a decision that only makes sense after an earlier one is settled, and a one-line ternary expression for simple either/or choices. This is the first control structure in the course, and nearly everything taught afterward — loops, functions, and full applications — assumes you can already make a decision inside code.

## 5. Core Concepts

### 5.1 What a conditional is

A **conditional** is a control structure that runs one block of code or a different block, depending on whether a **condition** — a boolean expression, meaning one that evaluates to `True` or `False` — holds [1]. In Python the basic conditional keyword is `if`:

```python
temperature = 30

if temperature > 25:
    print("It is warm.")
```

Read this as "if the condition `temperature > 25` is `True`, run the indented line below it." Since `30 > 25` is `True`, `print("It is warm.")` runs. Had `temperature` been `20`, the condition would be `False` and Python would skip that line entirely, without raising any error — it simply moves on to whatever comes after the conditional [1].

A condition can be anything that evaluates to `True` or `False`: a comparison (`age >= 18`), a variable that already holds a `bool` value (`if is_logged_in:`), or a compound expression built from `and`/`or`/`not` (covered in 5.4). All of these were introduced as boolean-producing tools in Unit 1.3 — this unit is about *attaching* them to a branch of code.

### 5.2 Blocks and indentation

The **block** is the group of statements that belong to one branch of a conditional — the lines that run together if that branch's condition is met. Python does not use punctuation such as curly braces `{ }` to mark where a block starts and ends, the way some other languages do. Instead, it uses **indentation**: the whitespace at the start of a line. Every statement that belongs to the same block must be indented by the same amount, and the standard is 4 spaces per level [1].

```python
if temperature > 25:
    print("It is warm.")   # indented — inside the block
print("Done checking.")    # not indented — outside the block, always runs
```

Because indentation is part of Python's grammar rather than just a style preference, getting it wrong is not a cosmetic issue — it is a real error. If the lines inside a block are indented inconsistently, or mix tabs and spaces, Python raises an **`IndentationError`** and refuses to run the program at all [1]. Every `if`, `elif`, or `else` line ends with a colon (`:`), which is Python's signal that "the indented block that follows belongs to this branch."

### 5.3 Multi-branch decisions: `elif` and `else`

A single `if` can only decide between "run this block" and "don't." Most real decisions have more than two outcomes, which is where `elif` (short for "else if") and `else` come in [1]:

```python
if condition_1:
    statement(s)          # runs only if condition_1 is True
elif condition_2:
    statement(s)          # runs only if condition_1 is False and condition_2 is True
else:
    statement(s)          # runs only if every condition above was False
```

A **branch** is one of these possible paths — one `if`, `elif`, or `else` block. The rules governing them are strict and worth stating precisely [1]:

- A conditional must start with exactly one `if`. `elif` and `else` are both optional.
- There can be any number of `elif` branches, but at most one `else`, and it must be the last branch.
- Conditions are checked **in order, top to bottom**. Python runs the block belonging to the **first** condition that is `True`, and then skips every remaining `elif`/`else` — even if a later condition would also have evaluated to `True`.
- If no condition is `True` and there is no `else`, Python runs nothing from the conditional and continues with whatever comes next. This is silent — no error, no message — which is precisely why a missing `else` can hide a bug: an unexpected input is handled by doing nothing at all, and nobody is told.

This "first match wins, then stop" behavior means branch **order matters**. Consider classifying a cart value into a delivery-fee tier:

```python
cart_value = 350.0

if cart_value >= 500:
    delivery_fee = 0.0
elif cart_value >= 200:
    delivery_fee = 20.0
else:
    delivery_fee = 40.0
```

`cart_value >= 500` is `350.0 >= 500` → `False`, so Python moves to the `elif`. `cart_value >= 200` is `350.0 >= 200` → `True`, so `delivery_fee` becomes `20.0`, and the `else` is skipped even though it was never actually checked. If the broader condition (`cart_value >= 200`) had been written *before* the narrower one (`cart_value >= 500`), the narrower branch would become unreachable — every cart worth 500 or more also satisfies "at least 200," so it would always match the wrong tier first. Ordering from most specific to least specific (or most urgent to least urgent) is what keeps an `elif` chain correct [1].

### 5.4 Compound conditions inside an `if`

A **compound condition** combines two or more boolean expressions using `and`, `or`, or `not` — the same logical operators from Unit 1.3 — directly inside an `if`/`elif` [1]:

```python
cart_value = 350.0
is_premium_member = True

if cart_value >= 500 or is_premium_member:
    delivery_fee = 0.0
else:
    delivery_fee = 40.0
```

Here `cart_value >= 500` is `False`, but `is_premium_member` is `True`, and because the operator is `or`, only one side needs to hold for the whole condition to be `True`. Had the operator been `and`, both sides would need to be `True`. A **chained comparison** — Python's shorthand for a range check, such as `0 <= score <= 100`, equivalent to `score >= 0 and score <= 100` — was already introduced in Unit 1.3 and works exactly the same way when it is the condition of an `if`. When mixing `and` and `or` in one condition, wrap the parts in parentheses, e.g. `if (age >= 18 and has_id) or is_guest:`, so the intended grouping is unambiguous rather than relying on operator precedence rules that are easy to misremember [1].

### 5.5 Nesting

**Nesting** means placing one conditional entirely inside the block of another conditional [1]. It is appropriate when an inner decision only makes sense *after* an outer question has already been answered:

```python
restaurant_open = True
cart_value = 350.0
is_premium_member = True

if restaurant_open:
    if cart_value >= 500 or is_premium_member:
        delivery_fee = 0.0
    else:
        delivery_fee = 40.0
    print("Delivery fee: Rs.", delivery_fee)
else:
    print("Restaurant is currently closed.")
```

Computing a delivery fee only makes sense if the restaurant is open in the first place — asking "what's the fee?" for a closed restaurant is a meaningless question. That dependency is exactly why nesting is justified here: the outer `if restaurant_open:` is checked first, and the inner fee logic is only reached, and only evaluated, once the outer condition is confirmed `True`. If two conditions are simply both *required together*, with no such dependency, a flattened compound condition (`if a and b:`) almost always reads more clearly than two nested `if` blocks stacked on top of each other. Stacking three or four levels of nested `if` statements when a single `and`/`or` expression would say the same thing is a common source of hard-to-follow code [1].

### 5.6 The conditional (ternary) expression

Everything above is a **statement**: it directs which lines of code run, but the `if`/`elif`/`else` construct itself does not produce a value that can be stored or printed. Python also has a **conditional (ternary) expression** — a one-line form that *does* produce a value [1]:

```
value_if_true if condition else value_if_false
```

```python
age = 20
status = "adult" if age >= 18 else "minor"
print(status)
```

Python evaluates the condition `age >= 18` first: `20 >= 18` is `True`, so the entire expression evaluates to `value_if_true`, the string `"adult"` — `value_if_false` (`"minor"`) is never even looked at. That resulting value is then stored in `status` exactly as any ordinary assignment expression would be. Had `age` been `15`, the condition would be `False`, and the whole expression would instead produce `"minor"`. The defining difference from a plain `if`/`else` statement is this: a ternary expression is a value-producing expression, one you can assign, print, or hand to another piece of code, while an `if`/`else` statement only directs control flow and produces no value of its own [1]. The ternary form is best reserved for simple, single-line, two-outcome choices; chaining several ternaries together to cover more than two outcomes produces a line that is technically valid Python but difficult to read, and a full `if`/`elif`/`else` chain should be used instead [1].

## 6. Implementation

The mental procedure Python follows for any `if`/`elif`/`else` chain is the same every time, and it is worth internalizing as a fixed sequence of steps:

1. Evaluate the condition next to `if`. If it is `True`, run that block, then skip every `elif` and `else` below it and continue with whatever comes after the whole conditional.
2. If the `if` condition was `False`, move to the first `elif` (if one exists) and evaluate its condition. If `True`, run that block, then skip everything remaining.
3. Repeat step 2 for each subsequent `elif`, in the order they are written, stopping at the first one that is `True`.
4. If every `if` and `elif` condition was `False`, run the `else` block, if one is present.
5. If every condition was `False` and there is no `else`, run nothing from the conditional at all, and continue with the rest of the program.

For a ternary expression, the procedure is shorter: evaluate `condition`; if `True`, the whole expression's value is `value_if_true` (and `value_if_false` is never evaluated); if `False`, the value is `value_if_false` (and `value_if_true` is never evaluated).

Tracing this procedure by hand on paper before typing any code is a useful habit while this topic is new. Take the task-priority example: with `is_blocked = True` and `priority = 1`, step 1 checks `is_blocked` first — not the priority — because the blocked check is written as the very first branch. It is `True`, so its block runs immediately, and steps 2 and 3 (the `elif priority == 1:` branch that would otherwise match) are never even reached. Writing out each step by hand like this, before running the code, is exactly how to catch a wrong branch order or a missing `else` before either becomes a bug that only shows up with unusual input.

## 7. Real-World Patterns

Conditionals appear anywhere a program has to react differently to different data [1]:

- **Banking:** a withdrawal request checks `if amount <= balance:` before releasing funds — getting the branch order wrong here could allow an overdraft.
- **Payment systems:** a confirmation screen uses `if`/`else` to display "Payment Successful" or "Payment Failed," sometimes with a nested check that further classifies a failure as "Insufficient Balance" or "Bank Server Error."
- **E-commerce:** a checkout page decides shipping cost from cart value and membership status — the exact pattern used in this unit's delivery-fee examples.
- **Healthcare dashboards:** a temperature reading is classified into "Normal," "Fever," or "High Fever" using an `if`/`elif`/`else` chain, the same multi-branch shape used to classify task priority in a to-do app.
- **Booking systems:** `if seats_available > 0:` guards whether a ticket can be confirmed at all, often with a nested check for a passenger's concession category.
- **AI/ML systems:** a model's output confidence score is frequently turned into an accept/reject decision with a conditional — `if confidence >= 0.8:` accept the prediction automatically, otherwise flag it for a human to review [1]. The model itself is a more advanced topic covered later in the course; here it is simply the source of the number the `if` reacts to.

## 8. Best Practices

- Order an `elif` chain from most specific (or most urgent) condition to least specific — since only the first `True` match runs, the order determines which branch actually fires [1].
- Include a final `else` whenever an unexpected value is possible, rather than letting the program silently do nothing.
- Prefer a flattened compound condition (`if logged_in and is_admin:`) over two nested `if` blocks when both conditions are simply required together; reserve nesting for when the inner decision genuinely only makes sense after the outer one is settled.
- Parenthesize compound conditions that mix `and` and `or`, e.g. `if (age >= 18 and has_id) or is_guest:`, so the grouping is unambiguous at a glance.
- Use a chained comparison such as `0 <= marks <= 100` instead of `marks >= 0 and marks <= 100` for range checks — it is shorter and mirrors the mathematical notation directly.
- Keep the ternary expression to simple, single-line, two-outcome choices; switch to a full `if`/`elif`/`else` chain once more than two outcomes are involved.

### 8.1 Common mistakes to avoid

- **Using `=` instead of `==` in a condition.** `if age = 18:` is a `SyntaxError` in Python — `=` is the assignment operator from Unit 1.2, while `==` is the comparison operator a condition actually needs [1].
- **Wrong branch order in an `elif` chain.** Placing a broader condition before a narrower one can make the narrower one unreachable — for example, checking `score >= 50` before `score >= 90` means the `90` case never gets its own message, because the broader condition already matched first [1].
- **Forgetting `else`.** Leaving out `else` when an unexpected value is possible means the program silently does nothing for that case, which can hide a real problem instead of surfacing it [1].
- **Inconsistent indentation.** Mixing 2 spaces and 4 spaces, or tabs and spaces, within the same block raises an `IndentationError`; every line in a block must line up exactly [1].
- **Over-nesting.** Stacking three or four levels of nested `if` statements when a single compound condition with `and`/`or` would say the same thing is harder to read and easier to get wrong [1].

## 9. Hands-On Exercise

Using a food-delivery scenario, write, in order: (1) a plain `if`/`else` on a `restaurant_open` boolean that prints one of two messages; (2) an `elif` chain that sets a `delivery_fee` from a `cart_value` across three tiers; (3) the same chain with an added `or is_premium_member` compound condition; (4) the fee logic nested inside the `restaurant_open` check, since a fee is meaningless for a closed restaurant; and (5) a ternary expression that turns the final `delivery_fee` into a `"Free Delivery"` / `"Delivery Charges Apply"` label. Run each version with at least two different sets of input values — including one that should trigger the `else` branch and one that should trigger the ternary's `value_if_false` — and confirm the printed output matches your prediction before moving to the next step.

## 10. Key Takeaways

- A conditional runs one branch of code, and only one, based on whether a boolean expression evaluates to `True` or `False`.
- `if`/`elif`/`else` branches are checked top to bottom, and only the first `True` condition's block runs — the order of branches changes the outcome.
- Indentation is part of Python's grammar, not a style choice; inconsistent indentation raises an `IndentationError`.
- A compound condition combines tests with `and`, `or`, and `not`; nesting is justified only when an inner decision depends on an outer one already being settled, otherwise a flattened compound condition is clearer.
- A conditional (ternary) expression — `value_if_true if condition else value_if_false` — produces a value in one line, unlike an `if`/`else` statement, which only directs control flow.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
