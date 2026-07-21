---
topic_id: 1792db9e-4427-4ba3-b2b3-619b7a559105
title: Unit 1.3 - Operators & Expressions
position_in_module: 3
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 1.3 - Operators & Expressions — Topic Corpus

## 2. Prerequisites

This topic builds directly on two earlier units in "P1 . Module I . Introduction":

- **{9ac73bbb-9280-4975-a311-3079a78c38c4} Unit 1.1 - The Python Environment** — you should already be comfortable running code in a Google Colab notebook cell and using `print()` to display a value.
- **{44580fc5-261e-4a0a-bc9b-22972428f96e} Unit 1.2 - Variables, Identifiers & Types** — you should already know how to create a variable with `=`, understand the `int`, `float`, `str`, and `bool` types, and know that Python is dynamically typed (a variable's type is decided by the value currently stored in it, not fixed in advance).

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Apply** Python's arithmetic operators (`+`, `-`, `*`, `/`, `//`, `%`, `**`) to build correct expressions, including telling true division apart from floor division.
- **Explain** operator precedence — the fixed order in which Python evaluates an expression containing more than one operator — and use parentheses to control that order on purpose.
- **Apply** comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`) to test relationships between values, and state that every comparison produces a `bool`.
- **Differentiate** between the assignment operator `=` and the equality operator `==`, the single most common mistake beginners make in this area.
- **Analyze** compound conditions built from the logical operators `and`, `or`, and `not`, and explain short-circuit evaluation in your own words.
- **Identify** which Python values count as truthy and which count as falsy, and describe how that feeds directly into logical operators.

## 4. Introduction

Imagine you are at a college snack stall. You know the price of a samosa and how many you bought — that's just data sitting there, like the variables you learned in Unit 1.2. But the moment you ask "how much do I owe in total?" or "do I have enough cash?", you are no longer just storing numbers — you are *doing something* with them. That "doing something" is exactly what this topic is about.

An **operator** is a symbol or short keyword that tells Python to perform an action on one or more values — add them, compare them, or combine two yes/no questions into one. Chain values, variables, and operators together and you get an **expression**: something Python can work through step by step until it collapses down to a single answer. `4 * 15` is an expression. So is `age >= 18`. So is `is_member and not is_banned`.

This topic covers five ideas you will lean on in nearly every Python program you ever write: doing arithmetic (calculations), knowing which operator Python runs first when there's more than one (precedence), asking true/false questions about values (comparison), combining those questions (logical operators), and Python's rule for judging *any* value as "true-ish" or "false-ish" (truthiness). Whether it's a food-delivery app totalling your bill, a banking app checking a withdrawal amount, or an e-commerce site deciding if you qualify for free shipping, this exact toolkit is running underneath. [1]

## 5. Core Concepts

### 5.1 Operators, Operands, and Expressions

An **operator** is a symbol (like `+`) or keyword (like `and`) that performs one specific operation on values. An **operand** is a value the operator works on. In `200 + 50`, the operator is `+` and the operands are `200` and `50`. [1]

An **expression** is any combination of values, variables, and operators that Python can evaluate down to exactly one result. `total = 200 + 50` contains the expression `200 + 50`; Python fully computes that expression to `250` *before* storing it in `total` with the assignment operator `=` from Unit 1.2. Every expression, however long, always reduces to one single value. [1]

### 5.2 Arithmetic Operators

**Arithmetic operators** perform mathematical calculations on `int` and `float` values (the number types from Unit 1.2). [1]

| Operator | Name | Example | Result |
|---|---|---|---|
| `+` | addition | `7 + 3` | `10` |
| `-` | subtraction | `7 - 3` | `4` |
| `*` | multiplication | `7 * 3` | `21` |
| `/` | true division | `7 / 2` | `3.5` |
| `//` | floor division | `7 // 2` | `3` |
| `%` | modulo (remainder) | `7 % 2` | `1` |
| `**` | exponentiation | `7 ** 2` | `49` |

Three of these deserve a closer look, because they trip up almost every beginner:

- **True division (`/`)** always produces a `float` result, even when the numbers divide evenly. `6 / 2` is `3.0`, not `3`. [1]
- **Floor division (`//`)** rounds the result *down* toward negative infinity, discarding whatever is left over. For positive numbers this feels like "just drop the decimal part" — `7 // 2` is `3`. But for negative numbers it is not the same as rounding toward zero: `-7 // 2` is `-4` (the true mathematical answer is `-3.5`, and rounding *down* from there lands on `-4`), not `-3`. [1]
- **Modulo (`%`)** gives you the remainder left over after floor division. It takes the **sign of the divisor** (the number on the right), so `-7 % 2` is `1`, not `-1`. Floor division and modulo always fit together with this identity: `(a // b) * b + (a % b)` reconstructs `a` exactly. [1]
- **Exponentiation (`**`)** raises a number to a power: `7 ** 2` means "7 squared," giving `49`.

The *type* of an arithmetic result depends on the operand types you started with: if both operands are `int`, then `+`, `-`, and `*` give back an `int`. The instant even one operand is a `float`, the result "promotes" to `float` — `7 + 3` is `10`, but `7 + 3.0` is `10.0`. Dividing by zero with `/`, `//`, or `%` does not silently give `0` or "infinity" — it raises a **`ZeroDivisionError`**, which is Python's way of stopping the program and reporting that this particular calculation is impossible. (`ZeroDivisionError` works the same way as the `NameError` and `SyntaxError` you saw in Unit 1.2 — Python halts and reports exactly what went wrong.) [1]

### 5.3 Operator Precedence and Associativity

When an expression contains more than one operator, Python does **not** simply read it left to right. It follows a fixed ranking called **operator precedence** — the rule that decides which operator gets evaluated first. [1]

From highest (evaluated first) to lowest (evaluated last):

1. `()` — parentheses, for grouping
2. `**` — exponentiation
3. `-x` — unary minus (negating a single value)
4. `*`, `/`, `//`, `%` — multiplication/division family
5. `+`, `-` — addition/subtraction
6. `<`, `<=`, `>`, `>=`, `==`, `!=` — comparisons
7. `not`
8. `and`
9. `or` — evaluated last

Two consequences fall directly out of this ladder. First, `**` binds *tighter* than unary minus, so `-2 ** 2` is read as "negate the result of `2 ** 2`," giving `-4`, not `4`. To actually square `-2`, you must write `(-2) ** 2`. Second, `2 + 3 > 4 and 1 < 2` is read as `((2 + 3) > 4) and (1 < 2)`, because arithmetic outranks comparison, and comparison outranks logical operators. [1]

**Associativity** is the tie-breaking rule for when two operators share the *same* precedence level. Python resolves ties by working left to right, so `20 / 4 * 2` is `(20 / 4) * 2 = 10.0`, not `20 / (4 * 2) = 2.5`. [1]

You never have to memorize this ladder. Any time the intended order isn't obvious at a glance, wrap the part you want computed first in parentheses `()`. Parentheses always override precedence, they cost nothing to add, and they can never turn a correct expression into a wrong one. [1]

### 5.4 Comparison Operators

A **comparison operator** compares two operands and always produces a `bool` (`True` or `False`) — never anything else. [1]

| Operator | Name | Example | Result |
|---|---|---|---|
| `==` | equal to | `5 == 5` | `True` |
| `!=` | not equal to | `5 != 3` | `True` |
| `<` | less than | `3 < 5` | `True` |
| `>` | greater than | `3 > 5` | `False` |
| `<=` | less than or equal to | `5 <= 5` | `True` |
| `>=` | greater than or equal to | `3 >= 5` | `False` |

Comparisons can be **chained**: writing `1 < x < 10` evaluates as a single combined condition, meaning "is `x` between 1 and 10?" — much closer to how you'd say it in plain English than writing it out as two separate comparisons joined by `and`. [1]

It is legal Python to compare values of different types, such as `5 == "5"` (a number compared to text). This does not crash — it simply evaluates to `False`, because a number and a string are never considered equal, regardless of how they look when printed. [1]

### 5.5 `=` versus `==` — the Single Most Common Beginner Mistake

This distinction is worth its own subsection because it causes more early bugs than anything else in this topic. [1]

| Aspect | `=` (Assignment) | `==` (Equality Comparison) |
|---|---|---|
| Purpose | Binds a name to a value (from Unit 1.2) | Asks whether two values are equal |
| Result | No result value — it performs an action | Always produces a `bool`: `True` or `False` |
| Example | `x = 5` — stores `5` in `x` | `x == 5` — asks "does `x` currently equal `5`?" |
| Where it's used | Only in a statement, to create or update a variable | Inside any expression — conditions, print statements, calculations |

A single `=` **stores** a value. A double `==` **asks a question** and hands back an answer. Reading code aloud, mentally say "gets" for `=` (`x gets 5`) and "equals?" for `==` (`x equals 5?`) — that habit alone prevents most of the confusion.

### 5.6 Logical Operators and Short-Circuit Evaluation

A **logical operator** — `and`, `or`, or `not` — combines or inverts `bool` values (or, as covered in 5.7, any value at all, via truthiness). [1]

| Operator | Name | Example | Result |
|---|---|---|---|
| `and` | true only if both sides are true | `True and False` | `False` |
| `or` | true if at least one side is true | `True or False` | `True` |
| `not` | flips a single value | `not True` | `False` |

`and` and `or` use **short-circuit evaluation**: Python stops evaluating the instant the final answer is already known, without ever looking at the remaining side. For `A and B`: if `A` is already falsy, the whole expression must be `False`, so `B` is never evaluated. For `A or B`: if `A` is already truthy, the whole expression must be `True`, so `B` is never evaluated. [1]

This has a real, sometimes surprising consequence: code written on the right-hand side of `and`/`or` that would normally raise an error (for example, a division by zero) may never actually run at all, because the left side already decided the outcome. Knowing this behaviour matters — it is not just trivia, it is *why* an expression like `False and expensive_check()` never calls `expensive_check()`.

### 5.7 Truthiness

**Truthiness** is Python's rule for treating *any* value — not only the literal `True`/`False` from the `bool` type in Unit 1.2 — as true-ish or false-ish inside a logical context (such as being an operand of `and`, `or`, or `not`). [1]

Python keeps the list of **falsy** values short and fixed: `False`, `0`, `0.0`, and `""` (the empty string). Every other value is **truthy** — including negative numbers and, importantly, the *text* `"False"`, which is a non-empty string and therefore truthy, not falsy. Wrapping any value in the `bool()` function (from Unit 1.2) reveals its truthiness directly: `bool("")` gives `False`, while `bool("hello")` gives `True`.

### 5.8 Putting It Together — A Worked Example

Consider a small snack-stall bill. This example uses every operator family from this topic in one place: [1]

```python
samosa_price = 15
samosa_qty = 4
packing_fee = 10
friends_sharing = 3
has_student_card = True
coupon_code = ""
order_cancelled = False

final_bill = samosa_price * samosa_qty + packing_fee
share_per_friend = final_bill // friends_sharing
leftover_rupees = final_bill % friends_sharing

qualifies_by_amount = final_bill >= 50
has_coupon = bool(coupon_code)
not_cancelled = not order_cancelled
discount_eligible = qualifies_by_amount and has_student_card and not_cancelled
free_packing = discount_eligible or has_coupon

print(final_bill)
print(share_per_friend, leftover_rupees)
print(discount_eligible, free_packing)
```

Walking through it line by line:

- `final_bill = samosa_price * samosa_qty + packing_fee` — no parentheses are needed because `*` already outranks `+` on the precedence ladder (Section 5.3): Python computes `15 * 4 = 60` first, then adds `10`, giving `70`.
- `share_per_friend = final_bill // friends_sharing` — floor division splits the bill into whole rupees per friend: `70 // 3` is `23`.
- `leftover_rupees = final_bill % friends_sharing` — modulo gives whatever floor division couldn't split evenly: `70 % 3` is `1`. Checking the identity from Section 5.2: `(23 * 3) + 1 = 70`, which reconstructs `final_bill` exactly.
- `qualifies_by_amount = final_bill >= 50` — a comparison; `70 >= 50` is `True`.
- `has_coupon = bool(coupon_code)` — relies on truthiness: an empty string is falsy, so this is `False`, with no need to write `coupon_code != ""`.
- `not_cancelled = not order_cancelled` — `not` flips the stored `False` to `True`.
- `discount_eligible = qualifies_by_amount and has_student_card and not_cancelled` — `and` needs every operand `True`; all three are, so the result is `True`.
- `free_packing = discount_eligible or has_coupon` — `or` needs only one side `True`; because `discount_eligible` is already `True`, this **short-circuits** and Python never even looks at `has_coupon`.

Output:

```
70
23 1
True True
```

## 6. Implementation

There is no algorithm to implement here, but there is a reliable mental procedure for reading any expression that mixes several operators, which mirrors how Python itself evaluates it:

1. **Scan for parentheses** `()` first — anything inside them is resolved before anything outside.
2. **Resolve exponentiation** `**` next, then unary minus `-x`.
3. **Resolve multiplication/division family** `*`, `/`, `//`, `%`, working left to right when several appear at the same level.
4. **Resolve addition/subtraction** `+`, `-`, again left to right.
5. **Resolve comparisons** `<`, `<=`, `>`, `>=`, `==`, `!=` — these now work on the arithmetic results from steps 1–4.
6. **Resolve logical operators**, in the order `not`, then `and`, then `or` — these now work on the `bool` results from step 5.

Walking through `2 + 3 > 4 and 1 < 2` with this procedure: step 4 computes `2 + 3 = 5`; step 5 computes `5 > 4 = True` and `1 < 2 = True`; step 6 computes `True and True = True`.

## 7. Real-World Patterns

- **UPI / payment apps** compare an entered amount against an account balance (`amount <= balance`) and combine that with a PIN check using `and` — short-circuit evaluation means the balance check is skipped the instant the PIN comparison has already failed. [1]
- **E-commerce checkout** pages calculate a final price with arithmetic (`subtotal - discount + delivery_fee`) and then use a comparison to decide whether free shipping applies. [1]
- **Food delivery apps** recompute a live logical expression — such as whether a "free delivery" banner should show — every time the cart total or distance changes. [1]
- **Banking systems** combine a minimum-balance check (`balance >= 10000`) and a fraud-flag check (`not is_flagged`) with `and` before allowing a transaction to proceed. [1]
- **Booking systems** (railway, flight, movie) check `available_seats > 0` before allowing a confirmation, and use `%` to distribute passengers evenly across coaches or rows. [1]

## 8. Best Practices

- Add parentheses to make your intended order of evaluation obvious, even when precedence would already give the right answer — clarity for the next reader matters more than saving a few characters. [1]
- Decide up front whether you need a fractional result or a whole-number count, and pick `/` or `//` accordingly — never assume one behaves like the other. [1]
- Only compare values of compatible types where the comparison actually means something; `5 == "5"` is legal but always `False`, since a number and text are never equal. [1]
- Prefer chained comparisons (`18 <= age < 65`) over the longer `18 <= age and age < 65` — they mean the same thing, but the chained form reads closer to plain English. [1]
- Rely on truthiness directly (`if cart_items:`) instead of writing it out longhand (`if cart_items != ""`) — Python code that uses truthiness idiomatically is considered cleaner style. [1]
- When a condition grows long, break it into separate variables with descriptive names (as in the worked example below) rather than writing one giant expression — it is far easier to check each piece. [1]

## 9. Hands-On Exercise

A snack stall sells `5` samosas at `₹15` each and `2` cold drinks at `₹20` each, plus a `₹10` packing fee. Write code that: (1) computes `final_bill` with arithmetic; (2) stores `qualifies = final_bill >= 100`; (3) stores `has_coupon = bool("")`; (4) combines them with `discount = qualifies and has_coupon`; (5) prints all four values and explain, in one sentence, why `discount` comes out `False` even though `qualifies` is `True`.

## 10. Key Takeaways

- An **operator** performs an operation on **operands**; an **expression** is any combination of values, variables, and operators that Python reduces to exactly one result.
- Python has two kinds of division — `/` (true division, always returns a `float`) and `//` (floor division, rounds toward negative infinity) — plus `%` (remainder, whose sign follows the divisor); mixing these up is a common source of bugs.
- **Operator precedence** is fixed (parentheses, then `**`, then unary minus, then `*`/`/`/`//`/`%`, then `+`/`-`, then comparisons, then `not`/`and`/`or`), but parentheses `()` always override it and should be used whenever the order isn't obvious.
- Every comparison operator produces a `bool`, and confusing the assignment operator `=` with the equality operator `==` is the single most common beginner mistake in this area.
- **Logical operators** (`and`, `or`, `not`) combine or invert conditions using **short-circuit evaluation**, and Python's fixed list of **falsy** values (`False`, `0`, `0.0`, `""`) — with everything else **truthy** — governs how any value behaves in a logical context.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
