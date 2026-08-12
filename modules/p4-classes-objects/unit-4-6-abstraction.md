# Abstraction: Showing What Matters and Hiding the Rest

Polymorphism let every class answer the same method name in its own way. A `Creator` printed one kind of report and an `Advertiser` printed another, and the loop calling them worked either way.

But nothing checked whether a class had written that method at all. A class could stay silent, and Python would not complain.

Abstraction deals with two things: how much a class shows to the outside, and what it forces its children to write.

## Abstraction

Abstraction is the practice of exposing only what someone needs in order to use a class. Every detail of how the work is done stays hidden.

The vocabulary comes in four terms.

- An **abstract method** is a method that is declared but deliberately left empty. It states that the method must exist, and nothing more.
- An **abstract class** is a class holding at least one abstract method. It cannot create objects. It exists to be inherited.
- A **concrete method** is an ordinary method, with a working body, written inside that same abstract class.
- Python supplies all of this through a built-in module named **`abc`**, short for *abstract base class*.

An abstract class is free to hold both kinds together. It writes the methods that work the same way for every child. It leaves abstract the ones each child must write for itself. A class with three concrete methods and one abstract method is still an abstract class.

Why any of this is needed shows up fastest in a class that does without it.

## Why Inheritance Alone Is Not Enough

Below, `Account` has a `monthly_report()` method. `Creator` writes its own version. `Advertiser` was built in a hurry and the method was never added.

```python
class Account:
    def __init__(self, handle):
        self.handle = handle

    def monthly_report(self):
        print(f"@{self.handle} | No report available")


class Creator(Account):
    def monthly_report(self):
        print(f"@{self.handle} | Earned: ₹45000")


class Advertiser(Account):
    pass


for account in [Creator("priya_cooks"), Advertiser("zomato")]:
    account.monthly_report()
```

**Output**

```text
@priya_cooks | Earned: ₹45000
@zomato | No report available
```

Python raised nothing. `Advertiser` had no `monthly_report()`, so Python went up to `Account` and used what it found there.

That is the danger. The program runs, the reports page loads, and Zomato's report says nothing. No warning appears anywhere. The mistake surfaces when a customer asks why their report is empty.

What the parent needs is a way to state a requirement rather than a fallback: *every account type must write its own `monthly_report()`, and Python must refuse any class that does not.*

## Declaring an Abstract Class

The line `from abc import ABC, abstractmethod` brings in the two names needed. `ABC` is a class you inherit from. `abstractmethod` is a marker you place above a method.

That marker is written as `@abstractmethod` on the line above it. The `@` form is called a **decorator** — it attaches a marker to the method written underneath. Here it marks the method as one every child is required to provide.

`Account` below carries one method of each kind. `show_handle()` is concrete, with a working body. `monthly_report()` is abstract, with `pass` for a body.

```python
from abc import ABC, abstractmethod


class Account(ABC):
    def __init__(self, handle):
        self.handle = handle

    def show_handle(self):
        print(f"@{self.handle}")

    @abstractmethod
    def monthly_report(self):
        pass


class Creator(Account):
    def monthly_report(self):
        print(f"@{self.handle} | Earned: ₹45000")


class Advertiser(Account):
    def monthly_report(self):
        print(f"@{self.handle} | Spent: ₹80000")


priya = Creator("priya_cooks")
priya.show_handle()
priya.monthly_report()

Advertiser("zomato").monthly_report()
```

**Output**

```text
@priya_cooks
@priya_cooks | Earned: ₹45000
@zomato | Spent: ₹80000
```

Three things are worth noticing.

- `Account` inherits from `ABC`. That is what switches the checking on.
- `show_handle()` was written once in `Account`, and both children use it without writing anything. Displaying a handle is the same for every account type.
- `monthly_report()` carries `@abstractmethod`, so `Creator` and `Advertiser` each had to write their own. A report differs for every account type, so `Account` only asked for it.

`Advertiser` could no longer stay empty the way it did earlier. The blank report is gone, because Python would not have allowed the class to exist without a real `monthly_report()`.

`pass` in an abstract method is not laziness. `Account` has nothing sensible to put there, because only the child knows what its own report should say. The parent states the requirement; the child supplies the answer.

Python has no `interface` keyword, which you may have seen in Java. An abstract class whose methods are all abstract does that same job here.

## The Rules Python Enforces

Two rules now hold, and Python applies them on its own.

**An abstract class cannot create an object.**

```python
from abc import ABC, abstractmethod


class Account(ABC):
    def __init__(self, handle):
        self.handle = handle

    @abstractmethod
    def monthly_report(self):
        pass


account = Account("priya_cooks")
```

**Output**

```text
Traceback (most recent call last):
  File "app.py", line 13, in <module>
    account = Account("priya_cooks")
              ^^^^^^^^^^^^^^^^^^^^^^
TypeError: Can't instantiate abstract class Account without an implementation for abstract method 'monthly_report'
```

`Account` describes what an account must be able to do. It never describes a real account, so building one from it is meaningless — and Python says so.

**A child that skips an abstract method cannot create an object either.**

```python
from abc import ABC, abstractmethod


class Account(ABC):
    def __init__(self, handle):
        self.handle = handle

    @abstractmethod
    def monthly_report(self):
        pass


class Advertiser(Account):
    pass


zomato = Advertiser("zomato")
```

**Output**

```text
Traceback (most recent call last):
  File "app.py", line 17, in <module>
    zomato = Advertiser("zomato")
             ^^^^^^^^^^^^^^^^^^^^
TypeError: Can't instantiate abstract class Advertiser without an implementation for abstract method 'monthly_report'
```

This is the rule that closes the hole. That same empty `Advertiser` printed a blank report earlier. Now it stops the program on the line that tries to create it. Caught in seconds, not by a customer weeks later.

Two details are easy to get wrong.

- `@abstractmethod` on its own enforces nothing. Leave out `ABC` and Python will build objects from the class quite happily. `ABC` is what switches the checking on.
- A child must provide every abstract method the parent declares. Missing one out of three leaves the child unusable.

## Abstraction and Encapsulation

These two get mixed up constantly, because both hide something. They hide different things, for different reasons.

| Question | Encapsulation | Abstraction |
|---|---|---|
| What is hidden | The data inside an object | The steps inside a method |
| Purpose | Stop the data being changed wrongly | Save the caller from knowing how the work is done |
| Written using | `_handle`, `__handle`, getter and setter methods | `ABC`, `@abstractmethod`, private helper methods |

Encapsulation protects. Abstraction simplifies. Most well-built classes do both: private data reached only through checked methods, and a small set of public methods that hide the work behind them.

## Practice Exercises

Build a delivery app's notification system, one step at a time. Each task continues the file from the task before it.

1. **Watch the silent fallback.** Write an ordinary class `Notifier` with `__init__(self, user)` and a `send()` method printing `no channel set`. Add `class SmsNotifier(Notifier)` with its own `send()` printing an SMS line. Add `class EmailNotifier(Notifier): pass`. Put one of each in a list, loop over it calling `send()`, and note which line is wrong.

2. **Make the parent abstract.** Add `from abc import ABC, abstractmethod` at the top. Make `Notifier` inherit from `ABC` and mark `send()` with `@abstractmethod`, body `pass`. Run the file and read the `TypeError`.

3. **Satisfy the requirement.** Give `EmailNotifier` its own `send()`. Run the loop again and confirm both lines are now correct.

4. **Try to break it.** On a new line, attempt `Notifier("Arun")` directly. Read the `TypeError`, then turn that line into a comment so the file keeps running.

5. **Add a concrete method.** Add `show_user()` to `Notifier` with a working body that prints the user's name. Call it on both children without writing it in either one. Then add a comment naming which method is abstract, which is concrete, and why each belongs where it is.

---

Four pillars, four different questions.

Encapsulation asked who may touch an object's data. Inheritance asked how related classes share what they have in common. Polymorphism asked how one name can serve many classes. Abstraction asked how little a class needs to show, and how much it can demand.

None of them make a program run faster. All of them keep it understandable once it grows past the size one person can hold in their head.

---

## Reference Links

- [Python Official Docs — `abc` module](https://docs.python.org/3/library/abc.html)
- [Python Official Docs — `abc.abstractmethod`](https://docs.python.org/3/library/abc.html#abc.abstractmethod)
- [GeeksforGeeks — Data Abstraction in Python](https://www.geeksforgeeks.org/python/data-abstraction-in-python/)
- [GeeksforGeeks — Abstract Classes in Python](https://www.geeksforgeeks.org/python/abstract-classes-in-python/)
