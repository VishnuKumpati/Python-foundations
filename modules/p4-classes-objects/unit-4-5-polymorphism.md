# Polymorphism: One Name, Many Behaviours

Every child class in the previous chapter did one of two things. It used the parent's methods exactly as they were, or it added new methods of its own.

A third possibility was left open. A `Creator` needs a profile line that shows earnings, and the parent's `show_profile()` knows nothing about earnings. The child does not want a new method name — `show_profile()` is the right name. It wants its own version of that same name.

The word for one name behaving differently in different places is **polymorphism**. It comes from the Greek *poly*, meaning many, and *morph*, meaning form. This chapter starts with the situation above. Step by step it widens, until the same name works across classes that are not related at all.

## Method Overriding

So the child writes its own `show_profile()`. The parent already has a method with that name. The child's version now takes over for every `Creator` object.

This is called **method overriding**.

```python
class User:
    def __init__(self, name):
        self.name = name

    def show_profile(self):
        print(f"Profile: {self.name}")


class Creator(User):
    def __init__(self, name, earnings):
        super().__init__(name)
        self._earnings = earnings

    def show_profile(self):
        print(f"Creator: {self.name} | Earnings: ₹{self._earnings}")


rahul = User("Rahul")
priya = Creator("Priya", 45000)

rahul.show_profile()
priya.show_profile()
```

**Output**

```text
Profile: Rahul
Creator: Priya | Earnings: ₹45000
```

Both lines called the same method name and got different results. Python chooses the version by looking at the class each object was made from. It stops at the first match it finds.

`rahul` was made from `User`, so Python ran `User`'s `show_profile()`. `priya` was made from `Creator`. That class has a `show_profile()` of its own, so Python ran it and never went up to `User`. Nothing was taken away from `User` in the process. Its method is intact, which is why `rahul` still prints the plain profile line.

Overriding therefore needs two classes with inheritance between them. The parent writes the method, and the child writes it again under the same name.

## Duck Typing

Python does not check which class an object belongs to before calling a method on it. It looks for a method of the right name on that object. If the method is there, Python calls it. If it is not, the program stops at that line.

This is called **duck typing**, from the old saying: if it walks like a duck and quacks like a duck, treat it as a duck. Python does not ask *what class are you*, it asks *do you have the method I need*.

So two classes with no parent in common can still be used together. A loop calling `show_profile()` works on anything that has one. The price is that a missing method is found only when that line runs.

Duck typing let one name work across unrelated classes. Inside a single class, the same name cannot be used twice.

## Method Overloading and Why Python Does Not Support It

Other languages let one name be used twice inside a single class with different parameters — `post(text)` and `post(text, tag)`. This is **method overloading**, and Python does not have it.

A `def` inside a class creates a name and attaches a function to it. A second `def post` reuses that name, so it replaces the first function completely. Those languages pick a version by matching the arguments before the program runs. Python matches on the name alone while the program runs. It never looks at the parameters, so nothing is left that could tell two versions apart. Calling the one-parameter version afterwards raises a `TypeError`.

The need behind overloading is real, though. One method often has to handle different numbers of arguments, and Python answers that a different way.

## Accepting Any Number of Arguments with *args and **kwargs

Two special parameter forms let one method accept as many arguments as the caller sends.

- `*args` collects any number of extra positional arguments into a tuple.
- `**kwargs` collects any number of extra keyword arguments into a dictionary.

The names `args` and `kwargs` are only a convention. The `*` and `**` are what matter.

```python
class Creator:
    def post(self, text, *tags):
        print(f"Posted: {text}")
        for tag in tags:
            print(f"  #{tag}")

    def update_profile(self, **details):
        for key, value in details.items():
            print(f"{key}: {value}")


priya = Creator()
priya.post("Hello")
priya.post("Hello", "python", "oop")
priya.update_profile(city="Chennai", language="Tamil")
```

**Output**

```text
Posted: Hello
Posted: Hello
  #python
  #oop
city: Chennai
language: Tamil
```

`post()` was called with one argument and then with three, and one definition served both. Inside the method, `tags` was an empty tuple the first time and held two items the second time.

`update_profile()` was called with named arguments it was never told to expect. Inside the method, `details` held exactly what was passed, as a dictionary.

This is what replaces method overloading in Python. Instead of several definitions chosen by argument count, there is one definition that inspects what it received.

## Practice Exercises

Build a payment screen for a shopping app, one step at a time. Each task continues the file from the task before it.

1. **Two classes, one method name.** Write `Payment` with `__init__(self, amount)` and a `show()` method that prints the amount. Then write `class UpiPayment(Payment)` with its own `show()` that prints the amount and the words `paid by UPI`. Create one object of each, call `show()` on both, and confirm the two lines differ.

2. **A third version of the same name.** Add `CardPayment(Payment)` with its own `show()` printing the amount and `paid by card`. Call `show()` on all three objects.

3. **One call, three classes.** Put the three objects into a list and loop over it calling `show()`. Three different lines come from a single call, with no `if` anywhere in the loop.

4. **Drop the parent.** Write a class `Cash` that does not inherit from `Payment` at all, but has its own `show()`. Add a `Cash` object to the same list and run the loop again. It still works, because the loop only ever needed a method called `show()`.

5. **Watch overloading fail, then fix it.** Add two `refund()` methods to `Payment` — one taking only `self`, one taking `self` and `reason`. Call the no-argument version and read the `TypeError`. Then replace both with a single `refund(self, *reason)` that prints the reason when one is given and `No reason given` when none is.

---

Every class in this chapter had to be written out in full before it could be used. A parent gave children something to inherit. But nothing stopped a child from skipping a method the rest of the program depended on. Nothing in `User` said that every account type *must* provide its own `show_profile()`.

Declaring what a child is obliged to provide, while hiding how it is done, is the last pillar.

The next chapter covers **Abstraction**.

---

## Reference Links

- [Python Official Docs — `super()`](https://docs.python.org/3/library/functions.html#super)
- [Python Official Docs — Arbitrary Argument Lists](https://docs.python.org/3/tutorial/controlflow.html#arbitrary-argument-lists)
- [W3Schools — Python Polymorphism](https://www.w3schools.com/python/python_polymorphism.asp)
- [GeeksforGeeks — Polymorphism in Python](https://www.geeksforgeeks.org/python/polymorphism-in-python/)
- [GeeksforGeeks — Method Overloading vs Method Overriding](https://www.geeksforgeeks.org/difference-between-method-overloading-and-method-overriding-in-python/)
