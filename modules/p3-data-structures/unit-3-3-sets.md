#  Sets

---

## Why Sets?

Lists and tuples keep every item, in order, duplicates and all. A **set** does the opposite: it keeps only **unique** values, in no particular order, and is built for one thing especially well, checking whether something is in it, fast.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Set** | An unordered collection of unique values, written with `{ }` |
| **Uniqueness** | A set automatically drops duplicates, keeping only one of each value |
| **Unordered** | Items have no fixed position, no indexing, no slicing |
| **Union** | All items from either set, combined |
| **Intersection** | Only items present in both sets |
| **Difference** | Items in one set but not the other |

---

## Creating a Set

Written with curly braces, values separated by commas:

```python
fruits = {"apple", "banana", "cherry"}
print(fruits)
```
```
{'apple', 'banana', 'cherry'}
```

**Duplicates are automatically dropped:**

```python
numbers = {1, 2, 2, 3, 3, 3}
print(numbers)
```
```
{1, 2, 3}
```

**One sharp trap:** `{}` on its own creates an empty **dictionary**, not an empty set, dictionaries get their own unit right after this one, but this gotcha is worth knowing now. Use `set()` instead:

```python
empty = {}
print(type(empty))       # <class 'dict'>, not what you might expect

actually_empty = set()
print(type(actually_empty))   # <class 'set'>
```

`type()` and `len()` work the same way you already know:

```python
print(type(fruits))   # <class 'set'>
print(len(fruits))    # 3
```

---

## No Indexing, No Order

A set has no positions, so this doesn't work:

```python
fruits[0]   # TypeError: 'set' object is not subscriptable
```

There's no guaranteed order either, printing the same set twice in different sessions can show items in a different sequence. If order matters to you, a list or tuple is the right tool, not a set.

**What you *can* do is loop through one, and check membership**, same `in` operator from lists and tuples:

```python
for fruit in fruits:
    print(fruit)

print("apple" in fruits)       # True
print("mango" not in fruits)    # False
```

**This membership check is the whole reason sets exist.** Checking `in` on a set is dramatically faster than checking `in` on a list, especially as the collection grows, because of how a set is stored internally. You don't need to know the mechanics to benefit from it, just remember: if you're mainly checking "is this value already in here?" over and over, a set is usually the right structure.

---

## Adding and Removing

```python
fruits = {"apple", "banana"}

fruits.add("cherry")
print(fruits)
```
```
{'apple', 'banana', 'cherry'}
```

| Method | Does what |
|---|---|
| `.add(x)` | adds `x`, does nothing if it's already there |
| `.remove(x)` | removes `x`, raises `KeyError` if it's not present |
| `.discard(x)` | removes `x` if present, does nothing (no error) if not |
| `.pop()` | removes and returns an arbitrary item, sets have no "first" item |
| `.clear()` | removes everything |

```python
fruits.remove("mango")    # KeyError, not in the set
fruits.discard("mango")   # no error, just does nothing
```

`.discard()` is the safer choice whenever you're not sure the value is actually there.

---

## Set Operations

This is where sets really shine, combining and comparing collections, using either operators or their equivalent methods:

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

print(a | b)    # union: {1, 2, 3, 4, 5, 6}, everything from either
print(a & b)    # intersection: {3, 4}, only in both
print(a - b)    # difference: {1, 2}, in a but not b
print(a ^ b)    # symmetric difference: {1, 2, 5, 6}, in one but not both
```

| Operator | Method | Meaning |
|---|---|---|
| `\|` | `.union()` | everything from either set |
| `&` | `.intersection()` | only items in both |
| `-` | `.difference()` | in the first, not the second |
| `^` | `.symmetric_difference()` | in one or the other, not both |

**A genuinely practical use case:** finding common items between two collections, cleanly, without writing a loop yourself:

```python
students_python = {"Ananya", "Rahul", "Priya"}
students_java = {"Rahul", "Priya", "Karthik"}

print(students_python & students_java)   # {'Rahul', 'Priya'}, taking both courses
```

---

## Set Comprehension

Same idea as list comprehension, just with `{ }` instead of `[ ]`:

```python
numbers = [1, 2, 2, 3, 4, 4, 5]
unique_doubled = {n * 2 for n in numbers}
print(unique_doubled)
```
```
{2, 4, 6, 8, 10}
```

Notice the duplicates in `numbers` (two `2`s, two `4`s) automatically collapsed, that's the set's uniqueness rule applying, even inside a comprehension.

---

## `frozenset`, a Brief Look

A **frozenset** is exactly like a set, unique, unordered, fast membership checks, except it's immutable, same idea as a tuple being an immutable list:

```python
frozen = frozenset([1, 2, 3])
frozen.add(4)   # AttributeError, frozensets have no .add()
```

You won't reach for this often, but it exists for the same reason tuples do, when you need an unchangeable collection, and, like a tuple, a frozenset is hashable, so it can be used as a dictionary key, something a regular set can't do.

---

## Try it Yourself

**(a)** Create a set from the list `["red", "blue", "red", "green", "blue"]`. Print it, and confirm the duplicates are gone.

**(b)** Two friend groups: `group_a = {"Ananya", "Rahul", "Priya", "Karthik"}` and `group_b = {"Priya", "Karthik", "Meera"}`. Find who's in both groups, and who's only in `group_a`.

**(c)** Write a set comprehension that collects only the unique first letters from this list: `["apple", "avocado", "banana", "blueberry", "cherry"]`.

**Your turn:** create an empty set correctly (not `{}`), add three items to it one at a time using `.add()`, then remove one using `.discard()` and confirm it's gone.

---

## Common Mistakes

- Writing `{}` expecting an empty set, it creates an empty dictionary instead, use `set()`
- Trying to index a set, `fruits[0]`, sets have no positions
- Using `.remove()` on a value that might not exist, raising a `KeyError`, `.discard()` is safer when unsure
- Assuming a set preserves the order you added items in, it doesn't, don't rely on it
- Forgetting sets can only hold **hashable** (immutable) items, you can put a tuple inside a set, but not a list, `{[1, 2]}` raises a `TypeError`

---

## Interview Questions

**Q1: What's the main difference between a set and a list?**

A: A set is unordered and only holds unique values, with no indexing. A list is ordered, allows duplicates, and supports indexing and slicing.

**Q2: Why is checking membership (`in`) faster on a set than a list?**

A: A set is stored in a way that lets Python check for a value's presence directly, without scanning through every item one by one, the way it has to on a list. This difference grows as the collection gets larger.

**Q3: What's the difference between `.remove()` and `.discard()`?**

A: `.remove(x)` raises a `KeyError` if `x` isn't in the set. `.discard(x)` does nothing if it's missing, no error either way.

**Q4: What's a frozenset, and why would you use one?**

A: An immutable version of a set. You'd use it when you need a fixed collection of unique values that also needs to be hashable, for example, usable as a dictionary key, which a regular set can't be.

---

## Quick Recap

- A set holds unique, unordered values, written with `{ }`; use `set()` for an empty one, `{}` makes a dictionary instead.
- No indexing or slicing, sets have no positions; membership checks (`in`) are their main strength, and are fast.
- `.add()`, `.remove()` (raises an error if missing), `.discard()` (doesn't), `.pop()`, and `.clear()` modify a set in place.
- `|`, `&`, `-`, `^` give you union, intersection, difference, and symmetric difference.
- Set comprehension (`{x for x in ...}`) works exactly like list comprehension, with automatic deduplication.
- A `frozenset` is an immutable set, usable as a dictionary key, unlike a regular set.

---

##  Reference Links

- [Python 3 Documentation — Set Types: set, frozenset](https://docs.python.org/3/library/stdtypes.html#set-types-set-frozenset)
- [The Python Tutorial — Data Structures: Sets](https://docs.python.org/3/tutorial/datastructures.html#sets)
- [Real Python — Sets in Python](https://realpython.com/python-sets/)
- [W3Schools — Python Sets](https://www.w3schools.com/python/python_sets.asp)
