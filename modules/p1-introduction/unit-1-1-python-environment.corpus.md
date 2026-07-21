---
topic_id: 9ac73bbb-9280-4975-a311-3079a78c38c4
title: Unit 1.1 - The Python Environment
position_in_module: 1
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 1.1 - The Python Environment — Topic Corpus

## 2. Prerequisites

_No formal prerequisites._

## 3. Learning Objectives

By the end of this topic, a learner will be able to:

- **Explain** why Python is widely used in the software industry, in terms of readability and its ecosystem of ready-made libraries.
- **Differentiate** an interpreted language from a compiled language, describing how each turns typed instructions into a running result.
- **Describe** what an interpreter does and what "interactive mode" (the REPL) means.
- **Identify** the three parts of Google Colab relevant to a beginner — the notebook, the cell, and the runtime — and state what each one is for.
- **Apply** the `print()` function inside a Colab cell to produce and read text output.
- **Apply** the restart-and-rerun habit to recover from a cell that appears stuck or a runtime that behaves unexpectedly.

## 4. Introduction

Imagine you have just joined your first job, and on day one your laptop shows a blank screen with a blinking cursor. Before you can write anything useful, two questions have to be answered: what language will you type your instructions in, and where will those instructions actually be carried out so you can see a result? This topic answers exactly those two questions — nothing more.

The language is **Python**, and the place it will run is **Google Colab**, a free tool that runs entirely inside your web browser. There is no software to install and no setup to configure — you open a web page, sign in with a Google account, and you can be running code within a minute. This matters for an absolute beginner because it removes an entire category of early frustration (installation errors, mismatched versions, "it works on my machine") before you have written a single instruction.

It is tempting, as a beginner, to want to jump straight to "writing programs." But every line of code you will ever write has to pass through some piece of software that actually carries out its instructions on the machine. If you do not know what that piece of software is or how it behaves, later mistakes will feel mysterious and unfixable. So before anything else, this topic builds a mental model of how your typed instructions become a visible result on the screen, and gives you hands-on practice with the one tool — Colab — you will use for every exercise from here on.

## 5. Core Concepts

**Programming language.** A programming language is a formal, precise set of words and symbols that lets a person give instructions to a computer. Ordinary human languages like English allow the same sentence to be read in more than one way; a programming language does not — every correctly written instruction has exactly one meaning. **Python** is one such language. It was designed to be easy to read and easy to write, while still being powerful enough to run banking systems, e-commerce platforms, and AI systems [1].

**Source code.** Source code is simply the instructions you type, written out as plain text, following the rules of a particular programming language. When you type `print("Hello, world!")`, that line of text is source code — it means nothing to the computer's hardware by itself; something else has to read it and act on it.

**Why Python specifically.** Two plain-language reasons explain why Python is usually the first language taught today. First, readability: Python code reads close to plain English, so a beginner spends their effort learning to think through a problem step by step, rather than fighting unfamiliar punctuation. Second, ecosystem: an **ecosystem**, in this context, means the large collection of ready-made, reusable code — often called libraries — that other programmers have already written and shared publicly. Python has an unusually rich ecosystem for data analysis, automation, and AI work, so a task that might take days to build from scratch in another language often takes a few lines in Python because someone has already written and shared that logic [1].

**Interpreter.** An interpreter is a program that reads your Python source code and carries out its instructions one line at a time, in the order they appear. Think of it as a very literal-minded assistant who reads your instructions one sentence at a time and immediately does what each sentence says, rather than reading the whole page first.

**Compiler, for contrast.** A compiler is a different kind of program: instead of running your instructions as it reads them, it translates your *entire* set of instructions, ahead of time, into a separate file that the machine can run later (languages like C and Java typically work this way) [1]. The practical, beginner-relevant difference is timing and feedback: with a compiled language, you must finish translating (compiling) before you can test anything; with an interpreted language like Python, you can test a single line and see a result immediately. For day-to-day purposes in this course, it is accurate and expected to describe Python as an **interpreted language** [1].

**Interactive mode (the REPL).** Interactive mode is a way of using the interpreter where you type one instruction, immediately see its result, and then type the next instruction. The common short name for this cycle is the **REPL**, an acronym (a word formed from the first letters of several words) standing for Read, Evaluate, Print, Loop: the interpreter *reads* what you typed, *evaluates* (runs) it, *prints* the result, and *loops* back to wait for your next instruction [1]. This tight loop — type something small, see what happens, then type the next thing — is the rhythm you will use throughout this entire course.

**Google Colab.** Google Colab is a free, browser-based tool from Google that lets you write and run Python code without installing anything on your own computer. Because it runs on Google's servers rather than your laptop, the same notebook and the same behavior are available from any device once you sign in with your Google account [1].

**Notebook.** A notebook is the actual document you create inside Colab — the file where your code, your typed notes, and your output all live together. It carries the file ending `.ipynb`, short for "IPython Notebook," which is simply the file format Colab uses [1].

**Cell.** A cell is a single box inside a notebook that holds either code or ordinary written text. Cells are run individually — you click a "play" button or press a keyboard shortcut to run just that one cell — and each cell's output appears directly beneath it once it finishes running [1].

**Runtime.** The runtime is the live Python session running on Google's servers that remembers everything you have run so far in your current session — for example, if a later cell depends on something an earlier cell set up, the runtime is what "remembers" that earlier work. If a notebook starts behaving strangely, restarting the runtime clears this memory completely and gives you a clean slate, though it means you must rerun your cells afterward [1].

**The `print()` function.** `print()` is a built-in function — meaning it already exists in Python and you do not have to create it yourself — that displays whatever you give it on the screen. Writing `print("Hello, world!")` tells Python: take the text between the quotation marks and show it as output. The word `print` names *which* action to take; the parentheses `(` and `)` mark where you hand information *into* the function; whatever is placed inside those parentheses is called an **argument** [1].

**String (string literal).** A string is a piece of text data, written between matching quotation marks — either `"double quotes"` or `'single quotes'`, as long as the opening and closing quote match. The quotes are what tell Python "treat this exactly as literal text, not as an instruction to figure out." Without quotes, Python would try to treat the words themselves as code it needs to recognize, and would fail with an error [1].

**How the pieces fit together.** When you type `print("Hello, world!")` into a Colab cell and run that cell, the following happens in order: you (the human) supply the source code; the Python interpreter running inside your Colab runtime reads that one line; it recognizes `print` as a function call and `"Hello, world!"` as the string argument to display; it carries out the instruction; and the result appears as output directly below the cell. This is the entire "code becomes output" loop for this topic — no branching, no repeated instructions, no stored values yet, just one instruction executed and one result shown.

## 6. Implementation

Getting from "nothing open" to "my first program ran" follows a fixed, repeatable procedure. This is the only procedural skill in this topic, so it is worth stating as explicit steps:

1. Open a web browser and go to `colab.research.google.com`.
2. Sign in with a Google account (the same kind of account used for Gmail).
3. From the **File** menu, choose **New notebook**. This opens a notebook with one empty cell.
4. Click inside the empty cell and type a line of source code, for example `print("Hello, world!")`.
5. Run the cell — either click its ▶ Run button, or press **Shift + Enter**.
6. Read the output that appears directly beneath the cell, and confirm it matches what you expected.
7. If a cell appears stuck, or the runtime is behaving unexpectedly, restart the runtime and rerun your cells from the top, rather than guessing at the cause.

Each step depends on the one before it — for example, there is no cell to type into until a notebook exists, and there is no output to read until the cell has actually been run. "Typing" a program and "running" it are two separate actions; nothing happens on the screen from typing alone.

## 7. Real-World Patterns

The exact loop practiced above — write an instruction, run it, read the result — is not a "toy" beginner exercise; it is the same basic shape used inside real, professional software, just operating at a larger scale [1]. A banking backend that processes a transaction, or a UPI payment app that shows "Payment Successful," is running Python (or a similar language) through an interpreter, executing instructions in order, and producing a result — the result is just written to a database or sent back over the internet instead of printed to a screen. Tools that automatically grade student code — including tools that may check your own coursework — work the same way: they run your Python code through an interpreter and compare its output to an expected result. Every AI system you will later use or build is, underneath, ordinary Python code that runs and produces output, following this same execute-and-observe pattern.

## 8. Best Practices

- Run notebook cells **top to bottom**, in order, especially while learning — running cells out of sequence is one of the most common sources of "it worked a moment ago" confusion.
- Give notebooks a meaningful name (for example, `unit-1-1-first-program`) instead of leaving the default name — this saves time later when you need to find a specific notebook again.
- Read the output of every cell you run, even simple ones. Building a "run, then read" habit now prevents much larger debugging headaches later.
- If a runtime feels stuck or behaves unexpectedly, restart it and rerun from the top rather than guessing at the cause — it is a cheap, reliable reset.
- Save work to Google Drive frequently rather than relying on a single open browser tab as the only copy (Colab typically saves automatically, but manual saving with **Ctrl + S** is a safe habit).
- Do not confuse writing code with running code — a program produces no output at all until its cell is actually executed.

## 9. Hands-On Exercise

In a new Colab cell, use only `print()` and string literals to build a small "first day" card about yourself: (a) print one line welcoming yourself by name; (b) add two more lines printing your name and your course/batch, so the cell prints three lines total; (c) add a simple header and footer line made of repeated `=` or `-` characters, so the final output looks like a small formatted card. Run the cell after each change and confirm the output matches what you expect before adding the next line.

## 10. Key Takeaways

- Python is widely used in industry because it is readable and has a rich ecosystem of ready-made libraries, especially for AI and data work.
- Python is accurately described, for practical purposes, as an interpreted language — the interpreter carries out source code one line at a time, without a separate compile step the programmer manages.
- Interactive mode (the REPL) means Read, Evaluate, Print, Loop — one instruction is run and its result shown before the next instruction is typed.
- Google Colab provides a notebook made of cells, running inside a live runtime on Google's servers, with no installation required.
- `print()` is a built-in function that displays whatever argument it is given; text arguments must be a string — wrapped in matching quotation marks.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
