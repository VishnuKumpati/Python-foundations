---
topic_id: 7cb62f0c-485f-4798-a6bb-390a3f1a8542
title: Unit 3.3 - Sets
position_in_module: 3
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 3.3 - Sets — Topic Corpus

## 2. Prerequisites

- Unit 3.1 - Lists (`fe27250e-c2c2-491c-afd7-57d9b533deab`) — sets are introduced by contrast with lists: both store multiple values, but a list keeps order and duplicates while a set does not.
- Unit 3.2 - Tuples (`18ffde38-766c-4912-bd6a-bfd0e238a750`) — this unit reuses the idea of **hashability** introduced there (a value whose contents never change, so Python can compute a stable fingerprint for it) as the requirement for anything stored inside a set.

## 3. Learning Objectives

- **Explain** what a set is, and why its uniqueness guarantee makes it behave differently from a list or a tuple.
- **Create** sets using the `{}` literal syntax and the `set()` constructor, both from listed values and from an existing iterable such as a list.
- **Apply** the four core set operations — union, intersection, difference, and symmetric difference — using both their operator symbols and their method names.
- **Differentiate** a list from a set on the basis of duplicates, order, indexing, and membership-check speed, and choose the correct one for a given task.
- **Implement** membership testing with `in` and safe mutation of a set using `add()`, `remove()`, and `discard()`.
- **Debug** the most common set mistakes, including confusing `{}` with an empty dictionary, assuming set order is preserved, and trying to index a set.

## 4. Introduction

Imagine you are building a sign-up form for a workshop, and 500 students submit their names through a Google Form. Several students accidentally click "submit" twice, so their name appears in the data twice. You need one thing: a clean list of every distinct student who signed up, with no repeats. You could write a loop that checks each new name against every name already collected before adding it — but that gets slower and slower as the list grows, because Python has to scan the whole list from the start every single time. Python has a built-in answer to exactly this kind of problem: the **set**.

Picture a set as a bag of unique items. You can drop values into it, but the bag never allows a duplicate to sit inside — if you try to add something that is already there, nothing changes. This one property is what a set is built entirely around, and it is what makes questions like "has this value already been seen?" or "which items do these two groups have in common?" fast and simple to answer in Python.

This unit teaches how to create a set, how to test whether a value belongs to one, how to safely add and remove values, how to combine or compare two sets using the four core set operations, and how to build a set in a single line with a set comprehension.

## 5. Core Concepts

### 5.1 What a Set Is

A **set** is an unordered collection of unique, immutable elements [1]. Three separate ideas are packed into that sentence, and each one matters:

- **Unordered.** A set does not remember the position you added things in. There is no "first" or "last" element in any reliable sense, so a set cannot be indexed the way a list can — `my_set[0]` is not valid.
- **Unique.** A set never stores two equal values. If you try to add a value that is already present, the set silently stays exactly the same — no error, no duplicate. This is called the **uniqueness guarantee** [1].
- **Immutable elements.** Every value placed inside a set must itself be a type that cannot change after it is created — a number, a string, or a tuple, for example. A list cannot be placed inside a set, because a list *can* change after creation. This requirement is called being **hashable**: a hashable value's contents never change, so Python can compute a stable "fingerprint" for it once and rely on that fingerprint later [1]. You met this same idea, hashability, when comparing tuples to lists in Unit 3.2 — a set requires it for exactly the same reason a dictionary will in Unit 3.4 (dictionaries use this same fingerprinting trick internally for their keys — the mechanics of that are covered when you reach that unit).

### 5.2 Creating a Set

You create a set by writing comma-separated values inside curly braces, or by passing an existing iterable — such as a list, tuple, or string — into the `set()` constructor [1]:

```python
fruits = {"apple", "banana", "mango"}
from_a_list = set([1, 2, 2, 3, 3, 3])   # becomes {1, 2, 3}
```

Passing a list with repeated values into `set()` is one of the most common ways to de-duplicate data in Python — the repeats simply do not survive, because the uniqueness guarantee applies the moment the set is built.

There is one sharp trap here: bare `{}` does **not** create an empty set. It creates an empty **dictionary** instead — a different data structure you will meet properly in Unit 3.4, which pairs each value with its own label instead of just holding bare values. Because `{}` is already claimed by dictionaries, the only way to create an empty set is to call `set()` with nothing inside it [1]:

```python
empty = set()      # correct — an empty set
not_a_set = {}      # this is an empty dictionary, not a set
```

### 5.3 Membership Testing

**Membership** is the question "is this value present in the collection?" [1] You test it using the same `in` operator you already used with lists and strings:

```python
delivery_pincodes = {"560001", "560002", "560034"}
print("560001" in delivery_pincodes)   # True
print("560099" in delivery_pincodes)   # False
```

This is the single biggest practical reason sets exist. Checking membership against a set stays roughly constant in speed no matter how large the set grows, while checking membership against a list gets slower as the list grows, because Python has to scan a list from the start each time. A set achieves this speed through an internal storage technique that computes a quick fingerprint for each value it stores — the same technique dictionaries use for their keys, covered when you reach Unit 3.4. The one consequence you need now is this: because a set is organized around fingerprints rather than position, it has no reliable order, and that is the direct cause of the "unordered" property from Section 5.1.

### 5.4 Mutation: `add()`, `remove()`, and `discard()`

**Mutation** means changing a set's contents after it has already been created [1]. Three methods handle this, and they behave differently on a value that is missing:

| Method | What it does | Behaviour if the value is missing |
|---|---|---|
| `add(value)` | Inserts `value` into the set. | If already present, nothing happens — no error, no duplicate. |
| `remove(value)` | Removes `value` from the set. | Raises a `KeyError` — an error Python raises when you ask a collection to act on a value it does not actually contain. |
| `discard(value)` | Removes `value` from the set if present. | Does nothing at all — no error. |

```python
cart_items = {"soap", "shampoo"}
cart_items.add("toothpaste")
cart_items.discard("shampoo")
cart_items.discard("perfume")   # not present, but no error
print(cart_items)   # {'soap', 'toothpaste'}
```

Because `discard()` never raises an error, it is the safer default whenever a missing value should simply be ignored. Reach for `remove()` only when a missing value would genuinely indicate a bug you want Python to flag loudly.

### 5.5 The Four Core Set Operations

Sets exist not just to hold unique values, but to answer questions about how two collections relate to each other. Python gives you four operations for this, each with an operator symbol and an equivalent method name [1]:

| Operation | Meaning | Operator | Method |
|---|---|---|---|
| **Union** | Every element that appears in either set, or both, combined with no repeats. | `a \| b` | `a.union(b)` |
| **Intersection** | Only the elements that appear in *both* sets. | `a & b` | `a.intersection(b)` |
| **Difference** | Elements in the first set that do **not** appear in the second. | `a - b` | `a.difference(b)` |
| **Symmetric difference** | Elements that appear in exactly one of the two sets, but not both. | `a ^ b` | `a.symmetric_difference(b)` |

```python
priya_courses = {"Python", "DBMS", "AI"}
rohan_courses = {"DBMS", "AI", "Cloud Computing"}

print(priya_courses & rohan_courses)   # {'DBMS', 'AI'}          — intersection
print(priya_courses - rohan_courses)   # {'Python'}              — difference
print(priya_courses | rohan_courses)   # union of both sets
print(priya_courses ^ rohan_courses)   # symmetric difference
```

One detail matters for how you write these: the operator forms (`|`, `&`, `-`, `^`) require **both sides to already be sets**. The method forms (`.union()`, `.intersection()`, and so on) are more flexible — they accept any iterable on the right-hand side, so you can compare a set directly against a plain list without converting it first [1].

### 5.6 Set Comprehensions

A **set comprehension** builds a set in a single line, using the same shape as a list comprehension but with curly braces instead of square brackets: `{expression for item in iterable}` [1]. Because the result is a set, it is automatically de-duplicated as it is built:

```python
raw_emails = ["Asha@Gmail.com", "asha@gmail.com", "Ravi@Yahoo.com"]
unique_emails = {email.lower() for email in raw_emails}
print(unique_emails)   # {'asha@gmail.com', 'ravi@yahoo.com'}
```

This is especially useful when a transform and a de-duplication need to happen together, since the transform can map several different-looking inputs onto the same output value — here, lower-casing collapses `"Asha@Gmail.com"` and `"asha@gmail.com"` into a single entry.

### 5.7 A Note on `frozenset`

A **`frozenset`** is an immutable version of a set — once built, it can never be changed (no `add()`, `remove()`, or `discard()`) [1]. It is mentioned here only by name, as a variant you may see in code that needs a set-like value guaranteed never to change after creation, such as a fixed collection of allowed roles; its full behavior is outside the scope of this unit.

## 6. Implementation

To clean a messy, duplicate-ridden list of names into a set of distinct values and compare it against another such list — a pattern you will reuse constantly — follow these steps:

1. Collect the raw data as an ordinary list (it may contain duplicates and inconsistent capitalization).
2. Build a set comprehension that normalizes each value (for example, `.lower()`) while it de-duplicates: `{item.lower() for item in raw_list}`.
3. Repeat step 2 for the second list you want to compare against.
4. Use `|` for "everyone in either group," `&` for "only people in both groups," `-` for "only in this group and not the other," and `^` for "in exactly one of the two groups."
5. Test membership on the cleaned sets with `in`, not on the original raw lists.

## 7. Real-World Patterns

The clearest way to see sets in production is two organizations each tracking a collection of unique values and needing to compare them [1]. Picture two food delivery apps, each holding the pincodes (postal area codes) they currently deliver to:

```python
app_a_pincodes = {"560001", "560002", "560034", "560045"}
app_b_pincodes = {"560002", "560034", "560099"}
```

Every question a business analyst would ask here maps directly onto a set operation: "Which pincodes does at least one app serve?" is the union. "Which pincodes do both apps serve?" is the intersection. "Which pincodes are exclusive to App A?" is the difference. "Can App A deliver to this one pincode?" is a membership test. The same shape reappears constantly elsewhere: a bank's reconciliation job de-duplicating transaction IDs, a payments system checking a customer's UPI ID (a `name@bank` payment identifier) against a blacklist in a split second, or two students comparing which courses they have in common. All of these are the same underlying pattern — unique values, compared or checked, with no manual loop required.

## 8. Best Practices

- Reach for a set when the task is about **uniqueness** or **overlap between collections** — not when order or position matters, since a list or tuple exists for those.
- Always write `set()` to create an empty set; never assume bare `{}` behaves like one.
- Prefer the method form (`a.intersection(some_list)`) when the value you are comparing against is a plain list — it saves you from converting it to a set first.
- If you plan to test membership repeatedly inside a loop, convert the list you are checking against into a set **once, up front** — this is usually the single biggest speed-up available.
- Use a set comprehension when you need to transform values (such as lower-casing) while de-duplicating them in the same step.
- A set only ever compares values exactly as written — `"Ana"` and `"ana"` are different strings and both will be kept unless you normalize the case yourself before adding them.

## 9. Hands-On Exercise

Given a raw list of user-submitted tags with repeats and mixed case, `tags_raw = ["Python", "AI", "python", "SQL", "ai"]`, build a cleaned, de-duplicated set of lowercase tags using a set comprehension. Then, given a second list of a different user's tags, compute which tags the two users share (intersection) and which tags only the first user has (difference).

## 10. Key Takeaways

- A **set** is an unordered collection of unique, immutable (hashable) elements — duplicates are dropped automatically at creation, and a set cannot be indexed or sliced.
- Create a set with `{value1, value2}` or `set(iterable)`; always use `set()` for an **empty** set, since bare `{}` creates an empty dictionary instead.
- The four core operations — **union (`|`)**, **intersection (`&`)**, **difference (`-`)**, and **symmetric difference (`^`)** — each have an equivalent method (`.union()`, `.intersection()`, `.difference()`, `.symmetric_difference()`) that accepts any iterable, not only another set.
- Membership testing with `in` on a set stays fast even as the set grows, unlike on a list, which is why sets are the right tool for repeated "have I seen this before?" checks.
- Mutate a set safely with `add()` (insert), `discard()` (remove if present, no error otherwise), and `remove()` (remove, but raises `KeyError` if the value is missing).

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
