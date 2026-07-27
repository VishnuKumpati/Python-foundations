# Tuples

Unit 3.1 introduced the list — an ordered, mutable collection. Not every group of values should be changeable, though: some things, like a date of birth or GPS coordinates, are meant to stay fixed once created. Store your date of birth as a list, `[15, 8, 2005]`, and nothing in the language stops some other part of your program from quietly overwriting the day with `20`, or appending a stray fourth number by mistake. You wouldn't have "the same birthday, slightly corrected" — you'd have a different date entirely, silently, with no error raised anywhere.

Python's answer to this problem looks almost identical to a list on the surface — you index into it, slice it, loop over it, check membership with `in` — but it enforces one deliberate restriction that a list does not: once built, it can never be changed. That structure is the **tuple**, and the restriction is not a limitation to work around; it is the entire reason the tuple exists. Every time a function hands back more than one value at once — a coordinate pair, a status code alongside a message, three test scores from evaluating a model — a tuple is doing that job, usually without anyone even naming it out loud.

By the end of this chapter, "should this be a list or a tuple?" will be a question you can answer in one breath, and you'll have built, unpacked, and deliberately broken a tuple yourself so the immutability guarantee stops being an abstract rule and starts being something you've actually felt.

---

## Why a list isn't always the right tool

Think about a handful of everyday systems and what they actually need to store as *one fixed unit*:

| System | Fixed group of values that belong together |
|---|---|
| UPI payment app | A completed transaction's ID, status, and timestamp, handed back together |
| Ride-hailing / logistics app | A GPS reading — latitude and longitude, always as a pair |
| Railway booking (IRCTC-style) | A seat allotment — coach and seat number, never one without the other |
| Banking app | An account number paired with its IFSC code |
| AI/ML evaluation pipeline | A model's accuracy, precision, and recall, returned from one evaluation run |

Notice what every row has in common: the values are *related*, *fixed in number*, and meant to travel together without one part being edited while the rest stays the same. A list would technically hold any of these, but it would also silently allow a bug where someone appends an extra GPS coordinate, or overwrites just the IFSC code and leaves the account number pointing at a different bank's confirmation. Pick one row above and say out loud, in one sentence, what could quietly go wrong if that data were stored in a list instead of something locked — that's precisely the gap a tuple closes.

## Creating a tuple

A **tuple** is an ordered, immutable sequence of values. "Ordered" means exactly what it meant for a list: every value sits at a fixed position, called an **index**, starting at `0`. "Immutable" is the new idea — once a tuple is built, none of its elements can be added, removed, or replaced, ever, for the rest of its life.

You write a tuple as comma-separated values, usually wrapped in parentheses:

```python
point = (3, 5)
rgb = (255, 128, 0)
person = ("Ada", 36, True)
```

Run this cell and — predict it before you check — nothing prints at all. These three lines only create and store tuples; nothing here asks Python to display anything. Just like a list, a single tuple can freely mix types: `person` holds a string, an integer, and a boolean side by side, with no complaint from Python.

Here is the detail almost every learner misses at least once: **the comma is what actually creates the tuple — not the parentheses.** Try this yourself before reading on:

```python
a = 1, 2, 3
b = (1, 2, 3)
print(a == b)
```

```
True
```

Both lines produce the exact same tuple. The parentheses exist mainly for readability, and are only strictly required in a few places — an empty tuple, `()`, or when a tuple is passed directly as a function argument.

That fact about commas leads straight into the single most common tuple bug in the entire language: **a single-element tuple must carry a trailing comma.**

```python
single = (42,)
not_a_tuple = (42)

print(type(single))
print(type(not_a_tuple))
```

```
<class 'tuple'>
<class 'int'>
```

`(42,)` is a tuple holding one value. `(42)` is nothing more than the integer `42` sitting inside a pair of ordinary, redundant parentheses — the parentheses here mean grouping, exactly like `(2 + 3) * 4`, not "tuple." This is a genuine quirk of Python's grammar rather than something you can reason your way to from first principles, so it has to be memorised, and it is worth memorising deliberately now rather than discovering it while debugging.

You can also build a tuple out of any existing iterable — a list, a string, anything you can loop over — using the built-in `tuple()` function:

```python
letters = tuple("Pune")
print(letters)
```

```
('P', 'u', 'n', 'e')
```

**`tuple()` takes exactly one argument, and that argument must itself be an iterable.** `tuple([1, 2, 3])` works and produces `(1, 2, 3)`; `tuple(1, 2, 3)` raises a `TypeError`, because `tuple()` was never designed to accept several separate values — only one thing to iterate over.

## Immutability: what it forbids, and why

**Immutability** means an object's contents cannot change after it is created. For a tuple, that guarantee is total: there is no `append()`, no `remove()`, no `sort()`, no equivalent of any list method that edits contents in place. In fact, a tuple exposes exactly two methods, `count()` and `index()`, and both only *read* the tuple — neither one changes it. That short list is itself a visible expression of the immutability guarantee: there is simply nothing else a tuple is allowed to do to itself.

```python
student_record = ("STU2026047", "Ananya Rao", "CSE")

student_record[1] = "Ananya R. Rao"
```

```
TypeError: 'tuple' object does not support item assignment
```

**Attempting `my_tuple[index] = new_value` always raises a `TypeError` — a tuple has no mechanism at all for replacing one of its own elements.** Before checking, predict what would happen if you tried `student_record.append("2026")` on the same tuple instead — you would get an `AttributeError` this time, not a `TypeError`, because `append` isn't a method a tuple has in the first place; it isn't even a legal thing to try, let alone a legal thing to do.

Why enforce this at all? A list's flexibility is a genuine feature when data is expected to change — a shopping cart, a running list of scores. But that same flexibility becomes a liability the moment data is *not* supposed to change: any piece of code holding a reference to a mutable list can alter it, deliberately or by accident, and every other piece of code sharing that same list sees the change too. A tuple removes that risk entirely. Once you receive one — say, back from a function call — you can trust it will still look exactly the same later, no matter how many other parts of the program are holding onto it. This is precisely why a function that needs to hand back more than one value almost always packages them as a tuple: the caller can trust the group of values will not be silently altered somewhere between the `return` and the moment it's actually used.

So how do you "change" a tuple, if you genuinely need to? You never edit it in place — you build a completely new tuple with the corrected values and rebind the variable name to point at that new tuple:

```python
fare_record = ("PNR4821093", 1250.00)
fare_record = (fare_record[0], 1500.00)
print(fare_record)
```

```
('PNR4821093', 1500.0)
```

The original `("PNR4821093", 1250.00)` tuple isn't edited anywhere — it's simply discarded once `fare_record` is reassigned to point at a brand-new tuple.

## Unpacking: spreading a tuple into variables

**Unpacking** is spreading a tuple's values into separate variables in a single assignment statement:

```python
booking = ("PNR4821093", "Ananya Sharma", 1250.00)
pnr, passenger_name, fare = booking
print(pnr)
print(passenger_name)
print(fare)
```

```
PNR4821093
Ananya Sharma
1250.0
```

Python binds each name on the left to the value in the matching position on the right, all in one statement. **The number of variable names on the left must exactly match the number of values on the right.**

```python
a, b = (1, 2, 3)
```

```
ValueError: too many values to unpack (expected 2)
```

Before checking, predict what error message you'd get running `a, b, c, d = (1, 2, 3)` instead — you'd see the opposite complaint, "not enough values to unpack," because now there are too *few* values for the variables waiting to receive them.

Unpacking is also exactly what powers Python's classic no-temporary-variable swap:

```python
a = 10
b = 20
a, b = b, a
print(a, b)
```

```
20 10
```

Python first packs the right-hand side into a temporary tuple, `(b, a)`, i.e. `(20, 10)`, then unpacks that tuple straight back into `a` and `b` in their new order — all in one statement, with no separate temporary variable that you have to manage yourself.

When you don't know the exact number of values in advance, or only the first one matters with "everything else" collected separately, **star-unpacking** handles it:

```python
semester_marks = (78, 82, 85, 90, 91)
first_semester, *later_semesters = semester_marks
print(first_semester)
print(later_semesters)
```

```
78
[82, 85, 90, 91]
```

`first_semester` takes the first value, `78`, and `*later_semesters` scoops up every remaining value into a brand-new **list**, however many values there turn out to be — notice that the star-collected portion comes back as a list, not a tuple, since its size is flexible.

## Nested tuples

A **nested tuple** is a tuple that contains another tuple as one of its own elements:

```python
student = ("Ananya Rao", "CSE", ("Mysuru", "Karnataka"))
print(student[2])
print(student[2][0])
```

```
('Mysuru', 'Karnataka')
Mysuru
```

Before checking, predict what the second `print()` produces. `student[2]` retrieves the whole nested tuple, `("Mysuru", "Karnataka")`. `student[2][0]` uses **chained indexing**: the first `[2]` reaches the nested tuple, and the second `[0]` reaches inside *that* tuple to its first element — the city, `Mysuru`.

One subtlety about immutability deserves precision here: **immutability only locks the outer tuple's own slots — it says nothing about the mutability of whatever those slots point to.** Consider a record that deliberately mixes a tuple with a nested list:

```python
record = ("Ada", [90, 85])
record[1].append(78)
print(record)
```

```
('Ada', [90, 85, 78])
```

`record` itself cannot have its two slots reassigned — `record[0] = "Grace"` would still raise a `TypeError`. But the list sitting inside slot `1` is still a perfectly ordinary, mutable list, and `.append()` edits it in place without touching the outer tuple's structure at all. The tuple only guarantees "these two slots will always point at the same two objects" — it says nothing about whether those objects can change themselves.

## Basic operations every tuple supports

Beyond indexing and slicing — which follow exactly the same rules you already know from lists, including negative indices and `start:stop:step` slicing — a tuple supports a small, fixed set of read-only operations:

```python
semester_marks = (78, 82, 85, 90, 91)
semester_6_marks = (88, 84)

all_marks = semester_marks + semester_6_marks
print(all_marks)

placeholder = (0,) * 3
print(placeholder)

print(90 in all_marks)
print(all_marks.count(84))
print(all_marks.index(85))
```

```
(78, 82, 85, 90, 91, 88, 84)
(0, 0, 0)
True
1
2
```

- **Concatenation** (`+`) joins two tuples into a brand-new, longer tuple — neither original tuple is touched.
- **Repetition** (`*`) repeats a tuple's contents a given number of times — a quick way to build a fixed-size placeholder, like three zero-scores for a newly admitted student who hasn't sat an exam yet.
- **Membership testing** (`in`) checks whether a value appears anywhere in the tuple, returning `True` or `False`.
- **`count()`** reports how many times a given value appears.
- **`index()`** reports the position of the *first* occurrence of a given value, raising `ValueError` if it isn't found at all.

Two tuples can even be compared directly with `<` and `>`, using **lexicographic comparison** — element by element, left to right, exactly the way words are ordered in a dictionary: `(1, 2) < (1, 3)` is `True`, because the first elements tie and the second pair decides it.

## Why a tuple can be a dictionary key and a list never can

An object is **hashable** if its value can never change over its lifetime, which lets Python compute one stable, reusable hash value for it — and only a hashable object can be used as a dictionary key or stored inside a set (both covered properly in later units, but the underlying idea belongs here). Because a tuple's contents can never change once created, Python can safely compute a hash for it: **a tuple is hashable, a list never is, and that single fact is one of the most common Python interview questions there is.**

There's a sharp exception worth remembering, though: a tuple is only hashable if *every* element inside it is also hashable. `("Ananya", "CSE")` is hashable — both elements are themselves immutable strings. `("Ada", [90, 85])` is **not** hashable, because it contains a list, and a list is never hashable, no matter what tuple it's sitting inside. Try predicting, before checking, whether `("STU2026047", ("Mysuru", "Karnataka"))` is hashable — it is, because a nested tuple of strings is itself hashable, and hashability only breaks down the moment something genuinely mutable, like a list, enters the chain.

## List or tuple — deciding in one question

| | List | Tuple |
|---|---|---|
| Brackets | Square brackets `[1, 2, 3]` | Parentheses (or just commas) `(1, 2, 3)` |
| Changeable after creation? | Yes | No |
| Methods available | Many — `append()`, `remove()`, `sort()`, and more | Only two — `count()`, `index()` |
| Usable as a dictionary key / set element | Never | Yes, if every element inside is also hashable |
| Typical use case | A collection expected to grow, shrink, or reorder | A fixed group of related values, or a function's bundled return values |

**Everything above collapses to one question: will this collection's values or membership ever need to change over the life of the program?** A "yes" points straight at a list. A fixed, related group that should always look the same once created points straight at a tuple. A bank confirming a fund transfer, for instance, hands back something like `("TXN783420", "HDFC0001234", 15000.00, "SUCCESS")` — the caller unpacks it into `txn_id, ifsc, amount, status`, checks `status == "SUCCESS"`, and trusts that none of those four values will quietly change underneath it later. That single shape — a fixed, ordered group of facts, returned together and never edited — is exactly what you'll recognise again and again: a GPS reading as `(latitude, longitude)`, a railway seat allotment as `(coach, seat_number)`, a model's evaluation scores as `(accuracy, precision, recall)`.

A short list of mistakes worth watching for deliberately while this is still new:

- Leaving off the trailing comma on a single-element tuple — `(42)` is just an `int`, not a tuple.
- Trying to edit a tuple in place with `my_tuple[0] = ...` or `.append()`, instead of building a new tuple and rebinding the name.
- Unpacking into the wrong number of variables, raising a `ValueError` about too many or too few values.
- Calling `tuple()` with several separate arguments instead of one iterable — `tuple(1, 2, 3)` is a `TypeError`.
- Assuming immutability reaches inside a nested mutable object, when it only actually locks the outer tuple's own slots.

## Try it yourself

Do this in a Colab cell before moving on. Model a railway ticket confirmation: create a tuple `booking` holding a PNR string, a passenger name, a seat allotment stored as a nested tuple `(coach, seat_number)`, and a fare as a float. Unpack `booking` into four variables, then unpack the nested seat tuple into two more, and print every value. Next, wrap an attempt to change the fare with `booking[3] = ...` inside a `try`/`except TypeError` block, and print the error message it produces. Finally, "update" the fare correctly by building a brand-new tuple with the changed value and rebinding `booking` to it, then print the new fare to confirm the change actually took effect.

---

### Key Terminology

- **Tuple** — an ordered, immutable sequence of values, typically written with parentheses.
- **Packing** — writing several values together, separated by commas, to build a tuple.
- **Immutability** — the property that an object's contents cannot be changed after creation.
- **Unpacking** — spreading a tuple's values into separate variables in one assignment statement.
- **Star-unpacking** — using `*name` to collect a variable number of extra values into a list during unpacking.
- **Nested tuple** — a tuple that contains another tuple as one of its elements.
- **Chained indexing** — using two or more `[]` accesses in a row, such as `outer[2][0]`, to reach a value inside a nested structure.
- **`tuple()`** — the built-in function that builds a tuple from a single iterable argument.
- **Hashable** — an object whose value can never change, allowing Python to use it as a dictionary key or set element.
- **Lexicographic comparison** — comparing two sequences element by element, left to right, the way words are ordered in a dictionary.
- **`TypeError`** — the error raised when code attempts an operation, such as item assignment, that a tuple does not support.
- **`ValueError`** — the error raised when unpacking a tuple into the wrong number of variables.

### Mastery Checkpoint

Before moving to Unit 3.3, check that you can answer these without looking back:

1. What actually creates a tuple in Python — the parentheses or the comma — and how does that explain why `(42)` is an `int` while `(42,)` is a tuple?
2. Why does `my_tuple[0] = "new value"` always raise a `TypeError`, and what does the fact that a tuple has only two methods have to do with that?
3. A tuple contains a nested list, like `("Ada", [90, 85])`. Can the list be edited in place? Can the tuple's own slots be reassigned? Why is the answer different for each?
4. Why is a tuple hashable while a list never is, and what's the one exception that can make an otherwise "hashable-looking" tuple not hashable after all?
5. Given a piece of data you need to store, what single question decides whether it should be a list or a tuple?

### Summary

You now know what a tuple is and how it differs from the list you met in Unit 3.1: an ordered sequence built by the comma, not the parentheses, that locks its own contents the moment it's created and exposes only two read-only methods, `count()` and `index()`, as proof of that guarantee. You've unpacked tuples into variables, including the no-temporary-variable swap and star-unpacking, worked with nested tuples and chained indexing, and seen precisely where immutability's guarantee stops — at the tuple's own slots, not at whatever mutable object might be sitting inside one of them. You've also used concatenation, repetition, membership testing, and the hashability that makes a tuple usable as a dictionary key, and you can now decide confidently, in one question, whether a given piece of data calls for a list or a tuple. From here, the next step is meeting a collection built around a different guarantee entirely — that every value inside it is unique — starting with Unit 3.3, Sets.

### Additional Resources

- [Python Tutorial — official docs: "Tuples and Sequences"](https://docs.python.org/3/tutorial/datastructures.html#tuples-and-sequences)
- [Python 3 Documentation — Built-in Types: Sequence Types (list, tuple, range)](https://docs.python.org/3/library/stdtypes.html#sequence-types-list-tuple-range)
- [Python 3 Documentation — `tuple()` built-in function](https://docs.python.org/3/library/functions.html#func-tuple)
- [Python 3 Documentation — Glossary: hashable](https://docs.python.org/3/glossary.html#term-hashable)
- [W3Schools — Python Tuples](https://www.w3schools.com/python/python_tuples.asp)
- [W3Schools — Python Unpacking Tuples](https://www.w3schools.com/python/python_tuples_unpack.asp)
