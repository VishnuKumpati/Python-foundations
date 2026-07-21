---
topic_id: 44580fc5-261e-4a0a-bc9b-22972428f96e
title: Unit 1.2 - Variables, Identifiers & Types
position_in_module: 2
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 1.2 - Variables, Identifiers & Types — Topic Corpus

## 2. Prerequisites

This topic builds directly on **9ac73bbb-9280-4975-a311-3079a78c38c4 — "Unit 1.1 - The Python Environment."** That unit covered how to type code into a Google Colab notebook cell, run it in the runtime, and use the `print()` function to display a result, including printing a string (text data). This corpus assumes you can already open a Colab notebook, type code into a cell, run it, and read printed output — it does not re-teach any of that.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Create** a variable by using the assignment operator (`=`) to bind a name to a value, and **update** it by reassigning a new value to the same name.
- **Apply** Python's identifier rules and the `snake_case` naming convention to write variable names that are both legal and readable.
- **Identify** which of the four basic types — `int`, `float`, `str`, or `bool` — a given value belongs to, based on how it is written.
- **Use** the built-in `type()` function to check the type of any value stored in a variable.
- **Explain**, in plain language, why Python is described as a dynamically typed language, and how that differs from a statically typed language.
- **Recognize** the two most common beginner errors in this area — `NameError` and `SyntaxError` — and state what causes each one.

## 4. Introduction

Imagine a food delivery app. You add a dish to your cart, the app remembers the price. You type your delivery address, the app remembers it until the order is placed. You confirm payment, and the app remembers that the order is now paid for. None of that "remembering" is magic — every one of those facts is being held in the computer's memory under a name the program chose, so it can be looked up and changed again later.

In Unit 1.1, you learned to run a line of code in a Colab notebook cell and use `print()` to show something on the screen once. But a value that only gets printed and then vanishes is not very useful — a real program needs to hang on to values, use them more than once, and change them as things happen. That is the exact gap this topic fills.

In this topic you will learn the single idea that almost every program you will ever write depends on: giving a value a name so you can refer to it again. Along the way, you will learn the rules for choosing legal names, the four basic kinds of values Python works with, a built-in tool for checking what kind of value you are holding, and why Python never makes you announce that kind in advance.

## 5. Core Concepts

### 5.1 Variables and the assignment operator

A **variable** is a name that refers to a value stored in the computer's memory. A useful mental picture is a labelled box: the label is the name you pick, and the box holds whatever value you put inside it. You create a variable with the **assignment operator**, the single equals sign `=`, written with the name on the left and the value on the right [1]:

```python
item_price = 149
print(item_price)
```

Running this prints `149`. The first line does not compare two things the way `=` does in a math class. It performs an action: "let the name `item_price` refer to the value `149`." Read `x = 5` as "let `x` refer to `5`," never as "`x` equals `5`" — that habit will save you confusion later when you meet a different, comparison-only symbol in the next unit (only named here, not taught yet).

Once a variable exists, you can use its name anywhere you would use the value itself. Typing `print(item_price)` looks up whatever `item_price` currently refers to and displays it — you never have to retype `149`.

### 5.2 Reassignment

**Reassignment** means giving a variable that already exists a new value, which completely replaces the old one [1]:

```python
item_price = 149
print(item_price)      # 149

item_price = 249
print(item_price)      # 249
```

The second `item_price = 249` does not create a second, separate variable — it is the same name, now pointing at a different value. The old value, `149`, is gone the instant the reassignment happens; there is no way to get it back unless you had saved it under another name first. This matters because it is exactly how a running program keeps up with the real world: a bank balance changes after a withdrawal, a game score changes after a level, an order's "paid" status changes the moment payment succeeds — all through reassignment of the same name.

### 5.3 Identifiers and naming rules

The technical term for the name you give a variable is an **identifier** [1]. Python enforces a small, strict set of rules about what an identifier is allowed to contain. Breaking one of these rules stops your code from running at all, before a single line executes:

- An identifier may contain letters, digits, and the underscore character `_`.
- It must **not start with a digit** — `age2` is a legal name, but `2age` is not.
- It may not contain spaces or symbols such as `-`, `!`, or `$`.
- It must not be a **reserved keyword** — a word Python has already claimed for its own grammar, such as `if`, `for`, or `class`. You cannot name a variable `class`, but `class_name` works fine.

Violating any of these produces a **`SyntaxError`**, the error Python raises when code breaks the language's basic grammar rules [1]. A `SyntaxError` on a variable name is Python telling you it cannot even parse what you typed as a valid instruction — this is different from an error that shows up while the code is running, which you will meet next.

### 5.4 Case sensitivity

Python is **case sensitive**: it treats uppercase and lowercase letters as different characters, so `total`, `Total`, and `TOTAL` are three completely separate, unrelated variable names [1]. This is not a Python quirk to memorize and forget — it is one of the most common sources of a real bug. If you create `total_price` in one line and later, by accident, type `Total_price`, Python does not "know what you meant." It sees a name that was never assigned.

That leads to the second common error in this topic: **`NameError`**, the error Python raises when you try to use a variable that was never assigned [1]. Unlike a `SyntaxError`, which stops the program before it starts, a `NameError` happens while the program is running, at the exact line where the unknown name is used. A variable must be assigned at least once, earlier in the program, before you can refer to it — using it any sooner (or misspelling its case) raises a `NameError`.

### 5.5 The `snake_case` naming convention

Python's identifier rules tell you what is *legal*. They say nothing about what is *readable*. That is where convention comes in. The Python community's official style guide, called **PEP 8**, recommends **`snake_case`** for variable names: all lowercase words, separated by underscores, such as `total_price` or `customer_name` [1]. Unlike the identifier rules in section 5.3, `snake_case` is not enforced by Python itself — code using `TotalPrice` or `totalprice` will still run. It is a shared agreement among Python programmers that keeps code consistent and easy for other people (and your future self) to read. A good identifier also describes the value's *purpose* rather than its type or a placeholder — `customer_age` communicates far more than `x`.

### 5.6 The four basic types

Every value in Python belongs to a **type** — a category describing what kind of value it is: a number, some text, or a true/false flag [1]. This unit introduces the four most basic types, and in every case Python figures out the type automatically from how the value is written — you never state it yourself:

| Type | Meaning | How it is written | Example |
|---|---|---|---|
| **`int`** | A whole number, with no decimal point. | Digits only, no dot. | `149`, `2`, `0` |
| **`float`** | A number written with a decimal point. | Digits with a `.` in them. | `29.50`, `745.5` |
| **`str`** | Text data (a **string**, already introduced in Unit 1.1). | Wrapped in quotation marks. | `"Ananya Roy"`, `"SWG10234"` |
| **`bool`** | A truth value — exactly one of two possibilities. | The literal word `True` or `False`, no quotes. | `True`, `False` |

A subtlety worth catching early: something that *looks* numeric is not automatically an `int` or `float`. `order_id = "SWG10234"` stores a `str`, not a number, because it is wrapped in quotes — even though it contains digits, Python treats it purely as text to display, not a value to calculate with. Likewise, `"5"` (quoted, a string) and `5` (unquoted, an integer) look similar on the page but are entirely different types [1].

### 5.7 The `type()` function

Rather than guessing a value's type from how it looks, Python gives you a direct way to check: the built-in **`type()`** function, which reports the type of any value you hand it [1].

```python
delivery_fee = 29.50
print(type(delivery_fee))
```

This prints `<class 'float'>` — Python's way of stating "this value belongs to the `float` type." You can call `type()` on any variable, at any point in a program, to confirm exactly what you are working with instead of assuming.

### 5.8 Dynamic typing

Notice that none of the examples above ever told Python in advance, "this variable will hold a whole number" or "this variable will hold text." Python worked that out by itself, the instant each assignment ran. This behavior is called **dynamic typing**: Python determines a value's type automatically at assignment time, rather than requiring you to declare it up front [1].

This stands in contrast to a **statically typed language** (languages such as Java or C, named here only for contrast — their syntax is out of scope for this topic), where you must state a variable's type before using it, and that type is fixed forever, checked before the program even runs. In Python, by comparison, the type belongs to the *value*, not to the *name* — so the very same variable name can refer to an `int` at one point in a program and a `str` at another:

```python
data = 100
print(type(data))          # <class 'int'>

data = "one hundred"
print(type(data))          # <class 'str'>
```

Nothing about this is an error. It is simply Python re-inferring the type each time a new value is bound to `data`. A frequently asked beginner-level interview question is exactly this: "Is Python statically or dynamically typed?" The confident, correct answer is: dynamically typed — Python infers a value's type automatically from the assignment, at the moment the program runs, and that type can change if the same name is reassigned to a different kind of value later [1].

### 5.9 Getting a value from the user: `input()`

Every example so far has assigned a value that was typed directly into the code. Real programs usually need to ask the *person running the program* for a value instead — a name, an address, an answer to a question. The built-in **`input()`** function does exactly this: it pauses the program, optionally displays a prompt message, waits for the user to type something and press Enter, and hands back whatever they typed [1]:

```python
name = input("Enter your name: ")
print("Hello,", name)
```

*Line-by-line explanation:*
- `input("Enter your name: ")` displays the text `Enter your name: ` and then pauses — the program does nothing further until the user types a response and presses Enter.
- Whatever the user typed is handed back by `input()`, and `name = ...` assigns that value to `name`, exactly like any other assignment from section 5.1.
- `print("Hello,", name)` displays a greeting using whatever was entered.
- Sample run (user types `Ada` and presses Enter):
  ```
  Enter your name: Ada
  Hello, Ada
  ```

There is one fact about `input()` that catches almost every beginner at least once: **it always hands back a `str`**, no matter what the user types — even if they type digits that look like a number [1]. You can prove this with `type()`:

```python
age = input("Enter your age: ")
print(type(age))
```

Sample run (user types `20` and presses Enter):

```
Enter your age: 20
<class 'str'>
```

Even though `20` looks exactly like a number, `type()` confirms it is a `str`. This is `input()`'s behavior, not a mistake — it always returns text, because it has no way of knowing in advance whether the user meant to type a number, a name, or anything else. Turning that text into an actual `int` or `float` requires an explicit conversion step, which is covered later (Unit 1.4) and is not taught here — for now, the important habit is to check with `type()` rather than assume, whenever a value came from `input()`.

## 6. Implementation

Creating and checking a variable always follows the same small sequence of steps, whether you are storing a price, a name, or a flag:

1. **Choose an identifier** that describes the value's purpose and follows the naming rules from section 5.3 (no leading digit, no reserved keyword, letters/digits/underscore only).
2. **Write the assignment**: identifier, then `=`, then the value — Python evaluates the right-hand side completely first, then binds the name to the result [1].
3. **Use the name** anywhere you need that value again — in a `print()` call, in a later calculation, or as part of another assignment.
4. **Reassign when the value needs to change**, using the exact same syntax (`name = new_value`) — the old value is simply replaced.
5. **Check the type whenever you are unsure**, by wrapping the name in `type(...)` and printing the result, rather than guessing from appearance.

Applied to a small food-delivery-style example, all five steps look like this:

```python
customer_name = "Ananya Roy"     # step 1 & 2: str
delivery_fee = 29.50             # step 1 & 2: float
is_paid = False                  # step 1 & 2: bool

print(type(customer_name))       # step 5
print(type(delivery_fee))        # step 5
print(type(is_paid))             # step 5

is_paid = True                   # step 4: reassignment
print("Payment status:", is_paid)
```

Output:

```
<class 'str'>
<class 'float'>
<class 'bool'>
Payment status: True
```

Each variable was created once, its type was confirmed with `type()`, and `is_paid` was later reassigned to reflect a change — the same pattern scales up to any program you will write in this course [1].

## 7. Real-World Patterns

The moment you start naming and storing values in a Colab cell, you are doing a smaller-scale version of what production software does every day [1]:

- **Banking and payment apps** store an account holder's name (`str`), a balance (`float`), and whether the account is active (`bool`) — the same three types from this topic.
- **E-commerce carts** hold an item's price (`float`), a quantity (`int`), a product name (`str`), and whether a coupon was applied (`bool`).
- **Booking systems** (railway, flight, movie tickets) store a passenger's name (`str`), a fare (`float`), a seat count (`int`), and a confirmed/not-confirmed flag (`bool`) — and update that flag the moment payment succeeds, exactly like the `is_paid` reassignment above.
- **Healthcare records** store a patient's name (`str`), age (`int`), a temperature reading (`float`), and an admitted/not-admitted flag (`bool`).

Across every one of these systems, the pattern never changes: name a value, let Python infer its type, and update it by reassignment as events happen. This is why "variables and types" is universally one of the first topics in any programming course — nearly everything built afterward assumes you already have this down.

## 8. Best Practices

- Use `snake_case` for every variable name (`total_price`, not `TotalPrice` or `totalprice`), following PEP 8 [1].
- Name a variable after what it *represents*, not its type or a placeholder letter — `customer_age`, not `x` or `intvalue`.
- Pick one spelling and one letter-case for a variable and use it consistently for the rest of the program; never mix `total` and `Total` for the same value.
- When you are not sure what type a value has, call `type()` on it rather than guessing from how it is written — quotes, digits, and decimal points can be easy to misread at a glance.
- Assign a variable close to where you first use it, so a reader does not have to search far up the program to see where it came from.

## 9. Hands-On Exercise

Using a single Colab cell, model a small movie-ticket booking: create `movie_name` (a `str`), `ticket_price` (a `float`), `seats_booked` (an `int`), and `booking_confirmed` starting at `False` (a `bool`). Print the type of all four variables using `type()`. Then reassign `booking_confirmed` to `True` and print a message showing the updated value, confirming that the same name now refers to a different value than it did at the start. As a final step, use `input()` to ask for the customer's name into a variable called `customer_name`, print `type(customer_name)`, and confirm for yourself that it reports `<class 'str'>` even if you type only digits when prompted.

## 10. Key Takeaways

- A **variable** is a name bound to a value via the assignment operator `=`; **reassignment** replaces the old value under the same name with a new one.
- **Identifiers** must follow Python's rules — letters, digits, and underscores only, no leading digit, no reserved keywords — while `snake_case` is a readability convention (PEP 8), not a rule Python enforces.
- Python is **case sensitive**, so `total` and `Total` are two unrelated variables; using a name that was never assigned raises a `NameError`, and an illegal identifier raises a `SyntaxError` before the program even runs.
- The four basic types covered here are **`int`** (whole numbers), **`float`** (decimal numbers), **`str`** (quoted text), and **`bool`** (`True`/`False`); the built-in **`type()`** function reveals a value's type at any point in a program.
- Python uses **dynamic typing**: a value's type is inferred automatically at assignment time and belongs to the value, not the name, so the same variable can hold different types at different points in a program.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
