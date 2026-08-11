# Tuples

---

## Why Tuples?

You've actually already used tuples without knowing it. That swap trick, `x, y = y, x`, and a function returning more than one value, both relied on this same type behind the scenes. A **tuple** is an ordered collection, just like a list, with one key difference: once created, it can't be changed.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Tuple** | Python's ordered, unchangeable collection, written with `( )` |
| **Packing** | Creating a tuple by writing values separated by commas |
| **Unpacking** | Assigning a tuple's values into separate variables at once |
| **Immutable** | Can't be changed after creation, same idea you saw with strings |
| **Hashable** | Usable as a dictionary key, a benefit immutability enables |

---

## Creating and Accessing

A tuple is written with parentheses, values separated by commas:

```python
point = (3, 4)
print(point)
```
```
(3, 4)
```

**The parentheses are actually optional**, the commas are what really make it a tuple, this is called **packing**:

```python
point = 3, 4
print(point)
```
```
(3, 4)
```

**One easy trap:** a single-item tuple needs a trailing comma, or Python just sees ordinary parentheses around one value:

```python
not_a_tuple = (5)
print(type(not_a_tuple))   # <class 'int'>

single_tuple = (5,)
print(type(single_tuple))  # <class 'tuple'>
```

**Indexing and slicing work exactly like they do on lists:**

```python
colors = ("red", "green", "blue")
print(colors[0])     # red
print(colors[-1])    # blue
print(colors[0:2])   # ('red', 'green')
```

`type()` and `len()` work the same way too:

```python
print(type(colors))   # <class 'tuple'>
print(len(colors))    # 3
```

---

## Tuple Assignment and Unpacking

**Unpacking** takes a tuple's values and assigns them to separate variables in one line:

```python
point = (3, 4)
x, y = point
print(x)
print(y)
```
```
3
4
```

The number of variables has to match the number of values, same rule you already know from plain multiple assignment:

```python
x, y, z = point   # ValueError: not enough values to unpack
```

**This is exactly what makes the swap trick work:**

```python
a = 1
b = 2
a, b = b, a
print(a, b)
```
```
2 1
```

`b, a` on the right packs a tuple `(2, 1)` first, *then* `a, b =` unpacks it into the two names, all in one step, no temporary variable needed.

**And it's exactly what happens when a function returns multiple values:**

```python
def min_max(a, b, c):
    return min(a, b, c), max(a, b, c)

lowest, highest = min_max(4, 9, 2)
```

`return min(a, b, c), max(a, b, c)` packs both values into a tuple behind the scenes, and `lowest, highest = ...` unpacks them right back out.

---

## Nested Tuples

A tuple can hold other tuples, same idea as nested lists:

```python
student = ("Ananya", (85, 90, 78))
print(student[0])       # Ananya
print(student[1])       # (85, 90, 78)
print(student[1][0])    # 85, first score
```

Unpacking can go one level deeper too:

```python
name, scores = student
print(name)
print(scores)
```
```
Ananya
(85, 90, 78)
```

**One subtlety worth seeing directly:** a tuple's immutability only applies to what the tuple itself holds, not necessarily what's *inside* those items:

```python
record = ("Ananya", [85, 90, 78])   # a list stored inside a tuple

record[0] = "Priya"     # TypeError, can't replace what the tuple holds
record[1].append(100)   # works fine, the list itself is still mutable
print(record)
```
```
('Ananya', [85, 90, 78, 100])
```

The tuple never let go of its second item, that list is still the exact same list it started with, it just changed *its own* contents. The tuple's guarantee is only about its own slots, not about what a mutable item stored inside one of those slots can do on its own.

---

## Immutability

Once created, a tuple **cannot be changed**, same rule you already know from strings:

```python
point = (3, 4)
point[0] = 10   # TypeError: 'tuple' object does not support item assignment
```

There's no `.append()`, `.remove()`, or `.sort()` either, none of the in-place list methods exist on a tuple, because none of them would make sense on something that can't change.

**That doesn't mean tuples have zero methods, though.** Two non-mutating ones still exist, the same `.count()` and `.index()` you already know from lists:

```python
scores = (85, 90, 85, 78)
print(scores.count(85))    # 2
print(scores.index(90))    # 1
```

Neither of these changes the tuple, they just read information from it, which is exactly why they're allowed.

**So when would you actually prefer a tuple over a list?**

- **The data genuinely shouldn't change.** A point's coordinates, a date, an RGB color, these are naturally fixed once created, using a tuple makes that intent obvious just from the type.
- **You need it as a dictionary key.** Only immutable, **hashable** types can be used as dictionary keys, a list can't, but a tuple can, this becomes directly useful once you reach dictionaries.
- **You're returning multiple values from a function.** As shown above, this happens through a tuple automatically, whether you think about it or not.

If the collection genuinely needs to grow, shrink, or be edited later, a list is still the right tool, immutability is a feature here, not a limitation, but only when you actually want it.

---

## Basic Tuple Operations

**Concatenation**, joining two tuples with `+`, produces a **new** tuple:

```python
a = (1, 2)
b = (3, 4)
print(a + b)     # (1, 2, 3, 4)
```

**Repetition**, with `*`:

```python
print((0,) * 4)    # (0, 0, 0, 0)
```

**Membership**, with `in` / `not in`, same as lists and strings:

```python
colors = ("red", "green", "blue")
print("green" in colors)       # True
print("purple" not in colors)   # False
```

---

## Try it Yourself

**(a)** Create a tuple `dimensions = (1920, 1080)`, then unpack it into `width` and `height`. Print both.

**(b)** Write a function `divide(a, b)` that returns both the quotient and the remainder as a tuple (hint: `//` and `%` from earlier). Unpack the result when calling it.

**(c)** Create `location = ("Chennai", (13.08, 80.27))`. Print just the latitude (the first value inside the nested tuple).

**Your turn:** try to change one value inside a tuple you created, confirm you get a `TypeError`, and write one sentence explaining why that's actually useful in this case.

---

## Common Mistakes

- Forgetting the trailing comma on a single-item tuple, `(5)` is just the number `5`, not a tuple
- Trying to use `.append()` or any in-place method on a tuple, none of them exist, tuples don't have them
- Unpacking into the wrong number of variables, the count has to match exactly, or you get a `ValueError`
- Assuming parentheses are what make a tuple, they don't, the commas do, `3, 4` is already a tuple without any `()`
- Forgetting a tuple's immutability only covers its own slots, a list stored inside a tuple can still be edited in place, as shown above

---

## Interview Questions

**Q1: What's the main difference between a tuple and a list?**

A: Both are ordered collections, but a tuple is immutable, it can't be changed after creation, while a list is mutable and can be updated, added to, or shortened freely.

**Q2: What actually makes something a tuple, the parentheses or the commas?**

A: The commas. Parentheses are optional and mostly used for readability or grouping, `3, 4` is already a tuple on its own.

**Q3: Why would you use a tuple instead of a list?**

A: When the data genuinely shouldn't change (fixed coordinates, a date), when you need it as a dictionary key (only immutable types are hashable), or automatically, whenever a function returns multiple values.

**Q4: How do you create a tuple with just one item?**

A: With a trailing comma, `(5,)`. Without the comma, `(5)` is just parentheses around an integer, not a tuple at all.

**Q5: Do tuples have any methods at all, given they're immutable?**

A: Yes, two, `.count()` and `.index()`. Both only read information, they don't change the tuple, which is exactly why they're allowed to exist on an immutable type.

**Q6: If a tuple holds a list, can that list still be changed?**

A: Yes. The tuple guarantees its own slots won't be reassigned to a different object, but a mutable item stored in one of those slots, like a list, can still be modified in place through its own methods.

---

## Quick Recap

- A tuple is an ordered, immutable collection; commas make it a tuple, parentheses are optional.
- Unpacking assigns a tuple's values to separate variables at once; this is what powers the swap trick and multi-value function returns.
- A single-item tuple needs a trailing comma, `(5,)`, or it's just a plain value.
- Tuples can be nested, same idea as nested lists, indexing into them one level at a time.
- No in-place methods exist, immutability means no `.append()`, `.remove()`, or `.sort()`; `.count()` and `.index()` still work, since neither one mutates.
- A tuple's immutability covers its own slots, not necessarily what's inside them, a mutable item like a list, stored inside a tuple, can still change on its own.
- Prefer a tuple when data shouldn't change, or when it needs to be hashable, like a dictionary key.
- `+`, `*`, and `in`/`not in` all work on tuples, same as lists and strings.


## Reference Links

- [The Python Tutorial — Data Structures (Tuples and Sequences)](https://docs.python.org/3/tutorial/datastructures.html#tuples-and-sequences)
- [Python 3 Documentation — Built-in Types (Sequence Types)](https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range)
- [Real Python — Python Tuples](https://realpython.com/python-tuple/)
- [W3Schools — Python Tuples](https://www.w3schools.com/python/python_tuples.asp)
