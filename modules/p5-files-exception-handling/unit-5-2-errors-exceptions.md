# Errors & Exceptions

The last chapter ended mid-crash:

```python
import csv

with open("accounts.csv", newline="") as file:
    reader = csv.reader(file)
```

```text
FileNotFoundError: [Errno 2] No such file or directory: 'accounts.csv'
```

`accounts.csv` was never created in this run, and Python stopped the program dead, right on that line. Nothing after it ran. That's not acceptable for a real system — a script loading yesterday's roster, a service reading a config file at startup — so this chapter is about detecting a problem like this one, deciding what to do about it, and letting the rest of the program keep running anyway.

---

## Two Kinds of Broken

Before fixing that crash, it's worth being precise about what kind of failure it even is, because Python has two genuinely different ones.

```python
if True
    print("missing a colon")
```

```text
  File "<cell>", line 1
    if True
           ^
SyntaxError: expected ':'
```

Nothing ran here — not even `print`. A **syntax error** is a defect in the code's structure. Python reads the code, cannot make sense of its grammar, and refuses to start executing even the first line.

```python
with open("accounts.csv", newline="") as file:
    reader = csv.reader(file)
```

This code is grammatically perfect Python. It *started* running — `open()` was actually called — and only then, partway through, did something go wrong: the file genuinely wasn't there. A **runtime exception** (usually just called an **exception**) is valid code that hits a real problem while it's executing.

| | Syntax Error | Runtime Exception |
|---|---|---|
| When it happens | Before the program starts | While the program is running |
| Cause | Malformed code Python can't parse | Valid code that hits a real-world problem |
| Fixable by handling it in code? | No — the code itself must be edited | Yes — that's exactly what this chapter covers |
| Example | A missing colon | `FileNotFoundError` |

A syntax error is a bug fixed once, by editing the code. A runtime exception can happen even in code that is completely correct — a file that used to exist got deleted, a user typed something unexpected. The rest of this chapter is entirely about that second kind.

---

## Catching a Runtime Exception

```python
try:
    with open("accounts.csv", newline="") as file:
        reader = csv.reader(file)
        rows = list(reader)
except FileNotFoundError:
    print("accounts.csv not found — starting with an empty roster.")
    rows = []
```

```text
accounts.csv not found — starting with an empty roster.
```

Here's what Python actually does with this: everything inside `try` runs first, normally. If it finishes without incident, `except` never runs at all. But the moment `FileNotFoundError` is raised anywhere inside `try`, Python immediately abandons the rest of that block — `rows = list(reader)` never gets a chance to run — and jumps straight into the matching `except`.

```text
try block runs
     │
     ▼  (something goes wrong)
Python jumps to the matching except
     │
     ▼
except block runs
     │
     ▼
program continues normally
```

Compare that to what happened at the very top of this chapter, with no `try` at all: the exception propagated all the way up, out of every enclosing function, until nothing was left to catch it — and the entire program halted. `try`/`except` is what stops that propagation at a point you choose, on your own terms.

---

## Catching the Right Thing, Not Everything

`except FileNotFoundError` only catches that one specific exception. Suppose `accounts.csv` does exist, but only has three accounts in it, and the program tries to reach a fourth:

```python
try:
    with open("accounts.csv", newline="") as file:
        reader = csv.reader(file)
        rows = list(reader)
    fourth_row = rows[4]
except FileNotFoundError:
    print("accounts.csv not found.")
```

If `rows` only has three items, `rows[4]` raises `IndexError` — and this `except` clause, written only for `FileNotFoundError`, doesn't match it. The `IndexError` propagates straight past, uncaught, exactly as if no `try` existed at all.

Handling both means listing both:

```python
try:
    with open("accounts.csv", newline="") as file:
        reader = csv.reader(file)
        rows = list(reader)
    fourth_row = rows[4]
except FileNotFoundError:
    print("accounts.csv not found.")
    rows = []
except IndexError:
    print("accounts.csv doesn't have a fourth row.")
```

Python checks each `except` in order and runs the first one that matches. Two different problems, two different responses — and neither block masks the other, because each is looking for something distinct.

It's tempting to shortcut this with one `except` that catches *anything*:

```python
try:
    with open("accounts.csv", newline="") as file:
        reader = csv.reader(file)
        rows = list(reader)
    fourth_row = rows[4]
except:
    print("Something went wrong.")
    rows = []
```

This runs without crashing, but at a real cost: `except:` with no exception type catches *every* exception, including ones that have nothing to do with the file at all. A typo like `rows = []` two lines further down, which would normally surface as a loud, obvious `NameError`, instead gets silently swallowed by this same bare `except`, printing the same unhelpful "Something went wrong." Now a real bug in your own code looks identical to a missing fourth row, and there's no way to tell them apart from the output alone.

> Always name the specific exception you expect. A bare `except:` doesn't handle errors — it hides them, including ones you never meant to catch.

---

## Two Guarantees `try`/`except` Doesn't Give You

`try`/`except` answers "what if something goes wrong." Two more clauses answer questions it alone can't:

```python
try:
    with open("accounts.csv", newline="") as file:
        reader = csv.reader(file)
        rows = list(reader)
except FileNotFoundError:
    print("accounts.csv not found.")
    rows = []
else:
    print("Roster loaded successfully.")
finally:
    print("Done attempting to load the roster.")
```

- **`else`** runs only if `try` completed with *no* exception at all. Put code here that should run *after* a success, but that shouldn't be mistaken for part of the risky operation itself.
- **`finally`** runs *unconditionally* — success, a handled failure, or even an unhandled failure about to crash the program. It's the one block guaranteed to execute no matter what happened above it.

| Clause | Runs when... |
|---|---|
| `try` | Always attempted first |
| `except` | Only if a matching exception was raised inside `try` |
| `else` | Only if `try` raised nothing at all |
| `finally` | Always — regardless of what happened in any block above |

`finally` should feel familiar — it's the same unconditional guarantee `with` already gave you for closing files in the last chapter, generalized to *any* cleanup, not just closing a file.

---

## Exceptions You'll Meet Constantly

Six show up often enough to recognize on sight:

| Exception | Typical trigger |
|---|---|
| `ValueError` | A value has the right type but an invalid value — converting `"abc"` to a number |
| `TypeError` | An operation gets the wrong type entirely — adding a string and a number |
| `ZeroDivisionError` | Dividing by zero |
| `FileNotFoundError` | Opening a path that doesn't exist |
| `KeyError` | Looking up a dictionary key that isn't there — `row["email"]` when a CSV row has no `email` column |
| `IndexError` | Indexing a position that doesn't exist — `rows[4]` when only three rows were read |

Each of these is a class — `ValueError`, `KeyError`, and the rest all inherit from a common base, `Exception`, the same inheritance you already know from the OOP chapters. That's not incidental; it's what makes the rest of this chapter possible.

---

## Getting at the Exception Itself

`except FileNotFoundError:` catches the exception but discards it. Often the exception object itself carries a useful message worth keeping:

```python
try:
    with open("accounts.csv", newline="") as file:
        pass
except FileNotFoundError as e:
    print(f"Couldn't load the roster: {e}")
```

```text
Couldn't load the roster: [Errno 2] No such file or directory: 'accounts.csv'
```

`as e` binds the actual exception object to `e`, letting you log its message or inspect it, instead of writing your own generic text and throwing that information away.

Because `FileNotFoundError`, `IndexError`, and friends share `Exception` as a common ancestor, one `except` clause can catch several related exceptions at once by listing them in a tuple:

```python
try:
    with open("accounts.csv", newline="") as file:
        reader = csv.reader(file)
        rows = list(reader)
    fourth_row = rows[4]
except (FileNotFoundError, IndexError) as e:
    print(f"Couldn't load the fourth account: {e}")
```

---

## Raising Your Own Exceptions

Everything so far has reacted to exceptions Python raised on its own. Your own code can trigger the exact same mechanism, the moment it detects a problem no built-in exception was going to catch for you:

```python
def register_student(name, email, student_id):
    if not student_id.startswith("S"):
        raise ValueError(f"student_id must start with 'S', got {student_id!r}")
    return Student(name, email, student_id)


register_student("Vikram", "vikram@example.com", "104")
```

```text
ValueError: student_id must start with 'S', got '104'
```

`raise` stops execution immediately, right at that line — precisely like an exception Python triggered internally, because from Python's point of view, there's no difference. Whoever calls `register_student()` can wrap it in a `try` and handle `ValueError` exactly the way they'd handle one from `open()` or `int()`.

### When a Built-In Exception Doesn't Fit

`ValueError` is a reasonable fit for a malformed `student_id`. It's a poor fit for something like "this email is already registered" — that's not a value being wrong in the general sense; it's a rule specific to this platform. Python lets you define a new exception type for exactly that case, by inheriting from `Exception`:

```python
class DuplicateAccountError(Exception):
    pass


def add_account(accounts, new_account):
    for account in accounts:
        if account.email == new_account.email:
            raise DuplicateAccountError(f"{new_account.email} is already registered")
    accounts.append(new_account)
```

```python
accounts = [Student("Rahul", "rahul@example.com", "S101")]

try:
    add_account(accounts, Student("Rahul", "rahul@example.com", "S102"))
except DuplicateAccountError as e:
    print(f"Registration blocked: {e}")
```

```text
Registration blocked: rahul@example.com is already registered
```

`class DuplicateAccountError(Exception): pass` is doing real work despite its single line — by inheriting from `Exception`, it automatically gains everything a normal exception needs: it can be `raise`d, carry a message, and be caught by name in an `except` clause, all without writing any of that machinery by hand. This is the same inheritance you've used before, applied to Python's own exception hierarchy instead of a class you designed from scratch. A custom exception name also documents intent directly in the code — `except DuplicateAccountError` tells a reader exactly what went wrong, in a way `except ValueError` never could here.

---

## Try It Yourself

1. Wrap the code that opens `login_log.txt` from the last chapter in a `try`/`except FileNotFoundError`, and print a friendly message instead of letting the program crash if the file isn't there yet.
2. Write a function `get_account(accounts, email)` that searches a list of accounts for a matching email and returns it. Raise a custom `AccountNotFoundError` if no match exists, and catch it where you call the function.
3. The function below is supposed to skip a missing file gracefully, but it doesn't. Find and fix the bug:

   ```python
   def load_roster():
       try:
           with open("accounts.csv", newline="") as file:
               return list(csv.reader(file))
       except ValueError:
           print("accounts.csv not found.")
           return []
   ```

4. Add an `else` clause to exercise 1 that prints `"Log loaded."` only when the file opens successfully, and a `finally` clause that always prints `"Done checking the log."`.

---

## Key Takeaways

- A syntax error stops a program before it runs at all; a runtime exception happens mid-execution, in code that was otherwise valid — only the second kind can be handled with `try`/`except`.
- `except` only catches the exception type it names. A bare `except:` catches everything, including bugs that have nothing to do with the failure you were expecting, and should be avoided.
- `else` runs only when `try` raised nothing; `finally` runs no matter what — the same unconditional guarantee `with` already relies on for closing files.
- `raise` lets your own code trigger an exception the moment it detects a problem, using a built-in exception type or a custom one you define by inheriting from `Exception`.

---

## What's Still Missing

Every fix in this chapter has handled one problem at a time, in isolation. A real file, like `accounts.csv`, can have several bad rows in it at once — a missing email here, an unrecognized role there — and a real program has to work through the whole file, keeping what's valid and setting aside what isn't, without losing track of why. That's the subject of the next chapter.
