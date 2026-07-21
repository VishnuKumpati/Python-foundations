---
topic_id: 07a78e31-727b-4c0d-95aa-4e7712dceda2
title: Unit 2.4 - Functional Constructs
position_in_module: 4
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 2.4 - Functional Constructs — Topic Corpus

## 2. Prerequisites

This topic builds directly on two earlier units, and cannot be taught without them:

- **Unit 2.3 - Functions** (`ed4fbf1a-b7e5-4df2-b368-1ecd21e1c99c`): every construct in this unit is built out of `def`, `return`, parameters, arguments, `*args`, local scope, and docstrings. A learner who cannot yet write and call a basic function is not ready for this unit.
- **Unit 2.2 - Loops** (`0ef5b1c5-43be-4f7a-8e00-ac4ab29451e5`): `for` loops are how the values produced by generators and by `map()`/`filter()` actually get consumed, and `range()` reappears in generator examples.

Unit 1.4's tuple unpacking and Unit 1.3's operators are also assumed background but are not central.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** what it means for a function to be a first-class value in Python, and why that idea is what makes lambdas, higher-order functions, decorators, and generators possible in the first place.
- **Write** a lambda expression and identify when it is appropriate to use one instead of a regular `def` function.
- **Apply** the higher-order functions `map()`, `filter()`, and `sorted(key=...)` to transform, filter, and rank a sequence of values.
- **Implement** a basic decorator using `@` syntax that wraps an existing function with extra behavior, such as logging or timing, without modifying the original function's code.
- **Create** a generator function using `yield` and describe how its lazy, one-value-at-a-time behavior differs from a regular function that returns a complete result.
- **Identify** common mistakes involving lambdas, decorators, and generators, such as a missing `return` inside a wrapper or assuming a generator can be looped over twice.

## 4. Introduction

Imagine you are running a small online bookstore. You have a list of book prices, and you want to do three unrelated things with them: rank them from cheapest to most expensive, apply a 10% discount to every price in one step, and print a log message every time an order is processed — without rewriting your order-processing function to add that logging by hand. Up to now, every function you have written in this course has been a fixed, named block of code that you call by name. This unit introduces a different way of thinking about functions in Python: a function can be treated exactly like a number or a piece of text — stored in a variable, handed to another function as an argument, or even built and returned by another function.

That single shift in thinking is what makes four practical tools possible, and all four show up constantly in real Python code. A **lambda** lets you write a tiny, throwaway function in a single line, right at the spot where you need it — for example, telling Python exactly how to rank a list of prices. A **higher-order function** is a function that takes your function as input (or hands one back) and does something with it across a whole collection of values — `map()`, `filter()`, and `sorted()` are the three you will use constantly. A **decorator** lets you wrap extra behavior — like logging, timing, or security checks — around an existing function without touching a single line of that function's original code. A **generator** produces a sequence of values one at a time, on demand, instead of building the entire sequence in memory before handing any of it back — which matters enormously once a sequence is very large, or has no natural end at all.

None of this requires new syntax you have not already been introduced to. Everything here is built out of `def`, `return`, parameters, and `*args`, which you already know from functions[1]. What changes is the idea that a function itself is just another value you can pass around — the same way a number or a string is, and the same way a `for` loop can walk over a list of numbers, it can just as easily walk over values a function produces one at a time.

## 5. Core Concepts

### 5.1 Functions as first-class values

In every previous unit, a function was something you defined once and then *called* by writing its name followed by parentheses, like `greet()`. This unit rests on one foundational idea: in Python, a function is a **first-class object**[1] — meaning it can be treated exactly like an `int`, a `str`, or any other value already familiar to you. Concretely, this means a function can be:

- **stored in a variable** — not called, just assigned, the same way `x = 5` stores a number;
- **passed into another function as an argument** — the same way you would pass a number or a string;
- **returned from another function** — handed back the same way a `return` statement hands back a number.

Every construct in the rest of this unit — lambdas, higher-order functions, decorators, and generators — is simply a different application of this one idea: a function is a value, not just an action.

### 5.2 Lambda functions

A **lambda** is a small, unnamed function written on a single line, using the form `lambda parameters: expression`[1]. It behaves exactly like a tiny `def` function with one very specific restriction: its body must be exactly one **expression** (something that produces a value), and whatever that expression evaluates to is automatically returned — there is no `return` keyword inside a lambda at all.

```python
apply_discount = lambda price: price * 0.9
print(apply_discount(500))   # 450.0
```

Here, `apply_discount` is a variable that now refers to a function, exactly the way `x = 5` makes `x` refer to a number. Calling `apply_discount(500)` runs the lambda with `price` set to `500`, evaluates `500 * 0.9`, and that result — `450.0` — is what comes back, with no `return` written anywhere[1].

A lambda is deliberately more limited than a regular function: it cannot contain loops, `if`/`elif`/`else` blocks made of statements, multiple lines, assignment statements, or a docstring. If your logic needs any of those things, or needs to be called from many different places under a memorable name, write a regular `def` function instead — forcing complex logic into a lambda just to save a few lines makes code harder to read, not easier[1]. A lambda also shows up as `<lambda>` in error tracebacks rather than its own name, which makes it harder to debug than a named function[1]. The right use case for a lambda is short, one-off logic handed directly to another function as an argument — which is exactly the role it plays in the next section.

### 5.3 Higher-order functions

A **higher-order function** is a function that accepts another function as an argument, returns a function, or both[1]. This is only possible because functions are first-class values (Section 5.1) — you cannot pass a function into another function unless the language treats it as an ordinary value in the first place. Three built-in higher-order functions matter most for a beginner:

- **`map(function, iterable)`** applies `function` to every item of an *iterable* — something you can loop over one item at a time, such as a list — and produces the results, one per input item[1].
- **`filter(function, iterable)`** calls `function` once per item and keeps only the items for which the function's result is truthy (recall truthy/falsy from Unit 1.3), discarding the rest[1].
- **`sorted(iterable, key=function)`** sorts `iterable`, but instead of comparing the items directly, it calls `function` once per item and uses *that result* to decide the order[1].

```python
book_prices = [500, 250, 1200, 90, 650]

discounted_prices = map(apply_discount, book_prices)
print(list(discounted_prices))          # [450.0, 225.0, 1080.0, 81.0, 585.0]

affordable = filter(lambda price: price < 300, book_prices)
print(list(affordable))                 # [250, 90]

print(sorted(book_prices, key=lambda price: price))   # [90, 250, 500, 650, 1200]
```

A detail that trips up nearly every beginner: `map()` and `filter()` do **not** return a list. They return a lazy object that only produces its values when something asks for them — you must wrap the call in `list(...)` to see or reuse the contents right away[1]. This laziness is a small preview of the much more general idea of lazy evaluation covered fully with generators in Section 5.6.

`sorted(..., key=...)` deserves special attention because it is the most common place a beginner writes their first lambda: the `key` function is called exactly once per item, and its *return value* — not the item itself — is what `sorted()` compares to decide the order[1]. Writing `key=lambda price: price` in the example above just means "rank by the price itself," but the same pattern lets you rank a list of any objects by any single piece of information about them.

### 5.4 Decorators

A **decorator** is a function that takes another function as input and returns a new, wrapped version of it — adding extra behavior around the original function without changing a single line of its actual code[1]. A useful mental picture: the original function is a gift, and the decorator is the gift-wrapping paper around it — the gift itself never changes, but something extra now surrounds it before it reaches whoever "opens" it by calling the function.

Decorators exist because the same extra behavior — logging every call, timing how long something takes, checking permissions before running — is often needed on *many different functions*. Without a decorator, that same wrapper code would have to be copy-pasted inside every function that needed it[1]. A decorator lets you write that logic exactly once and apply it to any function with a single line.

```python
def decorator_name(func):
    def wrapper(*args, **kwargs):
        # extra behaviour here
        return func(*args, **kwargs)
    return wrapper

@decorator_name
def target_function(...):
    ...
```

Two new pieces of vocabulary matter here, both defined only for this unit:

- A **wrapper function** is the inner function defined inside a decorator; it is what actually replaces the original function, and it is where the extra behavior lives[1].
- **`**kwargs`** (short for "keyword arguments") is the counterpart to the `*args` you already met in Unit 2.3: where `*args` collects any number of extra *positional* arguments into a tuple, `**kwargs` collects any number of extra *keyword* arguments into a collection of matching name-value pairs (called a dictionary — a data type covered later). Writing `wrapper(*args, **kwargs)` lets the wrapper accept absolutely any combination of arguments, so the same decorator can wrap any function regardless of its own parameter list[1].

The `@` **syntax** is shorthand: writing `@decorator_name` directly above a `def` is exactly equivalent to writing `target_function = decorator_name(target_function)` immediately after the function is defined[1]. Nothing magical happens — it is plain reassignment, using the first-class-function idea from Section 5.1.

A slightly larger example shows why `*args, **kwargs` matter in practice:

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
# Processing order for a book listed at Rs. 500
# Final payable amount: Rs. 450.0
```

`wrapper(*args, **kwargs)` accepts whatever `process_book_order` accepts — here, just one positional argument, `price` — without `log_order` ever needing to know in advance how many arguments the function it wraps will take[1]. `args[0]` reaches into the collected positional arguments to read the original price for the log message, `func(*args, **kwargs)` forwards those same arguments to the real `process_book_order`, and `return result` makes sure the caller of `process_book_order(500)` still receives the correct discounted amount — `450.0` — rather than losing it inside the wrapper.

A minimal, self-contained example makes the mechanics of wrapping even clearer:

```python
def shout(func):
    def wrapper():
        result = func()
        return result.upper()
    return wrapper

@shout
def greet():
    return "hello"

print(greet())   # HELLO
```

Walking through this: `shout` is the decorator — it receives `greet` as `func`. Inside `shout`, `wrapper` is defined as a brand-new function that will completely replace `greet`. `@shout` above `def greet():` causes Python to immediately run `greet = shout(greet)`, so after this point, the name `greet` no longer refers to the original function at all — it refers to `wrapper`. When `print(greet())` runs, it is really calling `wrapper()`, which calls the *original* `greet()` internally (still reachable through the `func` parameter), gets back `"hello"`, converts it to uppercase, and returns `"HELLO"`[1].

The outer decorator function must always **return a callable** — normally the `wrapper` function itself, not a call to it (`return wrapper`, never `return wrapper()`). Returning anything else breaks whatever function is being decorated, since that return value is what the original function's name now points to[1].

When several decorators are stacked above one function, they apply from the **bottom up**: `@a` written above `@b` written above `def f():` means `f = a(b(f))` — `b` wraps `f` first, and `a` then wraps the *result* of that[1].

### 5.5 Closures

A **closure** is what makes a decorator's `wrapper` able to still "see" and use the original function (`func`) even after the outer decorator function has already finished running and returned[1]. Normally, a variable that exists only inside a function (its local scope, from Unit 2.3) disappears once that function returns. A closure is the exception: when an inner function (like `wrapper`) is defined inside an outer function and refers to one of the outer function's variables, Python keeps that variable alive and reachable for as long as the inner function itself still exists — even though the outer function's own execution is long over.

This is precisely why, in the `shout` example above, `wrapper` can still call `func()` correctly every single time `greet()` is called — `wrapper` remembers which specific function `func` referred to, permanently, because of the closure formed when `wrapper` was created inside `shout`.

Closures are not unique to decorators — any inner function that refers to a variable from its enclosing function forms one, whether or not `@` syntax is involved:

```python
def make_multiplier(factor):
    def multiply(number):
        return number * factor
    return multiply

double = make_multiplier(2)
triple = make_multiplier(3)

print(double(5))   # 10
print(triple(5))   # 15
```

`make_multiplier(2)` runs, creates `multiply`, and returns it — at that point `make_multiplier`'s own execution is completely finished, and normally `factor` (its local variable) would simply disappear. But `multiply` refers to `factor` inside its own body, so Python keeps that specific value of `factor` (here, `2`) alive and attached to the function object now stored in `double`. Calling `make_multiplier(3)` creates an entirely separate closure, remembering `factor = 3` for `triple`, completely independent of `double`'s. This is exactly the mechanism — one function remembering a value from the outer function that built it — that lets a decorator's `wrapper` keep reaching the original `func` long after the decorator itself has returned.

### 5.6 Generators and `yield`

A **generator function** is a function whose body contains at least one **`yield`** statement, instead of (or alongside) `return`[1]. It looks almost identical to a normal function definition — no special keyword marks it as a generator anywhere in its `def` line; Python decides purely based on whether `yield` appears anywhere in the body[1].

`yield` hands back one value and pauses the function's execution *at that exact point*, freezing every local variable's current value, until the next value is asked for[1]. Calling a generator function does **not** run its body immediately the way calling a regular function does — it instead returns a paused **generator object**[1]. The body only actually executes, one step at a time, as values are requested from that object — typically by a `for` loop, or by calling the built-in `next()` on it directly.

```python
def count_up_to(limit):
    current = 1
    while current <= limit:
        yield current
        current += 1

for number in count_up_to(5):
    print(number)
# 1
# 2
# 3
# 4
# 5
```

Calling `count_up_to(5)` does not run `current = 1` yet — it only builds a paused generator object. Each time the `for` loop asks for the next value, the function body runs forward from wherever it last paused: the very first request runs `current = 1`, enters the `while` loop, and pauses at `yield current`, handing back `1`. The *next* request resumes exactly after that `yield`, runs `current += 1`, loops back around, and pauses again at the next `yield`, handing back `2` — and so on, until `current` exceeds `limit` and the loop (and the generator) end naturally[1].

### 5.7 Lazy evaluation

**Lazy evaluation** is the general principle behind generators: producing a value only at the moment it is actually asked for, instead of computing and storing an entire sequence up front[1]. This is the opposite of what a regular function does: a function that `return`s a value must finish building that *entire* value — say, a full list — before it can hand any of it back. That is wasteful when a sequence has millions of entries and only the first few are ever needed, and it is outright impossible for a sequence that has no natural end at all, such as an endless stream of incrementing invoice numbers[1].

A generator object embodies laziness directly: at any given moment, only the single value currently paused at a `yield` exists — nothing before or after it has been computed yet. This is summarized well by comparing a list (built eagerly, all at once) against a generator (built lazily, one value at a time):

| Aspect | List | Generator |
|---|---|---|
| When values are produced | All at once, immediately | One at a time, only when requested |
| Memory usage | Holds every value at once | Holds only the current value |
| Can represent an infinite sequence? | No — would never finish building | Yes — values are produced on demand |
| Can be looped over more than once? | Yes, as many times as needed | No — exhausted after one full pass |
| Created with | `[...]` or `list(...)` | A function containing `yield` |

The last row matters in practice: a generator object can only be iterated **forward, once**[1]. Once a `for` loop has fully consumed it, looping over that exact same object again produces nothing at all — getting a fresh run requires calling the generator function again to create a brand-new generator object.

### 5.8 Quick distinctions worth remembering

A few points come up often enough, in practice and in beginner interviews, that they are worth stating explicitly and on their own:

- **"Why not always use lambda instead of `def`?"** A lambda is restricted to a single expression, cannot carry a docstring, and shows up as `<lambda>` in a traceback rather than its own name, which makes mistakes harder to trace back to their source[1]. `def` is the right default for anything beyond short, disposable, one-line logic.
- **The memory difference between a list and a generator** is the reason generators exist at all: a list holding a million computed values sits entirely in memory at once, while a generator never holds more than the one value currently paused at its `yield`[1]. This is the standard answer to "how would you process a huge file, or an endless stream, without running out of memory?"
- **`@decorator` is only syntax sugar** for `function = decorator(function)`[1] — being able to write out that equivalent by hand, rather than only recognizing the `@` symbol, is a reliable sign of actually understanding what a decorator does.
- **Stacked decorators apply bottom-up.** `@a` above `@b` above `def f():` means `f = a(b(f))` — the decorator closest to the `def` wraps the function first, and each decorator above it wraps the result of the one below[1].

## 6. Implementation

Decorators and generators can both feel abstract the first time they are read, because the code that defines them does not run in the same order it is written down. The two step-by-step breakdowns below make that ordering explicit, so that "what runs when" stops being a source of confusion.

**How Python executes a decorator, step by step:**

1. Python reads the decorator function's definition (for example, `log_order`) and the inner `wrapper` function defined inside it, but runs neither yet.
2. Python reads the target function's `def` block (for example, `process_book_order`).
3. Because `@log_order` sits directly above that `def`, Python immediately runs `process_book_order = log_order(process_book_order)`[1].
4. Running `log_order(process_book_order)` executes only the *outer* decorator function's body: it defines `wrapper` (forming a closure over the original `process_book_order`, now called `func` inside `log_order`) and returns `wrapper` itself.
5. The name `process_book_order` now refers to `wrapper`, not to the original function body.
6. Every future call to `process_book_order(...)` actually runs `wrapper(...)`, which performs its extra behavior, calls the real original function through the closure, and returns that function's real result.

**How Python executes a generator, step by step:**

1. Calling a generator function (for example, `count_up_to(5)`) does not execute any code inside it — it immediately returns a paused generator object[1].
2. When something (typically a `for` loop, or `next()`) requests the first value, the function body starts running from the top.
3. Execution runs until it reaches a `yield` statement, hands back that value, and pauses — freezing every local variable exactly as it was at that moment[1].
4. When the next value is requested, execution resumes immediately after the `yield` that paused it, and continues until the next `yield` (or until the function ends).
5. This repeats until either the function body finishes naturally (ending the generator), or the code consuming the generator stops asking for more values (as `zip()` does once its shorter input is exhausted, or as a `for` loop does after a `break`).

**Combining all three constructs — a worked walkthrough.** A small e-commerce script needs three things at once: sequential order IDs starting at `1001` with no natural end, timing information around order processing, and orders handled cheapest-first. Each need maps to exactly one construct from this unit: an ever-increasing sequence with no end is a generator; wrapper behavior (timing) applied without editing the original function is a decorator; a one-off ranking rule for `sorted()` is a lambda[1].

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

Reading this step by step: `order_id_generator(1001)` returns a paused generator object immediately — none of its body has run yet. `sorted_amounts` places `99.0` first and `1499.0` last, because the lambda's key is simply each amount itself, so `sorted()` ranks in plain ascending order. `zip()` then walks the generator and `sorted_amounts` together, pulling exactly four values from the generator — `1001, 1002, 1003, 1004` — one per amount in the four-item list, and stopping there even though `order_id_generator` would happily keep producing IDs forever if asked[1]. For each pair, `process_order` runs first (printing its own message as the last thing it does), and only once it returns does `wrapper` print the elapsed-time message — because that `print` sits *after* `result = func(*args, **kwargs)` inside `wrapper`'s body. That ordering is exactly what a decorator guarantees: the original function's behavior is untouched, and the extra behavior wraps cleanly around it, running before and/or after the real call[1].

## 7. Real-World Patterns

- **Banking and FinTech:** decorators wrap core banking functions to log every fund transfer, time slow database queries, or enforce authentication checks — all without touching the original transfer logic, so the same audit or security rule can be applied consistently across dozens of unrelated functions[1].
- **Payment and UPI-style systems:** generators produce transaction IDs or one-time codes one at a time, exactly as needed, instead of pre-computing a large batch that mostly goes unused — important when the total number of codes that will ever be needed is not known in advance[1].
- **E-commerce:** `sorted(order_amounts, key=lambda amount: amount)` ranks orders, products, or search results by price or relevance in a single line; decorators time how long checkout or payment steps take, which is often exactly the data needed to spot a slow step in a checkout flow[1].
- **Healthcare:** a generator can stream patient vitals readings from a monitoring device one reading at a time, rather than holding an entire day's readings in memory before processing any of them — the monitoring software only ever needs "the current reading," never the full day's history at once[1].
- **Railway and travel booking systems:** a generator can stream available seat numbers coach by coach as a user scrolls through options, and a lambda passed to `sorted()` can rank available trains by fare or by journey duration[1].
- **AI/ML pipelines:** generators are the standard way to feed a model training loop with batches of data from a dataset far too large to load into memory all at once — only the current batch is ever held in memory, no matter how large the full dataset is[1].
- **Cloud applications:** decorators are the standard mechanism behind logging, caching, retry logic, and access control in production-grade Python web frameworks, precisely because the same cross-cutting behavior needs to apply to many independent route-handling functions without duplicating code inside each one[1].

## 8. Best Practices

- Use a lambda only for short, throwaway logic passed directly into another function, such as the `key` argument of `sorted()` — if the logic needs more than one expression, a name, or a docstring, write a regular `def` function instead[1].
- Keep a decorator's `wrapper` generic by writing `*args, **kwargs` and passing them straight through to the original function, so the same decorator works on any function it is applied to[1].
- Always make `wrapper` `return` the original function's result — a decorator that forgets this makes the wrapped function silently return `None` to its caller, even though the "extra behavior" (logging, timing) appears to work fine[1].
- Prefer a generator over building a full list whenever a sequence could be very large, has no natural end, or is only ever going to be consumed once[1].
- Give generator functions clear, descriptive names (`read_large_file`, `generate_order_ids`) so it is obvious from the call site that values arrive lazily, one at a time[1].
- Remember that `map()` and `filter()` return lazy objects, not lists — wrap them in `list(...)` if you need to view or reuse the results more than once[1].
- If a generator is written with an infinite loop (`while True: yield ...`), make sure the code consuming it has its own stopping condition — such as `break`, or being paired with a shorter sequence via `zip()` — since the generator itself will never stop on its own[1].

**Common mistakes to watch for:**

- **Writing multiple statements inside a lambda** — a lambda supports exactly one expression; anything more raises a `SyntaxError`[1].
- **Forgetting `return result` inside a decorator's `wrapper`** — the wrapped function still appears to "work," because the extra behavior (timing, logging) runs fine, but the caller silently receives `None` instead of the real result[1].
- **Writing `wrapper(*args)` without `**kwargs`** — the decorator breaks the moment someone calls the decorated function using a keyword argument[1].
- **Assuming `map()` or `filter()` gives you a list directly** — both return a lazy object that must be wrapped in `list(...)` to view or reuse its contents[1].
- **Assuming a generator can be looped over twice** — once a `for` loop has fully consumed a generator object, looping over that same object again produces nothing; a fresh call to the generator function is required[1].

## 9. Hands-On Exercise

Using `book_prices = [500, 250, 1200, 90, 650]`:

1. Write a lambda `add_gst` that adds 10% to a price (`price * 1.10`, rounded to two decimals).
2. Use `map()` with `add_gst` to produce the GST-inclusive prices, and print the result as a list.
3. Use `filter()` to keep only prices at or above 300, then use `sorted(..., key=..., reverse=True)` to print them from most to least expensive.
4. Write a generator `invoice_number_generator(start)` that yields `f"INV{n}"` forever, starting from `start`, and use `next()` to print one invoice number before each price from step 3.

Check your work by asking three questions afterward: did your lambda contain exactly one expression with no `return`; does calling `invoice_number_generator(start)` on its own (with no `next()` or `for` loop) print nothing at all, confirming the body has not run yet; and does printing the same `map()` or `filter()` result a second time show anything, confirming whether you actually materialized it with `list(...)` or left it as a one-time-use lazy object.

## 10. Key Takeaways

- A function in Python is a first-class value — it can be stored in a variable, passed as an argument, or returned from another function — and this single fact is what makes lambdas, higher-order functions, decorators, and generators all possible.
- A lambda is a single-expression, unnamed function best used for short, one-off logic passed directly into another function, such as `sorted()`'s `key` argument; anything needing more than one line, a docstring, or reuse under a name belongs in a regular `def` function.
- A higher-order function like `map()`, `filter()`, or `sorted(key=...)` takes a function as an argument (or returns one) and applies it across a sequence; `map()` and `filter()` both return lazy objects that must be wrapped in `list(...)` to view immediately.
- A decorator is a function that takes another function and returns a wrapped replacement; `@decorator` is shorthand for `func = decorator(func)`, the wrapper must accept `*args, **kwargs` to work generically, and a closure is what lets the wrapper still reach the original function after the decorator itself has finished running.
- A generator function uses `yield` to produce values one at a time and pause in between, giving lazy evaluation that can represent sequences too large — or too infinite — to ever build as a complete list in memory; a generator object can only be iterated forward, once.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
