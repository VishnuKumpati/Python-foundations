# The Python Environment

Every semester, lakhs of first-year engineering students across India sit down to write their first computer program, and nearly all of them ask the same two questions: which language should I learn, and where do I actually type it? This chapter answers both. The language is Python — the single most in-demand programming language in Indian and global software teams today, used to build everything from UPI payment apps to the recommendation systems behind e-commerce platforms. The workspace is Google Colab, a free tool that runs entirely inside your web browser, so there is nothing to install and nothing that can be "set up wrong" on your machine.

Before you write anything, it helps to know what actually happens when a computer "runs" your code. You will meet two very different strategies for turning your instructions into action — compiling and interpreting — and see which one Python uses, and why that matters for how you will work day to day. You will then open Google Colab yourself, learn what a notebook, a cell, and a runtime are, and walk through the exact clicks that take you from a blank browser tab to a working notebook. Finally, you will write your first program: one line that prints a message to the screen — and in doing so, you will already be practising two habits that separate confident programmers from frustrated ones: reading error messages carefully, and restarting your environment the moment something feels stuck.

By the time you finish this chapter, "running a program" will no longer be a mysterious phrase. It will be something you have done yourself, watched happen, and understood.

---

## Why the whole industry is standardising on Python

Open any job listing for a software or data role at an Indian IT company — TCS, Infosys, a fintech startup building UPI infrastructure, or a global bank's technology centre in Bengaluru — and Python appears constantly. That is not hype; it is a genuine, industry-wide pattern.

| Where it's used | Concrete example |
|---|---|
| Banking & fintech | Fraud-detection logic, UPI transaction-processing pipelines |
| E-commerce | Product recommendation engines, price-tracking bots |
| Data & AI | Training machine learning models, cleaning large datasets |
| Automation | Scripts that generate reports, send reminders, scrape web data |
| Web backends | Server logic behind apps built with Django or FastAPI |

Two things put Python in all of these places at once. First, its syntax — the set of rules for what counts as valid, correctly written code — reads close to plain English, so you spend less time fighting punctuation and more time solving the actual problem. Second, Python ships with, or can install, ready-made libraries for almost anything: talking to a database, sending an email, training a neural network. You will feel both advantages within your first few weeks.

Pick any one of the five rows above and name an Indian app or company you have personally used that could plausibly rely on Python behind the scenes — a useful gut-check before moving on. Whatever you eventually type, though, has to reach the processor somehow, and that journey can happen in two very different ways.

## Two ways a computer can run your code

A computer's processor understands exactly one language: sequences of 0s and 1s called machine code. The words you type — `print`, `if`, `for` — mean nothing to the processor directly, so something has to stand between your readable code and the machine code the processor executes. Programming languages take one of two broad approaches to that translation.

Think of it the way you would think about translating a speech. One approach is to translate the entire written speech into Hindi in advance and hand over the finished document before anyone speaks a word — that is compiling. The other is to have a live interpreter standing beside the speaker, translating sentence by sentence as the words are spoken — that is interpreting, and it is exactly where Python's interpreter borrows its name from.

**Compiled languages**, such as C or C++, translate your entire program into machine code *before* you run it, producing a separate executable file. **Interpreted languages**, such as Python, translate and run your code line by line, at the moment you execute it, with no separate translation step for you to manage.

| | Compiled language | Interpreted language |
|---|---|---|
| Translation happens | Once, ahead of time, into a standalone file | Every time you run the program, one line at a time |
| Tool that does it | A compiler | An interpreter |
| Feedback speed | Wait for the whole program to compile first | See results almost immediately |
| Example | C, C++ | Python, JavaScript |

Python's line-by-line approach is why you can type a single instruction, run it, and see the result in under a second — there is no separate "build" stage standing between you and your output. It is also why Python is so forgiving to learn in: a mistake shows up close to where you made it, one line at a time, instead of buried inside a wall of compiler messages.

One detail is worth knowing precisely, because interviewers like asking it, and because it trips up learners who over-memorise the "Python is never compiled" rule: Python's standard interpreter — called **CPython** — actually translates your code into a lower-level, in-between form called **bytecode** before running it, which is technically a small compiling step of its own. Day to day, and for the rest of this course, calling Python "an interpreted language" is accurate and is exactly how the industry talks about it — there is still no separate file you compile and manage yourself, unlike C or Java. Try saying, out loud and in one sentence, why Python still counts as interpreted despite this bytecode step — if the sentence comes easily, you have understood the distinction rather than just memorised it.

## Meeting the interpreter

The program that performs this line-by-line translation for Python is called the **Python interpreter**. You can talk to it directly, one instruction at a time, in what is called **interactive mode** (also known as a REPL — Read, Evaluate, Print, Loop): you type a line, press Enter, and the interpreter replies immediately, the same way a calculator answers the instant you press "=".

Before checking, try predicting what would appear directly below `2 + 2` if you typed it into an interpreter and pressed Enter.

```
>>> 2 + 2
4
>>> print("Namaste, world")
Namaste, world
```

Line 1 in each pair is what you type; line 2 is the interpreter's reply. That back-and-forth — one line in, one result out — is exactly what interactive mode means. Notice that the `>>>` is something the interpreter displays for you, not something you type yourself — a mix-up worth avoiding the first time you see it in the official Python documentation. Google Colab, which you will use for this entire course, runs a full Python interpreter behind the scenes, but wraps it in a more convenient notebook interface rather than this bare line-by-line prompt.

Interactive mode is not just a classroom exercise — professional developers reach for it constantly:

- Testing a single calculation or a suspicious line before it goes into a larger program
- Checking what a library function returns before relying on it elsewhere
- Debugging, by running one line in isolation to see exactly where behaviour breaks

## Google Colab: your notebook, cell, and runtime

Google Colab (short for "Colaboratory") is a free, browser-based tool built by Google that gives every student a working Python interpreter with nothing to install. You open a link, sign in with a Google account, and within seconds you have a place to write and run code. Three words describe everything you will interact with inside it.

- **Notebook** — the document itself, saved to your Google Drive, made up of an ordered sequence of cells.
- **Cell** — a single block inside a notebook: either a *code cell* (Python you run) or a *text cell* (notes you write for yourself).
- **Runtime** — the actual machine, running on Google's servers, that executes your code cells and remembers their results in memory.

Getting from a blank browser tab to a working notebook takes a handful of clicks, and after the first time it becomes automatic:

1. Open your browser and go to `colab.research.google.com`.
2. Sign in with a Google account — the same one you would use for Gmail.
3. Click **File → New notebook**; Colab opens a fresh notebook with one empty cell.
4. Click the default title (e.g. `Untitled0.ipynb`) and rename it to something meaningful, such as `unit-1-1-first-program`.
5. Click inside the empty cell and type your Python code.
6. Run it with the ▶ button beside the cell, or by pressing **Shift + Enter**.

**Name your notebooks meaningfully and let Colab auto-save to your Drive as you work — both are cheap habits now that save real time later, especially once you have dozens of notebooks to find again.**

The runtime is the part that trips up beginners, so it deserves a closer look. When you run a code cell, the runtime keeps whatever it produced in memory for your *next* cell to use. This is unlike, say, a Word document, where each paragraph stands alone — in a notebook, cell 5 can depend entirely on something you defined back in cell 2. Restart the runtime and then run only cell 5 without rerunning cell 2 first, and you will see a `NameError` complaining that whatever cell 2 defined no longer exists — this is the single most common Colab mistake in your first few weeks, and the fix is always to rerun from the top.

Do this once for yourself before continuing: open Colab, create a new notebook, rename it, type a single `print()` line into the first cell, and run it — the muscle memory matters more than reading about it.

## Writing your first program

A Python program, at its simplest, is one or more instructions the interpreter carries out in order. The very first instruction nearly every learner writes calls a built-in function named `print()`, whose job is to display whatever you hand it inside the parentheses onto the screen.

Think of a function the way you would think of a bank counter. You fill in a slip with whatever detail you want acted on and hand it through the window — that slip is the function's **argument** — and the person behind the counter performs one specific, predictable action with it. `print()` is a function whose one job is to take whatever you hand it and display it on the screen, nothing more.

```python
print("Namaste, world")
```

Type this into a Colab code cell and run it (Shift+Enter, or the ▶ button beside the cell):

```
Step 1 — Python reads the line print("Namaste, world")
Step 2 — it recognises print as a function call, and "Namaste, world" as the value to hand it
Step 3 — it sends that text to the output area below the cell
Output: Namaste, world
```

Notice the double quotes around the text. In Python, text you want treated as literal characters — not as code to execute — must be wrapped in quotes, and this is called a **string**. Drop the quotes and write `print(Namaste, world)` instead, and the interpreter complains, because now it assumes `Namaste` and `world` are meant to be *names* it should already recognise, not plain text.

**A missing quote, a missing parenthesis, or the wrong case (`Print` instead of `print`) is the single most common reason a beginner's first program fails.**

- Forgetting the closing parenthesis or quote — Python will not run the line until every opening symbol has a matching close.
- Typing `Print` or `PRINT` — Python is case-sensitive, so only the lowercase `print` refers to the built-in function.
- Mixing quote styles, such as opening with `"` and closing with `'` — pick one style per string and stay consistent.
- Typing code into a cell but never running it, then wondering why nothing happened — writing code and running code are two separate steps; a cell does nothing until you execute it.

Before reading further, try predicting the output of `print("Marks:", 95)` on your own, then run it in a Colab cell to check whether you guessed the spacing correctly.

This one function already does real work outside the classroom. Professional developers sprinkle `print()` calls through programs that are misbehaving, just to see what a value looks like at a given point — an everyday technique informally called **print debugging** — and you will lean on exactly this habit once your own programs grow past a single line.

## When the runtime gets stuck

Because the runtime remembers everything you have run so far, it can end up in a confusing state — for instance, from running cells out of order, or from code that loops forever and never finishes. When output stops appearing, or a cell seems to hang indefinitely, resist the urge to keep clicking run.

**When a cell hangs or stops making sense, restart the runtime before trying anything else.**

The fix, in almost every such case, is the same three-step habit every experienced Python programmer reaches for first: open the **Runtime** menu, choose **Restart runtime**, and re-run your cells from the top. This clears the runtime's memory completely and gives you a clean slate, the same way restarting a hung mobile app clears whatever confused state it was stuck in. Treat this as your default first move rather than a last resort — it costs a few seconds and resolves the large majority of "nothing is working" moments you will run into in this course.

The next time a cell genuinely refuses to respond, try this sequence yourself — Runtime menu, Restart runtime, rerun from the top — and notice how quickly the confusion clears.

---

### Key Terminology

- **Machine code** — the raw 0s and 1s a processor executes directly.
- **Compiler** — a tool that translates an entire program into machine code before it runs.
- **Interpreter** — a tool that translates and runs code one line at a time, as it executes.
- **CPython** — the standard, most widely used implementation of the Python interpreter.
- **Bytecode** — the lower-level, in-between form CPython translates your code into before running it.
- **Interactive mode (REPL)** — using the interpreter by typing one instruction, seeing its result immediately, then typing the next.
- **Notebook** — a Colab document containing an ordered sequence of cells.
- **Cell** — a single block within a notebook; either code (Python to run) or text (notes).
- **Runtime** — the remote machine that executes a notebook's code cells and holds their results in memory.
- **String** — text data in Python, written inside a matching pair of quotes.
- **Syntax** — the set of rules that determine what counts as valid, runnable code.

### Mastery Checkpoint

Before moving to Unit 1.2, check that you can answer these without looking back:

1. Why is it accurate to call Python "an interpreted language" even though CPython technically compiles your code to bytecode first?
2. What is the difference between a notebook, a cell, and a runtime in Google Colab?
3. What output does `print("5 + 3")` produce, and why is it different from the output of `print(5 + 3)`?
4. Your Colab cell has been "running" for two minutes with no output. What is the first thing you should try?
5. You typed `print("Hello")` into a cell but nothing appeared below it. Before assuming Python is broken, what should you check first?

### Summary

You now know what stands between the code you type and the processor that runs it, why Python's interpreted, line-by-line approach makes it especially beginner-friendly, and how Google Colab's notebooks, cells, and runtimes fit together into a working Python environment with nothing to install. You have opened Colab yourself, written and traced your first program, and picked up the restart-and-rerun habit — and the naming and saving conventions — that will save you time for the rest of this course. From here, the next step is learning how Python stores the data your programs work with — starting with variables.

### Additional Resources

- [Python Tutorial — official docs: "Using the Python Interpreter"](https://docs.python.org/3/tutorial/interpreter.html)
- [Python 3 Documentation — `print()` built-in function](https://docs.python.org/3/library/functions.html#print)
- [W3Schools — Python Introduction](https://www.w3schools.com/python/python_intro.asp)
- [W3Schools — Python Getting Started](https://www.w3schools.com/python/python_getstarted.asp)
