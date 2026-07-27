# Sets

Lists (Unit 3.1) keep order and allow duplicates; tuples (Unit 3.2) lock values in place once created. But some problems care about neither order nor duplicates — only about whether something is present at all, and how quickly you can check. Has this phone number already registered on this app? Has this OTP already been used? Which pincodes does a food-delivery service actually cover? None of these questions care what order the data arrived in, and none of them want a value counted twice.

Picture a college placement cell collecting sign-ups for a workshop through a Google Form. Three hundred students submit their names, but a few click "submit" twice by mistake, so a handful of names appear twice in the raw data. You need one thing out of this mess: a clean list of distinct students, with no repeats. You could write a loop that checks every new name against every name already collected — but that gets slower and slower as the list grows, because Python has to scan the whole list from the start each time. Python has a purpose-built answer to exactly this kind of problem.

This chapter introduces that answer: the **set**. You will learn what makes a set fundamentally different from a list, how to create one, how to test membership and mutate it safely, and how to combine or compare two sets using four operations that show up constantly in real systems — from checking a UPI ID against a blacklist to reconciling a bank's transaction records.

---

## The bag that never holds a duplicate

A **set** is an unordered collection of unique, immutable elements. Picture it as a bag of distinct items: you can drop values in, but the bag simply refuses a duplicate — try to add something already inside, and nothing happens, no error, no second copy sitting alongside the first. That single property, the **uniqueness guarantee**, is what a set is built entirely around, and it is what makes questions like "have I seen this before?" and "what do these two groups have in common?" fast and simple to answer.

Three separate ideas sit inside that one definition, and each one matters on its own:

- **Unordered** — a set does not remember the position you added things in. There is no reliable "first" or "last" element, so a set cannot be indexed the way a list can; `my_set[0]` is not valid Python.
- **Unique** — a set never stores two equal values. Add a value that is already present and the set silently stays exactly the same.
- **Immutable elements** — every value placed inside a set must itself be a type that cannot change after creation: a number, a string, or a tuple, for example. A list cannot go inside a set, because a list can change after it is created.

That third point has a name you already met in Unit 3.2: **hashable**. A hashable value's contents never change, so Python can compute a stable "fingerprint" for it once and rely on that fingerprint forever after. A set requires every element it stores to be hashable, for exactly the reason a dictionary will require the same thing of its keys in Unit 3.4 — both structures use this fingerprinting trick internally to locate values almost instantly, instead of scanning through everything one at a time.

Before reading on, predict this for yourself: if you tried `{"soap", ["shampoo"], "toothpaste"}`, would Python accept it? It would not — a list is not hashable, so Python raises `TypeError: unhashable type: 'list'` the moment it tries to store it.

## Creating a set — and the trap waiting inside `{}`

You build a set by writing comma-separated values inside curly braces, or by handing an existing iterable — a list, tuple, or string — to the `set()` constructor:

```python
fruits = {"apple", "banana", "mango"}
from_a_list = set([1, 2, 2, 3, 3, 3])
print(fruits)
print(from_a_list)
```

```
{'apple', 'mango', 'banana'}
{1, 2, 3}
```

Notice two things in that output. First, `fruits` printed in a different order than you typed it — that is the "unordered" property from the previous section, made visible. Second, `from_a_list` collapsed six numbers down to three, because the repeats simply did not survive; the uniqueness guarantee applied the instant the set was built. Passing a duplicate-heavy list into `set()` is one of the most common one-line ways to de-duplicate data in Python.

Now for the trap. Every other collection you have met so far uses an empty pair of brackets for "nothing in it yet" — `[]` for an empty list, `()` for an empty tuple. You would reasonably expect `{}` to mean an empty set. It does not.

**Bare `{}` always creates an empty dictionary, never an empty set** — dictionaries (Unit 3.4) claimed that syntax first, and Python has no way to tell from `{}` alone which one you meant. The only way to create a genuinely empty set is to call `set()` with nothing inside it:

```python
empty = set()
not_a_set = {}
print(type(empty))
print(type(not_a_set))
```

```
<class 'set'>
<class 'dict'>
```

Try this yourself in a Colab cell before moving on: write `looks_empty = {}`, then run `print(type(looks_empty))` — seeing `<class 'dict'>` with your own eyes, once, is what makes this trap stop catching you.

## Membership testing: the whole reason sets exist

**Membership** is the question "is this value present in the collection?", and you test it with the same `in` operator you already used with lists and strings:

```python
delivery_pincodes = {"560001", "560002", "560034"}
print("560001" in delivery_pincodes)
print("560099" in delivery_pincodes)
```

```
True
False
```

This is the single biggest practical reason sets exist. A set is stored internally using the same hashing technique mentioned above — each element's fingerprint tells Python almost exactly where to look for it, so checking membership stays roughly constant in speed no matter how large the set grows. Checking membership against a list, by contrast, gets slower and slower as the list grows, because Python has to scan it from the start every single time, the same limitation the placement-cell loop ran into at the start of this chapter.

| | Membership check on a `list` | Membership check on a `set` |
|---|---|---|
| How it works | Scans from the start until it finds a match, or reaches the end | Computes a fingerprint and jumps almost straight to the answer |
| Speed as the collection grows | Gets slower | Stays roughly constant |
| Good for | Small collections, or when order also matters | Large collections, repeated "have I seen this?" checks |

A payments platform checking whether a UPI ID appears on a fraud blacklist, an app checking whether an OTP has already been used, a website checking whether a username is already taken — all of these are, underneath, exactly this one operation, run against a set specifically because it needs to stay fast as the blacklist grows into the millions.

## Mutation: `add()`, `remove()`, and `discard()`

**Mutation** means changing a set's contents after it has already been created. Three methods handle this, and the difference between two of them matters a great deal:

| Method | What it does | If the value is missing |
|---|---|---|
| `add(value)` | Inserts `value` into the set. | Nothing happens — no error, no duplicate. |
| `remove(value)` | Removes `value` from the set. | Raises a `KeyError`. |
| `discard(value)` | Removes `value` from the set if present. | Does nothing at all — no error. |

```python
cart_items = {"soap", "shampoo"}
cart_items.add("toothpaste")
cart_items.discard("shampoo")
cart_items.discard("perfume")   # not present, but no error
print(cart_items)
```

```
{'soap', 'toothpaste'}
```

Before checking, predict what happens if that last line were `cart_items.remove("perfume")` instead of `discard("perfume")` — Python raises `KeyError: 'perfume'`, because `remove()` treats a missing value as an error worth stopping for, while `discard()` treats it as a non-event.

**Default to `discard()` unless a missing value would genuinely indicate a bug you want Python to flag loudly** — an e-commerce cart removing an item that might already be gone should stay quiet; a banking system removing an account that is supposed to exist should not.

## The four core set operations

Sets exist not just to hold unique values, but to answer questions about how two collections relate to each other. Picture two food-delivery apps, each maintaining the pincodes (postal area codes) they currently serve in a city:

```python
app_a_pincodes = {"560001", "560002", "560034", "560045"}
app_b_pincodes = {"560002", "560034", "560099"}
```

Every question a business analyst would ask about these two sets maps directly onto one of four operations, each with an operator symbol and an equivalent method name:

| Operation | Meaning | Operator | Method |
|---|---|---|---|
| **Union** | Every element in either set, or both, combined with no repeats | `a \| b` | `a.union(b)` |
| **Intersection** | Only the elements that appear in *both* sets | `a & b` | `a.intersection(b)` |
| **Difference** | Elements in the first set that do **not** appear in the second | `a - b` | `a.difference(b)` |
| **Symmetric difference** | Elements that appear in exactly one of the two sets, but not both | `a ^ b` | `a.symmetric_difference(b)` |

```python
print(app_a_pincodes | app_b_pincodes)
print(app_a_pincodes & app_b_pincodes)
print(app_a_pincodes - app_b_pincodes)
print(app_a_pincodes ^ app_b_pincodes)
```

```
{'560001', '560002', '560034', '560045', '560099'}
{'560002', '560034'}
{'560001', '560045'}
{'560001', '560045', '560099'}
```

Read these off in plain English before moving on. "Which pincodes does at least one app serve?" is the union. "Which pincodes do both apps serve?" is the intersection. "Which pincodes are exclusive to App A?" is the difference. "Which pincodes are served by exactly one app, not both?" is the symmetric difference. A bank's reconciliation job de-duplicating transaction IDs, and two students comparing which courses they share, are the exact same shape wearing different names.

**The operator forms (`|`, `&`, `-`, `^`) require both sides to already be sets; the method forms (`.union()`, `.intersection()`, and so on) accept any iterable on the right, so you can compare a set directly against a plain list without converting it first.** This is worth choosing deliberately: if the data you are comparing against is naturally a list, reach for the method form and skip the conversion step.

Each of these four operations also has a mutating counterpart — `update()`, `intersection_update()`, `difference_update()`, and `symmetric_difference_update()` — which change the set on the left in place instead of building a brand-new one:

```python
app_a_pincodes.update(app_b_pincodes)
print(app_a_pincodes)
```

```
{'560001', '560002', '560034', '560045', '560099'}
```

Reach for the in-place form when you genuinely mean to grow or shrink an existing set as new data arrives — say, an app absorbing a partner's coverage area into its own — and reach for the plain operator when you want to keep both original sets untouched and work with a fresh result instead.

## Choosing between a list and a set

By now you have enough to make this choice deliberately rather than by habit:

| | List | Set |
|---|---|---|
| Allows duplicates | Yes | No — dropped automatically |
| Preserves the order you added things | Yes | No — order is not guaranteed |
| Supports indexing (`my_collection[0]`) | Yes | No |
| Membership check (`in`) as it grows | Gets slower | Stays roughly constant |
| Reach for it when | Order or position matters, or duplicates are meaningful | Uniqueness or overlap between collections is what matters |

An IRCTC-style booking system keeps the *sequence* of stops on a route as a list — order is the entire point — but might keep the set of pincodes eligible for doorstep delivery of a physical ticket as a set, because only "is this pincode covered?" matters there, not any particular order.

## Set comprehensions

A **set comprehension** builds a set in a single line, using the same shape as the list comprehensions you have already met, but with curly braces instead of square brackets: `{expression for item in iterable}`. Because the result is a set, it is automatically de-duplicated as it is built:

```python
raw_emails = ["Asha@Gmail.com", "asha@gmail.com", "Ravi@Yahoo.com"]
unique_emails = {email.lower() for email in raw_emails}
print(unique_emails)
```

```
{'asha@gmail.com', 'ravi@yahoo.com'}
```

Three raw entries collapse into two, because lower-casing maps `"Asha@Gmail.com"` and `"asha@gmail.com"` onto the exact same string. This is precisely the tool the placement-cell problem from the opening of this chapter needs: a set comprehension can normalise capitalisation and de-duplicate names in the very same line, rather than requiring a separate cleaning pass followed by a separate `set()` call.

**A set only ever compares values exactly as written — `"Ana"` and `"ana"` are different strings, and a set keeps both unless you normalise the case yourself before building it.** A set comprehension is exactly where that normalising step belongs.

## A note on `frozenset`

A **`frozenset`** is an immutable version of a set — once built, it can never be changed; there is no `add()`, `remove()`, or `discard()` available on it. You will not use it heavily in this course, but it is worth recognising by name: it appears in code that needs a set-like value guaranteed never to change after creation, such as a fixed collection of roles a system permits (`{"admin", "editor", "viewer"}` locked down as a `frozenset` so no part of the codebase can accidentally add a rogue role later).

A short list of mistakes worth watching for deliberately while sets are still new:

- Writing `{}` and expecting an empty set — it is always an empty dictionary.
- Trying to index or slice a set (`my_set[0]`) — this raises `TypeError`, because a set has no concept of position.
- Assuming a set preserves the order values were added in, and writing code that quietly depends on it.
- Placing a mutable value, such as a list, inside a set — this raises `TypeError: unhashable type`.
- Reaching for `remove()` when a missing value is expected and harmless — prefer `discard()` unless a missing value is genuinely a bug.

## Try it yourself

Do this in a Colab cell before moving on. Start with `tags_raw = ["Python", "AI", "python", "SQL", "ai", "SQL"]`, a messy list of user-submitted tags with repeats and mixed capitalisation. Build a cleaned set of lowercase tags with a single set comprehension. Then create a second learner's tags, `friend_tags = {"ai", "cloud", "sql"}`, and compute, using the operators from this chapter: which tags the two of you share (intersection), which tags only you have (difference), and the full combined set of tags between you (union). Finally, use `in` to check whether `"java"` belongs to your cleaned tag set.

---

### Key Terminology

- **Set** — an unordered collection of unique, immutable elements, written with `{}` or built with `set()`.
- **Uniqueness guarantee** — a set silently refuses to store a value it already contains, with no error and no duplicate.
- **Hashable** — describes a value whose contents never change, letting Python compute a stable fingerprint for it; required for anything stored inside a set.
- **Membership** — the question "is this value present?", tested with the `in` operator.
- **Mutation** — changing a set's contents after creation, using `add()`, `remove()`, or `discard()`.
- **Union (`|`)** — every element present in either of two sets, combined with no repeats.
- **Intersection (`&`)** — only the elements present in both sets.
- **Difference (`-`)** — elements in the first set that are not in the second.
- **Symmetric difference (`^`)** — elements present in exactly one of the two sets, not both.
- **Set comprehension** — the one-line form `{expression for item in iterable}` that builds a de-duplicated set.
- **`frozenset`** — an immutable version of a set that can never be changed after creation.
- **`KeyError`** — the error `remove()` raises when the value it is asked to remove is not present.

### Mastery Checkpoint

Before moving to Unit 3.4, check that you can answer these without looking back:

1. Why does writing `looks_empty = {}` not create an empty set, and what should you write instead?
2. Why does checking membership with `in` stay fast on a set as it grows, while the same check on a list gets slower?
3. What is the difference in behaviour between `remove()` and `discard()` when the value being removed is not actually present?
4. Given `a = {"x", "y", "z"}` and `b = {"y", "z", "w"}`, what does each of `a | b`, `a & b`, `a - b`, and `a ^ b` produce?
5. Why must every element stored inside a set be hashable, and why does that rule out storing a list inside a set?

### Summary

You now know what makes a set fundamentally different from the lists and tuples you have already met: it drops duplicates automatically, keeps no reliable order, and requires every element to be hashable. You have created sets with `{}` and `set()`, side-stepped the empty-set trap, tested and mutated membership safely with `in`, `add()`, `remove()`, and `discard()`, and used the four core operations — union, intersection, difference, and symmetric difference — to compare collections the way real systems compare pincodes, blacklists, and course lists. You have also built a set in a single line with a set comprehension, normalising and de-duplicating data in the same step. From here, the next step is learning how to pair each value with its own label instead of just checking for its bare presence — starting with Unit 3.4, Dictionaries.

### Additional Resources

- [Python 3 Documentation — Set Types: `set`, `frozenset`](https://docs.python.org/3/library/stdtypes.html#set-types-set-frozenset)
- [Python Tutorial — official docs: "Data Structures" (Sets)](https://docs.python.org/3/tutorial/datastructures.html#sets)
- [Python 3 Documentation — Built-in `set` and `frozenset` methods](https://docs.python.org/3/library/stdtypes.html#frozenset.union)
- [W3Schools — Python Sets](https://www.w3schools.com/python/python_sets.asp)
- [W3Schools — Python Set Methods](https://www.w3schools.com/python/python_ref_set.asp)
