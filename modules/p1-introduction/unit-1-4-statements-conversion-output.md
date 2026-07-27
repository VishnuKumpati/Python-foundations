# Statements, Conversion & Output

By now you can create a value, give it a name, and combine several of them with the operators from Unit 1.3 into a single expression. That is enough to compute something — but almost no real program stops there. Picture a food delivery app: a customer taps a quantity box and types `2`. On the screen it looks exactly like the number two. Underneath, though, everything typed into a form arrives as plain text — the character `2`, not the number `2`. Ask Python to multiply that text by a price and it will not quietly do what you meant; it will refuse outright, because text and numbers are different types, and Python never guesses on your behalf.

This chapter gives you the bridge you are missing: a precise way to turn "data that arrived as text" into "a number I can actually compute with," and, just as importantly, a way to turn a computed number back into something a person wants to read — `₹247.70`, not `247.69999999999999`. Along the way you will meet Python's two basic kinds of instruction (statements), the four conversion functions that move values between types on purpose, f-strings for building clean output, type hints and `match`/`case` for writing code that documents itself and reads clearly, and comments — the small habits that keep a script readable once it grows past a few lines.

None of what follows is exotic. It is the everyday glue every working Python programmer reaches for dozens of times a day, and by the end of this chapter — the last of Module I — you will have everything you need to start writing programs that make real decisions, which is exactly where Module II begins.

---

## Every line you write is a statement

A **statement** is one complete instruction that Python carries out. Every line of code you have written since Unit 1.1 has been a statement of some kind, even though nobody called it that yet. This unit sorts the ones you already know — and a couple of new variations — into two families you will keep meeting for the rest of the course.

An **assignment statement** binds a value to a name using `=`, for example `total = 45.0`. It *stores* something; on its own, it produces nothing you can see. An **expression statement** is an expression — from Unit 1.3 — written on its own line and evaluated for its result or effect, the clearest example being `print(total)`, which evaluates the name `total` and displays whatever it currently refers to.

Two compact forms of assignment are worth having in your toolkit from day one:

```python
a = b = 5
print(a, b)

x, y = 1, 2
print(x, y)
```

```
5 5
1 2
```

The first line is **chained assignment** — one statement binding the same value, `5`, to two names at once. The second is **tuple unpacking** — assigning several values in a single statement by matching position, left to right: `x` gets `1`, `y` gets `2`. The number of names on the left must match the number of values on the right, or Python raises an error; `x, y = 1, 2, 3` fails outright, because three values cannot be squeezed into two names.

Tuple unpacking hides a rule that is worth knowing precisely, because it explains a trick you will see constantly in other people's code: Python always evaluates the *entire* right-hand side before binding anything on the left.

```python
x = 1
y = 2
x, y = y, x
print(x, y)
```

```
2 1
```

Before checking that output, try predicting it yourself — most learners expect `x` and `y` to somehow overwrite each other along the way. They don't: Python reads the current values of `y` and `x` first, builds them into a pair, and only afterward assigns that pair to `x, y`. **This is exactly why Python can swap two variables in one line with no temporary variable at all — something languages without this rule cannot do so simply.**

## Converting values on purpose: type conversion

**Type conversion**, also called **casting**, means producing a new value of a different type from an existing one, without disturbing the original value at all. Python gives you four built-in functions for this, one for each type you met in Unit 1.2:

| Function | Converts to | Example | Result |
|---|---|---|---|
| `str(x)` | Text (`str`) | `str(42)` | `"42"` |
| `int(x)` | Whole number (`int`) | `int("5")`, `int(3.9)` | `5`, `3` |
| `float(x)` | Decimal number (`float`) | `float("3.14")`, `float(42)` | `3.14`, `42.0` |
| `bool(x)` | Truth value (`bool`) | `bool(0)`, `bool("hi")` | `False`, `True` |

Two behaviours here catch almost everyone at least once, so it is worth sitting with them rather than skimming past.

**`int()` on a float truncates — it chops off the decimal part and moves toward zero, it never rounds.** Predict the output of `int(3.9)` and `int(-3.9)` before you run them: the answers are `3` and `-3`, not `4` and `-4`. If you actually wanted rounding, `int()` is the wrong tool.

`int()` on text only succeeds if the text is digits only, with no decimal point anywhere in it. `int("5")` works fine; `int("3.14")` raises a **`ValueError`** — the error Python raises whenever a conversion function is handed text it cannot parse into the target type. Whenever a decimal point is even possible in the incoming text, reach for `float()` instead.

It helps to name the difference between conversion Python does for you and conversion you must ask for yourself. **Implicit conversion** happens automatically, but only between compatible *numeric* types: `3 + 4.0` becomes `7.0` because Python quietly promotes the `int` to a `float` before adding. Python never does this between unrelated types such as `str` and `int` — that always requires **explicit conversion**, meaning you write `int("5") + 3` yourself. Skip that step and `"5" + 3` raises a **`TypeError`** — the error Python raises when an operation is attempted between incompatible types. This is, without exaggeration, the single most common bug new programmers hit once their programs start dealing with data that arrived as text: a value typed into a form, read from a file, or handed back by an app's server *looks* like a number on the screen, but it is a `str` until you convert it yourself, every single time.

A short list of the mistakes this trips up most often:

- Adding `input()`'s result directly to a number without converting it first, and getting a confusing `TypeError`.
- Calling `int()` on text that contains a decimal point, `.`, and getting a `ValueError` instead of the expected number.
- Expecting `int()` to round instead of truncate, and being surprised when `int(9.99)` comes back as `9`.
- Forgetting that `bool("False")` is `True` — any non-empty string is truthy, including the literal text `"False"`.

## Strings you can join and read into

Two operations on strings round out what you need before formatting proper output. **Concatenation** joins strings together with `+`: `"Tea" + " " + "Bill"` produces `"Tea Bill"`. **Indexing** reads a single character out of a string by its position, counting from `0` — for `"Tea"`, position `0` is `"T"`, position `1` is `"e"`, position `2` is `"a"`. Both operations work on `str` values only; try `+` between a `str` and an `int` without converting first, and you are back to the same `TypeError` from the previous section. Type still governs what an operator is allowed to do, exactly as it did with arithmetic in Unit 1.3.

## Making numbers speak like humans expect: f-strings

A converted, correctly typed value still needs to be *displayed* the way a person expects to read it — two decimal places for a price, values lined up neatly in a column, not a raw float with fifteen digits after the point. Python's tool for this is the **f-string**: a string literal prefixed with the letter `f`, in which anything inside curly braces `{}` is treated as an expression Python evaluates and drops into the text at that exact spot.

```python
price = 19.5
print(f"Price: {price}")
```

```
Price: 19.5
```

The `{}` can hold *any* valid expression, not only a bare variable name — arithmetic, comparisons, even a function call — and whatever it evaluates to is converted to text and inserted right there. Try predicting the output of `f"Total: {price * 2}"` before you run it.

Often "just show it" is not enough — that is what a **format specifier** is for: text placed after a colon inside the `{}`, controlling exactly how the value is displayed.

| Format specifier | Meaning |
|---|---|
| `:.2f` | Show a float to 2 decimal places — ideal for money. |
| `:d` | Show an integer in plain decimal form. |
| `:>10` | Right-align the value inside a field 10 characters wide. |
| `:^15` | Centre the value inside a field 15 characters wide. |

Run all four together to see exactly what each one does:

```python
price = 19.5
quantity = 7
item = "Tea"

print(f"{price:.2f}")
print(f"{quantity:d}")
print(f"[{item:>10}]")
print(f"[{item:^15}]")
```

```
19.50
7
[       Tea]
[      Tea      ]
```

Read it line by line. `price` is `19.5`, and `:.2f` forces exactly two digits after the decimal point, giving `19.50` — this is precisely how you avoid a UPI receipt or e-commerce checkout page showing `499.00000001` instead of `499.00`. `quantity` is already `7`, and `:d` simply prints its plain digits. `item` is `"Tea"`, three characters long; `:>10` places it inside an invisible 10-character field and right-aligns it, padding with seven leading spaces — the square brackets in the example are not part of the specifier, they are only there so you can *see* the padding. `:^15` centres the same text in a 15-character field, splitting the twelve leftover spaces evenly, six on each side.

**The letter or symbol in a format specifier says what to do — round, pad, align — and the number says how wide the field should be.** Once that clicks for one example, it applies to every other f-string you write for the rest of this course.

Two older ways of formatting text — `.format()` and `%` formatting — exist, and you will occasionally meet them in older code, but f-strings, introduced in Python 3.6, are the modern, preferred choice: the variable sits directly where it is used instead of in a separate placeholder list, which makes the code far easier to read at a glance.

## Documenting intent without changing behaviour: type hints

A **type hint**, also called an **annotation**, is a colon-based note recording the type a variable is expected to hold, written as `name: type = value` — for instance `count: int = 0`. `count` is the name being annotated, `: int` records the expected type, and `= 0` is the ordinary assignment you already know.

Here is the detail worth getting precisely right, because it surprises almost every beginner the first time: **a type hint is never enforced by Python at runtime.** `age: int = "twenty"` runs without a single complaint, even though the hint says `int` and the value is plainly a `str`. Hints exist purely as documentation — for the next human reading your code, and for editor tools that can flag a mismatch before you even run anything — not as a promise Python checks for you. Say out loud, in one sentence, why a type hint and an actual type conversion (like `int(x)`) are solving two completely different problems — if the sentence comes easily, the distinction has landed.

## Choosing between a fixed set of options: `match`/`case`

`match`/`case` is Python's structural pattern matching statement, built for exactly one job: comparing a single value against a fixed list of candidate outcomes and running the first block that fits.

```python
match subject:
    case pattern_1:
        pass  # runs if subject matches pattern_1
    case pattern_2:
        pass  # runs if subject matches pattern_2
    case _:
        pass  # wildcard — runs if nothing above matched
```

`match subject:` names the value being compared, evaluated once. Each `case pattern:` is one candidate, checked top to bottom. `case _:` is the **wildcard pattern** — the underscore `_` matches anything not already claimed by a case above it, acting as a catch-all for the unexpected. Because Python stops checking the moment it finds a match, **the wildcard case must always be written last** — put it earlier, and every case beneath it becomes unreachable, since `_` would already have grabbed the match first.

Here it is doing real work, tracking a food delivery order the way an app's backend might:

```python
order_status = "OUT_FOR_DELIVERY"

match order_status:
    case "PLACED":
        print("Your order has been placed.")
    case "OUT_FOR_DELIVERY":
        print("Your order is on the way!")
    case "DELIVERED":
        print("Your order has been delivered.")
    case _:
        print("Status unavailable. Please check the app.")
```

```
Your order is on the way!
```

Since `order_status` equals `"OUT_FOR_DELIVERY"`, only that one block runs — `"PLACED"`, `"DELIVERED"`, and the wildcard are all skipped entirely, because matching stopped the moment a case fit. The same shape works just as naturally for a railway booking status (`"CONFIRMED"`, `"WAITLISTED"`, `"RAC"`) or a UPI payment result (`"SUCCESS"`, `"FAILED"`, `"PENDING"`) — anywhere your program needs to react to one value out of a small, known set.

## Comments and PEP 8: writing for the next reader

A **comment** is text starting with `#` that Python ignores completely — it exists only for the humans who read the code afterwards, including you, three months from now. A comment runs to the end of its line; Python has no separate symbol for a comment spanning several lines. **Good comments explain *why* a piece of code exists or a particular choice was made — not *what* the code does, since the code itself already shows that.**

**PEP 8**, Python's official style guide, is the same set of conventions Unit 1.2 introduced for naming (`snake_case`, not `TotalPrice`), now extended to spacing and layout: put spaces around `=`, write one statement per line, and use blank lines to separate a script's logical sections. None of this changes what your code does — it changes how quickly the next person, including you, can understand it.

## Try it yourself

Do this in a Colab cell before moving on. Given `raw_price = "89.5"` — text, exactly as it might arrive from an order form — and `quantity = 4`: convert `raw_price` to a `float`, compute `total = price * quantity`, and print one f-string line showing an item name right-aligned in a 12-character field, the quantity as a plain integer, and both `price` and `total` to two decimal places. Then write a `match`/`case` block for a `payment_mode` variable that can be `"CASH"`, `"UPI"`, or `"CARD"`, with a wildcard case for anything else, and test it by setting `payment_mode = "CARD"`.

---

### Key Terminology

- **Statement** — one complete instruction Python executes; assignment and expression statements are the two kinds covered here.
- **Chained assignment** — binding the same value to several names in one statement, e.g. `a = b = 5`.
- **Tuple unpacking** — assigning several values in one statement by matching position, e.g. `x, y = 1, 2`.
- **Type conversion (casting)** — producing a new value of a different type from an existing one, via `str()`, `int()`, `float()`, or `bool()`.
- **Truncation** — `int()`'s behaviour on a float: chopping the decimal part off toward zero, never rounding.
- **`ValueError`** — raised when a conversion function is given text it cannot parse into the target type.
- **Implicit conversion** — automatic conversion Python performs between compatible numeric types, e.g. `int` + `float`.
- **Explicit conversion** — conversion you must trigger yourself, e.g. `int("5")`, when types are not automatically compatible.
- **`TypeError`** — raised when an operation is attempted between incompatible types.
- **Concatenation** — joining strings together with `+`.
- **Indexing** — reading a single character from a string by its position, counting from `0`.
- **f-string** — a string literal prefixed with `f`, in which `{}` holds an expression Python evaluates and inserts as text.
- **Format specifier** — text after a colon inside `{}` in an f-string, controlling precision, alignment, or width.
- **Type hint (annotation)** — a `name: type = value` note documenting a variable's expected type; never enforced at runtime.
- **`match`/`case`** — Python's structural pattern matching statement, comparing one value against ordered candidate patterns.
- **Wildcard pattern (`_`)** — the catch-all `case` that matches anything not already matched; must always be written last.
- **Comment** — text starting with `#`, ignored by Python, written for human readers.
- **PEP 8** — Python's official style guide for naming, spacing, and layout.

### Mastery Checkpoint

Before moving to Unit 2.1, check that you can answer these without looking back:

1. Why does `x, y = y, x` correctly swap two variables' values in a single line, with no temporary variable involved?
2. What is the difference between what `int(3.9)` does and what a learner might expect it to do, and why does `int("3.14")` fail outright?
3. Given `price = 19.5`, what does `f"{price:.2f}"` produce, and what job is the `.2f` actually doing?
4. If `age: int = "twenty"` runs without any error, what does that tell you about what a type hint actually is — and is not?
5. In a `match`/`case` block, why must `case _:` always be the last case rather than the first?

### Summary

This unit closed the gap between "I can create and combine values" and "I can build a real, readable program" — you now know the difference between an assignment statement and an expression statement, can use chained assignment and tuple unpacking with confidence, and understand exactly why the no-temporary-variable swap works. You can convert values between types on purpose with `str()`, `int()`, `float()`, and `bool()`, and you know precisely where that conversion bites beginners — truncation instead of rounding, and a stray `ValueError` or `TypeError` when a value is not the type you assumed. You can build clean, aligned output with f-strings and format specifiers, document a variable's intended type with a type hint without expecting Python to enforce it, choose between a fixed set of outcomes with `match`/`case`, and write comments and PEP 8–consistent code that stays readable as a script grows. That closes out Module I — you now have the complete foundation of values, variables, operators, conversion, and output. Module II starts exactly where this leaves off: teaching your programs to actually make decisions, beginning with Unit 2.1 on conditionals.

### Additional Resources

- [Python Tutorial — official docs: "More Control Flow Tools" (statements overview)](https://docs.python.org/3/tutorial/controlflow.html)
- [Python 3 Documentation — Built-in Functions (`int()`, `float()`, `str()`, `bool()`)](https://docs.python.org/3/library/functions.html)
- [Python 3 Documentation — Format String Syntax](https://docs.python.org/3/library/string.html#format-string-syntax)
- [Python Tutorial — official docs: "Input and Output" (f-strings and formatting)](https://docs.python.org/3/tutorial/inputoutput.html)
- [PEP 634 — Structural Pattern Matching: Specification](https://peps.python.org/pep-0634/)
- [PEP 8 — Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [W3Schools — Python String Formatting](https://www.w3schools.com/python/python_string_formatting.asp)
