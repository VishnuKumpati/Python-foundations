# The Python Environment

---

## What is Python?

Python is a programming language. It's simple to read, simple to write, and it's the language most used today for AI, data work, and automation.

You write instructions in Python, a program called an **interpreter** runs them, and you see the result. That's it. That loop, write, run, see result, is what this whole unit is about.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Interpreter** | Reads your code and runs it, one line at a time |
| **Interactive mode / REPL** | Type a line, see the result immediately, type the next |
| **Notebook** | A document made of cells, where you write and run code |
| **Cell** | A single box in a notebook, holds code or text |
| **Runtime** | The live Python session that remembers what you've run so far |
| **String** | Text in quotes, e.g. `"Hello, world!"` |

---

## Why Python?

- It reads almost like English. `print("Hello, world!")` doesn't need much explaining.
- It has a huge collection of ready-made code (called **libraries**) that other people have already written, for AI, data, web apps, almost anything.
- It's the top language in Indian and global IT hiring right now.

Here's the readability point in action, same output, two languages:

```java
// Java
System.out.println("Hello, world!");
```

```python
# Python
print("Hello, world!")
```

No semicolon, no `System.out.`, no extra typing. That's what people mean by "low syntax overhead."

---

## Compiled vs Interpreted

There are two ways a computer can turn your code into something that actually runs:

| | Compiled (C, Java) | Interpreted (Python) |
|---|---|---|
| When it's translated | All at once, before running | Line by line, while running |
| Extra file created? | Yes (`.exe`, `.class`) | Yes, but hidden: a `.pyc` bytecode cache, usually in `__pycache__` |
| Test a quick change | Recompile first | Just run it again |

Python is called an **interpreted language**. In reality, Python first turns your code into something called **bytecode**, and then runs that, but for everyday use it's fine to just say "Python is interpreted."

That `.pyc` file is just a cache. It speeds up the *next* run by skipping the bytecode step if the source hasn't changed. You never create or manage it yourself, and Colab handles this invisibly, so you won't see it in a notebook. It only shows up if you run `.py` files directly on your own machine.

---

## What is a REPL?

**REPL** = **R**ead, **E**valuate, **P**rint, **L**oop.

You type one line, Python runs it, shows you the result, and waits for your next line. That's interactive mode.

```
 ┌──────┐      ┌──────────┐      ┌───────┐
 │ Read │ ───► │ Evaluate │ ───► │ Print │
 └──────┘      └──────────┘      └───┬───┘
     ▲                               │
     │                               │
     └─────────── Loop ◄─────────────┘
```

- **Read**: you type a line of code
- **Evaluate**: Python runs it
- **Print**: the result shows up
- **Loop**: back to Read, ready for your next line

This is exactly what happens in Google Colab, one cell at a time.

---

## Getting Started with Google Colab

Colab is a free tool from Google. It runs Python in your browser, nothing to install.

**Steps:**

1. Go to `colab.research.google.com`
2. Sign in with your Google account
3. You'll land on the **Welcome to Colab** screen. Just close the popup, you don't need the example notebooks yet

<img src="../../Images/image%20%289%29.png" alt="The Welcome to Colab page, showing the toolbar of an empty text cell highlighted in red" width="480">

4. Click **File → New notebook**

<img src="../../Images/image%20%288%29.png" alt="The same File menu with New notebook highlighted (dark theme)" width="480">

5. Rename it (click the default name at the top, type something like `unit-1-1-first-program`)
6. Click into the empty cell and type your code
7. Run it, either click the ▶ button, or press **Shift + Enter**
8. Read the output below the cell

Colab saves your work automatically to Google Drive. If it looks stuck, refresh the page.

> No Colab access right now? You can try the code straight in your browser at [programiz.com/python-programming/online-compiler](https://www.programiz.com/python-programming/online-compiler), no sign-in needed.

---

## Your First Program

```python
print("Hello, world!")
```

Run it. You'll see:

```
Hello, world!
```

Breaking it down:

- `print()`: a built-in function. It means "show this on screen."
- `"Hello, world!"`: a **string**. Text in quotes.
- No semicolon needed. Python uses new lines, not `;`.

**Rules to remember:**
- Quotes must match: `"like this"` or `'like this'`, not mixed.
- `print` and `Print` are different to Python. Case matters.
- Typing code does nothing until you *run* the cell.

---

## Try it Yourself

Print more than one line, just call `print()` again:

```python
print("Welcome to your first day!")
print("Employee Name: Aditi Sharma")
print("Employee ID: EMP-2026-0143")
```

Output:

```
Welcome to your first day!
Employee Name: Aditi Sharma
Employee ID: EMP-2026-0143
```

Each `print()` is its own line. Python runs them top to bottom, in order.

Now make it look like a receipt, using `=` as a border:

```python
print("======================================")
print("Employee Name: Aditi Sharma")
print("Employee ID: EMP-2026-0143")
print("Status: ACTIVE")
print("======================================")
```

**Your turn:** using only `print()`, build the same kind of card for yourself:

- Print a line welcoming yourself to your first day
- Add your name and your batch/course as two more lines
- Add a border above and below using `=` or `-`

No new syntax needed, just more `print()` lines, same as above.

---

## Common Mistakes

- Forgetting quotes: `print(Hello)` instead of `print("Hello")`
- Mixing quote types: `'text"`
- Editing a cell but forgetting to run it again
- Not reading the error message. Python usually tells you exactly what's wrong

---

## Interview Questions

**Q1: Is Python compiled or interpreted?**

A: Interpreted. Python runs your code line by line. Under the hood, it first turns your code into bytecode and then runs that. Mentioning this shows you know it beyond the surface-level answer.

**Q2: What does REPL stand for?**

A: Read, Evaluate, Print, Loop. You type a line, Python runs it, shows the result, and waits for your next line.

**Q3: Why is Python popular for AI and data work?**

A: Two reasons: it's easy to read and write, and it has a huge ecosystem of ready-made libraries for AI, data analysis, and automation.

---

## Quick Recap

- Python is interpreted: it runs line by line.
- REPL = Read, Evaluate, Print, Loop.
- Colab is a free, browser-based place to write and run Python.
- `print()` displays whatever string you give it.
- Run cells top to bottom, and always read the output.
