# Functional Constructs: Lambdas, Higher-Order Functions, Decorators & Generators

You can now write and call your own functions — `def`, parameters, `return`, `*args`, all of it. But every function you have written so far has been used the same way: define it once, call it by name, wait for its answer. The next step is realising that a function itself is not fixed to that one role. In Python, a function is just another value — the same way `149` or `"Ananya Roy"` is a value — which means it can be stored in a variable, handed to another function as an argument, or even built and returned by another function.

That single shift in thinking unlocks four tools that show up constantly in real Python code, from a food-delivery backend ranking restaurants by rating, to a banking system logging every fund transfer without touching the transfer logic itself. A **lambda** lets you write a tiny, disposable function in one line, right where you need it. A **higher-order function** takes your function as input and applies it across a whole collection of values. A **decorator** wraps extra behaviour around an existing function without changing a single line of that function's own code. And a **generator** produces a sequence of values one at a time, on demand, instead of building the whole thing in memory before handing any of it back.

None of this needs new syntax — everything here is built out of `def`, `return`, and parameters, which you already know. What changes is a single idea: a function is a value you can move around, not just an action you trigger by name.

---

## A function is just another value

Think back to `item_price = 149` from Unit 1.2. That line bound a name to a number sitting in memory. Nothing stops you from doing exactly the same thing with a function:

```python
def greet(name):
    return f"Namaste, {name}"

say_hello = greet
print(say_hello("Priya"))
```

```
Namaste, Priya
```

`say_hello` was never called with parentheses on that middle line — it was *assigned*, the same way a number or a string would be. `say_hello` now refers to the exact same function object that `greet` refers to, and calling either name produces the identical result. This property — a function behaving like any other value — is called being **first-class**, and Python grants it to every function you write. Concretely, being first-class means a function can be:

- **stored in a variable**, without being called;
- **passed into another function as an argument**, the same way you'd pass a number;
- **returned from another function**, the same way a `return` statement hands back a number.

Every construct in the rest of this unit — lambdas, `map()`/`filter()`/`sorted(key=...)`, decorators, generators — is a different application of this one fact. Before moving on, say out loud, in one sentence, what the difference is between writing `greet` and writing `greet()` — one refers to the function itself, the other calls it and refers to whatever it returns. That distinction matters everywhere from here on.

## Lambda: a function small enough to write inline

A **lambda** is a small, unnamed function written on a single line, in the form `lambda parameters: expression`. It behaves like a miniature `def` function with one firm restriction: its body must be exactly one **expression** — something that produces a value — and whatever that expression evaluates to is automatically handed back. There is no `return` keyword inside a lambda anywhere.

```python
apply_discount = lambda price: price * 0.9
print(apply_discount(500))
```

```
450.0
```

Read this the same way you read the earlier `say_hello` example: `apply_discount` is a variable, and it now refers to a function. Calling `apply_discount(500)` runs the lambda with `price` set to `500`, evaluates `price * 0.9`, and that result — `450.0` — comes back with no `return` written anywhere.

Think of a lambda as a sticky note version of a function: quick to write, meant to be used once, right where you stuck it — not something you'd file away for later reference. A lambda cannot contain loops, `if`/`elif`/`else` statements, multiple lines, or a docstring. If your logic needs any of that, or needs to be called by name from several places, write a regular `def` function instead.

**A lambda that needs more than one line to express what it does is a sign you should be writing a `def` function, not squeezing logic into a lambda to save space.**

There's a practical cost to overusing lambdas, too: when something goes wrong inside one, Python's error traceback shows the name `<lambda>` rather than a meaningful function name, which makes the mistake harder to trace back to its source. Keep lambdas short, throwaway, and usually handed straight to another function as an argument — which is exactly the role they play next.

## Higher-order functions: functions that work on functions

A **higher-order function** is a function that accepts another function as an argument, returns a function, or both. This is only possible because functions are first-class values — you can't hand a function to another function unless the language treats it as an ordinary value to begin with. Three built-in higher-order functions cover the overwhelming majority of everyday use:

| Function | What it does | Returns |
|---|---|---|
| `map(function, iterable)` | Applies `function` to every item | The transformed items, one per input |
| `filter(function, iterable)` | Calls `function` on every item, keeps only truthy results | The items that passed |
| `sorted(iterable, key=function)` | Sorts using `function`'s result, not the item itself | A new sorted list |

Picture a small online bookstore with a list of prices. Ranking them, discounting every one of them, and picking out only the affordable ones are three unrelated jobs, and all three lean on a higher-order function plus a lambda:

```python
book_prices = [500, 250, 1200, 90, 650]

discounted_prices = map(apply_discount, book_prices)
print(list(discounted_prices))

affordable = filter(lambda price: price < 300, book_prices)
print(list(affordable))

print(sorted(book_prices, key=lambda price: price))
```

```
[450.0, 225.0, 1080.0, 81.0, 585.0]
[250, 90]
[90, 250, 500, 650, 1200]
```

Before checking, try predicting what `list(map(apply_discount, book_prices))` gives you, purely by tracing the discount lambda across each item in turn — that's exactly the muscle you'll use every time you reach for `map()`.

**`map()` and `filter()` do not hand you back a list — they hand back a lazy object that only produces values when something asks for them, so you must wrap the call in `list(...)` to see or reuse the contents.** This laziness is a small preview of a much bigger idea you'll meet properly with generators later in this chapter.

`sorted(..., key=...)` deserves a closer look, because it's the most common place a beginner writes their first lambda. The `key` function is called exactly once per item, and *its return value* — not the item itself — is what `sorted()` compares to decide the order. Writing `key=lambda price: price` just means "rank by the price itself," but swap in a different lambda and you can rank a list of customer records by age, a list of orders by delivery time, or a list of restaurants by rating, all with the same one-line pattern.

## Decorators: wrapping a function without touching it

Picture a gift, already wrapped and taped shut. Wrapping it in another layer of paper doesn't change what's inside — it just adds something extra around it before it reaches whoever eventually opens it. A **decorator** does exactly that to a function: it takes an existing function as input and returns a new, wrapped version of it, adding behaviour around the original without changing a single line of the original's own code.

Decorators exist because the same extra behaviour — logging every call, timing how long something takes, checking permissions before running — is often needed on many different functions at once. Without a decorator, you'd have to copy-paste that same wrapper code inside every function that needed it. A decorator lets you write that logic exactly once and apply it anywhere with a single line.

```python
def decorator_name(func):
    def wrapper(*args, **kwargs):
        # extra behaviour goes here
        return func(*args, **kwargs)
    return wrapper

@decorator_name
def target_function(...):
    ...
```

Two pieces of vocabulary matter here. The **wrapper function** is the inner function defined inside a decorator — it's what actually replaces the original function, and it's where the extra behaviour lives. And **`**kwargs`** ("keyword arguments") is the counterpart to the `*args` you already know from Unit 2.3: where `*args` collects extra positional arguments into a tuple, `**kwargs` collects extra keyword arguments into name-value pairs. Writing `wrapper(*args, **kwargs)` lets the wrapper accept absolutely any combination of arguments, so the same decorator can wrap any function, regardless of that function's own parameter list.

The `@` symbol is shorthand, nothing more: writing `@decorator_name` directly above a `def` is exactly equivalent to writing `target_function = decorator_name(target_function)` right after the function is defined. No magic happens — it's plain reassignment, leaning on the first-class-function idea from earlier in this chapter.

```python
def log_order(func):
    def wrapper(*args, **kwargs):
        print(f"Processing order for a book listed at Rs. {args[0]}")
        result = func(*args, **kwargs)
        print(f"Final payable amount: Rs. {result}")
        return result
    return wrapper

@log_order
def process_book_order(price):
    return apply_discount(price)

process_book_order(500)
```

```
Processing order for a book listed at Rs. 500
Final payable amount: Rs. 450.0
```

`wrapper(*args, **kwargs)` accepts whatever `process_book_order` accepts — here, just one positional argument — without `log_order` ever needing to know in advance how many arguments the function it wraps will take. `args[0]` reaches into the collected positional arguments to read the original price for the log message, `func(*args, **kwargs)` forwards those same arguments to the real function, and `return result` makes sure the caller still receives `450.0` rather than losing it inside the wrapper.

**The outer decorator function must always return the wrapper itself (`return wrapper`), never a call to it (`return wrapper()`) — otherwise the function being decorated breaks, since that return value is what its name now points to.** And forgetting `return result` inside `wrapper` is arguably the single most common decorator bug: the wrapped function still appears to work, because the logging or timing runs fine, but the caller silently gets back `None` instead of the real answer.

When several decorators stack above one function, they apply from the bottom up: `@a` above `@b` above `def f():` means `f = a(b(f))` — the decorator closest to the `def` wraps the function first, and each one above it wraps the result of the one below.

## Closures: how the wrapper remembers the original function

A natural question follows immediately: once `log_order` has finished running and returned `wrapper`, how does `wrapper` still know which function to call through `func`? Normally, a variable that exists only inside a function disappears the instant that function returns. A **closure** is the exception — when an inner function is defined inside an outer function and refers to one of the outer function's variables, Python keeps that variable alive and reachable for as long as the inner function itself still exists, even though the outer function's own run is long over.

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))
print(triple(5))
```

```
10
15
```

`make_multiplier(2)` runs, builds `multiply`, and returns it — at that point `make_multiplier`'s own execution is completely finished, and its local variable `factor` would normally vanish. But `multiply` refers to `factor` inside its own body, so Python keeps that specific value — `2` — attached to the function object now stored in `double`. Calling `make_multiplier(3)` forms an entirely separate closure, remembering `factor = 3` for `triple`, completely independent of `double`'s. This is exactly the mechanism that lets a decorator's `wrapper` keep reaching the original `func` long after the decorator itself has returned — no different from `double` remembering `2` forever.

## Generators: producing values one at a time

Every function you've written so far works like ordering a full thali at once — it prepares everything and hands you the complete plate in one go. A **generator function** works more like a vending machine: it produces exactly one item the moment you ask for it, and nothing else exists until you ask again.

A generator function looks almost identical to a normal `def` — the only difference is that its body contains at least one **`yield`** statement instead of, or alongside, `return`. `yield` hands back one value and *pauses* the function exactly at that point, freezing every local variable's current value, until the next value is requested.

```python
def count_up_to(limit):
    current = 1
    while current <= limit:
        yield current
        current += 1

for number in count_up_to(5):
    print(number)
```

```
1
2
3
4
5
```

Here is the detail that catches nearly everyone the first time: calling `count_up_to(5)` does **not** run `current = 1` immediately. It only builds a paused **generator object** — the body executes step by step only as values are actually requested, typically by a `for` loop or by calling `next()` directly. The first request runs `current = 1`, enters the `while` loop, and pauses at `yield current`, handing back `1`. The next request resumes exactly after that `yield`, runs `current += 1`, loops back, and pauses again — and so on, until `current` exceeds `limit` and the generator ends naturally.

Before checking, predict what `print(count_up_to(5))` alone — with no `for` loop, no `next()` — would display. It would not print `1` through `5`. It would print something like `<generator object count_up_to at 0x...>`, because merely calling the function only creates the paused object; nothing inside it has run yet.

## Lazy evaluation: why generators exist at all

**Lazy evaluation** is the general principle behind generators: produce a value only at the moment it's actually needed, instead of computing and storing an entire sequence up front. A regular function that returns a list must finish building that *entire* list before handing any of it back. That's wasteful when a sequence has millions of entries and only the first few are ever needed — and it's outright impossible for a sequence with no natural end at all, such as an endless stream of incrementing UPI transaction references.

| Aspect | List | Generator |
|---|---|---|
| When values are produced | All at once, immediately | One at a time, only when asked for |
| Memory used | Holds every value at once | Holds only the current value |
| Can represent an infinite sequence? | No | Yes |
| Can be looped over more than once? | Yes, as many times as needed | No — exhausted after one full pass |
| Created with | `[...]` or `list(...)` | A function containing `yield` |

That last row bites people in practice: **a generator object can only be iterated forward, once — loop over it fully, and looping again produces nothing at all.** Getting a fresh run means calling the generator function again to build a brand-new generator object.

## Putting all four together: an order-processing script

A small e-commerce script needs three things at once: an ever-increasing order ID with no natural end, timing information around processing each order, and orders handled cheapest-first. Each need maps to exactly one construct from this chapter — an endless sequence is a generator, timing wrapped around an existing function without editing it is a decorator, and a one-off ranking rule is a lambda.

```python
import time

def order_id_generator(start):
    current = start
    while True:
        yield current
        current += 1

def track_time(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} took {end_time - start_time:.4f} seconds")
        return result
    return wrapper

@track_time
def process_order(order_id, amount):
    print(f"Processing order {order_id} for Rs. {amount}")
    return order_id

order_amounts = [799.0, 249.5, 1499.0, 99.0]
sorted_amounts = sorted(order_amounts, key=lambda amount: amount)

for order_id, amount in zip(order_id_generator(1001), sorted_amounts):
    process_order(order_id, amount)
```

```
Processing order 1001 for Rs. 99.0
process_order took 0.0000 seconds
Processing order 1002 for Rs. 249.5
process_order took 0.0000 seconds
Processing order 1003 for Rs. 799.0
process_order took 0.0000 seconds
Processing order 1004 for Rs. 1499.0
process_order took 0.0000 seconds
```

`order_id_generator(1001)` returns a paused generator object immediately — none of its body has run yet. `sorted_amounts` places `99.0` first and `1499.0` last, since the lambda's key is just each amount itself. `zip()` then walks the generator and `sorted_amounts` together, pulling exactly four values from the generator — `1001` through `1004` — one per amount, and stopping there even though `order_id_generator` would happily keep producing IDs forever if asked. For each pair, `process_order` runs first, printing its own message as the last thing it does, and only once it returns does `wrapper` print the elapsed-time line — because that `print` sits *after* `result = func(*args, **kwargs)` inside `wrapper`.

A handful of mistakes are worth watching for deliberately as you start writing these constructs yourself:

- Writing more than one expression inside a lambda — a lambda supports exactly one expression, and anything more raises a `SyntaxError`.
- Forgetting `return result` inside a decorator's `wrapper` — the wrapped function appears to work, but its caller silently receives `None`.
- Writing `wrapper(*args)` without `**kwargs` — the decorator breaks the moment someone calls the decorated function with a keyword argument.
- Assuming `map()` or `filter()` hands you a list directly — both return a lazy object that needs `list(...)` before you can view or reuse it.
- Assuming a generator can be looped over twice — once it's exhausted, you need a fresh call to the generator function, not a second loop over the same object.

## Try it yourself

Do this in a Colab cell before moving on. Using `book_prices = [500, 250, 1200, 90, 650]`: write a lambda `add_gst` that adds 10% to a price. Use `map()` with `add_gst` to produce GST-inclusive prices and print them as a list. Then use `filter()` to keep only prices at or above 300, and `sorted(..., key=..., reverse=True)` to print them most-expensive-first. Finally, write a generator `invoice_number_generator(start)` that yields `f"INV{n}"` forever from `start`, and use `next()` to print one invoice number before each price. Afterwards, ask yourself: does calling `invoice_number_generator(start)` on its own, with no `next()` or `for` loop, print anything at all? If it doesn't, you've correctly understood that a generator's body hasn't run until something asks it for a value.

---

### Key Terminology

- **First-class function** — a function Python treats as an ordinary value: it can be stored, passed as an argument, or returned.
- **Lambda** — a small, unnamed, single-expression function written as `lambda parameters: expression`.
- **Higher-order function** — a function that takes another function as an argument, returns one, or both.
- **`map(function, iterable)`** — applies `function` to every item, returning a lazy object of the results.
- **`filter(function, iterable)`** — keeps only the items for which `function` returns a truthy result.
- **`sorted(iterable, key=function)`** — sorts by the result of calling `function` on each item, not the item itself.
- **Decorator** — a function that takes another function and returns a wrapped replacement, applied with `@`.
- **Wrapper function** — the inner function inside a decorator that actually replaces the original and holds the extra behaviour.
- **`**kwargs`** — collects any number of extra keyword arguments, the counterpart to `*args`.
- **Closure** — an inner function's ability to keep using a variable from its enclosing function, even after that outer function has returned.
- **Generator function** — a function containing at least one `yield`, which produces a paused generator object rather than running immediately.
- **`yield`** — hands back one value and pauses the function exactly there, until the next value is requested.
- **Generator object** — the paused, resumable object a generator function returns; iterable forward, once.
- **Lazy evaluation** — producing a value only when it's actually requested, instead of computing an entire sequence up front.

### Mastery Checkpoint

Before moving to Unit 2.5, check that you can answer these without looking back:

1. What does it mean for a function to be a "first-class value" in Python, and why is that fact a prerequisite for lambdas, decorators, and higher-order functions all existing?
2. Why do `map()` and `filter()` need to be wrapped in `list(...)` before you can print or reuse their results directly?
3. Write out, by hand, what `@log_order` above `def process_book_order(price):` is exact shorthand for.
4. What does a closure let a decorator's `wrapper` do, and why would `func` otherwise be unreachable once the outer decorator function has returned?
5. Why can a generator represent an infinite sequence when a list never could, and what happens if you try to loop over an already-exhausted generator object a second time?

### Summary

You've moved from calling a fixed function by name to treating a function as a value you can store, pass around, and build with — the single idea that makes lambdas, `map()`/`filter()`/`sorted(key=...)`, decorators, and generators all possible. You've written throwaway single-expression lambdas, applied higher-order functions across a list of prices, wrapped an existing function with extra behaviour using `@` without touching its original code, and seen how a closure lets that wrapper keep reaching the function it wraps. Finally, you've met generators and `yield`, and the lazy-evaluation principle that lets a sequence produce values one at a time instead of building the whole thing in memory up front. From here, the next step is packaging the functions and code you've written into modules of your own, and using professional tooling to organise a real Python project — starting with Unit 2.5, Modules, Packaging & Professional Tooling.

### Additional Resources

- [Python Tutorial — official docs: "Lambda Expressions"](https://docs.python.org/3/tutorial/controlflow.html#lambda-expressions)
- [Python 3 Documentation — Built-in Functions: `map()`](https://docs.python.org/3/library/functions.html#map)
- [Python 3 Documentation — Built-in Functions: `filter()`](https://docs.python.org/3/library/functions.html#filter)
- [Python 3 Documentation — Built-in Functions: `sorted()`](https://docs.python.org/3/library/functions.html#sorted)
- [Python 3 Documentation — "More on Defining Functions" (closures, `*args`, `**kwargs`)](https://docs.python.org/3/tutorial/controlflow.html#more-on-defining-functions)
- [Python 3 Documentation — "Generators"](https://docs.python.org/3/tutorial/classes.html#generators)
- [W3Schools — Python Lambda](https://www.w3schools.com/python/python_lambda.asp)
