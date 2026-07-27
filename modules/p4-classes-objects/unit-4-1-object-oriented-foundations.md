# Object-Oriented Foundations

## Why Object-Oriented Programming Exists

Imagine you're building a simple social media app. With one user,
storing their details is easy:

``` python
username = "Rahul"
followers = 520
posts = 18
```

Add a second user, and you just duplicate the variables:

``` python
user1_name = "Rahul"
user1_followers = 520

user2_name = "Ananya"
user2_followers = 830
```

Still fine. But real apps don't have two users — they have
**millions**. Duplicating variables that far looks like this:

``` python
user1_name = "Rahul"
user2_name = "Ananya"
user3_name = "Vikram"
...
user1000000_name = "..."
```

Now say every user needs an **email address** added. You'd have to
touch all one million of these variables, one by one — a five-minute
feature turns into a nightmare.

> **Can we describe what a user looks like just once, and then create
> as many users as we need from that one description?**

That question is the foundation of **Object-Oriented Programming
(OOP)**.

> **💡 Interesting Fact**
>
> Apps like **Instagram**, **Amazon**, **Netflix**, and **Uber** create
> and manage millions of objects every day. Without OOP, that would be
> nearly impossible to build or maintain.

------------------------------------------------------------------------

## What is Object-Oriented Programming (OOP)?

**OOP** is a way of designing programs around **objects**, instead of
repeating the same variables and functions for every single user.

You describe the reusable design once — what data a user holds, what
actions a user can perform — then create as many objects as you need
from that one description.

Every user in our app has the same shape:

-   Username
-   Followers
-   Posts
-   Profile Picture

Define this once, reuse it forever. That's what makes OOP programs
easier to write, read, maintain, and scale.

------------------------------------------------------------------------

## How OOP Solves the Original Problem

### Without OOP

``` text
User 1
 ├── Username
 ├── Followers
 └── Posts

User 2
 ├── Username
 ├── Followers
 └── Posts

User 3
 ├── Username
 ├── Followers
 └── Posts

...
```

The same structure is repeated again and again.

### With OOP

``` text
             Reusable Design
                  │
                  ▼
               User Class
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
   User 1      User 2      User 3
```

The design is written once.

Objects are created whenever they are needed.

Now that we know **how OOP solves the problem**, the next question is:

> **Where do we write this reusable design?**

The answer is **a Class**.

------------------------------------------------------------------------

# What is a Class?

A **class** is the reusable design used in Object-Oriented Programming.

It defines:

-   What information every object should store.
-   What actions every object should perform.

For our application, every user should have:

-   Username
-   Followers
-   Posts
-   Profile Picture

(Notice these are the exact same three fields — `username`, `followers`,
`posts` — you already saw hard-coded at the very top of this page.)

Instead of defining these fields every time a new user joins, we define
them **once** inside a class.

## Creating Your First Class

``` python
class User:
    username = ...
    followers = ...
    posts = ...
```

Congratulations — you've just created your first class! This is the
**blueprint**. It doesn't create a real user — it simply describes
what every `User` will look like: a `username`, a `followers` count,
and a `posts` count. Each one is set to `...` for now — Python's own
placeholder token, meaning "a name exists here, but nothing real is
filled in yet." That's exactly what a blueprint should be: the
*shape* of the data, with nothing real attached.

Describe the shape once, then build as many real users as you want
from it. That's exactly what an **object** does next.

### Understanding the Syntax

| Code | Explanation |
|---|---|
| `class` | Tells Python you're creating a class. |
| `User` | The name of the class. By convention, class names start with a capital letter. |
| `:` | Marks the beginning of the class definition. |
| `username = ...` | A placeholder field — no real value yet, just a name showing what data a `User` will hold. |
| `followers = ...`, `posts = ...` | Declared the same way — placeholders showing the shape of the data, not real values. |

### What Does Python Know Now?

``` text
            User Class

      Username
      Followers
      Posts
      Profile Picture

            ↓

      Actual Users Created?

             ❌ No
```

The class exists.

The users do not.

> **💡 Interesting Fact**
>
> Creating a class **does not create objects automatically**. Python
> creates an object **only when you explicitly ask for one**.

Now the next question becomes obvious.

> **If the class doesn't create users, how do we create one?**

That's where **objects** come in.

------------------------------------------------------------------------

# What is an Object?

An **object** is an actual instance created from a class.

If the class is the reusable design,

the object is the real user created using that design.

Creating an object is simple.

``` python
user1 = User()
user2 = User()
user3 = User()
```

Each line builds a brand-new, **empty** object. None of them carry a
`username`, `followers`, or `posts` yet — creating an object doesn't
copy the class's example values, it just gives you a fresh, blank
one. Think of it as ordering three blank forms: same form, three
separate blank copies.

Now we fill each one in individually, using the dot (`.`) to attach
data directly onto one specific object:

``` python
user1.username = "Rahul"
user1.followers = 520
user1.posts = 18

user2.username = "Ananya"
user2.followers = 830

user3.username = "Vikram"
```

`user1.username = "Rahul"` only touches `user1` — it has no effect on
`user2` or `user3` at all. That's the entire point of an object: all
three were built from the exact same class, but each one now holds
its own separate, independent data.

``` python
print(user1.username, user1.followers, user1.posts)
print(user2.username, user2.followers)
```

``` text
Rahul 520 18
Ananya 830
```

Same class, same field names — but three completely separate objects,
each holding its own real data.

## How Does This Solve Our Original Problem?

We describe the user **once** using a **class**.

Then we create as many **objects** as needed.

``` text
            User Class

                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼

    user1     user2     user3

      ▼         ▼         ▼

    Rahul    Ananya    Vikram
```

Whether your application has:

-   10 users
-   10,000 users
-   10 million users

The **same class** can be reused to create every user.

------------------------------------------------------------------------

## Class vs Object

| Class | Object |
|---|---|
| A reusable design | An actual user created from the design |
| Created once | Can be created many times |
| Defines what an object should have | Stores actual data |
| Does not store user information | Stores information for one specific user |

------------------------------------------------------------------------

# The `__init__()` Constructor

Look back at how we gave `user1` and `user2` their data:

``` python
user1.username = "Rahul"
user1.followers = 520
user1.posts = 18

user2.username = "Ananya"
user2.followers = 830
```

One field, one line, for every single user. That's the exact same
repetitive problem we started this page with — it just moved to
*after* the object is created instead of before.

What we actually want is for every object to receive its data
**automatically**, the moment it's created — just by writing
`User("Rahul", 520, 18)`. Python gives us exactly this tool: a special
method called `__init__`.

``` python
class User:
    def __init__(self, username, followers, posts):
        self.username = username
        self.followers = followers
        self.posts = posts
```

`__init__` runs **automatically** every time a new object is created
from this class — you never call it yourself. Notice it takes one
more parameter than you actually type when calling `User(...)` — that
first one, `self`, deserves its own close look.

### The `self` Keyword

`self` is not a special reserved word — it's just a regular parameter
name, and by strong, near-universal convention, every Python
programmer names it `self`. What it holds is simple: **whichever
object the method is currently running on.**

Here's the part that trips people up: you never pass `self` yourself.
Python fills it in for you, automatically, using the object sitting
right before the dot. Line up what you *write* against what Python
*actually does*:

``` python
user1 = User("Rahul", 520, 18)
```

is really, behind the scenes:

``` python
User.__init__(user1, "Rahul", 520, 18)
```

`user1` — the brand-new, blank object Python just built — is slotted
in as the first argument, `self`, without you writing it anywhere.
That's the entire reason `__init__`'s parameter list has one extra
slot compared to what you type at the call site.

Trace it for two different users:

``` python
user1 = User("Rahul", 520, 18)
user2 = User("Ananya", 830, 45)
```

- On the first line, `self` is `user1` — so `self.username = username`
  really means `user1.username = "Rahul"`.
- On the second line, `self` is `user2` instead — the *exact same*
  code inside `__init__` now means `user2.username = "Ananya"`.

Same three lines, run twice, correctly setting data on two completely
different objects — purely because `self` points somewhere different
each time.

> **💡 Rule of thumb:** wherever you see `self` inside a class,
> mentally replace it with *"the object standing to the left of the
> dot, in whatever line made this call."* That one substitution is
> all `self` ever means.

Now creating a fully-loaded user takes one line:

``` python
user1 = User("Rahul", 520, 18)
user2 = User("Ananya", 830, 45)
user3 = User("Vikram", 210, 9)

print(user1.username, user1.followers, user1.posts)
print(user2.username, user2.followers, user2.posts)
```

``` text
Rahul 520 18
Ananya 830 45
```

No more assigning fields one at a time after the fact — `__init__`
does it in the same line the object is created. This is the missing
piece that finally solves the problem from the very top of this page:
describe a user once, and create a million different, independent
users, each with its own real data, in one line each.

------------------------------------------------------------------------

# Giving Objects Something to Do — Methods

So far, `User` only stores data — `username`, `followers`, `posts`.
But a real user doesn't just sit there holding numbers; they *do*
things: they post, they gain followers, someone looks up their
profile. A **method** is how a class describes an action an object
can perform, not just data it can hold.

### Creating a Method

A method is written exactly like a regular function — same `def`,
same indentation rules — except it lives inside the class body, and
takes `self` as its first parameter, for the same reason `__init__`
does:

``` python
class User:
    def __init__(self, username, followers, posts):
        self.username = username
        self.followers = followers
        self.posts = posts

    def add_post(self):
        self.posts = self.posts + 1

    def show_profile(self):
        print(self.username, "has", self.followers, "followers and", self.posts, "posts")
```

`add_post` and `show_profile` sit right below `__init__`, in the same
class body — matching the ordering convention from earlier:
constructor first, other methods after it. `add_post` takes only
`self`; a method can take extra parameters too, exactly like
`__init__` does, if it needs more information to do its job.

### Calling a Method — Which Object, and When

You call a method the same way you read an attribute — with a dot —
but with parentheses at the end:

``` python
user1 = User("Rahul", 520, 18)
user2 = User("Ananya", 830, 45)

user1.add_post()
user1.show_profile()
user2.show_profile()
```

``` text
Rahul has 520 followers and 19 posts
Ananya has 830 followers and 45 posts
```

**Which object does it run on?** Always the one written immediately
before the dot. `user1.add_post()` is really Python doing
`User.add_post(user1)` behind the scenes — `user1` slots into `self`,
the same automatic substitution `__init__` uses. So
`self.posts = self.posts + 1` really means `user1.posts = user1.posts
+ 1`, and `user2` is never touched by that line at all. That's why
`user1`'s post count moves from `18` to `19`, while `user2`'s
`show_profile()` still prints its original, unchanged `45`.

**When does it actually run?** Only at the exact moment you write
`object.method()` — nothing happens before that line, and nothing
happens automatically. This is a real contrast with `__init__`, which
Python calls for you the instant an object is created; an ordinary
method like `add_post` only ever runs when you explicitly call it,
exactly once per call, on exactly the object you called it on.

This is the whole idea, finally complete. A class bundles **data**
(`username`, `followers`, `posts`) with the **behavior** that acts on
that data (`add_post`, `show_profile`), and `self` is the thread that
connects a method back to the one specific object it was called on.
Everything on this page — the problem with plain variables, the
class as a blueprint, the object as a real instance, `__init__`
filling it in automatically, and now methods giving it actions — is
one continuous idea: describe a user once, and let every real user
you create carry both their own data and their own behavior with
them.

------------------------------------------------------------------------

## Reference Links

-   [Python Official Docs — Classes](https://docs.python.org/3/tutorial/classes.html)
-   [Python Official Docs — `__init__` and Class Objects](https://docs.python.org/3/tutorial/classes.html#class-objects)
-   [W3Schools — Python Classes and Objects](https://www.w3schools.com/python/python_classes.asp)
-   [W3Schools — Python Constructors](https://www.w3schools.com/python/python_classes.asp#h1)
