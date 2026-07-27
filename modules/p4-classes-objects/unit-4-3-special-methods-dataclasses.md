# Special Methods & Dataclasses

Unit 4.2 covered the four pillars of OOP — encapsulation, abstraction, inheritance, polymorphism. But a plain class you write yourself still behaves oddly next to Python's own built-in types: print an object and you get a cryptic memory address, not a readable summary; try to add two of them and Python has no idea what that should mean; and every class you've written so far has needed a hand-written `__init__` with every attribute typed out by hand. None of this means your classes are broken — it means Python is waiting for you to answer a few specific questions it can never guess on its own.

This unit answers those questions. You'll meet **dunder methods** — the small, specially named methods Python calls automatically behind the scenes whenever you use `print()`, `==`, or `+` on an object — and learn exactly which one to write for which behaviour. Then you'll meet a shortcut: the `@dataclass` decorator, which generates the most common of these methods for you, so a class that is mostly just a bundle of attributes doesn't need dozens of hand-typed lines to behave sensibly.

By the time you finish, you'll be able to make your own classes print, compare, and combine exactly the way you intend — and you'll know when to stop writing that boilerplate by hand entirely. This is also the final unit of Module IV: once you're done here, you'll have covered everything from a plain class and its objects, through the four pillars, to this last layer of polish — a complete picture of object-oriented Python.

---

## Why `print()` and `+` don't just work on your own classes

Try this in a Colab cell with any class you've written so far — say a `Wallet` that stores a UPI user's `owner_name` and `balance`:

```python
class Wallet:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self.balance = balance

wallet_1 = Wallet("Rohit", 500.0)
print(wallet_1)
```

```
<__main__.Wallet object at 0x7f2a4c1b3d90>
```

Nothing here is broken. `object` — the base class every class in Python ultimately extends, whether you write it explicitly or not — has to have *some* default behaviour for printing an object it knows nothing about, and a memory address is the only honest thing it can show. Try `wallet_1 == Wallet("Rohit", 500.0)` next, and you get `False`, even though the data is identical — because `object`'s default `==` checks whether two names point to the *literal same object in memory*, not whether their data matches. Try `wallet_1 + wallet_1`, hoping to combine two balances the way you'd add two numbers, and Python refuses outright with a `TypeError`.

Before reading on, predict what all three of these would look like for a plain `int` or `str` instead of a `Wallet` — `print(500)`, `500 == 500`, `500 + 500` all behave exactly as you'd expect, because Python's *own* built-in types already answer these questions for themselves. Your job in this unit is to give your own classes the same courtesy.

## Dunder methods: hooks Python calls for you

A **dunder method** (short for "double underscore," also called a **magic method** or **special method**) is a method whose name begins and ends with two underscores — `__init__`, `__str__`, `__repr__`, `__eq__`, and `__add__` are all examples. You've already used one without necessarily naming it: writing `Wallet("Rohit", 500.0)` triggers `__init__` automatically — you never type `wallet.__init__(...)` yourself. Every other dunder method works the same way.

Think of dunder methods as hooks: sockets Python has already wired up throughout the language, waiting for you to plug your own logic into them. `print(obj)` is Python quietly asking your class "how do you want to be displayed?" `obj1 == obj2` is Python asking "what makes two of you equal?" `obj1 + obj2` is Python asking "what does combining two of you even mean?" If you never plug anything into that socket, Python falls back to `object`'s generic default — which is exactly the memory address, identity check, and `TypeError` you just saw above.

**You never call a dunder method by its own name — you write ordinary Python (`print()`, `==`, `+`) and Python calls the matching dunder method for you, behind the scenes.** Say out loud, in one sentence, what `wallet_1 + wallet_2` is actually doing underneath — it's calling `wallet_1.__add__(wallet_2)`, whether you ever type that or not.

## Two different audiences: `__str__` versus `__repr__`

Two separate dunder methods control how an object gets displayed, and they exist because printing an object serves two genuinely different audiences.

- **`__str__`** is called by `print()`, by `str()`, and by f-strings, whenever it's defined. Its job is a **readable, human-facing** description — something a non-technical end user would find friendly.
- **`__repr__`** is called by `repr()`, and by an interactive shell echoing a value back at you. Its job is a **precise, developer-facing** description, ideally one that looks like the code needed to recreate the object.

```python
class Wallet:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self.balance = balance

    def __repr__(self):
        return f"Wallet(owner_name={self.owner_name!r}, balance={self.balance})"

    def __str__(self):
        return f"{self.owner_name}'s wallet: Rs. {self.balance:.2f}"

wallet_1 = Wallet("Rohit", 500.0)
print(wallet_1)
print(repr(wallet_1))
```

```
Rohit's wallet: Rs. 500.00
Wallet(owner_name='Rohit', balance=500.0)
```

`__repr__` uses the `!r` conversion inside the f-string, which wraps a string value in quotes — `owner_name={self.owner_name!r}` shows `owner_name='Rohit'`, exactly as you'd type it into code yourself. `__str__` formats `balance` to two decimal places instead, because this value represents money and a human reader expects `500.00`, not `500.0`.

| | `__str__` | `__repr__` |
|---|---|---|
| Audience | End user | Developer, logs, debugging |
| Called by | `print()`, `str()`, f-strings | `repr()`, an interactive shell |
| Goal | Friendly, readable | Unambiguous, code-like |
| If missing | Falls back to `__repr__` | Falls back to `object`'s memory address |

That fallback matters: `object`'s own default `__str__` is written to simply call `self.__repr__()`. So a class that defines `__repr__` but never defines `__str__` still prints something useful — `print()` silently falls back to whatever `__repr__` produces. Only when *both* are defined does `print()` prefer the friendlier `__str__`, while `repr()` still shows the developer-facing version when asked directly. Before checking, predict what `print(wallet_1)` would show if only `__repr__` existed above — it would show the exact same `Wallet(owner_name='Rohit', balance=500.0)` line that `repr()` shows, because there's nothing else for `print()` to fall back on.

**A dunder method meant to produce text must actually `return` a string — forgetting the `return` doesn't silently fail; Python raises `TypeError: __str__ returned non-string (type NoneType)` the moment the value is used.**

## Comparing by data, not by memory address: `__eq__`

`==` between two custom objects, with no dunder method involved, checks **identity**: are these literally the same object sitting at the same place in memory? Two separately created objects holding identical attribute values still compare `False`, because they're two different objects, even though their data matches exactly — the same gap you saw at the very start of this unit.

**`__eq__`** is the dunder method called by `==`. Defining it lets you redefine what "equal" means for your class — typically, "same type, and every attribute matches":

```python
class Wallet:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self.balance = balance

    def __eq__(self, other):
        return (isinstance(other, Wallet)
                and self.owner_name == other.owner_name
                and self.balance == other.balance)

wallet_1 = Wallet("Rohit", 500.0)
wallet_2 = Wallet("Rohit", 500.0)
print(wallet_1 == wallet_2)
print(wallet_1 is wallet_2)
```

```
True
False
```

Two details matter here. First, `other` — the second value being compared — arrives as an ordinary parameter, with no guarantee it's even a `Wallet` at all; writing `wallet_1 == 5` is perfectly legal Python, so `__eq__` must handle it gracefully rather than crash. That's exactly why `isinstance(other, Wallet)` is checked *first*: `and` evaluates left to right and stops the moment one side is `False`, so if `other` isn't a `Wallet`, the expression returns `False` immediately without ever touching `other.owner_name` — an attribute that might not even exist on `other`, and would otherwise raise an `AttributeError` like the one you met in Unit 4.2. Second, once `isinstance` confirms the type, every attribute you care about is compared with plain `==`, and only if *all* of them match does the whole expression become `True`.

This is exactly how a healthcare system would spot a duplicate patient record submitted from two different hospital counters — two `PatientRecord` objects arrive as two separate objects in memory, but a custom `__eq__` comparing patient ID, name, and date of birth can still recognise them as "the same patient," where the default identity check never would.

**Without `__eq__`, two objects holding identical data still compare `False` — and that silent gap is one of the most common surprises for anyone testing their own classes for the first time.**

## Operator overloading: teaching `+`, `<`, and friends what to do

**Operator overloading** is the general technique you've just been using one operator at a time: defining a dunder method so a standard Python operator does something meaningful for your own class, instead of raising an error. `__eq__` is operator overloading for `==`; `__add__` is the same idea applied to `+`.

Python does not support what languages like Java or C++ call "method overloading" — writing several methods with the *same name* that differ only by parameter type, resolved automatically. Python has no equivalent mechanism. Instead, each operator maps to exactly *one* dunder method slot per class — `+` always calls `__add__`, `<` always calls `__lt__` — and whatever logic you write inside that one slot decides how to handle the situation.

```python
class Wallet:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self.balance = balance

    def __str__(self):
        return f"{self.owner_name}'s wallet: Rs. {self.balance:.2f}"

    def __add__(self, other):
        combined_balance = self.balance + other.balance
        return Wallet(f"{self.owner_name} + {other.owner_name}", combined_balance)

wallet_1 = Wallet("Rohit", 500.0)
cashback_wallet = Wallet("Rohit-Cashback", 45.50)

total_wallet = wallet_1 + cashback_wallet
print(total_wallet)
```

```
Rohit + Rohit-Cashback's wallet: Rs. 545.50
```

Writing `wallet_1 + cashback_wallet` calls `wallet_1.__add__(cashback_wallet)` implicitly — `self` is the left-hand operand, `other` is the right-hand one. Notice `self.balance + other.balance` inside the method body is plain numeric addition on two `float` values; nothing there is being overloaded — the overloading is the outer `__add__` definition itself. The method builds and returns a **brand-new** `Wallet` rather than modifying `self` in place, matching how `+` behaves for ordinary numbers: `3 + 4` never changes the value `3`. Without `__add__` defined at all, `wallet_1 + cashback_wallet` raises `TypeError: unsupported operand type(s) for +: 'Wallet' and 'Wallet'` — a real UPI wallet-recharge feature relies on exactly this pattern to combine a main balance and a cashback balance in one clean expression.

The same idea extends to comparison operators. Suppose a bank wants to rank a list of `Wallet` objects by balance for a report:

```python
class Wallet:
    def __init__(self, owner_name, balance):
        self.owner_name = owner_name
        self.balance = balance

    def __repr__(self):
        return f"Wallet({self.owner_name!r}, {self.balance})"

    def __lt__(self, other):
        return self.balance < other.balance

wallets = [Wallet("Rohit", 500.0), Wallet("Ananya", 1200.0), Wallet("Priya", 90.0)]
print(sorted(wallets))
```

```
[Wallet('Priya', 90.0), Wallet('Rohit', 500.0), Wallet('Ananya', 1200.0)]
```

`sorted()` needs some way to decide which of two `Wallet` objects comes first, and `__lt__` (the dunder method behind `<`) is exactly what it calls to find out. Before checking, predict what `sorted(wallets, reverse=True)` would produce — the same three wallets, but ordered from the highest balance down to the lowest.

| Operator | Dunder method called |
|---|---|
| `==` | `__eq__` |
| `<` | `__lt__` |
| `+` | `__add__` |
| `print()` / `str()` | `__str__` |
| `repr()` | `__repr__` |

One advanced detail worth naming without dwelling on: a dunder method can return the special value `NotImplemented` to tell Python "I don't know how to combine these two types," letting Python try other options before giving up. This matters for library authors supporting many mixed-type operations; it's rarely needed for the straightforward classes you're writing here, so treat it as a name to recognise rather than a technique to master right now.

## Dataclasses: when a class is mostly just data

Many classes in real projects are little more than a labelled bundle of attributes with almost no custom behaviour — a product listing, a configuration object, a confirmed railway booking. Writing `__init__`, `__repr__`, and `__eq__` by hand for every such class is repetitive, and repetition is exactly where bugs hide: add a new attribute to `__init__` but forget to add it to a hand-written `__eq__`, and equality silently stops checking that attribute.

A **dataclass** is an ordinary class, decorated with `@dataclass` from Python's built-in `dataclasses` module, that has `__init__`, `__repr__`, and `__eq__` generated automatically from a short list of type-hinted attributes, instead of you writing them by hand:

```python
from dataclasses import dataclass

@dataclass
class Wallet:
    owner_name: str
    balance: float

wallet_1 = Wallet("Rohit", 500.0)
wallet_2 = Wallet("Rohit", 500.0)

print(wallet_1)
print(wallet_1 == wallet_2)
```

```
Wallet(owner_name='Rohit', balance=500.0)
True
```

No `__init__`, `__repr__`, or `__eq__` appears anywhere in that class body — `@dataclass` read the two type-hinted attributes, called **fields**, and generated all three from them. `owner_name: str` and `balance: float` each become one parameter, in the same order, of the generated `__init__`.

An e-commerce cart shows the same pattern with a different shape of data:

```python
from dataclasses import dataclass

@dataclass
class CartItem:
    product_name: str
    price: float
    quantity: int = 1

item = CartItem("Notebook", 149.0)
print(item)
```

```
CartItem(product_name='Notebook', price=149.0, quantity=1)
```

`quantity: int = 1` gives that field a **default value**, making it optional — leave it out of the call, as above, and it falls back to `1`. **Every field with a default must be declared after every field without one**, because the generated `__init__` places parameters in exactly the order the fields were declared. Try writing `price: float = 0.0` *before* `product_name: str` with no default, and Python raises a `TypeError` at the moment the class itself is defined — before your program even runs — because a required parameter can never come after an optional one in a valid function signature. This same shape shows up constantly in AI/ML work: a model's hyperparameters — `batch_size`, `learning_rate`, `epochs` — are almost always modelled as a dataclass, since a configuration is pure data with no behaviour of its own.

Two limits matter just as much as what `@dataclass` generates. First, a plain attribute with no type hint is not recognised as a field at all — it's silently left out of the generated `__init__`, `__repr__`, and `__eq__`. Second, and easy to forget: **`@dataclass` never generates `__str__`.** If you print a dataclass instance without writing `__str__` yourself, you see exactly what `__repr__` produces — purely because of the same fallback rule you met earlier in this unit. A dataclass is still an ordinary class underneath the decorator: you can add hand-written methods like `__str__` or `__add__` to it exactly as you would to any class, and it still supports inheritance the same way Unit 4.2 covered.

| | Hand-written class | `@dataclass` |
|---|---|---|
| `__init__` | Written by hand | Generated from type-hinted fields |
| `__repr__` / `__eq__` | Written by hand, if wanted at all | Generated automatically |
| `__str__` | Written by hand, if wanted | **Never generated** — falls back to `__repr__` |
| Best suited for | Classes with real behaviour, validation, invariants | Classes that are mostly data — configs, records |
| Boilerplate | More typing, more places for a bug to hide | A short, type-hinted field list |

Putting it together, `Wallet` itself is mostly data (`owner_name`, `balance`) with a little genuine custom behaviour layered on top (`__str__`, `__add__`). `@dataclass` can generate the purely data-driven parts while you keep writing the parts that are genuinely yours to decide:

```python
from dataclasses import dataclass

@dataclass
class Wallet:
    owner_name: str
    balance: float

    def __str__(self):
        return f"{self.owner_name}'s wallet: Rs. {self.balance:.2f}"

    def __add__(self, other):
        return Wallet(f"{self.owner_name} + {other.owner_name}", self.balance + other.balance)

wallet_1 = Wallet("Rohit", 500.0)
cashback_wallet = Wallet("Rohit-Cashback", 45.50)
print(wallet_1 + cashback_wallet)
```

```
Rohit + Rohit-Cashback's wallet: Rs. 545.50
```

**Reach for `@dataclass` when a class is mostly a bundle of attributes with little custom logic; keep writing a hand-written class the moment real validation or behaviour is genuinely part of what makes that class what it is.**

## How every library you'll use next builds on exactly this

Nothing in this unit is classroom-only ceremony. The moment you start importing real libraries — later in this programme, that will mean things like `pandas` for data analysis or `requests` for talking to a web API — you'll find their objects behaving exactly the way this unit taught you to make your own objects behave: a `pandas` DataFrame prints as a neat table because someone implemented `__repr__` for it; comparing two `datetime` objects with `==` or `<` works because `__eq__` and `__lt__` are implemented underneath; adding two NumPy arrays with `+` combines them element-by-element because `__add__` was overloaded to mean exactly that for that type. Every well-designed Python package leans on this same small set of hooks — what you've learned here isn't a beginner-only trick, it's the actual mechanism the entire ecosystem is built on.

A short list of mistakes worth watching for deliberately while this is still new:

- Confusing `__str__` and `__repr__`, then being surprised that `print()` and `repr()` show different things once both are defined.
- Forgetting to `return` a value from a dunder method, producing a `TypeError` or a silent `None`.
- Writing `__eq__` without an `isinstance` guard, then crashing the moment someone compares your object to an unrelated type.
- Forcing a class with genuine validation logic or custom behaviour into `@dataclass` just to save typing, when a hand-written class communicates the design better.
- Leaving a dataclass field without a type hint, so it silently vanishes from the generated `__init__`, `__repr__`, and `__eq__`.
- Placing a required field after a defaulted one inside a dataclass, triggering a `TypeError` before the program even runs.

## Try it yourself

Do this in a Colab cell before moving on. Write a plain `Wallet` class with only `__init__`, confirm `print()` shows a memory address and `==` compares by identity. Then add `__repr__`, confirm `print()` now falls back to it usefully. Add `__str__` and confirm `print()` now prefers the friendlier line instead. Add `__eq__`, guarded with `isinstance`, and confirm two separately created wallets with identical data now compare `True` while `is` still says `False`. Add `__add__` and combine two wallets with `+`. Finally, rewrite the whole class as a `@dataclass`, keeping only `__str__` and `__add__` hand-written, and confirm every printed, compared, and combined result is unchanged.

---

### Key Terminology

- **Dunder / magic / special method** — a method whose name starts and ends with `__`, called implicitly by Python in response to built-in syntax, such as `print()`, `==`, or `+`.
- **`__str__`** — called by `print()`, `str()`, and f-strings; produces a readable, human-facing description.
- **`__repr__`** — called by `repr()` and an interactive shell; produces a precise, developer-facing description.
- **`__eq__`** — called by `==`; lets two objects compare equal based on data instead of memory identity.
- **`__add__`** — called by `+` when the left-hand operand is an instance of your class.
- **`__lt__`** — called by `<`, and used by `sorted()` to decide ordering between two instances.
- **Operator overloading** — defining a dunder method so a built-in operator behaves meaningfully for a custom class.
- **`@dataclass`** — a decorator from Python's `dataclasses` module that generates `__init__`, `__repr__`, and `__eq__` from a class's type-hinted fields.
- **Field** — a type-hinted attribute declared inside a dataclass; becomes one parameter of the generated `__init__`.
- **`NotImplemented`** — a special value a dunder method can return to tell Python it doesn't know how to handle a given combination of types.

### Mastery Checkpoint

Before moving to Unit 5.1, check that you can answer these without looking back:

1. Why does `print(my_object)` show a bare memory address for a plain class, and what dunder method fixes it?
2. If a class defines `__repr__` but not `__str__`, what does `print()` show, and why?
3. Why must `__eq__` check `isinstance(other, ClassName)` before comparing any attributes of `other`?
4. What three methods does `@dataclass` generate for you, and which common method does it never generate?
5. In a dataclass, why must every field with a default value come after every field without one?

### Summary

You now know why a plain class's default printing, comparing, and adding behaviour looks the way it does, and how dunder methods let you redefine every one of those behaviours for your own classes: `__str__` and `__repr__` for two different printing audiences, `__eq__` for comparing by data instead of identity, and `__add__`/`__lt__` as two examples of operator overloading in general. You've also seen how `@dataclass` generates `__init__`, `__repr__`, and `__eq__` automatically from a short, type-hinted field list — saving you from repetitive, bug-prone boilerplate — while still leaving room for hand-written methods like `__str__` on top of it. That closes out Module IV: you've now gone from a plain class and its objects, through encapsulation, abstraction, inheritance, and polymorphism, to this final layer of polish that makes your own objects behave as naturally as Python's built-in types. From here, the next step is a genuinely new topic — Unit 5.1, File Handling — where your programs start reading from, and writing to, real files on disk instead of holding everything only in memory.

### Additional Resources

- [Python Data Model — Special Method Names](https://docs.python.org/3/reference/datamodel.html#special-method-names)
- [Python 3 Documentation — `dataclasses` Module](https://docs.python.org/3/library/dataclasses.html)
- [Python 3 Documentation — `object.__repr__` and `object.__str__`](https://docs.python.org/3/reference/datamodel.html#object.__repr__)
- [Python 3 Documentation — Built-in Exceptions (`TypeError`)](https://docs.python.org/3/library/exceptions.html#TypeError)
- [W3Schools — Python Classes and Objects](https://www.w3schools.com/python/python_classes.asp)
- [W3Schools — Python Operator Overloading](https://www.w3schools.com/python/python_polymorphism.asp)
