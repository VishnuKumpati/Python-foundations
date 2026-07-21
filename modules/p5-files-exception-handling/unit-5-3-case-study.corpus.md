---
topic_id: a902447a-e5e3-4ff5-b531-fc737c1fc624
title: Unit 5.3 - Case Study
position_in_module: 3
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 5.3 - Case Study — Topic Corpus

## 2. Prerequisites

This topic combines, and depends completely on, two earlier units — it introduces no new syntax of its own:

- **Unit 5.1 - File Handling** (id: `24960c98-315e-402d-9539-10ed668e7a80`) — you must already be comfortable with `open(filename, mode)`, the file object it returns, the `with` statement (a context manager that closes a file automatically), and `csv.reader()` for turning a CSV file into rows.
- **Unit 5.2 - Errors & Exceptions** (id: `9c02fab8-78bc-45e9-bce7-a8fcf1522422`) — you must already be comfortable with `try`/`except`, catching a specific exception like `ValueError` instead of a bare `except:`, and using `raise` to trigger an exception yourself.

If either of those feels shaky, this unit will be hard to follow, since every line of code here is built directly out of them.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** what a "robust file reader" is and why real-world data files almost always contain at least a few bad rows.
- **Differentiate** between a fail-fast approach (stop at the first bad row) and a fail-soft, "skip and log" approach (keep going and record the problem), and identify which situation calls for which.
- **Apply** the pattern of sorting every row into one of two separate collections — valid records and invalid records — instead of mixing successes and failures together.
- **Implement** a CSV reader that places `try`/`except` inside a `for` loop so that one bad row never stops the rest of the file from being processed.
- **Debug** the two most common mistakes in this pattern: crashing on the first bad row, and silently losing bad rows with no record of what happened.

## 4. Introduction

Imagine you are given a spreadsheet of a thousand student exam scores and asked to write a program that reads it and calculates the class average. You write a simple loop, run it on the first ten rows during testing, and it works perfectly. Then you run it on the real file — and on row 347, someone typed the word "absent" instead of a number. Your program crashes. Not only do you lose row 347; you also lose the 653 perfectly good rows that came after it, because the program never got that far.

This is not a rare or unlucky scenario — it is the *normal* condition of any file that comes from outside your own program. Someone typed it, another system exported it, or a device recorded it, and somewhere in a large file, a few rows will be broken: a blank field, a stray text value where a number belongs, an amount that doesn't make sense. A program that assumes every row is perfect is a program that has not yet met real data.

This unit does not teach you any new Python syntax. Instead, it takes two things you already know — reading files with `open()` and `csv.reader()` from Unit 5.1, and catching errors with `try`/`except` from Unit 5.2 — and shows you the one disciplined way to combine them so that a single bad row can never take down the rest of the file. By the end, you will be able to build the kind of file reader that real, professional systems actually use.

Think of it like a busy admissions office sorting a stack of paper application forms. A careless clerk who throws the whole stack in the bin the moment one form is filled out wrong would be useless — the office needs every complete form processed, and a separate, labeled pile for the incomplete ones so someone can follow up. That is precisely the job this unit's pattern does for data files: sort, don't stop.

## 5. Core Concepts

### 5.1 What "robust" means here

A **robust file reader** is a program that reads a data file row by row, tries to convert and check each row's data, and — instead of stopping the instant one row fails — keeps going through the entire file, sorting every row into one of two outcomes: rows that worked, and rows that didn't [1]. "Robust" simply means "does not break easily" — a robust reader survives bad input instead of crashing on it.

A **valid record** is a row that was read, converted to the right data type (for example, turning the text `"78"` into the number `78`), and passed every check — it is now safe to use in the rest of the program [1]. An **invalid record**, also called a **malformed row**, is a row that failed to convert (for example, the text `"eighty"` cannot become a number) or that converted fine but broke a rule the program cares about (for example, a mark of `150` out of 100) [1]. Every row in the file must end up in exactly one of these two groups — never both, and never neither.

### 5.2 Fail-fast versus fail-soft

There are two fundamentally different ways a program can react to a bad row.

**Fail-fast** means the program stops immediately the moment something goes wrong, raising the exception right there and refusing to continue [1]. This is the *default* behavior of Python when you don't use `try`/`except` at all — one error ends the whole run.

**Fail-soft**, also called **skip and log**, means the program notices the problem, records it, and keeps going, only finishing once it has looked at every row in the file [1]. "Skip" means the bad row is set aside instead of used; "log" here simply means keeping a note of what happened — recording which rows were rejected and why, instead of letting that information vanish the moment the program moves on [1].

Neither approach is "correct" in every situation — they suit different jobs. Fail-fast is the right choice for a single, critical action that must not proceed on bad data at all — for example, validating one bank transfer before it executes; if the amount is wrong, you want that one operation to stop right there, not to guess and continue. Fail-soft is the right choice for **batch** processing — working through many independent rows in one run, such as a CSV of a thousand bookings — where one bad row has nothing to do with the row next to it, and losing 999 good rows because of one bad one would be a far worse outcome than just setting the bad one aside.

### 5.3 Graceful degradation

**Graceful degradation** is the general idea behind fail-soft: a program continuing to produce useful, partial results even when part of its input is broken, rather than stopping entirely [1]. A payroll program that refuses to pay 200 employees because one employee's row has a typo is not behaving gracefully; a payroll program that pays the 199 correct rows and clearly flags the one broken row for a human to fix, is.

### 5.4 Business rules versus conversion failures

Not every bad value fails to *convert*. Converting the text `"150"` to a number with `int("150")` succeeds without any error — Python has no way of knowing, on its own, that 150 is not a possible exam mark out of 100. A rule like "marks must be between 0 and 100" or "a fare must be greater than 0" is called a **business rule** — a condition specific to your program's meaning of the data, not a rule Python's own type conversion enforces. You already know how to enforce a business rule: `raise` your own exception (Unit 5.2) when the check fails, using the *same* exception type (typically `ValueError`) that the conversion itself would have raised on bad text. That way, one `except ValueError:` block downstream catches both kinds of failure — a conversion that failed outright, and a conversion that succeeded but broke a rule — without needing two separate except blocks.

It helps to notice that this pattern produces two distinct causes of the same visible outcome. A row with the text `"eighty"` in the marks column fails at the moment `int(row[1])` runs — Python itself raises the exception, because the text simply cannot become a number. A row with `"150"` in the marks column does not fail at that point at all; `int("150")` returns the number `150` with no complaint. It fails one line later, at the `if not (0 <= marks <= 100):` check, because *you* decided 150 is out of bounds and told Python to `raise ValueError` yourself. Both rows end up in `invalid_records` for the same reason from the outside — they broke a rule — but the two failures happen for genuinely different reasons on the inside. Understanding this distinction is what separates "my code sort of works" from actually understanding why the `except` block catches what it catches.

### 5.5 Why the placement of `try`/`except` matters

The entire pattern hinges on exactly where the `try`/`except` sits. If it wraps the *whole* `for` loop, a single bad row still raises an exception that escapes the loop entirely, and every row after it is never reached — this is fail-fast in disguise, even though `try`/`except` is technically present. If it wraps only the processing of *one row at a time*, sitting **inside** the loop's body, a failure on that one row is caught, the loop's next iteration still runs, and the file is read to the end regardless of how many rows fail. This placement — inside the loop, around one row — is the single structural idea that makes a reader robust.

## 6. Implementation

Building a robust reader follows the same sequence every time, using only pieces you already know from Units 5.1 and 5.2:

1. Create two empty lists before the loop starts: one for valid records, one for invalid records.
2. Open the file with `with open(path, "r", ...) as file:` — the same context manager from Unit 5.1, which closes the file automatically even if a row later raises an error.
3. Wrap the file with `csv.reader(file)` (or `csv.DictReader(file)`) to turn each line into a row.
4. If the file has a header row, skip it with `next(reader)` before the loop begins.
5. Loop over the rows with `for row in reader:`.
6. Inside the loop, pull out any values that *cannot* fail (for example, a name, just `row[0]`) before the `try` block — there is no need to protect code that cannot raise an exception.
7. Wrap only the risky part — the conversion (`int(...)`, `float(...)`) and any business-rule check — inside a `try:` block.
8. If a business rule is violated after a successful conversion, `raise` the same exception type the conversion would have raised, so one `except` block below catches both cases.
9. On success, `append()` the row's data to the valid-records list. In the matching `except ValueError:` block, `append()` the original row to the invalid-records list.
10. After the loop ends (meaning every row in the file has now been looked at), report a summary: how many rows were valid, how many were invalid, and any calculation — such as an average — computed **only** over the valid-records list.

The skeleton looks like this:

```python
valid_records = []
invalid_records = []

with open("data.csv", "r", newline="") as file:
    reader = csv.reader(file)
    next(reader)                    # skip header row
    for row in reader:
        name = row[0]
        try:
            marks = int(row[1])
            if not (0 <= marks <= 100):
                raise ValueError(f"marks {marks} out of range")
            valid_records.append((name, marks))
        except ValueError:
            invalid_records.append(row)

total = len(valid_records) + len(invalid_records)
print(f"Processed {total} rows: {len(valid_records)} valid, {len(invalid_records)} invalid.")
```

Walking through why each part matters: `name = row[0]` sits outside the `try` because reading a value out of a list by position cannot itself raise an exception here — only the conversion of `row[1]` can. The `if not (0 <= marks <= 100): raise ValueError(...)` line only runs after the conversion has already succeeded, so it is checking the business rule, not the data type. Because both the conversion and the business-rule check can only produce a `ValueError`, a single `except ValueError:` block cleanly catches either kind of problem, and the row is added to `invalid_records` either way, preserving its original, unmodified content for later review.

A further refinement — still no new syntax — is writing the rejected rows to their own file instead of only printing them, using `csv.writer()` from Unit 5.1: open a second file for writing, write a header row, then write every row from `invalid_records` at once. This turns the rejected data into something a person can open and review later, rather than output that disappears the moment the program's window closes.

The exact same shape works with `csv.DictReader()` from Unit 5.1 instead of `csv.reader()` — the only difference is that each `row` becomes a dictionary keyed by column name (for example `row["Marks"]`) instead of a list indexed by position (`row[1]`), so the risky line inside `try` would read `int(row["Marks"])` instead of `int(row[1])`. Everything else about the pattern — where the `try`/`except` sits, the two separate lists, the summary at the end — stays identical, because this is a structural pattern applied on top of file reading, not a feature tied to one specific way of reading a row.

## 7. Real-World Patterns

This exact "skip and log" pattern shows up wherever a real system has to accept data from the outside world, because outside data is never perfectly clean:

- **Railway or travel booking systems** process a bulk upload of bookings, letting every valid fare and passenger count through while routing rows with a missing or invalid fare into a rejection report for a staff member to review later.
- **Banking and payment systems** run an overnight batch job that separates out any transaction row that doesn't match the expected format into a review queue, instead of letting one malformed transaction halt the entire night's processing.
- **E-commerce catalog uploads** accept every well-formed product row and report back the rows with a missing price or invalid category, rather than rejecting an entire catalog file over one bad line.
- **Healthcare data imports** log any patient record row with an invalid date of birth or missing ID, while still importing every record that is complete and correct.
- **AI and machine learning data pipelines** reading a large training dataset skip and log malformed or incomplete rows rather than halting a multi-hour training run because of a handful of corrupted lines — this is the same pattern that keeps a much bigger, more expensive job running.

Data-engineering teams have a name for this pattern — "skip and log" or "graceful degradation" — and it is considered a baseline expectation for any program that processes files at scale, not an advanced technique.

## 8. Best Practices

- **Never let one bad row take down the whole batch.** A malformed row is an ordinary, expected event in real data — not an emergency that should end the program.
- **Always report what was skipped and why.** Silently discarding a bad row is just as harmful as crashing, because either way the data is gone with no trace it ever existed.
- **Keep valid and invalid results in separate collections.** Never merge them, and never run a calculation like an average or total over a collection that mixes the two — a rejected row that carries no real number can quietly distort the result if it's counted anyway.
- **Catch the specific exception you expect** (typically `ValueError`), not a bare `except:`. A bare `except:` also hides real bugs in your own code, such as a typo in a variable name, and you would never find out.
- **Don't assume a successful conversion means a valid value.** `int("150")` succeeds without complaint, but 150 may still break a business rule your program cares about — only your own check, enforced with `raise`, catches that.
- **Make sure the reader finishes the whole file regardless of how many rows fail.** If it stops early for any reason, it is not actually robust, no matter how much `try`/`except` code it contains.
- **Check the placement of `try`/`except` before checking anything else.** If a reader still fails on the second row after a first bad row, the very first thing to check is whether the `try`/`except` accidentally wraps the whole `for` loop instead of a single iteration of it — this one placement mistake explains the large majority of "robust" readers that turn out not to be.

## 9. Hands-On Exercise

Given a CSV file `orders.csv` with columns `Customer,Amount`, where some rows have a blank amount, some have non-numeric text, and one has a negative amount: write a program that (1) reads the file and separates rows into `valid_records` and `invalid_records`, treating an amount as valid only if it converts to a number and is greater than 0; (2) prints a summary count of valid versus invalid rows; (3) prints the total of all valid amounts, computed only over `valid_records`. Confirm that a negative amount is correctly rejected even though it converts to a number without error, and that the program still reaches and processes every row after each bad one. As a stretch goal, write `invalid_records` out to a second CSV file, `invalid_orders.csv`, with its own header row, using `csv.writer()` from Unit 5.1.

## 10. Key Takeaways

- A robust file reader processes every row it can, sets aside every row it can't, and reports both outcomes — it never crashes on the first bad row and never silently loses bad data either.
- This topic introduces no new syntax: it is `with open()` and `csv.reader()` from Unit 5.1 combined with `try`/`except` and `raise` from Unit 5.2, arranged in one disciplined shape.
- The structural rule that makes this work is placing `try`/`except` **inside** the loop, around one row at a time — not around the entire loop.
- Valid records and invalid records must live in two separate collections, and any summary calculation must run only over the valid ones.
- Fail-soft ("skip and log") suits processing many independent rows in a batch; fail-fast still has its place for a single, critical operation that must not proceed on bad data.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
