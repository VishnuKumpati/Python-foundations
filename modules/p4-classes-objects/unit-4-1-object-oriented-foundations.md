# Object-Oriented Foundations

Unit 3.5 closed out Module III by showing you how Python actually walks through a collection under the hood, rounding off everything you had learned about lists, tuples, sets, and dictionaries — four different ways of organising many *values* under one name. That skill is genuinely powerful: a dictionary can hold a student's name and marks, a list can hold a hundred orders, a set can track which seats on a train are already booked. But notice what none of those structures do. A dictionary happily lets any function in your program reach in and overwrite `student["marks"]` with a negative number, a string, or nothing sensible at all. Nothing in the language ties the data to the operations that are actually allowed to touch it — that connection exists only in your own memory, and only for as long as you're careful.

Module IV, starting here, is about closing exactly that gap. Instead of organising values alone, you're about to learn how to organise a value *and* the behaviour that's allowed to act on it, together, as one unit. A bank account isn't just a number sitting in a dictionary — it's a balance plus a strict set of rules about how that balance may change. A student isn't just a name and a mark — it's data plus the ability to calculate its own grade. This unit builds the four ideas that make that bundling possible: the abstract data type, the class, the object, and the constructor-plus-methods pairing that gives an object both its data and its actions.

By the end, you'll have built classes of your own, created several independent objects from the same blueprint, and be ready for Unit 4.2, where you'll see how classes can be extended, protected, and reshaped — the four pillars of OOP.

---

## The gap a dictionary and a function can't close

Picture how you'd track one student's marks using only what you already know:

```python
student = {"name": "Rahul", "marks": [85, 90, 78]}

def calculate_average(marks):
    return sum(marks) / len(marks)

print(calculate_average(student["marks"]))
```

```
84.33333333333333
```

This works perfectly well for one student. Now picture a real college with five thousand students, each needing several pieces of data — name, roll number, marks, department — and several operations: calculate an average, display a summary, check pass or fail. Keep using separate dictionaries and separate floating functions, and nothing in Python stops you from accidentally handing one student's marks into a function that was meant to process somebody else's data. The data and the operations that logically belong together are held apart, connected only by your own careful naming and memory — and at five thousand students, that discipline eventually slips.

| App or system | What has to stay bundled, not just nearby |
|---|---|
| Banking app | An account's balance, plus the exact deposit/withdraw logic allowed to change it |
| UPI payment app | A transaction's payer, payee, and amount, plus the logic that marks it successful |
| Railway booking (IRCTC-style) | A ticket's passenger and fare, plus the logic that confirms or cancels it |
| Swiggy-style delivery app | An order's items and total, plus the logic that applies a coupon or marks it delivered |

In every row, the data on its own is inert — it's the actions performed *on* that data that make it meaningful, and those actions only make sense in the context of that one specific account, transaction, or order. Pick one row and say out loud, in one sentence, what could go wrong if the data and the logic that changes it were kept as separate, loosely connected pieces instead of one bundled unit — that's precisely the problem this unit's central idea, the **class**, exists to solve.

## Describing a "thing" before you write any code: the abstract data type

Before touching a keyboard, it helps to describe what a "thing" is and what it can do, in plain words. That description is called an **abstract data type (ADT)**: it lists the thing's **data** (its attributes) and its **behaviour** (what actions it supports), without saying one word about how any of it will actually be implemented in Python.

For a student, an ADT might read like this:

- **Data** — name, roll number, marks
- **Behaviour** — calculate a grade, display a summary

Notice that nothing here is Python yet — no `def`, no `class`, no code at all. That's deliberate. Thinking in ADT terms first is what tells you, before you write a single line, exactly which attributes and which methods a class actually needs. Try it yourself before reading on: sketch the ADT for a bank account in the same two-line shape — data, then behaviour — and you'll already have the outline for the `BankAccount` class you're about to build later in this unit.

## The class: a blueprint you write once

Think of a class the way you'd think of a cookie cutter. The cutter itself contains no dough, no sugar, no actual cookie — it's just a shape, a template describing what every cookie stamped out with it will look like. A **class** is exactly that: a blueprint that defines what attributes and what methods every object built from it will have. A class, on its own, never holds any real data — it only describes the *shape* that data will take, once something is actually built from it.

```python
class Student:
    pass
```

Run this cell and — predict it before you check — nothing happens. No output at all. Writing a `class` block, just like writing a `def` block back in Unit 2.3, doesn't run anything by itself; it only teaches Python the name `Student` and what any object built from it will contain. `pass` here means exactly what it meant inside an empty function or loop body: "deliberately nothing here yet."

`class Student:` opens the blueprint: the reserved keyword `class`, a name you choose (in `PascalCase` — capitalise each word, no underscores, so `Student` and `BankAccount`, never `student` or `bank_account`), and a colon, exactly mirroring how a function's `def name():` line opens a function body.

## The object: one specific thing built from the blueprint

An **object**, also called an **instance**, is one specific thing created from a class, holding its own real values. If the class is the cookie cutter, the object is one actual cookie — a real, edible thing you could hand to somebody, made using the cutter's shape but existing independently of it and of every other cookie stamped from the same cutter.

You create an object by calling the class name as though it were a function — this act is called **instantiation**:

```python
class Student:
    pass

first_student = Student()
print(type(first_student))
```

```
<class '__main__.Student'>
```

`Student()` is what actually produces a usable object. `type()` — the exact same built-in you've used since Unit 1.2 to confirm whether a value was an `int` or a `str` — confirms here that `first_student` was built from the `Student` class.

**Instantiating a class always requires parentheses, even when nothing goes inside them — the parentheses are precisely what trigger the creation of a new object.** Write `account = BankAccount` without parentheses, and `account` doesn't become a new account at all; it simply becomes another name pointing at the class itself, and calling a method on it later fails in a confusing way, because no object — and no instance attributes — were ever actually created.

You can build as many independent objects as you like from one class, and none of them share data with each other:

```python
s1 = Student()
s2 = Student()

print(s1 is s2)
```

```
False
```

`is` checks whether two names point to the *exact same* object in memory, not whether they merely look alike. `False` here confirms `Student()` produced two genuinely separate objects, even though both came from the identical blueprint — exactly the way one cookie is never literally the same cookie as another, even when both were cut from the same cutter.

| Aspect | Class | Object (instance) |
|---|---|---|
| What it is | A blueprint / template | One specific thing built from the blueprint |
| How many exist | Usually one definition in your code | As many as you choose to create |
| Holds actual data? | No — only describes what data instances will hold | Yes — each object holds its own real values |
| Created with | `class ClassName:` | `ClassName(...)` — instantiation |
| Example | `Student` — the idea of "a student" | `Student("Priya Nair", 91)` — one real student |

## Giving every object its starting data: the constructor, `__init__`

An empty class is a start, but a `Student` with no name and no marks isn't useful yet. What you need is a way to hand each new object its own data the moment it's created. That's the job of the **constructor** — a special method named exactly `__init__` that Python calls automatically, every single time you instantiate a class:

```python
class Student:
    def __init__(self, name, marks):
        self.name = name
        self.marks = marks

s1 = Student("Priya Nair", 91)
s2 = Student("Arjun Rao", 68)

print(s1.name, s1.marks)
print(s2.name, s2.marks)
```

```
Priya Nair 91
Arjun Rao 68
```

`def __init__(self, name, marks):` looks exactly like an ordinary function definition, because it is one — it's just written inside a class body and given this one exact reserved name. `self.name = name` takes the `name` argument you passed in and stores it as an **instance attribute** — a piece of data that belongs to this one specific object alone. The same happens for `marks`. **You never call `__init__` directly yourself** — writing `Student("Priya Nair", 91)` is what silently triggers Python to build a new, empty object and immediately call `__init__` on it, before handing the finished object back to you.

Before checking, predict what `s1.name` and `s2.name` will each print, given that they were built from the exact same class with different arguments. Each object keeps its own independent copy of `name` and `marks` — changing `s1.marks` later would have absolutely no effect on `s2.marks`, in the same way editing one cookie's icing never touches any other cookie cut from the same cutter.

## Instance attributes vs class attributes

Every attribute you've seen so far — `self.name`, `self.marks` — is an **instance attribute**: set inside `__init__` through `self`, and unique to the one object it belongs to. But some data genuinely is identical across every object you'll ever create from a class — a college's name, for instance, doesn't change from student to student. For exactly that case, Python gives you a **class attribute**: a value written directly in the class body, outside any method, shared by every instance:

```python
class Student:
    college_name = "Crescent College"   # class attribute

    def __init__(self, name, marks):
        self.name = name                # instance attribute
        self.marks = marks              # instance attribute

s1 = Student("Priya Nair", 91)
s2 = Student("Arjun Rao", 68)

print(s1.college_name)
print(s2.college_name)
```

```
Crescent College
Crescent College
```

Notice `college_name` never appears inside `__init__`, and neither object was ever individually assigned it — both simply read the identical string straight from the class itself.

| | Instance attribute | Class attribute |
|---|---|---|
| Defined | Inside `__init__` (or another method), via `self.name = value` | Directly in the class body, outside every method |
| Shared across objects? | No — each object gets its own copy | Yes — until one instance overrides it locally |
| Right for | Data that varies per object — almost everything you'll model | Data that's genuinely identical for every object |
| Example | `self.marks` — every student's marks differ | `college_name` — every student attends the same college |

**Use a class attribute only for a value that's genuinely identical across every object; reach for an instance attribute, set through `self` inside `__init__`, for anything that could ever differ per object — which, in practice, is nearly everything you model.** If one particular student later transfers colleges and you write `s2.college_name = "Northside College"` directly, only `s2` sees that new value from then on; `s1` still reads the original class attribute untouched. Assigning through `self` on one instance quietly creates a brand-new instance attribute that shadows the class attribute for that one object alone, without ever touching the class attribute itself.

## Methods and `self`: the piece that ties data to behaviour

Data alone still isn't the point of this unit — the whole idea was to bundle data *with* the behaviour that acts on it. A **method** is simply a function defined inside a class body, and it's what gives an object the "behaviour" half of its ADT:

```python
class Student:
    college_name = "Crescent College"

    def __init__(self, name, marks):
        self.name = name
        self.marks = marks

    def calculate_grade(self):
        if self.marks >= 90:
            return "A"
        elif self.marks >= 75:
            return "B"
        else:
            return "C"

    def describe(self):
        return f"[{self.college_name}] {self.name} scored {self.marks} marks - Grade {self.calculate_grade()}"

s1 = Student("Priya Nair", 91)
s2 = Student("Arjun Rao", 68)

print(s1.describe())
print(s2.describe())
```

```
[Crescent College] Priya Nair scored 91 marks - Grade A
[Crescent College] Arjun Rao scored 68 marks - Grade C
```

Syntactically, `calculate_grade` and `describe` are written with the exact same `def` keyword as any function you met in Unit 2.3. What makes them methods, rather than plain functions, is simply that they live inside a `class` body and take **`self`** as their first parameter.

`self` refers to the specific object a method was called on, and Python supplies it automatically — you never pass it yourself. Concretely, `s1.describe()` is handled by Python as though you had written `Student.describe(s1)`: the object to the left of the dot becomes the value bound to `self` inside the method. That's why `self.marks` inside `calculate_grade()` means "*this* object's marks" — when the call is `s1.calculate_grade()`, `self` is `s1`; when it's `s2.calculate_grade()`, `self` is `s2`, and the exact same code produces a different answer each time because it's reading different data.

Before checking, try predicting what `s2.describe()` prints, tracing through `calculate_grade()` by hand the way you'd trace an `if`/`elif`/`else` chain from Unit 2.1 — `self.marks` is `68`, which is neither `>= 90` nor `>= 75`, so the method returns `"C"`.

**Forgetting `self` as a method's first parameter is the single most common mistake in beginner Python OOP code, and the error it produces is genuinely confusing at first glance.** Python still automatically supplies the object as the first argument at the call site — so a method written as `def describe():` (no `self`) ends up receiving one argument too many the moment you call `s1.describe()`, and Python raises a `TypeError` complaining about argument counts, not an obvious message about a missing `self`. If you ever see a `TypeError` mentioning one too many positional arguments on a method call, a missing `self` in the method's definition is the very first thing worth checking.

## Putting it all together: a complete, working class

Every class you'll write, no matter how complex it eventually becomes, follows the same repeatable shape. A `BankAccount` makes the pattern concrete with a second real-world example, alongside `Student`:

```python
class BankAccount:
    bank_name = "First National"          # class attribute

    def __init__(self, owner_name, balance):
        self.owner_name = owner_name      # instance attribute
        self.balance = balance            # instance attribute

    def deposit(self, amount):
        self.balance = self.balance + amount

    def withdraw(self, amount):
        self.balance = self.balance - amount

    def describe(self):
        return f"{self.owner_name}'s account at {self.bank_name}: Rs.{self.balance}"

account = BankAccount("Priya Nair", 500)
account.deposit(150)
account.withdraw(80)
print(account.describe())
```

```
Priya Nair's account at First National: Rs.570
```

Trace this by hand before trusting the output. `balance` starts at `500` inside `__init__`. `account.deposit(150)` binds `account` as `self`, reads `self.balance` (`500`), adds `150`, and writes `650` back into `self.balance`. `account.withdraw(80)` repeats the same read-modify-write pattern, taking `balance` from `650` down to `570`. `describe()` then reads the object's *current* state — `owner_name`, the shared `bank_name`, and the now-updated `balance` — and returns them combined into one string, which `print()` displays exactly as returned.

Notice that `deposit()` and `withdraw()` never *print* anything themselves — they only change `self.balance` and hand nothing back, exactly the "action, not display" role a method like this should play; `describe()` is the one method whose entire job is to report state back out, echoing the `return`-versus-`print()` distinction from Unit 2.3.

| Building a class, step by step | What you do |
|---|---|
| 1. Describe the ADT | List the data (attributes) and actions (methods) in plain words, before writing Python |
| 2. Open the class body | `class ClassName:` in `PascalCase` |
| 3. Add class attributes, if any | Only for values genuinely identical across every instance |
| 4. Write the constructor | `def __init__(self, ...):`, assigning each parameter to `self.attribute = value` |
| 5. Add methods | One per behaviour identified in step 1, each starting with `self` |
| 6. Instantiate | `obj = ClassName(value1, value2)` — repeat for every independent thing you need |
| 7. Interact | `obj.attribute` to read data, `obj.method(args)` to trigger behaviour |

## A dictionary record vs a class instance

It's worth putting the opening problem and this unit's solution side by side, now that you've seen both in full:

| | Dictionary + separate function | Class instance |
|---|---|---|
| Data and logic | Held apart, connected only by your own care | Bundled together in one object |
| Adding a new operation | Write yet another standalone function, hoping it's used on the right dictionary | Add a method; it only ever operates on `self` |
| Guarantee against mixing up data | None — nothing stops passing the wrong dictionary in | Strong — a method always operates on the object it was called on |
| Scaling to thousands of records | Thousands of near-identical dictionaries, functions called correctly by convention | Thousands of independent objects, each carrying its own data and behaviour |

## Classes in the real world

The same shape — state set up in `__init__`, behaviour written as methods, `self` tying the two together — appears anywhere real code needs to represent something with its own identity and its own data:

- **Banking and fintech** — a `BankAccount` class holding `owner_name` and `balance`, with `deposit()` and `withdraw()` methods; every account is a separate object, so one customer's balance can never leak into another's.
- **UPI payments** — a `Transaction` object bundling a payer, payee, amount, and status, with a method such as `mark_success()`, exactly the shape a real payment gateway builds for every transaction it processes.
- **Swiggy-style delivery** — an `Order` class holding items and a total, with methods like `apply_coupon()` and `mark_delivered()`; every order placed is one independent object.
- **IRCTC-style booking** — a `Ticket` class holding a passenger, a fare, and a confirmation status, with a method like `confirm_booking()`.
- **Healthcare** — a `PatientRecord` class holding a name, age, and diagnosis, with a method like `admit()` that updates that one patient's own state without touching anybody else's record.
- **AI/ML** — a `Model` object bundling training data and hyperparameters with methods like `train()` and `predict()`, so several differently configured models can exist side by side without interfering with each other.

The underlying pattern never changes as the software you write grows more complex: identify the real-world thing, decide what it needs to remember, decide what it needs to do, and let a class tie the two together in one unit.

A short list of mistakes worth watching for deliberately while this is still new:

- Forgetting `self` as a method's first parameter, producing a confusing `TypeError` about argument counts.
- Forgetting the parentheses at instantiation (`account = BankAccount` instead of `BankAccount(...)`), leaving a name pointing at the class itself rather than a new object.
- Forgetting the `self.` prefix inside a method body (`balance = balance + amount` instead of `self.balance = ...`), which creates a throwaway local variable instead of updating the object's real attribute.
- Reading an instance attribute before it has ever been assigned through `self`, which raises an `AttributeError`.
- Reaching for a class attribute for data that actually varies per object, causing every instance to silently share a value that should have been independent.

## Try it yourself

Do this in a Colab cell before moving on. Define a `Student` class exactly as built in this unit — `college_name` as a class attribute, `name` and `marks` set in `__init__`, plus `calculate_grade()` and `describe()`. Add one more method, `passed(self)`, that returns `True` if `self.marks` is `40` or higher and `False` otherwise. Create three `Student` objects with different names and marks, loop over them, and for each one print its `describe()` result together with whether `passed()` returns `True` or `False`. Before running, predict for yourself which of your three students should pass.

---

### Key Terminology

- **Abstract data type (ADT)** — a description of a thing by its data (attributes) and behaviour (methods), written before any code, independent of implementation.
- **Class** — a blueprint that defines what attributes and methods every object built from it will have.
- **Object / Instance** — one specific thing created from a class, holding its own real values.
- **Instantiation** — the act of creating an object from a class, by calling the class name like a function: `ClassName(...)`.
- **Attribute** — a variable that belongs to a class or an object — the "data" half of an ADT.
- **Instance attribute** — an attribute that belongs to one specific object alone, usually set with `self.attribute = value` inside `__init__`.
- **Class attribute** — an attribute defined directly in the class body, shared by every instance unless one instance is given its own attribute of the same name.
- **Method** — a function defined inside a class body — the "behaviour" half of an ADT.
- **Constructor (`__init__`)** — the special method Python calls automatically every time a new object is created, used to set up its starting state.
- **`self`** — the first parameter of every instance method, referring to the specific object the method was called on; supplied automatically by Python.
- **`AttributeError`** — raised when code reads an attribute that was never assigned on that object.

### Mastery Checkpoint

Before moving to Unit 4.2, check that you can answer these without looking back:

1. Why does bundling data and behaviour into a class avoid a bug that a dictionary plus a separate function cannot fully rule out?
2. What is the difference between what a class describes and what an object actually holds?
3. Why does `Student("Priya Nair", 91)` never require you to call `__init__` yourself, even though `__init__` is what actually sets up `name` and `marks`?
4. `s1.describe()` internally calls `self.calculate_grade()`. What value is `self` bound to during that entire call, and how did Python decide that?
5. A method is defined as `def describe():`, missing `self`. What error appears when you call `s1.describe()`, and why is that error message not an obvious complaint about `self`?

### Summary

You now know why bundling state and behaviour together — rather than keeping a dictionary and a handful of loose functions side by side — is the problem object-oriented programming exists to solve, and you've learned to describe a real-world thing as an abstract data type before writing a single line of code. You've built classes as blueprints, created independent objects from them through instantiation, given each object its starting data with a constructor, distinguished instance attributes from class attributes, and written methods that use `self` to read and change an object's own state. From here, the next step is extending and protecting these classes further — starting with Unit 4.2, The Four Pillars of OOP.

### Additional Resources

- [Python Tutorial — official docs: "Classes"](https://docs.python.org/3/tutorial/classes.html)
- [Python 3 Documentation — Built-in Types](https://docs.python.org/3/library/stdtypes.html)
- [Python 3 Documentation — Built-in Exceptions (`AttributeError`, `TypeError`)](https://docs.python.org/3/library/exceptions.html)
- [PEP 8 — Style Guide for Python Code (class naming conventions)](https://peps.python.org/pep-0008/#class-names)
- [W3Schools — Python Classes/Objects](https://www.w3schools.com/python/python_classes.asp)
- [W3Schools — Python Inheritance](https://www.w3schools.com/python/python_inheritance.asp)
