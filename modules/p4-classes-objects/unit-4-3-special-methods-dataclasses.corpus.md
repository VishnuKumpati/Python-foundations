---
topic_id: 5c3d6c3e-8443-48e6-8568-b6fd236d1bea
title: Unit 4.3 - Special Methods & Dataclasses
position_in_module: 3
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 4.3 - Special Methods & Dataclasses — Topic Corpus

## 2. Prerequisites

- `e92e9967-9d4c-4992-a8f3-5681762aaeaa` — **Unit 4.1: Object-Oriented Foundations**. This unit assumes you are already comfortable creating a class, writing `__init__`, and calling methods with `self` — every example below builds directly on that pattern rather than re-explaining it.
- `b626a741-e204-40ec-9d6a-85ca553262f9` — **Unit 4.2: Inheritance & Encapsulation**. This unit assumes you already understand that a class can extend another with `super()`, and that Python resolves attribute access errors as `AttributeError` — both ideas reappear briefly below (dataclasses still support inheritance; comparing mismatched objects still needs care).

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** what a dunder (special) method is, and why Python calls methods like `__init__`, `__str__`, and `__add__` automatically instead of you calling them by name.
- **Implement** `__str__` and `__repr__` on a class to control how its instances print, and explain what happens when only one of the two is defined.
- **Apply** `__eq__` so that two instances with matching data compare as equal, instead of Python's default identity-based comparison.
- **Implement** operator overloading using `__add__`, so a custom class responds meaningfully to the `+` operator.
- **Create** simple data-holding classes using the `@dataclass` decorator, and identify exactly which methods it generates — and which it never generates.
- **Differentiate** a hand-written class from a `@dataclass`, and decide which one fits a given class's purpose.

## 4. Introduction

Try this with any object you build yourself: `print(my_object)`. Instead of anything useful, Python shows something like `<__main__.Wallet object at 0x7f2a4c1b3d90>` — a bare memory address. Now try comparing two objects that clearly hold the same data: `wallet1 == wallet2` still comes back `False`, even when every attribute matches. And if you try `wallet1 + wallet2`, hoping to combine two balances the way you'd add two numbers, Python refuses outright.

None of this is Python being broken. Every one of these behaviours — printing, comparing with `==`, combining with `+` — is something Python only knows how to do for the *data types it ships with*, like numbers and strings. For a class you write yourself, Python has no idea what "printing it nicely," "being equal," or "adding two of them" should even mean. It needs you to tell it. This unit is about the small set of methods Python uses to ask that question, and about a shortcut — the `@dataclass` decorator — that answers the most common version of it automatically, so you don't have to write the same boilerplate by hand for every class that is mostly just a bundle of data.

This matters immediately, not just in theory. Any capstone tool you build with classes — a wallet tracker, a booking system, a small inventory manager — will eventually need to print a record to a log, check whether two records represent "the same thing," or combine two amounts. Getting comfortable with this now is what makes those later features straightforward instead of confusing.

## 5. Core Concepts

### 5.1 Dunder Methods: How Python Talks to Your Class

A **dunder method** (short for "double underscore," also called a **magic method** or **special method**) is a method whose name begins and ends with two underscores — `__init__`, `__str__`, `__repr__`, `__eq__`, and `__add__` are all examples [1]. You have already used one of these without necessarily naming it: writing `Wallet("Rohit", 500.0)` triggers `__init__` automatically — you never type `.` `__init__` `(...)` yourself. Every other dunder method works the same way: Python calls it *implicitly*, behind the scenes, in response to some everyday piece of syntax, rather than you calling it by name like a normal method.

The pattern to notice is: a dunder method is Python's way of asking your class "what should this built-in operation mean for you?" `print(obj)` is really Python asking "how do you want to be displayed?" `obj1 == obj2` is Python asking "what makes two of you equal?" `obj1 + obj2` is Python asking "what does combining two of you produce?" If you never answer, Python falls back to a generic default defined on `object` — the base class every class in Python ultimately extends, whether it says so or not. That default has no idea what your data represents, so it does the only honest thing it can: show a memory address for printing, and compare by identity (are these literally the same object in memory?) for `==`.

### 5.2 Controlling How an Object Prints: `__str__` and `__repr__`

Two separate dunder methods control how an object is displayed, aimed at two different audiences [1]:

- **`__str__`** is called by `print()`, by `str()`, and by f-strings, whenever it exists. Its job is to produce a **readable, human-facing** description — something a non-technical end user would find friendly, e.g. `"Rohit's wallet: Rs. 500.00"`.
- **`__repr__`** is called by `repr()`, and by an interactive shell echoing a value back at you. Its job is to produce a **precise, developer-facing** description, ideally one that looks like the code needed to recreate the object, e.g. `"Wallet(owner_name='Rohit', balance=500.0)"`.

A concrete example makes the split obvious. Suppose `Wallet` has `owner_name` and `balance` as instance attributes, set in `__init__` exactly as in Unit 4.1:

```python
def __repr__(self):
    return f"Wallet(owner_name={self.owner_name!r}, balance={self.balance})"

def __str__(self):
    return f"{self.owner_name}'s wallet: Rs. {self.balance:.2f}"
```

`__repr__` uses the `!r` conversion inside the f-string, which wraps the string value in quotes — so `owner_name={self.owner_name!r}` shows `owner_name='Rohit'`, exactly as it would need to look if you typed it into code yourself. `__str__` instead formats `balance` to two decimal places, because this value represents money and a human reader expects `500.00`, not `500.0`. Calling `print(wallet)` shows the friendly `__str__` line; calling `repr(wallet)` (or just evaluating `wallet` in an interactive shell) shows the precise `__repr__` line.

The relationship between the two matters: `object`'s own default `__str__` is written to simply call `self.__repr__()`. So if a class defines `__repr__` but never defines `__str__`, `print()` still shows something useful — it silently falls back to whatever `__repr__` produces. Only when *both* are defined does `print()` prefer the friendlier `__str__`, and `repr()` still shows the developer-facing version when asked directly. If neither is defined, there is nothing to fall back to, and you get the bare memory address from `object`'s built-in default.

A dunder method that is meant to produce text must actually `return` a string — forgetting the `return` doesn't silently fail; Python raises an error at the moment the value is used (`TypeError: __str__ returned non-string (type NoneType)`), because `print()` and `str()` genuinely need a string back to display. (As with the `TypeError` raised for an unsupported `+` in section 5.4, formally catching and handling this kind of error with `try`/`except` is a technique covered in a later module — here it only matters that you recognize the message and fix the missing `return`.)

### 5.3 Comparing by Data, Not Identity: `__eq__`

`==` between two custom objects, with no dunder method involved, checks **identity**: are these literally the same object sitting at the same place in memory? Two separately-created objects holding identical attribute values still compare `False`, because they are two different objects, even though their data matches.

**`__eq__`** is the dunder method called by `==`. Defining it lets you redefine what "equal" means for your class — typically, "same type, and every attribute matches" [1]:

```python
def __eq__(self, other):
    return isinstance(other, Wallet) and self.owner_name == other.owner_name and self.balance == other.balance
```

Two details matter here. First, `other` — the second value being compared — arrives as an ordinary parameter with no guarantee it is even the same type as `self`; comparing `wallet == 5` is perfectly legal Python, so `__eq__` must handle it gracefully rather than crash. That is why `isinstance(other, Wallet)` is checked *first*: `and` evaluates left to right and stops as soon as one side is `False`, so if `other` is not a `Wallet` at all, the expression returns `False` immediately without ever touching `other.owner_name` — an attribute that might not even exist on `other` and would otherwise raise an `AttributeError`, the same error you met evaluating attribute access in Unit 4.2. Second, once `isinstance` confirms the type, every attribute you care about is compared with plain `==`, and only if *all* of them match does the whole expression become `True`.

### 5.4 Operator Overloading: Making `+` Mean Something for Your Class

**Operator overloading** is the general technique this unit has been demonstrating one operator at a time: defining a dunder method so that a standard Python operator does something meaningful for your own class, instead of raising an error or falling back to something useless [1]. `__eq__` is operator overloading for `==`; `__add__` is the same idea applied to `+`.

Python does not support what languages like Java or C++ call "method overloading" — writing several methods with the *same name* that differ only by parameter type, resolved automatically at compile time. Python has no equivalent mechanism. Instead, each operator maps to exactly *one* dunder method slot per class (`+` always calls `__add__`, `==` always calls `__eq__`), and whatever logic you write inside that one slot decides how to handle the situation.

```python
def __add__(self, other):
    combined_balance = self.balance + other.balance
    return Wallet(f"{self.owner_name} + {other.owner_name}", combined_balance)
```

Writing `wallet1 + wallet2` calls `wallet1.__add__(wallet2)` implicitly — `self` is the left-hand operand, `other` is the right-hand one. Note that `self.balance + other.balance` inside the method body is plain numeric addition on two `float` values; nothing there is being overloaded — the overloading is the outer `__add__` definition itself, not every `+` that happens to appear inside it. The method builds and returns a **brand-new** `Wallet` rather than modifying `self` in place, matching how `+` behaves for ordinary numbers: `3 + 4` never changes the value `3`. Without any `__add__` defined, `wallet1 + wallet2` raises `TypeError: unsupported operand type(s) for +: 'Wallet' and 'Wallet'` — a runtime error, since `object` has no built-in notion of "adding" two custom objects. (Handling that kind of error gracefully with `try`/`except` is a technique covered in a later module; here it is enough to recognize why the error happens and how `__add__` prevents it.)

Calling this method looks like ordinary arithmetic, but two separate function calls are hiding underneath it:

```python
wallet1 = Wallet("Rohit", 500.0)
cashback_wallet = Wallet("Rohit-Cashback", 45.50)
total_wallet = wallet1 + cashback_wallet   # calls wallet1.__add__(cashback_wallet)
print(total_wallet)                        # calls total_wallet.__str__()
```

`wallet1 + cashback_wallet` is translated by Python into `wallet1.__add__(cashback_wallet)` — you never write that call yourself, but it is exactly what happens. The result, `total_wallet`, is a brand-new `Wallet`, so it can immediately be printed, compared with `__eq__`, or even added again, exactly like any other `Wallet`.

One advanced detail is worth naming without teaching it in depth: a dunder method can return the special value `NotImplemented` to tell Python "I don't know how to combine these two types" and let Python try other options before giving up. This matters for library authors supporting many mixed-type operations; it is rarely needed for the straightforward classes built in this unit, so it is mentioned here only so the name is recognizable if you encounter it later.

### 5.5 Dataclasses: Removing Boilerplate for Data-Only Classes

Many classes in real projects are little more than a labeled bundle of attributes with almost no custom behaviour — a configuration object, a record fetched from storage, a simple booking. Writing `__init__`, `__repr__`, and `__eq__ ` by hand for every such class is repetitive, and repetition is exactly where bugs hide: add a new attribute to `__init__` but forget to add it to a hand-written `__eq__`, and equality silently stops checking that attribute.

A **dataclass** is an ordinary class, decorated with `@dataclass` from Python's built-in `dataclasses` module, that has `__init__`, `__repr__`, and `__eq__` generated automatically from a short list of type-hinted attributes, instead of you writing them by hand [1]:

```python
from dataclasses import dataclass

@dataclass
class Wallet:
    owner_name: str
    balance: float
```

Each type-hinted attribute declared this way is called a **field** — it becomes one parameter, in the same order, of the generated `__init__`. `@dataclass` reads these fields and builds `__init__`, `__repr__`, and `__eq__` from them without you writing a single line of that code yourself. A field can carry a default value (`balance: float = 0.0`), which makes it an optional parameter — but every field with a default must be declared *after* every field without one, since the generated `__init__` places parameters in declaration order; putting a required field after a defaulted one raises a `TypeError` at the moment the class itself is defined, before the program even runs.

Two limits matter just as much as what `@dataclass` generates. First, a plain attribute with no type hint is not recognized as a field at all — it is silently left out of the generated `__init__`, `__repr__`, and `__eq__`. Second, and easy to forget: **`@dataclass` never generates `__str__`**. If you print a dataclass instance without writing `__str__` yourself, you see exactly what `__repr__` produces — purely because of the same fallback rule from section 5.2 (`object`'s default `__str__` calls `__repr__`). A dataclass is still an ordinary class underneath the decorator: you can add hand-written methods like `__str__` or `__add__` to it exactly as you would to any class, and it still supports inheritance the same way covered in Unit 4.2.

## 6. Implementation

Building a class's dunder methods (or replacing them with a dataclass) is easiest done as an incremental sequence, adding one capability at a time and confirming it before moving to the next [1]:

1. **Start with a plain class.** Write `__init__` only, exactly as in Unit 4.1. Confirm `print()` shows a bare memory address and `==` compares by identity — this is the baseline you are improving on.
2. **Add `__repr__`.** Return an unambiguous, code-like string built from the object's own attributes (e.g. using f-strings and the `!r` conversion to show strings with quotes). Confirm `repr(obj)` now shows something useful, and that `print(obj)` does too, via the fallback rule.
3. **Add `__str__` if the friendly, human-facing message should differ from the debugging one.** Confirm `print(obj)` now prefers this version over `__repr__`.
4. **Add `__eq__` if two instances with matching data should compare equal.** Always guard with `isinstance(other, ClassName)` before touching any attribute of `other`.
5. **Add `__add__` (or another operator's dunder method) only if combining two instances with that operator is meaningful for this class.** Build and return a new instance; never mutate `self`.
6. **Decide whether the class should become a `@dataclass` instead.** If, after this process, the class is mostly `__init__` + `__repr__` + `__eq__` with little or no other behaviour, replace those three with `@dataclass` and keep only the genuinely custom methods (like a hand-written `__str__` or `__add__`) in the class body.

This order matters pedagogically more than technically — writing `__repr__` before `__eq__` means you can already *see* a meaningful failure ("these look identical but compare `False`") before you fix it, rather than fixing something you can't yet observe.

## 7. Real-World Patterns

- **Banking and FinTech:** An `Account` class implements `__repr__` so a crash log shows `Account(id=48213, balance=15420.75)` instead of a memory address, giving an engineer enough detail to diagnose a problem without re-running the program [1].
- **UPI and payment systems:** Two receipt objects compare equal through a custom `__eq__` that checks transaction IDs rather than object identity — this is the identity-vs-data distinction from section 5.3, applied to a real feature; a wallet-recharge feature overloads `__add__` to combine a main balance and a cashback balance in one expression [1].
- **E-commerce:** A `CartItem` dataclass holds `product_name`, `price`, and `quantity` — pure data, with generated `__repr__` and `__eq__` included for free and no custom behaviour needed [1].
- **Healthcare:** A `PatientRecord` class overrides `__eq__` to detect duplicate patient entries submitted from two different hospital counters, even though the two submissions arrive as two separate objects in memory [1].
- **AI/ML configuration:** A model's hyperparameters — `batch_size`, `learning_rate`, `epochs` — are almost always modeled as a dataclass, since a configuration is pure data with no behaviour of its own, precisely the situation `@dataclass` exists for [1].
- **Railway and travel booking systems:** A `Ticket` dataclass models a confirmed booking as data ready to be printed, compared, or stored, without any hand-written boilerplate [1].
- **Cloud applications:** Microservices routinely pass small, well-defined data objects between services, relying on a dataclass's auto-generated `__repr__` to make request and response logging readable during debugging [1].

## 8. Best Practices

- Implement `__repr__` on any class you expect to debug or log — an unambiguous representation saves real time when something goes wrong.
- Only implement `__str__` when the human-facing message genuinely needs to differ from the debugging one; relying on the `__repr__` fallback is perfectly fine otherwise.
- Reach for `@dataclass` when a class is mostly data — configuration, records, simple bundles — and reserve hand-written classes for ones with real behaviour or invariants to protect.
- Always guard `__eq__` with `isinstance(other, ClassName)` before comparing attributes, so comparing against an unrelated type returns `False` cleanly instead of raising an error.
- Keep the object returned by `__add__` (or any operator method) the same type as the operands, so the result itself can still be printed, compared, or combined again.
- Don't force a class with genuine validation logic or custom behaviour into `@dataclass` just to save typing — a hand-written class communicates that design better.

## 9. Hands-On Exercise

Using a plain `Wallet` class with `owner_name` and `balance`: (1) add `__repr__` and confirm `print()` now shows something useful even before `__str__` exists; (2) add `__eq__` and verify two separately-created wallets with identical data compare `True`, while `is` on the same pair still returns `False`; (3) add `__add__` and combine two wallets with `+`; (4) rewrite the whole class as a `@dataclass`, keeping only `__str__` and `__add__` hand-written, and confirm the printed, compared, and combined results are unchanged.

## 10. Key Takeaways

- A **dunder method** is a method Python calls implicitly in response to built-in syntax — `__str__`/`__repr__` for printing, `__eq__` for `==`, `__add__` for `+` — rather than something you call by name.
- **`__repr__`** is the developer-facing, unambiguous description; **`__str__`** is the human-facing one. Without `__str__`, `print()` falls back to `__repr__`, because `object`'s default `__str__` calls `self.__repr__()`.
- **`__eq__`** redefines `==` to compare data instead of memory identity; without it, two objects holding identical attributes still compare `False`.
- **Operator overloading** means defining one dunder method per operator (like `__add__` for `+`) so it behaves meaningfully for a custom class — Python has no separate "method overloading by parameter type" the way Java or C++ do.
- **`@dataclass`** generates `__init__`, `__repr__`, and `__eq__` from type-hinted fields — but never `__str__` — provided every field has a type hint and every defaulted field comes after every required one; it remains an ordinary, inheritable class underneath.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
