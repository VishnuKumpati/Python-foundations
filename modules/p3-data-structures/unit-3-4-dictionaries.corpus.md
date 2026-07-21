---
topic_id: 3695f9f8-f7e3-4f7e-933d-95006fced9d1
title: Unit 3.4 - Dictionaries
position_in_module: 4
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 3.4 - Dictionaries — Topic Corpus

## 2. Prerequisites

- Unit 3.1 - Lists (`fe27250e-c2c2-491c-afd7-57d9b533deab`) — dictionaries are compared against lists throughout this unit, so you should already be comfortable with indexing, mutability, and list comprehensions.
- Unit 3.2 - Tuples (`18ffde38-766c-4912-bd6a-bfd0e238a750`) — tuples are one of the valid dictionary key types, and the idea of immutability introduced there is reused directly here.
- Unit 3.3 - Sets (`7cb62f0c-485f-4798-a6bb-390a3f1a8542`) — sets introduced hashing and the concept of a hashable value; a dictionary key must be hashable in exactly the same sense.

## 3. Learning Objectives

- **Explain** what a dictionary is, why every key inside it must be unique and hashable, and how a dictionary differs from a list and a set.
- **Create** a dictionary using a dictionary literal or the `dict()` constructor, and **access** a value safely with `[]` and with `.get()`.
- **Apply** assignment, `del`, and `.pop()` to add, modify, and delete key-value pairs.
- **Apply** the three looping styles — `.keys()`, `.values()`, `.items()` — and sort a dictionary's contents by key or by value.
- **Explain** how a nested dictionary models a real record with sub-fields, and access a value that sits two or more levels deep.
- **Create** a dictionary comprehension to build a new dictionary from an iterable.

## 4. Introduction

Think about your own college mark sheet. It is not just a pile of numbers — every number has a label attached to it: "Maths: 88," "Roll No: 101," "Name: Ananya." A list, which you met in Unit 3.1, only stores values in order — it has no built-in way to say "this particular value is the roll number and that one is the marks." A set, from Unit 3.3, does not even preserve order, and it only cares about whether a value is present or not. Neither structure captures the idea of "this label points to this value."

That gap is exactly what a **dictionary** fills. A dictionary is Python's built-in way of storing **key-value pairs** — a label (the key) paired with the data it refers to (the value) — so that instead of asking "what is stored at position 2?" you ask "what value belongs to the label `'marks'`?" This is also, not by coincidence, the shape that data takes almost everywhere outside your own code: a bank account record, a UPI payment, a product listing on an e-commerce site, and a response from a web API are all naturally a set of named fields — which is precisely what a dictionary represents [1].

By the end of this unit you will be able to build a dictionary, read and change what is inside it safely, loop through it in three different styles, nest one dictionary inside another to model a real record, and build a brand-new dictionary in a single line with a dictionary comprehension. Every one of these skills is something you will use again the moment this course starts working with real records — whether that record is a student, a bank transaction, or, later on, the input and output of an AI model.

## 5. Core Concepts

### 5.1 What a dictionary is

A **dictionary** (Python type `dict`) is a mutable collection of **key-value pairs**, where each **key** is unique and is used to look up the **value** it is paired with [1]. "Mutable" means it can be changed after it is created — the same meaning the word has had since lists were introduced in Unit 3.1. You write a dictionary with curly braces `{}`, a colon between each key and its value, and commas between pairs:

```python
student = {"name": "Ananya", "roll_no": 101, "marks": 87}
```

Here `"name"`, `"roll_no"`, and `"marks"` are keys; `"Ananya"`, `101`, and `87` are the values paired with them. `"name": "Ananya"` together is called one **key-value pair**. A dictionary is Python's built-in example of a **mapping** — the general term for any type that connects keys to values [1].

### 5.2 Why a dictionary instead of a list or a set

Suppose you tried to store the same student record using two separate lists — one for field names, one for values:

```python
fields = ["name", "roll_no", "marks"]
values = ["Ananya", 101, 87]
```

To find the marks you would first have to find the *position* of `"marks"` inside `fields`, then use that same position to look inside `values`. This is fragile: if one list is ever updated without updating the other, the two lists fall out of sync and your data becomes silently wrong, with nothing built into Python to catch the mistake [1]. A dictionary avoids this entirely by storing the label and its value **together, as a single unit**, so there is never any doubt about which value belongs to which key. This is why dictionaries are the natural fit for records (a student, a bank account), lookup tables (translating one thing into another), and counting or grouping data keyed by the thing being counted [1].

The table below summarizes how the three collections you now know compare:

| Aspect | List | Set | Dict |
|---|---|---|---|
| Access by | position, `lst[i]` | membership only (`in`) | **key**, `d[key]` |
| Ordering | insertion order | unordered | insertion order (since Python 3.7) |
| Duplicates | allowed | never | no duplicate **keys** (values may repeat) |
| Written with | `[ ]` | `{ }` / `set()` | `{key: value}` |

### 5.3 Keys must be unique and hashable

A dictionary can never hold the same key twice. If you assign to a key that already exists, Python overwrites its value rather than creating a second entry.

A key must also be **hashable** — a term introduced in Unit 3.3 for set elements and reused here with the exact same meaning: a value is hashable if Python can compute a fixed "fingerprint" for it that never changes for as long as the value exists. Strings, numbers, and tuples are hashable and so are valid dictionary keys. A list is *not* hashable, because a list can be changed after it is created — its fingerprint could not stay fixed — so trying to use a list as a key raises `TypeError: unhashable type: 'list'`. Values, by contrast, can be absolutely anything — a number, a string, a list, or even another dictionary — and two different keys are allowed to point to the same value.

### 5.4 Creating and accessing a dictionary

Besides writing a dictionary literal directly, you can build one with the `dict()` constructor, including from an iterable of `(key, value)` pairs.

There are two ways to read a value out of a dictionary:

```python
d = {"a": 1, "b": 2}
value = d["a"]           # bracket access — raises KeyError if the key is missing
value = d.get("c", 0)    # safe access — returns 0 instead of raising an error
```

- `d[key]` — **bracket access**, using the same square brackets you used for list indexing in Unit 3.1, except the "index" is now a key rather than a position. It raises a **`KeyError`** — a new error type specific to dictionaries — if `key` is not present.
- `d.get(key, default)` — the `.get()` method reads the value for `key`, or returns `default` (or `None`, if you leave `default` out) instead of raising an error when the key is missing. Use `[]` when a missing key would mean a genuine bug you want surfaced immediately; use `.get()` whenever the key is optional, such as a field that might or might not be present in data coming from outside your program [1].

### 5.5 Adding, modifying, and deleting entries

A dictionary has no separate "insert" syntax — adding and modifying both use the same assignment form:

```python
d["c"] = 3        # "c" is new -> adds a third key-value pair
d["a"] = 99        # "a" already exists -> overwrites its value
del d["b"]         # removes the key "b" and its value entirely
```

- `d[key] = value` — if `key` is not already in the dictionary, this **adds** it; if `key` already exists, this **overwrites** its old value. This is the same assignment operator (`=`) from Unit 1.2, now used to write into a dictionary rather than to a plain variable.
- `del d[key]` — the `del` statement removes a key and its value completely; it raises `KeyError` if the key does not exist.
- `d.pop(key, default)` — removes `key` and *returns* its value, which is useful when you need the value on your way out; it returns `default` instead of raising an error if you supply one.
- `d.update(other)` — adds every key-value pair from another dictionary `other` into `d`, overwriting any keys that already exist in both. This is a quick way to merge in several new or changed fields at once instead of assigning them one at a time.
- `d.setdefault(key, default)` — returns the value already stored for `key` if it exists; otherwise it inserts `key: default` into the dictionary and returns `default`. It combines a check-and-insert into a single step.

You can also build a dictionary with the `dict()` constructor instead of a literal — for example, `dict(name="Ananya", roll_no=101)` produces the same dictionary as `{"name": "Ananya", "roll_no": 101}`, using keyword arguments (Unit 2.3) as the keys.

### 5.6 Looping over a dictionary three ways

A `for` loop, from Unit 2.2, can traverse a dictionary in three distinct styles, each returning a different **view object** — an object that stays "live" and reflects later changes to the dictionary, rather than a fixed, independent list:

```python
for key in d.keys():
    print(key)

for value in d.values():
    print(value)

for key, value in d.items():
    print(key, "->", value)
```

- `.keys()` visits only the keys.
- `.values()` visits only the values, with no key attached.
- `.items()` visits both together, as `(key, value)` tuples — a use of tuple unpacking from Unit 3.2 to split each pair into `key` and `value` in one step. This is the style you should reach for whenever you need both the key and its value together [1].

A very common beginner mix-up: writing `for x in d:` on its own gives you the **keys**, not the values — if you print `x` expecting a value and get a key instead, this is almost always why.

### 5.7 Sorting a dictionary

`sorted(d)` sorts the dictionary's **keys** and returns them as a new list — it does not touch the dictionary itself. To sort by *value* instead, sort `d.items()` and tell `sorted()` to look at the second element of each pair using a `lambda`, the anonymous-function syntax from Unit 2.4:

```python
sorted(d.items(), key=lambda kv: kv[1])              # ascending, by value
sorted(d.items(), key=lambda kv: kv[1], reverse=True) # descending, by value
```

`sorted()` never changes the original dictionary — it always returns a new list, exactly as it did for lists in Unit 3.1.

### 5.8 Nested dictionaries

A **nested dictionary** is a dictionary whose value is itself another dictionary — the natural way to model a record that has its own sub-fields [1]:

```python
students = {
    101: {"name": "Ananya", "marks": {"maths": 88, "science": 92}},
    102: {"name": "Rohit", "marks": {"maths": 65, "science": 70}},
}
print(students[101]["marks"]["maths"])   # 88
```

Here the outer dictionary's keys are roll numbers; each value is a smaller dictionary holding `name` and a further nested dictionary, `marks`. Reaching a deeply nested value means **chaining bracket lookups**: first into the outer dictionary by roll number, then into the resulting record by `"marks"`, then into that by the subject name. Chaining as many brackets as there are nested levels is the general pattern for any depth of nesting.

### 5.9 Dictionary comprehensions

A **dictionary comprehension** is a one-line expression that builds a new dictionary from an iterable, following the same comprehension idea introduced for lists in Unit 3.1 and sets in Unit 3.3, but producing `key: value` pairs instead of single elements:

```python
{key: value for item in iterable}
```

For example, starting from the `students` dictionary above, this builds a new dictionary mapping roll number to name, keeping only students whose maths score is above 70:

```python
top = {
    roll_no: record["name"]
    for roll_no, record in students.items()
    if record["marks"]["maths"] > 70
}
```

This loops over every `(roll_no, record)` pair from `.items()`, keeps only the ones matching the `if` condition, and stores `record["name"]` against `roll_no` in a brand-new dictionary called `top` — the original `students` dictionary is left completely unchanged.

## 6. Implementation

A common task is building a dictionary-based **lookup table** and then using it to enrich data — for example, turning a list of state codes into full state names. The general procedure:

1. **Build the lookup dictionary once**, mapping each short code to its full value, e.g. `codes = {"MH": "Maharashtra", "KA": "Karnataka", "TN": "Tamil Nadu"}`.
2. **For each input value**, look it up with `.get(code, "Unknown")` rather than `codes[code]`, so an unexpected code never crashes the program with a `KeyError`.
3. **Collect the results** — either by printing them in a loop, or by building a second dictionary or list out of the looked-up values.
4. **If the input itself is a nested structure** (say, a list of student records, each a dictionary), chain the lookup inside a loop over `.items()` or over the list, reading whichever field you need with bracket access or `.get()`.
5. **If you need the final result summarized rather than printed row by row**, reach for a dictionary comprehension (§5.9) instead of a manual loop that builds up a new dictionary one assignment at a time — it expresses the same steps in one line and makes the filtering condition, if any, explicit.

This same shape — build once, look up many times, fall back safely on a miss — underlies almost every use of a dictionary as a lookup table, whether the lookup is a handful of state codes or a large data set loaded at the start of a program.

## 7. Real-World Patterns

Dictionaries are the single most-used non-trivial data structure in production Python code, because so much real data is naturally a set of named fields rather than a plain sequence [1]:

- **Banking and payments** — an account record (account number, holder name, balance) or a UPI transaction (payer, payee, amount, status) is naturally a dictionary; a bank's customer database is conceptually a dictionary of dictionaries keyed by account number [1].
- **E-commerce** — a product catalog is a dictionary keyed by product ID; a shopping cart is a dictionary mapping product ID to quantity.
- **Records with sub-fields** — a patient record, a student information system, or a railway booking (PNR, passenger name, seat, fare) is exactly the nested-dictionary pattern from §5.8 [1].
- **APIs and cloud services** — request and response data is almost always exchanged as a dictionary in Python, matching the JSON format used by nearly every web API; this connection becomes central once this course reaches API integration later in the program [1].
- **Storing prompts, results, and scores** — later labs in this course that work with AI models store a prompt's text, its generated result, and a score or rating together as one dictionary record, keyed by an ID, exactly as the `wallet_history` and `students` examples in this unit key a record by transaction ID or roll number [1]. The nested-dictionary and dictionary-comprehension patterns from §5.8–5.9 are the same patterns you will reuse to filter and summarize those records.

## 8. Best Practices

- Use `.get(key, default)` instead of `d[key]` whenever a key might genuinely be missing — it avoids an unhandled `KeyError` crashing your program [1].
- Choose one consistent spelling for a key name (`"roll_no"`, not sometimes `"roll_no"` and sometimes `"rollNumber"`) — a typo in a key name is one of the most common real dictionary bugs [1].
- Prefer looping with `.items()` over looping `.keys()` and then re-looking-up each value with `d[k]` — it is both cleaner and avoids a second lookup [1].
- Keep a dictionary's values consistent in shape where possible — for example, every student record having the same set of fields — so code that processes many of them does not need special cases [1].
- Do not assume a missing key returns `None` silently — only `.get()` does that; direct bracket access raises `KeyError` [1].

## 9. Hands-On Exercise

Using a small dictionary of three products, each mapping a product ID (key) to a nested dictionary of `name`, `price`, and `stock` (value): (1) add a new product with assignment; (2) use `.get()` to safely read a `"discount"` field that does not exist, falling back to `0`; (3) loop with `.items()` and print only products where `stock` is less than 5; (4) write a dictionary comprehension mapping product ID to `name`, keeping only products priced above 500; (5) use `sorted()` on the same dictionary's items to print all three products ordered from highest price to lowest, without changing the original dictionary.

## 10. Key Takeaways

- A dictionary is a mutable collection of unique, hashable keys mapped to values; `d[key]` raises `KeyError` if the key is missing, while `d.get(key, default)` returns a fallback instead.
- `d[key] = value` adds a new key or overwrites an existing one; `del d[key]` removes a key; both operate in place without rebuilding the dictionary.
- The three looping styles are `.keys()`, `.values()`, and `.items()`; `for k, v in d.items()` is the cleanest way to work with both the key and value together, and a bare `for x in d:` gives keys, not values.
- `sorted(d)` sorts keys into a new list; sorting by value requires `sorted(d.items(), key=lambda kv: kv[1])` — neither form changes the original dictionary.
- A nested dictionary — a dictionary whose value is itself a dictionary — models a real record with sub-fields, reached by chaining bracket lookups such as `students[101]["marks"]["maths"]`.
- A dictionary comprehension, `{key: value for item in iterable}`, builds a new dictionary in one line, commonly filtered with an `if` clause.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
