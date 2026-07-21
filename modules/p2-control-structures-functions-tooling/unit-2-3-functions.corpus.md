---
topic_id: ed4fbf1a-b7e5-4df2-b368-1ecd21e1c99c
title: Unit 2.3 - Functions
position_in_module: 3
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 2.3 - Functions — Topic Corpus

## 2. Prerequisites

This topic builds directly on:
- **Unit 2.2 - Loops** (id: `0ef5b1c5-43be-4f7a-8e00-ac4ab29451e5`) — the `for` loop is reused here to walk through the values a function collects, and the idea of an off-by-one error carries over to getting a recursive step exactly right.
- **Unit 2.1 - Conditionals** (id: `c9feb41c-1aee-404d-b359-d468df334892`) — `if`/`else` is the mechanism used inside a function to decide between a base case and a recursive case.
- **Unit 1.4 - Statements, Conversion & Output** (id: `3c04d4c0-e9ca-4a4a-9d5e-368effd5d162`) — comments and f-strings are used in the examples below exactly as introduced there.
- **Unit 1.2 - Variables, Identifiers & Types** (id: `44580fc5-261e-4a0a-bc9b-22972428f96e`) — variables, assignment, and `NameError` are the foundation that "scope" (Section 5 below) refines.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** what a function is and why organizing code into functions makes a program easier to build, read, and fix.
- **Implement** a function using `def`, give it one or more parameters, and hand a value back to the caller with `return`.
- **Differentiate** positional arguments, keyword arguments, default arguments, and the extra arguments collected by `*args`.
- **Describe** the difference between a local variable and a global variable, and identify when the `global` keyword is genuinely needed.
- **Apply** the base-case-plus-recursive-case pattern to write a simple recursive function, and recognize what causes a `RecursionError`.
- **Apply** a docstring to a function you write, following the basic convention of stating what it does, what it expects, and what it returns.

## 4. Introduction

Imagine you are building the checkout page for a food delivery app. Every single order needs its bill calculated the same way: add up the item prices, add a delivery fee, maybe add a packaging charge. If you typed that calculation out fresh, by hand, everywhere in your program it was needed — once for a normal order, once for a bulk order, once for a test — you would have the same few lines of arithmetic copy-pasted all over the place. The day the delivery fee changes from ₹30 to ₹35, you would have to hunt down and fix every single copy. Miss one, and the bug hides until a customer complains about the wrong bill.

A **function** solves this by giving that calculation a name, once, in one place. Anywhere else in the program that needs a bill calculated, it just calls that name. Change the calculation once, and every caller gets the fix automatically. You have actually been *using* functions since Unit 1.1 — every time you wrote `print("hello")` or `type(x)`, you were calling a function someone else had already written and named for you. This topic teaches you to write your own.

By the end of this topic you will be able to package a task into a function, feed it different inputs in several ways, understand why a value created inside a function disappears the moment the function finishes, and write a function that solves a problem by calling itself.

## 5. Core Concepts

### 5.1 Defining and calling a function

A **function** is a named, reusable block of code that performs one specific task[1]. You write the task's logic once; then you **call** — run — that function as many times as you like, from anywhere in your program, without retyping the logic.

You create a function with the `def` keyword (short for "define"), followed by a name you choose, a pair of parentheses, and a colon. The indented lines underneath are the function's **body** — the instructions it runs each time it is called[1]:

```python
def greet():
    print("Hello there")
```

Writing this `def` block does **not** run anything by itself — it only teaches Python the name `greet` and what should happen when that name is called. Nothing is printed yet. To actually run the body, you write the function's name followed by parentheses — this is a **function call**:

```python
greet()   # runs the body -> Hello there
```

If `greet` (without parentheses) is written on its own, Python treats it as a reference to the function itself, not an instruction to run it. Parentheses are what trigger the call[1].

### 5.2 `return` versus `print()`, and `None`

A function that only prints something has shown a value on the screen, but it has not handed anything back to whoever called it — nothing else in the program can retrieve that value afterward. The **`return`** statement solves this: it ends the function immediately and produces a **return value**, a result the caller can store in a variable, print, or use in a further calculation[1]:

```python
def square(n):
    return n * n

answer = square(5)   # answer is now 25
```

This is the single most important distinction in this topic. `print()` displays a value on screen, and that value is then gone — nothing else in the program, including an automated test, can get it back. `return` hands the value to the caller, so the rest of the program can use it[1]. If a function has no `return` statement at all, or a bare `return` with nothing after it, Python automatically hands back a special value called **`None`** — a built-in value meaning "no meaningful result." A function that only prints and never returns anything has, from the rest of the program's point of view, produced `None`.

### 5.3 Parameters and arguments

A **parameter** is a name listed inside the parentheses when a function is **defined** — a placeholder for a value that will be supplied later. An **argument** is the actual value supplied when the function is **called**[1]. In `def power(base, exponent):`, `base` and `exponent` are parameters; in the call `power(2, 3)`, `2` and `3` are arguments. Keeping these two words straight matters: parameter = the name in the definition, argument = the value in the call[1].

Arguments can be supplied in three ways:

- **Positional arguments** are matched to parameters purely by their order in the call[1]:
  ```python
  def power(base, exponent):
      result = 1
      for _ in range(exponent):
          result = result * base
      return result

  power(2, 3)   # base=2, exponent=3 -> 8
  ```
- **Keyword arguments** are matched by name instead of position, so the order in which you write them stops mattering[1]:
  ```python
  power(exponent=3, base=2)   # still 8 -- names win over position
  ```
- **Default arguments** give a parameter a fallback value that Python uses automatically whenever the caller leaves that argument out, which makes supplying it optional[1]:
  ```python
  def greet(name, greeting="Hello"):
      return greeting + ", " + name

  greet("Sam")               # "Hello, Sam"
  greet("Sam", "Welcome")    # "Welcome, Sam"
  ```

Two ordering rules follow directly from this: in a `def` line, any parameter with a default must come *after* every parameter without one; in a call, positional arguments must appear *before* any keyword arguments[1]. One trap worth knowing early: a default value is calculated once, at the moment Python reads the `def` line — not freshly on every call. For a plain number or string this never causes a problem, so the safe habit is to keep every default simple and fixed: a number, a string, or `None`[1].

### 5.4 `*args` — collecting extra positional arguments

Sometimes a function cannot know in advance how many arguments a caller will pass. Writing a single `*` before a parameter name tells Python to gather every extra positional argument together under that one name — by convention, `*args`[1]:

```python
def total(*args):
    running = 0
    for value in args:
        running = running + value
    return running

total(3, 4, 5)   # 12
total()          # 0
```

Inside the function, `args` behaves like a plain sequence of values that a `for` loop (Unit 2.2) can walk through one at a time. The `*` symbol is what does the actual work of gathering; `args` is simply the conventional name everyone uses[1]. The idea to hold onto: `*args` lets one function accept a flexible, unknown number of positional inputs instead of forcing an exact, fixed parameter list.

### 5.5 Scope — local versus global

**Scope** is the region of a program where a given name is visible and usable[1]. A variable created inside a function is **local** to that function: it exists only while the function is running, and code outside the function cannot see or use it[1]:

```python
def compute():
    temp = 42        # local variable — exists only inside compute
    return temp

compute()
print(temp)          # NameError: temp is not defined out here
```

A variable defined at the top level of a file, outside every function, is a **global variable**, and any function can *read* it freely[1]. The subtlety: if a function *assigns* a value to a name anywhere in its body, Python treats that name as local to that function by default — even if a global variable with the same name already exists elsewhere. This protects one function from accidentally overwriting another function's variables. When a function genuinely needs to reassign a global variable on purpose, the **`global`** keyword states that intention explicitly[1]:

```python
counter = 0

def bump():
    global counter
    counter = counter + 1

bump()
print(counter)       # 1
```

Functions that take input through parameters and hand results back through `return` are easier to read, reuse, and test than functions that quietly reach outside themselves and change shared state — so `global` should be treated as a rare, deliberate exception rather than routine practice[1].

### 5.6 Recursion — base case and recursive case

**Recursion** is a technique where a function solves a problem by calling itself on a smaller version of the same problem[1]. Every correct recursive function needs exactly two parts:

- The **base case** is the simplest possible input — one small enough that the answer is already known, with no further calling required. It is what stops the recursion from running forever[1].
- The **recursive case** is the branch where the function calls itself on a smaller or simpler input, then combines that result to produce its own answer[1].

The classic example is factorial: `n!` (read "n factorial") means `n × (n-1) × (n-2) × ... × 1`, with `0!` defined as `1`. Since `n!` equals `n × (n-1)!`, the definition is already recursive in shape[1]:

```python
def factorial(n):
    """Return n! for a non-negative integer n."""
    if n == 0:                      # base case
        return 1
    return n * factorial(n - 1)     # recursive case

factorial(4)   # 4 * 3 * 2 * 1 = 24
```

Every call to `factorial` that has not yet reached the base case waits, unfinished, on the **call stack** — the mechanism Python uses internally to track every function call still in progress and still waiting on a result[1]. `factorial(4)` calls `factorial(3)`, which calls `factorial(2)`, which calls `factorial(1)`, which calls `factorial(0)` — the base case — and then each waiting call finishes in reverse order, multiplying as it goes, back up to `24`.

If a base case were missing, or written so it could never actually be reached, the function would keep calling itself indefinitely, adding a new entry to the call stack on every call. Python limits how deep this stack may grow; once a program exceeds that limit, Python stops it and raises a **`RecursionError`**, reporting "maximum recursion depth exceeded"[1]. This is a controlled, catchable failure — not a silent hang and not a crashed program. The exact same kind of repeated calculation can also be done with a `for` loop and an accumulator (Unit 2.2); recursion is not faster, it is a different way of *thinking* about a problem that already looks like a smaller copy of itself. Getting the base case exactly right is the entire game[1].

### 5.7 Docstrings

A **docstring** is a string literal placed as the very first statement inside a function's body, written in triple quotes[1]. Python stores this string so that other developers — or tools reading the code — can see what the function does without reading its implementation. The convention for writing docstrings is formalized in **PEP 257** ("PEP" stands for Python Enhancement Proposal — a numbered design document that records an agreed Python convention; you already met one, PEP 8, in Unit 1.4, which covers code style)[1]:

```python
def factorial(n):
    """Return n! for a non-negative integer n."""
    if n == 0:
        return 1
    return n * factorial(n - 1)
```

A good docstring states, in one or a few plain sentences, what the function does, what it expects as input, and what it returns[1]. Because it lives inside the function itself, it travels everywhere the code travels and cannot silently drift out of date the way a separate document might.

## 6. Implementation

Writing a function reliably follows the same short sequence of decisions every time:

1. **Name the task.** Choose a name that states what the function does (`calculate_bill`, not `do_stuff`).
2. **List the parameters.** Decide what inputs the task genuinely needs; give any optional input a default value, written after all required parameters.
3. **Write the docstring** as the first line of the body, stating what the function returns.
4. **Write the body**, using ordinary statements — assignments, loops, conditionals — exactly as covered in earlier units.
5. **Return the result** explicitly with `return`, rather than only printing it.
6. **Call the function** from elsewhere in the program, and store or use the returned value.

For a recursive function specifically, the sequence narrows to two design questions, answered in this order:

1. **What is the base case?** Identify the smallest input for which the answer is already known without any further calling.
2. **What is the recursive case?** Identify how a larger input reduces to a smaller one, and how that smaller result combines into the final answer. Confirm that every recursive call moves strictly closer to the base case — never stays the same size or moves away from it — otherwise the base case can never be reached and a `RecursionError` follows.

## 7. Real-World Patterns

Functions are the basic organizing unit of any serious codebase an engineer will touch:

- **Banking and FinTech:** interest calculations, EMI (equated monthly instalment) schedules, and balance updates are each implemented as small functions — each taking clear inputs (principal, rate, tenure) and returning one clear value[1].
- **UPI (Unified Payments Interface) and payment systems:** a payment flow calls a chain of functions — validate account, check balance, debit sender, credit receiver, generate transaction ID — each independently testable because each returns a value the next function, or an automated test, can check[1].
- **E-commerce:** a checkout page calls functions to total item prices, apply a discount, add a delivery charge, and compute a final bill — the same shape as the billing examples used to teach this topic[1].
- **AI (artificial intelligence) and cloud applications:** every model call made later in this program is, underneath, a function that takes an input, performs a computation, and returns an output — the exact `def` / parameters / `return` shape taught here, just wrapped inside a larger library[1].

Returning a value, rather than only printing it, matters especially in these settings because an automated test checks a function's *returned* value without a human watching the screen[1].

## 8. Best Practices

- Make every function do exactly **one job**, and choose a name that states what that job is.
- Prefer `return` over `print()` inside a function meant to be reused — printing is for showing a human a final result, not for handing a value to the rest of the program[1].
- Write a one-line docstring for every function, stating what it does, what it expects, and what it returns[1].
- Pass data in through parameters and get results out through `return`, instead of reaching for global variables — this keeps a function testable on its own[1].
- Keep default argument values simple and fixed — a number, a string, or `None` — never something that changes after creation[1].
- Reserve the `global` keyword for the rare case where a function must genuinely update shared state, not as everyday habit[1].
- Before trusting a recursive function, double-check that its base case is actually reachable from every input the recursive case can produce[1].

Common mistakes worth watching for: forgetting `return` (the function silently hands back `None`); confusing `print()` with `return`; assigning to a name inside a function without `global` when a global update was intended (this creates a new local variable instead, and if that name is also read before the assignment, Python raises `UnboundLocalError`); and a missing or unreachable recursion base case, which ends in a `RecursionError`[1].

## 9. Hands-On Exercise

Write a function `order_total(item_price, quantity, packing_fee=10)` that returns `item_price * quantity + packing_fee`, with a one-line docstring. Call it twice: once supplying only `item_price` and `quantity` (letting `packing_fee` default), and once overriding `packing_fee` to `15` using a keyword argument. Print both results, then write a small recursive function `orders_placed(day)` that returns the running total of orders processed from day 1 through `day`, where day `n` processes `n` orders that day; confirm `orders_placed(4)` returns `10`.

## 10. Key Takeaways

- A **function** is defined with `def`, does one job, and hands a result back with `return`; returning — not printing — is what makes that result usable by the rest of a program or by an automated test.
- A **parameter** is the name in a function's definition; an **argument** is the value supplied in a call — arguments can be positional, keyword, or fall back to a default, and `*args` collects any extra positional arguments into one name to loop over.
- A variable assigned inside a function is **local** by default and disappears when the function ends; a **global** variable can be read freely from inside a function, but reassigning it there requires the `global` keyword.
- **Recursion** solves a problem by having a function call itself on a smaller version of that problem; it always needs a reachable **base case** and a **recursive case** that moves measurably closer to it, or Python eventually raises a `RecursionError`.
- A **docstring** — a triple-quoted string as a function's first statement — documents what the function does, following the PEP 257 convention, and travels with the code wherever it is used.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
