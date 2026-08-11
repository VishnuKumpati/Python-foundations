# Iterators, Generators & Collections

---

## Why This Unit?

You've been looping over strings, lists, dictionaries, and sets, all through the same `for` loop, without ever asking what makes that possible. This unit answers that, and then hands you three ready-made tools that solve extremely common problems in a single line each, instead of writing the pattern yourself every time.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Iterable** | Anything you can loop over, a string, list, dict, set, range |
| **Iterator** | The object that actually does the looping, remembers position, produced by `iter()` |
| **`next()`** | Gets the next value from an iterator |
| **`StopIteration`** | The error Python raises internally when an iterator runs out, `for` catches it for you |
| **`Counter`** | A dictionary built specifically for counting things |
| **`defaultdict`** | A dictionary that auto-creates a default value for missing keys |
| **`namedtuple`** | A tuple whose fields can be accessed by name, not just position |

---

## Iterables vs Iterators

**Every iterable, a string, a list, a dictionary, has been quietly hiding one step** every time you wrote a `for` loop. Underneath, Python calls `iter()` on it to get an **iterator**, then calls `next()` on that iterator, over and over, until there's nothing left.

You can do this by hand, to see it happen:

```python
fruits = ["apple", "banana", "cherry"]
fruit_iterator = iter(fruits)

print(next(fruit_iterator))   # apple
print(next(fruit_iterator))   # banana
print(next(fruit_iterator))   # cherry
print(next(fruit_iterator))   # crashes here, raises StopIteration
```

`iter(fruits)` produces the iterator, a separate object that remembers exactly how far through the list it's gotten. Each `next()` call moves it forward by one and hands back the value. Once it runs out, that 4th call doesn't print anything, it actually **crashes the program** with a `StopIteration` error, unless something catches it.

**That "unless something catches it" is exactly what a `for` loop does for you, silently, every single time.** You've been triggering `StopIteration` this entire course, every loop that ever finished hit it internally, `for` just catches it automatically and stops cleanly instead of crashing. That's the whole trick this section is revealing.

**The distinction, in one line:** the list itself is the **iterable**, the thing you can loop over. `fruit_iterator` is the **iterator**, the thing actually doing the looping, one step at a time.

---

## Generators, a Recap, and the Connection

You already met generators, functions that use `yield` instead of `return`, pausing between values instead of computing everything at once.

```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i
        i += 1
```

**Here's the piece that ties this whole unit together: a generator *is* an iterator.** Calling `count_up_to(5)` doesn't run the function, it produces an object that already knows how to respond to `next()`, exactly like `fruit_iterator` above:

```python
gen = count_up_to(3)
print(next(gen))   # 1
print(next(gen))   # 2
print(next(gen))   # 3
print(next(gen))   # crashes here, raises StopIteration, same as the list iterator above
```

That's not a coincidence, or a separate feature, `yield` is Python's built-in way of writing an iterator without manually managing the "remember where I am" bookkeeping yourself. The `for` loop you used to loop over a generator was doing the exact same `iter()`/`next()` dance the whole time.

---

## The `collections` Module

Python's standard library includes a module called `collections`, with a handful of specialized versions of dictionaries and tuples, that solve common, repetitive patterns in one line.

### `Counter`

**Remember counting word frequency with `.setdefault()`?**

```python
word_counts = {}
words = ["apple", "banana", "apple", "cherry", "apple"]

for word in words:
    word_counts.setdefault(word, 0)
    word_counts[word] += 1
```

`Counter` does this entire pattern for you, in one line:

```python
from collections import Counter

words = ["apple", "banana", "apple", "cherry", "apple"]
word_counts = Counter(words)
print(word_counts)
```
```
Counter({'apple': 3, 'banana': 1, 'cherry': 1})
```

A `Counter` behaves like a dictionary, you can still index it, loop over it, and use `.items()`, plus a couple of counting-specific extras:

```python
print(word_counts["apple"])          # 3
print(word_counts.most_common(2))     # [('apple', 3), ('banana', 1)]
```

### `defaultdict`

The other half of that same `.setdefault()` pattern, having a default ready automatically, without writing the check yourself:

```python
from collections import defaultdict

word_counts = defaultdict(int)   # int() gives 0 as the default

for word in words:
    word_counts[word] += 1

print(word_counts)
```
```
defaultdict(<class 'int'>, {'apple': 3, 'banana': 1, 'cherry': 1})
```

`defaultdict(int)` means "if a key doesn't exist yet, create it with `int()`'s default, `0`, automatically." No `.setdefault()` line needed at all, the very first time a new word shows up, it's already treated as `0` before you add `1` to it.

**`Counter` versus `defaultdict`:** if counting is specifically what you're doing, `Counter` is more direct and comes with counting-specific extras like `.most_common()`. `defaultdict` is more general, useful any time you want a default value for missing keys, not just counting.

### `namedtuple` (an Introduction)

A regular tuple only lets you access fields by position, `point[0]`, `point[1]`, which can get hard to read. A **namedtuple** gives each position a name too:

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)

print(p.x)      # 3, by name
print(p[0])      # 3, still works by position too
print(p)          # Point(x=3, y=4)
```

`Point` here is a new type, built on the fly, `p` is a normal tuple underneath, still ordered, still immutable, everything you already know about tuples still applies, you've just given its fields readable names instead of only remembering "position 0 is x."

---

## Try it Yourself

**(a)** Create a list `numbers = [5, 3, 8, 1]`. Manually call `iter()` on it, and use `next()` four times to print every value, then try a fifth `next()` and watch it crash with `StopIteration`, exactly what a `for` loop is quietly catching for you every time one finishes.

**(b)** You have `grades = ["A", "B", "A", "C", "B", "A"]`. Use `Counter` to find how many of each grade there are, then print the single most common one.

**(c)** Use `defaultdict(list)` to group these words by their first letter: `["apple", "avocado", "banana", "blueberry", "cherry"]`. (Hint: `groups[word[0]].append(word)`.)

**Your turn:** create a `namedtuple` called `Student` with fields `name` and `score`. Create two students, and print each one's name and score using the named fields.

---

## Common Mistakes

- Calling `next()` past the end of an iterator without expecting `StopIteration`, once it's exhausted, it stays exhausted, you can't rewind it
- Assuming a generator can be looped over twice, it can't, once exhausted, you need to call the generator function again to get a fresh one
- Forgetting `defaultdict` needs a callable as its default, `defaultdict(int)` or `defaultdict(list)`, not `defaultdict(0)`
- Using `Counter` when you actually need a regular dictionary with custom values, `Counter` is specifically for counting, not general-purpose key-value storage
- Trying to modify a `namedtuple` field directly, it's still a tuple underneath, still immutable

---

## Interview Questions

**Q1: What's the difference between an iterable and an iterator?**

A: An iterable is anything you can loop over, a list, string, or dict. An iterator is the object actually doing the looping, produced by calling `iter()` on an iterable, and advanced one step at a time with `next()`.

**Q2: Is a generator an iterator?**

A: Yes. A generator function, using `yield`, produces an object that already implements the iterator protocol, responding to `next()` and raising `StopIteration` when exhausted, exactly like any other iterator.

**Q3: What does `Counter` give you that a plain dictionary doesn't?**

A: Automatic counting in one line, `Counter(iterable)`, plus counting-specific extras like `.most_common()`, without manually checking whether a key exists first.

**Q4: What's the advantage of a `namedtuple` over a regular tuple?**

A: You can access fields by name (`p.x`) instead of only by position (`p[0]`), which is far more readable, while keeping everything else about tuples, ordering, immutability, unchanged.

---

## Quick Recap

- Every iterable produces an iterator via `iter()`; `next()` advances it one step, raising `StopIteration` when exhausted, this is what a `for` loop is doing internally the whole time.
- A generator (`yield`) is itself an iterator, not a separate concept, just a convenient way to write one.
- `Counter(iterable)` counts occurrences in one line, and adds extras like `.most_common()`.
- `defaultdict(type)` auto-creates a default value for missing keys, removing the need for manual `.setdefault()` checks.
- `namedtuple` gives tuple fields readable names, while staying ordered and immutable underneath.


##  Reference Links

- [Python 3 Documentation — `collections` Module](https://docs.python.org/3/library/collections.html)
- [The Python Tutorial — Iterators](https://docs.python.org/3/tutorial/classes.html#iterators)
- [The Python Tutorial — Generators](https://docs.python.org/3/tutorial/classes.html#generators)
- [Real Python — Iterators and Iterables in Python](https://realpython.com/python-iterators-iterables/)
- [Real Python — How to Use Generators and yield in Python](https://realpython.com/introduction-to-python-generators/)
- [Real Python — Python's collections: A Buffet of Specialized Data Types](https://realpython.com/python-collections-module/)
- [W3Schools — Python Iterators](https://www.w3schools.com/python/python_iterators.asp)
