# Iterators, Generators & Collections

Units 3.1 through 3.4 covered the four core collections — list, tuple, set, dict. Every one of them can be looped over with a `for` loop, which raises a question worth answering properly: what is actually happening, mechanically, when `for` pulls one item at a time out of any of them? You have typed `for order in orders:` dozens of times by now without ever needing to ask, and that is exactly the sign of a good abstraction — it works so smoothly that the machinery underneath stays invisible. But invisible is not the same as unimportant, and the moment you write your own class in Module IV, or debug a loop that behaves unexpectedly, that hidden machinery stops being optional knowledge.

This unit pulls back the curtain on exactly that machinery: the difference between an **iterable** (anything you can loop over) and an **iterator** (the object that actually does the looping, one step at a time), and the small, strict handshake — `iter()`, `next()`, `StopIteration` — that every single `for` loop performs underneath, no matter which of the four collections it is walking. You already met generators in Unit 2.4 as "a function that uses `yield`"; here you will see the one precise fact that finishes that story — a generator *is* an iterator, which is exactly why it slots into a `for` loop and exactly why it saves memory. From there, this unit closes out Module III by introducing three ready-made tools from Python's `collections` module — `Counter`, `defaultdict`, and `namedtuple` — that quietly replace hand-written counting loops, fragile key checks, and hard-to-read `tuple[0]`-style code with one clean line apiece.

By the time you finish this chapter, you will never look at a `for` loop the same way again — and, since this is the last unit of Module III, you will also have every core data structure this course covers sitting in one connected mental picture: list, tuple, set, dict, and now the mechanism that loops over all of them, plus the toolkit built on top.

---

## What a `for` loop is quietly doing for you

Picture a food-delivery platform — call it Chennai Bites — printing today's orders with a simple loop:

```python
orders = ["Saravana Bhavan", "Biryani Blues", "Pizza Hub"]

for restaurant in orders:
    print("Processing:", restaurant)
```

```
Processing: Saravana Bhavan
Processing: Biryani Blues
Processing: Pizza Hub
```

Nothing about that looks mysterious — until you ask exactly *how* Python knows which item comes "next," or how it knows to stop after the third one instead of trying a fourth. The answer is that Python is not asking the list directly for each item. It is asking the list for a *helper* that knows how to hand out items one at a time, and then asking that helper, repeatedly, "what's next?" That helper is called an **iterator**, and the object it was built from — the list itself — is called an **iterable**.

An **iterable** is any object you are allowed to place after `in` in a `for` loop: a list, tuple, set, dictionary, or string all qualify — every collection from Units 3.1 through 3.4. An **iterator** is a different, simpler kind of object: it remembers exactly one thing, its current position, and hands out the next value on request until there is nothing left to give.

Think of an iterable as a whole book sitting on a shelf, and an iterator as a bookmark placed inside a *copy* of that book. The book itself does not remember what page you're on — the bookmark does. Hand the same book to two different readers, and each one gets their own bookmark, starting fresh from page one, completely unaware of where the other reader has gotten to. That is exactly why two separate `for` loops over the same list both start again from the beginning: a list is iterable, but a list is *not* itself an iterator — it does not track "where you currently are." Every time a `for` loop begins, it asks the list for a brand-new bookmark.

**Every iterator is iterable — you can loop over an iterator itself — but not every iterable is an iterator.** A list can produce a bookmark; it isn't one.

## Getting the bookmark yourself: `iter()` and `next()`

You don't have to wait for a `for` loop to see this mechanism — two built-in functions let you trigger it by hand. **`iter(obj)`** takes an iterable and returns a fresh iterator, positioned before the first value. **`next(it)`** takes an iterator and returns its next value, moving the position forward by exactly one.

```python
orders = ["Saravana Bhavan", "Biryani Blues", "Pizza Hub"]
it = iter(orders)

print(next(it))
print(next(it))
print(next(it))
```

```
Saravana Bhavan
Biryani Blues
Pizza Hub
```

Before reading on, predict what a fourth `print(next(it))` would do here. There is nothing left in `it` to hand back — every value has already been given out — so Python raises **`StopIteration`**. That is not a bug, and it is not `None`; it is a special signal meaning "there is nothing left to give," and it is the *expected*, normal way an iterator announces it has finished:

```python
print(next(it))
```

```
StopIteration
```

**Once an iterator has given out its last value, it does not restart on its own — every further call to `next()` on that same iterator raises `StopIteration` again, forever.** To walk the same values a second time, you cannot reuse `it`; you must call `iter(orders)` again, on the *original iterable*, to build a completely new, independent bookmark. The iterable is reusable. Each individual iterator built from it is not.

## What a `for` loop actually does, spelled out

With `iter()`, `next()`, and `StopIteration` in hand, you can now write out, by hand, exactly what `for order in orders:` does underneath. This code:

```python
for order in orders:
    print("Processing:", order)
```

is equivalent to this, with nothing hidden:

```python
it = iter(orders)
while True:
    try:
        order = next(it)
    except StopIteration:
        break
    print("Processing:", order)
```

The `for` loop calls `iter()` exactly once to get a fresh bookmark, then calls `next()` repeatedly, running the loop body each time a value comes back — and the moment `StopIteration` is raised, it stops silently, without that exception ever reaching your own code. This single explanation covers every `for` loop you have written so far in this course, whether the iterable was a list, tuple, set, dictionary, or string, because all of them honour the same `iter()`/`next()`/`StopIteration` contract. Say out loud, in one sentence, why this contract is useful to Python itself: because every iterable agrees to the same handshake, a `for` loop needs exactly one implementation, not a separate special case for each of the four collections.

| | Iterable | Iterator |
|---|---|---|
| What it is | Anything you can put after `in` in a `for` loop | The object that actually tracks position and hands out values |
| Remembers position? | No | Yes — exactly one position, moving forward only |
| Created with | Already exists as a list, tuple, set, dict, string | `iter(iterable)` |
| Advances with | (a `for` loop calls this for you) | `next(iterator)` |
| Reusable across multiple loops? | Yes | No — build a fresh one with `iter()` each time |
| Signals "no more values" with | — | Raising `StopIteration` |

## Generators: the iterator you already met

Unit 2.4 introduced the **generator function** — a function whose body contains `yield` instead of, or alongside, `return` — and showed that calling one does not run its body immediately; it produces a paused **generator object**. This unit adds exactly one precise fact that completes that story: a generator object *is* an iterator. It honours the same `iter()`/`next()`/`StopIteration` contract as a list's bookmark, which is exactly why a generator can sit directly inside a `for` loop with no extra work.

```python
def order_stream(order_list):
    for order in order_list:
        yield order

for order in order_stream(orders):
    print("Order received:", order)
```

```
Order received: Saravana Bhavan
Order received: Biryani Blues
Order received: Pizza Hub
```

Every time the `for` loop calls `next()` on this generator (automatically, behind the scenes), the function body runs forward until it reaches `yield order`, hands back one value, and *freezes* — every local variable is preserved exactly where it paused — until the next `next()` call resumes it right after that `yield`. When there is no `yield` left to reach, the generator raises `StopIteration`, exactly like any exhausted iterator built from a list.

Because a generator is a special case of an iterator, everything from the last two sections applies to it directly: **a generator is one-shot — loop over it fully once, and looping over the very same generator object again produces nothing at all, silently, with no error.**

```python
def big_orders(order_list, minimum):
    for amount, customer in order_list:
        if amount >= minimum:
            yield customer

orders_today = [(799, "Priya"), (250, "Arjun"), (1499, "Rohan")]
top_spenders = big_orders(orders_today, 400)

for customer in top_spenders:
    print(customer)

for customer in top_spenders:
    print(customer)
```

```
Priya
Rohan
```

The second loop prints nothing whatsoever — not an error, not a blank line, nothing — because `top_spenders` was already walked to exhaustion by the first loop. To get the values again, you must either call `big_orders(orders_today, 400)` a fresh time to build a brand-new generator object, or, if you know you'll need the results more than once, materialise them into a list immediately with `list(...)`.

A **generator expression** gives you the same one-shot, lazy machinery in a single line, shaped exactly like the list comprehensions from Unit 3.1, but written with `()` instead of `[]`:

```python
all_amounts_list = [n * 10 for n in range(1_000_000)]   # builds and stores a million values now
all_amounts_gen  = (n * 10 for n in range(1_000_000))   # stores only the plan
```

The list comprehension computes and stores every one of the million values immediately, occupying several megabytes before a single one has even been used. The generator expression stores only the *plan* for producing each value, and computes one only the instant something asks for it — its memory footprint stays a few hundred bytes whether it is eventually asked for a thousand values or a billion, because it was never holding "all of them" to begin with.

| Aspect | List comprehension | Generator expression |
|---|---|---|
| When values are produced | All at once, immediately | One at a time, only when asked |
| Memory used | Grows with the number of items | Stays small and constant |
| Can represent an endless sequence? | No | Yes |
| Looped over more than once? | Yes, as many times as needed | No — exhausted after one full pass |

A bank statement feature that streams a year's worth of transactions to your screen one row at a time, instead of loading the entire year into memory before showing you row one, is exactly this trade-off in production: a generator, doing precisely what `order_stream` does above, is what keeps that feature fast and light regardless of how many transactions exist.

## Introducing the `collections` module

Python ships with a standard library — the built-in collection of modules that comes with every Python installation, which you first met in Unit 2.5 — and tucked inside it is a module literally named **`collections`**. It provides ready-made, purpose-built variants of `dict` and `tuple` for patterns that come up constantly once you start writing real programs: tallying occurrences, grouping by key, and giving a tuple's positions readable names. Each of the next three tools replaces a small, easy-to-get-wrong pattern you would otherwise write by hand with one import and one line.

## `Counter`: tallying without writing a counting loop

Imagine building the exact same "most-used restaurant today" tile that a UPI-style app might show for "most-used payment method today." Without any new tools, you'd reach for a plain dictionary and a manual counting loop — checking whether a key already exists before you can safely add one to its count. `Counter` removes that entire pattern:

```python
from collections import Counter

restaurant_orders = ["Saravana Bhavan", "Biryani Blues", "Saravana Bhavan",
                      "Pizza Hub", "Saravana Bhavan", "Biryani Blues"]

restaurant_counts = Counter(restaurant_orders)
print(restaurant_counts)
```

```
Counter({'Saravana Bhavan': 3, 'Biryani Blues': 2, 'Pizza Hub': 1})
```

A `Counter` is a specialised dictionary — one key per distinct item, one value per count — built from an entire iterable in a single call. Because it still behaves like a dictionary in every other respect, you can look up any key you like, but with one convenient difference from a plain `dict`: **looking up a key that never appeared returns `0` instead of raising a `KeyError`.**

```python
print(restaurant_counts["Domino's"])
```

```
0
```

The method **`most_common(n)`** returns the `n` most frequent items, already sorted from the highest count down:

```python
print(restaurant_counts.most_common(1))
```

```
[('Saravana Bhavan', 3)]
```

Before checking, predict what `restaurant_counts.most_common(2)` would return, purely by reading the counts already printed above — it's the same tie-breaking-free ranking, just two entries deep instead of one.

## `defaultdict`: no more manual key checks

Now imagine grouping today's Chennai Bites customers by which restaurant they ordered from — the exact kind of task an e-commerce platform faces constantly when grouping orders by delivery city, where a brand-new city can appear at any moment with no warning. A plain dictionary forces you to check, every single time, whether a key already exists before you can append to its list. **`defaultdict`** removes that check entirely, by auto-creating a default value for a brand-new key instead of raising `KeyError`.

```python
from collections import defaultdict

orders = [("Saravana Bhavan", "Priya"), ("Biryani Blues", "Rohan"),
          ("Saravana Bhavan", "Arjun"), ("Pizza Hub", "Meera"),
          ("Saravana Bhavan", "Kavya")]

customers_by_restaurant = defaultdict(list)
for restaurant, customer in orders:
    customers_by_restaurant[restaurant].append(customer)

print(dict(customers_by_restaurant))
```

```
{'Saravana Bhavan': ['Priya', 'Arjun', 'Kavya'], 'Biryani Blues': ['Rohan'], 'Pizza Hub': ['Meera']}
```

`defaultdict` requires a **factory function** — a callable that takes no arguments, most often `list` or `int` — supplied the moment the `defaultdict` is created. That factory runs automatically, behind the scenes, the very first time a brand-new key is used. Here, the first time `"Saravana Bhavan"` appears as a key, `defaultdict(list)` calls `list()` to build an empty list for it on the spot, so `.append(...)` works immediately with no `if restaurant not in customers_by_restaurant:` check written anywhere.

`defaultdict(int)` follows the identical idea for counting: `int()` produces `0` for a brand-new key, so `counts[key] += 1` works correctly even on a key's very first appearance.

```python
order_totals = defaultdict(int)
for restaurant, _ in orders:
    order_totals[restaurant] += 1

print(dict(order_totals))
```

```
{'Saravana Bhavan': 3, 'Biryani Blues': 1, 'Pizza Hub': 1}
```

**A `defaultdict` created without a factory function — plain `defaultdict()` — behaves exactly like an ordinary `dict` and still raises `KeyError` on a missing key.** The factory argument is what does all the work; leaving it out silently gives up the one feature you reached for `defaultdict` to get.

## `namedtuple`: a tuple with named seats

Recall from Unit 3.2 that a plain tuple is immutable, unpackable, and indexed by position — but that position-only access has a real cost. `reading[0]`, `reading[1]`, `reading[2]` tells you nothing about what those slots actually hold, unless the reader has memorised the order. A healthcare monitoring script passing around a patient's vitals — heart rate, temperature, timestamp — is a natural place to feel this pain, and it's exactly what **`namedtuple`** exists to fix.

```python
from collections import namedtuple

VitalsReading = namedtuple("VitalsReading", ["heart_rate", "temperature", "timestamp"])
reading = VitalsReading(78, 98.6, "09:15")

print(reading.heart_rate)
print(reading[0])
heart_rate, temperature, timestamp = reading
print(heart_rate, temperature, timestamp)
```

```
78
78
78 98.6 09:15
```

`namedtuple` is a function that builds a brand-new tuple-like type, whose positions also carry readable field names. `reading.heart_rate` is immediately clearer to any future reader than `reading[0]` — but `reading[0]` still works, because a `namedtuple` object remains a genuine tuple underneath: it still unpacks, it still supports positional indexing, and it is still fully **immutable**.

```python
reading.heart_rate = 82
```

```
AttributeError: can't set attribute
```

**Attempting to reassign a field on a `namedtuple`, exactly like attempting to reassign an element of an ordinary tuple, raises an `AttributeError` — Python's signal that this kind of object simply does not support what you just tried to do to it.** If a value genuinely needs to change after creation, a `namedtuple` is the wrong tool; a small class, which you'll meet properly in Module IV, is the right one.

## Putting the toolkit together

A single Chennai Bites script can lean on all three tools at once, plus everything from earlier in this unit, without writing a single manual counting loop or key check:

```python
from collections import Counter, defaultdict, namedtuple

Order = namedtuple("Order", ["customer", "restaurant", "amount"])

todays_orders = [
    Order("Priya", "Saravana Bhavan", 350),
    Order("Rohan", "Biryani Blues", 620),
    Order("Arjun", "Saravana Bhavan", 210),
    Order("Meera", "Pizza Hub", 480),
]

restaurant_counts = Counter(order.restaurant for order in todays_orders)
customers_by_restaurant = defaultdict(list)
for order in todays_orders:
    customers_by_restaurant[order.restaurant].append(order.customer)

print(restaurant_counts.most_common(1))
print(dict(customers_by_restaurant))
```

```
[('Saravana Bhavan', 2)]
{'Saravana Bhavan': ['Priya', 'Arjun'], 'Biryani Blues': ['Rohan'], 'Pizza Hub': ['Meera']}
```

Notice the first line: `Counter(order.restaurant for order in todays_orders)` is being handed a generator expression directly, not a list — `Counter` happily consumes any iterable, walking it exactly once via the same `iter()`/`next()` protocol this entire unit has been building towards. Every idea in this chapter — iterable, iterator, generator, and the `collections` toolkit — is quietly working together in these four lines.

A short list of mistakes worth watching for deliberately while these tools are still new:

- Assuming `next()` on an exhausted iterator will restart it, or assuming a generator can be looped over twice like a list — both fail silently, producing nothing, rather than crashing loudly.
- Treating `StopIteration` as a bug to catch and suppress everywhere in your own code — it is the normal termination signal a `for` loop already handles for you.
- Creating a `defaultdict` with no factory function, or passing a non-callable value such as `0` instead of `int` — the factory must be something Python can call with zero arguments.
- Trying to reassign a `namedtuple` field like a list element, forgetting it is still a fully immutable tuple underneath.
- Reaching for a list comprehension out of habit when a dataset is large and will only be walked once — a generator expression gives the same result with a fraction of the memory.

## Try it yourself

Do this in a Colab cell before moving on. Given `logins = ["alice", "bob", "alice", "carol", "bob", "alice"]`: build an iterator with `iter(logins)` and call `next()` on it four times, predicting the fourth result before you run it. Then use `Counter(logins)` to find how many times each user logged in, and print the single most frequent user with `most_common(1)`. Finally, write a generator function `repeat_logins(names)` that yields only the names appearing more than once (combine it with the `Counter` from the previous step), walk it once with a `for` loop to see the repeat users, then walk the *same* generator object with a second `for` loop and confirm for yourself that it prints nothing at all.

---

### Key Terminology

- **Iterable** — any object you can place after `in` in a `for` loop, such as a list, tuple, set, dict, or string.
- **Iterator** — the object that tracks a current position and hands out the next value on request, until nothing is left.
- **`iter(obj)`** — the built-in function that returns a fresh iterator from an iterable.
- **`next(it)`** — the built-in function that returns an iterator's next value and advances its position by one.
- **`StopIteration`** — the signal an iterator raises when it has no more values to give; the normal, expected end-of-iteration signal, not an error to fix.
- **Generator object** — the paused, resumable object a generator function returns; a special case of an iterator.
- **Generator expression** — a one-line, lazy equivalent of a list comprehension, written with `()` instead of `[]`.
- **`collections` module** — the standard-library module providing purpose-built variants of `dict` and `tuple` for common patterns.
- **`Counter`** — a specialised dictionary that tallies occurrences of items and ranks them with `most_common(n)`.
- **`defaultdict`** — a specialised dictionary that auto-creates a default value for a brand-new key using a supplied factory function.
- **Factory function** — a callable taking no arguments, supplied to `defaultdict`, that produces the default value for a new key.
- **`namedtuple`** — a function that builds a tuple-like type whose positions also have readable field names, while remaining immutable, unpackable, and indexable.

### Mastery Checkpoint

Before moving to Unit 4.1, check that you can answer these without looking back:

1. What is the difference between an iterable and an iterator, and why can a list produce an iterator without being one itself?
2. Write out, from memory, the four-step loop of `iter()`, `next()`, and `StopIteration` that a `for` loop performs underneath.
3. Why is a generator object considered a special case of an iterator, and what happens if you try to loop over an already-exhausted generator a second time?
4. What single fact about `defaultdict` removes the need for a manual `if key not in dict:` check, and what goes wrong if you create a `defaultdict` with no factory function at all?
5. What can you do with a `namedtuple` that you cannot do with a plain dictionary, and what happens if you try to reassign one of its fields?

### Summary

You now know exactly what a `for` loop has been doing underneath every time you've written one: asking an iterable for a fresh iterator with `iter()`, pulling values one at a time with `next()`, and stopping silently the moment `StopIteration` is raised. You've confirmed that a generator, first met in Unit 2.4, is really just a special, lazy, one-shot iterator that computes each value only when asked — and you've added `Counter`, `defaultdict`, and `namedtuple` to your toolkit, three small `collections`-module tools that replace hand-written counting loops, fragile key checks, and unreadable positional tuples with a single clean line each. This closes out Module III: between Units 3.1 and 3.5 you have now covered every core Python data structure — list, tuple, set, dict — plus the iteration mechanism underneath all of them and the standard-library tools built on top. From here, Module IV shifts from organising *data* to organising *data and behaviour together*, starting with Unit 4.1, Object-Oriented Foundations.

### Additional Resources

- [Python Tutorial — official docs: "Iterators"](https://docs.python.org/3/tutorial/classes.html#iterators)
- [Python Tutorial — official docs: "Generators"](https://docs.python.org/3/tutorial/classes.html#generators)
- [Python 3 Documentation — Built-in Functions: `iter()`](https://docs.python.org/3/library/functions.html#iter)
- [Python 3 Documentation — Built-in Functions: `next()`](https://docs.python.org/3/library/functions.html#next)
- [Python 3 Documentation — `collections` module](https://docs.python.org/3/library/collections.html)
- [Python 3 Documentation — `collections.Counter`](https://docs.python.org/3/library/collections.html#collections.Counter)
- [W3Schools — Python Iterators](https://www.w3schools.com/python/python_iterators.asp)
