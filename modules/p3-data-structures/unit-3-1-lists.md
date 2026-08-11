# Lists

---

## Why Lists?

You've actually been seeing lists for a while now, without a proper introduction. `.split()` handed one back when we covered strings. `sorted()` worked on one too, when we covered functional constructs. This is that proper introduction: a **list** is Python's ordered collection type, holding many values of **any type**, even a mix, under one name, and unlike strings, you can change it after creating it.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **List** | Python's ordered, changeable collection, written with `[ ]` |
| **Element** | One individual value stored inside a list |
| **Index** | The position of an element, starting at `0`, same idea as string indexing |
| **Mutability** | Being changeable after creation, lists are mutable |
| **In place** | A change made directly to the existing list, without creating a new one |
| **Alias** | A second variable name pointing to the exact same list in memory, not a copy |
| **`in` / `not in`** | Checks whether a value exists in a sequence, returns a `bool` |

---

## Creating and Indexing

A list is written with square brackets, items separated by commas:

```python
fruits = ["apple", "banana", "cherry"]
print(fruits)
```
```
['apple', 'banana', 'cherry']
```

As already mentioned, a list can hold values of any type, not just matching ones:

```python
mixed = ["Ananya", 21, 29.5, True]
```

Two functions you'll use constantly, both already familiar:

```python
print(type(fruits))   # <class 'list'>
print(len(fruits))    # 3, number of items
```

**Indexing works exactly like it did on strings:**

```python
print(fruits[0])    # apple
print(fruits[-1])   # cherry, negative counts from the end
```

```python
fruits[10]   # IndexError: list index out of range
```

---

## Slicing with Step

Slicing on a list uses the exact same `start:stop:step` syntax you already know from strings:

```python
numbers = [10, 20, 30, 40, 50, 60]

print(numbers[1:5:2])   # [20, 40], step of 2, stop excluded as always
print(numbers[::2])     # [10, 30, 50], every 2nd item
print(numbers[::-1])    # [60, 50, 40, 30, 20, 10], reversed
```

If this already feels familiar, that's the entire point of learning it on strings first, nothing new to memorize here, just the same syntax on a different type.

---

## Updating Items and Mutability

This is the one real difference from strings. A string can't be changed in place, a list can:

```python
name = "python"
name[0] = "P"   # TypeError, you saw this earlier, with strings

fruits = ["apple", "banana", "cherry"]
fruits[0] = "avocado"   # works fine
print(fruits)
```
```
['avocado', 'banana', 'cherry']
```

No reassignment needed, `fruits` is still the same list, just with different contents now. This is **in-place** modification.

**Mutability has a side effect worth knowing about, aliasing:**

```python
original = [1, 2, 3]
alias = original       # NOT a copy, alias points to the exact same list

alias.append(4)
print(original)        # [1, 2, 3, 4], original changed too!
```

`alias = original` didn't create a second list, it just gave the same list a second name. Changing one changes both, because there's only ever one list in memory. If you actually need an independent copy, use `.copy()`:

```python
real_copy = original.copy()
real_copy.append(5)
print(original)     # unaffected: [1, 2, 3, 4]
print(real_copy)    # [1, 2, 3, 4, 5]
```

---

## List Operators: `in`, `+`, `*`

**`in` and `not in`** check whether a value exists in a list, this is actually a general Python operator, it works on strings too, we just hadn't needed it until now:

```python
fruits = ["apple", "banana", "cherry"]

print("banana" in fruits)       # True
print("mango" not in fruits)    # True
```

This returns a plain `bool`, so it works directly inside an `if`:

```python
if "banana" in fruits:
    print("We have bananas.")
```

**`+` joins two lists together** (concatenation, same idea as joining strings earlier), producing a **new** list:

```python
a = [1, 2, 3]
b = [4, 5]
print(a + b)     # [1, 2, 3, 4, 5]
print(a)         # [1, 2, 3], unchanged, this is different from the in-place methods below, + never modifies in place
```

**`*` repeats a list:**

```python
print([0] * 5)    # [0, 0, 0, 0, 0]
```

---

## List Methods

All of these change the list **in place**, and most of them return `None`, not the list itself, don't write `fruits = fruits.append(...)`.

| Method | Does what | Example |
|---|---|---|
| `.append(x)` | adds `x` to the end | `fruits.append("mango")` |
| `.insert(i, x)` | inserts `x` at position `i` | `fruits.insert(0, "fig")` |
| `.remove(x)` | removes the first matching value `x` | `fruits.remove("banana")` |
| `.pop(i)` | removes and returns the item at `i` (last item if empty) | `fruits.pop()` |
| `.extend(other)` | adds every item from another list | `fruits.extend(["fig", "date"])` |
| `.index(x)` | returns the position of the first `x` | `fruits.index("cherry")` |
| `.count(x)` | counts how many times `x` appears | `fruits.count("apple")` |
| `.clear()` | removes everything, leaves an empty list | `fruits.clear()` |

```python
fruits = ["apple", "banana"]
fruits.append("cherry")
fruits.insert(1, "fig")
print(fruits)
```
```
['apple', 'fig', 'banana', 'cherry']
```

`.remove(x)` raises a `ValueError` if `x` isn't actually in the list, and `.pop()` on an empty list raises an `IndexError`, worth checking `if x in list_name` first if you're not sure, using the `in` operator from above.

**One more way to remove things, the `del` keyword:**

```python
fruits = ["apple", "banana", "cherry"]
del fruits[0]        # removes by position
print(fruits)         # ['banana', 'cherry']

del fruits[0:2]       # can also delete a whole slice at once
print(fruits)          # []
```

Three ways to remove something now exist, and each answers a different question: `.remove(x)` when you know the **value**, `.pop(i)` when you want the removed item **back**, `del list[i]` when you just want it **gone**, by position, no return value at all.

**An empty list is falsy**, same rule you learned for `0`, `0.0`, and `""`:

```python
items = []
if items:
    print("Has items.")
else:
    print("Empty.")
```
```
Empty.
```

This is a common, idiomatic way to check "is this list empty?" instead of writing `if len(items) == 0:`.

---

## Looping Through a List

The `for` loop works directly on a list, no `range()` needed:

```python
fruits = ["apple", "banana", "cherry"]

for fruit in fruits:
    print(fruit)
```
```
apple
banana
cherry
```

Need the index too? `enumerate()` works here exactly the same way it did on strings:

```python
for index, fruit in enumerate(fruits):
    print(index, fruit)
```
```
0 apple
1 banana
2 cherry
```

---

## Sorting

**`sorted()`** returns a **new**, sorted list, the original is untouched:

```python
numbers = [4, 1, 3, 2]
result = sorted(numbers)
print(result)      # [1, 2, 3, 4]
print(numbers)      # [4, 1, 3, 2], unchanged
```

**`.sort()`** sorts the list **in place**, and returns `None`:

```python
numbers = [4, 1, 3, 2]
numbers.sort()
print(numbers)      # [1, 2, 3, 4], changed directly
```

| | `sorted(list)` | `list.sort()` |
|---|---|---|
| Returns | A new, sorted list | `None` |
| Original list | Unchanged | Changed in place |
| Works on | Any iterable (strings, ranges, lists) | Lists only |

**Both accept `reverse=True` and `key=`,** same `key=` idea from the `sorted(words, key=len)` example you saw earlier:

```python
numbers = [4, 1, 3, 2]
print(sorted(numbers, reverse=True))          # [4, 3, 2, 1]

words = ["banana", "fig", "apple"]
print(sorted(words, key=len))                  # ['fig', 'apple', 'banana']
```

---

## Nested Lists

A list can hold other lists, useful for grids, tables, or rows of data:

```python
grid = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]

print(grid[0])       # [1, 2, 3], the first row
print(grid[0][1])    # 2, first row, second item
```

Looping through a nested list needs a loop inside a loop, same nested-loop idea from earlier:

```python
for row in grid:
    for item in row:
        print(item, end=" ")
    print()
```
```
1 2 3
4 5 6
7 8 9
```

---

## List Comprehension

A compact, one-line way to build a new list from an existing sequence, optionally transforming or filtering it as it goes:

```python
numbers = [1, 2, 3, 4, 5]
doubled = [n * 2 for n in numbers]
print(doubled)
```
```
[2, 4, 6, 8, 10]
```

Read it as: "`n * 2`, for every `n` in `numbers`." This does the same job as looping and calling `.append()` for each item, just in one line:

```python
doubled = []
for n in numbers:
    doubled.append(n * 2)
```

**Add a condition to filter, same idea as `filter()`:**

```python
evens = [n for n in numbers if n % 2 == 0]
print(evens)
```
```
[2, 4]
```

You can combine both, transforming and filtering in the same expression:

```python
result = [n * 2 for n in numbers if n % 2 == 0]
print(result)
```
```
[4, 8]
```

Comprehensions are popular because they're compact, but readability matters more than brevity, if the logic gets complicated, a regular loop is often clearer.

---

## Try it Yourself

**(a)** Create a list of your favorite 5 movies. Print the first, the last, and a slice of the middle three.

**(b)** Take `scores = [88, 92, 79, 95, 60]`. Sort it in descending order without changing the original list, then print both.

**(c)** Write a list comprehension that squares every number in `[1, 2, 3, 4, 5]`, but only for numbers greater than `2`.

**Your turn:** create `matrix = [[1, 2], [3, 4], [5, 6]]`. Using nested loops, print the sum of every number in the matrix.

---

## Common Mistakes

- Assuming `alias = original` makes a copy, it doesn't, both names point to the same list, use `.copy()` for an independent one
- Writing `fruits = fruits.append(x)`, `.append()` returns `None`, this overwrites your list with `None`
- Confusing `.sort()` (in place, returns `None`) with `sorted()` (returns a new list)
- Calling `.remove()` on a value that isn't in the list, or `.pop()` on an empty one, both raise errors
- Forgetting slicing's `stop` is excluded, same rule as strings, `a[1:4]` gives 3 items, not 4
- Over-nesting list comprehensions until they're harder to read than a plain loop would be
- Using `+` expecting it to modify a list in place, it always returns a brand-new list instead
- Checking `if len(items) == 0:` when `if not items:` (or `if items:`) is the more idiomatic, Pythonic way

---

## Interview Questions

**Q1: What's the difference between a list and a string, in terms of mutability?**

A: A string is immutable, you can't change it in place. A list is mutable, you can update, add, or remove elements directly, without creating a new list.

**Q2: What's the difference between `.sort()` and `sorted()`?**

A: `.sort()` sorts a list in place and returns `None`. `sorted()` returns a brand-new sorted list and leaves the original untouched, and it works on any iterable, not just lists.

**Q3: What happens when you do `alias = original` with two lists?**

A: `alias` doesn't become a copy, it becomes a second name pointing to the exact same list in memory. Changing one through either name changes what both names see.

**Q4: What does a list comprehension actually do?**

A: It builds a new list from an existing iterable in one expression, optionally transforming each item and/or filtering which items are included, equivalent to a `for` loop with `.append()`, just more compact.

**Q5: What's the difference between `.remove()`, `.pop()`, and `del`?**

A: `.remove(x)` deletes by value, the first match. `.pop(i)` deletes by position and returns the removed item. `del list[i]` deletes by position (or a whole slice) with nothing returned at all.

**Q6: Does `+` modify a list in place?**

A: No. `a + b` always returns a brand-new list, the originals are untouched. To add items to an existing list in place, use `.extend()` instead.

---

## Quick Recap

- A list is an ordered, mutable collection, written with `[ ]`; indexing and slicing work exactly like they did on strings.
- `type()`, `len()`, and `in`/`not in` all work on lists, same as you'd expect from strings.
- `+` joins two lists into a new one; `*` repeats a list; neither modifies in place.
- Lists can be changed in place: `.append()`, `.insert()`, `.remove()`, `.pop()`, `.extend()`, `.index()`, `.count()`, `.clear()`, and `del` for removing by position or slice.
- `alias = original` doesn't copy a list, both names share the same one; use `.copy()` for an independent copy.
- `sorted()` returns a new list; `.sort()` sorts in place and returns `None`. Both take `reverse=` and `key=`.
- An empty list is falsy, same rule as `0` and `""`; `if items:` is the idiomatic emptiness check.
- Nested lists (lists of lists) need nested loops to fully iterate through.
- List comprehensions build a new list in one line, optionally with a filter, but stay readable over clever.


##  Reference Links

- [The Python Tutorial — Data Structures (Lists)](https://docs.python.org/3/tutorial/datastructures.html)
- [Python 3 Documentation — Common Sequence Operations and List Methods](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations)
- [Python 3 Documentation — List Comprehensions](https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions)
- [Real Python — Lists and Tuples in Python](https://realpython.com/python-lists-tuples/)
- [W3Schools — Python Lists](https://www.w3schools.com/python/python_lists.asp)
