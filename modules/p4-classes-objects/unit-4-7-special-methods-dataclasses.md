# Special Methods and Dataclasses: Making a Class Feel Built In

The four pillars covered how classes are designed. This chapter covers how a class behaves once someone starts using it.

Python already knows how to print a number, compare two strings and measure a list. It knows none of that about a class you wrote. Until you teach it, your class stays a stranger to Python's own syntax.

## The Class Python Cannot Work With

Here is a `Post` class holding a caption and a like count. It works, in the sense that it stores what it was given.

```python
class Post:
    def __init__(self, caption, likes):
        self.caption = caption
        self.likes = likes


post1 = Post("First day at college!", 240)
post2 = Post("First day at college!", 240)

print(post1)
print(post1 == post2)
print(len(post1))
```

**Output**

```text
<__main__.Post object at 0x7f9bedbff2f0>
False
Traceback (most recent call last):
  File "app.py", line 12, in <module>
    print(len(post1))
          ^^^^^^^^^^
TypeError: object of type 'Post' has no len()
```

Three things went wrong on three lines.

- `print(post1)` showed a class name and a memory address. The number after `0x` will be different on your machine, and different again next run. Nothing about the caption or the likes appeared.
- `post1 == post2` said `False`, even though both posts hold exactly the same caption and the same like count.
- `len(post1)` did not run at all.

None of these are bugs in the class. Python has simply never been told three things: what printing a post means, what makes two posts equal, and what the length of a post is.

## Special Methods

A **special method** is a method Python calls on your behalf when you use ordinary syntax on an object.

You never call one directly. You write `print(post1)` and Python calls `__str__` for you. You write `post1 == post2` and Python calls `__eq__`.

The vocabulary is short.

- A **special method** has two underscores before and after its name, like `__str__`.
- Because of those underscores they are also called **dunder methods**, short for *double underscore*. Some books call them *magic methods*.
- Giving your class a special method so that an operator works on it is called **operator overloading**.

You have already written one. `__init__` is a special method — you never call `__init__` yourself, you write `Post("Exam over!", 90)` and Python calls it.

WhatsApp shows why this matters. A message on screen is an object in code. Yet it prints as readable text, sorts by time, and compares as equal when it is the same message. Somebody taught those objects how to print, sort and compare. Without that, every screen would be full of memory addresses.

## Controlling What print() Shows

Two special methods deal with turning an object into text.

- `__str__` returns text for a person reading the screen.
- `__repr__` returns text for a programmer debugging the code, and usually looks like the code that would rebuild the object.

```python
class Post:
    def __init__(self, caption, likes):
        self.caption = caption
        self.likes = likes

    def __str__(self):
        return f"{self.caption} — {self.likes} likes"

    def __repr__(self):
        return f"Post(caption={self.caption!r}, likes={self.likes})"


post1 = Post("First day at college!", 240)

print(post1)
print(repr(post1))
```

**Output**

```text
First day at college! — 240 likes
Post(caption='First day at college!', likes=240)
```

`print()` used `__str__`. The built-in `repr()` used `__repr__`.

If you write only one of the two, write `__repr__`. When `__str__` is missing, `print()` falls back to `__repr__`, so one method covers both jobs.

Printing a list of objects also uses `__repr__`, not `__str__`. `print([post1])` shows `[Post(caption='First day at college!', likes=240)]`. That is why `__repr__` is the one worth writing first.

## Comparing, Measuring and Adding Posts

Two more special methods fix the other two failures.

- `__eq__` decides what `==` means for your objects.
- `__len__` decides what `len()` returns.
- `__add__` decides what `+` does with two of your objects.

Each of these takes a second parameter named `other`. When you write `post1 == post2`, Python calls `post1.__eq__(post2)`, so `self` is the object on the left of the operator and `other` is the object on the right.

```python
class Post:
    def __init__(self, caption, likes):
        self.caption = caption
        self.likes = likes

    def __eq__(self, other):
        return self.caption == other.caption and self.likes == other.likes

    def __len__(self):
        return len(self.caption)

    def __add__(self, other):
        return self.likes + other.likes


post1 = Post("First day at college!", 240)
post2 = Post("First day at college!", 240)
post3 = Post("Exam over!", 90)

print(post1 == post2)
print(post1 == post3)
print(len(post1))
print(post1 + post3)
```

**Output**

```text
True
False
21
330
```

`==` now compares the caption and the likes instead of asking whether the two objects sit at the same place in memory. `len()` returns the number of characters in the caption, which is the count Instagram uses against its caption limit.

`post1 + post3` gave `330`, the two like counts added. Nothing about `+` forced that meaning. `__add__` could have joined the captions instead, or returned a whole new post. Python supplies the operator and you decide what it does — that is what overloading an operator means.

Every operator works this way. The method name is fixed; what it returns is yours to decide.

| Write this method | And this starts working |
|---|---|
| `__str__` | `print(post)` |
| `__repr__` | `repr(post)` |
| `__eq__` | `post1 == post2` |
| `__lt__` | `post1 < post2` |
| `__len__` | `len(post)` |
| `__add__` | `post1 + post2` |

## Dataclasses

Look at what the `Post` class now costs. A constructor, a `__repr__` and an `__eq__` come to roughly fifteen lines. Every one of them only moves `caption` and `likes` around. The class carries two pieces of data and a page of ceremony.

A **dataclass** is a class where Python writes those methods for you.

- The `dataclasses` module is built into Python. `from dataclasses import dataclass` brings in what you need.
- `@dataclass` written above a class tells Python to generate `__init__`, `__repr__` and `__eq__` from the fields you list.
- A **field** is a name written in the class body with a type after it, such as `caption: str`.

That `: str` part is a **type hint**. It says what kind of value the field is meant to hold. Python does not check it while the program runs, but `@dataclass` reads it to find out which names are fields.

```python
from dataclasses import dataclass


@dataclass
class Post:
    caption: str
    likes: int

    def is_popular(self):
        return self.likes > 100


post1 = Post("First day at college!", 240)

print(post1)
print(post1.is_popular())
```

**Output**

```text
Post(caption='First day at college!', likes=240)
True
```

Two field lines describe the data. There is no `__init__`, no `__repr__` and no `__eq__` anywhere in the class.

Yet `Post("First day at college!", 240)` worked and printing worked. Python generated those methods from the two field lines. Comparing two posts with `==` works as well, from the generated `__eq__`.

`is_popular()` is there to make one point clear: **a dataclass is an ordinary class.** It can hold any method you want. The decorator only adds the boring ones; everything you already know about writing methods still applies.

## Default Values and Lists

A field can be given a default, written the same way as a normal assignment.

One rule comes with defaults: fields without a default must be listed first. Putting `likes: int = 0` above `caption: str` raises `TypeError: non-default argument 'caption' follows default argument`. The generated `__init__` would otherwise have a required parameter after an optional one, which Python does not allow.

Lists need one extra step. Writing `tags: list = []` raises a `ValueError` telling you to use `default_factory` instead. A single list written in the class body would be shared by every post ever created.

```python
from dataclasses import dataclass, field


@dataclass
class Post:
    caption: str
    likes: int = 0
    tags: list = field(default_factory=list)


post1 = Post("Exam over!")
post1.tags.append("college")

print(post1)
print(Post("Study time", 12, ["notes", "python"]))
```

**Output**

```text
Post(caption='Exam over!', likes=0, tags=['college'])
Post(caption='Study time', likes=12, tags=['notes', 'python'])
```

`Post("Exam over!")` supplied only the caption. `likes` fell back to `0` and `tags` to a fresh empty list. `field(default_factory=list)` means *make a new list for every object*, which is exactly what was needed.

## Frozen Dataclasses

Some data should never change after it is created. A post's original caption, a payment amount, a booking reference.

Writing `@dataclass(frozen=True)` makes every field read-only.

```python
from dataclasses import dataclass


@dataclass(frozen=True)
class Post:
    caption: str
    likes: int


post1 = Post("First day at college!", 240)
print(post1)

post1.likes = 5000
```

**Output**

```text
Post(caption='First day at college!', likes=240)
Traceback (most recent call last):
  File "app.py", line 13, in <module>
    post1.likes = 5000
    ^^^^^^^^^^^
  File "<string>", line 4, in __setattr__
dataclasses.FrozenInstanceError: cannot assign to field 'likes'
```

Reading worked. Writing was refused. `frozen=True` is the shortest way to say *this data is final*.

## Sorting Dataclass Objects

`order=True` is the other common option. It generates `__lt__`, `__gt__` and the rest, so `sorted()` works on your objects with no extra code.

Comparison runs field by field, in the order the fields are written. Put the field you want to sort by first.

```python
from dataclasses import dataclass


@dataclass(order=True)
class Post:
    likes: int
    caption: str


posts = [Post(240, "First day at college!"), Post(90, "Exam over!"), Post(512, "Fest!")]

for post in sorted(posts):
    print(post)
```

**Output**

```text
Post(likes=90, caption='Exam over!')
Post(likes=240, caption='First day at college!')
Post(likes=512, caption='Fest!')
```

`likes` is written first, so posts sort by like count. Writing `caption` first would have sorted them alphabetically instead.

## Choosing Between a Class and a Dataclass

A dataclass is not a replacement for the classes in the earlier chapters. It is the right tool for one job.

| Use a dataclass when | Use a normal class when |
|---|---|
| The class mainly holds data | The class mainly does work |
| You want printing and `==` for free | Behaviour matters more than fields |
| Fields are known and few | Data must be validated or hidden behind getters |

The encapsulated `User` class, with its private follower count and its checking setter, should stay a normal class. Its whole purpose is to control access, and a dataclass exposes every field openly. A `Post` that only carries a caption, a like count and some tags is exactly what a dataclass is for.

## Practice Exercises

Build a `Comment` class for the app, one step at a time. Each task continues the file from the task before it.

1. **See the problem.** Write an ordinary class `Comment` with `__init__(self, text, likes)`. Create two comments with the same text and the same likes. Print one of them, then print whether the two are equal. Note both results.

2. **Teach it to print.** Add `__repr__` returning something like `Comment(text='Nice!', likes=3)`. Print the object again and compare with what you saw in task 1.

3. **Teach it to compare.** Add `__eq__` returning `True` when the text and the likes both match. Print the comparison again and confirm it is now `True`.

4. **Let Python write it.** Start a new `Comment` class using `@dataclass`, with the fields `text: str` and `likes: int`. Add an ordinary method `is_liked()` that returns `True` when likes are above zero, to prove a dataclass takes normal methods. Keep your earlier version too, and count the lines each one took.

5. **Add a default, then freeze, then sort.** Give `likes` a default of `0` and create a comment with only the text. Try putting the defaulted field first and read the `TypeError` it causes, then put it back. Change the decorator to `@dataclass(frozen=True)`, attempt to assign a new value to `likes`, and read that error too. Finally use `@dataclass(order=True)` with `likes` written first, and print a sorted list of three comments.

---

Special methods are how a class stops being a stranger to Python's syntax. Dataclasses are how you get the common ones without typing them.

Neither adds a new idea to object-oriented programming. Both remove work. The classes that come out are shorter and easier to read than the one this chapter started with.

---

## Reference Links

- [Python Official Docs — Special Method Names](https://docs.python.org/3/reference/datamodel.html#special-method-names)
- [Python Official Docs — `dataclasses`](https://docs.python.org/3/library/dataclasses.html)
- [GeeksforGeeks — Understanding Python Dataclasses](https://www.geeksforgeeks.org/python/understanding-python-dataclasses/)
- [GeeksforGeeks — Data Classes Decorator Parameters](https://www.geeksforgeeks.org/python/data-classes-in-python-set-2-decorator-parameters/)
