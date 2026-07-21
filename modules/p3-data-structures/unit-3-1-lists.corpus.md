---
topic_id: fe27250e-c2c2-491c-afd7-57d9b533deab
title: Unit 3.1 - Lists
position_in_module: 1
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 3.1 - Lists — Topic Corpus

## 2. Prerequisites

This topic builds directly on skills from earlier units — nothing here is taught from a blank slate:

- **Unit 1.2 - Variables, Identifiers & Types** (`44580fc5-261e-4a0a-bc9b-22972428f96e`) — you already know what a variable, a type, and dynamic typing are; a list is simply a new type that a variable can hold.
- **Unit 1.4 - Statements, Conversion & Output** (`3c04d4c0-e9ca-4a4a-9d5e-368effd5d162`) — you already met **indexing** (reaching into a string by position) and **tuple unpacking** (splitting a sequence of values into several names at once); both ideas reappear here applied to lists.
- **Unit 2.2 - Loops** (`0ef5b1c5-43be-4f7a-8e00-ac4ab29451e5`) — you already know what a **for loop**, an **iterable**, `enumerate()`, and `range()` are; this unit gives your loops a much richer thing to loop over.
- **Unit 2.4 - Functional Constructs** (`07a78e31-727b-4c0d-95aa-4e7712dceda2`) — you already met `sorted(key=...)` in passing; this unit puts it to full use on lists specifically.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Create** a list using square-bracket literal syntax and **apply** positive and negative indexing to read individual elements.
- **Apply** slicing (`start:stop:step`) to extract a sub-list without changing the original list, including the `[::2]` and `[::-1]` idioms.
- **Explain** what mutability means for a list, and **differentiate** an alias (`b = a`) from an independent copy (`a.copy()` or `a[:]`).
- **Apply** the core list methods (`append`, `insert`, `remove`, `pop`, `extend`, `index`, `count`, `clear`, `sort`) to add, remove, search, and reorder elements.
- **Differentiate** `list.sort()` from the built-in `sorted()`, and **apply** nested-list indexing (`grid[row][col]`) to grid-shaped data.
- **Create** a list comprehension that maps and/or filters an existing sequence into a new list in a single expression.

## 4. Introduction

Imagine you are asked to store the scores of five students on a quiz. Using what you know so far, you might write `score1 = 78`, `score2 = 85`, `score3 = 92`, and so on. That works for five students. It falls apart the moment the class has thirty students, or when you do not know in advance how many scores there will be — which is almost always the case in a real program. What you actually need is one variable that can hold *many* values at once, in order, and that you can grow or shrink as the program runs. That is exactly what a Python **list** gives you: `scores = [78, 85, 92, 66, 74]` — five values, one name.

Lists are Python's default, everyday way of grouping data. A shopping cart is a list of prices. A chat transcript is a list of messages. A batch of results coming back from a piece of software is a list of records. In the AI labs this course builds toward, a list is how you will hold a batch of prompts sent to a model, a batch of scores returned for those prompts, or a running record of results — the module this unit belongs to exists precisely because every one of those AI labs needs somewhere to put "many prompts," "many results," "many scores" (see the Module III description). You already know how to write a `for` loop; a list is the everyday thing you will now give that loop to work on.

This unit teaches you everything you need to create a list, reach into it by position, take a chunk out of it, understand what changing it "in place" really means, use its built-in methods, sort it two different ways, nest one list inside another, and finally compress the common "build an empty list, loop, and add to it" pattern into a single compact expression called a list comprehension.

## 5. Core Concepts

**Quick-reference glossary** (each term is defined in full in the subsection noted):

| Term | Simple meaning | Defined in |
|---|---|---|
| List | Python's ordered, mutable collection, written with `[ ]` | §5.1 |
| Element | One individual value stored inside a list | §5.1 |
| Index | The zero-based numbered position of an element | §5.2 |
| Slice | A sub-section of a list, extracted as a new list | §5.3 |
| Mutability | The property of being changeable after creation | §5.4 |
| Alias | A second name referring to the exact same list, not a copy | §5.4 |
| Shallow copy | An independent outer list whose inner lists may still be shared | §5.4 |
| List method | A built-in function attached to a list, called with `.` syntax | §5.5 |
| Nested list | A list containing other lists as its elements | §5.7 |
| List comprehension | A one-line expression that builds a new list from a sequence | §5.8 |
| `IndexError` | The error raised when an index does not exist in the list | §5.2 |

### 5.1 What a List Is

A **list** is Python's built-in, ordered, changeable collection of values, written as comma-separated items inside square brackets `[ ]` [1]. "Ordered" means the items stay in the position you placed them — Python does not shuffle them on its own. "Changeable," more precisely called **mutable**, means you can add, remove, or replace items after the list already exists, without having to build a brand-new list from scratch each time.

```python
fruits = ["apple", "banana", "cherry"]
numbers = [10, 20, 30, 40, 50]
mixed = ["Rohit", 25, True, 3.14]   # one list, several different types
empty = []                          # a list with zero items is still a valid list
```

Each individual value stored inside a list is called an **element**. Calling `type(fruits)` reports `<class 'list'>`, and `len(fruits)` reports the element count — `3` in this case. A list can hold zero elements, one element, or millions; the same square-bracket syntax and the same operations apply no matter the size [1].

### 5.2 Indexing a List

You already learned **indexing** in Unit 1.4 as a way to reach into a string by position. A list uses exactly the same idea: every element sits at a numbered position, called its **index**, and Python always starts counting from `0`, never `1`. Writing the list's name followed by an index in square brackets reads the single value sitting at that position [1]:

```python
fruits = ["apple", "banana", "cherry"]
print(fruits[0])    # apple  — the 1st element, at index 0
print(fruits[2])    # cherry — the 3rd element, at index 2
```

Python also allows **negative indexing**: counting backward from the end, where `-1` is the last element, `-2` is the second-to-last, and so on. This saves you from writing `fruits[len(fruits) - 1]` every time you need the final item:

```python
print(fruits[-1])   # cherry — last element
print(fruits[-3])   # apple  — same slot as fruits[0]
```

Asking for a position that does not exist — `fruits[3]` on this three-element list — raises an **`IndexError`**, an error type specific to sequences being accessed out of range. For a list of length `n`, valid positive indices run from `0` to `n - 1`, and valid negative indices run from `-1` to `-n`. A very common beginner mistake is forgetting that the last valid positive index is `len(my_list) - 1`, not `len(my_list)`.

### 5.3 Slicing a List

**Slicing** pulls out a whole chunk of a list at once instead of one element at a time. You describe the chunk with up to three numbers separated by colons — `start` (included), `stop` (excluded), and `step` (how many positions to jump each time) — written as `my_list[start:stop:step]`. Python always hands back a **brand-new list** containing that chunk, leaving the original untouched. Leaving out `start` defaults it to `0`; leaving out `stop` defaults it to `len(my_list)`; so `a[:]` copies the entire list [1]:

```python
a = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(a[0:2])      # [0, 1]           — start at 0, stop before 2
print(a[1:5])      # [1, 2, 3, 4]     — index 5 is excluded
print(a[:3])       # [0, 1, 2]        — start defaults to 0
print(a[7:])       # [7, 8, 9]        — stop defaults to len(a)
print(a[::2])      # [0, 2, 4, 6, 8]  — every 2nd element
print(a[::-1])     # [9, 8, 7, 6, ...] — a reversed copy
```

Two slicing idioms are worth memorizing because they appear constantly in real code: `a[::2]` (every second element) and `a[::-1]` (the list reversed, because a negative step walks backward). Unlike indexing, a slice with an out-of-range `start` or `stop` never raises an `IndexError` — Python simply clamps to what is available, and an empty result produces `[]`. Because a slice always builds a new list, slicing itself never changes the original list — a property that matters a great deal once you meet mutability in the next section.

Negative numbers work inside a slice exactly as they do inside a single index — counting from the end — which makes "the last three elements" a one-line expression:

```python
scores = [55, 60, 70, 85, 90, 95]
print(scores[-3:])     # [85, 90, 95] — last three elements
print(scores[:-2])     # [55, 60, 70, 85] — everything except the last two
```

### 5.3.1 Indexing/Slicing Inside a Loop

Because a list is directly iterable, most everyday code loops straight over the elements (`for score in scores:`) rather than looping over positions and then indexing (`for i in range(len(scores)): scores[i]`). The index-based style from Unit 2.2 still works and remains useful whenever the position itself is needed — for example alongside `enumerate(scores)` to print each score together with its 1-based rank.

### 5.4 Mutability, Aliasing, and Copying

A list is **mutable**: its contents can be changed after creation without building a new list. Assigning to a single index replaces that element in place; assigning to a slice can replace several elements at once [1]:

```python
colors = ["red", "green", "blue"]
colors[1] = "yellow"               # ['red', 'yellow', 'blue']
```

Mutability has a consequence every beginner must understand early. A variable holding a list actually holds a *reference* to that list's location in memory — not a private, independent copy of its values. If you assign one list variable to another name (`b = a`), both names become **aliases**: two labels pointing at the exact same underlying list. Changing the list through either name is visible through the other, because there is only ever one list:

```python
a = [1, 2, 3]
b = a              # b is an alias for a, not a copy
b.append(4)
print(a)           # [1, 2, 3, 4] — a changed too, even though only b was touched
```

This surprises many beginners, because with simple types like `int` or `str` (Unit 1.2), reassigning a value never affects the original variable — lists behave differently because they are mutable and referenced, not copied, on assignment.

To get an independent list, use a full slice (`b = a[:]`) or the `.copy()` method (`b = a.copy()`). Both produce a **shallow copy**: a new outer list whose elements are copied by reference. For a list of plain values (numbers, strings), a shallow copy behaves exactly like a fully independent copy. For a *nested* list (§5.7), a shallow copy only duplicates the outer list — the inner lists inside it are still shared with the original, so changing an inner list through the copy still changes the original too. This is a well-known source of hard-to-find bugs and is worth remembering as you move toward more advanced structures later in this module.

Aliasing matters just as much when a list is passed as an **argument** to a **function** (Unit 2.3). Because the function's **parameter** receives a reference to the same list, not a copy, any in-place method the function calls on that parameter — `append`, `sort`, and so on — is visible to the caller once the function returns, even though nothing was explicitly `return`ed:

```python
def add_bonus_mark(marks):
    marks.append(100)     # mutates the caller's list in place

class_marks = [78, 85, 92]
add_bonus_mark(class_marks)
print(class_marks)        # [78, 85, 92, 100] — changed, even with no return value
```

This is expected behavior once mutability and aliasing are understood together, but it surprises many beginners the first time they see a function change a list "from the outside."

### 5.4.1 Membership Testing with `in`

A separate but related question is simply "does this value already exist somewhere in the list?" The `in` operator answers this directly, returning a **`bool`** (`True` or `False`) without revealing a position:

```python
fruits = ["apple", "banana", "cherry"]
print("banana" in fruits)       # True
print("mango" in fruits)        # False
print("mango" not in fruits)    # True
```

This is different from the `for x in iterable:` form met in Unit 2.2 — there, `in` drives a loop over every element; here, `in` is a standalone expression that immediately produces a single `True`/`False` answer. Use `in` for a plain existence check, and reserve `index()` (§5.5) for when the actual position is also needed.

### 5.5 List Methods

Lists carry built-in **methods** — functions attached to the list object and called with dot syntax, `my_list.method(...)`. The everyday set falls into three groups [1]:

**Add elements:**
- `append(x)` — adds `x` as a single new element at the end.
- `insert(i, x)` — inserts `x` so it lands at index `i`, shifting later elements one position to the right.
- `extend(iterable)` — adds *each* item from another sequence to the end, one by one.

**Remove elements:**
- `remove(x)` — deletes the first element equal to `x` (raises `ValueError` if `x` is not present).
- `pop(i)` — removes **and returns** the element at index `i`; with no argument, removes and returns the last element.
- `clear()` — removes every element, leaving `[]`.

**Search and count (these never modify the list):**
- `index(x)` — returns the index of the first element equal to `x` (raises `ValueError` if not found).
- `count(x)` — returns how many times `x` appears.

```python
nums = [1, 2, 3]
nums.append(4)         # [1, 2, 3, 4]
nums.insert(0, 99)     # [99, 1, 2, 3, 4]
nums.extend([5, 6])    # [99, 1, 2, 3, 4, 5, 6]
last = nums.pop()      # last = 6, nums = [99, 1, 2, 3, 4, 5]
```

A subtle but important distinction: `nums.append([5, 6])` adds the list `[5, 6]` as **one** nested element, while `nums.extend([5, 6])` unpacks it, adding `5` and `6` as two separate elements. All six methods above that add or remove elements act **in place** — this is mutability in action — and all of them return `None`, except `pop()`, which returns the removed value.

### 5.6 Sorting: `sort()` versus `sorted()`

Python offers two distinct ways to order a list, and the difference between them matters a great deal:

| Aspect | `list.sort()` | `sorted(list)` |
|---|---|---|
| What it is | A **method** called on the list itself | A **built-in function**, given the list as an argument |
| Return value | `None` | A brand-new sorted list |
| Effect on original | Rearranges the list **in place** | Leaves the original list completely untouched |

```python
nums = [3, 1, 2]
nums.sort()                   # nums is now [1, 2, 3]; sort() returns None

original = [3, 1, 2]
result = sorted(original)     # result == [1, 2, 3]; original is still [3, 1, 2]
```

A frequent bug is writing `nums = nums.sort()` — because `sort()` returns `None`, this line silently destroys the list. Use `sort()` when the previous order no longer matters; use `sorted()` when you still need the original order elsewhere in the program.

Both accept the same two keyword arguments: `reverse=True` sorts largest-to-smallest, and `key=` (already introduced with functions in Unit 2.4) takes a function that Python applies to every element first, sorting by the function's *result* rather than the raw value:

```python
words = ["banana", "apple", "kiwi"]
print(sorted(words, reverse=True))   # ['kiwi', 'banana', 'apple']
print(sorted(words, key=len))        # ['kiwi', 'apple', 'banana'] — shortest to longest
```

`key=` becomes especially useful once a list holds nested lists (§5.7): sorting a list of `[name, mark]` pairs by the mark, not by the name, means the key function must reach into each inner list and pull out the second value:

```python
students = [["Aarav", 70], ["Diya", 92], ["Kabir", 85]]
by_mark = sorted(students, key=lambda s: s[1], reverse=True)
print(by_mark)   # [['Diya', 92], ['Kabir', 85], ['Aarav', 70]]
```

Here `lambda s: s[1]` (the anonymous-function syntax from Unit 2.4) tells `sorted()` to compare each inner list by its mark (index `1`) rather than by the whole `[name, mark]` pair.

### 5.7 Nested Lists

A list element can itself be a list — this is called a **nested list**, and it is a natural way to model a grid, a table, or rows of related data (a spreadsheet of marks, a set of passenger records). The first index selects the "row"; a second index then reaches inside that row:

```python
grid = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print(grid[0])       # [1, 2, 3]  — the first row, itself a list
print(grid[0][2])    # 3          — row 0, then column 2 inside that row

for row in grid:
    for value in row:
        print(value, end=" ")
    print()
```

Each inner list is a complete list in its own right — every indexing, slicing, and method operation covered above already works on it. Visiting every value in a nested list means nesting one `for` loop inside another, exactly as shown: the outer loop walks the rows, the inner loop walks the values within each row.

A common use of a nested list is pairing a label with a value, one small list per record:

```python
students = [["Aarav", 70], ["Diya", 74], ["Kabir", 85]]

for student in students:
    name, mark = student          # tuple unpacking, from Unit 1.4
    print(f"{name}: {mark}")
```

`for student in students` visits one `[name, mark]` inner list at a time, and `name, mark = student` unpacks its two values into two separate names in a single step — clearer to read than reaching in twice with `student[0]` and `student[1]`.

### 5.8 List Comprehensions

A **list comprehension** builds a new list from an existing sequence in one expression, replacing the longer "create an empty list, loop, and `append`" pattern. Read the general form `[expression for item in iterable]` left to right: "the expression, for each item in the iterable." A comprehension can map, filter, or do both together [1]:

- **Map** (transform every item): `[n * n for n in range(5)]` → `[0, 1, 4, 9, 16]`.
- **Filter** (add an `if` clause to keep only qualifying items): `[n for n in range(10) if n % 2 == 0]` → `[0, 2, 4, 6, 8]`.
- **Both together**: `[n * 2 for n in nums if n % 2 == 0]` transforms and selects in a single pass.

```python
nums = [1, 2, 3, 4, 5, 6]
doubled_evens = [n * 2 for n in nums if n % 2 == 0]
print(doubled_evens)   # [4, 8, 12]
```

Reading it step by step: `for n in nums` walks `1, 2, 3, 4, 5, 6` one at a time; `if n % 2 == 0` keeps only `2, 4, 6`, silently dropping the rest; `n * 2` is the expression applied to each survivor, giving `4, 8, 12`. A comprehension always returns a fresh list and never touches the source sequence.

Comprehensions and nested lists (§5.7) combine naturally: a comprehension can appear *inside* another comprehension to build a grid in one line, instead of writing nested `for` loops:

```python
grid = [[row * 3 + col for col in range(3)] for row in range(3)]
print(grid)   # [[0, 1, 2], [3, 4, 5], [6, 7, 8]]
```

The outer comprehension (`for row in range(3)`) produces one row per iteration; for each row, the inner comprehension (`for col in range(3)`) produces the three values of that row. The result is the same shape of nested list introduced in §5.7, just built in a single expression rather than with an explicit loop and repeated `append` calls.

### 5.9 Common Mistakes

Certain list mistakes come up so often for beginners that it is worth naming each one explicitly, alongside the fix:

- **`IndexError: list index out of range`** — accessing a position that does not exist, most often the classic off-by-one slip of assuming the last valid index is `len(my_list)` instead of `len(my_list) - 1`.
- **Off-by-one confusion in slicing** — forgetting that `stop` is excluded, so `my_list[0:3]` returns 3 elements (indices `0, 1, 2`), not 4; and misreading `my_list[1:3]` as "elements 1 through 3" instead of "elements 1 up to but not including 3."
- **Mutating a list while iterating over it** — calling `remove()` or `pop()` inside a `for` loop that is walking the same list can silently skip elements, because the list shifts under the loop's feet as items disappear. Loop over a copy (`for x in my_list[:]:`) or build a fresh list with a comprehension instead.
- **Assuming `new_list = old_list` makes a copy** — it only creates a second name for the very same list (§5.4); changes made through either name affect both. Use `.copy()` or `old_list[:]` when an independent list is actually intended.
- **Writing `my_list = my_list.sort()`** — this destroys the list entirely, because `sort()` always returns `None` (§5.6).
- **Confusing `append()` with `extend()`** — `my_list.append([1, 2])` adds one nested list as a single new element; `my_list.extend([1, 2])` adds `1` and `2` as two separate elements (§5.5).

## 6. Implementation

Three short, reusable procedures come up constantly once you start working with lists.

**How to safely remove items while looping (avoiding the "shifting list" bug):**
1. Never call `remove()` or `pop()` directly inside a `for` loop that is walking the very same list — the list shifts under the loop as items disappear, and elements can be silently skipped.
2. Instead, either loop over an explicit copy (`for x in my_list[:]:`), or build a brand-new list of only the items you want to keep — most cleanly with a list comprehension (§5.8).
3. Reassign the result back to the original name if you need the filtered version to replace the original list.

**How to read a slice expression `a[start:stop:step]` (mental checklist):**
1. Start at index `start` (default `0` if omitted).
2. Keep taking elements while the index is *before* `stop` (default `len(a)` if omitted) — `stop` itself is never included.
3. Move `step` positions each time (default `1`); a negative `step` walks backward through the list, which is how `a[::-1]` reverses it.
4. Collect whatever elements were visited into a brand-new list; if no elements matched, the result is `[]`, never an error.

**How to turn a manual loop into a list comprehension (a habit worth building):**
1. Start from the longhand version: create an empty list, `for` loop over the source iterable, and `append()` one computed value per iteration, optionally behind an `if` check.
2. Identify the three moving parts in that loop: the expression being appended, the loop variable and source iterable, and (if present) the filtering condition.
3. Reassemble them in the order `[expression for loop_variable in iterable if condition]`, dropping the `if condition` entirely when there is no filtering.
4. Verify by comparing the comprehension's result to the original loop's result on the same input — they should be identical, since a comprehension is exactly this loop pattern written as a single expression, not a different behavior.

**Worked walkthrough — a teacher tracking five quiz marks, using every skill from this unit in one running scenario:**

```python
marks = [78, 85, 92, 66, 74]

marks[3] = 70                 # correct a mis-entered mark (mutability, §5.4)
marks.append(88)              # a 6th mark arrives late (§5.5)
marks.remove(78)              # the first student was disqualified (§5.5)
marks.sort()                  # rank the remaining marks ascending, in place (§5.6)

toppers = [m for m in marks if m >= 85]   # who scored 85 or above (§5.8)

print(marks)     # [70, 74, 85, 88, 92]
print(toppers)   # [85, 88, 92]
```

Reading this top to bottom: the index assignment fixes one value in place without rebuilding the list; `append` and `remove` grow and shrink the list in place; `sort()` reorders it in place and returns `None`, so its result is never reassigned back to `marks`; and the final comprehension walks the now-sorted `marks`, keeping only values `85` or above, producing a brand-new list without disturbing `marks` itself.

**A second walkthrough — a shopping-cart update that keeps a ranked view separate from the working list [1]:**

A cart holds `cart_prices = [199, 349, 99, 599, 249]`. A newly chosen item (`149`) must be added, an unwanted item (`99`) must be removed, and the program must report the total of the three most expensive remaining items without disturbing the cart's own order.

```python
cart_prices = [199, 349, 99, 599, 249]

cart_prices.append(149)      # add the new item, in place
cart_prices.remove(99)       # drop the unwanted item, in place

top_three = sorted(cart_prices, reverse=True)[:3]   # a new, ranked list
total_top_three = sum(top_three)

print(cart_prices)       # [199, 349, 599, 249, 149]
print(top_three)         # [599, 349, 249]
print(total_top_three)   # 1197
```

The key detail: `sorted(cart_prices, reverse=True)` builds a brand-new list ranked highest-to-lowest and never touches `cart_prices` itself; slicing `[:3]` off that ranked list, then summing, gives the answer while `cart_prices` keeps its own, unsorted, append-and-remove order for every other part of the program that still needs it.

## 7. Real-World Patterns

- **AI/ML pipelines (this module's focus):** a batch of prompts sent to a model, the batch of responses that come back, and the batch of scores used to grade those responses are, in nearly every case, plain Python lists — this is exactly why Module III exists, and every later lab in this course will pass lists like these between functions.
- **Shopping carts and billing:** an e-commerce cart is a list of item prices; `sorted(cart, reverse=True)[:3]` instantly gives the three most expensive items, and a comprehension applies a discount or a tax to every item in one line.
- **Records and statements:** a bank statement or a gradebook is a list of numeric records; sorting ranks them, and filtering with a comprehension isolates the ones that matter (failed transactions, passing scores).
- **Batch processing systems:** a payment or booking system collects pending records into a list before processing them together, then uses a comprehension to pull out just the failed or pending ones for a retry pass.
- **Tabular or grid data:** a nested list is the simplest way to represent a small table, a grid of sensor readings, or a set of passenger records — accessed with two indices, `grid[row][col]` — before a more specialized structure is introduced later in this module.
- **Everyday scripting:** any time a program reads several lines from a file, several rows from an API response, or several answers from a form, the natural first container to hold them in is a list, because the count is rarely known in advance.

Note the boundary of this topic: iterating over a list manually with a loop and `.append()`/`.pop()` calls is one way of processing a sequence of values one at a time; Python also has a more general mechanism for stepping through a sequence lazily, called an **iterator** — that mechanism, along with the wider `collections` module toolbox mentioned in this module's overview, is covered later in Module III, not here.

## 8. Best Practices

- Prefer a list comprehension over a manual "empty list + loop + `append`" pattern whenever the transformation is simple enough to read on one line.
- Use `sorted()`, not `sort()`, whenever the original unsorted list is still needed later in the program.
- Use `.copy()` or `a[:]` explicitly whenever an independent list is intended — never assume `b = a` produces a separate list.
- Prefer `if x in my_list:` for a simple membership check, and reserve `index()`/`count()` for when the actual position or frequency is needed.
- When both position and value are needed in a loop, use `enumerate(my_list)` (Unit 2.2) instead of manually managing `range(len(my_list))`.
- Give lists plural, descriptive names (`student_names`, `cart_prices`) so it is obvious at a glance that the variable holds many values, not one.
- Reach for the in-place methods (`append`, `remove`, `sort`, and the rest) when the existing list itself should change; reach for `sorted()` or a slice or a comprehension when a *new* list should be produced and the original left alone — deciding this up front avoids most of the bugs in §5.9.
- When a list holds structured records (like the `["Aarav", 70]` name-and-mark pairs in §5.7), unpack them with a descriptive tuple-unpacking assignment (`name, mark = student`, from Unit 1.4) inside the loop rather than repeatedly indexing with `student[0]` and `student[1]` — it reads far more clearly.

## 9. Hands-On Exercise

Given `marks = [55, 90, 40, 85, 60]`:
1. Sort `marks` ascending and print the highest mark using indexing (not `max()`).
2. `append(95)` for a late entrant and `remove(40)` for a no-show, then print the updated list.
3. Build a comprehension called `passers` containing only marks `>= 60`, and print how many students passed using `len(passers)`.
4. *Stretch:* wrap each mark in a `[student_id, mark]` pair (a nested list) and re-sort by mark using `sorted(records, key=lambda r: r[1])`, to practice the `key=` pattern from §5.6 on nested data.

## 10. Key Takeaways

- A **list** is Python's ordered, mutable collection written with `[ ]`; elements are reached with a zero-based **index**, `0` for the first and `-1` for the last, and an out-of-range index raises `IndexError`.
- **Slicing** (`my_list[start:stop:step]`) always returns a new list, never raises `IndexError`, and always excludes `stop`; `a[::2]` and `a[::-1]` are idioms worth memorizing.
- Because lists are mutable, `b = a` creates an **alias**, not a copy — use `.copy()` or `a[:]` for an independent list, remembering that a shallow copy of a nested list still shares its inner lists with the original.
- In-place methods (`append`, `insert`, `extend`, `remove`, `pop`, `clear`, `sort`) modify the list directly and return `None`; `sorted()` and slices always return a brand-new list instead.
- A **nested list** models rows of related data via two indices (`grid[row][col]`), and a **list comprehension** (`[expr for item in iterable if condition]`) maps and/or filters a sequence into a new list in one line.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
