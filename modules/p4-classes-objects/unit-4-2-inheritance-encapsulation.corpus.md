---
topic_id: b626a741-e204-40ec-9d6a-85ca553262f9
title: Unit 4.2 - Inheritance & Encapsulation
position_in_module: 2
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 4.2 - Inheritance & Encapsulation — Topic Corpus

## 2. Prerequisites

This topic builds directly on **e92e9967-9d4c-4992-a8f3-5681762aaeaa — Unit 4.1: Object-Oriented Foundations**, which introduced the abstract data type (ADT), class, object/instance, the `__init__` constructor, instance attributes, class attributes, methods, `self`, and instantiation. Everything in this unit assumes you can already write a plain class with an `__init__` method, add instance attributes inside it, define ordinary methods, and create objects from that class through instantiation.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** what inheritance is and why a subclass automatically gains the attributes and methods of an existing class without any code being copied.
- **Implement** a subclass using `super()` to reuse a superclass's constructor and methods instead of retyping them.
- **Differentiate** single inheritance, multi-level inheritance, and multiple inheritance, and describe in your own words what Method Resolution Order (MRO) is for.
- **Analyze** a small class hierarchy — including a diamond-shaped one — by reading `ClassName.__mro__` to predict which method runs.
- **Apply** Python's underscore naming conventions (`_name` and `__name`) to signal which attributes of an object are internal.
- **Explain** why forgetting `super().__init__()` in a subclass causes an `AttributeError`, and what that reveals about how inheritance actually works.

## 4. Introduction

Imagine you already built a `Student` class — it stores a student's `name`, `roll_number`, and `marks`, and has a method `has_passed()` that checks if `marks` is at least 40. Now your college asks you to also track graduate students, who have everything a regular student has, plus one extra piece of information: a `thesis_topic`. Do you rewrite the whole class from scratch, just to add one field? That would mean two nearly-identical classes, and if you ever fixed a bug in `Student`, you would have to remember to fix it a second time in the copy. Python gives you a better option: build a new class that **reuses** everything `Student` already does, and only adds what's different. That mechanism is called **inheritance**, and it is the subject of this unit.[1]

Inheritance is not a special-purpose trick — it is how real systems avoid repeating themselves. A banking app does not write one giant class that handles savings accounts, current accounts, and loan accounts all at once; it writes one shared `BankAccount` class and lets `SavingsAccount` and `CurrentAccount` build on top of it, each adding only what makes it different.[1] This unit also answers a question that naturally follows once you start reusing classes: what happens when a class needs to combine features from *two* unrelated classes at once, not just one? And separately, once your class has internal bookkeeping data (like a bank balance), how do you signal to other programmers "don't touch this directly from outside"? That second question is answered by **encapsulation** — a naming convention, not a wall, but one every professional Python codebase relies on.

Consider the bank balance itself: a `BankAccount` object holds a number representing how much money someone has. If any code anywhere could freely rewrite that number directly, there would be no reliable place to enforce rules like "never let the balance go negative without an overdraft limit." Python cannot force other programmers to go through your methods, but it gives you a naming convention to signal your intent clearly — and every professional Python codebase leans on that convention, so recognizing it is essential even before you write your own.

By the end of this unit you will be able to look at a chain of classes several levels deep, or two classes combined at once, and know exactly which class's code Python will actually run — and you'll know how to mark which parts of your own objects are meant to stay internal.

## 5. Core Concepts

### 5.1 Dot Notation: Reading and Writing Through an Object

Before going further, one small piece of syntax needs to be named explicitly, because this unit uses it constantly: **dot notation**. When you write `student.name` or `account.deposit(500)`, the dot (`.`) is how Python reaches "inside" an object to get an attribute or call a method that belongs to it. `student.name` means "look at the object `student` is pointing to, and get its `name` attribute." `account.deposit(500)` means "look at the object `account` points to, and call its `deposit` method, passing `500` as an argument." You already used dot notation informally with `self.name` inside a class body in Unit 4.1 — the same dot works identically from outside the class, on any object you have a reference to. Every example in this unit relies on dot notation to read attributes, call methods, and — as you'll see in §5.10 — to (attempt to) reach into another object's internal data.

### 5.2 Inheritance: Superclass and Subclass

**Inheritance** is a mechanism where a new class automatically acquires the attributes and methods of an existing class, without any of that code being copied or retyped.[1] The existing class being built on top of is called the **superclass** (also called the **parent class** or **base class**). The new class is called the **subclass** (also called the **child class** or **derived class**). The subclass gets everything the superclass has, and can add its own extra attributes and methods on top.

Here is the entire mechanism, in one line of syntax:

```python
class Student:
    def __init__(self, name, roll_number, marks):
        self.name = name
        self.roll_number = roll_number
        self.marks = marks

    def has_passed(self):
        return self.marks >= 40


class GraduateStudent(Student):   # GraduateStudent is a subclass of Student
    pass
```

Even though `GraduateStudent` has an empty body (`pass` means "do nothing else"), it already has a working `__init__`, `has_passed()`, and every attribute `Student` sets up — because `class GraduateStudent(Student):` tells Python "build this new class on top of `Student`." The name in parentheses is the superclass.

Compare this to the alternative — writing `GraduateStudent` completely from scratch, without inheritance:

```python
class GraduateStudentWithoutInheritance:   # everything retyped by hand
    def __init__(self, name, roll_number, marks):
        self.name = name
        self.roll_number = roll_number
        self.marks = marks

    def has_passed(self):
        return self.marks >= 40
```

This works, but every single line is a duplicate of code that already exists in `Student`. If a bug were later found in how `marks` gets validated, it would need to be fixed in two places, and it is easy to forget the second one. The one-line version, `class GraduateStudent(Student): pass`, avoids that risk entirely by construction — there is nothing to keep in sync, because there is only one copy of the logic.

Why does this matter? Without inheritance, every related class has to be written from scratch, and any shared bug fix or improvement has to be repeated in every copy by hand.[1] Inheritance solves three concrete problems at once:

- **It reduces duplication.** A method like `has_passed()` is written once, in `Student`, and reused by every subclass that doesn't need to change it.
- **It represents a genuine real-world relationship.** A graduate student **is a** student — with something extra. Wherever this "is-a" relationship genuinely holds, inheritance is a natural fit.
- **It makes maintenance easier.** If you later fix a bug inside `Student.has_passed()`, every subclass that inherits it automatically gets the fix — nothing needs to be touched in the subclasses themselves.

### 5.3 `super()`: Reusing the Superclass's Code Instead of Retyping It

A subclass almost always needs to add its *own* new attribute — in our example, `thesis_topic`. That means `GraduateStudent` needs its own `__init__`. But if you write a new `__init__` in the subclass, does the superclass's `__init__` still run? No — **once a subclass defines its own `__init__`, the superclass's `__init__` does not run automatically.** You have to call it yourself, explicitly, using a built-in function called **`super()`**.

`super()` is a built-in function that gives a subclass a way to reach the superclass's version of a method — most often its `__init__` — so the subclass can reuse that code instead of duplicating it.[1]

```python
class GraduateStudent(Student):
    def __init__(self, name, roll_number, marks, thesis_topic):
        super().__init__(name, roll_number, marks)   # runs Student's __init__
        self.thesis_topic = thesis_topic              # then add the new field

    def has_passed(self):
        return self.marks >= 50   # graduate-level pass mark is stricter
```

`super().__init__(name, roll_number, marks)` calls `Student.__init__`, which sets `self.name`, `self.roll_number`, and `self.marks` on the new object — exactly as it would for a plain `Student`. Only *after* that line does `GraduateStudent` add its own attribute, `self.thesis_topic`. This is the standard pattern: **call `super().__init__()` first, before adding anything new**, so the object's inherited part is fully built before the subclass adds to it.

Notice that `has_passed()` is rewritten completely in `GraduateStudent`, with no call to `super()` at all. That is intentional and leads to the next concept.

### 5.4 Overriding vs. Extending a Method

**Overriding** means a subclass defines its own version of a method that the superclass already has; when you call that method on a subclass object, the subclass's version runs, not the superclass's.[1] `GraduateStudent.has_passed()` overrides `Student.has_passed()` — the rule genuinely changes (40 becomes 50), so the subclass's version completely replaces the superclass's, with no need to reuse any of the old logic.

**Extending** is a related but different idea: the subclass calls `super()` to *reuse* the superclass's version of a method, and then adds more on top, rather than throwing it away entirely. For example, if `Student` had a `describe()` method returning a short summary string, `GraduateStudent` could extend it like this:

```python
def describe(self):
    base = super().describe()          # reuse Student's version
    return f"{base} (thesis: {self.thesis_topic})"
```

The rule of thumb: if the subclass's behavior is a complete replacement, override without calling `super()` (as `has_passed()` does above). If the subclass's behavior is "do what the superclass does, plus a bit more," call `super()` inside the override and add to its result (as `describe()` does here). Both are called "overriding" a method name, but only the second one is also "extending" it.

It is easy to override a method by accident, simply by reusing a name the superclass already defines, without realizing the superclass's version is now unreachable through that name on the subclass. If `GraduateStudent` defined a method also called `has_passed` while intending to add a brand-new, differently named method, the superclass's `has_passed()` would silently stop being used for any `GraduateStudent` object — Python does not warn you when this happens, so it's worth deliberately deciding, for every method name a subclass reuses, whether you intend to override, extend, or genuinely picked a name that collides by accident.

### 5.5 Multi-Level Inheritance

**Multi-level inheritance** is a chain of more than two classes, where each class extends the one directly before it — for example `A` → `B` → `C`, where `C` extends `B`, and `B` extends `A`.[1] Continuing the student example, suppose a `PhDStudent` needs everything `GraduateStudent` has, plus a `supervisor_name`:

```python
class PhDStudent(GraduateStudent):
    def __init__(self, name, roll_number, marks, thesis_topic, supervisor_name):
        super().__init__(name, roll_number, marks, thesis_topic)
        self.supervisor_name = supervisor_name
```

Here, `super().__init__(...)` calls `GraduateStudent.__init__`, which itself calls `super().__init__(...)` to reach `Student.__init__`. Each level only needs to worry about calling `super()` once, targeting the class directly above it in the chain — it does not need to know or care how many levels exist further up. The chain `Student` → `GraduateStudent` → `PhDStudent` means a `PhDStudent` object ends up with `name`, `roll_number`, `marks` (from `Student`), `thesis_topic` (from `GraduateStudent`), and `supervisor_name` (its own) — all fully set up, with each `__init__` handing off responsibility to the level above it.

The main risk with multi-level inheritance is not incorrectness but readability: a chain that grows too long (four or five levels deep) becomes hard to trace when you're trying to figure out where a particular attribute or method actually comes from.[1]

Creating and checking a `PhDStudent` object shows the chain working end to end:

```python
phd = PhDStudent("Ananya", 501, 92.0, "Reinforcement Learning", "Dr. Rao")
print(phd.name)                      # Ananya — came from Student, two levels up
print(phd.thesis_topic)              # Reinforcement Learning — came from GraduateStudent
print(phd.supervisor_name)           # Dr. Rao — defined on PhDStudent itself
print(isinstance(phd, Student))      # True — Student is in PhDStudent's MRO
```

Every attribute is present and correct, even though `name` was set two classes away from `PhDStudent`'s own `__init__`, because each level's `super().__init__()` call faithfully passed the responsibility upward before adding its own piece.

### 5.6 Multiple Inheritance and Method Resolution Order (MRO)

**Multiple inheritance** is different from multi-level inheritance: it is when a single subclass extends *more than one* direct superclass at the same time, written `class C(A, B):`.[1] This lets one class combine behavior from two independent classes in one step. For example, suppose you have an unrelated small class that only adds one reusable ability:

```python
class NotifiableMixin:
    def notify(self):
        return "Email sent to registrar"


class GraduateStudent(Student, NotifiableMixin):
    def __init__(self, name, roll_number, marks, thesis_topic):
        super().__init__(name, roll_number, marks)
        self.thesis_topic = thesis_topic
```

`GraduateStudent` now has everything from `Student` *and* the `notify()` method from `NotifiableMixin`, combined through multiple inheritance.

This raises a question: when there are two (or more) superclasses, and you call `super()`, or you call a method that isn't defined on the subclass itself, which class does Python actually search first? The answer is the **Method Resolution Order**, or **MRO** — the fixed order Python computes, once, at the moment a class is defined, listing exactly which classes it will search through (and in what order) when looking up a method or attribute.[1] You can inspect it directly:

```python
print(GraduateStudent.__mro__)
# (<class 'GraduateStudent'>, <class 'Student'>, <class 'NotifiableMixin'>, <class 'object'>)
```

`ClassName.__mro__` is a special attribute Python builds automatically for every class; reading it tells you, in order, every class Python will check when it looks up a name on that class. `__mro__` shows `GraduateStudent` first (its own definitions win first), then `Student`, then `NotifiableMixin`, then `object` (the built-in base class every Python class ultimately extends). The rule that decides this order, in general: Python fully searches the first listed superclass's entire chain before moving on to the next one written in the parentheses.[1] Since `Student` was written first in `class GraduateStudent(Student, NotifiableMixin):`, its whole chain is searched before `NotifiableMixin` is even considered.

The single most important fact about `super()` becomes visible only here: **`super()` does not mean "my literal parent class." It means "the next class in the computed MRO."**[1] With single inheritance those two things happen to be the same class, which is why it's easy to misunderstand `super()` as "go to my parent" — multiple inheritance is where the distinction actually matters.

### 5.7 The Diamond Problem

The **diamond problem** is a specific, well-known situation inside multiple inheritance: two superclasses share a common ancestor, and a subclass inherits from both of them at once — so the class hierarchy, drawn out, looks like a diamond shape.[1] For example:

```
        Student
        /      \
GraduateStudent   NotifiableStudent
        \      /
     ResearchFellow
```

Here `ResearchFellow(GraduateStudent, NotifiableStudent)` inherits from two classes that both, in turn, inherit from `Student`. The question this raises is: when `ResearchFellow` calls `super()`, or looks up a method it doesn't define itself, does Python check `Student` once, or twice, or in what order relative to `NotifiableStudent`? Python answers this the same way as any other multiple inheritance case — by computing one single MRO for `ResearchFellow` that lists every ancestor exactly once, in a consistent, predictable order (fully resolving `GraduateStudent`'s side before moving to `NotifiableStudent`'s side):

```python
class Student:
    def __init__(self, name):
        self.name = name


class GraduateStudent(Student):
    def __init__(self, name, thesis_topic):
        super().__init__(name)
        self.thesis_topic = thesis_topic


class NotifiableStudent(Student):
    def notify(self):
        return "Email sent to registrar"


class ResearchFellow(GraduateStudent, NotifiableStudent):
    pass


rf = ResearchFellow("Priya", "Applied Machine Learning")
print(rf.notify())
print(ResearchFellow.__mro__)
```

```
Email sent to registrar
(<class 'ResearchFellow'>, <class 'GraduateStudent'>, <class 'NotifiableStudent'>, <class 'Student'>, <class 'object'>)
```

Notice that `Student` appears in the MRO only **once**, at the end, right before `object` — even though both `GraduateStudent` and `NotifiableStudent` extend it. Python's MRO computation guarantees each ancestor is listed exactly one time, in an order where every class appears before its own superclasses, and a class is never listed before any of the classes that were written ahead of it in the subclass's parentheses. This is precisely what makes the "diamond" shape safe to use: without this guarantee, `Student.__init__` could theoretically run twice or in an unpredictable order; with it, `ResearchFellow` still has a single, well-defined instance of `Student` behavior. The diamond problem is not something you need to solve yourself — Python's MRO algorithm resolves it automatically and consistently — but you do need to be able to read `ClassName.__mro__` to know exactly what order it landed on, rather than guessing.[1]

### 5.8 Mixins

A **mixin** is a small superclass written only to add one specific, focused piece of reusable behavior through multiple inheritance — it is not meant to be used on its own as a genuine "is-a" relationship.[1] `NotifiableMixin` in §5.6 is a mixin: nobody would create a bare `NotifiableMixin()` object on its own; it exists purely to be combined with a "real" class like `Student` or `GraduateStudent` to add one capability (`notify()`) without duplicating it across every class that needs notifications. Naming a class `...Mixin` is a common convention that signals this intent to other programmers reading your code. Keeping each mixin narrowly focused on one responsibility is what keeps a multiple-inheritance MRO easy to reason about.[1] A useful test when deciding whether a class is a genuine "is-a" superclass or a mixin: would it ever make sense to create a bare instance of it on its own? `Student()` makes sense on its own; `NotifiableMixin()` does not — it only has meaning attached to something else.

### 5.9 `isinstance()` and `issubclass()`

**`isinstance(obj, Cls)`** is a built-in function that checks whether an object belongs to a given class *or any of its superclasses* — it returns `True` if `Cls` appears anywhere in the object's class's MRO, not only when `Cls` is the object's exact, immediate class.[1]

```python
gs = GraduateStudent("Arjun", 301, 88.0, "Computer Vision for Traffic Systems")
print(isinstance(gs, Student))          # True — Student is in GraduateStudent's MRO
print(isinstance(gs, GraduateStudent))  # True — it is exactly this class too
```

This is genuinely useful: even though `gs` was built from `GraduateStudent`, `isinstance(gs, Student)` still returns `True`, because inheritance means a `GraduateStudent` really is a `Student`, plus more. **`issubclass(Cls1, Cls2)`** checks the same kind of relationship, but between two classes directly, rather than between an object and a class — `issubclass(GraduateStudent, Student)` returns `True` without needing any object to exist at all.

It's worth noticing why `isinstance()` is usually the better tool compared to checking `type(obj) == SomeClass` directly: an exact `type()` comparison only matches one specific class, while `isinstance()` respects the entire MRO. `type(gs) == Student` would return `False` (because `gs`'s exact type is `GraduateStudent`, not `Student`), even though `gs` genuinely behaves like a `Student` in every way that matters. `isinstance(gs, Student)`, by contrast, correctly recognizes that relationship — which is exactly why library code and built-in functions almost always use `isinstance()` rather than an exact type check.

### 5.10 Encapsulation: Naming Conventions for Internal State

**Encapsulation** is the practice of keeping an object's data and the methods that work on that data together inside one class, while also signaling which parts of that data are meant to stay internal — for use only by the class's own methods, not by outside code.[1] Python does not have a hard "private" keyword the way some other languages do; instead, it uses **naming conventions** — patterns in how you name an attribute — to communicate intent.

There are three levels:

- **Public attribute** — a normal name, like `self.balance`. No naming signal is given; any code, anywhere, is free to read or change it directly.
- **Protected attribute (`_name`)** — a single leading underscore, like `self._balance`. This is a convention meaning "this is internal bookkeeping — don't rely on this from outside the class," but Python does **not** actually stop outside code from reading or writing it.[1]
- **Private attribute (`__name`)** — a double leading underscore, like `self.__pin`. This triggers something called **name mangling**: Python automatically rewrites `self.__pin`, wherever it's written inside the class body, to `self._ClassName__pin`, using the exact name of the class the code is written in.[1]

```python
class BankAccount:
    def __init__(self, balance):
        self._balance = balance     # protected: a convention only
        self.__pin = "1234"         # private: gets name-mangled

    def show_balance(self):
        return f"Balance: {self._balance}"


acc = BankAccount(5000)
print(acc.show_balance())        # works as intended
print(acc._balance)               # 5000 — works; the convention is not enforced
print(acc._BankAccount__pin)      # 1234 — the mangled name still works
```

This example proves the key nuance about Python's encapsulation: it is **convention, not enforcement**. `acc._balance` is read successfully from outside the class — the single underscore never stopped anyone; it only signals intent. `acc.__pin` (the un-mangled name) would fail, but `acc._BankAccount__pin` (the actual mangled name Python rewrote it to) still works perfectly. Name mangling's real purpose is to avoid accidental attribute name collisions across a class hierarchy — for example, if a subclass happens to also use `self.__pin` for something unrelated, mangling keeps the two separate — not to provide genuine security.[1]

The three levels, side by side:

| Naming style | Example | Enforced by Python? |
|---|---|---|
| Public | `self.balance` | N/A — no restriction is signaled at all |
| Protected (`_name`) | `self._balance` | No — a social convention between developers only |
| Private (`__name`) | `self.__pin` | Partially — the original name stops working, but the mangled name (`self._ClassName__pin`) still works fine |

Why bother with encapsulation at all, then, if it isn't enforced? Because it keeps related data and behavior together in one place, protects important data by encouraging access through methods (like `show_balance()`) instead of direct manipulation, and makes code easier to read and maintain — anyone scanning a class can immediately tell, from the underscore, which attributes are meant to stay internal.[1] Two related dunder (double-underscore) items — Python's special methods such as `__repr__` and `__eq__`, and the `dataclasses` module that reduces this kind of boilerplate — are named here only to be pointed at; they are taught in full in Unit 4.3, not here.

## 6. Implementation

Building a correct subclass reliably follows the same sequence of steps every time. Use this checklist whenever you extend an existing class:

1. **Confirm the relationship is genuinely "is-a."** Ask: is the new thing really a specialized version of the existing class (a `GraduateStudent` *is a* `Student`)? If the relationship is closer to "has-a" instead, inheritance is the wrong tool — that's a design topic beyond this unit's scope, but it's worth pausing on before writing `class Child(Parent):`.[1]
2. **Declare the subclass**, naming the superclass in parentheses: `class GraduateStudent(Student):`. This single line is what grants the new class every attribute-setting behavior and every method the superclass already has.
3. **Write the subclass's own `__init__`**, and make its very first line `super().__init__(...)`, passing along every argument the superclass needs. This guarantees the superclass's part of the object is fully built before anything new is added — skipping this step is the single most common inheritance mistake beginners make (see §8).
4. **Add the subclass's new attributes** after the `super().__init__()` call, using ordinary `self.attribute = value` assignments. Anything assigned before the `super().__init__()` call risks being overwritten or built on an incomplete object, so order matters.
5. **Override only the methods that genuinely need to differ.** If a method's behavior is a complete replacement, write a new version with no `super()` call. If it should reuse the superclass's version and add to it, call `super().method_name()` inside the override and build on its result.
6. **If combining more than one superclass**, list them in `class C(A, B):`, then check `C.__mro__` to see the exact search order Python computed, rather than guessing which class's method will run. Do this especially when two of the superclasses share a common ancestor (the diamond shape from §5.7).
7. **Verify with `isinstance()`** that objects of the new subclass are still recognized as instances of every class further up the hierarchy — this is a quick, reliable sanity check that the chain is wired correctly, and it is exactly what production test suites check after any change to a class hierarchy.

## 7. Real-World Patterns

Inheritance shows up anywhere a system has several variations of one underlying concept. A banking application typically has one shared `BankAccount` class, extended separately by `SavingsAccount` and `CurrentAccount`, each adding only the rules that make it different (interest calculation, overdraft limits) while reusing shared logic like `deposit()`.[1] A payment gateway might define one base `PaymentMethod` class, extended by `UPIPayment`, `CardPayment`, and `NetBankingPayment`, each overriding a `process()` method with its own validation while sharing common logging behavior.[1] An e-commerce catalog might extend a base `Product` class into `ElectronicsProduct` and `GroceryProduct`, each adding fields like a warranty period or an expiry date, while both reuse shared pricing logic.[1]

A food delivery platform typically has a base `DeliveryPartner` class extended by `BikePartner` and `CarPartner`; combining a partner class with an independent `RatingMixin` through multiple inheritance is a realistic, everyday use of the MRO concept from §5.6.[1] A healthcare records system might extend a base `Patient` class into `InpatientRecord` and `OutpatientRecord`, each adding fields specific to that kind of visit while sharing common demographic fields through inheritance.[1] A railway-booking system might extend a base `Passenger` class into `SeniorCitizenPassenger` or `TatkalBooking`, each overriding its own fare-calculation logic while reusing shared booking and cancellation methods from the base class.[1]

One pattern worth knowing because you will meet it immediately in ordinary Python code, not just in course examples: Python's own built-in errors are organized through inheritance. `ValueError` and `TypeError` both extend a common built-in class called `Exception`. Production code routinely extends further — for instance, defining a custom `InvalidPinError` that extends a more general `ValidationError`. Because `isinstance()` respects the whole hierarchy, code that catches `ValidationError` automatically catches `InvalidPinError` too, without needing to check for each specific error type by name.[1]

## 8. Best Practices

- **Call `super().__init__()` first**, before setting any new attributes in a subclass's `__init__` — this guarantees the superclass's part of the object is fully built before the subclass adds anything.[1]
- **Favor composition over deep inheritance chains.** If a relationship is not genuinely "is-a," consider giving one class an instance of another instead of forcing an inheritance relationship that doesn't really fit.[1]
- **Keep hierarchies shallow.** Two or three levels are usually enough for a chain to stay easy to trace; five levels deep becomes hard to debug.[1]
- **Default to a single leading underscore (`_balance`)** for internal attributes; reach for a double leading underscore only when you specifically need to avoid a name collision across a hierarchy.[1]
- **Prefer calling a well-defined method (like `deposit()`) over touching an attribute directly**, even one without any underscore — this keeps any validation logic in one place instead of scattered wherever the attribute is used.[1]
- **Keep each class used in multiple inheritance narrowly focused on one responsibility** (a true mixin) so that the resulting MRO stays predictable and easy to reason about.[1]
- **Do not treat a double-underscore attribute as real security.** It prevents accidental name collisions; it does not stop a determined outside caller who knows the mangled name.[1]

**Common mistakes to avoid:**[1]

- **Forgetting to call `super().__init__()`** — the superclass's attributes never get set, so any method relying on them later fails with an `AttributeError` (see the Worked Example logic in §5.3/§10).
- **Assuming a subclass "automatically" has the parent's data** — inheriting a *method* only makes it available; the object's actual *data* exists only if `__init__` genuinely ran and assigned it.
- **Diamond-problem confusion** — assuming `super()` inside a class always jumps to "its" direct parent; it actually jumps to the next class in the MRO, which in a diamond shape can be a sibling class rather than the shared ancestor.
- **Assuming Python enforces true private variables** — `self.__pin` is still reachable from outside as `self._ClassName__pin`; the double underscore avoids accidental name collisions, but it is not a security feature.
- **Building unnecessarily deep inheritance chains** just to reuse a couple of methods, when a flatter design would be easier to read and maintain.
- **Overriding a method without realizing it** — accidentally reusing a superclass's method name and silently losing access to its original behavior, without ever calling `super()` to reuse it.

## 9. Hands-On Exercise

Using the `Student` / `GraduateStudent` hierarchy from §5.2–5.3, extend it one level further: write `PhDStudent(GraduateStudent)` whose `__init__` takes the same fields as `GraduateStudent` plus `supervisor_name`, calling `super().__init__()` first. Override `has_passed()` in `PhDStudent` to require `marks >= 60`. Create one `PhDStudent` object, then print `has_passed()`, `isinstance(your_object, Student)`, and `PhDStudent.__mro__` to confirm all three classes (plus `object`) appear in the expected order. As a stretch goal, add a small `NotifiableMixin` with an `announce()` method, mix it into a second class alongside `PhDStudent`, and inspect that new class's `__mro__` to confirm you can predict the search order before running the code.

## 10. Key Takeaways

- **Inheritance** (`class Child(Parent):`) gives a subclass every attribute-setting and method the superclass has, automatically, with zero code duplication.
- **`super()`** calls the next class in the object's computed Method Resolution Order (MRO) — not necessarily a literal "parent" — which is what lets a subclass extend inherited behavior instead of retyping it; this distinction only becomes visible with multiple inheritance and the diamond problem.
- **Multiple inheritance** (`class C(A, B):`) combines behavior from more than one direct superclass into one class, and Python resolves any ambiguity using a fixed MRO you can always inspect via `ClassName.__mro__`.
- **Encapsulation in Python is convention, not enforcement**: a single underscore (`_name`) signals "internal use" without being enforced, and a double underscore (`__name`) triggers name mangling (rewriting to `_ClassName__name`) mainly to avoid attribute collisions, not to guarantee privacy.
- **Skipping `super().__init__()`** in a subclass means the superclass's attributes never get created on the object, which surfaces later as an `AttributeError` the first time any code tries to use them — proving that inheriting a method only makes it *available*, while an object's actual data exists only once `__init__` genuinely runs and sets it.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
