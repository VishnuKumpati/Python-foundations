# Dictionaries

---

## Why Dictionaries?

Every collection so far has been about position, index `0`, index `1`, and so on. A **dictionary** organizes data differently, by **key** instead of position. Instead of "give me the item at index 2," you ask "give me the value for `'name'`," which is often exactly how real data actually looks.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Dictionary** | An unordered collection of key-value pairs, written with `{ }` |
| **Key** | The name you look a value up by, must be hashable (immutable) |
| **Value** | The data stored under a key, can be any type |
| **Key-value pair** | One entry in a dictionary, a key and its associated value together |

---

## Creating a Dictionary

Written with curly braces, `key: value` pairs separated by commas:

```python
student = {
    "name": "Ananya",
    "age": 21,
    "course": "AI Native Engineering"
}
print(student)
```
```
{'name': 'Ananya', 'age': 21, 'course': 'AI Native Engineering'}
```

You've actually already seen this exact `{}` syntax, from the Sets unit, `{}` on its own is an empty dictionary, not an empty set, this is the reason why.

```python
print(type(student))   # <class 'dict'>
print(len(student))    # 3, number of key-value pairs
```

---

## Accessing and Updating Values

**Access a value by its key**, using square brackets, same bracket syntax as list indexing, but with a key instead of a position:

```python
print(student["name"])   # Ananya
```

**A missing key raises an error:**

```python
print(student["grade"])   # KeyError: 'grade'
```

**`.get()` is the safer alternative**, it returns `None` instead of crashing if the key doesn't exist, and lets you supply your own fallback:

```python
print(student.get("grade"))            # None, no error
print(student.get("grade", "N/A"))     # N/A, your own fallback
```

**Updating works by assigning to a key**, if the key exists, it's updated, if it doesn't, it's created:

```python
student["age"] = 22          # updates an existing key
student["email"] = "ananya@example.com"   # adds a brand new key
print(student)
```
```
{'name': 'Ananya', 'age': 22, 'course': 'AI Native Engineering', 'email': 'ananya@example.com'}
```

Dictionaries are **mutable**, same idea as lists, you can change one in place without reassigning the whole thing.

**`del` removes a key entirely:**

```python
del student["email"]
print(student)
```
```
{'name': 'Ananya', 'age': 22, 'course': 'AI Native Engineering'}
```

Same `del` keyword you already know from lists, here it removes by key instead of by position. It raises a `KeyError` if the key doesn't exist, same as `dict["key"]` does.

---

## Keys Must Be Hashable

Not just anything can be a key, only **immutable**, hashable types, strings, numbers, and tuples all qualify, lists and sets don't:

```python
valid = {("lat", "long"): (13.08, 80.27)}   # a tuple as a key, works fine

invalid = {[1, 2]: "value"}   # TypeError: unhashable type: 'list'
```

This is exactly why tuples exist alongside lists, when you need a fixed, unchangeable collection that can also serve as a key.

**Values, on the other hand, can be absolutely anything**, including a list, or another dictionary:

```python
student = {
    "name": "Ananya",
    "scores": [85, 90, 78]   # a list as a value, perfectly fine
}
```

---

## Dictionary Methods

| Method | Does what |
|---|---|
| `.keys()` | returns all the keys |
| `.values()` | returns all the values |
| `.items()` | returns all key-value pairs together |
| `.get(key, default)` | safely looks up a value, no error if missing |
| `.pop(key)` | removes a key and returns its value |
| `.update(other_dict)` | merges another dictionary in, overwriting matching keys |
| `.setdefault(key, default)` | returns the value if the key exists, otherwise sets and returns the default |

```python
student = {"name": "Ananya", "age": 21}

print(student.keys())     # dict_keys(['name', 'age'])
print(student.values())   # dict_values(['Ananya', 21])
print(student.items())    # dict_items([('name', 'Ananya'), ('age', 21)])
```

**`.update()` merges another dictionary in:**

```python
student.update({"age": 22, "course": "AI Native Engineering"})
print(student)
```
```
{'name': 'Ananya', 'age': 22, 'course': 'AI Native Engineering'}
```

`age` was already a key, so it got overwritten, `course` was new, so it got added.

**`.pop()` removes a key and hands back its value**, same idea as `.pop()` on a list, but by key instead of position:

```python
removed = student.pop("age")
print(removed)      # 22
print(student)       # {'name': 'Ananya', 'course': 'AI Native Engineering'}
```

**`.setdefault()` is a two-in-one: get a value if the key exists, or set it (and return it) if it doesn't:**

```python
student.setdefault("age", 18)   # key missing, so it's added with 18
print(student)
```
```
{'name': 'Ananya', 'course': 'AI Native Engineering', 'age': 18}
```

```python
student.setdefault("name", "Unknown")   # key already exists, untouched
print(student["name"])
```
```
Ananya
```

This is genuinely useful for counting things, adding a key the first time you see it, and leaving it alone every time after:

```python
word_counts = {}
words = ["apple", "banana", "apple", "cherry", "apple"]

for word in words:
    word_counts.setdefault(word, 0)
    word_counts[word] += 1

print(word_counts)
```
```
{'apple': 3, 'banana': 1, 'cherry': 1}
```

---

## Membership and Looping

**`in` checks keys, not values, by default:**

```python
print("name" in student)      # True
print("Ananya" in student)    # False, that's a value, not a key
```

**Looping over a dictionary by default gives you the keys:**

```python
for key in student:
    print(key)
```
```
name
course
```

**Loop over both key and value together with `.items()`:**

```python
for key, value in student.items():
    print(key, ":", value)
```
```
name : Ananya
course : AI Native Engineering
```

This is by far the most common way you'll loop through a dictionary, unpacking each key-value pair directly, same unpacking idea from tuples.

---

## Sorting Dictionaries

Dictionaries don't have a `.sort()` of their own, but `sorted()` (the same function you already know from lists) works on them too, you just need to be specific about what you're sorting by.

**Sort by key:**

```python
prices = {"banana": 20, "apple": 50, "fig": 35}

for key in sorted(prices):
    print(key, ":", prices[key])
```
```
apple : 50
banana : 20
fig : 35
```

`sorted(prices)` sorts the keys alphabetically, looping directly over a dictionary already gives you keys, so this reads naturally.

**Sort by value**, using `key=` (from Lists), pointed at `.items()` so you have both the key and value to work with:

```python
for key, value in sorted(prices.items(), key=lambda item: item[1]):
    print(key, ":", value)
```
```
banana : 20
fig : 35
apple : 50
```

`item[1]` is the value half of each `(key, value)` tuple `.items()` produces, that's exactly what `key=` sorts by here. Add `reverse=True` for highest-to-lowest.

---

## Nested Dictionaries

A dictionary can hold other dictionaries as values, common for representing structured, real-world data:

```python
students = {
    "ananya": {"age": 21, "course": "AI Native Engineering"},
    "rahul": {"age": 23, "course": "Data Engineering"}
}

print(students["ananya"]["course"])   # AI Native Engineering
```

Looping through a nested dictionary needs a loop inside a loop, same nested-loop idea from lists:

```python
for username, details in students.items():
    print(username, ":", details["course"])
```
```
ananya : AI Native Engineering
rahul : Data Engineering
```

---

## Dictionary Comprehension

Same idea as list and set comprehension, `key: value` instead of just a value:

```python
numbers = [1, 2, 3, 4, 5]
squares = {n: n * n for n in numbers}
print(squares)
```
```
{1: 1, 2: 4, 3: 9, 4: 16, 5: 25}
```

Read it as: "`n: n * n`, for every `n` in `numbers`." A filter condition works the same way it did on lists and sets:

```python
even_squares = {n: n * n for n in numbers if n % 2 == 0}
print(even_squares)
```
```
{2: 4, 4: 16}
```

**Comprehension also works for inverting a dictionary, swapping keys and values:**

```python
prices = {"banana": 20, "apple": 50, "fig": 35}
by_price = {value: key for key, value in prices.items()}
print(by_price)
```
```
{20: 'banana', 50: 'apple', 35: 'fig'}
```

`.items()` gives you both halves to work with, and the comprehension just writes them back in the opposite order. This only works cleanly if the original values are themselves unique and hashable, if two keys shared the same value, one would silently overwrite the other in the inverted dictionary.

---

## Try it Yourself

**(a)** Create a dictionary `menu` with three items and their prices, `{"samosa": 15, "coffee": 35, "tea": 20}`. Print the price of `"coffee"` using `.get()`, with a fallback of `0` in case it's missing.

**(b)** Add a new item, `"sandwich"` at `60`, then update `"samosa"` to `18`. Print the final dictionary.

**(c)** Loop through `menu` using `.items()` and print each item formatted as `"samosa: Rs.15"`.

**Your turn:** write a dictionary comprehension that builds `{item: price for item > 20}` from `menu`, only items priced above 20.

---

## Common Mistakes

- Using `dict["key"]` on a key that might not exist, raises a `KeyError`, use `.get()` when you're not sure
- Assuming `in` checks values, it checks keys by default
- Trying to use a list (or another dictionary) as a key, only hashable, immutable types are allowed
- Looping with `for item in dict:` and expecting values, you get keys, use `.items()` for both
- Confusing `.pop(key)` (removes by key, dictionary) with `.pop()` (removes by position, list), same method name, different behavior depending on the type
- Forgetting `{}` alone creates a dictionary, not an empty set, same gotcha carried over from sets
- Calling `sorted()` directly on a dictionary expecting values, it sorts keys by default, use `.items()` with `key=` to sort by value instead
- Inverting a dictionary when the values aren't actually unique, duplicate values silently overwrite each other as keys in the result

---

## Interview Questions

**Q1: What's the difference between accessing a value with `dict["key"]` and `dict.get("key")`?**

A: `dict["key"]` raises a `KeyError` if the key doesn't exist. `dict.get("key")` returns `None` instead, or a fallback value you supply, no error either way.

**Q2: What types can be used as dictionary keys?**

A: Only hashable, immutable types, strings, numbers, and tuples. Lists and other dictionaries can't be used as keys, but they can be used as values.

**Q3: What does looping directly over a dictionary give you, keys or values?**

A: Keys, by default. Use `.items()` if you need both the key and value together in the loop.

**Q4: What's the difference between `.pop()` on a dictionary and `.pop()` on a list?**

A: On a dictionary, `.pop(key)` removes an entry by its key and returns the value. On a list, `.pop(index)` removes an entry by position, defaulting to the last item if no index is given.

**Q5: What does `.setdefault()` actually do?**

A: It returns a key's value if it exists. If it doesn't, it adds the key with the default value you supply, and returns that. It's commonly used for counting or grouping, adding a key only the first time you encounter it.

**Q6: How do you sort a dictionary by its values instead of its keys?**

A: `sorted()` doesn't sort dictionaries directly by value, use `sorted(d.items(), key=lambda item: item[1])`, which sorts the key-value pairs by the value half of each tuple.

---

## Quick Recap

- A dictionary stores key-value pairs, written with `{ }`; look up a value by its key, not its position.
- `dict["key"]` raises a `KeyError` if missing; `.get("key", default)` is the safer alternative.
- Dictionaries are mutable, assigning to a key updates it if it exists, or creates it if it doesn't; `del dict[key]` removes an entry entirely.
- Keys must be hashable (strings, numbers, tuples); values can be absolutely anything, including lists or other dictionaries.
- `.keys()`, `.values()`, `.items()`, `.update()`, `.pop()`, and `.setdefault()` are the core methods to know.
- `in` checks keys by default; looping directly gives keys too, use `.items()` for both key and value together.
- Sort by key with `sorted(d)`; sort by value with `sorted(d.items(), key=lambda item: item[1])`.
- Dictionary comprehension (`{k: v for ...}`) works the same way list and set comprehension do, and can also invert a dictionary by swapping key and value.

##  Reference Links

- [The Python Tutorial — Dictionaries](https://docs.python.org/3/tutorial/datastructures.html#dictionaries)
- [Python 3 Documentation — Mapping Types (`dict`)](https://docs.python.org/3/library/stdtypes.html#mapping-types-dict)
- [Real Python — Dictionaries in Python](https://realpython.com/python-dicts/)
- [W3Schools — Python Dictionaries](https://www.w3schools.com/python/python_dictionaries.asp)
