# Abstraction: Showing What Matters and Hiding the Rest

Polymorphism let many classes answer the same method call in their own way. But nothing checked that a class had actually written that method. A class could stay silent, and Python would not complain.

Abstraction does two things. It hides how a class works, and it forces every child to provide what the parent demands.

## Abstraction

Think about paying on a shopping app. You tap **Pay**, and the money moves. You never see which bank server was contacted, which check ran first, or what got written where. You only need the button.

That is abstraction. A class offers a few simple things you can do with it, and keeps the working parts out of sight.

Four terms carry the idea.

- An **abstract method** is a method that is declared but left empty on purpose. It says the method must exist, and nothing more.
- An **abstract class** is a class holding at least one abstract method. It cannot create objects. It exists to be inherited.
- A **concrete method** is an ordinary method, with a working body, written inside that same abstract class.
- Python supplies all of this through a built-in module named **`abc`**, short for *abstract base class*.

One abstract class can hold both kinds at once. It writes out what is the same for every child, and leaves abstract what each child must write itself.

## Why Inheritance Alone Is Not Enough

A shopping app takes payment by UPI, by card, and in cash. Each one completes the payment differently, so each is a child of `Payment`.

`UpiPayment` below writes its own `pay()`. `CardPayment` was written in a hurry, and nobody added one.

```python
class Payment:
    def pay(self, amount):
        print("Payment done")


class UpiPayment(Payment):
    def pay(self, amount):
        print(f"₹{amount} paid by UPI")


class CardPayment(Payment):
    pass


UpiPayment().pay(500)
CardPayment().pay(500)
```

**Output**

```text
₹500 paid by UPI
Payment done
```

The UPI line is correct. The card line is a disaster.

`CardPayment` has no `pay()` of its own, so Python used the one in `Payment`. That method prints `Payment done` and charges nobody. The customer sees a success message, the order is placed, and no money ever moved.

`Payment` was too helpful. It kept a spare `pay()` ready, so nobody was forced to write a real one. What it should do instead is refuse.

## Declaring an Abstract Class

One line brings in what you need: `from abc import ABC, abstractmethod`.

`ABC` is a class. A parent that inherits from it becomes an abstract class. `abstractmethod` is a marker. Write `@abstractmethod` above a method, and every child must write that method.

The `@` symbol makes a **decorator**, a line placed above a method to change how it is treated. Decorators come later as a topic. This is the only one needed here.

Below, `pay()` is abstract with `pass` as its body. `checkout()` is an ordinary method with a working body.

```python
from abc import ABC, abstractmethod


class Payment(ABC):
    def checkout(self, amount):
        print(f"Amount: ₹{amount}")
        self.pay(amount)

    @abstractmethod
    def pay(self, amount):
        pass


class UpiPayment(Payment):
    def pay(self, amount):
        print(f"₹{amount} paid by UPI")


class CardPayment(Payment):
    def pay(self, amount):
        print(f"₹{amount} paid by card")


UpiPayment().checkout(500)
CardPayment().checkout(500)
```

**Output**

```text
Amount: ₹500
₹500 paid by UPI
Amount: ₹500
₹500 paid by card
```

Both payments now work properly. Three things did that.

- `Payment` inherits from `ABC`. Without it, the marker does nothing.
- `pay()` is marked abstract, so both children were forced to write their own.
- `checkout()` was written once and both children use it free, because showing the amount is the same job every time.

Look again at `checkout()`. It shows the amount and then calls `self.pay(amount)`, without knowing how any payment works. The marker is what makes that trust safe.

Why is the body of `pay()` just `pass`? Because `Payment` has nothing to put there. There is no way to pay in general. Each child decides.

If you have seen Java, this is the job an `interface` does. Python has no `interface` keyword, and an abstract class fills the same role.

## The Rules Python Enforces

**An abstract class cannot create an object.**

```python
from abc import ABC, abstractmethod


class Payment(ABC):
    @abstractmethod
    def pay(self, amount):
        pass


payment = Payment()
```

**Output**

```text
Traceback (most recent call last):
  File "app.py", line 10, in <module>
    payment = Payment()
              ^^^^^^^^^
TypeError: Can't instantiate abstract class Payment without an implementation for abstract method 'pay'
```

This matches the app you already use. No checkout screen lets you simply pay. You pick UPI, card, or cash first. `Payment` describes what every method must do and never describes a real one, so Python refuses to make one.

**A child that skips the abstract method cannot create an object either.**

```python
from abc import ABC, abstractmethod


class Payment(ABC):
    @abstractmethod
    def pay(self, amount):
        pass


class CardPayment(Payment):
    pass


card = CardPayment()
```

**Output**

```text
Traceback (most recent call last):
  File "app.py", line 14, in <module>
    card = CardPayment()
           ^^^^^^^^^^^^^
TypeError: Can't instantiate abstract class CardPayment without an implementation for abstract method 'pay'
```

This is the rule that kills the earlier bug. The same empty `CardPayment` now stops the program on the line that creates it. The mistake is caught in seconds, not after a customer complains about money.

Two details are easy to get wrong.

- `@abstractmethod` alone enforces nothing. Leave out `ABC` and Python builds objects happily. `ABC` is what makes Python check.
- A child must write every abstract method the parent declares. Missing one out of three leaves the child unusable.

## Abstraction and Encapsulation

Both hide something, which is why they get mixed up. They hide different things.

| Question | Encapsulation | Abstraction |
|---|---|---|
| What is hidden | The data inside an object | The steps inside a method |
| Purpose | Stop the data being changed wrongly | Save the caller from knowing how the work is done |
| Written using | `_name`, `__name`, getter and setter methods | `ABC`, `@abstractmethod`, private helper methods |

Encapsulation protects. Abstraction simplifies. Most good classes do both.

## Practice Exercises

Build a notification system, one step at a time. Each task continues the file from the task before it.

1. **See the bug.** Write a normal class `Notifier` with a `send()` method that prints `Sent`. Add `SmsNotifier(Notifier)` with its own `send()` printing an SMS line. Add `class EmailNotifier(Notifier): pass`. Call `send()` on one of each and note which line is wrong.

2. **Make the parent abstract.** Add `from abc import ABC, abstractmethod`. Make `Notifier` inherit from `ABC` and mark `send()` with `@abstractmethod`, body `pass`. Run it and read the `TypeError`.

3. **Satisfy the rule.** Give `EmailNotifier` its own `send()`. Run it again and confirm both lines are correct.

4. **Try to break it.** Attempt `Notifier()` on a new line. Read the `TypeError`, then comment that line out.

5. **Add a concrete method.** Give `Notifier` a `notify(user)` method with a working body that prints the user's name and then calls `self.send()`. Call `notify()` on both children without writing it in either one.

## Reference Links

- [Python Official Docs — `abc` module](https://docs.python.org/3/library/abc.html)
- [Python Official Docs — `abc.abstractmethod`](https://docs.python.org/3/library/abc.html#abc.abstractmethod)
- [GeeksforGeeks — Data Abstraction in Python](https://www.geeksforgeeks.org/python/data-abstraction-in-python/)
- [GeeksforGeeks — Abstract Classes in Python](https://www.geeksforgeeks.org/python/abstract-classes-in-python/)
