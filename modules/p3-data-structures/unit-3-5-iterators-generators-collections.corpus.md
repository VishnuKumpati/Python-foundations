---
topic_id: e982ff05-b296-4d16-b191-80078c70898a
title: Unit 3.5 - Iterators, Generators & Collections
position_in_module: 5
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 3.5 - Iterators, Generators & Collections — Topic Corpus

## 2. Prerequisites

This topic builds directly on:

- **Unit 2.2 - Loops** (id: `0ef5b1c5-43be-4f7a-8e00-ac4ab29451e5`) — you already know what a `for` loop is, what a loop body is, and what an "iterable/sequence" means informally. This unit explains the exact mechanism a `for` loop uses underneath.
- **Unit 2.4 - Functional Constructs** (id: `07a78e31-727b-4c0d-95aa-4e7712dceda2`) — you already met `generator function`, `yield`, `generator object`, and `lazy evaluation` there. This unit deepens those ideas by connecting them to the iterator protocol and showing exactly why they save memory.
- **Unit 3.1 - Lists** (id: `fe27250e-c2c2-491c-afd7-57d9b533deab`) and **Unit 3.4 - Dictionaries** (id: `3695f9f8-f7e3-4f7e-933d-95006fced9d1`) — you already know list/dict basics, `KeyError`, `.get()`, hashability, and comprehensions, all of which this unit's `collections` tools build on top of.
- **Unit 3.2 - Tuples** (id: `18ffde38-766c-4912-bd6a-bfd0e238a750`) — you already know tuple immutability, unpacking, and indexing, which `namedtuple` extends with readable field names.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** the difference between an iterable and an iterator, and describe the `iter()` / `next()` / `StopIteration` protocol that every `for` loop runs underneath.
- **Explain** why a generator is a special, one-shot kind of iterator, and why it uses less memory than building a full list up front.
- **Apply** `Counter` to tally occurrences of items in one line instead of writing a manual counting loop.
- **Apply** `defaultdict` to group or count items by key without writing manual key-existence checks.
- **Identify** when a `namedtuple` improves on a plain tuple for readability, and predict the outcome of the most common mistakes made with `Counter`, `defaultdict`, and `namedtuple`.

## 4. Introduction

Picture a food-delivery app called "Chennai Bites." Every time it prints a list of today's orders using a `for` loop, something is happening underneath that you have never had to name out loud: Python is quietly asking the list, "give me a helper that can hand you out one order at a time," then repeatedly asking that helper, "what's next?" until there is nothing left. That hidden helper is called an **iterator**, and the request-and-response pattern it follows is called the **iterator protocol**. Once you can see this mechanism directly, a `for` loop stops being "magic that walks a list" and becomes a small, predictable sequence of function calls — the same sequence of calls that works identically for a list, a tuple, a set, a dictionary, a string, or a generator.

This topic also finishes the story on **generators**, which you first saw in Unit 2.4 as "a function that uses `yield`." Here you will see exactly *why* a generator saves memory: instead of computing every value in advance and storing all of them, it computes one value only at the moment it is asked for, then pauses. This matters enormously once "all the values" could mean a multi-gigabyte log file, or a live stream of incoming transactions that has no defined end.

Finally, you will meet three ready-made tools from Python's `collections` module — `Counter`, `defaultdict`, and `namedtuple` — that quietly replace hand-written counting loops, error-prone key checks, and hard-to-read `tuple[0]`-style code with one clean line each. This is also the last topic in Module III (Data Structures): after this, the course moves on to organizing data and behavior together using classes and objects.

## 5. Core Concepts

### 5.1 Iterable vs. iterator

An **iterable** is any object you are allowed to place after `in` in a `for` loop — a list, tuple, set, dictionary, or string all qualify, and you already know all of these from Units 3.1–3.4. An **iterator** is a different, simpler kind of object: it does the actual walking. It remembers exactly one thing — its current position — and hands out the next value on request until there is nothing left to give [1].

The distinction matters because every iterator is iterable (you can put an iterator itself in a `for` loop), but not every iterable is an iterator. A list is iterable, but a list is *not* itself an iterator: it does not track "where you currently are" — that is why two separate `for` loops over the same list both start again from the beginning. Under the hood, every iterable object knows how to produce a fresh iterator when asked, and every iterator knows how to produce its next value when asked; you do not need to write this machinery yourself in this topic — it is built with class-based tools that are covered later, in Object-Oriented Foundations. For now, treat "asking an object for an iterator" and "asking an iterator for its next value" as two built-in operations you can trigger with two functions.

### 5.2 `iter()`, `next()`, and `StopIteration`

`iter(obj)` is a built-in function that takes an iterable and returns a fresh iterator positioned before the first value [1]. `next(it)` is a built-in function that takes an iterator and returns its next value, moving its position forward by one [1].

```python
orders = ["Saravana Bhavan", "Biryani Blues", "Pizza Hub"]
it = iter(orders)

print(next(it))   # Saravana Bhavan
print(next(it))   # Biryani Blues
print(next(it))   # Pizza Hub
print(next(it))   # raises StopIteration
```

The first three calls each return the next order in sequence. The fourth call has nothing left to return, so Python raises **`StopIteration`** — a special signal, not a bug and not `None`, meaning "there is nothing left to give" [1]. `StopIteration` is not an error you need to fix; it is the *expected*, normal way an iterator announces it has finished. Two rules follow directly from this: `next()` never rewinds — once an iterator has given out a value, it never gives that value again — and once an iterator is exhausted, every future call to `next()` on it raises `StopIteration` again; the iterator does not restart on its own [1]. To loop over the same values again, you must call `iter()` on the *original iterable* to get a brand-new, independent iterator; the iterable itself (the list) is reusable, but each individual iterator built from it is not.

### 5.3 What a `for` loop actually does

A `for` loop is syntax sugar for exactly the mechanism above. Writing:

```python
for order in orders:
    print("Processing:", order)
```

is equivalent, underneath, to:

```python
it = iter(orders)
while True:
    try:
        order = next(it)
    except StopIteration:
        break
    print("Processing:", order)
```

The `for` loop calls `iter()` once to get a fresh iterator, then calls `next()` repeatedly, and the moment `StopIteration` is raised, it stops silently — no error ever reaches your code [1]. This single explanation covers every `for` loop you have written in this course so far, regardless of whether the iterable was a list, tuple, set, dictionary, or string, because all of them honor the same `iter()`/`next()`/`StopIteration` contract. This uniform contract is also *why* the mechanism exists at all: without it, Python's `for` loop would need separate, special-cased logic for every different kind of collection.

### 5.4 Generators as a special kind of iterator

You already know from Unit 2.4 that a **generator function** is a function that uses `yield` instead of `return`, and that calling it produces a **generator object** rather than immediately running the function body. This topic adds one precise fact: a generator object *is* an iterator — it honors the exact same `iter()`/`next()`/`StopIteration` contract as everything else, which is why it can appear directly inside a `for` loop.

```python
def order_stream(order_list):
    for order in order_list:
        yield order

for order in order_stream(orders):
    print("Order received:", order)
```

Calling `order_stream(orders)` does not run the function body yet — it only creates a generator object. Each time the `for` loop calls `next()` on it (automatically), the function body runs until it reaches `yield order`, hands back that one value, and *freezes* — all of its local state (the loop variable, its current position) is preserved exactly where it paused — until the next `next()` call resumes it right after the `yield`. When there is no `yield` left to reach, the generator raises `StopIteration`, exactly like any other exhausted iterator.

This is precisely why a generator is called a **lazy** iterator (recall "lazy evaluation" from Unit 2.4): it computes a value only at the moment it is requested, instead of computing every value in advance and holding them all in memory. Because a generator is a special case of an iterator, everything from section 5.2 applies to it: it is one-shot, and looping over an already-exhausted generator object a second time produces no output and no error — it simply has nothing left to yield.

```python
def big_orders(order_list, minimum):
    for o in order_list:
        if o.amount >= minimum:
            yield o.customer

top_spenders = big_orders(orders_today, 400)
for customer in top_spenders:
    print(customer)      # prints the qualifying customers

for customer in top_spenders:
    print(customer)      # prints nothing — top_spenders is already exhausted
```

To get the values again, you must either call `big_orders(orders_today, 400)` a fresh time to build a brand-new generator object, or materialize the results once into a list with `list(...)` if you know you will need to revisit them.

### 5.5 Generator expressions

A **generator expression** is a compact, one-line way to write a generator, shaped exactly like the list comprehensions you already know from Unit 3.1, but written with `()` instead of `[]`:

```python
all_amounts_list = [n * 10 for n in range(1_000_000)]   # list comprehension
all_amounts_gen  = (n * 10 for n in range(1_000_000))   # generator expression
```

The list comprehension builds and stores all one million values immediately. The generator expression stores only *the plan* for producing each value, computing one only when asked. Measuring memory with `sys.getsizeof()` (a function that reports how many bytes an object itself occupies) shows the list costing roughly 8 megabytes before a single value has been used, while the generator expression costs only a couple hundred bytes — and that stays true whether it is asked to eventually produce a thousand values or a billion, because it stores a plan, not the values [1]. (The exact byte counts can shift slightly between Python versions; the point that matters is that the generator's footprint stays tiny and constant.)

### 5.6 `collections.Counter`

The **`collections` module** is part of Python's standard library — the built-in collection of modules that ships with every Python installation (recall "standard library" from Unit 2.5) — and it provides ready-to-use, purpose-built versions of `dict` and `tuple` for patterns that come up constantly in real programs [1].

**`Counter`** is a specialized version of a dictionary that tallies how many times each hashable item appears in an iterable, in one call [1]:

```python
from collections import Counter

restaurant_counts = Counter(o.restaurant for o in orders_today)
print(restaurant_counts)
# Counter({'Saravana Bhavan': 3, 'Biryani Blues': 2, 'Pizza Hub': 1})
```

Because it behaves like a dictionary in every other way, looking up a key that never appeared returns `0` rather than raising a `KeyError` — a small but important difference from a plain dictionary. **`most_common(n)`** is a `Counter` method that returns the `n` most frequent items, already sorted from highest count to lowest [1]:

```python
print(restaurant_counts.most_common(1))
# [('Saravana Bhavan', 3)]
```

### 5.7 `collections.defaultdict`

**`defaultdict`** is a specialized version of a dictionary that auto-creates a default value for a brand-new key instead of raising `KeyError` [1]. It requires a **factory function** — a callable that takes no arguments, most commonly `list` or `int` — supplied when the `defaultdict` is created; that factory is called automatically, behind the scenes, the very first time a new key is used [1]:

```python
from collections import defaultdict

customers_by_restaurant = defaultdict(list)
for o in orders_today:
    customers_by_restaurant[o.restaurant].append(o.customer)

print(dict(customers_by_restaurant))
# {'Saravana Bhavan': ['Priya', 'Arjun', 'Kavya'], 'Biryani Blues': ['Rohan', 'Sara'], 'Pizza Hub': ['Meera']}
```

The first time `"Saravana Bhavan"` is used as a key, `defaultdict(list)` calls `list()` automatically to create an empty list for it, so `.append(...)` works immediately, with no `if key not in dict:` check written anywhere. `defaultdict(int)` follows the same idea for counting: the factory `int()` produces `0` for a brand-new key, so `counts[key] += 1` works on the very first use of any key.

### 5.8 `collections.namedtuple`

**`namedtuple`** is a function that builds a specialized version of a tuple whose positions also have readable field names [1]:

```python
from collections import namedtuple

Order = namedtuple("Order", ["customer", "restaurant", "amount"])
o = Order("Priya", "Saravana Bhavan", 350)

print(o.customer)     # "Priya"  — named access
print(o[0])           # "Priya"  — still works, because it is still a tuple
name, restaurant, amount = o   # still unpacks, exactly like an ordinary tuple
```

A `namedtuple` object remains a true tuple: it is **immutable** (you cannot reassign a field after creation, recall immutability from Unit 3.2), it supports unpacking, and it supports plain positional indexing in addition to named access. It exists to remove the guesswork of "position 0 is the name, position 1 is the amount" for every future reader of the code.

## 6. Implementation

The iterator protocol, spelled out as the steps a `for` loop always performs, regardless of what kind of iterable it is given:

1. Call `iter(iterable)` once, producing a fresh iterator positioned before the first value.
2. Call `next(iterator)`.
3. If a value is returned, run the loop body with that value, then go back to step 2.
4. If `StopIteration` is raised instead, stop the loop silently — no exception is shown to the code that wrote the `for` loop.

A generator function follows the same four steps from the *caller's* side, but produces its values differently on the inside:

1. Calling the generator function does not run its body — it returns a generator object immediately.
2. Each `next()` call on that generator object runs the function body starting from where it last paused (or from the top, on the first call), until it reaches a `yield`.
3. The `yield`ed value is handed back, and the function's entire local state is frozen in place.
4. When the function body finishes running with no more `yield` to reach, `StopIteration` is raised, exactly as with any other iterator.

## 7. Real-World Patterns

- **Banking and FinTech**: A bank statement generator that streams thousands of transaction rows one at a time, instead of loading an entire year's history into memory at once, is a generator doing exactly what section 5.4's `order_stream` does — producing the next value only when asked [1].
- **UPI / payment systems**: A dashboard tile showing "most-used payment method today" is `Counter` tallying every transaction's method in a single pass, then reading the top result with `most_common()` [1].
- **E-commerce**: Grouping today's orders by delivery city, where a brand-new city key can appear at any moment, is exactly what `defaultdict(list)` is built for — no `KeyError`, no manual existence checks [1].
- **Healthcare**: A patient vitals reading (heart rate, temperature, timestamp) passed around a monitoring script is a natural fit for `namedtuple`, so the code reads `reading.heart_rate` instead of decoding what position `1` was supposed to mean [1].
- **AI/ML pipelines**: Reading a multi-gigabyte training dataset or log file one line at a time uses a generator so the program never needs the whole file in memory at once — this is a core reason generators exist in the language at all, and it is exactly the pattern every later AI lab in this course will use when it processes prompts, model outputs, or evaluation scores in bulk [1].

## 8. Best Practices

- Reach for **`Counter`** the moment you catch yourself writing a manual counting loop with a dictionary — it is the same result in one line, with `most_common()` built in.
- Reach for **`defaultdict(list)`** whenever you are grouping items under keys, and **`defaultdict(int)`** whenever you are counting by key — both remove a manual key-existence check every single time.
- Prefer a **generator expression** over a list comprehension whenever you only need to walk the values once and the dataset could be large — you save memory with no change to how the loop reads.
- Reach for **`namedtuple`** the moment a plain tuple's meaning depends on remembering which position holds which piece of data.
- Do not treat `StopIteration` as a bug to catch and suppress everywhere — it is the normal termination signal that a `for` loop already handles for you silently.
- Do not assume `next()` on an exhausted iterator will restart it, and do not assume a generator can be looped over twice like a list — both assumptions are wrong, and both fail silently (no output, no error) rather than crashing loudly.
- If you genuinely need to reuse a generator's values more than once, materialize them explicitly with `list(...)` the first time, or call the generator function again to get a fresh generator object.
- Do not create a `defaultdict` without a factory function (plain `defaultdict()` behaves like an ordinary `dict` and still raises `KeyError`), and do not pass a non-callable value like `0` instead of `int` — the factory must be something Python can call with no arguments.
- Do not attempt to reassign a `namedtuple` field like a list element (`student.marks = 90`) — like any tuple, it is immutable, and this raises an `AttributeError` (Python's error signal for "this kind of object does not support what you just tried to do to it").

## 9. Hands-On Exercise

Given `logins = ["alice", "bob", "alice", "carol", "bob", "alice"]`:

1. Build an iterator with `iter(logins)` and call `next()` on it four times, predicting the fourth result before running it.
2. Use `Counter` to find how many times each user logged in, and print the single most frequent user with `most_common(1)`.
3. Write a generator function `repeat_logins(names)` that yields only the names that appear more than once (hint: combine it with the `Counter` from step 2), then walk it with a `for` loop twice in a row and observe what the second loop prints.

## 10. Key Takeaways

- An **iterable** is anything loopable; an **iterator** is the object that hands out one value at a time from it, via `iter()` and `next()`, until it raises `StopIteration`.
- A `for` loop is exactly this protocol in disguise — it calls `iter()` once, then `next()` repeatedly, stopping silently the instant `StopIteration` is raised.
- A **generator** is a special, lazy iterator: it computes each value only when asked instead of storing all of them, which is why it can use dramatically less memory than the equivalent list, and, like any iterator, it can only be walked through once.
- **`Counter`** tallies occurrences in one line and ranks them with `most_common()`; **`defaultdict`** removes manual key-existence checks by auto-creating a default value (`list` for grouping, `int` for counting) the first time a new key is used.
- **`namedtuple`** adds readable, named access to a tuple's positions while keeping it fully immutable, unpackable, and indexable.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
