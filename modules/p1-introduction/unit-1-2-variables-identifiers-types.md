# Variables, Identifiers & Types

In Unit 1.1 you opened Colab, typed `print("Namaste, world")` into a cell, ran it, and watched the interpreter answer back immediately. That was real progress — but notice what happens the moment the cell finishes: the value you printed is gone. Run the cell again and Python does the exact same work from scratch, remembering nothing about what happened last time. A banking app that forgot your balance the instant it displayed it, or a UPI app that forgot the amount you were about to pay, would be useless. Somewhere, a real program has to hold on to a value, not just display it once and move on.

That is exactly the gap this chapter closes. You will learn how to give a value a name so your program can refer to it again — a **variable** — the rules Python enforces on what you're allowed to call that name, and the four basic kinds of value (`int`, `float`, `str`, `bool`) every Python program leans on constantly. You will meet a built-in tool, `type()`, that tells you exactly what kind of value you're holding instead of making you guess, and you will see why Python never asks you to announce that kind in advance — a behaviour called dynamic typing. Along the way you will collect your first value directly from a person using your program, with the built-in `input()` function.

By the time you finish this chapter, "give it a name, use the name" — the single most-used idea in all of software development — will be something you have typed, run, and reasoned about yourself, not just read about.

---

## The gap `print()` alone can't close

Think about what a handful of everyday apps actually have to remember, not just display once:

| App | What it has to remember |
|---|---|
| UPI payment app | The amount you're about to pay, right up until you confirm it |
| Food delivery app (Swiggy-style) | Every item's price as you keep adding to your cart |
| Banking app | Your account balance, updated after each transaction |
| Railway booking (IRCTC-style) | Whether your seat is confirmed, from the moment you pay to the moment you board |

In every row, a value is created once and then used again later, often after something else has happened in between. `print()` cannot do that on its own — it shows you a value and then lets it go. What every one of these apps needs is a way to store a value under a name it can come back to, check, and change. Pick one row from the table and say out loud, in one sentence, what would break if the app simply couldn't remember that value between steps — that's the exact problem a variable exists to solve.

## Giving a value a name: variables and assignment

A **variable** is a name that refers to a value stored in the computer's memory. The simplest mental picture is a labelled box: the label is the name you choose, the box holds whatever value you put inside it. You create one with the **assignment operator** — a single equals sign, `=` — name on the left, value on the right:

```python
item_price = 149
print(item_price)
```

```
149
```

Read `item_price = 149` as "let `item_price` refer to `149`" — never as "`item_price` equals `149`" in the mathematical sense. In maths, `=` states a fact that is either true or false. In Python, `=` performs an action: it binds the name on the left to the value on the right, and nothing is being compared. Once that binding exists, `print(item_price)` doesn't magically know the number 149 — it looks up whatever `item_price` currently refers to and displays that.

**`=` in Python is an instruction, not a claim — you'll meet the symbol that actually checks equality (`==`) in the next unit, and mixing the two up is one of the most common early bugs.**

## Reassignment: the same name, a new value

Real values change. A customer edits their order, a bank balance drops after a withdrawal, a booking flips from unconfirmed to confirmed. **Reassignment** is giving a variable that already exists a new value, under the exact same name:

```python
item_price = 149
print(item_price)

item_price = 249
print(item_price)
```

```
149
249
```

Before checking, predict for yourself: after these four lines run, does Python still remember `149` anywhere? It does not. The second `item_price = 249` is not a second variable — it's the same name, now pointing at a different value, and the old value `149` is simply gone unless you had saved it under some other name first. This is exactly how a running program keeps up with the real world: an account balance, a game score, a booking's confirmed status all change through reassignment of one name, not by creating a fresh variable every time something happens.

## The rules for choosing a name

The technical term for a variable's name is an **identifier**, and Python enforces a small, strict set of rules on it — break one, and your code fails before a single line runs:

- May contain letters, digits, and the underscore `_`.
- Must **not start with a digit** — `age2` is legal, `2age` is not.
- May not contain spaces or symbols such as `-`, `!`, or `$`.
- Must not be a **reserved keyword** — a word Python has already claimed for its own grammar, such as `if`, `for`, or `class`.

Try it yourself before reading on: would `customer-name`, `2nd_item`, or `class` be legal Python identifiers? All three break one of the four rules above, and Python will refuse to run any of them, raising a **`SyntaxError`** — the same category of error you'd get from a missing parenthesis in Unit 1.1, just triggered by an illegal name instead.

Identifier rules tell you what's *legal*. They say nothing about what's *readable* — that's where convention, not enforcement, takes over.

| | Identifier rules (§ above) | `snake_case` convention |
|---|---|---|
| Who enforces it | Python itself | Nobody — it's a team agreement |
| Break it and... | Code fails with a `SyntaxError` | Code still runs fine |
| Source | Python's grammar | PEP 8, Python's official style guide |
| Example | `2age` is illegal | `TotalPrice` is legal but discouraged |

PEP 8 recommends **`snake_case`**: all lowercase words, separated by underscores — `total_price`, not `TotalPrice` or `totalprice`. A good name also describes the value's *purpose*, not its type or a placeholder letter: `customer_age` tells the next reader far more than `x` ever could.

One more rule catches almost everyone at least once: Python is **case sensitive**, so `total`, `Total`, and `TOTAL` are three completely separate, unrelated names.

**Typing `total_price` in one line and `Total_price` in another doesn't create a typo Python forgives — it creates two unrelated variables, and using the one that was never assigned raises a `NameError`.**

Unlike a `SyntaxError`, a `NameError` doesn't stop your program before it starts — it happens mid-run, at the exact line where the unknown name gets used, because a variable must be assigned at least once before you're allowed to refer to it.

## The four basic types of value

Every value in Python belongs to a **type** — a category describing what kind of value it is. This unit introduces the four you'll use constantly, and in every case Python works out the type automatically from how you write the value:

| Type | Meaning | How it's written | Example |
|---|---|---|---|
| `int` | A whole number, no decimal point | Digits only | `149`, `2`, `0` |
| `float` | A number with a decimal point | Digits with a `.` | `29.50`, `745.5` |
| `str` | Text data (a **string**) | Wrapped in quotes | `"Ananya Roy"`, `"SWG10234"` |
| `bool` | A truth value | The literal word `True` or `False`, no quotes | `True`, `False` |

Before reading on, guess: is `order_id = "SWG10234"` a `str` or an `int`? It's a `str` — the quotes are the only thing that matters here, not the digits inside them. Python treats it purely as text to display, never as a value you could calculate with. The same trap appears in miniature with `"5"` versus `5`: they look almost identical on the page but are entirely different types, one text, one number.

## Checking a type instead of guessing: `type()`

Rather than eyeballing quotes and decimal points, Python gives you a direct way to check: the built-in **`type()`** function, which reports the type of whatever value you hand it.

```python
delivery_fee = 29.50
print(type(delivery_fee))
```

```
<class 'float'>
```

That output is Python's way of stating "this value belongs to the `float` type." Call `type()` on any variable, at any point in a program, whenever you're not fully sure what you're holding — it costs one line and removes all the guesswork.

## Same name, different type: dynamic typing

Notice that nothing in this unit so far has told Python in advance, "this variable will hold a whole number" or "this variable will hold text." Python worked that out itself, the instant each assignment ran. That behaviour is called **dynamic typing**.

| | Statically typed (e.g. Java, C) | Dynamically typed (Python) |
|---|---|---|
| Type declaration | Declared up front — `int age = 30;` | Never declared — Python infers it |
| When the type is checked | Before the program runs | While the program runs |
| Can a variable change type? | No — fixed forever | Yes — same name, different type later |

The type belongs to the *value*, not to the *name* — so the exact same variable can refer to an `int` at one point and a `str` later, with nothing wrong happening at all:

```python
data = 100
print(type(data))          # <class 'int'>

data = "one hundred"
print(type(data))          # <class 'str'>
```

A frequently asked entry-level interview question is exactly this: "Is Python statically or dynamically typed?" Try answering it out loud, in one sentence, before moving on — if you can explain it in your own words, you've understood the idea rather than memorised the label.

## Asking the user for a value: `input()`

Every example so far has assigned a value typed directly into the code. A real program usually needs to ask the *person running it* for a value instead — a name, an address, an answer. The built-in **`input()`** function does exactly that: it pauses the program, optionally shows a prompt, waits for the user to type something and press Enter, then hands back whatever they typed.

```python
name = input("Enter your name: ")
print("Hello,", name)
```

Sample run (user types `Ada` and presses Enter):

```
Enter your name: Ada
Hello, Ada
```

Here's the detail that catches almost every beginner once: `input()` **always** hands back a `str`, no matter what the user types — even digits that look exactly like a number. Predict the output of this before you check it:

```python
age = input("Enter your age: ")
print(type(age))
```

Sample run (user types `20` and presses Enter):

```
Enter your age: 20
<class 'str'>
```

Even though `20` looks exactly like a number, `type()` confirms it's a `str`. Try `age + 5` right now and Python raises a `TypeError`, because you cannot add a number to text — you would first need to convert `age` with `int(age)`, a step covered later when we look at statements and conversion. For now, the habit that matters is checking with `type()` rather than assuming, any time a value came from `input()`.

## Try it yourself

Do this in a Colab cell before continuing. Model a small movie-ticket booking: create `movie_name` (a `str`), `ticket_price` (a `float`), `seats_booked` (an `int`), and `booking_confirmed` starting at `False`. Print the type of all four. Then reassign `booking_confirmed` to `True` and print a message showing the new value. Finally, use `input()` to collect a `customer_name`, and print its `type()` to confirm it reports `<class 'str'>` — even if you type nothing but digits when prompted.

---

### Key Terminology

- **Variable** — a name that refers to a value stored in memory.
- **Assignment operator (`=`)** — binds a name to a value; an action, not a mathematical equality.
- **Reassignment** — giving an existing variable a new value, replacing the old one.
- **Identifier** — the technical term for a variable's (or function's) name.
- **Reserved keyword** — a word Python has already claimed for its own grammar (`if`, `for`, `class`, etc.) — illegal as a variable name.
- **`snake_case`** — the PEP 8 naming convention: lowercase words separated by underscores.
- **Case sensitivity** — Python treats uppercase and lowercase letters as different, so `total` and `Total` are unrelated names.
- **Type** — a category describing what kind of value something is.
- **`int` / `float` / `str` / `bool`** — whole number, decimal number, quoted text, and `True`/`False` respectively.
- **`type()`** — the built-in function that reports a value's type.
- **Dynamic typing** — Python inferring a value's type automatically at assignment time, rather than requiring it declared in advance.
- **`input()`** — the built-in function that pauses a program, optionally shows a prompt, and returns whatever the user typed, always as a `str`.
- **`NameError`** — raised when code refers to a variable that was never assigned.
- **`SyntaxError`** — raised when code breaks Python's basic grammar rules, such as an illegal identifier.

### Mastery Checkpoint

Before moving to Unit 1.3, check that you can answer these without looking back:

1. What's the difference between how `=` behaves in Python and how it behaves in a maths class?
2. Why is `order_id = "SWG10234"` a `str` rather than a number, even though it's made entirely of digits?
3. `total_price` was assigned earlier in your program. You then write `print(Total_price)`. What happens, and why?
4. What is the one fact about `input()` that's true no matter what the user types in — and what problem does it cause if you forget it?
5. Is Python statically or dynamically typed, and what does that mean for a variable that's assigned an `int` and later reassigned a `str`?

### Summary

You now know how to give a value a name that survives past a single `print()`, the rules Python enforces on that name versus the `snake_case` convention that keeps it readable, and the four basic types — `int`, `float`, `str`, `bool` — that cover almost everything you'll store in this course. You've used `type()` to check a value instead of guessing, seen why Python's dynamic typing lets the same name hold different kinds of value over time, and collected your first value directly from a user with `input()` — along with the one gotcha, "it's always a `str`," that trips up nearly everyone the first time. From here, the next step is learning how to combine and compare the values you now know how to store — starting with operators and expressions.

### Additional Resources

- [Python Tutorial — official docs: "An Informal Introduction to Python" (numbers, strings, first steps)](https://docs.python.org/3/tutorial/introduction.html)
- [Python 3 Documentation — Built-in Types](https://docs.python.org/3/library/stdtypes.html)
- [Python 3 Documentation — `input()` built-in function](https://docs.python.org/3/library/functions.html#input)
- [PEP 8 — Style Guide for Python Code (Naming Conventions)](https://peps.python.org/pep-0008/#naming-conventions)
- [W3Schools — Python Variables](https://www.w3schools.com/python/python_variables.asp)
- [W3Schools — Python Data Types](https://www.w3schools.com/python/python_datatypes.asp)
- [W3Schools — Python User Input](https://www.w3schools.com/python/python_user_input.asp)
