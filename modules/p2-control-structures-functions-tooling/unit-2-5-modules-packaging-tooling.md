# 2.5 Modules, Packaging & Professional Tooling

---

[← Previous: 2.4 Functional Constructs](unit-2-4-functional-constructs.md) | [Go back to TOC](../../README.md) | [Next: 3.1 Lists →](../p3-data-structures/unit-3-1-lists.md)


---

## Why Modules?

Every real project is bigger than one file. A **module** is just a Python file whose code you can reuse elsewhere, and a **package** is a folder of related modules bundled together. This unit covers using code other people have already written, both built into Python and installed from outside, and a first look at the tooling professional Python projects actually use.

**A quick note before we start:** most of this unit works directly in Colab, including virtual environments, with a small syntax difference explained below. Poetry and Pytest are tools built around managing a real project folder with multiple files, which doesn't map cleanly onto a single Colab notebook. You'll get what they are and why they matter here, and try them properly once you're working locally, with an editor like VS Code.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Module** | A single Python file whose code you can import and reuse |
| **Package** | A folder of related modules bundled together |
| **Standard Library** | Modules that ship with Python itself, no install needed |
| **`pip`** | Python's tool for installing packages other people have published |
| **Virtual environment** | An isolated space for a project's packages, kept separate from other projects |
| **Dependency** | A package your project relies on to run |

---

## Modules and Imports

Bring in code from another module with `import`:

```python
import math

print(math.sqrt(16))
```
```
4.0
```

`math.sqrt` reads as "the `sqrt` function, inside the `math` module." You always need the `math.` part, that's called **namespacing**, it keeps names from different modules from clashing with each other or with your own variables.

**`from ... import`** pulls out just one specific thing, so you can use it without the prefix:

```python
from math import sqrt

print(sqrt(16))
```
```
4.0
```

**`as`** gives a module (or an imported name) a shorter alias:

```python
import math as m

print(m.sqrt(16))
```
```
4.0
```

**Why namespacing matters:** imagine two different modules each had their own `sqrt` function that worked differently. Without namespacing, importing both would cause one to silently overwrite the other. `math.sqrt` versus `numpy.sqrt` (if you use both someday) makes it completely unambiguous which one you're calling.

**One thing to avoid:** `from module import *` pulls in *everything* from a module, with no prefix at all. It looks convenient, but it's exactly how those name clashes happen, and makes it unclear later where a given name actually came from. Stick to `import module` or `from module import specific_thing`.

---

## The Python Standard Library

Python ships with a huge collection of modules built in, no installing anything, just `import` and go. Three you'll use constantly:

**`random`**, generating random values:

```python
import random

print(random.randint(1, 10))       # a random whole number, 1 to 10 inclusive
print(random.choice(["A", "B", "C"]))   # a random pick from a sequence
```

**`math`**, mathematical functions and constants:

```python
import math

print(math.sqrt(25))
print(math.pi)
print(math.ceil(4.2))    # rounds up: 5
print(math.floor(4.8))   # rounds down: 4
```

**`datetime`**, working with dates and times:

```python
from datetime import datetime

now = datetime.now()
print(now)
print(now.year)
```

There are hundreds of standard library modules, these three are just the ones you'll reach for early and often. Whenever you need to do something common, checking whether Python already has a module for it is always worth a search before writing it yourself.

---

## Package Management with `pip`

The standard library covers a lot, but not everything. `pip` installs packages other people have published, for things like making web requests or working with data:

```
pip install requests
```

In a Colab cell specifically, prefix it with `!` to run it as a shell command:

```python
!pip install requests
```

Once installed, you `import` it exactly like a standard library module:

```python
import requests
```

Packages installed this way aren't part of Python itself, they're downloaded from the internet (from a registry called PyPI), so an internet connection is needed the first time.

---

## Virtual Environments (`venv`)

**The problem:** two different projects on your machine might need different, conflicting versions of the same package. Installing packages globally means every project shares the same set, and upgrading one for Project A might quietly break Project B.

**The fix:** a **virtual environment** is an isolated folder holding its own separate set of installed packages, one per project, so they never interfere with each other.

On your own machine, in a terminal:

```
python -m venv myenv
```

This creates the isolated environment. You then **activate** it before working:

```
# macOS/Linux
source myenv/bin/activate

# Windows
myenv\Scripts\activate
```

Once activated, anything you `pip install` goes into that environment only, not your whole system. You'll see the environment's name appear in your terminal prompt as confirmation it's active.

**You can actually try this in Colab too**, with one difference. Each Colab cell runs as its own fresh subprocess, so a normal `activate` command doesn't "stick" between cells the way it does in a real terminal. Instead, create the environment once, then call its `python`/`pip` directly by path, in every cell that needs it:

```python
!python -m venv myenv
!myenv/bin/pip install requests
!myenv/bin/python -c "import requests; print(requests.__version__)"
```

This genuinely creates an isolated environment inside your Colab session, packages installed into `myenv` won't touch Colab's own default packages. It's a slightly different workflow from a local terminal (no persistent "activated" state), but the isolation itself is real, not just a concept to read about.

---

## Poetry

`venv` plus `pip` works, but tracking exactly which package versions a project needs, and reproducing that setup elsewhere, is easy to get wrong by hand. **Poetry** is a newer tool that manages a project's dependencies, virtual environment, and version tracking together, in one place.

```
poetry new my-project
```

This scaffolds a new project folder with a `pyproject.toml` file, where Poetry records exactly which packages (and versions) your project depends on. Anyone else can then recreate your exact setup with a single command, instead of manually installing each package and hoping the versions match.

This is a concept to recognize for now, not something to practice inside Colab, it's a local, terminal-based tool for structuring real projects.

---

## Pytest

As programs grow, manually re-checking that everything still works after a change doesn't scale. **Pytest** is a tool for writing automated tests, small functions that check your code actually does what it's supposed to.

A simple test file, `test_math_utils.py`:

```python
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
```

`assert` checks that a condition is `True`, if it isn't, the test fails loudly and tells you exactly what went wrong. Running `pytest` in the terminal (from the same folder) automatically finds every function starting with `test_` and runs it:

```
pytest
```

```
1 passed in 0.01s
```

You *can* try `assert` itself directly in Colab, it's a regular Python keyword:

```python
assert 2 + 2 == 4     # passes silently
assert 2 + 2 == 5     # AssertionError
```

But `pytest` itself, discovering and running test files automatically, is a local, terminal-based workflow, one you'll build on properly once you're working across multiple files in a real project.

---

## Try it Yourself

**(a)** Import `random` and print 5 random numbers between `1` and `100`, using a loop from Unit 2.2.

**(b)** Using `math`, write a function `circle_area(radius)` that returns the area of a circle (`math.pi * radius ** 2`), with type hints from Unit 2.3.

**(c)** Using `datetime`, print the current year and figure out (using ordinary subtraction) how many years until 2050.

**Your turn:** write an `assert` statement that checks whether `circle_area(0)` correctly returns `0.0`.

---

## Common Mistakes

- Forgetting the `module.` prefix after a plain `import`, `math.sqrt(16)`, not just `sqrt(16)`
- Using `from module import *`, invites name clashes and makes code harder to trace
- Forgetting the `!` prefix when running `pip install` inside a Colab cell
- Assuming a package is available without installing it first, standard library modules need only `import`, everything else needs `pip install` first
- Confusing venv (isolating packages per project) with Poetry (a full dependency and project manager built on top of that same idea)
- Trying to `source activate` a venv inside a Colab cell and expecting it to carry over to the next cell, it doesn't, call the venv's binaries by path instead

---

## Interview Questions

**Q1: What's the difference between a module and a package?**

A: A module is a single Python file. A package is a folder containing multiple related modules, bundled together.

**Q2: Why avoid `from module import *`?**

A: It pulls in every name from that module with no prefix, which can silently overwrite existing names and makes it unclear later where a given name actually came from.

**Q3: Why do virtual environments matter?**

A: Different projects can need different, conflicting versions of the same package. A virtual environment isolates each project's packages so installing or upgrading one project never affects another.

**Q4: What does `pytest` actually do when you run it?**

A: It automatically finds every function starting with `test_` in your project and runs it, reporting which passed and which failed, based on their `assert` statements.

---

## Quick Recap

- A module is one file; a package is a folder of related modules.
- `import`, `from ... import`, and `import ... as` all bring in code, namespacing (`module.name`) keeps names from clashing.
- The standard library (`random`, `math`, `datetime`, and hundreds more) ships with Python, no install needed.
- `pip install package` brings in external packages; use `!pip install` inside a Colab cell.
- Virtual environments (`venv`) isolate a project's packages; you can create and use one in Colab too, just by calling its binaries directly instead of `activate`.
- Poetry manages a project's dependencies and environment together; Pytest automates checking your code with `assert`-based tests. Both are best understood conceptually here, and tried properly once you're working across multiple files in a real local project.


## Reference Links

- [The Python Tutorial — Modules](https://docs.python.org/3/tutorial/modules.html)
- [Python 3 Documentation — The Python Standard Library](https://docs.python.org/3/library/index.html)
- [pip Documentation — Installing Packages](https://pip.pypa.io/en/stable/user_guide/#installing-packages)
- [Python Documentation — `venv`: Creation of Virtual Environments](https://docs.python.org/3/library/venv.html)
- [Pytest Documentation — Getting Started](https://docs.pytest.org/en/stable/getting-started.html)
- [Real Python — Python Modules and Packages: An Introduction](https://realpython.com/python-modules-packages/)
- [W3Schools — Python Modules](https://www.w3schools.com/python/python_modules.asp)

[← Previous: 2.4 Functional Constructs](unit-2-4-functional-constructs.md) | [Go back to TOC](../../README.md) | [Next: 3.1 Lists →](../p3-data-structures/unit-3-1-lists.md)

---

*© 2026 Revature · AI Native Engineering — Foundations · Unit 2.5 · Version 2.0*
