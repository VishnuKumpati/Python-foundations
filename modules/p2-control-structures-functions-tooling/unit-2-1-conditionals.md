## 2.1 Conditionals

---

[← Previous: 1.4 Statements, Conversion & Output](../p1-introduction/unit-1-4-statements-conversion-output.md) | [Go back to TOC](../../README.md) | [Next: 2.2 Loops →](unit-2-2-loops.md)


---

## Why Conditionals?

Every program you've written so far runs the exact same lines, every single time. Real programs need to make choices, show a discount only if the cart total is high enough, block a login if the password is wrong, print a different message depending on the weather.

A **conditional** lets your code run different lines depending on whether something is `True` or `False`.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Conditional** | Code that runs only if a condition is true |
| **Branch** | One possible path your code can take |
| **Indentation** | The spaces at the start of a line, Python uses this to know what's "inside" a block |
| **Nested conditional** | A conditional written inside another conditional |
| **Ternary expression** | A one-line if/else that produces a value |

---

## `if`, `elif`, `else`

```python
age = 20

if age >= 18:
    print("You can vote.")
```
```
You can vote.
```

The colon `:` and the indented line below it are both required, that indented block is what actually runs when the condition is `True`.

**Add an alternative with `else`:**

```python
age = 15

if age >= 18:
    print("You can vote.")
else:
    print("Not old enough yet.")
```
```
Not old enough yet.
```

**More than two options? Use `elif`** (short for "else if"):

```python
marks = 72

if marks >= 90:
    print("Grade: A")
elif marks >= 75:
    print("Grade: B")
elif marks >= 60:
    print("Grade: C")
else:
    print("Grade: F")
```
```
Grade: C
```

Python checks each condition top to bottom, and runs the **first** one that's `True`, then skips the rest entirely, even if a later condition would also be `True`.

---

## Indentation is Structure, Not Style

In most languages, indentation is just for readability. In Python, it's the actual syntax, it tells Python what belongs inside the `if` block and what doesn't.

```python
if age >= 18:
    print("You can vote.")
    print("Welcome!")
print("This always runs.")
```

The first two `print()` lines are indented, so they only run when the condition is `True`. The third line isn't indented, so it runs regardless, it isn't part of the `if` block at all.

**Mismatched indentation breaks your code:**

```python
if age >= 18:
print("You can vote.")   # IndentationError: expected an indented block
```

---

## Nested Conditionals

An `if` can sit inside another `if`, for decisions that depend on more than one thing:

```python
has_ticket = True
age = 16

if has_ticket:
    if age >= 18:
        print("Entry allowed.")
    else:
        print("Entry allowed with guardian.")
else:
    print("No ticket, no entry.")
```
```
Entry allowed with guardian.
```

**Nesting gets hard to read fast.** Two levels is usually fine, three or more, and it's often worth combining conditions with `and` instead (see below). This is a real trade-off: nesting is easy to write in the moment, but harder for someone else (or you, later) to follow.

---

## Boolean Expressions in Conditions

You already met `and`, `or`, and `not` in Unit 1.3, conditionals are where they really earn their keep, combining multiple checks into one:

```python
age = 20
has_id = True

if age >= 18 and has_id:
    print("Entry allowed.")
else:
    print("Entry denied.")
```
```
Entry allowed.
```

```python
is_weekend = True
is_holiday = False

if is_weekend or is_holiday:
    print("No classes today.")
```
```
No classes today.
```

```python
is_banned = False

if not is_banned:
    print("Access granted.")
```
```
Access granted.
```

Remember short-circuit evaluation from 1.3: in `age >= 18 and has_id`, if `age >= 18` is already `False`, Python never even checks `has_id`.

---

## Conditional (Ternary) Expression

For simple either/or decisions, you don't always need a full `if`/`else` block, a one-liner works:

```python
value_if_true if condition else value_if_false
```

```python
age = 20
status = "Adult" if age >= 18 else "Minor"
print(status)
```
```
Adult
```

This is an **expression**, it produces a value you can store or print directly, unlike a regular `if` statement. Use it for short, simple choices; for anything with more than one condition or multiple actions, a normal `if`/`elif`/`else` block reads better.

---

## Try it Yourself

A canteen entry check: entry requires being a student **or** having a valid guest pass, **and** not being on the blocklist.

```python
is_student = True
has_guest_pass = False
is_blocked = False

if (is_student or has_guest_pass) and not is_blocked:
    print("Entry allowed.")
else:
    print("Entry denied.")
```
```
Entry allowed.
```

**(a)** Change `is_blocked` to `True` and predict the output before running it.

**(b)** Write a grading program using `if`/`elif`/`else` for marks `85`, printing `"A"` for 90+, `"B"` for 75+, `"C"` for 60+, and `"F"` otherwise.

**Your turn:** rewrite this using a ternary expression instead of `if`/`else`:

```python
temperature = 40
if temperature > 35:
    weather = "Hot"
else:
    weather = "Pleasant"
print(weather)
```

---

## Common Mistakes

- Forgetting the colon `:` at the end of `if`, `elif`, or `else`
- Inconsistent indentation, mixing tabs and spaces, or misaligning lines within the same block
- Using `=` instead of `==` inside a condition (from Unit 1.3, still the most common bug)
- Writing `elif` after `else`, `else` must always come last
- Over-nesting `if` statements when combining conditions with `and`/`or` would be clearer
- Forgetting a ternary expression is a value, not a statement, `if age >= 18: status = "Adult"` alone doesn't work as a one-liner without `else`

---

## Interview Questions

**Q1: What's the difference between `if`/`elif` and a series of separate `if` statements?**

A: `elif` only runs if all the earlier conditions were `False`, and stops at the first match. Separate `if` statements each get checked independently, so more than one could run. This matters when conditions overlap.

**Q2: Why does Python use indentation instead of braces `{}`?**

A: It's a deliberate design choice, forcing consistent formatting so code structure is always visually obvious, instead of relying on a separate style guide.

**Q3: When would you use a ternary expression instead of a full `if`/`else`?**

A: For short, simple value assignments with exactly two outcomes. Anything more complex (multiple conditions, multiple actions) reads better as a regular `if`/`elif`/`else` block.

---

## Quick Recap

- `if`/`elif`/`else` runs different code depending on a condition; Python stops at the first `True` branch.
- Indentation isn't optional, it defines what's inside a block.
- Nested conditionals handle multiple layered decisions, but get hard to read past two levels.
- `and`, `or`, `not` combine conditions inside an `if`, with short-circuit evaluation still applying.
- A ternary expression (`x if condition else y`) is a compact one-line if/else that produces a value.


##  Reference Links

- [The Python Tutorial — More Control Flow Tools (if Statements)](https://docs.python.org/3/tutorial/controlflow.html#if-statements)
- [Python 3 Language Reference — Compound Statements (if)](https://docs.python.org/3/reference/compound_stmts.html#the-if-statement)
- [Python 3 Documentation — Conditional Expressions](https://docs.python.org/3/reference/expressions.html#conditional-expressions)
- [Real Python — Conditional Statements in Python](https://realpython.com/python-conditional-statements/)
- [W3Schools — Python If...Else](https://www.w3schools.com/python/python_conditions.asp)

[← Previous: 1.4 Statements, Conversion & Output](../p1-introduction/unit-1-4-statements-conversion-output.md) | [Go back to TOC](../../README.md) | [Next: 2.2 Loops →](unit-2-2-loops.md)

---

*© 2026 Revature · AI Native Engineering — Foundations · Unit 2.1 · Version 2.0*
