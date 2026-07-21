---
topic_id: cb1b0d0b-dc25-433d-8bce-01f583eb93c7
title: Unit 2.5 - Modules, Packaging & Professional Tooling
position_in_module: 5
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 2.5 - Modules, Packaging & Professional Tooling — Topic Corpus

## 2. Prerequisites

This topic builds directly on **Unit 2.3 - Functions** (`ed4fbf1a-b7e5-4df2-b368-1ecd21e1c99c`), because every module in this unit is, underneath, a file full of functions defined with `def` and `return`. It also assumes the full run of Module P2 so far — **Unit 2.1 - Conditionals** (`c9feb41c-1aee-404d-b359-d468df334892`) and **Unit 2.2 - Loops** (`0ef5b1c5-43be-4f7a-8e00-ac4ab29451e5`) for the general idea of writing multi-line, structured code, and **Unit 2.4 - Functional Constructs** (`07a78e31-727b-4c0d-95aa-4e7712dceda2`) as the most recent step before this one. No new control-flow ideas are introduced here — this unit is about *organizing and protecting* the code you already know how to write.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** what a module and a package are, and describe how Python's `import` statement brings code from one file into another.
- **Differentiate** between the `import x`, `from x import y`, and `import x as y` forms, and identify which one fits a given situation.
- **Describe** what the standard library, PyPI, and `pip` are, and **apply** `pip` to install a third-party package.
- **Explain** the problem a virtual environment solves, and **describe** the steps to create, activate, and install packages into one.
- **Apply** Pytest's `test_*` and `assert` conventions to explain how an automated test proves a function still works.
- **Identify** the most common beginner tooling mistakes — a `ModuleNotFoundError`, an unactivated virtual environment, and an unpinned dependency — and explain what causes each one.

## 4. Introduction

Imagine you have written a function that calculates a discounted price for a campus store. It works perfectly in one file. Now imagine three other files in the same project also need that exact calculation — a checkout screen, a receipt printer, and a nightly sales report. Do you retype the function three more times? If you later find a bug in it, do you remember to fix it in all four places? Real projects run into this problem within days, not years, and Python's answer to it is the **module**: a file of code written once and reused everywhere it is needed.

That single idea — write once, reuse anywhere — is the seed of everything in this unit. Once code lives in reusable files, a natural next question appears: why retype something that a stranger on the internet has already written, tested, and published? That is what `pip` and Python's package catalog are for. But borrowing other people's code creates a new risk — two projects on the same computer might need two different, incompatible versions of the same borrowed code. A **virtual environment** solves that. And once a project is built from many reused pieces, someone has to keep proving, automatically, that all of it still works correctly every time something changes — that is the job of **Pytest**.

None of this is optional in a real job. On your first day working with an existing Python codebase, you will almost certainly run an `import` statement, activate a virtual environment, run a `pip install`, and run a test suite — likely before you write a single new line of business logic [1]. This unit is your introduction to that everyday reality.

## 5. Core Concepts

### 5.1 Modules and importing

A **module** is a single `.py` file containing code — functions, variables, anything at all — that is written so it can be reused inside a different file instead of being retyped from scratch [1]. Any Python file you save automatically becomes a module the instant another file wants to reuse it; there is nothing special you have to add to a file to qualify it as one.

**Importing** is the act of bringing a module's contents into your own program with an `import` statement [1]. For example:

```python
import math
print(math.sqrt(81))
```

`math` is a module that ships with Python itself. `import math` loads everything inside that file and makes it available through the name `math`. `math.sqrt(81)` then reaches into that module with a dot (`.`) to use one specific function it contains, `sqrt()`, which computes a square root.

Python supports four ways of writing an `import`, each suited to a different situation [1]:

| Form | Example | What it does |
|---|---|---|
| `import module` | `import math` | Loads the whole module under its own name; every item inside it is reached with a dot, e.g. `math.sqrt(9)`. |
| `from module import name` | `from math import sqrt` | Pulls out one specific item so it can be used directly, with no dot: `sqrt(9)`. |
| `import module as alias` | `import math as m` | Loads the whole module but gives it a shorter or clearer name to use afterward: `m.sqrt(9)`. |
| `from module import name as alias` | `from math import sqrt as square_root` | Combines both — pulls out one item and renames it. |

`from module import name` only makes the one named item reachable — everything else inside that module stays out of reach unless it is imported too [1]. This matters because it is a common source of confusion: importing `sqrt` from `math` does not also give you `math.floor`, for instance, unless you import that separately.

If you write your own file, say `pricing.py`, containing a function, another file in the same folder can `import pricing` and reach that function as `pricing.some_function()` — exactly the same mechanism as `import math`. The only difference between a module you wrote two minutes ago and one the Python team wrote years ago is who wrote it and how long ago; the `import` mechanism treats them identically [1].

### 5.2 Packages, the standard library, and PyPI

A **package** is a related group of modules bundled together and distributed as a single unit, so that installing or importing one name gives access to everything inside it [1]. When code is described as "a package," think of it as a folder of related modules rather than one lone file.

The **standard library** is the large collection of modules that ships together with Python itself, ready to use the moment Python is installed, with no extra installation step required [1]. `math` and `datetime` are both standard library modules — they exist on any machine that has Python, without anyone running an install command first.

Beyond the standard library sits **PyPI**, the **Python Package Index** — the public, official catalog of Python packages that any developer can publish to, and any other developer can install from [1]. PyPI is where third-party (meaning: written by someone outside the Python core team) packages live. When you install "a package" with a tool, this is almost always where it comes from.

### 5.3 `pip`: installing packages beyond the standard library

`pip` is Python's official package installer — a program you run from a **terminal** (a text-based window where you type commands directly, instead of clicking buttons in an app) that downloads and installs packages from PyPI by name [1].

```bash
pip install requests
```

This single command downloads the `requests` package (a widely used package for talking to web services) from PyPI and installs it so any Python program on the machine can `import requests`.

A **dependency** is any external package that a project needs installed in order to run correctly [1]. The moment a project's code contains `import requests`, `requests` has become one of that project's dependencies — something that must be installed wherever the project is meant to run.

Packages carry version numbers, and `pip` can target one exactly:

```bash
pip install requests==2.31.0
```

This is called **pinning** a version — recording the exact version number a project depends on, instead of accepting "whatever is newest today" [1]. Pinning matters because a newer version of a package can quietly change behavior; a project that says only `requests` (no version) might work perfectly today and break tomorrow, on someone else's machine, for a reason that has nothing to do with anyone's own code.

The command `pip freeze > requirements.txt` writes every currently installed package, along with its exact version, into a plain text file named `requirements.txt` [1]. Keeping that file alongside a project's code means anyone else — a teammate, a server, your own machine six months from now — can install the identical set of pinned dependencies again and get identical behavior.

### 5.4 Virtual environments: isolating one project's packages from another's

Here is the problem a **virtual environment** exists to solve. `pip install` has to install a package *somewhere*. Without any special setup, that "somewhere" is one single, shared, system-wide copy of Python — the same one every project on the machine uses. If Project A needs version 1 of a package and Project B needs version 2 of that same package, they cannot both be satisfied by one shared copy at the same time. Installing the version Project B needs would silently break Project A, and neither developer might realize it until something fails.

A **virtual environment** is an isolated, project-specific space holding its own copy of installed packages, kept separate from every other project's [1]. Each project gets its own private set of installed packages, so two projects needing two different — even conflicting — versions of the same package can coexist on one machine without interfering with each other.

Setting one up takes two steps, and both matter:

```bash
python -m venv venv
venv\Scripts\activate
```

`python -m venv venv` creates a new, empty virtual environment in a folder named `venv` — an isolated space, but with nothing installed into it yet [1]. `venv\Scripts\activate` (on Windows; `source venv/bin/activate` on Mac/Linux) **activates** it, meaning every `pip install` run afterward, in that same terminal session, lands inside this private folder instead of the shared system-wide Python [1]. Creating a virtual environment without activating it changes nothing about where packages land — activation is the step that actually redirects `pip install`.

### 5.5 `ModuleNotFoundError` and how Python finds a module

A `ModuleNotFoundError` is the error Python raises when an `import` names a module it cannot locate anywhere it looks [1]. Python looks in a fixed order: first among its own built-in modules, then among the packages installed in whichever environment is currently active, then among `.py` files sitting in the same folder as the script being run. If none of those contain a matching name, Python gives up and raises `ModuleNotFoundError`.

This means the same error can come from three different causes that look identical to a beginner: the module name is simply misspelled; the package was genuinely never installed anywhere; or — the trickiest case — it was installed, but into a *different* environment than the one currently running the code (for example, installed system-wide while a virtual environment is active, or vice versa) [1]. A **dependency conflict** is the related situation where two required packages, or two projects sharing one environment, need incompatible versions of the same underlying package at the same time — a virtual environment is what prevents this from happening in the first place.

### 5.6 Pytest: proving code still works

**Pytest** is the most widely used Python testing framework — a tool that discovers and runs automated test functions and reports which ones pass and which fail [1]. Instead of a developer manually re-running a program and eyeballing whether the output still looks right, Pytest runs small, automated checks and reports the result in seconds.

Pytest finds tests using a naming convention: it automatically discovers files named `test_*.py` or `*_test.py`, and inside them, functions whose names start with `test_` [1]. A function named `check_add()` will never be found or run by Pytest, no matter how correct its logic is — only a name starting with `test_` is discovered.

Inside a test function, an **assert statement** is a claim that must hold true. It is written as `assert` followed by an expression that Python evaluates; if that expression is true, nothing visible happens and the test continues; if it is false, Pytest reports that specific test as failed and shows exactly which `assert` broke [1].

```python
def apply_discount(price, percent_off):
    return price - (price * percent_off / 100)

def test_apply_discount():
    assert apply_discount(1000.0, 20) == 800.0
    assert apply_discount(500.0, 0) == 500.0
```

Running `pytest` from a terminal in the same folder scans for matching files and functions, runs `test_apply_discount()`, and checks both `assert` lines. If a later change to `apply_discount()` ever produced a wrong result, the first `assert` that no longer holds turns this into an immediately reported failure — pointing straight at the broken behavior before anyone using the finished program ever sees a wrong number [1].

## 6. Implementation

**How Python resolves an `import` statement**, in the order it actually checks [1]:

1. Has this exact module already been loaded once during the current run? If yes, reuse it — no repeated work.
2. Is it part of the standard library? If yes, load it directly — nothing needs to be installed.
3. Is it installed in the currently active environment (for example, via a prior `pip install`)? If yes, load it from there.
4. Is it a `.py` file sitting in a folder Python searches, typically the same folder as the file doing the importing? If yes, load that file as a module.
5. If none of the above match, Python raises `ModuleNotFoundError`.

**Setting up a small project the professional way** — the same sequence appears at the start of essentially every real Python project:

1. Create the project's folder and write the first `.py` module.
2. Create a virtual environment (`python -m venv venv`) and activate it.
3. Install any needed third-party packages with `pip install`, pinning exact versions.
4. Write the application code inside its own module(s), using `import` to reuse code across files.
5. Write test functions named `test_*`, each containing `assert` statements that check known-correct results.
6. Run `pytest`. If every test passes, the code — and its recorded dependencies — is ready to be shared with teammates. If a test fails, return to step 4 and fix the code before trying again.

## 7. Real-World Patterns

- **E-commerce checkout:** A checkout service is typically covered by dozens of Pytest tests that run automatically before any code change goes live — catching a broken discount or tax calculation before a single customer ever sees a wrong total [1].
- **Banking and fintech:** A bank's transaction-processing service is built from many internal modules (interest calculation, fraud checks, statement generation), each pinned to exact dependency versions in production, because an unpinned version upgrade quietly changing behavior is not an acceptable risk when real money is involved [1].
- **Railway-style booking systems:** A fare-calculation module — base fare multiplied by number of seats — is exactly the kind of small, reusable logic that gets its own Pytest test, so a booking engine used by millions of people never silently starts charging the wrong fare after a change [1].
- **Education platforms:** A student-grading calculation is a small, well-tested, reusable module imported wherever grading needs to happen across an ed-tech backend [1].

The pattern repeats across all of these: isolate a project's dependencies in their own virtual environment, pin their exact versions, and let an automated test — not a person re-checking by hand — confirm the code still behaves correctly before it reaches real users [1].

## 8. Best Practices

- **Pin your dependencies.** Record the exact version of every package a project needs (`requests==2.31.0`, not just `requests`), so the project behaves identically on every machine and at every point in time [1].
- **Always use a virtual environment for real project work.** Never install project-specific packages straight into the system-wide Python.
- **Keep the dependency file under version control.** Commit `requirements.txt` alongside the code so teammates can rebuild the exact same setup.
- **Name every test function `test_*`**, with a name that describes what it checks (`test_calculate_fare_with_discount`, not `test_1`), so Pytest can discover it and so its purpose is obvious later.
- **Place imports at the top of a file.** Python allows an `import` almost anywhere, but grouping them at the top makes a file's dependencies obvious at a glance.
- **Prefer the standard library first.** Reach for `pip install` only when the standard library genuinely cannot do the job — fewer installed dependencies means fewer things that can break or conflict later.

Two anti-patterns are worth naming directly, because they are the two most common mistakes freshers make: running `pip install` with no virtual environment active, which quietly changes the one shared system-wide copy of a package that some unrelated project may depend on; and creating a virtual environment but forgetting to activate it, so every `pip install` still lands system-wide even though an isolated folder was created for nothing [1].

## 9. Hands-On Exercise

Create a file, `converter.py`, with one function, `celsius_to_fahrenheit(celsius)`, that returns `celsius * 9/5 + 32`. In the same file, write a test function `test_celsius_to_fahrenheit()` with at least two `assert` statements — one normal case (such as `0` degrees Celsius) and one edge case (such as a negative temperature). Run `pytest converter.py` from a terminal and confirm it reports every test as passed; then deliberately break the function (for example, forget the `+ 32`) and re-run `pytest` to see exactly how a failure is reported.

## 10. Key Takeaways

- A **module** is a single `.py` file of reusable code; a **package** bundles related modules together as one installable, importable unit.
- `import x`, `from x import y`, and `import x as y` are three ways to bring code into a file — the whole module with a dot, one specific item without a dot, or either one under a renamed alias.
- The **standard library** ships with Python for free; **`pip`** installs anything beyond that from **PyPI**, and pinning exact versions keeps a project's behavior identical across machines and time.
- A **virtual environment** must be created *and* activated before installed packages land inside it instead of the shared system-wide Python — this is what lets two projects depend on different, even conflicting, package versions without breaking each other.
- **Pytest** automatically discovers functions named `test_*` and runs their `assert` statements, reporting exactly which checks passed and which one broke — turning "did I break something?" into an automatic, repeatable answer instead of a guess.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
