---
topic_id: e92e9967-9d4c-4992-a8f3-5681762aaeaa
title: Unit 4.1 - Object-Oriented Foundations
position_in_module: 1
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 4.1 - Object-Oriented Foundations — Topic Corpus

## 2. Prerequisites

- Unit 2.3 - Functions (`44580fc5`-series concept set: `def`, parameter, argument, `return`) — a method, introduced in this unit, is built on the same `def` syntax you already know; the only new part is where it is written and what its first parameter means.
- Unit 3.4 - Dictionaries (`3695f9f8-f7e3-4f7e-933d-95006fced9d1`) — the Introduction below contrasts a class with the dictionary-plus-function approach you already know, so you can see exactly what problem a class solves.
- Unit 1.4 - Statements, Conversion & Output (`3c04d4c0-e9ca-4a4a-9d5e-368effd5d162`) — f-strings are used throughout this unit's examples to build summary text from an object's data.
- Unit 1.2 - Variables, Identifiers & Types (`44580fc5-261e-4a0a-bc9b-22972428f96e`) — `type()` reappears here to confirm what class an object was built from, exactly as it previously reported `int`, `str`, and other types.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** what an abstract data type (ADT) is and why bundling related data and the functions that act on it into one unit models a real-world thing more faithfully than separate variables and functions.
- **Differentiate** a class (a blueprint) from an object/instance (one specific thing built from that blueprint).
- **Implement** a constructor (`__init__`) that sets up instance attributes for every new object of a class.
- **Create** instance methods, using `self` correctly as the first parameter, and call them on an object using dot notation.
- **Identify** the difference between an instance attribute and a class attribute, and state when each is the appropriate choice.
- **Debug** the most common beginner mistakes in Python object-oriented code, such as a missing `self` parameter or a missing pair of parentheses at instantiation.

## 4. Introduction

Imagine you are tracking one student's marks for a college. You store the name and the marks in a dictionary, and you write a separate function that calculates the average:

```python
student = {"name": "Rahul", "marks": [85, 90, 78]}

def calculate_average(marks):
    return sum(marks) / len(marks)

print(calculate_average(student["marks"]))
```

This works fine for one student. But a real college has thousands of students, each needing several pieces of data (name, roll number, marks, department) and several operations (calculate the average, display the details, check pass/fail). If you keep using separate dictionaries and separate functions, nothing in the language stops you from accidentally passing one student's marks into a function meant to process a different student's data. The data and the operations that belong together are held apart, connected only by your own memory and careful naming.

Object-oriented programming (OOP) is a way of writing code that keeps related data and the functions that work on that data bundled together in a single unit. Instead of a dictionary plus a floating function, you build one `Student` "thing" that carries its own name and marks, and that also knows how to calculate its own grade or describe itself. Once thousands of students exist, each one is simply another copy of the same "thing," never confused with any other, because each carries its own data with it rather than sharing a pool of loose variables.

This unit introduces the four ideas every other topic in this module builds on: the abstract data type as a way of thinking about a "thing" before writing any code, the class as the blueprint you write in Python, the object as one concrete thing produced from that blueprint, and the constructor plus methods that give an object its data and its behavior[1]. The module description for this section of the course promises that you will go on to extend classes with inheritance and simplify them with dataclasses — both of those are named here only as a pointer to what comes next (Unit 4.2 and Unit 4.3 respectively) and are not explained in this topic.

## 5. Core Concepts

**Abstract data type (ADT).** An abstract data type describes *what* a thing is and *what* it can do, without yet describing *how* any of it is implemented in code. For a student, the description might be: data (attributes) — name, roll number, marks; behavior — calculate a grade, display details. At the ADT stage you are only listing what the thing contains and what actions it supports; you have not written any Python yet[1]. Thinking in terms of an ADT first — before touching the keyboard — is what tells you which attributes and which methods a class needs.

**Class.** A class is how Python lets you turn an ADT into real code: it is a blueprint that defines what attributes and what methods every object built from it will have. A class by itself never holds any real data — it only describes the shape that data will take once an object is built from it[1].

```python
class Student:
    pass

first_student = Student()
print(type(first_student))
```

Output:

```
<class '__main__.Student'>
```

`class Student:` opens the blueprint (the reserved keyword `class`, followed by the name and a colon, exactly like a function's `def name():` line opens a function body). `pass` means "this body is deliberately empty for now" — the same placeholder use of `pass` you may have seen inside an empty function or loop body. `Student()` is what actually produces a usable thing; `type()` — the same built-in you have used since Unit 1.2 to check whether a value was an `int` or a `str` — confirms that `first_student` was built from the `Student` class.

**Object / Instance.** An object, also called an instance, is one specific thing created from a class, holding its own actual values. Where the class is the idea of "a student in general," an object is one real student — with a real name and a real set of marks. You can build as many independent objects as you want from the same class, and none of them share data with each other.

**Instantiation.** Instantiation is the act of creating an object from a class. You do it by calling the class name as if it were a function: `ClassName(...)`. This always requires parentheses, even if no arguments are needed — the parentheses are what actually trigger the creation of a new object.

**Attribute.** An attribute is a variable that belongs to a class or to an object — it is the "state" half of an ADT: the data a thing carries around with it.

**Instance attribute.** An instance attribute is an attribute that belongs to one specific object alone. It is usually set with `self.attribute_name = value` inside the constructor (defined next), and every object gets its own independent copy — changing one object's instance attribute never affects any other object's.

**Class attribute.** A class attribute is defined directly in the class body, not inside any method. It is shared by every instance of the class until (and unless) one particular instance is given its own attribute of the same name, which then applies only to that one instance. A class attribute is the right choice only for a value that is genuinely identical across every object — a college name shared by every student, for example — never for anything that is expected to differ per object.

**Method.** A method is a function defined inside a class body — the "behavior" half of an ADT: an action the thing can perform, usually using its own attributes. Syntactically a method is written with the exact same `def` keyword as any other function; what makes it a method is simply that it lives inside a `class` body and takes `self` as its first parameter.

**`self`.** `self` is the first parameter of every instance method, including the constructor. It refers to the specific object the method was called on. You never supply `self` yourself when calling a method — Python fills it in automatically. Concretely, `account.deposit(100)` is handled by Python as though it had been written `BankAccount.deposit(account, 100)`: the object to the left of the dot becomes the value bound to `self`.

**Constructor (`__init__`).** The constructor is a special method named exactly `__init__` that Python calls automatically every single time a new object is created from a class. Its job is to set up the object's starting state, typically by assigning each incoming parameter to an instance attribute of the same name. You never call `__init__` directly — writing `ClassName(...)` is what triggers Python to call it for you, immediately after the new (still-empty) object is created and just before that object is handed back to you.

Putting all of this together, the general shape looks like this:

```python
class ClassName:
    class_attribute = value          # shared by every instance

    def __init__(self, param1, param2):
        self.param1 = param1         # instance attribute
        self.param2 = param2         # instance attribute

    def method_name(self, extra_arg):
        # method body: can read or change self.param1, self.param2
        ...

obj = ClassName(value1, value2)      # instantiation
```

A worked comparison makes the class-versus-object distinction concrete:

| Aspect | Class | Object (Instance) |
|---|---|---|
| What it is | A blueprint / template | A specific thing built from the blueprint |
| How many exist | Usually one definition in your code | As many as you choose to create |
| Holds actual data? | No — describes what data instances will hold | Yes — each object holds its own real values |
| Created with | `class ClassName:` | `ClassName(...)` (instantiation) |
| Example | `Student` (the idea of "a student") | `Student("Priya Nair", 91)` (one real student) |

A small, complete example ties every term together:

```python
class Student:
    college_name = "Crescent College"      # class attribute

    def __init__(self, name, marks):
        self.name = name                   # instance attribute
        self.marks = marks                 # instance attribute

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

Output:

```
[Crescent College] Priya Nair scored 91 marks - Grade A
[Crescent College] Arjun Rao scored 68 marks - Grade C
```

`Student("Priya Nair", 91)` instantiates the class: Python builds a new object, then calls `__init__` on it automatically with `name="Priya Nair"` and `marks=91`, so `s1.name` and `s1.marks` are set. `s2` is built the same way with different arguments, giving it entirely independent values for `name` and `marks`. `s1.describe()` looks up the `describe` method on the `Student` class, binds `s1` as `self` for the whole call — including the nested `self.calculate_grade()` call inside it — and returns a string built from `s1`'s own attributes plus the shared `college_name` class attribute. `s2.describe()` repeats the exact same steps with `s2` bound as `self`, producing a completely independent result[1].

Two details are easy to miss the first time you see this pattern. First, `college_name` never appears inside `__init__` and is never assigned through `self` for either object — it lives directly in the class body, so both `s1.college_name` and `s2.college_name` read the identical `"Crescent College"` string without either object needing its own copy. Second, if a third object were created and one line, say `s3.college_name = "Northside College"`, assigned directly to that one instance, only `s3` would see the new value from then on — `s1` and `s2` would still read the original class attribute, because assigning through `self` on one instance creates an instance attribute that shadows the class attribute for that instance alone, without touching the class attribute itself or any other instance.

## 6. Implementation

Building any class follows the same repeatable sequence of steps, regardless of what real-world thing it models:

1. **Describe the ADT first, in plain words.** List the data the thing needs to remember (its attributes) and the actions it needs to support (its methods) — before writing any Python.
2. **Open the class body.** Write `class ClassName:` using `PascalCase` for the name (capitalize each word, no underscores — `Student`, `BankAccount`).
3. **Add class attributes, if any, directly in the class body.** Only for values that are genuinely identical across every instance.
4. **Write the constructor.** Define `def __init__(self, ...):` with `self` first, followed by one parameter for every piece of data an object needs at creation time. Inside the body, assign each parameter to an instance attribute: `self.param = param`.
5. **Add methods for each behavior identified in step 1.** Each method definition starts with `self` as its first parameter, followed by any extra parameters the action needs. Inside the body, read or update the object's own state through `self.attribute_name`.
6. **Instantiate the class** wherever you need one of these things: `obj = ClassName(value1, value2)`. Repeat as many times as needed — each call produces a fully independent object.
7. **Interact with the object** through dot notation: `obj.attribute_name` to read data, `obj.method_name(args)` to trigger behavior.

Tracing this sequence for a `BankAccount` makes it concrete. The ADT (step 1) is: data — owner name, balance; behavior — deposit, withdraw, describe. Steps 2–5 produce:

```python
class BankAccount:
    bank_name = "First National"          # step 3: class attribute

    def __init__(self, owner_name, balance):   # step 4: constructor
        self.owner_name = owner_name
        self.balance = balance

    def deposit(self, amount):            # step 5: method
        self.balance = self.balance + amount

    def withdraw(self, amount):           # step 5: method
        self.balance = self.balance - amount

    def describe(self):                   # step 5: method
        return f"{self.owner_name}'s account at {self.bank_name}: Rs.{self.balance}"
```

Step 6 instantiates one account, and step 7 interacts with it:

```python
account = BankAccount("Priya Nair", 500)   # step 6: instantiation
account.deposit(150)                       # step 7: balance becomes 650
account.withdraw(80)                       # step 7: balance becomes 570
print(account.describe())
```

Output: `Priya Nair's account at First National: Rs.570`. `balance` starts at `500` inside `__init__`. `deposit(150)` reads the current `self.balance`, adds `150`, and writes `650` back into `self.balance`. `withdraw(80)` then reads `650`, subtracts `80`, and writes `570` back. `describe()` reads the object's state at the moment it is called — the instance attributes `owner_name` and the now-updated `balance`, plus the shared class attribute `bank_name` — and returns them combined into one formatted string[1].

## 7. Real-World Patterns

The same shape — state set up in `__init__`, behavior written as methods, `self` tying the two together — appears anywhere real code needs to represent something with its own identity and its own data[1]:

- **Banking:** a `BankAccount` class holding `owner_name` and `balance` as instance attributes, with `deposit()` and `withdraw()` methods — every account is a separate object, so one customer's balance can never leak into another's.
- **E-commerce:** an `Order` class holding a customer and a list of items, with methods like `apply_coupon()` and `mark_shipped()` — every order placed on a platform is one independent object.
- **Education:** a `Student` class, exactly as built in this unit, holding a name and marks with a method to compute a grade — the same shape a college's result-processing system relies on for every enrolled student.
- **Payments:** a `Transaction` object bundling a payer, an amount, and a status, with a method such as `mark_success()` — the kind of object a real payment gateway creates for every transaction it processes.
- **Healthcare:** a `PatientRecord` class holding a patient's name, age, and current diagnosis, with a method like `admit()` that updates that one patient's own state without touching any other patient's record.
- **Cloud applications:** a `CloudResource` object (a virtual machine, a storage bucket) holding its own configuration and status, with methods like `start()` and `stop()`, mirroring how cloud provider software development kits (SDKs) represent the resources they manage.

The underlying pattern never changes as the software gets more complex: identify the real-world thing, decide what it needs to remember, decide what it needs to do, and let a class tie the two together in one unit[1].

## 8. Best Practices

- Name classes in `PascalCase` (`Student`, `BankAccount`); name the objects you create from them in `snake_case`, exactly as you already name any other variable.
- Keep `__init__` focused purely on setup — assigning parameters to attributes and establishing sensible starting values. Avoid unrelated calculations or printing inside it.
- Name constructor parameters the same as the attribute they populate (`self.name = name`) so the mapping is obvious to any reader.
- Give methods verb-like names describing the action performed (`deposit`, `calculate_grade`) — the same convention already used for ordinary functions.
- Reserve a class attribute for a value that is genuinely identical across every instance; use an instance attribute, set inside `__init__`, for anything that can differ per object — in practice, almost everything you model.
- Watch for the most common beginner mistakes: forgetting `self` as a method's first parameter (Python still passes the object automatically, so the call ends up with one argument too many, producing a `TypeError` about argument counts); forgetting the parentheses when instantiating, which leaves a name pointing at the class itself rather than a new object; and forgetting the `self.` prefix inside a method body, which creates a throwaway local variable instead of updating the object's real attribute.

## 9. Hands-On Exercise

Using the `Student` class shown in Core Concepts, create three `Student` objects with different names and marks. Add one more method, `passed(self)`, that returns `True` if `self.marks` is 40 or higher and `False` otherwise (reusing the comparison and boolean ideas from Units 1.3 and 2.1). Loop over your three objects, printing each one's `describe()` result together with whether `passed()` returns `True` or `False`.

## 10. Key Takeaways

- An abstract data type describes a thing by its state and behavior together; a Python class is how that idea gets implemented in code.
- A class is a blueprint; an object (instance) is one specific thing built from that blueprint, holding its own independent state.
- `__init__` is the constructor: Python calls it automatically every time a class is instantiated, and it is where instance attributes are normally assigned through `self.attribute = value`.
- Instance attributes vary per object; class attributes, defined directly in the class body, are shared by every instance unless one instance is given its own attribute of the same name.
- `self` is how a method refers to the specific object it was called on, supplied automatically by Python — never passed manually — and instantiation always requires parentheses because that is what actually triggers `__init__`.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
