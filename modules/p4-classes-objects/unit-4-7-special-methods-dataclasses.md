# Special Methods and Dataclasses

A `Creator` inherited `show_profile()` from its parent. That was enough until the creator needed a different profile display, and polymorphism gave the class its own version of that method.

But classes have another problem. Python itself expects objects to work with common operations. `print()`, `len()`, `==`, and `repr()` already have meanings. A normal class does not automatically know how those operations should describe its objects.

This chapter teaches how a class can define those behaviours. It then uses dataclasses to remove repetitive code when a class mainly stores data.

## Special Methods - Methods Python Calls for You

A special method has a name wrapped in double underscores, like `__str__` or `__len__`. Those underscores are part of the name, not decoration.

You never write `user.__str__()` yourself. You write `print(user)`, and Python finds the method on its own. Length works the same way through `__len__()`, and the equality operator through `__eq__()`.

## `__str__()` - Printing an Object in a Readable Way

Start with the `User` class from the social media example.

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

user = User("Rahul", "rahul@example.com")
print(user)  # <__main__.User object at 0x7f479b1ff0b0>
```

The hexadecimal address changes between runs. Your address will differ.

Python has no useful human-friendly description for this class yet. We can define one with `__str__()`.

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def __str__(self):
        return f"{self.name} ({self.email})"

user = User("Rahul", "rahul@example.com")
print(user)  # Rahul (rahul@example.com)
```

`print(user)` now uses `User.__str__()`. The method must return a string, and it should stay short enough for a person to read.

## `__repr__()` - Showing an Object's Details

`__str__()` is for the person reading the screen. `__repr__()` is for you, inspecting the object while building the program, so it usually carries more detail.

The difference becomes clear when both are defined.

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def __str__(self):
        return f"{self.name} ({self.email})"

    def __repr__(self):
        return f"User(name={self.name!r}, email={self.email!r})"

user = User("Rahul", "rahul@example.com")

print(str(user))  # Rahul (rahul@example.com)
print(repr(user))  # User(name='Rahul', email='rahul@example.com')
```

The `!r` inside the f-string asks Python for the `repr()` form of the value.

There is also a fallback. A class that defines `__repr__()` but no `__str__()` gets that same method used whenever a string is requested, so one method covers both jobs.

## `__eq__()` - Checking If Two Objects Are Equal

Two objects can contain the same data without being the same object.

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

rahul_1 = User("Rahul", "rahul@example.com")
rahul_2 = User("Rahul", "rahul@example.com")

print(rahul_1 == rahul_2)  # False
```

The two objects hold the same data but are not the same object, and the class has never said what equality should mean. `__eq__()` says it.

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def __eq__(self, other):
        return self.name == other.name and self.email == other.email

rahul_1 = User("Rahul", "rahul@example.com")
rahul_2 = User("Rahul", "rahul@example.com")

print(rahul_1 == rahul_2)  # True
```

Now `==` compares the two users by the rule we wrote. Its second parameter, `other`, is the object on the right of the operator.

This is called operator overloading. The operator already exists, but our class gives it a meaning for its objects.

## `__len__()` - Giving an Object a Length

A `User` object has no length of its own. Our social media user can define length as the number of posts.

```python
class User:
    def __init__(self, name):
        self.name = name
        self.posts = []

    def __len__(self):
        return len(self.posts)

rahul = User("Rahul")
rahul.posts.append("Hello")
rahul.posts.append("Learning Python")

print(len(rahul))  # 2
```

`len()` calls the object's `__len__()` method, which must return an integer. What length means is the class's own decision, and here we chose the number of posts.

## Operator Overloading - Using Operators on Your Own Objects

The same idea extends beyond the four methods above. `__add__()` gives meaning to the plus operator, and `__lt__()` to the less-than operator.

Do not memorise the list. Learn the connection between an operation and the method it reaches.

| Operation | Special method | Purpose |
|---|---|---|
| `print(user)` | `__str__()` | Human-friendly text |
| `repr(user)` | `__repr__()` | Developer-friendly representation |
| `user1 == user2` | `__eq__()` | Defines equality |
| `len(user)` | `__len__()` | Defines the object's length |
| `user1 + user2` | `__add__()` | Defines addition |
| `user1 < user2` | `__lt__()` | Defines ordering |

## The Problem - Too Much Repeated Code

The `User` class now needs several pieces of code.

```python
class User:
    def __init__(self, name, email, followers):
        self.name = name
        self.email = email
        self.followers = followers

    def __repr__(self):
        return f"User(name={self.name!r}, email={self.email!r}, followers={self.followers})"

    def __eq__(self, other):
        return (self.name == other.name and self.email == other.email
                and self.followers == other.followers)

rahul = User("Rahul", "rahul@example.com", 120)
print(rahul)  # User(name='Rahul', email='rahul@example.com', followers=120)
print(rahul == User("Rahul", "rahul@example.com", 120))  # True
```

The class works. But almost every line exists only to store data and provide standard representation and equality. That is where dataclasses help.

## `@dataclass` - Letting Python Generate the Methods

A dataclass reduces repetitive code in classes that mainly hold data.

The `@dataclass` decorator comes from Python's `dataclasses` module. A decorator applies behaviour to the class written beneath it, and this one tells Python to generate methods for that class.

Fields are written with type hints such as `name: str`. The hint marks the name as a field and states the kind of value it is expected to hold.

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    email: str
    followers: int

rahul = User("Rahul", "rahul@example.com", 120)

print(rahul)  # User(name='Rahul', email='rahul@example.com', followers=120)
print(rahul == User("Rahul", "rahul@example.com", 120))  # True
```

The decorator generated `__init__()`, `__repr__()` and `__eq__()`, so none of the three had to be written. You describe the data, and the dataclass supplies the common methods for it. The generated equality compares fields between instances of the same dataclass.

**Where you see it:** records such as social media profiles, payment details, student records, and API data objects.

## Type Hints - Declaring the Fields

Each line in a dataclass declares a field. `name: str` declares a name field expected to hold a string, and `followers: int` declares one expected to hold an integer.

These hints do not convert values or validate them. The dataclass reads them to know which names are fields, and the generated constructor takes them in the order they are written.

## Default Values - Making a Field Optional

A field can have a default value.

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    email: str
    followers: int = 0

rahul = User("Rahul", "rahul@example.com")
priya = User("Priya", "priya@example.com", 250)

print(rahul)  # User(name='Rahul', email='rahul@example.com', followers=0)
print(priya)  # User(name='Priya', email='priya@example.com', followers=250)
```

Rahul supplies no `followers`, so the default `0` is used. Priya supplies a value, so hers is kept.

A field without a default must come before a field with a default.

## `default_factory` - Giving Each Object Its Own List

Lists and dictionaries are mutable objects. A single list must not accidentally become the default list for every object.

For a dataclass, use `field(default_factory=list)` when each object needs its own new list.

```python
from dataclasses import dataclass, field

@dataclass
class User:
    name: str
    posts: list = field(default_factory=list)

rahul = User("Rahul")
priya = User("Priya")

rahul.posts.append("My first post")

print(rahul.posts)  # ['My first post']
print(priya.posts)  # []
```

`default_factory=list` tells the dataclass to call `list()` each time a new object needs its default posts value. Every object therefore gets a separate list.

Do not write `posts: list = []` when every instance needs a fresh list. Python's dataclass machinery rejects mutable defaults such as lists, dicts and sets.

**Where you see it:** a social media account needs its own posts, followers, or saved items.

## `frozen=True` - Making an Object Unchangeable

Some objects should behave like a fixed record once created. `@dataclass(frozen=True)` does that.

With `frozen=True`, assigning a new value to a field is blocked. Writing `rahul.name = "Arun"` produces a `FrozenInstanceError`.

This does not make the object immutable in every sense. It blocks assignment to the dataclass's own fields.

**Where you see it:** a payment record or configuration record may need its stored values to remain unchanged after creation.

## `order=True` - Comparing and Sorting Objects

Adding `order=True` generates `__lt__()`, `__le__()`, `__gt__()` and `__ge__()`. They compare fields in the order the fields are written.

```python
from dataclasses import dataclass

@dataclass(order=True)
class User:
    followers: int
    name: str

rahul = User(120, "Rahul")
priya = User(250, "Priya")

print(rahul < priya)  # True
```

The first field is `followers`, so the comparison uses follower counts first.

Equality is generated by default; ordering is not. Use it only when one object being less than another actually means something.

**Where you see it:** a creator dashboard could order creators by a chosen numeric field such as follower count.

## Normal Methods - Adding Your Own Behaviour

A dataclass does not remove normal class behaviour. It reduces repetitive data-handling code.

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    email: str
    followers: int = 0

    def greet(self):
        return f"Hello, {self.name}"

rahul = User("Rahul", "rahul@example.com", 120)

print(rahul)  # User(name='Rahul', email='rahul@example.com', followers=120)
print(rahul.greet())  # Hello, Rahul
```

The dataclass generated the common data methods. We wrote `greet()` because it represents application behaviour. That gives a useful boundary:

- Use fields to describe the object's data.
- Write methods when the object needs behaviour.
- Let `@dataclass` handle common data-oriented methods when their default behaviour is correct.

## Combining Both - A Dataclass with Your Own `__str__()`

The decorator generates `__repr__()` and `__eq__()`. You can still write `__str__()` yourself when a shorter display is wanted.

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    email: str
    followers: int = 0

    def __str__(self):
        return f"{self.name} - {self.followers} followers"

rahul = User("Rahul", "rahul@example.com", 120)

print(rahul)  # Rahul - 120 followers
print(repr(rahul))  # User(name='Rahul', email='rahul@example.com', followers=120)
print(rahul == User("Rahul", "rahul@example.com", 120))  # True
```

The hand-written method handles the display for people. The generated ones keep working untouched.

## Ordinary Classes - When Not to Use a Dataclass

A dataclass is useful when a class mainly represents data. It is not automatically the best choice for every class.

A class may need a carefully controlled constructor, complex validation, or behaviour that should not follow generated equality and representation rules. A social media service class, for instance, performs actions rather than representing one record, and it stores no data of its own.

The key distinction is responsibility. A data record is a strong dataclass candidate. A service that performs application actions is usually an ordinary class.

## Manual Methods or `@dataclass` - Choosing Between Them

A manual class makes you write `__init__()`, `__repr__()` and `__eq__()` yourself. A dataclass lets you declare the fields and asks Python to generate them.

Use manual methods when the generated behaviour is not what your class needs. Use a dataclass when the class is mainly a structured collection of data and the generated behaviour matches its meaning.

The two ideas solve different problems and work well together. Special methods connect your class to Python's built-in operations. Dataclasses remove repetitive code from data-focused classes.

## Practice Exercises

Continue with the social media example, one step at a time. Each task continues the file from the task before it.

1. **Add a special method.** Create a `User` class with a name and an email. Implement `__str__()` so printing the object shows `Rahul - rahul@example.com`.

2. **Define equality.** Create two users holding the same name and email. Implement `__eq__()` so comparing them with `==` prints True.

3. **Define length.** Add a posts list to the class and implement `__len__()` so `len()` returns the number of posts. Add three posts and check the result.

4. **Refactor with a dataclass.** Replace the hand-written `__init__()`, `__repr__()` and `__eq__()` with `@dataclass`, keeping the same three fields. Confirm printing and comparing still work.

5. **Give every user a separate post list.** Add a posts field using `field(default_factory=list)`. Create Rahul and Priya, add a post only to Rahul, and confirm Priya's list is still empty.

6. **Trigger a failure deliberately.** Create a class with no `__len__()` and call `len()` on its object. Record the error Python reports. Then add the method, make the same call work, and explain why it could not before.

7. **Choose the right design.** Create one class that stores user data and one that performs actions on it. Decide which is the good dataclass candidate and write one sentence explaining why.

---

## Reference Links

- [Python Official Docs — Special Method Names](https://docs.python.org/3/reference/datamodel.html#special-method-names)
- [Python Official Docs — `dataclasses`](https://docs.python.org/3/library/dataclasses.html)
- [Python Official Docs — `dataclasses.field`](https://docs.python.org/3/library/dataclasses.html#dataclasses.field)
- [GeeksforGeeks — Understanding Python Dataclasses](https://www.geeksforgeeks.org/python/understanding-python-dataclasses/)
- [GeeksforGeeks — Data Classes Decorator Parameters](https://www.geeksforgeeks.org/python/data-classes-in-python-set-2-decorator-parameters/)
