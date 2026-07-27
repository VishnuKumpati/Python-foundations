# Dictionaries

Lists, tuples, and sets (Units 3.1-3.3) all organise values by position or by simple membership. But a lot of real data doesn't come as a bare position-based sequence at all — it comes as a label attached to a value: a name mapped to a phone number, a product code mapped to a price, a student ID mapped to a grade. Try storing that kind of data in a list and you're left tracking, by hand, which position corresponds to which label — position `0` is the name, position `1` is the roll number, position `2` is the marks — with nothing in Python stopping those two facts from drifting apart the moment one list changes and the other doesn't.

A set doesn't help either — it doesn't even preserve a value's position, and it only answers "is this here or not," never "what belongs to this." What every one of these examples actually needs is a structure built around exactly one idea: this label points to this value, permanently and unambiguously. That structure is the **dictionary**, and it is arguably the single most-used data structure in real Python code, because so much real-world data — a bank account, a UPI transaction, a product listing, a JSON response from a web API — is naturally a set of named fields rather than a plain ordered sequence.

By the end of this unit you'll be able to build a dictionary, read and change what's inside it safely, loop through it in three distinct styles, nest one dictionary inside another to model a real multi-field record, and build a brand-new dictionary in a single line with a dictionary comprehension.

---

## The gap a list and a set both leave open

Picture your own college mark sheet. It isn't just a pile of numbers sitting in some order — every number has a label glued to it: "Roll No: 101," "Name: Ananya," "Maths: 88." Now picture a handful of other everyday records and notice they all have exactly the same shape:

| Record | Labels attached to values |
|---|---|
| UPI transaction | payer, payee, amount, status |
| Bank account | account number, holder name, balance |
| E-commerce product | product ID, name, price, stock |
| Railway booking (IRCTC-style) | PNR, passenger name, seat, fare |

In every row, you don't want "the third value in some sequence" — you want "the value labelled `balance`," regardless of where it happens to sit. A dictionary is Python's built-in way of storing exactly this: **key-value pairs**, where a **key** (the label) is paired with the **value** (the data it refers to), so that instead of asking "what's stored at position 2?" you ask "what value belongs to the key `'marks'`?" Pick one row above and say out loud what would go wrong if you tried to track it using two separate lists — one for labels, one for values — kept in matching order by hand.

## Building your first dictionary

Think of a dictionary the way you'd think of an actual phone book, or a labelled filing cabinet drawer. You don't find a name in a phone book by reading every entry from the top until you stumble on the right one — you jump straight to the name and read off the number beside it. A dictionary works the same way: hand it a key, and it hands back the value filed under that key directly, with no scanning required.

You write a dictionary with curly braces `{}`, a colon between each key and its value, and commas between pairs:

```python
student = {"name": "Ananya", "roll_no": 101, "marks": 87}
print(student)
```

```
{'name': 'Ananya', 'roll_no': 101, 'marks': 87}
```

Here `"name"`, `"roll_no"`, and `"marks"` are keys; `"Ananya"`, `101`, and `87` are the values paired with them. `"name": "Ananya"` together is one key-value pair. Python's general term for any type that connects keys to values this way is a **mapping** — a dictionary is Python's built-in mapping type, `dict`.

You can also build one with the `dict()` constructor, passing each key as a keyword argument — the same keyword-argument syntax from Unit 2.3:

```python
student = dict(name="Ananya", roll_no=101, marks=87)
print(student)
```

```
{'name': 'Ananya', 'roll_no': 101, 'marks': 87}
```

Both lines above build the identical dictionary. The literal form (`{...}`) is what you'll reach for almost all the time; the `dict()` form reads slightly more naturally when every key happens to be a valid Python identifier written without quotes.

## Why not just two lists, or a set

Suppose you tried to avoid dictionaries entirely and stored the same student record as two parallel lists instead:

```python
fields = ["name", "roll_no", "marks"]
values = ["Ananya", 101, 87]
```

To find the marks, you'd first have to find the *position* of `"marks"` inside `fields`, then use that same position to look inside `values`. That works — right up until someone inserts a new field into one list and forgets to touch the other, and now position 2 in `fields` no longer matches position 2 in `values`. Nothing in Python warns you; your data has just gone silently wrong. A dictionary sidesteps the entire problem by storing the label and its value **together, as one unit**, so the question "which value belongs to which key" never has a chance to come apart.

| Aspect | List | Set | Dict |
|---|---|---|---|
| Access by | position, `lst[i]` | membership only (`in`) | **key**, `d[key]` |
| Ordering | insertion order | unordered | insertion order (since Python 3.7) |
| Duplicates | values may repeat | never repeats | no duplicate **keys** (values may repeat) |
| Written with | `[ ]` | `{ }` or `set()` | `{key: value}` |

## The rule every key must obey: unique and hashable

A dictionary can never hold the same key twice. Assign to a key that already exists, and Python quietly overwrites its old value rather than creating a second entry beside it — there is no error, and no warning; the previous value for that key is simply gone.

A key must also be **hashable** — the exact same term, with the exact same meaning, that you met for set elements in Unit 3.3: a value is hashable if Python can compute a fixed "fingerprint" for it that never changes for as long as the value exists. Strings, numbers, and tuples are all hashable, so all three are valid dictionary keys. A list is **not** hashable, because a list can change after it's created, so its fingerprint could never stay fixed — try to use one as a key and Python raises `TypeError: unhashable type: 'list'`.

**Values, by contrast, can be absolutely anything at all — a number, a string, a list, even another dictionary — and two different keys are perfectly free to point to the same value.** Only the *key* side of a dictionary has to be unique and hashable; the value side has no such restriction whatsoever.

## Reading a value: `[]` versus `.get()`

There are two ways to read a value back out of a dictionary, and they behave very differently the moment a key is missing:

```python
wallet = {"upi_id": "ananya@upi", "balance": 2500}

print(wallet["balance"])
print(wallet.get("balance"))
print(wallet.get("cashback", 0))
```

```
2500
2500
0
```

- `d[key]` — **bracket access**, the same square brackets you used for list indexing in Unit 3.1, except the "index" is now a key rather than a position. If `key` isn't present, Python raises a **`KeyError`** — a new error type specific to dictionaries, and one you will meet constantly the moment you mistype a key name.
- `d.get(key, default)` — reads the value for `key`, or returns `default` (or `None` if you leave `default` out entirely) instead of raising an error when the key is missing.

Before checking, predict what `wallet["cashback"]` would do on its own, without `.get()`. It raises `KeyError: 'cashback'`, because `wallet` has never held that key.

**Use `[]` when a missing key would mean a genuine bug you want surfaced immediately; use `.get()` whenever the key is genuinely optional — a field from an outside source, like a web API response, that might or might not be present.** Reaching for `[]` everywhere and hoping nothing is ever missing is exactly how a `KeyError` ends up crashing a program in production.

## Adding, modifying, and deleting entries

A dictionary has no separate "insert" syntax — adding a brand-new key and overwriting an existing one both use the exact same assignment form you already know from Unit 1.2:

```python
wallet = {"upi_id": "ananya@upi", "balance": 2500}

wallet["cashback"] = 50        # "cashback" is new -> adds a third pair
wallet["balance"] = 2550        # "balance" already exists -> overwrites it
del wallet["cashback"]          # removes "cashback" and its value entirely

print(wallet)
```

```
{'upi_id': 'ananya@upi', 'balance': 2550}
```

- `d[key] = value` — if `key` isn't already present, this **adds** a new pair; if it is, this **overwrites** the old value in place.
- `del d[key]` — removes a key and its value completely; it raises `KeyError` if that key doesn't exist, exactly like bracket access does when reading.
- `d.pop(key, default)` — removes `key` and *returns* its value in one step, which matters when you need that value on your way out; supply a `default` and it's returned instead of raising an error if the key is missing.
- `d.update(other)` — merges every key-value pair from another dictionary `other` into `d`, overwriting any keys that already exist in both, in one call instead of one assignment per field.
- `d.setdefault(key, default)` — returns the value already stored for `key` if it exists; otherwise it inserts `key: default` and returns `default`. It's a check-and-insert combined into a single step.

**Assigning to a dictionary changes it in place — nothing is being rebuilt or reordered behind the scenes; the same object simply gains, loses, or updates one entry.** Try predicting, before you run it, what `wallet.pop("upi_id")` returns *and* what `wallet` looks like immediately afterwards — the return value is the string `'ananya@upi'`, and `wallet` is left holding only `balance`.

## Looping over a dictionary, three ways

A `for` loop, from Unit 2.2, can walk through a dictionary in three distinct styles, each handing back a different **view** of the same underlying data:

```python
inventory = {"pen": 40, "notebook": 12, "eraser": 3}

for item in inventory.keys():
    print(item)

for count in inventory.values():
    print(count)

for item, count in inventory.items():
    print(item, "->", count)
```

```
pen
notebook
eraser
40
12
3
pen -> 40
notebook -> 12
eraser -> 3
```

- `.keys()` visits only the keys.
- `.values()` visits only the values, with no key attached at all.
- `.items()` visits both together, as `(key, value)` tuples — split apart into `item` and `count` in one step using tuple unpacking, exactly as you did in Unit 3.2. This is the style to reach for whenever you genuinely need the key *and* its value together.

**Writing `for x in inventory:` on its own gives you the keys, not the values — this is one of the most common dictionary mix-ups a beginner makes.** If you print `x` expecting `40` and instead see `'pen'`, this bare-loop behaviour is almost always why. Before moving on, predict what `for item in inventory:` alone would print — it behaves identically to looping over `.keys()`.

## Sorting a dictionary

`sorted(d)` sorts a dictionary's **keys** and hands back a brand-new list — the dictionary itself is left completely untouched:

```python
inventory = {"pen": 40, "notebook": 12, "eraser": 3}
print(sorted(inventory))
```

```
['eraser', 'notebook', 'pen']
```

Sorting by *value* instead takes one more step: sort `inventory.items()` and tell `sorted()` to look at the second element of each pair using a `lambda`, the anonymous-function syntax from Unit 2.4:

```python
by_stock = sorted(inventory.items(), key=lambda kv: kv[1])
print(by_stock)

by_stock_desc = sorted(inventory.items(), key=lambda kv: kv[1], reverse=True)
print(by_stock_desc)
```

```
[('eraser', 3), ('notebook', 12), ('pen', 40)]
[('pen', 40), ('notebook', 12), ('eraser', 3)]
```

`kv[1]` picks out the value half of each `(key, value)` tuple, so `sorted()` orders the pairs by stock count rather than by item name. **`sorted()` never changes the original dictionary — it always hands back a new list, exactly as it did for lists back in Unit 3.1.** If you need `inventory` itself sorted, you would have to rebuild it from this sorted list; plain dictionaries do not "remember" a sort order you apply to them.

## Nested dictionaries

A **nested dictionary** is a dictionary whose value is itself another dictionary — the natural way to model a record that has its own sub-fields, such as an IRCTC-style booking record or a student's marks broken down subject by subject:

```python
students = {
    101: {"name": "Ananya", "marks": {"maths": 88, "science": 92}},
    102: {"name": "Rohit", "marks": {"maths": 65, "science": 70}},
}

print(students[101]["marks"]["maths"])
```

```
88
```

Here the outer dictionary's keys are roll numbers; each value is a smaller dictionary holding `name` and a further nested dictionary, `marks`. Reaching a value that sits two levels deep means **chaining bracket lookups**: first into the outer dictionary by roll number (`students[101]`), then into that record by `"marks"`, then into the resulting dictionary by subject name. This same chaining pattern extends to any depth of nesting — a railway booking record could nest a passenger list inside a booking, and each passenger dictionary inside that.

Before checking, predict what `students[102]["marks"]["science"]` evaluates to — trace it one bracket at a time, the same way you traced `students[101]["marks"]["maths"]` above, and you'll land on `70`.

## Dictionary comprehensions

A **dictionary comprehension** is a one-line expression that builds a brand-new dictionary from an iterable — the same comprehension idea you met for lists in Unit 3.1 and sets in Unit 3.3, but now producing `key: value` pairs instead of single elements:

```python
top_scorers = {
    roll_no: record["name"]
    for roll_no, record in students.items()
    if record["marks"]["maths"] > 70
}

print(top_scorers)
```

```
{101: 'Ananya'}
```

Read this the same way you'd read a list comprehension: loop over every `(roll_no, record)` pair from `.items()`, keep only the ones where `record["marks"]["maths"] > 70` is `True`, and store `record["name"]` against `roll_no` in a brand-new dictionary. `students` itself is left completely unchanged — `top_scorers` is an entirely separate dictionary. Try writing, on your own, a one-line comprehension that maps roll number to the *science* mark instead of the name, for every student without filtering anything out.

## Dictionaries in the real world

Dictionaries turn up everywhere real Python code touches real data:

- **Banking and UPI payments** — an account record (account number, holder name, balance) or a UPI transaction (payer, payee, amount, status) is naturally a dictionary; a bank's whole customer database is conceptually a dictionary of dictionaries, keyed by account number.
- **E-commerce** — a product catalog is a dictionary keyed by product ID; a shopping cart is a dictionary mapping product ID to quantity.
- **Records with sub-fields** — a patient record, a student information system, or a railway PNR record is exactly the nested-dictionary pattern from the section above.
- **APIs and cloud services** — request and response data is almost always exchanged as a dictionary in Python, matching the JSON format used by nearly every web API, a connection that becomes central once this course reaches API integration.

A handful of mistakes are worth watching for deliberately while dictionaries are still new:

- Using `d[key]` when the key might genuinely be missing, instead of `.get(key, default)` — an unhandled `KeyError` is one of the most common real-world dictionary crashes.
- Assuming a missing key quietly returns `None` — only `.get()` does that; plain bracket access always raises `KeyError`.
- Trying to use a list as a dictionary key, forgetting that keys must be hashable — this raises `TypeError: unhashable type: 'list'`.
- Writing `for x in d:` and treating `x` as a value, when it's actually a key.
- Typing a key name slightly differently in different places (`"roll_no"` in one line, `"rollNumber"` in another) — a plain spelling mismatch is one of the most common real dictionary bugs, and it fails silently as a `KeyError` far from where the typo was actually made.

## Try it yourself

Do this in a Colab cell before moving on. Build a dictionary of three products, each key a product ID and each value a nested dictionary with `name`, `price`, and `stock`. Add a fourth product using assignment. Use `.get()` to safely read a `"discount"` field that doesn't exist on any product, falling back to `0`. Loop with `.items()` and print only the products whose `stock` is below `5`. Write a dictionary comprehension that maps product ID to `name`, keeping only products priced above `500`. Finally, use `sorted()` on the dictionary's `.items()` to print all products ordered from highest price to lowest, and confirm afterwards that the original dictionary is completely unchanged.

---

### Key Terminology

- **Dictionary (`dict`)** — a mutable collection of key-value pairs, where each key is unique and used to look up the value paired with it.
- **Key-value pair** — one label (key) together with the value it's paired with inside a dictionary.
- **Mapping** — the general term for any type, such as `dict`, that connects keys to values.
- **Hashable** — able to have a fixed "fingerprint" computed for it that never changes; required of every dictionary key.
- **Bracket access (`d[key]`)** — reads a value by key; raises `KeyError` if the key is missing.
- **`.get(key, default)`** — reads a value by key, returning `default` (or `None`) instead of raising an error if the key is missing.
- **`KeyError`** — raised when code looks up or deletes a dictionary key that isn't present.
- **`del`** — the statement that removes a key and its value from a dictionary entirely.
- **`.pop(key, default)`** — removes a key and returns its value, falling back to `default` if the key is missing.
- **`.update(other)`** — merges another dictionary's key-value pairs into this one, overwriting shared keys.
- **`.setdefault(key, default)`** — returns a key's existing value, or inserts and returns `default` if the key is missing.
- **View (`.keys()` / `.values()` / `.items()`)** — the three looping styles over a dictionary's keys, values, or both together as tuples.
- **Nested dictionary** — a dictionary whose value is itself another dictionary, used to model a record with sub-fields.
- **Dictionary comprehension** — a one-line expression, `{key: value for item in iterable}`, that builds a new dictionary.

### Mastery Checkpoint

Before moving to Unit 3.5, check that you can answer these without looking back:

1. Why does storing a labelled record as two separate parallel lists risk a bug that a single dictionary avoids entirely?
2. What's the difference between what `d[key]` does and what `d.get(key, default)` does when `key` is missing from `d`?
3. Why is a list illegal as a dictionary key, while a tuple is perfectly legal?
4. What's the difference between what `for x in d:` gives you and what `for k, v in d.items():` gives you?
5. Does `sorted(d.items(), key=lambda kv: kv[1])` change the order of `d` itself? What does it actually return?

### Summary

You now know how a dictionary stores key-value pairs so a label and its value can never silently drift apart the way two parallel lists can, why a key must be unique and hashable while a value can be anything at all, and the difference between bracket access and `.get()` for reading one safely. You've added, overwritten, and deleted entries with assignment, `del`, and `.pop()`, looped over a dictionary three distinct ways with `.keys()`, `.values()`, and `.items()`, sorted one by key and by value without ever changing the original, chained bracket lookups into a nested dictionary to reach a deeply buried field, and built a brand-new dictionary in one line with a dictionary comprehension. From here, the next step is learning how Python actually walks through any of these collections under the hood, and the broader family of tools built on that same idea — starting with Unit 3.5, Iterators, Generators & Collections.

### Additional Resources

- [Python Tutorial — official docs: "Dictionaries"](https://docs.python.org/3/tutorial/datastructures.html#dictionaries)
- [Python 3 Documentation — Mapping Types (`dict`)](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict)
- [Python 3 Documentation — Built-in Exceptions (`KeyError`)](https://docs.python.org/3/library/exceptions.html#KeyError)
- [Python Tutorial — official docs: "Looping Techniques" (`.items()`)](https://docs.python.org/3/tutorial/datastructures.html#looping-techniques)
- [W3Schools — Python Dictionaries](https://www.w3schools.com/python/python_dictionaries.asp)
- [W3Schools — Python Dictionary Methods](https://www.w3schools.com/python/python_ref_dictionary.asp)
- [W3Schools — Python Dictionary Comprehension](https://www.w3schools.com/python/python_dictionaries_comprehension.asp)
