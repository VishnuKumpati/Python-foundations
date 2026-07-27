# Lists

Module II left you with real power: you could make a program decide with `if`/`elif`/`else`, repeat work with loops, and package a calculation into a reusable function that other code could call without caring how it was built. Unit 2.5 rounded that off by showing you how to split your own code across modules and lean on professional tooling instead of one giant notebook. Every one of those skills, though, was still built around a single value at a time — one price, one name, one flag, one return value handed back from one function call.

Real problems rarely hand you one value. A quiz has thirty scores, not one. A shopping cart has however many items a customer chose to add. A railway booking holds a whole coach of passengers, not a single seat. You could try `score1`, `score2`, `score3`, and so on, but that idea collapses the moment the count isn't known in advance — which, in practice, is almost always. Module III exists to solve exactly this: it introduces the four core data structures Python gives you for holding *many* values under one name, and this unit opens with the one you will use the most, by far — the **list**.

By the time you finish this unit, "many values, one name, in order, changeable as the program runs" will be something you can create, search, slice, reorder, and build in a single line — not just a phrase.

---

## One variable per value doesn't scale

Picture storing the marks of five students the way Unit 1.2 taught you: `mark1 = 78`, `mark2 = 85`, `mark3 = 92`, `mark4 = 66`, `mark5 = 74`. That technically works. Now picture a class of thirty, or a Swiggy-style order with however many items a customer happened to add tonight, or a batch of UPI transactions arriving throughout the day — a count nobody can predict in advance.

| System | What it actually needs to hold |
|---|---|
| Quiz grading | However many students sat the quiz, in one ordered group |
| Swiggy-style cart | However many items a customer adds before checkout |
| UPI batch settlement | However many transactions arrive before the next settlement run |
| IRCTC-style booking | Every passenger travelling on one PNR, as a group |

Every row needs the same thing: one name that holds *several* values, in order, that can grow or shrink while the program runs. That is precisely what a **list** gives you — Python's built-in, ordered, changeable collection of values, written as comma-separated items inside square brackets:

```python
marks = [78, 85, 92, 66, 74]
print(marks)
print(type(marks))
```

```
[78, 85, 92, 66, 74]
<class 'list'>
```

Five values, one name — no `mark1` through `mark5` needed. "Ordered" means the values stay exactly where you placed them; Python never reshuffles a list on its own. A list can hold any type you met back in Unit 1.2, and even mix them in the same list:

```python
mixed = ["Ananya", 25, True, 3.14]
empty = []
```

`empty` is a perfectly valid list with zero elements — you will build on it constantly once you reach `append()` later in this unit. Each individual value sitting inside a list is called an **element**, and `len(marks)` tells you how many elements it holds — `5` here — without you counting by hand.

## Reaching into a list: indexing

Every element in a list sits at a numbered position called its **index**, and Python always starts counting from `0`, never `1` — the same rule you already met indexing strings by in Unit 1.4. Write the list's name followed by an index in square brackets to read the single value sitting there:

```python
fruits = ["apple", "banana", "cherry"]
print(fruits[0])
print(fruits[2])
```

```
apple
cherry
```

Before checking, predict what `fruits[1]` prints, and then what happens if you try `fruits[3]` on this same three-item list. The first is easy — `banana` — but the second raises an **`IndexError`**, because a three-element list only has valid positions `0`, `1`, and `2`. For a list of length `n`, the last valid position is always `n - 1`, never `n` — forgetting that "off by one" is one of the most common beginner slips with lists.

Python also lets you count backward from the end using **negative indexing**: `-1` is the last element, `-2` the second-to-last, and so on — sparing you from writing `fruits[len(fruits) - 1]` every time you need the final item.

```python
print(fruits[-1])
print(fruits[-3])
```

```
cherry
apple
```

**A three-element list has exactly six valid index values — `0, 1, 2` counting forward and `-3, -2, -1` counting backward — and nothing outside that range.** Try it yourself: for `fruits` above, work out on paper which positive index and which negative index both point at `"banana"`, then check your answer in a Colab cell.

## Slicing: taking a chunk instead of one element

Indexing gets you one element. **Slicing** gets you a whole chunk at once, described by up to three numbers separated by colons — `start` (included), `stop` (excluded), and `step` (how many positions to jump each time) — written `my_list[start:stop:step]`. Slicing always hands back a **brand-new list**; the original is never touched.

```python
a = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
print(a[0:2])
print(a[1:5])
print(a[:3])
print(a[7:])
```

```
[0, 1]
[1, 2, 3, 4]
[0, 1, 2]
[7, 8, 9]
```

Leaving out `start` defaults it to `0`; leaving out `stop` defaults it to `len(a)`. Notice `stop` is always excluded — `a[1:5]` stops just before index `5`, giving you four elements, not five. Before checking, predict how many elements `a[0:3]` returns, and whether index `3` itself is included — it isn't, and getting that habit right early avoids a whole category of off-by-one bugs later.

Two slicing idioms are worth memorising outright, because they show up constantly in real code:

```python
print(a[::2])
print(a[::-1])
```

```
[0, 2, 4, 6, 8]
[9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```

`a[::2]` walks the whole list taking every second element; `a[::-1]` walks it backward, which is the standard one-line way to reverse a list. Negative numbers work exactly the same way inside a slice as they do inside a single index — a genuinely handy shortcut for "the last few items," the kind of thing you'd reach for pulling the three most recent UPI transactions off a running list:

```python
scores = [55, 60, 70, 85, 90, 95]
print(scores[-3:])
print(scores[:-2])
```

```
[85, 90, 95]
[55, 60, 70, 85]
```

**Unlike indexing, slicing never raises an `IndexError` for an out-of-range `start` or `stop` — Python simply clamps to whatever is available, even returning `[]` when nothing matches.** That forgiving behaviour is one of the few places lists are more relaxed than indexing suggests they'd be.

## Mutability: a list changes after it's born

Back in Unit 1.2, reassigning a variable — `total = 250` and then `total = 300` — always replaced the old value entirely with a new one. A list works differently: it is **mutable**, meaning its contents can be changed after creation, in place, without building a new list from scratch.

```python
colors = ["red", "green", "blue"]
colors[1] = "yellow"
print(colors)
```

```
['red', 'yellow', 'blue']
```

**Mutability is the single defining fact that separates lists from the `int`, `float`, `str`, and `bool` values you met in Unit 1.2 — those never change in place; a list can, and constantly does.** Assigning to a single index like this replaces exactly that element and leaves every other position untouched.

## Aliasing: the same list, two names

Mutability has a consequence that catches almost every learner at least once. A variable holding a list doesn't hold a private copy of its values — it holds a reference to where that list lives in memory. Assign one list variable to another name, and both names become **aliases**: two labels pointing at the exact same underlying list.

```python
a = [1, 2, 3]
b = a
b.append(4)
print(a)
```

```
[1, 2, 3, 4]
```

Before checking, predict this for yourself: only `b` was touched with `.append(4)` — does `a` change too? It does, because `a` and `b` were never two separate lists; `b = a` never copied anything, it just gave the one existing list a second name. This is exactly why UPI apps, banking software, and every list-heavy system you'll touch later treat "who else has a reference to this list" as a genuinely important question, not a technicality.

| | Simple values (`int`, `str`, `bool`) | Lists |
|---|---|---|
| `b = a` produces | A completely independent value | An alias — a second name for the same list |
| Changing through `b` | Never affects `a` | Affects `a` too, because there's only one list |
| Getting an independent copy | Automatic, every time | Requires `.copy()` or `a[:]` explicitly |

To get a genuinely independent list, use a full slice (`b = a[:]`) or the `.copy()` method (`b = a.copy()`). Both produce a **shallow copy** — a new outer list, whose *elements* are copied by reference. For a list of plain numbers or strings this behaves just like a fully independent copy. **For a nested list (you'll meet these shortly), a shallow copy only duplicates the outer list — the inner lists inside it are still shared with the original**, a subtlety that causes real, hard-to-trace bugs once your data gets more structured.

Aliasing matters just as much the moment a list is passed as an argument to a function. The parameter receives a reference to the *same* list, not a copy, so any in-place method the function calls is visible to the caller once the function returns — even with no `return` statement at all:

```python
def add_bonus_mark(marks):
    marks.append(100)

class_marks = [78, 85, 92]
add_bonus_mark(class_marks)
print(class_marks)
```

```
[78, 85, 92, 100]
```

`add_bonus_mark` never returned anything, yet `class_marks` changed anyway — because `marks` inside the function and `class_marks` outside it were always the same list. Expect this the moment you understand mutability and aliasing together; it only feels surprising the first time.

## Checking if something's already there: `in`

Sometimes the only question you actually have is "does this value already exist in the list?" — not where. The `in` operator answers exactly that, returning a plain `bool`:

```python
fruits = ["apple", "banana", "cherry"]
print("banana" in fruits)
print("mango" in fruits)
print("mango" not in fruits)
```

```
True
False
True
```

This is a different use of `in` from the `for x in iterable:` loop syntax you met in Unit 2.2 — here it's a standalone expression producing one immediate answer, not something driving repetition. Reach for `in` for a plain existence check (has this order ID already been processed?), and save `.index()`, coming up next, for when you also need the position.

## List methods: adding, removing, and searching

Lists carry built-in **methods** — functions attached to the list itself, called with dot syntax. The everyday ones fall into three natural groups.

**Adding elements:**
- `append(x)` — adds `x` as one new element at the end.
- `insert(i, x)` — inserts `x` so it lands at index `i`, shifting everything after it one step right.
- `extend(iterable)` — adds *each* item from another sequence to the end, one by one.

**Removing elements:**
- `remove(x)` — deletes the first element equal to `x` (raises `ValueError` if it isn't present).
- `pop(i)` — removes **and returns** the element at index `i`; with no argument, removes and returns the last element.
- `clear()` — removes every element, leaving `[]`.

**Searching (these never modify the list):**
- `index(x)` — returns the position of the first element equal to `x` (raises `ValueError` if absent).
- `count(x)` — returns how many times `x` appears.

```python
nums = [1, 2, 3]
nums.append(4)
nums.insert(0, 99)
nums.extend([5, 6])
last = nums.pop()
print(nums)
print(last)
```

```
[99, 1, 2, 3, 4, 5]
6
```

**`nums.append([5, 6])` adds the list `[5, 6]` as one single nested element; `nums.extend([5, 6])` unpacks it, adding `5` and `6` as two separate elements — mixing these two up is one of the most common list bugs beginners write.** All six of the add/remove methods above act **in place** and return `None`, except `pop()`, which hands back the value it removed — reflect for a second on why that makes sense: `pop()` deletes *and* gives you the departing value in one step, precisely so you can act on it immediately.

## Two ways to sort: `sort()` versus `sorted()`

Python gives you two distinct ways to put a list in order, and mixing them up causes a genuinely famous beginner bug.

| | `list.sort()` | `sorted(list)` |
|---|---|---|
| What it is | A method, called on the list itself | A built-in function, given the list as an argument |
| Return value | `None` | A brand-new sorted list |
| Effect on the original | Rearranges it in place | Leaves the original completely untouched |

```python
nums = [3, 1, 2]
nums.sort()
print(nums)

original = [3, 1, 2]
result = sorted(original)
print(result)
print(original)
```

```
[1, 2, 3]
[1, 2, 3]
[3, 1, 2]
```

**Writing `nums = nums.sort()` is a real, frequent bug — because `sort()` always returns `None`, this line silently throws the entire list away.** Use `sort()` once the old order genuinely no longer matters anywhere else in your program; use `sorted()` the moment you still need that original order — say, an e-commerce cart that must keep its "items in the order added" view while also showing a "most expensive first" view somewhere else on the same screen.

Both accept the same two keyword arguments: `reverse=True` sorts largest-to-smallest, and `key=` — which you met briefly with functions in Unit 2.4 — takes a function Python applies to every element first, sorting by that function's *result* rather than the raw value:

```python
words = ["banana", "apple", "kiwi"]
print(sorted(words, reverse=True))
print(sorted(words, key=len))
```

```
['kiwi', 'banana', 'apple']
['kiwi', 'apple', 'banana']
```

Try predicting `sorted(words, key=len, reverse=True)` yourself before running it — reasoning through both keyword arguments together is exactly the kind of check that confirms you actually understand `key=`, rather than having memorised one example.

## Nested lists: rows inside rows

A list element can itself be a list — a **nested list** — and it is the natural, everyday way to model a grid, a table, or a batch of related records, like a set of IRCTC-style passenger entries or rows of sensor readings. The first index selects the "row," a second index reaches inside that row:

```python
grid = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
print(grid[0])
print(grid[0][2])
```

```
[1, 2, 3]
3
```

Every inner list is a complete list in its own right — everything covered in this unit already works on it. Visiting every value means nesting one `for` loop inside another: the outer loop walks the rows, the inner loop walks the values within each row.

```python
for row in grid:
    for value in row:
        print(value, end=" ")
    print()
```

```
1 2 3 
4 5 6 
7 8 9 
```

A frequent, practical shape for a nested list is pairing a label with a value — one small list per record, such as a student and their mark:

```python
students = [["Aarav", 70], ["Diya", 74], ["Kabir", 85]]

for student in students:
    name, mark = student
    print(f"{name}: {mark}")
```

```
Aarav: 70
Diya: 74
Kabir: 85
```

`name, mark = student` is tuple unpacking, from Unit 1.4, applied here to split each two-item inner list into two separate names in a single line — reading far more clearly than reaching in twice with `student[0]` and `student[1]`. This same nested shape is exactly what makes `key=` genuinely powerful: sorting `students` by mark, not name, means the key function must reach one level in:

```python
by_mark = sorted(students, key=lambda s: s[1], reverse=True)
print(by_mark)
```

```
[['Kabir', 85], ['Diya', 74], ['Aarav', 70]]
```

## List comprehensions: the loop-and-append pattern in one line

By now you've likely written the pattern "create an empty list, loop over something, `append` a computed value" more than once. A **list comprehension** compresses exactly that pattern into a single expression: `[expression for item in iterable]`, read left to right as "the expression, for each item in the iterable."

A comprehension can **map** (transform every item), **filter** (keep only some, with an `if` clause), or do both at once:

```python
squares = [n * n for n in range(5)]
evens = [n for n in range(10) if n % 2 == 0]
print(squares)
print(evens)
```

```
[0, 1, 4, 9, 16]
[0, 2, 4, 6, 8]
```

```python
nums = [1, 2, 3, 4, 5, 6]
doubled_evens = [n * 2 for n in nums if n % 2 == 0]
print(doubled_evens)
```

```
[4, 8, 12]
```

Trace it the way you would trace any new piece of syntax the first time: `for n in nums` walks `1` through `6` one at a time; `if n % 2 == 0` silently drops every odd value, keeping only `2, 4, 6`; `n * 2` is then applied to each survivor, giving `4, 8, 12`. Before checking, predict what `[n for n in nums if n > 3]` would produce on this same `nums` — a comprehension always builds a fresh list and never disturbs the source sequence.

**A list comprehension is not a different behaviour from a manual loop with `append()` — it is exactly that same loop, written as one expression instead of several lines.** Comprehensions and nested lists combine naturally too, letting you build a small grid in one line instead of writing two nested `for` loops:

```python
grid = [[row * 3 + col for col in range(3)] for row in range(3)]
print(grid)
```

```
[[0, 1, 2], [3, 4, 5], [6, 7, 8]]
```

## Lists in the real world

Almost every batch of data you'll touch for the rest of this course arrives as a list. In the AI labs later in this programme, a batch of prompts sent to a model, the batch of responses it returns, and the scores used to grade them are, in nearly every case, plain Python lists passed between functions exactly the way Unit 2.3 taught you to pass any argument. A Swiggy-style cart is a list of item prices, where `sorted(cart, reverse=True)[:3]` instantly surfaces the three priciest items. A bank statement or a gradebook is a list of numeric records, filtered down to failed transactions or passing scores with a single comprehension. A UPI settlement batch collects pending transactions into a list before processing them together, then uses a comprehension to isolate the ones that failed and need a retry.

A short set of mistakes is worth watching for deliberately while lists are still new:

- Assuming `new_list = old_list` makes an independent copy — it only creates a second name for the same list; use `.copy()` or `old_list[:]` when you genuinely need two separate lists.
- Writing `my_list = my_list.sort()`, which destroys the list because `sort()` always returns `None`.
- Confusing `append()` with `extend()` when adding another sequence's contents.
- Forgetting that the last valid positive index is `len(my_list) - 1`, not `len(my_list)`.
- Mutating a list (`remove()` or `pop()`) while a `for` loop is walking that very same list, which can silently skip elements as the list shifts underneath the loop — loop over a copy (`for x in my_list[:]:`) or build a fresh filtered list with a comprehension instead.

## Try it yourself

Do this in a Colab cell before moving on. Start with `marks = [55, 90, 40, 85, 60]`. Sort it ascending and print the highest mark using indexing, not `max()`. Then `append(95)` for a late entrant and `remove(40)` for a no-show, and print the updated list. Build a comprehension called `passers` holding only marks `>= 60`, and print how many students passed with `len(passers)`. Finally, as a stretch, wrap each mark in a `[student_id, mark]` pair — a nested list — and re-sort the whole thing by mark using `sorted(records, key=lambda r: r[1])`, to practise `key=` on nested data one more time before moving on.

---

### Key Terminology

- **List** — Python's built-in, ordered, mutable collection of values, written with `[ ]`.
- **Element** — one individual value stored inside a list.
- **Index** — the zero-based numbered position of an element; negative indices count backward from the end.
- **`IndexError`** — raised when an index does not exist in the list.
- **Slice** — a sub-section of a list, extracted with `[start:stop:step]` as a brand-new list.
- **Mutability** — the property of being changeable in place after creation, without building a new object.
- **Alias** — a second name referring to the exact same list, not an independent copy.
- **Shallow copy** — an independent outer list (via `.copy()` or `a[:]`) whose inner, nested lists may still be shared with the original.
- **`in`** — the operator that tests whether a value exists in a list, returning a `bool`.
- **List method** — a built-in function attached to a list, called with dot syntax (`append`, `remove`, `sort`, and others).
- **`list.sort()`** — sorts a list in place and returns `None`.
- **`sorted()`** — a built-in function that returns a new sorted list, leaving the original untouched.
- **Nested list** — a list containing other lists as its elements, accessed with two indices (`grid[row][col]`).
- **List comprehension** — the one-line form `[expression for item in iterable if condition]` that builds a new list from an existing sequence.

### Mastery Checkpoint

Before moving to Unit 3.2, check that you can answer these without looking back:

1. Why does `fruits[3]` raise an `IndexError` on a three-element list, and what are the six valid index values — positive and negative — for that same list?
2. What does `a[2:7:2]` actually do, and why does slicing never raise an `IndexError` even when `start` or `stop` is out of range?
3. `b = a` followed by `b.append(4)` changes `a` too. Why, and what would you write instead if you genuinely wanted an independent copy?
4. What is the key difference between `list.sort()` and `sorted(my_list)`, and what specific bug does `my_list = my_list.sort()` cause?
5. Given a manual loop that builds a new list with `append()` inside an `if` check, what are the three parts you'd identify to rewrite it as a single list comprehension?

### Summary

You now know how to hold many values under one name instead of one value per variable: creating a list, reading it by positive or negative index, and slicing out a chunk of it with `start:stop:step` without ever touching the original. You have seen why mutability makes lists fundamentally different from the simple types in Unit 1.2, why `b = a` creates an alias rather than a copy, and why that same aliasing means a function can change a list you passed it with no `return` needed at all. You have used the everyday list methods to add, remove, and search; sorted a list two different ways with `sort()` and `sorted()`; indexed into nested lists to model rows of data; and compressed the "empty list, loop, append" pattern into a single list comprehension. From here, the next step is meeting a second core data structure that looks similar at first glance but makes one very different promise — Unit 3.2, Tuples.

### Additional Resources

- [Python Tutorial — official docs: "Data Structures" (lists, list comprehensions)](https://docs.python.org/3/tutorial/datastructures.html)
- [Python 3 Documentation — Common Sequence Operations](https://docs.python.org/3/library/stdtypes.html#common-sequence-operations)
- [Python 3 Documentation — Mutable Sequence Types (list methods)](https://docs.python.org/3/library/stdtypes.html#mutable-sequence-types)
- [Python 3 Documentation — `sorted()` built-in function](https://docs.python.org/3/library/functions.html#sorted)
- [W3Schools — Python Lists](https://www.w3schools.com/python/python_lists.asp)
- [W3Schools — Python List Comprehension](https://www.w3schools.com/python/python_lists_comprehension.asp)
- [W3Schools — Python List Methods](https://www.w3schools.com/python/python_lists_methods.asp)
