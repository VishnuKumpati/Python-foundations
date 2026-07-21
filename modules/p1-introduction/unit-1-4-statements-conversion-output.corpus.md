---
topic_id: 3c04d4c0-e9ca-4a4a-9d5e-368effd5d162
title: Unit 1.4 - Statements, Conversion & Output
position_in_module: 4
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 1.4 - Statements, Conversion & Output — Topic Corpus

## 2. Prerequisites

- 9ac73bbb-9280-4975-a311-3079a78c38c4 — Unit 1.1 - The Python Environment (needed for: `print()`, string, running code in Google Colab)
- 44580fc5-261e-4a0a-bc9b-22972428f96e — Unit 1.2 - Variables, Identifiers & Types (needed for: variables, `=`, `int`/`float`/`str`/`bool` types, `type()`, dynamic typing)
- 1792db9e-4427-4ba3-b2b3-619b7a559105 — Unit 1.3 - Operators & Expressions (needed for: expressions, arithmetic operators, comparison operators, truthiness)

## 3. Learning Objectives

By the end of this unit, you will be able to:

- **Differentiate** an assignment statement from an expression statement, and use chained assignment and tuple unpacking to write compact, correct code.
- **Apply** `int()`, `float()`, `str()`, and `bool()` to convert values between types, correctly predicting truncation and truthiness outcomes.
- **Create** clean, aligned output using f-strings, embedding expressions and applying format specifiers such as `:.2f`, `:d`, `:>10`, and `:^15`.
- **Describe** the purpose of type hints (e.g. `count: int = 0`) and read a basic `match`/`case` block.
- **Apply** comments and basic PEP 8 style so that code stays readable to its author and to others.
- **Debug** the most common conversion errors — `TypeError` and `ValueError` — that occur when text-shaped data is used before being converted.

## 4. Introduction

Imagine a food delivery app. A customer types "2" into a quantity box on their phone. To the app's screen, that looks exactly like the number two. But underneath, everything typed into a form arrives as plain text — the characters `2`, not the number `2`. If the app tries to multiply that text by a price, Python will not silently do the right thing; it will refuse, because text and numbers are different types, and Python does not guess what you meant. Before this unit, you already know how to store values in variables (Unit 1.2) and combine them with operators (Unit 1.3). What you are missing is the piece that bridges "data that arrived as text" to "a number I can actually compute with" — and, just as importantly, the piece that turns a computed number back into something a human wants to read, like `₹247.70` instead of `247.69999999999999`.

This unit closes out Module 1 by giving you exactly those bridging tools. You will learn what a Python **statement** is in general, how to convert values between types on purpose (rather than by accident), how to build neatly formatted text output with f-strings, and how to keep code readable using comments, type hints, and a new way of choosing between fixed options called `match`/`case`. None of this is exotic; it is the everyday glue that turns "I computed something" into "I displayed it correctly" — the daily work of writing real programs [1].

## 5. Core Concepts

### 5.1 Statements: assignment vs. expression

A **statement** is one complete instruction that Python executes. Every line of code you have written so far is a statement of some kind [1]. This unit groups statements into two kinds you will use constantly:

- An **assignment statement** binds a value to a name using `=`, for example `total = 45.0`. It *stores* a value; it does not produce one you can see on its own.
- An **expression statement** is an expression (from Unit 1.3) written on its own line and evaluated for its result or side effect — the clearest example is `print(total)`, which evaluates the expression `total` and displays it.

Two compact assignment forms build on the basic assignment statement:

- **Chained assignment** binds the same value to several names in a single statement: `a = b = 5` gives both `a` and `b` the value `5`.
- **Tuple unpacking** assigns several values in one statement by matching positions left to right: `x, y = 1, 2` gives `x` the value `1` and `y` the value `2`. The number of names on the left must match the number of values on the right, or Python raises an error — `x, y = 1, 2, 3` fails because three values cannot fill two names.

A subtle but important rule makes tuple unpacking powerful: Python always evaluates the entire right-hand side *before* binding any names on the left. This is exactly why the no-temporary-variable swap works: `x, y = y, x` first reads the current values of `y` and `x` (in that order) and only afterward assigns them, so no separate temporary variable is needed to hold one value while the other is overwritten.

### 5.2 Type conversion (casting)

**Type conversion**, also called **casting**, means producing a new value of a different type from an existing value, without changing the original value at all. Python gives you four built-in functions for this, each converting to one of the types you already know from Unit 1.2:

| Function | Converts to | Example | Result |
|---|---|---|---|
| `str(x)` | Text (`str`) | `str(42)` | `"42"` |
| `int(x)` | Whole number (`int`) | `int("5")`, `int(3.9)` | `5`, `3` |
| `float(x)` | Decimal number (`float`) | `float("3.14")`, `float(42)` | `3.14`, `42.0` |
| `bool(x)` | Truth value (`bool`) | `bool(0)`, `bool("hi")` | `False`, `True` |

Two rules govern how these behave, and both are common sources of surprise for beginners [1]:

- `int()` on a float **truncates** — it chops off the decimal part and moves toward zero, it does not round. `int(3.9)` is `3` (not `4`), and `int(-3.9)` is `-3` (not `-4`).
- `int()` on text only works if the text is digit-only, with no decimal point: `int("5")` works, but `int("3.14")` raises a **`ValueError`** — the error Python raises when a conversion function is given text it cannot parse into the target type. If a decimal point is possible in the text, convert with `float()` instead.
- `bool()` follows the same truthiness rules from Unit 1.3: `0`, `0.0`, and `""` (the empty string) convert to `False`; almost everything else converts to `True`.

It is worth naming the difference between conversion Python does for you and conversion you must do yourself. **Implicit conversion** happens automatically, but *only* between compatible numeric types — `3 + 4.0` becomes `7.0` because Python promotes the `int` to a `float` for you. Python never does this between unrelated types like `str` and `int`. That always requires **explicit conversion**: writing `int("5") + 3` yourself. Without it, `"5" + 3` raises a **`TypeError`** — the error Python raises when an operation is attempted between incompatible types. This is the single most common bug new programmers hit when working with data that arrived as text, such as a value typed into a form, read from a file, or returned by a web request: it *looks* like a number, but it is a `str` until you convert it yourself, every time.

### 5.3 Strings you combine and read: concatenation and indexing

Two string operations round out what you need before formatting output. **Concatenation** is joining two strings together with `+`, for example `"Tea" + " " + "Bill"` produces `"Tea Bill"`. **Indexing** is reading one character out of a string by its position, where counting starts at `0` — for the string `"Tea"`, position `0` is `"T"`, position `1` is `"e"`, and position `2` is `"a"`. Both operations only work on `str` values; this is one more reason a value must be the correct type before you operate on it, echoing the conversion rules above.

### 5.4 f-strings and format specifiers

Once a value is the right type, it still needs to be *displayed* the way a person expects to read it — two decimal places for a price, values lined up neatly in a column. Python's tool for this is the **f-string**: a string literal prefixed with the letter `f`, in which anything written inside curly braces `{}` is treated as an expression that Python evaluates and inserts as text in that spot. For example:

```python
price = 19.5
print(f"Price: {price}")
```

prints `Price: 19.5`. The `{}` can hold *any* valid expression — not just a bare variable name, but arithmetic, comparisons, or function calls — and its result is converted to text and dropped into place.

Often you want more control than "just show it" — this is what a **format specifier** provides: text placed after a colon inside the `{}`, which controls exactly how the value is displayed.

| Format specifier | Meaning |
|---|---|
| `:.2f` | Show a float to 2 decimal places (ideal for money). |
| `:d` | Show an integer in plain decimal form. |
| `:>10` | Right-align the value inside a field 10 characters wide. |
| `:^15` | Center the value inside a field 15 characters wide. |

Run all four together to see exactly what each one does [1]:

```python
price = 19.5
quantity = 7
item = "Tea"

print(f"{price:.2f}")
print(f"{quantity:d}")
print(f"[{item:>10}]")
print(f"[{item:^15}]")
```

Output:
```
19.50
7
[       Tea]
[      Tea      ]
```

Reading this line by line: `price` is `19.5`, and `:.2f` forces exactly two digits after the decimal point, producing `19.50`. `quantity` is already the integer `7`, and `:d` simply displays it as plain digits. `item` is `"Tea"`, three characters long; `:>10` places it inside an invisible 10-character-wide field and right-aligns it, adding 7 leading spaces — the square brackets in the example are not part of the format specifier, they are only there so you can *see* where the padding spaces are. `:^15` does the same idea but centers `"Tea"` in a 15-character field, splitting the 12 leftover spaces evenly (6 before, 6 after). Once you can see the spaces in this one example, every other use of these four specifiers in this unit follows the same rule: the letter or symbol says *what* to do (round, pad, align), and the number says *how wide* the field should be.

f-strings are the modern, preferred way to build output. Two older approaches — `.format()` and `%` formatting — exist and you may see them in older code, but f-strings (introduced in Python 3.6) are more readable because the variable sits directly where it is used, rather than in a separate placeholder list, and they are the recommended choice for new code.

### 5.5 Type hints (annotations)

A **type hint** (or **annotation**) is a colon-based note recording the type a variable is expected to hold, written as `name: type = value` — for example `count: int = 0`. The `name` is the variable being annotated, `: type` records the expected type, and `= value` is the ordinary assignment, exactly as before. The critical thing to understand is that a type hint is **not enforced** by Python at runtime: writing `age: int = "twenty"` runs without any error at all. Hints exist purely as documentation — for humans reading the code and for editor tools that can warn you about mismatches — not as a guarantee Python checks for you.

### 5.6 `match`/`case`

`match`/`case` is Python's structural pattern matching statement, used to choose between a fixed set of possible values. It compares one value against a list of candidate patterns, checked top to bottom, and runs the first block whose pattern matches:

```python
match subject:
    case pattern_1:
        # runs if subject matches pattern_1
    case pattern_2:
        # runs if subject matches pattern_2
    case _:
        # wildcard — runs if nothing above matched
```

`match subject:` names the value being compared, evaluated once. Each `case pattern:` is one candidate, checked in order. `case _:` is the **wildcard pattern** — the underscore `_` matches anything not already matched by a case above it, acting as a catch-all. Because matching stops at the first hit, `case _:` should always be written last: placing it earlier makes every case below it unreachable, since the wildcard would already have claimed the match.

As a small example, a canteen order status might be reported like this:

```python
order_status = "PREPARING"

match order_status:
    case "PLACED":
        print("Order received by the canteen.")
    case "PREPARING":
        print("Your order is being prepared.")
    case "READY":
        print("Order ready for pickup!")
    case _:
        print("Status unavailable. Please check with the counter.")
```

Since `order_status` equals `"PREPARING"`, only that block runs, printing `Your order is being prepared.`; the remaining cases, including the wildcard, are skipped entirely.

### 5.7 Comments and PEP 8

A **comment** is text starting with `#` that Python ignores completely; it exists purely for humans reading the code. A comment extends only to the end of the line — Python has no separate symbol for a multi-line comment. Good comments explain *why* a piece of code exists or a choice was made, not *what* the code does — the code itself already shows what it does.

**PEP 8** is Python's official style guide, covering naming, spacing, and layout conventions such as putting spaces around `=`, writing one statement per line, and using blank lines to separate logical sections of a script. Following it consistently keeps code readable as a script grows from a few lines into something a whole team maintains.

## 6. Implementation

Putting every piece together, here is the general procedure for turning a raw, text-shaped input into a clean, displayed result — the exact shape of a canteen or delivery receipt line:

1. Receive the raw value (often text, e.g. a price or quantity typed into a form or returned by an app).
2. Convert it to the correct type immediately, using `int()` or `float()` as appropriate, before doing any arithmetic with it.
3. Compute the result you need using ordinary operators (Unit 1.3), now safe because every operand is the correct type.
4. Build an f-string that embeds the result with the right format specifier — `:.2f` for money, `:d` for a plain integer count, `:>N` or `:^N` for alignment.
5. If the program must report one of a fixed set of outcomes (an order status, a payment mode), use a `match`/`case` block with a trailing `case _:` wildcard so an unexpected value is still handled instead of silently doing nothing.

Worked through with real values:

```python
item: str = "Chicken Biryani"
price_text: str = "249.5"      # arrives as text, e.g. from an order API
quantity: int = 2
order_status = "OUT_FOR_DELIVERY"

price = float(price_text)      # convert text -> float (step 2)
total = price * quantity       # compute the line total (step 3)

print(f"{item:>18}: {quantity:d} x {price:.2f} = {total:.2f}")  # step 4

match order_status:            # step 5
    case "PLACED":
        print("Your order has been placed.")
    case "OUT_FOR_DELIVERY":
        print("Your order is on the way!")
    case "DELIVERED":
        print("Your order has been delivered.")
    case _:
        print("Status unavailable. Please check the app.")
```

Output:
```
  Chicken Biryani: 2 x 249.50 = 499.00
Your order is on the way!
```

`float(price_text)` turned `"249.5"` into `249.5` so the multiplication could run at all; without that conversion, `price_text * quantity` would have repeated the text rather than computed a total. The f-string's `:.2f` specifiers then pinned both `price` and `total` to exactly two decimals, and `:>18` right-aligned the item name in an 18-character-wide column, exactly as a real receipt would. `match` compared `order_status` against each case top to bottom and found it equal to `"OUT_FOR_DELIVERY"` on the second try, so only that message printed — `PLACED`, `DELIVERED`, and the wildcard never ran, because `match` stops at the first pattern that fits [1].

## 7. Real-World Patterns

- **Banking and FinTech:** account statements convert raw decimal values into two-decimal currency text using f-strings before they ever reach a customer's screen or PDF statement.
- **UPI and payment systems:** a transaction amount arrives from an app or API as text and must be converted to `float` before it can be validated or displayed; the transaction's status is commonly handled with a `match`/`case`-style decision between a fixed set of outcomes.
- **E-commerce:** a cart total computed with rounding noise is always formatted with `:.2f` before checkout, so a customer never sees a number like `499.00000001`.
- **Food delivery:** an order status such as `"PLACED"`, `"OUT_FOR_DELIVERY"`, or `"DELIVERED"` is exactly the kind of fixed set of values `match`/`case` is built to handle cleanly.
- **Railway booking systems:** a fare or seat count entered by a user arrives as text and is converted before being added into a total; the final ticket summary is built with f-strings for consistent column alignment [1].

## 8. Best Practices

- Convert input values to the correct type **immediately** after receiving them, rather than deep inside a later calculation where a failure is harder to trace back to its source.
- Prefer f-strings over joining strings with `+` or older formatting styles — they are more readable and less error-prone.
- Always format money and measurements with an explicit precision specifier (`:.2f`) rather than trusting a raw float to display cleanly on its own.
- Add type hints on variables whose purpose is not obvious from the name alone — they cost nothing to write and help both human readers and editor tools.
- Keep a `case _:` wildcard at the end of every `match` block, so an unexpected value is handled instead of being silently ignored.
- Write comments that explain **why** a choice was made, not **what** the code does — the code already shows what it does.
- Follow basic PEP 8 habits consistently: spaces around `=`, one statement per line, and blank lines separating logical sections.

## 9. Hands-On Exercise

Given `raw_price = "89.5"` (text, as it might arrive from an order form) and `quantity = 4`, write code that: (1) converts `raw_price` to a `float`; (2) computes `total = price * quantity`; (3) prints a single f-string line showing the item name right-aligned in a 12-character field, the quantity as a plain integer, and both `price` and `total` to two decimal places. Then add a `match`/`case` block for a `payment_mode` variable that can be `"CASH"`, `"UPI"`, or `"CARD"`, with a wildcard case for anything else, and test it with `payment_mode = "CARD"`.

## 10. Key Takeaways

- A **statement** is one complete instruction; **assignment statements** store values while **expression statements** (like `print()`) act immediately, and Python always evaluates the right-hand side of an assignment fully before binding any names, which is what makes `x, y = y, x` swap values correctly.
- `int()`, `float()`, `str()`, and `bool()` perform **explicit type conversion**: `int()` truncates toward zero rather than rounding, `int("3.14")` raises `ValueError`, and combining un-converted text with a number (`"5" + 3`) raises `TypeError`.
- **f-strings** embed expressions in `{}` and format them with specifiers such as `:.2f`, `:d`, `:>10`, and `:^15` to produce clean, aligned output, and are preferred over older formatting styles.
- **Type hints** (`count: int = 0`) document a variable's expected type for humans and tools but are never enforced by Python at runtime.
- **`match`/`case`** checks patterns top to bottom and runs the first match; the wildcard `case _:` must always come last, since anything after a matched case never runs.
- Comments (`#`) and basic PEP 8 style exist to keep code readable to its author and to teammates as a program grows beyond a few lines.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
