# Functions

By now you can make decisions with `if`/`elif`/`else` and repeat actions with `for` and `while` loops — real programming power. But picture writing the checkout logic for a food delivery app: add up the item prices, add a delivery fee, maybe add a packing charge. You need that exact calculation in a dozen places — once for a normal order, once for a bulk order, once for a test run. Copy-pasting the same few lines everywhere is exactly the kind of repetition programmers go out of their way to avoid, and it is a ticking time bomb: the day the delivery fee changes from ₹30 to ₹35, you have to hunt down and fix every single copy. Miss one, and the bug hides until a customer complains about the wrong bill.

There is a better way, and you have actually been using it since Unit 1.1 without necessarily noticing. Every time you wrote `print("hello")` or `type(x)`, you were calling a function — a named, reusable block of logic someone else had already written once and packaged up for you to reuse endlessly. This chapter teaches you to write your own: how to define one, how to feed it input in several different ways, why a value created inside one quietly vanishes the moment it finishes, and how a function can even call itself to solve a problem.

By the time you finish this chapter, packaging a calculation into a function — instead of retyping it — will feel like the obvious way to write code, not an extra step.

---

## Why copy-pasting logic doesn't scale

Think about a few everyday systems and how many times the "same" piece of logic has to run:

| System | Logic repeated everywhere |
|---|---|
| Food delivery app (Swiggy-style) | Calculate item total + delivery fee + packing charge, for every order |
| UPI payment app | Validate account, check balance, debit sender, credit receiver — for every transaction |
| Banking app | Compute EMI (equated monthly instalment) for every loan a customer opens |
| E-commerce checkout | Apply a discount and compute a final bill, for every cart |

In every row, the *logic* is identical each time — only the numbers change. If that logic is typed out fresh at every location it's needed, one bug fix means finding and repairing every copy by hand, and one missed copy means a silent, live bug. A **function** solves this by giving the calculation a name, once, in one place. Every other part of the program that needs that calculation just calls the name. Fix the logic once, and every caller gets the fix automatically — nothing to hunt down.

## Defining and calling a function

Think of a function the way you'd think of a kitchen appliance — a mixer, say. You build it once, and from then on you don't rebuild it every time you want to mix something; you just switch it on and feed it ingredients. A function works the same way: you write its logic once, and then you **call** it — run it — as many times as you like, from anywhere in your program, without retyping a single line.

You create one with the `def` keyword (short for "define"), followed by a name you choose, a pair of parentheses, and a colon. The indented lines underneath are the function's **body** — the instructions it runs each time it's called:

```python
def greet():
    print("Hello there")
```

Run this cell in Colab and — predict it before you check — nothing happens. No output appears at all. Writing a `def` block doesn't run anything; it only teaches Python the name `greet` and what should happen whenever that name gets called later. To actually run the body, you write the function's name followed by parentheses — this is a **function call**:

```python
greet()
```

```
Hello there
```

**Writing `greet` without the parentheses refers to the function itself, not an instruction to run it — the parentheses are what actually trigger the call.** This trips up nearly everyone at least once: type `greet` alone into a cell and Python shows you something like `<function greet at 0x...>` rather than running the body at all.

## `return` versus `print()` — the single most important distinction in this chapter

A function that only prints has shown a value on the screen, and that's it — the value is gone the instant the line finishes, and nothing else in your program can get it back. Compare that to a UPI app's balance check: if the function that checks your balance only *printed* "Sufficient funds," the rest of the payment flow would have no way to actually know whether to proceed. It needs the answer handed back to it, not merely displayed.

That's exactly what the **`return`** statement does: it ends the function immediately and hands a **return value** back to whoever called it — a result the caller can store in a variable, print, or feed into a further calculation.

```python
def square(n):
    return n * n

answer = square(5)
print(answer)
```

```
25
```

**`print()` displays a value and then it's gone; `return` hands the value to the caller so the rest of the program can actually use it.** If a function has no `return` statement at all — or a bare `return` with nothing after it — Python automatically hands back a special value called **`None`**, meaning "no meaningful result." A function that only prints, and never returns, has produced `None` as far as the rest of your program is concerned. Try it yourself: write `result = greet()` using the `greet()` function from above, then `print(result)` — you'll see `Hello there` printed once (from inside the call) followed by `None` (the value `result` actually holds).

## Parameters and arguments: feeding a function its inputs

A function that always does the exact same thing regardless of input is only marginally useful. What makes `power(2, 3)` or `print("hello")` genuinely flexible is that you can feed each call different values. A **parameter** is a name listed inside the parentheses when a function is *defined* — a placeholder for a value that will be supplied later. An **argument** is the actual value supplied when the function is *called*.

```python
def power(base, exponent):
    result = 1
    for _ in range(exponent):
        result = result * base
    return result

print(power(2, 3))
```

```
8
```

Here, `base` and `exponent` are parameters — they exist only in the `def` line, as placeholders. `2` and `3` are arguments — the actual values handed over in the call. Keep the two words straight: parameter is the name in the definition, argument is the value in the call — mixing them up in conversation is harmless, but confusing what each one *does* is not.

Python actually gives you three different ways to supply arguments, and a fourth mechanism for when you don't even know how many arguments are coming.

### Positional arguments

**Positional arguments** are matched to parameters purely by their order in the call — first argument to first parameter, second to second, and so on. `power(2, 3)` above is positional: `2` lands on `base` because it's written first, `3` lands on `exponent` because it's written second. Swap the order — `power(3, 2)` — and you get a completely different answer, because the *position* is all Python looks at.

### Keyword arguments

**Keyword arguments** are matched by name instead of position, so the order you write them in stops mattering:

```python
print(power(exponent=3, base=2))
```

```
8
```

Even though `exponent` is written first here, the result is identical to `power(2, 3)`, because naming each argument tells Python exactly which parameter it belongs to, regardless of order.

### Default arguments

**Default arguments** give a parameter a fallback value that Python uses automatically whenever the caller leaves that argument out — which makes supplying it optional:

```python
def greet_customer(name, greeting="Hello"):
    return greeting + ", " + name

print(greet_customer("Ananya"))
print(greet_customer("Ananya", "Welcome"))
```

```
Hello, Ananya
Welcome, Ananya
```

The first call omits `greeting` entirely, so Python falls back to `"Hello"`. The second call supplies its own value, `"Welcome"`, which overrides the default.

| Style | Matched by | Example | Order matters? |
|---|---|---|---|
| Positional | Position in the call | `power(2, 3)` | Yes |
| Keyword | Parameter name | `power(exponent=3, base=2)` | No |
| Default | Falls back if omitted | `greet_customer("Ananya")` | N/A — it's optional |

Two ordering rules follow directly from this and Python enforces them strictly: in a `def` line, any parameter with a default must come *after* every parameter without one — `def f(a, b=1)` is fine, `def f(a=1, b)` is a `SyntaxError`. And in a call, positional arguments must appear *before* any keyword arguments — `power(2, exponent=3)` works, `power(base=2, 3)` does not.

**A default value is calculated once, at the moment Python reads the `def` line — not freshly on every call — so keep every default simple and fixed: a number, a string, or `None`.** This is a genuinely famous Python gotcha once you reach mutable defaults like lists, and the safe habit avoids it entirely: never default to anything that could change after creation.

### `*args` — collecting an unknown number of extra arguments

Sometimes a function genuinely cannot know in advance how many arguments a caller will pass. A shopping cart total might need to add up two items today and fifteen tomorrow. Writing a single `*` before a parameter name tells Python to gather every extra positional argument together under that one name — by convention, `*args`:

```python
def cart_total(*args):
    running = 0
    for price in args:
        running = running + price
    return running

print(cart_total(149, 249, 99))
print(cart_total())
```

```
497
0
```

Inside the function, `args` behaves like a plain sequence of values — a `for` loop can walk through it one value at a time, exactly like walking through a list. The `*` is what does the actual work of gathering; `args` is simply the conventional name everyone in the Python community uses, so other developers instantly recognise it.

### `**kwargs` — collecting an unknown number of extra named arguments

`*args` handles a flexible number of *positional* arguments, but what about a flexible number of *named* ones? Writing two asterisks before a parameter name — by convention, `**kwargs`, short for "keyword arguments" — gathers every extra keyword argument into a dictionary, where each argument's name becomes a key and its value becomes that key's value:

```python
def build_order_summary(**kwargs):
    for item_name, price in kwargs.items():
        print(item_name, "->", price)

build_order_summary(samosa=15, cold_drink=20, packing_fee=10)
```

```
samosa -> 15
cold_drink -> 20
packing_fee -> 10
```

Here, nobody had to declare `samosa`, `cold_drink`, or `packing_fee` as parameters in advance — `**kwargs` accepted whatever named values the caller chose to pass, under whatever names the caller chose. This is precisely how an IRCTC-style booking function might accept an open-ended set of passenger details (`name=`, `age=`, `berth_preference=`) without needing a rigid, fixed parameter list for every possible detail a caller might supply.

| Mechanism | Collects | Appears inside the function as | Symbol |
|---|---|---|---|
| Positional | Extra un-named arguments, in order | A tuple | `*args` |
| Keyword | Extra named arguments | A dictionary | `**kwargs` |

**A single function can combine all four styles, but they must appear in this fixed order: regular positional parameters, then a default parameter, then `*args`, then `**kwargs`** — getting this order wrong raises a `SyntaxError` before your program even runs.

## Scope: where a name is actually visible

Here's a question worth predicting before you read on: if a variable is created *inside* a function, does the rest of your program know it exists once the function finishes? Try it:

```python
def compute():
    temp = 42
    return temp

compute()
print(temp)
```

```
NameError: name 'temp' is not defined
```

**Scope** is the region of a program where a given name is visible and usable. A variable created inside a function is **local** to that function: it exists only while the function is running, and the moment the function finishes, that name is gone — code outside cannot see or use it. This is not a limitation; it's a deliberate safety feature. It means one function's internal working variables can never accidentally collide with another function's variables of the same name, the same way two different UPI apps on your phone can each use a variable called `balance` internally without ever interfering with each other.

A variable defined at the top level of a file, outside every function, is a **global variable**, and any function can *read* it freely:

```python
tax_rate = 0.18

def price_with_tax(amount):
    return amount + (amount * tax_rate)

print(price_with_tax(100))
```

```
118.0
```

`price_with_tax` never received `tax_rate` as a parameter, yet it read it without any trouble — that's global read access.

Here's the subtlety that catches learners out: if a function *assigns* a value to a name anywhere in its body, Python treats that name as local to that function by default — even if a global variable with the same name already exists. Consider a counter meant to track total orders processed across the whole app:

```python
counter = 0

def bump():
    counter = counter + 1   # this creates a NEW local "counter"

bump()
print(counter)
```

```
0
```

Notice `counter` is still `0` after calling `bump()` — the function didn't touch the global `counter` at all; the line `counter = counter + 1` quietly created a brand-new local variable, used it once, and threw it away. When a function genuinely needs to reassign a global variable on purpose, the **`global`** keyword states that intention explicitly:

```python
counter = 0

def bump():
    global counter
    counter = counter + 1

bump()
print(counter)
```

```
1
```

**Assigning to a name inside a function makes it local by default, even if a global with the same name already exists — reach for the `global` keyword only when you genuinely mean to update shared state, not as routine practice.** Functions that take input through parameters and hand results back through `return` are far easier to read, reuse, and test than functions that quietly reach outside themselves and change shared state — a payment-processing function buried in a banking system that silently mutates a global `account_balance` is exactly the kind of code that becomes impossible to debug once ten other functions also touch that same global.

## Recursion: a function that calls itself

Some problems are naturally defined in terms of smaller versions of themselves. Consider factorial: `n!` (read "n factorial") means `n × (n-1) × (n-2) × ... × 1`, with `0!` defined as `1`. Notice that `n!` is just `n × (n-1)!` — the definition of factorial already contains a smaller factorial inside it. **Recursion** is a technique where a function solves a problem by calling itself on a smaller version of the same problem, and factorial is the textbook example:

```python
def factorial(n):
    if n == 0:
        return 1
    return n * factorial(n - 1)

print(factorial(4))
```

```
24
```

Every correct recursive function needs exactly two parts, and missing either one breaks it. The **base case** is the simplest possible input — one small enough that the answer is already known, with no further calling required; here, `n == 0` returning `1` is the base case, and it's what stops the recursion from running forever. The **recursive case** is the branch where the function calls itself on a smaller input and combines that result to produce its own answer; here, `n * factorial(n - 1)` is the recursive case.

Picture what actually happens when `factorial(4)` runs. Every call that hasn't yet reached the base case waits, unfinished, on what's called the **call stack** — the mechanism Python uses internally to track every function call still in progress, still waiting on a result:

```
factorial(4) calls factorial(3) calls factorial(2) calls factorial(1) calls factorial(0)
factorial(0) returns 1                                    <- base case reached
factorial(1) returns 1 * 1 = 1
factorial(2) returns 2 * 1 = 2
factorial(3) returns 3 * 2 = 6
factorial(4) returns 4 * 6 = 24
```

Each waiting call finishes in reverse order, multiplying as it unwinds back up, until `factorial(4)` finally hands back `24`.

**Forgetting a base case — or writing one that can never actually be reached — is the classic recursion mistake, and it does not cause a silent infinite hang.** Instead, the function keeps calling itself, adding a new entry to the call stack on every call, until Python's own limit on stack depth is exceeded and it raises a **`RecursionError`**, reporting "maximum recursion depth exceeded." This is a controlled, catchable failure, not a crashed program — a genuine safety net.

Before moving on, predict what `factorial(-1)` would do with the version above. There's no way to reach `n == 0` by repeatedly subtracting `1` from `-1` — you'd go `-1, -2, -3, ...` forever — so this call runs straight into a `RecursionError`. A production-quality version would guard against a negative input explicitly, but the point to internalise here is simpler: recursion is only as safe as its base case is reachable.

It's worth being clear that recursion is not some special, faster technique — the exact same factorial calculation can be written with a `for` loop and a running total, and for most simple cases a loop is actually the more efficient choice in Python. Recursion earns its place when a problem is *naturally* defined in terms of a smaller version of itself — walking a folder that contains folders that contain folders, for instance — where forcing a loop-based solution would be far more awkward than letting the function call itself.

## Docstrings: documenting a function where the code lives

A **docstring** is a string literal placed as the very first statement inside a function's body, written in triple quotes. Python stores this string so that other developers — or tools reading the code — can see what a function does without reading its implementation line by line:

```python
def factorial(n):
    """Return n! for a non-negative integer n."""
    if n == 0:
        return 1
    return n * factorial(n - 1)
```

The convention for writing docstrings is formalised in **PEP 257** ("PEP" stands for Python Enhancement Proposal — a numbered design document recording an agreed Python convention; you already met one, PEP 8, when you learned naming conventions). A good docstring states, in one or a few plain sentences, what the function does, what it expects as input, and what it returns.

**A docstring lives inside the function itself, so it travels everywhere the code travels and cannot silently drift out of date the way a separate design document can.** In a real engineering team — say, at a company maintaining a banking core system — a new developer can call `help(calculate_emi)` and immediately see what the function expects and returns, without having to track down whoever wrote it or read every line of its body.

## Functions in the real world

Functions are the basic organising unit of any serious codebase you'll touch as an engineer:

- **Banking and fintech** — interest calculations, EMI schedules, and balance updates are each their own small function, taking clear inputs (principal, rate, tenure) and returning one clear value.
- **UPI payment systems** — a single payment calls a chain of functions: validate account, check balance, debit sender, credit receiver, generate transaction ID — each independently testable because each *returns* a value the next function, or an automated test, can check.
- **E-commerce** — a checkout page calls functions to total item prices, apply a discount, add a delivery charge, and compute a final bill.
- **AI and cloud applications** — every model call you'll make later in this programme is, underneath, a function that takes an input, performs a computation, and returns an output — the exact `def` / parameters / `return` shape from this chapter, just wrapped inside a much larger library.

Returning a value rather than only printing it matters especially in these settings, because an automated test checks a function's *returned* value with no human watching the screen at all.

A handful of mistakes are worth watching for deliberately as you start writing your own functions:

- Forgetting `return` entirely — the function silently hands back `None`, and code that expected a real value breaks somewhere downstream.
- Confusing `print()` with `return` — one displays, the other hands a value back to the rest of the program.
- Assigning to a name inside a function without `global`, when you actually meant to update the outer variable — this quietly creates a new local variable instead, and if that same name is read before it's assigned, Python raises an `UnboundLocalError`.
- A missing or unreachable recursion base case — ending, sooner or later, in a `RecursionError`.
- Defaulting a parameter to something other than a simple, fixed value.

## Try it yourself

Do this in a Colab cell before moving on. Write a function `order_total(item_price, quantity, packing_fee=10)` that returns `item_price * quantity + packing_fee`, with a one-line docstring stating what it returns. Call it twice: once supplying only `item_price` and `quantity` and letting `packing_fee` default, and once overriding `packing_fee` to `15` using a keyword argument. Print both results. Then write a small recursive function `orders_placed(day)` that returns the running total of orders processed from day 1 through `day`, where day `n` processes `n` orders that day — confirm for yourself that `orders_placed(4)` returns `10`, and identify, in one sentence, what its base case and recursive case are.

---

### Key Terminology

- **Function** — a named, reusable block of code that performs one specific task.
- **`def`** — the keyword that defines a function.
- **Function call** — running a function's body by writing its name followed by parentheses.
- **`return`** — ends a function and hands a value back to the caller.
- **`None`** — the value Python returns automatically when a function has no explicit `return`.
- **Parameter** — a placeholder name listed in a function's definition.
- **Argument** — the actual value supplied in a function call.
- **Positional argument** — an argument matched to a parameter by its order in the call.
- **Keyword argument** — an argument matched to a parameter by name, regardless of order.
- **Default argument** — a parameter's fallback value, used automatically when the caller omits it.
- **`*args`** — collects extra positional arguments into a tuple inside the function.
- **`**kwargs`** — collects extra keyword arguments into a dictionary inside the function.
- **Scope** — the region of a program where a given name is visible and usable.
- **Local variable** — a variable that exists only inside the function where it was created.
- **Global variable** — a variable defined at the top level of a file, readable from any function.
- **`global`** — the keyword that lets a function explicitly reassign a global variable.
- **Recursion** — a technique where a function solves a problem by calling itself on a smaller version of it.
- **Base case** — the simplest input in a recursive function, where the answer is already known and no further call is made.
- **Recursive case** — the branch where a function calls itself on a smaller input.
- **Call stack** — the mechanism tracking every function call still in progress and waiting on a result.
- **`RecursionError`** — raised when recursion exceeds Python's maximum stack depth.
- **Docstring** — a triple-quoted string as a function's first statement, documenting what it does.

### Mastery Checkpoint

Before moving to Unit 2.4, check that you can answer these without looking back:

1. Why does calling a function that only contains `print()` statements and no `return` statement give you back `None` if you try to store its result in a variable?
2. What is the difference between a positional argument, a keyword argument, and a default argument — and what ordering rule does Python enforce for each in a `def` line versus a call?
3. `*args` and `**kwargs` both collect a flexible number of extra arguments — what is the key difference between what each one collects, and what data structure does each produce inside the function?
4. A function assigns to a variable name that also exists as a global variable, but never uses the `global` keyword. What actually happens, and why might this surprise someone expecting the global to be updated?
5. What are the two parts every correct recursive function must have, and what specifically goes wrong — and what error results — if one of them is missing or unreachable?

### Summary

You now know how to package a repeated calculation into a function with `def`, hand a result back to the caller with `return` instead of merely printing it, and feed that function input in four different ways — positional, keyword, default, and the flexible `*args`/`**kwargs`. You have seen why a variable created inside a function disappears once the function finishes, why assigning to a name inside a function makes it local by default, and when the `global` keyword is the genuine, deliberate exception to that rule. You have also written a recursive function, learned to identify its base case and recursive case, and understood what a `RecursionError` really means — and you have started documenting your own functions with a docstring, exactly as a professional codebase expects. From here, the next step is learning about functional constructs that treat functions themselves as values you can pass around — starting with Unit 2.4, Functional Constructs: lambdas, higher-order functions, decorators, and generators.

### Additional Resources

- [Python Tutorial — official docs: "Defining Functions"](https://docs.python.org/3/tutorial/controlflow.html#defining-functions)
- [Python Tutorial — official docs: "More on Defining Functions" (default arguments, keyword arguments, `*args`, `**kwargs`)](https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions)
- [Python 3 Documentation — `RecursionError`](https://docs.python.org/3/library/exceptions.html#RecursionError)
- [PEP 257 — Docstring Conventions](https://peps.python.org/pep-0257/)
- [W3Schools — Python Functions](https://www.w3schools.com/python/python_functions.asp)
- [W3Schools — Python Function Arguments](https://www.w3schools.com/python/python_functions_arguments.asp)
- [W3Schools — Python Scope](https://www.w3schools.com/python/python_scope.asp)
