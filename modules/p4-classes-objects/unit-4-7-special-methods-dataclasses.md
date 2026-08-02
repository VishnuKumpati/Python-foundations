# Special Methods & Dataclasses

**Teaching Python How to Think About Your Objects.**

Every chapter so far has asked "what problem does this solve?" This one asks something different: what does Python actually *do* when it encounters an operation on your object? By the end of it, you'll be thinking a little like the interpreter itself.

---

## Python Meets Your Object

```python
print(student)
len(student)
student1 + student2
student1 == student2
```

These operations work perfectly for strings, lists, and numbers:

```python
print("hello")      # hello
len([1, 2, 3])       # 3
"a" + "b"            # ab
[1, 2] == [1, 2]     # True
```

But `student` isn't a string, a list, or a number — it's a class we wrote ourselves. So how does Python know what `print(student)` or `student1 + student2` is even supposed to mean?

It doesn't guess. Every time Python meets one of these operations, it asks the object what to do — by calling one of the object's special methods.

---

## Special Methods vs. Normal Methods — Why "Special"?

Every method we've written so far — `login()`, `logout()`, `enroll_course()` — only runs when we explicitly call it:

```python
student.login()
```

We decide when that line runs, and nothing happens until we write it.

Special methods work the other way around. We rarely call them ourselves — Python calls them automatically, whenever it needs one:

```python
student.login()   # You call it.
print(student)     # Python calls __str__() for you.
```

That automatic behavior is what makes them "special":

| | Normal Method | Special Method |
|---|---|---|
| Called by | You | Usually Python, automatically |
| Name | Anything you choose | Fixed names, like `__str__`, `__len__` |
| Purpose | Your own business logic | Connects your object to a built-in Python operation |

---

## You've Already Met One

This isn't actually the first special method you've seen. Every class you've written has included this:

```python
def __init__(self, ...):
```

`__init__()` is a special method too. Python has been calling it automatically since the first day you wrote a class — every time you typed `Student("Rahul")`. We just never stopped to ask *why* Python called it on its own. By the end of this chapter, we will.

---

## You Rarely Call These Yourself

One more thing before we start. You *can* call a special method directly:

```python
student.__str__()
```

But that's almost never how they're actually used. In real code, you write ordinary Python — `print(student)`, `len(student)`, `student1 == student2` — and Python is the one reaching for the matching special method behind the scenes. You write the everyday syntax; Python does the calling.

---

## The General Rule

Everything in this chapter is one rule, applied to a different operation each time:

> **Every built-in Python operation has a matching special method.**

`print()` has `__str__()`. `len()` has `__len__()`. `+` has `__add__()`. Hold on to this rule — the summary table at the end of the chapter is just this same idea, filled in ten times.

---

## The Pattern Behind Every Section

```
Python encounters ...
        ↓
Python asks ...
        ↓
special method
        ↓
you implement it
        ↓
Python continues
```

Once this rhythm feels familiar, the rest of Python's special methods stop being a list to memorize and start being one repeated conversation.

---

## Section 1 — `__init__()`, Revisited

Python encounters:

```python
Student("Rahul")
```

You already know that Python calls `__init__()` here to set up the new object. What you may not have asked is: *who hands `__init__()` an empty object to fill in, in the first place?* Hold that question — we'll answer it properly once a few more special methods are on the table, in Section 10.

---

## Section 2 — Printing Objects

Python encounters:

```python
print(student)
```

Python thinks: "I don't know how to display this object."

Python asks: `student.__str__()`

You teach Python:

```python
class Student:
    def __init__(self, name, courses=None):
        self.name = name
        self.courses = courses or []

    def __str__(self):
        return f"Student: {self.name}"
```

Before:

```text
<__main__.Student object at 0x000001A2B3C4D5E6>
```

After:

```python
student = Student("Rahul")
print(student)
```

```text
Student: Rahul
```

---

## Section 3 — Developer Representation

Sometimes Python needs a more detailed view of an object — for example while debugging, or when displaying an object in the Python shell without `print()`:

```python
>>> student
```

Python asks: `student.__repr__()`

| | Audience | Method |
|---|---|---|
| `print(student)` | Users | `__str__()` |
| `student`, shown without `print()` | Developers, while debugging | `__repr__()` |

```python
class Student:
    def __init__(self, name, courses=None):
        self.name = name
        self.courses = courses or []

    def __str__(self):
        return f"Student: {self.name}"

    def __repr__(self):
        return f"Student(name={self.name!r}, courses={self.courses!r})"
```

```text
>>> student
Student(name='Rahul', courses=[])
```

`__str__()` reads like a sentence. `__repr__()` reads like something you could paste back into code.

---

## Section 4 — Finding Length

Python encounters:

```python
len(student)
```

Python asks: `student.__len__()`

```python
class Student:
    def __init__(self, name, courses=None):
        self.name = name
        self.courses = courses or []

    def __len__(self):
        return len(self.courses)
```

```python
student = Student("Rahul", ["Math", "Physics"])
print(len(student))
```

```text
2
```

`len()` never touches `self.courses` itself — it asks the object, and the object decides what "length" means for itself.

---

## Section 5 — Truth Value

Python encounters:

```python
if student:
```

Python asks: `student.__bool__()`

```python
class Student:
    def __init__(self, name, courses=None):
        self.name = name
        self.courses = courses or []

    def __bool__(self):
        return len(self.courses) > 0
```

```python
student = Student("Rahul")
if student:
    print("Enrolled")
else:
    print("Not enrolled yet")
```

```text
Not enrolled yet
```

`student` with no courses is `False`. Add one course, and the exact same `if student:` becomes `True` — nothing else in the code changes.

---

## Section 6 — Comparing Objects

Python encounters:

```python
student1 == student2
```

Python thinks: "How should I compare these?"

Python asks: `student1.__eq__(student2)`

```python
class Student:
    def __init__(self, name, courses=None):
        self.name = name
        self.courses = courses or []

    def __eq__(self, other):
        return self.name == other.name
```

```python
student1 = Student("Rahul")
student2 = Student("Rahul")
print(student1 == student2)
```

```text
True
```

Without `__eq__()`, `==` would only check whether the two are the same object in memory — not whether they represent the same student.

---

## Section 7 — Adding Objects

Python encounters:

```python
wallet1 + wallet2
```

Python asks: `wallet1.__add__(wallet2)`

```python
class Wallet:
    def __init__(self, balance):
        self.balance = balance

    def __add__(self, other):
        return Wallet(self.balance + other.balance)

    def __repr__(self):
        return f"Wallet(balance={self.balance})"
```

```python
wallet1 = Wallet(500)
wallet2 = Wallet(300)
print(wallet1 + wallet2)
```

```text
Wallet(balance=800)
```

`-`, `*`, and `/` work the same way, through `__sub__()`, `__mul__()`, and `__truediv__()`. Once `__add__()` makes sense, the rest are the same idea wearing a different name.

---

## Section 8 — Calling Objects

Python encounters:

```python
checker()
```

Python asks: `checker.__call__()`

A `Student` behaving like a function would feel forced, so here's a class built specifically to be called:

```python
class AttendanceChecker:
    def __init__(self, course):
        self.course = course

    def __call__(self, student_name):
        print(f"{student_name} marked present for {self.course}.")
```

```python
checker = AttendanceChecker("Physics")
checker("Rahul")
```

```text
Rahul marked present for Physics.
```

`__call__()` is what lets an object be used with the same `()` syntax as a function. `checker` still isn't a function — Python just doesn't see a difference once `__call__()` exists.

---

## Section 9 — Looping

Python encounters:

```python
for course in student:
```

Python asks: `student.__iter__()`

```python
class Student:
    def __init__(self, name, courses=None):
        self.name = name
        self.courses = courses or []

    def __iter__(self):
        return iter(self.courses)
```

```python
student = Student("Rahul", ["Math", "Physics", "Chemistry"])
for course in student:
    print(course)
```

```text
Math
Physics
Chemistry
```

Once Python receives what `__iter__()` returns, it repeatedly asks that for the next item until the loop runs out. `for` never knows anything about `courses` directly — it only knows to ask `student` for `__iter__()`.

---

## Section 10 — Object Creation, Completed

Back in Section 1, we said `__init__()` fills in a new object — but something has to create that empty object *before* `__init__()` can fill it in.

Python encounters:

```python
Student("Rahul")
```

```
Student("Rahul")
        ↓
Python calls Student.__new__()
        ↓
object created (empty, unassigned)
        ↓
Python calls __init__()
        ↓
object initialized (name = "Rahul")
```

`__new__()` builds the empty object; `__init__()` fills it in. Every class in this book has been using `__new__()` the whole time — it's just been invisible, because Python supplies a default one for free.

---

## Python Has Been Doing This All Along

| You Write | Python Internally Calls |
|---|---|
| `Student("Rahul")` | `Student.__new__()` → `__init__()` |
| `print(student)` | `student.__str__()` |
| `student` (without `print()`) | `student.__repr__()` |
| `len(student)` | `student.__len__()` |
| `if student:` | `student.__bool__()` |
| `student1 == student2` | `student1.__eq__(student2)` |
| `wallet1 + wallet2` | `wallet1.__add__(wallet2)` |
| `checker(...)` | `checker.__call__(...)` |
| `for x in student:` | `student.__iter__()` |

None of this was ever hidden magic. Every built-in operation Python offers is just a special method, called on your behalf, the moment you use the syntax.

---

## We've Been Writing These Methods Again and Again

Take a moment to look back at the classes we've written throughout this chapter. Different classes, different purposes — `Student`, `Wallet`, `AttendanceChecker`. And yet most of them began with the same handful of methods:

```python
__init__()
__repr__()
__eq__()
```

Earlier, writing these methods helped us understand how Python works underneath the surface. But now something has changed — we're no longer learning anything new by writing them. We're repeating ourselves.

Whenever programmers notice the same code appearing again and again, the natural question is: can Python write this code for us?

The answer is yes. That's exactly what dataclasses were created for.

---

## What Is a Dataclass?

A dataclass is a class whose main job is to hold data. Instead of writing `__init__()`, `__repr__()`, and `__eq__()` by hand, you describe the fields, and Python generates those methods for you.

---

## Why Do We Need It?

Here's a plain class holding three pieces of data about a student, written the way we have been all chapter:

```python
class Student:
    def __init__(self, name, age, gpa):
        self.name = name
        self.age = age
        self.gpa = gpa

    def __repr__(self):
        return f"Student(name={self.name!r}, age={self.age!r}, gpa={self.gpa!r})"

    def __eq__(self, other):
        return (self.name, self.age, self.gpa) == (other.name, other.age, other.gpa)
```

Here's the same class as a dataclass:

```python
from dataclasses import dataclass

@dataclass
class Student:
    name: str
    age: int
    gpa: float
```

Three lines instead of eleven — and it behaves identically.

---

## What Does It Generate?

```python
student1 = Student("Rahul", 20, 8.7)
student2 = Student("Rahul", 20, 8.7)

print(student1)
print(student1 == student2)
```

```text
Student(name='Rahul', age=20, gpa=8.7)
True
```

We never wrote `__init__()`, `__repr__()`, or `__eq__()` — `@dataclass` read the field list (`name: str`, `age: int`, `gpa: float`) and generated all three for us.

---

## The `@dataclass` Decorator

`@dataclass` is what tells Python to look at a class's annotated fields and build those methods automatically. Without it, `name: str` is just an annotation Python ignores at runtime. With it, that same line becomes an instruction: "add `name` as a parameter to `__init__()`, include it in `__repr__()`, and compare it in `__eq__()`."

---

## Default Values

Fields can have defaults, the same way function parameters can:

```python
@dataclass
class Student:
    name: str
    gpa: float = 0.0
```

Lists are the one place this gets tricky. You cannot write:

```python
@dataclass
class Student:
    name: str
    courses: list = []          # Error: mutable default not allowed
```

Every `Student` would end up sharing the *same* list, which is almost never what you want — the same trap as a mutable default argument. Dataclasses refuse to let you write this by accident, and ask for `field()` instead:

```python
from dataclasses import dataclass, field

@dataclass
class Student:
    name: str
    courses: list = field(default_factory=list)
```

`default_factory=list` tells Python to call `list()` fresh for every new `Student`, so each one gets its own empty list instead of sharing one.

---

## `frozen=True`

```python
@dataclass(frozen=True)
class Student:
    name: str
    age: int

student = Student("Rahul", 20)
student.age = 21
```

```text
dataclasses.FrozenInstanceError: cannot assign to field 'age'
```

`frozen=True` locks every field after `__init__()` runs. Use it when an object represents a fixed value that should never change once created.

---

## `order=True`

```python
@dataclass(order=True)
class Student:
    gpa: float
    name: str

students = [Student(8.7, "Rahul"), Student(9.2, "Meera")]
print(sorted(students))
```

```text
[Student(gpa=8.7, name='Rahul'), Student(gpa=9.2, name='Meera')]
```

`order=True` generates `__lt__()`, `__le__()`, `__gt__()`, and `__ge__()` for you, comparing fields in the order they're declared — here, by `gpa` first. That's what lets `sorted()` work without writing a comparison method by hand.

---

## When to Use Dataclasses

- The class's main job is to hold data — records, configuration, coordinates, API responses.
- You want `__init__()`, `__repr__()`, and `__eq__()`, and nothing about them needs to be unusual.
- You want sorting or ordering based on the fields themselves (`order=True`).

## When Not to Use Dataclasses

- The class has substantial behavior of its own — methods that do real work, not just store and return data.
- `__eq__()` or `__init__()` needs custom logic that field-by-field comparison or a plain constructor can't express.
- The class fits into an inheritance hierarchy with more complex setup than a flat list of fields.

A dataclass doesn't replace everything we've learned this chapter — it only removes the boilerplate for the small set of methods most classes were writing the same way anyway. Encapsulation, inheritance, polymorphism, and every special method beyond `__init__()`, `__repr__()`, and `__eq__()` still work exactly as they did before, on a dataclass or not.

---

## Chapter Summary

Throughout this chapter, we've taught Python how our objects should behave — how to print themselves, compare themselves, add themselves, loop over themselves, and even how to come into existence in the first place. Then we let Python take some of that work back: whenever `__init__()`, `__repr__()`, and `__eq__()` start looking the same across classes, `@dataclass` is there to write them for you, so you can spend your effort on what actually makes each class different.

---

## Reference Links

-   [Python Official Docs — Data Model (Special Method Names)](https://docs.python.org/3/reference/datamodel.html#special-method-names)
-   [Python Official Docs — Basic Customization (`__new__`, `__init__`, `__repr__`, `__eq__`)](https://docs.python.org/3/reference/datamodel.html#basic-customization)
-   [Python Official Docs — `dataclasses` Module](https://docs.python.org/3/library/dataclasses.html)
