# The Four Pillars of OOP

Unit 4.1 ended with a single, complete `BankAccount` class: an `__init__` that set up `owner_name` and `balance`, a `deposit()` and a `withdraw()` method, and a `describe()` method that printed a tidy summary. That class already works, and you already built objects from it. But look closely at what it does *not* stop you from doing: nothing prevents `account.balance = -100000` from running successfully, nothing forces a `CurrentAccount` you might add later to actually support the same operations as a `SavingsAccount`, and nothing lets you write one loop that processes a mixed list of both without checking, by hand, which one each object is.

That single class-and-object pattern from Unit 4.1 is the foundation, but object-oriented programming rests on four bigger ideas that make classes genuinely powerful once your programs grow past a handful of lines and a handful of objects: **encapsulation**, **abstraction**, **inheritance**, and **polymorphism**. Every serious object-oriented codebase you will ever touch — the account hierarchy behind a bank's core system, the payment classes behind a UPI gateway, the delivery-partner classes behind a Swiggy-style dispatch engine — is organised around exactly these four ideas, and once you can name which one you're using and why, reading someone else's class hierarchy stops feeling like guesswork.

By the end of this unit, you will take that same `BankAccount` idea and rebuild it four times over — protecting its balance, defining what every account type must be able to do, reusing its logic across several account types, and finally writing one loop that treats all of them correctly without ever asking "which one is this?"

---

## Same class, four different questions

Picture a real bank's engineering team looking at a plain `BankAccount` class and asking four separate design questions about it, each one a different pillar:

| Pillar | The question it answers |
|---|---|
| Encapsulation | How do we stop `balance` from being set to something invalid by code outside the class? |
| Abstraction | How do we describe *what* every account type must be able to do, without dictating exactly *how*? |
| Inheritance | How do we build a `SavingsAccount` and a `CurrentAccount` that reuse everything `BankAccount` already does? |
| Polymorphism | How can one line of calling code work correctly whether it's handed a `SavingsAccount`, a `CurrentAccount`, or a type nobody has written yet? |

None of these four questions is really about syntax — they are about *design*. Python gives you a small, specific piece of syntax for each one, and this unit takes them in turn, building up one continuous `BankAccount` hierarchy as it goes. Before reading on, glance back at the four questions above and, just from their wording, guess which pillar you'd reach for first when designing a brand-new class family from scratch. (Most engineers reach for abstraction first — deciding *what* a family of classes must do — before inheritance, which decides *how* that gets shared.)

## Encapsulation: keeping data safe behind the class

Think of encapsulation the way you'd think of a medicine capsule. The capsule's shell holds the actual drug inside it, released only in a controlled way, at the right time and place — you don't just pour the loose powder directly onto your tongue. **Encapsulation** is the practice of bundling an object's data together with the methods that operate on it, inside one class, while signalling — and where possible, controlling — which parts of that data outside code should touch directly.

In Unit 4.1, `self.balance` was a plain **public attribute**: any code, anywhere, could read it *or* overwrite it, with zero restriction. That is Python's default for every attribute you write, and it is exactly the gap that let `account.balance = -100000` run without complaint. Python signals a different intention through two naming conventions applied to an attribute name, both of which you write inside `__init__` exactly like any other attribute assignment.

A single leading underscore — `self._balance` — marks a **protected attribute**: a polite, human-readable signal meaning "this is internal bookkeeping; use the class's own methods instead of reaching in directly." A double leading underscore — `self.__pin` — goes one step further and triggers **name mangling**: Python automatically rewrites every occurrence of `self.__pin` inside the class body to `self._ClassName__pin`, substituting the exact name of the class the code is written in.

```python
class BankAccount:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self._balance = balance      # protected: a convention only
        self.__pin = "1234"          # private: gets name-mangled

    def show_balance(self):
        return f"{self.owner_name}'s balance: Rs. {self._balance}"


account = BankAccount("Priya Nair", 5000)
print(account.show_balance())
print(account._balance)
print(account._BankAccount__pin)
```

```
Priya Nair's balance: Rs. 5000
5000
1234
```

Notice what this proves: `account._balance` is read successfully from *outside* the class — the underscore never actually stopped anything, it only signalled intent. And `account.__pin` (the name as originally written) would fail, but `account._BankAccount__pin` — the mangled name Python actually rewrote it to — works perfectly. Try predicting, before you check, what would happen if you typed `account.__pin` directly into a cell right now — Python raises an `AttributeError`, because that literal name was never assigned; only the mangled version was.

**Python's encapsulation is convention, not enforcement — a single underscore is a request between developers, and a double underscore only renames the attribute, it does not lock it away.** Name mangling's real job is avoiding accidental collisions across a class hierarchy — for instance, if a subclass happens to define its own unrelated `self.__pin` — not providing genuine security.

| Naming style | Example | Enforced by Python? |
|---|---|---|
| Public | `self.balance` | No — no restriction is signalled at all |
| Protected (`_name`) | `self._balance` | No — a social convention between developers only |
| Private (`__name`) | `self.__pin` | Partially — the original name stops working, the mangled name still works fine |

So if a leading underscore doesn't actually stop anyone, how does a bank enforce a real rule like "balance can never go negative"? By pairing the protected attribute with a **property** — a method dressed up to look, from the outside, exactly like a plain attribute. You write it with the `@property` decorator for reading, and a matching `@balance.setter` for writing:

```python
class BankAccount:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self._balance = balance

    @property
    def balance(self):
        return self._balance

    @balance.setter
    def balance(self, new_amount):
        if new_amount < 0:
            raise ValueError("Balance cannot go negative")
        self._balance = new_amount


account = BankAccount("Priya Nair", 5000)
print(account.balance)
account.balance = 6200
print(account.balance)
account.balance = -100
```

```
5000
6200
ValueError: Balance cannot go negative
```

Notice the calling code never writes `account.get_balance()` or `account.set_balance(6200)` — it still writes `account.balance` and `account.balance = 6200`, exactly like touching a plain attribute. Behind that familiar dot-notation syntax, `@balance.setter` quietly runs the validation every single time, which is precisely why the last line raises instead of silently corrupting `_balance`. This is Python's idiomatic version of a getter and setter: it keeps the readable, attribute-style syntax you already know, while still enforcing a real rule.

- Confusing a protected attribute (`_balance`) with a truly hidden one, and being surprised when outside code reads it anyway.
- Adding a double underscore out of habit "for safety," when a single underscore (or nothing at all) would communicate intent just as well.
- Writing a `set_balance()`-style method instead of a `@property` setter, when the whole point of Python's convention is to keep the calling code looking like ordinary attribute access.

If a double underscore doesn't provide real security, what is it actually for? The honest answer is collision avoidance inside a hierarchy, not secrecy. Suppose a subclass happens to need its own, completely unrelated `__pin`:

```python
class GuestAccount(BankAccount):
    def __init__(self, owner_name, balance):
        super().__init__(owner_name, balance)
        self.__pin = "0000"     # a different __pin, for a different purpose


guest = GuestAccount("Visitor", 0)
print(guest._BankAccount__pin)   # the parent's own pin, "1234"
print(guest._GuestAccount__pin)  # this subclass's own pin, "0000"
```

```
1234
0000
```

Both `__pin` attributes exist on the very same `guest` object, side by side, without overwriting each other, purely because each one gets mangled with the name of the class it was written in. That is the entire point of name mangling: two independent pieces of internal bookkeeping, written by two different authors, can share a plain-looking name without silently colliding.

## Abstraction: hiding the how, exposing the what

Every time you press your car's accelerator pedal, the car moves faster. You do not need to know whether it burns petrol, runs on batteries, or does something stranger under the bonnet — the pedal is a simple, essential interface standing in front of a great deal of hidden complexity. That is **abstraction**: exposing only the essential behaviour an object offers, while hiding the internal detail of how it actually gets done. You have already relied on this constantly — calling `len(my_list)` never required you to know how Python actually counts elements internally.

In class design, abstraction usually means describing *what* every class in a family must be able to do, without dictating exactly *how* each one does it — a contract, not an implementation. Python provides a specific tool for making that contract enforceable: the **Abstract Base Class (ABC)**, from the built-in `abc` module, combined with the `@abstractmethod` decorator.

```python
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount):
        """Every payment method must define exactly how it processes a payment."""


class UPIPayment(PaymentMethod):
    def process(self, amount):
        return f"Paid Rs. {amount} via UPI"


upi = UPIPayment()
print(upi.process(500))

pm = PaymentMethod()
```

```
Paid Rs. 500 via UPI
TypeError: Can't instantiate abstract class PaymentMethod with abstract method process
```

`PaymentMethod` says *what* every payment method must do — accept an amount and process it — without providing any implementation of its own. `UPIPayment` fills that contract in, and only once it does is it actually usable. Try predicting, before you check, what would happen if `UPIPayment` were defined without a `process()` method at all — Python would refuse to create a `UPIPayment()` object too, with the identical `TypeError`, because an unfulfilled abstract method is inherited as unfulfilled.

An abstract base class is not required to leave *everything* unimplemented. Most real ones mix a few `@abstractmethod` entries with ordinary, fully working methods that every subclass simply inherits as-is:

```python
class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount):
        ...

    def log(self, amount):                  # a concrete method, shared as-is
        print(f"Processing Rs. {amount}")
```

Here, only `process()` is left for each subclass to define; `log()` already works, and every subclass — `UPIPayment`, a future `CardPayment`, anything else — gets it for free. This mixed style is what most production abstract classes actually look like: share whatever logic is genuinely identical, and leave `@abstractmethod` only for the parts that truly differ per subclass.

**Abstraction is the one pillar Python enforces mechanically rather than merely by convention: a class with any `@abstractmethod` still unimplemented simply cannot be instantiated, full stop.** This is exactly what makes abstraction the natural partner of the next two pillars — a shared contract is what inheritance builds on top of, and what makes polymorphism safe to rely on later in this unit, because every concrete class in the family is *guaranteed* to have the method you're about to call.

It's worth pausing on how abstraction differs from the encapsulation you just met, since learners frequently blur the two together:

| | Encapsulation | Abstraction |
|---|---|---|
| Question it answers | Who is allowed to touch this data? | What must this object be able to do? |
| Mechanism | Underscore naming, `@property` | `ABC`, `@abstractmethod` |
| Operates on | An object's data | A class's behaviour (its methods) |
| Enforced by Python? | Only partially (name mangling) | Yes — instantiation itself is blocked |

## Inheritance: reusing an existing class instead of rewriting it

Suppose the bank now needs a `SavingsAccount` that behaves exactly like `BankAccount`, plus an interest rate, and a `CurrentAccount` that behaves like `BankAccount` plus an overdraft limit. Retyping `owner_name`, `_balance`, and every method by hand into each new class would mean any future bug fix has to be repeated, correctly, in every copy. **Inheritance** is a mechanism where a new class automatically acquires the attributes and methods of an existing class, with zero code copied or retyped. The existing class is the **superclass** (or **parent class**); the new one is the **subclass** (or **child class**).

```python
class BankAccount:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self._balance = balance

    def deposit(self, amount):
        self._balance += amount

    def show_balance(self):
        return f"{self.owner_name}'s balance: Rs. {self._balance}"


class SavingsAccount(BankAccount):
    def __init__(self, owner_name, balance, interest_rate):
        super().__init__(owner_name, balance)
        self.interest_rate = interest_rate

    def add_interest(self):
        self._balance += self._balance * self.interest_rate / 100


sa = SavingsAccount("Rohit Verma", 50000, 4)
sa.add_interest()
print(sa.show_balance())
```

```
Rohit Verma's balance: Rs. 52000.0
```

`class SavingsAccount(BankAccount):` is the entire mechanism — that one line grants `SavingsAccount` everything `BankAccount` has. But since `SavingsAccount` needs its own new attribute, `interest_rate`, it must define its own `__init__` — and once a subclass defines its own `__init__`, the superclass's `__init__` no longer runs automatically. `super()` is the built-in function that hands the subclass a way back into the superclass's own version of a method; `super().__init__(owner_name, balance)` calls `BankAccount.__init__` directly, fully setting up the inherited part of the object *before* `self.interest_rate` gets added.

**Always call `super().__init__()` as the first line of a subclass's constructor, before adding anything new — the inherited part of the object needs to exist before you build on top of it.** Skip that call and `self._balance` never gets created at all; the first method that tries to read it fails with an `AttributeError`, not at the moment of the mistake, but later, wherever that attribute finally gets used.

Inheritance does not stop at one level. **Multi-level inheritance** is a chain of more than two classes, each extending the one directly above it:

```python
class MinorSavingsAccount(SavingsAccount):
    def __init__(self, owner_name, balance, interest_rate, guardian_name):
        super().__init__(owner_name, balance, interest_rate)
        self.guardian_name = guardian_name

    def show_balance(self):
        base = super().show_balance()
        return f"{base} (guardian: {self.guardian_name})"


minor = MinorSavingsAccount("Aditi Rao", 10000, 3, "Sunita Rao")
minor.add_interest()
print(minor.show_balance())
print(isinstance(minor, BankAccount))
```

```
Aditi Rao's balance: Rs. 10300.0 (guardian: Sunita Rao)
True
```

Each level's `super().__init__()` only needs to know about the class directly above it — `MinorSavingsAccount` hands off to `SavingsAccount`, which hands off to `BankAccount`, without `MinorSavingsAccount` ever needing to know `BankAccount` exists. `isinstance(minor, BankAccount)` returns `True` even though `BankAccount` is two levels away, because a `MinorSavingsAccount` genuinely *is a* `SavingsAccount`, which genuinely *is a* `BankAccount`. Notice too that `show_balance()` here calls `super().show_balance()` and builds on the result, rather than replacing it outright — this reuse-and-add pattern is worth remembering; you'll meet it again shortly as one specific style of polymorphism.

What happens when a class needs to combine two *unrelated* superclasses at once, rather than extending a single chain? That's **multiple inheritance**, written `class C(A, B):`:

```python
class SMSAlertMixin:
    def send_alert(self):
        return "SMS sent for this transaction"


class PremiumSavingsAccount(SavingsAccount, SMSAlertMixin):
    pass


premium = PremiumSavingsAccount("Karan Mehta", 75000, 5)
premium.add_interest()
print(premium.send_alert())
print(PremiumSavingsAccount.__mro__)
```

```
SMS sent for this transaction
(<class '__main__.PremiumSavingsAccount'>, <class '__main__.SavingsAccount'>, <class '__main__.BankAccount'>, <class '__main__.SMSAlertMixin'>, <class 'object'>)
```

`SMSAlertMixin` is a **mixin** — a small class written only to add one focused, reusable ability through multiple inheritance; nobody would ever create a bare `SMSAlertMixin()` on its own. `PremiumSavingsAccount` combines it with `SavingsAccount` in one step, gaining both `add_interest()` and `send_alert()` without either class knowing the other exists.

This raises a genuine question: when `PremiumSavingsAccount` needs a method it doesn't define itself, which class does Python search first? The answer is the **Method Resolution Order (MRO)** — a fixed order Python computes once, the moment a class is defined, listing every class it will search through, in order. `ClassName.__mro__` lets you read that order directly rather than guessing: here, `PremiumSavingsAccount` is searched first, then `SavingsAccount`, then `BankAccount`, then `SMSAlertMixin`, then `object` — Python fully resolves the first listed superclass's entire chain before even looking at the second one.

**`super()` does not mean "my direct parent" — it means "the next class in the computed MRO," and those two things only diverge once multiple inheritance is involved.** With a single superclass the two ideas happen to coincide, which is exactly why the distinction is easy to miss until you meet a hierarchy like this one.

| | Single inheritance | Multi-level inheritance | Multiple inheritance |
|---|---|---|---|
| Structure | One subclass, one superclass | A chain: `A` → `B` → `C` | One subclass, two+ superclasses at once |
| Syntax | `class B(A):` | `class B(A):` then `class C(B):` | `class C(A, B):` |
| `super()` resolves to | Always the one superclass | The class directly above, in the chain | The next class in the MRO — possibly a sibling, not a shared ancestor |
| Main risk | Very low | A chain that grows too long becomes hard to trace | Ambiguity between two superclasses, resolved automatically by the MRO |

`isinstance(minor, BankAccount)` in the multi-level example above is worth a second look, because it's genuinely useful beyond just this unit. **`isinstance(obj, Cls)`** returns `True` if `Cls` appears anywhere in `obj`'s class's MRO — not only when `Cls` is the object's exact, immediate class. A closely related check, **`issubclass(Cls1, Cls2)`**, asks the same question directly between two classes, with no object required at all: `issubclass(SavingsAccount, BankAccount)` returns `True` without ever creating an account. This is exactly why library code almost always reaches for `isinstance()` rather than an exact `type(obj) == BankAccount` comparison — `type(minor) == BankAccount` would return `False`, since `minor`'s exact type is `MinorSavingsAccount`, even though `minor` genuinely behaves like a `BankAccount` in every way that matters.

Before moving on, try tracing `PremiumSavingsAccount.__mro__` by hand from just the class definition, `class PremiumSavingsAccount(SavingsAccount, SMSAlertMixin):`, before checking it against the printed output above — if your predicted order matches, you've genuinely understood how the MRO is built, not just memorised this one example.

One shape of multiple inheritance deserves a specific name, because it looks alarming the first time you draw it out: the **diamond problem**. It happens when two superclasses that a subclass combines both trace back to the same shared ancestor — for instance, if `SavingsAccount` and `SMSAlertMixin` had *both* extended `BankAccount` themselves, `PremiumSavingsAccount(SavingsAccount, SMSAlertMixin)` would have `BankAccount` reachable through two separate paths at once, drawn as a diamond shape. You do not need to resolve this by hand — Python's MRO algorithm guarantees `BankAccount` still appears exactly once in `__mro__`, in a single, predictable position, no matter how many paths lead to it. The only thing genuinely required of you is the habit you've already been practising: read `ClassName.__mro__` to see exactly where Python landed, rather than assuming.

None of this means inheritance is always the right tool. Before writing `class Child(Parent):`, ask yourself a one-sentence test: is the new thing genuinely a specialised version of the existing class — a `SavingsAccount` really **is a** `BankAccount` — or does it merely *use* one? A `Branch` class that manages a collection of `BankAccount` objects is not itself a kind of account, so it should hold a list of accounts as an attribute (a "has-a" relationship, called **composition**) rather than extending `BankAccount` directly. **Favour composition over inheritance whenever the relationship is genuinely "has-a," and keep hierarchies you do build shallow — two or three levels stay easy to trace; five levels deep becomes genuinely hard to debug.**

## Polymorphism: one call, many behaviours

**Polymorphism** literally means "many forms." It lets one method call, written once, produce different behaviour depending on which actual object it runs on. You've already seen one version of it without naming it: `minor.show_balance()` and `sa.show_balance()` are the exact same line of calling code, yet they produce different results, because Python looks up `show_balance` on each object's *real* class at the moment the call happens — this is **method overriding**, one specific form of polymorphism, resolved at runtime based on the object, never on the type of the variable holding it.

```python
for account in [sa, minor, premium]:
    print(account.show_balance())
```

```
Rohit Verma's balance: Rs. 52000.0
Aditi Rao's balance: Rs. 10300.0 (guardian: Sunita Rao)
Karan Mehta's balance: Rs. 78750.0
```

One `for` loop, one line inside it — `account.show_balance()` — and three genuinely different outputs, because `sa` and `premium` both use `BankAccount`'s original version, while `minor`'s object runs `MinorSavingsAccount`'s overridden-and-extended version instead. Nothing in the loop itself checks which class each object belongs to; Python works that out automatically, every time, based on the object actually sitting in `account`.

It's worth naming the two flavours of overriding you've now seen separately. When a subclass's method is a complete replacement for the superclass's version — the way `GraduateStudent.has_passed()` might tighten a passing mark with no reference to the old rule at all — that's a plain **override**, with no `super()` call needed. When the subclass instead calls `super()` to reuse the superclass's version and builds something extra on top of the result — exactly what `MinorSavingsAccount.show_balance()` did above — that's usually called **extending** the method. Both are technically "overriding" the name; only the second one also reuses the original logic rather than discarding it.

A second, looser form of polymorphism needs no shared superclass at all. **Duck typing** — "if it walks like a duck and quacks like a duck, it's a duck" — means Python never checks an object's class before calling a method on it; it only checks, at the exact moment of the call, that the method exists:

```python
class UPIPayment:
    def pay(self, amount):
        return f"Rs. {amount} paid via UPI"


class CardPayment:
    def pay(self, amount):
        return f"Rs. {amount} paid via Card"


for method in [UPIPayment(), CardPayment()]:
    print(method.pay(500))
```

```
Rs. 500 paid via UPI
Rs. 500 paid via Card
```

`UPIPayment` and `CardPayment` share no common superclass whatsoever — no `PaymentMethod`, no inheritance relationship of any kind. The loop works purely because both objects happen to define a `pay()` method with a matching signature. Say out loud, in one sentence, what would happen if `CardPayment` had named its method `charge()` instead of `pay()` — the loop would fail with an `AttributeError` the moment it reached that object, precisely because duck typing checks for the method's existence, and nothing more, at the instant it's called.

This is exactly how a notification service in a real Indian tech company might be built: an `EmailNotifier` and an `SMSNotifier`, written by two different engineers, with no shared base class at all, both simply agreeing — by convention, not enforcement — to expose a `send(message)` method. A dispatcher can loop over a list of whichever notifier objects it was handed and call `.send(message)` on each, never once caring which concrete class it's actually looking at.

**Polymorphism lets one function or loop work with many different types through a single shared method name, replacing an `if isinstance(x, A): ... elif isinstance(x, B): ...` chain that would otherwise need editing by hand every time a new type appears.** There is a third flavour worth a preview, even though it belongs properly to the next unit: Python lets a custom class teach a built-in operator or function new, class-specific meaning through **dunder** (double-underscore) methods, such as `__str__` for `print()`:

```python
class BankAccount:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self._balance = balance

    def __str__(self):
        return f"{self.owner_name}'s balance: Rs. {self._balance}"


account = BankAccount("Priya Nair", 5000)
print(account)
```

```
Priya Nair's balance: Rs. 5000
```

`print(account)` never calls `show_balance()` at all — Python automatically calls `__str__` on whatever object you hand `print()`, and `BankAccount` now defines what that should produce. This is the same underlying idea as method overriding, just applied to a built-in operator or function instead of a method you named yourself, and it's exactly where Unit 4.3 picks up, once you have `@property` and encapsulation from this unit already in hand.

## Where these four pillars actually show up

These are not classroom-only ideas — they are how real Indian engineering teams structure everyday systems:

- **Banking & FinTech** — a `BankAccount` class encapsulates `_balance` behind `deposit()`/`withdraw()`, while `SavingsAccount` and `CurrentAccount` inherit its shared logic and add their own interest or overdraft rules, exactly as built up in this unit.
- **UPI payment systems** — an abstract `PaymentMethod` guarantees every payment type — `UPIPayment`, `CardPayment`, `NetBankingPayment` — implements `process()`, and a single loop calls `payment.process(amount)` polymorphically across all of them without checking which one it is.
- **E-commerce** — a `Product` base class is extended by `ElectronicsProduct` and `GroceryProduct`, each adding its own fields (a warranty period, an expiry date) while both encapsulate their own stock count behind a `restock()` method.
- **Food delivery (Swiggy-style)** — a `DeliveryPartner` base class is extended by `BikePartner` and `CarPartner`; a dispatch system calling `partner.calculate_fee()` on a mixed list of both is polymorphism doing exactly the job it's meant for.
- **Railway booking (IRCTC-style)** — a `Passenger` base class extended by `SeniorCitizenPassenger` or `TatkalBooking`, each overriding its own fare calculation while reusing shared booking and cancellation methods.
- **Healthcare** — a `Patient` base class extended into `InpatientRecord` and `OutpatientRecord`, each adding visit-specific fields while a sensitive field like `_medical_history` stays encapsulated behind controlled access.
- **AI/ML pipelines** — an abstract `Model` base class might require every subclass to implement `train()` and `predict()`, so a pipeline can call those two methods polymorphically on whichever specific model — a decision tree, a neural network — happens to be plugged in.

A handful of mistakes are worth watching for deliberately across all four pillars while they're still new:

- Assuming a single leading underscore (`_balance`) actually blocks outside access — it only signals intent; Python never enforces it.
- Treating a double leading underscore as real security, forgetting that the mangled name (`_ClassName__name`) still works perfectly from outside.
- Designing a class that "feels" abstract but skips `ABC`/`@abstractmethod` entirely — nothing then stops an incomplete subclass from being instantiated, and a missing method only fails later, the first time it's actually called.
- Forgetting `super().__init__()` in a subclass's constructor — the superclass's attributes are simply never created, surfacing as a confusing `AttributeError` far from the actual mistake.
- Assuming `super()` always means "my direct parent" — once multiple inheritance is involved, it means "the next class in the MRO," which can be a sibling class instead.
- Writing a long `isinstance()` chain to handle several types by hand, when a shared method name and a single loop would let polymorphism do the same job automatically.

## Try it yourself

Do this in a Colab cell before moving on. Starting from the `BankAccount` class in this unit, build a `CurrentAccount(BankAccount)` whose `__init__` takes `owner_name`, `balance`, and `overdraft_limit`, calling `super().__init__()` first. Give it its own `withdraw(amount)` method that only allows the balance to go as low as `-overdraft_limit`. Then add a `@property` named `balance` (backed by `self._balance`) with a setter that raises `ValueError` if you try to set it directly below `-overdraft_limit`. Create one `CurrentAccount`, deposit and withdraw a few amounts, and confirm the property rejects an invalid direct assignment. Finally, write a small unrelated `EmailAlertMixin` class with a `send_alert()` method, mix it into a new `PremiumCurrentAccount(CurrentAccount, EmailAlertMixin)`, and print its `__mro__` before checking it against your own prediction.

---

### Key Terminology

- **Encapsulation** — bundling an object's data with the methods that act on it, while signalling which parts should stay internal.
- **Public attribute** — a plain attribute name with no underscore; freely readable and writable from outside the class.
- **Protected attribute (`_name`)** — a single leading underscore; a convention meaning "internal use only," not enforced by Python.
- **Private attribute (`__name`)** — a double leading underscore; triggers name mangling to `_ClassName__name`, mainly to avoid attribute collisions.
- **Name mangling** — Python's automatic rewriting of a double-underscore attribute name to include its defining class's name.
- **Property (`@property` / `@x.setter`)** — a method that reads or writes an attribute-like value while still using plain dot notation, allowing validation to run behind the scenes.
- **Abstraction** — exposing only the essential behaviour of an object while hiding its internal implementation.
- **Abstract Base Class (ABC)** — a class, built on Python's `abc` module, that defines a contract of methods its subclasses must implement.
- **`@abstractmethod`** — marks a method that every concrete subclass must override before it can be instantiated.
- **Inheritance** — a mechanism letting a new class automatically acquire an existing class's attributes and methods.
- **Superclass / subclass** — the existing class being built on (also parent/base class) and the new class built from it (also child/derived class).
- **`super()`** — a built-in function referring to the next class in the object's computed MRO, most often used to reuse a superclass's method.
- **Multi-level inheritance** — a chain of more than two classes, each extending the one directly above it.
- **Multiple inheritance** — one subclass extending more than one direct superclass at once, written `class C(A, B):`.
- **Method Resolution Order (MRO)** — the fixed, computed order Python searches through a class's superclasses when looking up a method or attribute.
- **Mixin** — a small class written only to add one focused, reusable behaviour through multiple inheritance, not meant to be instantiated on its own.
- **`isinstance()` / `issubclass()`** — built-in functions checking whether an object's class (or a class directly) appears anywhere in another class's MRO.
- **Polymorphism** — letting the same method call, or operator, behave differently depending on the actual object involved.
- **Method overriding** — a subclass defining its own version of a method its superclass already has, completely replacing it.
- **Extending** — a subclass overriding a method but calling `super()` to reuse the superclass's version and build on top of it, rather than discarding it.
- **Duck typing** — calling a method on an object based purely on whether that method exists, with no shared superclass required.

### Mastery Checkpoint

Before moving to Unit 4.3, check that you can answer these without looking back:

1. What is the actual difference between what a single leading underscore signals and what a double leading underscore does to an attribute's name, and does either one truly stop outside code from reaching it?
2. Why does `PaymentMethod()` raise a `TypeError` while `UPIPayment()` does not, given that `UPIPayment` inherits from `PaymentMethod`?
3. `MinorSavingsAccount.show_balance()` calls `super().show_balance()`. When `PremiumSavingsAccount` also needs a method it doesn't define itself, how does Python decide which class to search, and where can you check that order directly?
4. What is the key difference between method overriding and duck typing — specifically, does duck typing require any inheritance relationship at all?
5. A subclass's `__init__` never calls `super().__init__()`. What specific error appears later, and why does it only show up once some other code tries to use the missing data, rather than immediately?

### Summary

You now know how to protect an object's internal state with underscore naming conventions and enforce a real rule with `@property`, even though Python's encapsulation is convention rather than a locked door; how to describe a shared contract with `ABC` and `@abstractmethod` so an incomplete subclass simply cannot be instantiated; how to reuse an existing class through single, multi-level, and multiple inheritance, calling `super().__init__()` to avoid retyping a superclass's setup and reading `ClassName.__mro__` to know exactly which class Python will search first; and how one shared method call can behave correctly across a whole hierarchy of subclasses through method overriding, or across entirely unrelated classes through duck typing. From here, the next step is learning how to control precisely how your own objects print, compare, and combine with operators — starting with Unit 4.3, Special Methods & Dataclasses.

### Additional Resources

- [Python 3 Documentation — Inheritance (Tutorial)](https://docs.python.org/3/tutorial/classes.html#inheritance)
- [Python 3 Documentation — Multiple Inheritance](https://docs.python.org/3/tutorial/classes.html#multiple-inheritance)
- [Python 3 Documentation — `super()` built-in function](https://docs.python.org/3/library/functions.html#super)
- [Python 3 Documentation — `abc` — Abstract Base Classes](https://docs.python.org/3/library/abc.html)
- [Python 3 Documentation — `property()` built-in function](https://docs.python.org/3/library/functions.html#property)
- [W3Schools — Python Inheritance](https://www.w3schools.com/python/python_inheritance.asp)
- [W3Schools — Python Polymorphism](https://www.w3schools.com/python/python_polymorphism.asp)
