# Modules, Packaging & Professional Tooling

So far, every program you have written has lived inside a single Colab notebook — one file, written entirely by you, run from top to bottom by hand. That has been the right way to learn functions, loops, and conditionals, but it is not how real software gets built. A production application at a company like TCS, Infosys, or a Bengaluru fintech startup is split across dozens or hundreds of files. Most of the code it depends on was never written by anyone on the team at all — it was written by a stranger, published publicly, and simply installed. And none of that borrowed or hand-written code reaches real users until an automated check has actually confirmed it still works.

That gap — between "code that runs on my machine, in one file" and "code a whole team can share, install, and trust" — is what this unit closes. You will meet the **module**, the plain mechanism Python uses to split code across files and reuse it; **pip** and the **Python Package Index**, which let you install code other people have already written and tested; the **virtual environment**, which keeps one project's installed packages from quietly colliding with another's; and **Pytest**, which proves, automatically and repeatedly, that your code still does what you think it does. None of this is optional once you join a real team — on your very first day working with an existing Python codebase, you will almost certainly run an `import`, activate a virtual environment, run a `pip install`, and run a test suite, likely before you write a single new line of business logic.

By the time you finish this chapter, setting up a small Python project "the professional way" will feel like a short, familiar checklist rather than a wall of unfamiliar commands.

---

## The problem that shows up the moment a project grows

Picture a campus store's checkout system. Someone writes a function that works out a discounted price, and it lives happily in one file. Then the receipt printer needs the exact same calculation. Then the nightly sales report needs it too. Then a manager finds a rounding bug in it. Do you fix the bug in one file and hope you remembered to copy the fix into the other three? Real projects hit this problem within days, not years — and copying logic instead of reusing it is exactly how the same bug ends up living in four places at once, fixed in only one of them.

Python's answer is the **module**: a single `.py` file containing code — functions, variables, anything at all — written once and reused wherever it is needed instead of retyped. The moment you save a `.py` file, it *is* a module; there is nothing extra you need to add to qualify it. If you write `pricing.py` with a function inside it, any other file sitting in the same folder can reach that function by writing `import pricing` — exactly the same mechanism Python uses to load code the Python team wrote years ago. The `import` system does not care who wrote a module or when; it treats your two-minute-old file and a module that ships with Python itself identically.

Once code can live in reusable files, a second, bigger question follows almost immediately: why retype something a stranger on the internet has already written, tested, and published? That question is where `pip` and Python's public package catalog come in — but before getting there, it helps to see exactly how `import` works.

## Bringing a module's contents into your file: `import`

**Importing** is the act of loading a module's contents into your own program with an `import` statement. Here is the simplest possible example, using `math`, a module that ships with Python itself:

```python
import math
print(math.sqrt(81))
```

```
9.0
```

`import math` loads everything inside the `math` module and makes it available through the name `math`. The dot in `math.sqrt(81)` reaches *into* that module to use one specific function it contains. Python actually gives you four different ways to write an import, and each one suits a slightly different situation.

| Form | Example | What it does |
|---|---|---|
| `import module` | `import math` | Loads the whole module under its own name; reach into it with a dot: `math.sqrt(9)`. |
| `from module import name` | `from math import sqrt` | Pulls out one specific item so you can use it directly, with no dot: `sqrt(9)`. |
| `import module as alias` | `import math as m` | Loads the whole module under a shorter or clearer name: `m.sqrt(9)`. |
| `from module import name as alias` | `from math import sqrt as square_root` | Combines both — pulls out one item and renames it. |

**`from module import name` only makes the one named item reachable — nothing else inside that module comes along for free.** Write `from math import sqrt` and try calling `math.floor(4.7)` right afterward, and Python has no idea what `math` even refers to, because you never imported the module itself, only one function out of it. This trips up beginners constantly, so before moving on, predict for yourself: after `from datetime import date`, would `datetime.timedelta(days=1)` work? It would not — you would need `import datetime` instead, or a second, separate `from datetime import timedelta`.

## Packages, the standard library, and PyPI

Where a module is one file, a **package** is a related group of modules bundled together and distributed as a single, installable, importable unit. Think of a module as one tool and a package as the whole toolbox it ships inside — installing or importing the toolbox's name gives you access to everything inside it at once.

Some modules need nothing installed at all, because they belong to the **standard library** — the large collection of modules that ships together with Python itself, ready to use the instant Python is installed. `math` and `datetime` are both standard library modules; every machine that has Python has them, with zero setup.

Beyond the standard library sits **PyPI**, the **Python Package Index** — the public, official catalogue of Python packages that any developer can publish to, and any other developer can install from. When someone says "install this package," PyPI is almost always where it is coming from. A **dependency** is any external package your project needs installed in order to run correctly; the moment your code contains `import requests`, `requests` has become a dependency of that project, something that must exist wherever the project runs.

## `pip`: reaching beyond the standard library

`pip` is Python's official package installer — a program you run from a **terminal** (a text window where you type commands directly, rather than clicking buttons) that downloads and installs packages from PyPI by name.

```bash
pip install requests
```

```
Collecting requests
Downloading requests-2.31.0-py3-none-any.whl
Successfully installed requests-2.31.0
```

Running this once means any Python program on that machine can now `import requests`, a widely used package for talking to web services. Packages carry version numbers, and `pip` can target one precisely:

```bash
pip install requests==2.31.0
```

Recording an exact version like this is called **pinning**. Pinning matters because a newer release of a package can quietly change how it behaves — a project that just says "install `requests`," with no version attached, might work perfectly today and break tomorrow on a colleague's machine, for a reason that has nothing to do with anything either of you wrote. **Leaving a dependency unpinned is how "works on my machine" quietly turns into a bug report from someone else's machine.** The command `pip freeze > requirements.txt` writes every currently installed package, with its exact pinned version, into a plain text file — keep that file alongside your code, and a teammate, a server, or your own machine six months from now can rebuild the identical setup with one command.

## A separate toolbox for every project: virtual environments

Here is the problem that shows up the moment two projects share one computer. `pip install` has to put a package *somewhere*, and without any special setup, that "somewhere" is one single, shared, system-wide copy of Python that every project on the machine uses. Suppose a payments project needs version 1 of a package and a reporting project on the same laptop needs version 2 of that exact same package. One shared copy cannot satisfy both at once — installing what the reporting project needs would silently break the payments project, and nobody might notice until something fails in production.

A **virtual environment** solves this by giving each project its own sealed toolbox: an isolated, project-specific folder holding its own copy of installed packages, kept completely separate from every other project's. Picture a mechanic who keeps one full set of tools per client, in labelled boxes, rather than one shared toolbox everyone digs through — nothing from one client's box ever ends up mixed into another's. Setting one up takes exactly two steps, and both of them matter:

```bash
python -m venv venv
venv\Scripts\activate
```

```
(venv) C:\projects\campus-store>
```

`python -m venv venv` creates a new, empty virtual environment in a folder named `venv` — an isolated space, but with nothing installed into it yet. `venv\Scripts\activate` (on Windows; `source venv/bin/activate` on Mac or Linux) **activates** it, meaning every `pip install` you run afterward, in that same terminal session, lands inside this private folder instead of the shared system-wide Python. You will usually see the environment's name appear in brackets at the start of your terminal prompt, like `(venv)` above, confirming it is active.

**Creating a virtual environment without activating it changes nothing about where your packages land.** This is the single most common tooling mistake among freshers: they run `python -m venv venv`, feel done, and then run `pip install` straight afterward without activating it — and every package still lands in the shared, system-wide Python, exactly as if the isolated folder had never been created. The other equally common mistake runs the opposite direction: skipping the virtual environment entirely and running `pip install` directly, quietly upgrading or downgrading a package that some unrelated project on the same machine depends on.

## A newer all-in-one option: Poetry

`pip` and `venv` are two separate tools that you learn to run together — one creates the isolated folder, the other installs into it, and a third file (`requirements.txt`) tracks what got installed. **Poetry** is a newer Python packaging tool that folds all three of those jobs into one: it creates and manages the virtual environment for you behind the scenes, and it records your project's exact, pinned dependencies in a single project file rather than a separate `requirements.txt` you update by hand.

| | pip + venv | Poetry |
|---|---|---|
| Creating the isolated environment | A separate step (`python -m venv venv`) | Handled automatically per project |
| Recording dependencies | A `requirements.txt` you regenerate yourself | A single project file, kept in sync automatically |
| Typical command to add a package | `pip install requests` then `pip freeze > requirements.txt` | `poetry add requests` |
| Who tends to use it | Every Python installation, no extra setup | Teams that want one consistent, all-in-one workflow |

You do not need to master Poetry's exact commands in this unit — what matters is recognising the idea: it exists to solve the same isolation and pinning problems as `pip` and `venv`, just packaged as a single, more convenient tool. Many professional Python teams use one or the other, and some use both depending on the project.

## When Python can't find what you asked for: `ModuleNotFoundError`

A `ModuleNotFoundError` is what Python raises when an `import` names a module it cannot locate anywhere it looks. Python checks, in a fixed order: first, has this exact module already been loaded once during the current run — if so, it just reuses it. Then, is it part of the standard library? Then, is it installed in whichever environment is currently active? Then, is it a `.py` file sitting in the same folder as the script being run? Only if none of those match does Python give up and raise the error.

That order matters because the *same* error message can come from three completely different causes that look identical to a beginner: the module name is simply misspelled; the package was genuinely never installed anywhere; or — the trickiest case of the three — it was installed, but into a *different* environment than the one currently running your code, for example installed system-wide while a virtual environment sits active and empty. A **dependency conflict** is the closely related situation where two packages, or two projects sharing one environment, need incompatible versions of the same underlying package at the same time — exactly what a virtual environment exists to prevent in the first place.

Before reading on, predict: you activate a virtual environment, forget you activated it, and run `pip install requests` in a *different* terminal tab where nothing is active. Then your script, run from the first tab, raises `ModuleNotFoundError: No module named 'requests'`. What actually went wrong? The package was installed into the system-wide Python, not into the virtual environment your script is actually running inside — the fix is reinstalling it with the correct environment active, not reinstalling it again blindly.

## Proving your code still works: Pytest

Every function you have written so far, you have tested by eye — running it once, looking at the output, and deciding it looks right. That does not scale. A checkout service touched by ten developers over a year needs something more reliable than "I re-ran it and it looked fine." **Pytest** is the most widely used Python testing framework — a tool that discovers and runs automated test functions and reports exactly which ones pass and which fail, in seconds, with no human eyeballing required.

Pytest finds tests by convention, not configuration: it automatically discovers files named `test_*.py` or `*_test.py`, and inside them, functions whose names start with `test_`. **A function named `check_discount()` will never be found or run by Pytest, no matter how correct its logic is — only a name starting with `test_` gets discovered.** Inside a test function, an `assert` statement is a claim that must hold true: Python evaluates the expression after `assert`, and if it's true, nothing visible happens and the test moves on; if it's false, Pytest reports that exact test as failed and shows precisely which line broke.

```python
# pricing.py
def apply_discount(price, percent_off):
    return price - (price * percent_off / 100)


def test_apply_discount():
    assert apply_discount(1000.0, 20) == 800.0
    assert apply_discount(500.0, 0) == 500.0
```

Run this from a terminal, in the same folder:

```bash
pytest pricing.py
```

```
========================= test session starts =========================
collected 1 item

pricing.py .                                                      [100%]

========================== 1 passed in 0.01s ===========================
```

Pytest scanned `pricing.py`, found one function matching its `test_*` rule, ran it, and checked both `assert` lines — both held, so it reports `1 passed`, with the single dot representing that one successful test. If a future change to `apply_discount()` — say, someone forgets a step in the arithmetic — ever produced a wrong result, the very first `assert` that no longer holds would immediately flip this into a reported failure, pointing straight at the broken line before a real customer ever sees a wrong total on their receipt.

## Putting the whole workflow together

Every real Python project you touch — in this course and afterward — tends to move through the same sequence, and you now have every piece of it:

1. Create the project folder and write your first `.py` module.
2. Create a virtual environment and activate it, *before* installing anything.
3. Install any needed third-party packages with `pip`, pinning exact versions.
4. Write your application code inside its own module(s), using `import` to reuse it across files.
5. Write test functions named `test_*`, with `assert` statements checking known-correct results.
6. Run `pytest`. If every test passes, the code — and its recorded dependencies — is ready to share with teammates. If a test fails, fix the code and run it again.

This exact pattern repeats across every domain you would plausibly work in. A bank's transaction-processing service is built from dozens of internal modules — interest calculation, fraud checks, statement generation — each pinned to exact dependency versions in production, because an unpinned version quietly changing behaviour is not a risk anyone takes with real money. A railway-style booking backend runs its fare-calculation module through Pytest on every single change, so a system used by millions never silently starts charging the wrong fare. An e-commerce checkout is typically covered by dozens of automated tests that run before any change goes live, catching a broken discount calculation before one customer sees a wrong total.

A few mistakes are worth watching for deliberately as you start doing this yourself:

- Running `pip install` with no virtual environment active, quietly changing the one shared, system-wide copy of a package.
- Creating a virtual environment but forgetting to activate it, so every `pip install` still lands system-wide anyway.
- Leaving a dependency unpinned, so "works on my machine today" silently stops being true somewhere else, later.
- Writing a test function that doesn't start with `test_`, so Pytest quietly never runs it at all.
- Assuming a `ModuleNotFoundError` always means "never installed," when it just as often means "installed into the wrong environment."

## Try it yourself

Do this in your own project folder before moving on. Create a file, `converter.py`, with one function, `celsius_to_fahrenheit(celsius)`, that returns `celsius * 9/5 + 32`. In the same file, write a test function `test_celsius_to_fahrenheit()` containing at least two `assert` statements — one ordinary case, such as `0` degrees, and one edge case, such as a negative temperature. Create and activate a virtual environment first, just to practise the habit, then run `pytest converter.py` from a terminal and confirm every test is reported as passed. Finally, deliberately break the function — remove the `+ 32` — and run `pytest` again to see exactly how a failure gets reported.

---

### Key Terminology

- **Module** — a single `.py` file containing reusable code — functions, variables, anything — that can be imported into another file.
- **Importing** — the act of loading a module's contents into your own program with an `import` statement.
- **Package** — a related group of modules bundled and distributed together as one installable, importable unit.
- **Standard library** — the large set of modules that ships with Python itself, needing no separate installation.
- **PyPI (Python Package Index)** — the public, official catalogue of Python packages that anyone can publish to and install from.
- **`pip`** — Python's official package installer; downloads and installs packages from PyPI by name.
- **Dependency** — any external package a project needs installed in order to run correctly.
- **Pinning** — recording the exact version of a dependency a project needs, instead of "whatever is newest."
- **Virtual environment** — an isolated, project-specific space holding its own copy of installed packages, separate from every other project's.
- **Activation** — the step that redirects subsequent `pip install` commands into the currently active virtual environment.
- **Poetry** — a newer packaging tool that combines environment creation and dependency pinning into a single, all-in-one workflow.
- **`ModuleNotFoundError`** — the error Python raises when an `import` names a module it cannot locate anywhere it looks.
- **Dependency conflict** — when two required packages, or two projects sharing one environment, need incompatible versions of the same underlying package at once.
- **Pytest** — the most widely used Python testing framework; discovers and runs automated test functions and reports pass/fail results.
- **`assert`** — a statement inside a test that must evaluate true; if false, Pytest reports that test as failed.

### Mastery Checkpoint

Before moving to Unit 3.1, check that you can answer these without looking back:

1. What is the difference between a module and a package, and how does `from module import name` differ from `import module` in terms of what actually becomes reachable in your file?
2. Why does pinning a dependency's version (`requests==2.31.0`) matter, and what can go wrong on someone else's machine if you leave it unpinned?
3. What specific problem does a virtual environment solve, and what happens if you create one with `python -m venv venv` but forget to activate it?
4. You get a `ModuleNotFoundError` for a package you're fairly sure you installed. What are the three possible causes, and how would you check which one it actually is?
5. Why will Pytest never discover or run a function named `check_discount()`, no matter how correct its logic is?

### Summary

You have now closed the gap between writing a single Python file and building something a real team could actually ship. You know how modules let you write logic once and reuse it everywhere it's needed, how the standard library and PyPI, reached through `pip`, let you build on code you never had to write yourself, and how a virtual environment keeps one project's dependencies from quietly colliding with another's — with Poetry as a newer tool that bundles those same ideas into one workflow. You have also seen how Pytest turns "did I break something?" from a guess into an automatic, repeatable answer. That closes out Module II — you have now covered control structures, functions, and the professional tooling that ties real projects together. Module III turns from single values and logic toward organizing many values at once, starting with Unit 3.1, your first core data structure: the list.

### Additional Resources

- [Python Tutorial — official docs: "Modules"](https://docs.python.org/3/tutorial/modules.html)
- [Python 3 Documentation — The Python Standard Library](https://docs.python.org/3/library/index.html)
- [Python 3 Documentation — `venv`: Creation of Virtual Environments](https://docs.python.org/3/library/venv.html)
- [Python 3 Documentation — `math` module](https://docs.python.org/3/library/math.html)
- [W3Schools — Python Modules](https://www.w3schools.com/python/python_modules.asp)
- [W3Schools — Python PIP](https://www.w3schools.com/python/python_pip.asp)
