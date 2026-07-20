# Python Foundations — Study Guide

*Consolidated revision guide for Part A. This will be filled in as each
module (P1–P5) is completed.*

## P1 · Module I · Introduction

**Quick recap:**

- Python is interpreted, run line by line, and used in Google Colab in the
  browser — no installation needed.
- Four basic types: `int`, `float`, `str`, `bool`. Check any value's type
  with `type()`.
- Arithmetic operators follow BODMAS-style precedence; use `()` to control
  order.
- `==` compares values; `=` assigns a value — do not confuse the two.
- f-strings (`f"{variable}"`) are the standard way to build formatted
  output text.

## P2 · Module II · Control Structures, Functions & Tooling

**Quick recap:**

- `if`/`elif`/`else` for decisions; ternary form: `x if condition else y`.
- `for` loops iterate over sequences; `while` loops run until a condition
  becomes false. Use `range()`, `enumerate()`, and `zip()` to control what
  you iterate over. `break` exits a loop early, `continue` skips to the
  next iteration.
- Functions are defined with `def`; parameters can be positional, keyword,
  default, or captured with `*args`/`**kwargs`. Recursion needs a base case.
- Lambdas are short anonymous functions, often used as a `sorted(key=...)`
  argument. Decorators (`@decorator`) wrap a function's behaviour.
  Generators use `yield` to produce values lazily.
- Professional tooling: `pip` installs packages, `venv` isolates project
  dependencies, Poetry manages projects, Pytest runs tests.

## P3 · Module III · Data Structures

**Quick recap:**

- Four core collections: `list` (ordered, mutable), `tuple` (ordered,
  immutable), `set` (unique, unordered), `dict` (key-value pairs).
- Slicing (`a[start:stop:step]`) and comprehensions (`[x for x in ...]`)
  work across lists, sets, and dicts.
- Tuple unpacking (`a, b = b, a`) is the standard way to swap values.
- `Counter` and `defaultdict` from the `collections` module solve common
  counting/grouping problems without extra `if` checks.

## P4 · Module IV · Classes & Objects

**Quick recap:**

- A class is a blueprint; an object (instance) is one thing built from it.
  `__init__` sets up instance attributes; `self` refers to the current
  instance inside a method.
- Inheritance (`class Child(Parent)`) reuses and extends behaviour;
  `super()` calls the parent's version of a method.
- Dunder methods like `__str__`, `__repr__`, and `__eq__` customise how
  objects are displayed and compared. `@dataclass` removes boilerplate for
  simple data-holding classes.

## P5 · Module V · Files & Exception Handling

**Quick recap:**

- Always open files with a `with` block — it closes the file automatically,
  even if an error occurs.
- `csv` and `json` modules handle structured file formats without manual
  string parsing.
- `try`/`except`/`else`/`finally` handles errors; catch specific exceptions
  (`FileNotFoundError`, `ValueError`, etc.) rather than a bare `except`.
- A robust file reader logs bad rows and keeps processing valid ones,
  instead of crashing on the first bad row.

## P6 · Git, GitHub & Portfolio

**Quick recap:**

- Git tracks changes to your code over time; the basic loop is
  repository → commit → push.
- GitHub Classroom is how the cohort's assignment repos are distributed and
  submitted.
- Keep commits small and descriptive; maintain a personal portfolio repo
  with a clear README — this is used across the whole programme.
- The Part A diagnostic (a Python script reading a CSV, committed and
  pushed to GitHub) gates entry into Part B.
