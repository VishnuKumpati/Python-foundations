# Functions
 
---

## Why Functions?

By now you've written the same kind of logic more than once, a receipt line, a discount check, a loop that sums numbers. A **function** lets you write that logic once, give it a name, and reuse it anywhere, with different inputs, without copying and pasting.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Function** | A named, reusable block of code |
| **Parameter** | A named input a function expects, defined when you write it |
| **Argument** | The actual value you pass in when you call the function |
| **Return value** | The result a function hands back to whatever called it |
| **Scope** | Where a variable is visible/usable from |
| **Recursion** | A function that calls itself |
| **Docstring** | A short description of what a function does, written right inside it |

---

## Defining and Calling Functions

```python
def greet():
    print("Hello!")

greet()
```
```
Hello!
```

`def` starts a function definition, `greet` is its name, `()` holds its parameters (empty here), and the indented block below is the function's body, same indentation rule as `if` and loops.

**Nothing happens until you call it.** Defining a function just teaches Python the recipe, `greet()` is what actually runs it.

**Functions can send a value back with `return`:**

```python
def add(a, b):
    return a + b

result = add(3, 4)
print(result)
```
```
7
```

`return` hands a value back to wherever the function was called from, and immediately exits the function, any code after `return` inside the function never runs.

**No `return` statement? The function gives back `None`:**

```python
def greet():
    print("Hello!")

result = greet()
print(result)
```
```
Hello!
None
```

`None` is Python's way of representing "nothing here." It's easy to miss this, calling a function that only `print()`s (and never `return`s) and then trying to use its result will quietly give you `None` instead of an error, which can be confusing to debug.

---

## Parameters and Arguments

**Positional arguments** are matched to parameters by their order:

```python
def describe(name, age):
    print(name, "is", age, "years old")

describe("Ananya", 21)
```
```
Ananya is 21 years old
```

**Keyword arguments** are matched by name instead, so order stops mattering:

```python
describe(age=21, name="Ananya")
```
```
Ananya is 21 years old
```

**Default arguments** let a parameter be optional, using a fallback value if the caller doesn't provide one:

```python
def describe(name, age=18):
    print(name, "is", age, "years old")

describe("Rahul")
describe("Ananya", 21)
```
```
Rahul is 18 years old
Ananya is 21 years old
```

**`*args` and `**kwargs` (a first look):** for when you don't know in advance how many arguments will be passed.

```python
def total(*numbers):
    return sum(numbers)

print(total(1, 2, 3))
print(total(10, 20))
```
```
6
30
```

`*args` collects any number of positional arguments into a single group you can loop over or use directly. `**kwargs` does the same for keyword arguments:

```python
def show_info(**details):
    for key, value in details.items():
        print(key, ":", value)

show_info(name="Ananya", city="Chennai")
```
```
name : Ananya
city : Chennai
```

You won't need these constantly at this stage, but recognizing them is important, they show up often in real codebases and libraries.

---

## Type Hints

A type hint documents what type a parameter or return value is *meant* to be:

```python
def add(a: int, b: int) -> int:
    return a + b
```

`a: int` and `b: int` hint the parameter types, `-> int` hints the return type. This is exactly where type hints earn their keep, a function signature with hints tells you what to pass in and what you'll get back, without reading the whole function body.

**Important:** Python does **not enforce** this at runtime.

```python
def add(a: int, b: int) -> int:
    return a + b

print(add("3", "4"))   # runs fine, prints "34" (string concatenation), no error
```

The hint is documentation for humans and code editors, not a guarantee. But it's genuinely useful documentation, and most real Python code you'll encounter uses it.

---

## Returning More Than One Value

A function can return multiple values at once, separated by commas, and you can unpack them into separate variables the same way you saw earlier, with statements:

```python
def min_max(a, b, c):
    return min(a, b, c), max(a, b, c)

lowest, highest = min_max(4, 9, 2)
print(lowest, highest)
```
```
2 9
```

Python bundles the two return values together behind the scenes, then `lowest, highest = ...` unpacks them, exactly like `x, y = 1, 2` did earlier. (What's actually happening under the hood, a **tuple**, gets its own proper unit later on.)

---

## Scope, Local vs Global

A variable created **inside** a function only exists inside that function, this is called **local scope**:

```python
def set_score():
    score = 100
    print(score)

set_score()
print(score)   # NameError: score doesn't exist out here
```

A variable created **outside** any function is in **global scope**, visible everywhere, including inside functions (for reading):

```python
score = 100

def show_score():
    print(score)   # can read the global variable fine

show_score()
```
```
100
```

**But you can't reassign a global variable from inside a function without saying so explicitly:**

```python
score = 100

def add_bonus():
    score = score + 10   # UnboundLocalError

add_bonus()
```

**The `global` keyword (brief):** tells Python you specifically mean the global variable, not a new local one:

```python
score = 100

def add_bonus():
    global score
    score = score + 10

add_bonus()
print(score)
```
```
110
```

Use this sparingly, functions that quietly change global state are harder to reason about. It's worth knowing it exists, not something to reach for by default.

---

## Recursion

A function that calls **itself** is recursive. Every recursive function needs two things: a **base case** (when to stop) and a **recursive case** (how it calls itself with a smaller problem).

**Worked example: factorial.** `5!` (factorial of 5) means `5 × 4 × 3 × 2 × 1`.

```python
def factorial(n):
    if n == 0:              # base case: stop here
        return 1
    return n * factorial(n - 1)   # recursive case

print(factorial(5))
```
```
120
```

Tracing through it: `factorial(5)` calls `factorial(4)`, which calls `factorial(3)`, and so on, down to `factorial(0)`, which finally returns `1` without calling itself again. Then each call multiplies its own `n` by what came back, unwinding all the way up: `1 → 1 → 2 → 6 → 24 → 120`.

**Miss the base case, and you get infinite recursion:**

```python
def factorial(n):
    return n * factorial(n - 1)   # never stops, no base case

factorial(5)   # RecursionError: maximum recursion depth exceeded
```

---

## Documentation, Docstrings

A **docstring** is a string written as the very first line inside a function, describing what it does:

```python
def add(a: int, b: int) -> int:
    """Return the sum of two numbers."""
    return a + b
```

Unlike a `#` comment, a docstring is stored by Python and can be read back with `help(add)` or `add.__doc__`, tools and editors use this to show documentation as you type. Keep it short, one line is often enough, focused on *what* the function does, not *how*.

---

## Try it Yourself

**(a)** Write a function `is_even(n)` that returns `True` if `n` is even, `False` otherwise. Add type hints and a docstring.

**(b)** Write a function `calculate_bill(item_price, quantity, discount=0)` with a default discount of `0`, returning the total after discount. Call it once using positional arguments and once using keyword arguments.

**(c)** Write a recursive function `count_down(n)` that prints every number from `n` down to `1`, with a base case for `n == 0`.

**Your turn:** write a function `describe_person(name, **details)` that prints the name, then each extra detail passed in as keyword arguments (like `age=21, city="Chennai"`).

---

## Common Mistakes

- Forgetting `return`, and being surprised the function's result is `None`
- Assuming a variable created inside a function is visible outside it, it isn't, that's local scope
- Trying to reassign a global variable inside a function without `global`, causing an `UnboundLocalError`
- Writing a recursive function with no base case, or a base case that's never actually reached
- Believing a type hint is enforced, it isn't, Python still runs the code either way
- Mixing up parameters (the names in the function definition) with arguments (the actual values passed in)

---

## Interview Questions

**Q1: What does a function return if it has no `return` statement?**

A: `None`. Every function returns something, if you never write `return`, Python returns `None` by default.

**Q2: What's the difference between a parameter and an argument?**

A: A parameter is the named placeholder in the function definition (`def greet(name):`). An argument is the actual value you pass in when calling it (`greet("Ananya")`).

**Q3: What's the difference between local and global scope?**

A: A local variable only exists inside the function it was created in. A global variable is created outside any function and is visible everywhere, though you need the `global` keyword to reassign it from inside a function.

**Q4: What two things does every recursive function need?**

A: A base case, the condition where it stops calling itself, and a recursive case, where it calls itself again with a smaller version of the problem. Without a base case, it never stops.

**Q5: Are type hints enforced by Python?**

A: No. They're documentation for humans and tools like editors and type checkers, Python itself will still run code that doesn't match the hinted types.

---

## Quick Recap

- `def` defines a function; nothing runs until you call it.
- `return` sends a value back and exits immediately; no `return` means the function gives back `None`.
- Arguments can be positional, keyword, or have defaults; `*args`/`**kwargs` collect an unknown number of extra arguments.
- Type hints (`def add(a: int, b: int) -> int:`) document expected types but aren't enforced.
- A function can return multiple values at once, unpacked the same way you saw earlier.
- Local scope stays inside the function; global scope is visible everywhere, but needs the `global` keyword to reassign from inside a function.
- Recursion needs a base case and a recursive case, or it never stops.
- Docstrings document what a function does, and are readable by tools, unlike regular comments.



## Reference Links

- [The Python Tutorial — Defining Functions](https://docs.python.org/3/tutorial/controlflow.html#defining-functions)
- [Python 3 Documentation — More on Defining Functions](https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions)
- [PEP 257 — Docstring Conventions](https://peps.python.org/pep-0257/)
- [Real Python — Defining Your Own Python Function](https://realpython.com/defining-your-own-python-function/)
- [Real Python — Thinking Recursively in Python](https://realpython.com/python-recursion/)
- [W3Schools — Python Functions](https://www.w3schools.com/python/python_functions.asp)
