# Putting the Four Pillars Together

Over the last four chapters, we built one small platform, one pillar at a time. Encapsulation protected a password. Inheritance let three roles share one parent. Polymorphism let a single call mean three different things. Abstraction made sure that call was always safe to make.

None of these were four separate systems bolted together. They were four decisions, made about the exact same handful of classes — `User`, `Student`, `Teacher`, `Admin`. This chapter steps back and looks at all four at once, in one place.

---

## One Class, Four Decisions

Here is the platform's core, complete, with every pillar still doing exactly the job it was built to do.

```python
class User:
    def __init__(self, name, email, password=None):
        self.name = name
        self.email = email
        self.__password = password              # Encapsulation: password is private

    def login(self):
        print(f"{self.name} logged in.")

    def logout(self):
        print(f"{self.name} logged out.")

    def get_password(self):                     # Encapsulation: controlled read access
        return self.__password

    def set_password(self, new_password):       # Encapsulation: controlled write access
        self.__password = new_password

    def perform_duty(self):                     # Abstraction: a required promise
        raise NotImplementedError(
            f"{type(self).__name__} must implement perform_duty()."
        )


class Student(User):                            # Inheritance: reuses User's code

    def __init__(self, name, email, student_id):
        super().__init__(name, email)
        self.student_id = student_id

    def enroll_course(self, course):
        print(f"{self.name} enrolled in {course}.")

    def perform_duty(self):
        print(f"{self.name} is attending classes and completing assignments.")


class Admin(User):                              # Inheritance: reuses User's code

    def create_user(self):
        print(f"{self.name} created a new user.")

    def login(self):                            # Polymorphism: overrides User.login()
        super().login()
        print("Security check passed for admin access.")

    def perform_duty(self):
        print(f"{self.name} is managing platform accounts.")
```

Four comments, four pillars, and none of them fighting for space. `__password` stays private and reachable only through its own methods — that's encapsulation, untouched since the chapter that built it. `Student` and `Admin` never rewrote `name`, `email`, `login()`, or `logout()` — that's inheritance, still doing exactly what it did when we introduced it. `Admin.login()` runs its own version and still reaches back for `User.login()` through `super()` — that's polymorphism. And every class here is contractually guaranteed to have a working `perform_duty()`, not just a hoped-for one — that's abstraction.

---

## One Loop, Four Pillars at Work

```python
accounts = [
    Student("Rahul", "rahul@example.com", "S101"),
    Admin("Priya", "priya@example.com"),
]

for account in accounts:
    account.login()
    account.perform_duty()
```

```text
Rahul logged in.
Rahul is attending classes and completing assignments.
Priya logged in.
Security check passed for admin access.
Priya is managing platform accounts.
```

Every single line of that loop leans on all four pillars simultaneously, whether or not we notice it:

- `account.login()` runs different code for `Student` and `Admin` — **polymorphism** — and it's only possible to write one shared line like this because both classes **inherit** from `User`.
- `account.perform_duty()` is guaranteed to exist and do something meaningful, for any class we ever plug into that list — **abstraction** — rather than being a hope resting on careful typing.
- Neither line ever touches `__password` directly, and never could — **encapsulation** is still quietly governing that piece of data underneath everything else happening here.

Four pillars, two lines, zero visible seams between them.

---

## A Mental Model: The Four Pillars as a Car

Code examples fade; a physical analogy tends to stick. Think of any well-built class as a car.

- **Encapsulation** is the engine under the hood. You never reach in and rewire it yourself — you press pedals, and the car decides how to translate that safely into motion.
- **Abstraction** is the pedals and the steering wheel themselves. Every car promises an accelerator, a brake, and a wheel that turns it — a guaranteed interface, regardless of what's underneath.
- **Inheritance** is the shared chassis a "SportsCar" and a "Truck" are both built on — the same frame, wheels, and engine mounts reused, with each variant adding only what makes it different.
- **Polymorphism** is pressing that same accelerator pedal in a electric car versus a gas car. The driver does the exact same thing either way; what happens underneath is entirely different.

You don't design a car by inventing "encapsulation" or "abstraction" as separate blueprints. You design one good car, and these four ideas describe different things you got right about it.

---

## The Four Pillars at a Glance

| Pillar | Core Question | Mechanism | Example Here |
|---|---|---|---|
| Encapsulation | Who is allowed to change this data, and how? | Private attributes, getters, setters | `__password`, `get_password()`, `set_password()` |
| Inheritance | How do related classes share one implementation? | A child class built from a parent | `Student(User)`, `Admin(User)` |
| Polymorphism | How does the same call behave differently across objects? | Method overriding, duck typing | `Admin.login()` vs. `Student.login()` |
| Abstraction | Is the promise a class makes actually kept? | Required methods, `NotImplementedError` / `abc` | `perform_duty()` |

---

## Hands-On Practice

1. **Annotate the loop.** Take the `for account in accounts:` loop from this chapter and, on paper or in a comment, label which pillar each line demonstrates and why — there's more than one right answer for some lines.

2. **Add a fifth pillar-complete class.** Write a `Teacher(User)` class that: keeps `name`/`email`/`__password` from `User` (encapsulation, inheritance), overrides `logout()` to print an extra line before calling `super().logout()` (polymorphism), and implements `perform_duty()` (abstraction). Add it to `accounts` and confirm the loop still works without any other changes.

---

## Key Takeaways

- The four pillars aren't four separate techniques applied in sequence — they're four properties of one well-designed class, usually all present at once.
- Encapsulation and abstraction both involve "hiding," but they hide different things: encapsulation hides data, abstraction hides implementation complexity and guarantees behavior.
- Inheritance and polymorphism both involve shared code, but inheritance is about reusing an implementation, while polymorphism is about the same call producing different results.
- A single line of calling code — like `account.perform_duty()` — can rely on all four pillars at once without ever mentioning any of them by name.

---

## Chapter Summary

Throughout this chapter, we focused on one central idea:

> **The four pillars are not four separate systems — they are four questions you can ask about the same class, all answered at once.**

We took the platform built across the last four chapters and looked at it as a whole: one class hierarchy where private data stays protected, related roles share one implementation, the same call adapts to whichever object receives it, and every required behavior is guaranteed to exist. We also stepped back with a simple analogy — a car — to give these four ideas a shape that outlasts any one code example.

This closes out Object-Oriented Programming's four pillars. In the next chapter, we'll look at Python's special methods and the `@dataclass` shortcut — tools that make writing classes like these noticeably shorter, without changing any of the ideas built here.
