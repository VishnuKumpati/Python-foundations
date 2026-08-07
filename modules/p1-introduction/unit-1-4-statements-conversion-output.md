# 1.4 Statements, Conversion & Output

---

[← Previous: 1.3 Operators & Expressions](unit-1-3-operators-expressions.md) | [Go back to TOC](../../README.md) | [Next: 2.1 Conditionals →](../p2-control-structures-functions-tooling/unit-2-1-conditionals.md)


---

## What is a Statement?

A **statement** is one complete instruction Python executes. Everything you've written so far, `x = 5`, `print(x)`, `total = price * qty`, is a statement.

This unit is about turning "I computed something" into "I displayed it correctly": converting values, working with text, and formatting output cleanly.

Open a new cell in your Colab notebook and try each example as you go.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Statement** | One instruction Python executes |
| **Type conversion / casting** | Turning a value into a different type |
| **Truncation** | Chopping off decimals when converting to `int`, no rounding |
| **Index** | The position of a character, starting at `0` |
| **Slice** | A section of a string, pulled out with `start:stop:step` |
| **Immutable** | Can't be changed after creation, a string stays as-is |
| **f-string** | A string with `{}` that inserts a value directly |

---

## Assignment vs Expression Statements

- **Assignment statement**: stores a value: `x = 5`
- **Expression statement**: runs and produces a result/side effect: `print(x)`

Two shortcuts worth knowing:

```python
a = b = 5          # chained assignment: both a and b become 5
x, y = 1, 2        # unpacking: x=1, y=2

x, y = y, x        # swap: no temporary variable needed
```

Python always evaluates the right-hand side fully first, that's exactly what makes the swap work.

**One thing to watch:** the number of names and values must match, or Python raises an error.

```python
x, y = 1, 2, 3   # ValueError: 2 names, 3 values
```

---

## Type Conversion

Real data almost always arrives as text, a price from a form, an age typed by a user. You have to convert it before doing math with it.

**Python does a little conversion automatically**, but only between compatible number types:

```python
print(3 + 4.0)   # 7.0, int promotes to float, no work needed from you
```

It will never do this between `str` and a number on its own, that always needs an explicit function:

| Function | Converts to | Example | Result |
|---|---|---|---|
| `str(x)` | text | `str(42)` | `"42"` |
| `int(x)` | whole number | `int("5")` | `5` |
| `float(x)` | decimal number | `float("3.14")` | `3.14` |
| `bool(x)` | true/false | `bool(0)` | `False` |

**Watch out for two things:**

- `int()` **truncates**, it doesn't round: `int(3.9)` is `3`, not `4`. And `int(-3.9)` is `-3`, always moving toward zero.
- `int("3.14")` raises a `ValueError`, the string must look like a *whole* number. Use `float()` first if there's a decimal point.

```python
raw_quantity = "3"
quantity = int(raw_quantity)
print(quantity)
print(type(quantity))
```
```
3
<class 'int'>
```

**Strings and numbers don't mix without conversion:**

```python
"5" + 3        # TypeError: can't add text and a number
int("5") + 3   # 8, works once converted
```

---

## Strings, Quotes, Concatenation & Indexing

You've used strings since Unit 1.1, here's a proper look at how they actually work.

**Single vs double quotes** are interchangeable, Python doesn't care which you use, as long as the opening and closing quote match:

```python
name1 = "Ananya"
name2 = 'Ananya'
```

Use double quotes if your text contains an apostrophe, or single quotes if it contains a double quote, so you don't have to escape anything:

```python
message = "It's a great day"
quote = 'She said "hello"'
```

If your text has **both** a `'` and a `"` in it, you can't dodge it with quote choice alone, use a backslash `\` to escape the one that matches your outer quotes:

```python
mixed = "She said, \"It's a great day\""
print(mixed)
```
```
She said, "It's a great day"
```

**Concatenation** joins strings together with `+` (you saw this in Unit 1.3):

```python
first = "Chai"
second = "Point"
print(first + second)
```
```
ChaiPoint
```

**Indexing** lets you pull out one character by its position, starting at `0`:

```python
word = "Python"
print(word[0])    # P
print(word[1])    # y
print(word[-1])   # n, negative counts from the end
```

`-1` is always the last character, `-2` the second-to-last, and so on, handy when you don't know the length in advance.

**Go past the end, and Python complains:**

```python
word[10]   # IndexError: string index out of range
```

---

## Slicing, Getting a Section

Indexing gets one character. **Slicing** with `start:stop:step` pulls out a whole chunk instead. **Stop is never included**, this trips up almost everyone at first.

```python
word = "Python"
print(word[0:3])   # Pyt, index 0, 1, 2 (not 3)
print(word[2:])     # thon, from index 2 to the end
print(word[:4])     # Pyth, from the start to index 3
print(word[::2])    # Pto, every 2nd character
print(word[::-1])   # nohtyP, reversed
```

This exact `start:stop:step` syntax works on lists too (coming up in Unit 3.1), once it clicks here, it's already familiar there.

---

## Common String Methods

Methods are actions you call with a dot. They don't change the original string, they hand back a **new** one.

| Method | Does what | Example | Result |
|---|---|---|---|
| `.upper()` | ALL CAPS | `"hi".upper()` | `"HI"` |
| `.lower()` | all lowercase | `"HI".lower()` | `"hi"` |
| `.strip()` | removes leading/trailing spaces | `"  hi  ".strip()` | `"hi"` |
| `.replace(a, b)` | swaps text | `"hi".replace("h", "H")` | `"Hi"` |
| `.split(sep)` | splits into a list | `"a,b,c".split(",")` | `['a', 'b', 'c']` |
| `.join(list)` | joins a list into one string | `"-".join(['a','b'])` | `"a-b"` |
| `len(text)` | length of the string | `len("hi")` | `2` |

```python
raw_input = "  Ananya Roy  "
clean_name = raw_input.strip()
print(clean_name.upper())
```
```
ANANYA ROY
```

---

## Strings Are Immutable

Once created, a string **cannot be changed in place**, every method that "changes" it actually returns a brand-new string.

```python
name = "python"
name[0] = "P"   # TypeError: strings can't be edited like this
```

The fix is always **reassignment**, replace the whole variable with a new string:

```python
name = "python"
name = "P" + name[1:]
print(name)
```
```
Python
```

This is the opposite of what you'll see with lists in Unit 3.1, which *can* be changed in place, that contrast is exactly why strings are worth understanding well now.

---

## Escape Characters

A backslash `\` inside a string means "the next character is special." You already used one above for escaping quotes, here are the other common ones:

| Escape | Means |
|---|---|
| `\n` | new line |
| `\t` | tab |
| `\"` | a literal quote mark, inside a double-quoted string |
| `\\` | a literal backslash |

```python
print("Line one\nLine two")
```
```
Line one
Line two
```

---

## f-strings, Clean Output

An f-string embeds a value directly in the text using `{}`:

```python
item = "Samosa"
price = 15.0
print(f"{item} costs Rs.{price:.2f} each")
```
```
Samosa costs Rs.15.00 each
```

**Forgetting the `f`** prints the literal text `{price}` instead of the value, a very easy typo to miss.

**Common format specifiers:**

| Specifier | Does what | Example | Result |
|---|---|---|---|
| `:.2f` | 2 decimal places (money) | `f"{15:.2f}"` | `15.00` |
| `:d` | plain integer | `f"{7:d}"` | `7` |
| `:>10` | right-align, 10 wide | `f"[{'Tea':>10}]"` | `[       Tea]` |
| `:^15` | center, 15 wide | `f"[{'Tea':^15}]"` | `[      Tea       ]` |

f-strings are the modern, recommended way to format output, cleaner than `.format()` or the older `%` style:

| Style | Example | Notes |
|---|---|---|
| f-string | `f"{name}"` | Cleanest, variable sits right where it's used |
| `.format()` | `"{}".format(name)` | Still works, seen in older code |
| `%` | `"%s" % name` | Oldest style, easy to mismatch, avoid in new code |

---

## Comments

```python
# this line is ignored by Python
```

Write comments that explain **why**, not what, the code already shows what it does. And keep spacing/style consistent (PEP 8) so teammates can read it easily.

---

## Try it Yourself

A canteen receipt, item, price (as text, like it would arrive from an order form), and quantity:

```python
item = "Samosa"
price_text = "15.0"
quantity = 3

price = float(price_text)
total = price * quantity

print(f"{item:>10}: {quantity:d} x Rs.{price:.2f} = Rs.{total:.2f}")
```
```
    Samosa: 3 x Rs.15.00 = Rs.45.00
```

**(a)** Add a second item, `"Coffee"`, priced at `"35.0"` for `2` cups. Print the same style of receipt line.

**(b)** Clean up this messy input and print it in uppercase:

```python
raw_name = "  ananya roy  "
```
```python
print(raw_name.strip().upper())
```
```
ANANYA ROY
```

**(c)** Split `"Samosa,Coffee,Tea"` into a list of items using `.split(",")`, then join them back together with a `-` instead.

**Your turn:** take the item name `"Samosa"`, print it reversed using `[::-1]`, then print just its first three letters using slicing.

---

## Common Mistakes

- Assuming text that "looks like a number" already behaves like one, it's `str` until you convert it
- Using `int()` on a decimal-looking string, `int("3.14")` raises `ValueError`, convert with `float()` first
- Expecting `int()` to round, it truncates toward zero instead
- Forgetting the `f` prefix on an f-string
- Mixing quote types on the same string, `'text"` doesn't work
- Forgetting that slicing's `stop` index is **excluded**, `word[0:3]` gives 3 characters, not 4
- Trying to edit a string directly (`name[0] = "P"`), always reassign instead
- Assuming a string method changes the original, it doesn't, you have to store the result

---

## Interview Questions

**Q1: What's the difference between implicit and explicit type conversion?**

A: Python does a small amount automatically, `3 + 4.0` becomes `7.0` because `int` promotes to `float`. But it never converts `str` to a number on its own, that always needs `int()`, `float()`, or similar, explicitly.

**Q2: Why does `int("3.14")` fail but `int(3.14)` work?**

A: `int()` on a string tries to parse whole-number digits only, a decimal point breaks that. `int()` on an actual float truncates it instead. Two different code paths inside the same function.

**Q3: Are strings mutable or immutable in Python?**

A: Immutable. Once created, a string can't be changed, every "changing" method returns a new string instead of editing the original.

**Q4: What does `word[2:5]` actually return?**

A: Characters at index 2, 3, and 4, the `stop` index (5) is never included.

**Q5: How do you reverse a string in Python?**

A: `word[::-1]`, an empty start and stop with a step of `-1` walks the string backward.

---

## Quick Recap

- A statement is one instruction; assignment stores, expressions run/produce a result.
- Convert text to numbers explicitly with `int()`/`float()` before doing math with them.
- `int()` truncates, never rounds; `int("3.14")` raises `ValueError`.
- Single and double quotes work the same, pick whichever avoids escaping; concatenate strings with `+`.
- Indexing gets one character; slicing (`start:stop:step`) gets a section, `stop` is always excluded.
- String methods (`.upper()`, `.strip()`, `.split()`, `.join()`, etc.) return a **new** string; strings are immutable.
- f-strings (`f"{value:.2f}"`) are the modern way to format output.
- Comments explain *why*, not *what*.


## Reference Links

- [Python 3 Documentation — Built-in Functions (`int`, `float`, `str`, `bool`)](https://docs.python.org/3/library/functions.html)
- [Python 3 Documentation — Formatted String Literals (f-strings)](https://docs.python.org/3/reference/lexical_analysis.html#f-strings)
- [PEP 498 — Literal String Interpolation](https://peps.python.org/pep-0498/)
- [PEP 634 — Structural Pattern Matching: Specification](https://peps.python.org/pep-0634/)
- [PEP 8 — Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [Real Python — Python's F-String for String Interpolation and Formatting](https://realpython.com/python-f-strings/)
- [W3Schools — Python Casting (Type Conversion)](https://www.w3schools.com/python/python_casting.asp)

[← Previous: 1.3 Operators & Expressions](unit-1-3-operators-expressions.md) | [Go back to TOC](../../README.md) | [Next up: **Unit 2.1 — Conditionals** — where your programs make their first real decisions with `if`, `elif`, and `else`. →](../p2-control-structures-functions-tooling/unit-2-1-conditionals.md)

---

*© 2026 Revature · AI Native Engineering — Foundations · Unit 1.4 · Version 2.0*
