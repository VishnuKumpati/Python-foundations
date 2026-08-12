# Inheritance: Reusing Code Across Related Classes

Encapsulation made one class trustworthy. It decided which data outsiders could read, which data they could change, and under what rules. Everything stayed inside a single class.

Real applications are not built from one class. A social media app has ordinary users, creators who earn money, advertisers who spend it, and verified brand pages. These classes are not strangers to each other — a creator *is* a user who does a little more.

Encapsulation has no answer for that overlap. Inheritance does.

## The Need for Code Reuse

Two related classes written independently must repeat whatever they share. Below, `User` and `Creator` were written separately. Today the profile format was updated to show a verified tick. It was updated in only one of the two classes.

```python
class User:
    def __init__(self, name):
        self.name = name

    def show_profile(self):
        print(f"Profile: {self.name} ✔")


class Creator:
    def __init__(self, name, earnings):
        self.name = name
        self._earnings = earnings

    def show_profile(self):
        print(f"Profile: {self.name}")

    def show_earnings(self):
        print(f"Earnings: ₹{self._earnings}")


User("Rahul").show_profile()
Creator("Priya", 45000).show_profile()
```

**Output**

```text
Profile: Rahul ✔
Profile: Priya
```

One feature, two different results. Rahul's profile shows the tick and Priya's does not, because the same method existed in two places and only one was updated.

Nothing raised an error. The two classes simply drifted apart, and the app now behaves inconsistently for different account types.

This is why inheritance exists:

- **Shared behaviour is written once.** One copy of `show_profile()`, not two.
- **A change happens in one place.** Update it once and every related class updates with it.
- **A new account type costs almost nothing.** Adding `Advertiser` means writing only what makes an advertiser different.
- **The code matches reality.** A creator really is a kind of user, and the code should say so.

## Inheritance

Inheritance lets you build a new class on top of an existing one. The new class automatically gets the existing class's methods, so you do not write them a second time.

The shared part is written once. The new class adds only what is different.

The two classes play different roles, and each role has a name.

## The Parent and Child Relationship

The class holding the shared part is the **parent**. The class holding the difference is the **child**.

Which is which is decided by how general each class is:

- The **parent is the more general** class. `User` describes what is true of every account — it has a name and a profile.
- The **child is the more specific** class. `Creator` describes one particular kind of user, the kind that also earns money.

The general class is written first. The specific class is then built on top of it.

It never works the other way around. The specific class adds to the general behaviour, so that behaviour has to exist first.

The relationship also runs in one direction only:

- The child can use the parent's members.
- The parent knows nothing about the child and gains nothing from it.
- Two children of the same parent know nothing about each other.

```mermaid
flowchart TD
    P["User (parent)<br/>the general account"] -->|"child may use<br/>the parent's members"| C["Creator (child)<br/>a specific kind of account"]

    classDef parent fill:#EEF2FF,stroke:#6366F1,stroke-width:2px,color:#1E1B4B
    classDef child fill:#ECFDF5,stroke:#10B981,stroke-width:2px,color:#064E3B

    class P parent
    class C child
```

The same two classes are also called base and derived class, or superclass and subclass. Parent and child are used from here onward.

## Declaring a Child Class

A child class is declared by writing the parent's name in brackets after the child's name.

```python
class User:
    def __init__(self, name):
        self.name = name

    def show_profile(self):
        print(f"Profile: {self.name}")


class Creator(User):
    pass


creator1 = Creator("Priya")
creator1.show_profile()
```

**Output**

```text
Profile: Priya
```

`pass` means the class body is empty. `Creator` defines nothing of its own: no constructor, no method, no attribute.

`Creator("Priya")` still worked because the child used the parent's `__init__`, and `show_profile()` still printed because the child used the parent's method. The duplicated code from the earlier example is gone, replaced by the single word `User` in brackets.

## How Two Classes Can Be Related

Two classes can be related in two different ways, and only one of them is inheritance.

An **is-a relationship** exists when one class is a specific kind of another class. A creator is a kind of user, so `Creator` and `User` stand in an is-a relationship, written as `class Creator(User):`. Inheritance is the tool that builds this relationship. That is why inheritance itself is also called an is-a relation.

A **has-a relationship** exists when one class holds an object of another class. A user owns a wallet, but a wallet is not a kind of user. Inheritance would be wrong here, because `class Wallet(User)` would give a wallet a name and a profile page.

The user object stores a wallet object instead. Building a class this way is called **composition**.

```python
class Wallet:
    def __init__(self, balance):
        self.balance = balance


class User:
    def __init__(self, name):
        self.name = name
        self.wallet = Wallet(500)


user1 = User("Rahul")

print(user1.name)
print(user1.wallet.balance)
```

**Output**

```text
Rahul
500
```

There are no brackets after `class User`. It inherits from nothing. The line `self.wallet = Wallet(500)` creates a wallet and keeps it inside the user. `user1.wallet.balance` then reads through the user to the wallet it holds.

To choose between the two, complete this sentence about the classes: *a Creator is a User*. If that sentence is true, use inheritance. If the only true sentence is *a User has a Wallet*, use composition.

### isinstance() and issubclass()

`isinstance()` asks whether an object belongs to a class. `issubclass()` asks whether one class inherits from another.

```python
class User:
    pass


class Creator(User):
    pass


print(isinstance(Creator(), User))
print(isinstance(User(), Creator))
print(issubclass(Creator, User))
```

**Output**

```text
True
False
True
```

A `Creator` object counts as a `User`, because the is-a relationship holds. A `User` object does not count as a `Creator`, because that relationship runs in one direction only.

## What the Child Inherits and What It Does Not

A child does not receive a copy of everything the parent has.

| What the parent has | Does the child get it | Why |
|---|---|---|
| Methods | Yes | Methods belong to the class, so Python finds them there |
| Object data such as `self.name` | Only if the parent's `__init__` runs | The data is created while `__init__` runs. Nothing stores it in the class. |
| Protected data `_name` | Yes | A single underscore is only a naming habit, nothing more |
| Private data `__name` | No | Python renames it using the class where the line is written, so the child looks for a name that was never created |

So a parent marks data `_name` when children are meant to use it, and `__name` when they are not.

The second row is the one that catches people. A child that writes its own `__init__` replaces the parent's. The parent's data then never gets created, unless the child asks for it.

## super() function

`super()` is a built-in function that stands for the parent class. Writing it inside a child method is the same as saying *the class I inherited from*.

Write `super()`, a dot, then the parent member you want. `super().__init__(name)` runs the parent's constructor. `super().show_profile()` runs the parent's `show_profile()`.

It is used for three reasons.

- A child that writes its own `__init__` has no other way to get the parent's data created. Its own `__init__` replaces the parent's, so the first line of a child's `__init__` is normally `super().__init__(...)`.
- The child avoids repeating work the parent already does.
- The parent's name never appears inside the child. Rename `User` tomorrow and every `super()` call keeps working.

## Types of Inheritance

Python supports five types, named after the shape the classes form. All five are built from the same `class Child(Parent)` line.

### Single Inheritance

One child inherits from one parent.

```mermaid
flowchart TD
    A["Account"] --> B["SavingsAccount"]

    classDef box fill:#EEF2FF,stroke:#6366F1,stroke-width:2px,color:#1E1B4B
    class A,B box
```

**Where you see it:** a banking app. Every account holds a number, a holder name, and a balance. A savings account is an account that additionally earns interest, so it is written as `class SavingsAccount(Account):` and adds only the interest calculation.

### Multilevel Inheritance

A child becomes the parent of the next class, forming a chain. The last class can use members from every class above it.

```mermaid
flowchart TD
    A["Vehicle"] --> B["Car"] --> C["ElectricCar"]

    classDef box fill:#EEF2FF,stroke:#6366F1,stroke-width:2px,color:#1E1B4B
    class A,B,C box
```

**Where you see it:** a vehicle registration system. `Vehicle` holds the registration number, `Car` adds the number of seats, and `ElectricCar` adds battery capacity. An electric car still has the registration number defined two levels above it.

### Hierarchical Inheritance

Two or more children inherit from the same parent. The children are unrelated to each other.

```mermaid
flowchart TD
    A["Payment"] --> B["UpiPayment"]
    A --> C["CardPayment"]

    classDef box fill:#EEF2FF,stroke:#6366F1,stroke-width:2px,color:#1E1B4B
    class A,B,C box
```

**Where you see it:** the checkout screen of any Indian shopping app. Every payment has an amount, a date, and a status. UPI payments add a VPA, card payments add the last four digits. Both are payments; neither is a kind of the other.

### Multiple Inheritance

One child inherits from more than one parent at once, written as `class Smartphone(Camera, Phone):`.

```mermaid
flowchart TD
    A["Camera"] --> C["Smartphone"]
    B["Phone"] --> C

    classDef box fill:#EEF2FF,stroke:#6366F1,stroke-width:2px,color:#1E1B4B
    class A,B,C box
```

**Where you see it:** the device in your hand. A smartphone genuinely is a camera and a phone at the same time, so it takes members from both.

When both parents define the same method, Python searches them left to right and uses the first match. `Camera` is listed first, so `Camera` wins above. That search order is called the **method resolution order**, shortened to MRO.

### Hybrid Inheritance

Two or more of the above types combined in one design.

```mermaid
flowchart TD
    A["Person"] --> B["Student"]
    A --> C["Staff"]
    B --> D["ResearchScholar"]
    C --> D

    classDef box fill:#EEF2FF,stroke:#6366F1,stroke-width:2px,color:#1E1B4B
    class A,B,C,D box
```

**Where you see it:** a university records system. `Student` and `Staff` are both children of `Person`, which is hierarchical. A research scholar who also teaches is both a student and a staff member, so `class ResearchScholar(Student, Staff):` is multiple inheritance. The two shapes together make the design hybrid, and Python uses the MRO to decide which `Person` member applies.

## Practice Exercises

Build a food delivery app's account hierarchy, one step at a time. Each task continues the file from the task before it.

1. **Declare a child that adds nothing.** Write `Account` with `__init__(self, name)` and a `show()` method that prints the name. Then write `class Restaurant(Account): pass`. Create a `Restaurant` object and call `show()`. It works even though `Restaurant` is empty.

2. **Give the child a method of its own.** Add a method `show_type()` to `Restaurant` that prints `This is a restaurant account.` Call `show()` and `show_type()` on the same object. One method came from the parent, the other from the child.

3. **Watch the constructor break.** Give `Restaurant` its own `__init__` that accepts only `rating` and sets `self._rating`, with no `super()` call. Create the object and call `show()`. Read the `AttributeError` and write down which attribute went missing and why.

4. **Repair it with super().** Change that `__init__` to accept `name` and `rating`, with `super().__init__(name)` as its first line. Call `show()` again and confirm it works.

5. **Name the shape.** Add `class DeliveryPartner(Account): pass`. You now have one parent with two children — name which type of inheritance that is. Then print `isinstance()` and `issubclass()` results to prove a `DeliveryPartner` object counts as an `Account`.

---

Every child in this chapter did one of two things: it used the parent's methods exactly as they were, or it added new methods of its own.

A third possibility was never tried. What if a child needs its own version of a method the parent already has? A `Creator` profile line, for example, must look different from a plain `User` profile line.

The method name would stay the same. The behaviour would change depending on which object is calling it.

That is the third pillar. The next chapter covers **Polymorphism**.

---

## Reference Links

- [Python Official Docs — Inheritance](https://docs.python.org/3/tutorial/classes.html#inheritance)
- [Python Official Docs — Multiple Inheritance](https://docs.python.org/3/tutorial/classes.html#multiple-inheritance)
- [Python Official Docs — `super()`](https://docs.python.org/3/library/functions.html#super)
- [Python Official Docs — Method Resolution Order](https://docs.python.org/3/glossary.html#term-method-resolution-order)
- [W3Schools — Python Inheritance](https://www.w3schools.com/python/python_inheritance.asp)
- [GeeksforGeeks — Types of Inheritance in Python](https://www.geeksforgeeks.org/python/types-of-inheritance-python/)
