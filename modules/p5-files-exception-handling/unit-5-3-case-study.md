# Building a Robust File Reader

Unit 5.1 taught you to open a file, read it as CSV rows, and close it safely with `with`. Unit 5.2 taught you to catch a runtime exception with `try`/`except` instead of letting your program die on the spot. Each unit, on its own, showed you one half of a real skill. This unit is a case study — it puts both halves together to build something you will genuinely reuse for the rest of your career: a file reader that doesn't collapse the instant it meets one bad row of data, but quietly notes the problem, keeps working, and hands you back the good data separated cleanly from the bad.

Picture a college exam cell handing you a spreadsheet of a thousand students' marks and asking you to calculate the class average. You write the obvious loop, test it on the first ten rows, and it works. Then you run it on the real file — and on row 347, someone has typed "absent" instead of a number. A program built the way you've built programs so far crashes right there. You don't just lose row 347; you lose the 653 good rows sitting behind it too, because your program never gets that far. This is not bad luck. It is the ordinary condition of any file that came from outside your own code — someone typed it, some other system exported it — and a reader that assumes every row is perfect simply hasn't met real data yet.

Nothing in this unit is new Python syntax. It is `open()` and `csv.reader()` from Unit 5.1, and `try`/`except`/`raise` from Unit 5.2, arranged in one disciplined shape that survives bad rows instead of dying on them. This is also the final unit of Module V — by the time you reach the closing summary, you will have covered files, exceptions, and now the one pattern that ties them together into something you would actually be comfortable shipping.

---

## The gap a "normal" reader leaves wide open

Think about the handful of Indian systems that read files like this every single day. A bank's overnight batch job reconciles thousands of transaction rows exported from another system. An IRCTC-style booking platform ingests a bulk upload of ticket bookings from a travel agent. An e-commerce seller uploads a catalog of five thousand products through a spreadsheet. In every one of these cases, the file was not typed by the program that reads it — it came from somewhere else, written by a human or exported by yet another system — and somewhere in a few thousand rows, a handful will be broken: a blank field, a stray word where a number belongs, a value that converts fine but makes no real-world sense.

A reader written the way you've written code up to now treats every row as a promise that will be kept. The moment one row breaks that promise, the whole read stops. Say out loud, in one sentence, what a college would lose if its results-processing script crashed on student 347 out of 1000 and never touched the other 653 — that lost work, multiplied across every file every system reads, is exactly the problem this unit exists to solve.

## Fail-fast versus fail-soft: two different jobs, two different reactions

There are two fundamentally different ways a program can react when it meets bad data, and neither one is simply "the right way" — each suits a different job.

**Fail-fast** means the program stops the instant something goes wrong, right there, and refuses to continue. This is what Python does by default, with no `try`/`except` at all: one exception, and the whole run ends. **Fail-soft** — also called **skip and log** — means the program notices the problem, makes a note of it, and keeps going, only finishing once it has looked at every row in front of it.

Think of the difference the way you'd think of two very different door policies at a venue. A fail-fast policy is a bouncer who shuts down the entire event the moment one guest shows up without a ticket — nobody else gets in either, no matter how many were waiting behind that one person. A fail-soft policy is a bouncer who simply turns that one guest away at the door and waves the next hundred people straight through — the event carries on exactly as planned, and only the one problem case was ever affected.

| | Fail-fast | Fail-soft (skip and log) |
|---|---|---|
| Reaction to a bad row | Stops immediately, the exception ends the run | Notes the problem, moves to the next row |
| What happens to good rows after the bad one | Never processed | Processed exactly as normal |
| Best suited for | One critical, all-or-nothing operation — validating a single bank transfer before it executes | Batch processing of many independent rows — a CSV of a thousand bookings or orders |
| What goes wrong if used in the wrong place | A single typo destroys an entire night's good work | A genuinely broken transaction could slip through unnoticed if nobody ever reviews the log |

Neither approach is universally correct. You would *want* fail-fast the moment a single UPI transfer is being validated right before it executes — if the amount looks wrong, that one transfer should stop cold, not guess and proceed. You want fail-soft the moment you're processing a batch of a thousand independent rows, where one bad booking has nothing whatsoever to do with the booking next to it, and losing 999 good rows over one bad one would be the far worse outcome. This unit builds the fail-soft version, because reading a data file row by row is exactly the batch situation fail-soft was built for.

## Stage 1: the naive reader, and where it breaks

Suppose your college's exam cell hands you `marks.csv`, a small file of student records:

```
Name,RollNo,Marks
Ananya Roy,101,88
Rohit Sharma,102,eighty
Meera Iyer,103,95
Karthik Nair,104,
Priya Singh,105,150
Arjun Mehta,106,72
```

Three rows here are fine. Three are not, for three different reasons: Rohit's marks column holds text instead of a number, Karthik's marks field is entirely blank, and Priya's marks column holds `150` — a value that will convert to a number without complaint, even though no exam out of 100 can produce it. Keep all three reasons in mind; you'll meet each one again shortly.

A first, naive attempt at reading this file and printing the class average looks exactly like the code you already know how to write from Units 5.1 and 2.2:

```python
import csv

def read_marks(path):
    total = 0
    count = 0
    with open(path, "r", newline="") as file:
        reader = csv.reader(file)
        next(reader)              # skip the header row
        for row in reader:
            name, roll_no, marks = row
            marks = int(marks)
            total += marks
            count += 1
    return total / count

print(read_marks("marks.csv"))
```

```
Traceback (most recent call last):
  ...
    marks = int(marks)
ValueError: invalid literal for int() with base 10: 'eighty'
```

Before reading on, predict for yourself exactly which row kills this program. It's Rohit's — the very second data row — and the moment `int("eighty")` is asked to convert text that isn't a number, Python raises `ValueError` and the entire function stops. Ananya's row already printed correctly in spirit, but you never even see it, because nothing prints until the function finishes — and it never finishes. Meera, Karthik, Priya, and Arjun are never looked at at all.

**A reader that assumes every row is well-formed will eventually meet a file where that assumption is false — and when it does, it doesn't just lose the bad row, it loses every good row still waiting behind it.**

## Stage 2: catching the exception, one row at a time

The fix is not to remove `try`/`except` from your toolkit and hope for the best — it's to place it exactly where Unit 5.2 taught you it belongs: wrapped around the risky part of processing *one row*, sitting inside the loop, not around the loop itself.

```python
import csv

def read_marks(path):
    total = 0
    count = 0
    with open(path, "r", newline="") as file:
        reader = csv.reader(file)
        next(reader)
        for row in reader:
            try:
                name, roll_no, marks = row
                marks = int(marks)
                total += marks
                count += 1
            except ValueError:
                print("Skipping row, could not read marks:", row)
    return total / count

print(read_marks("marks.csv"))
```

```
Skipping row, could not read marks: ['Rohit Sharma', '102', 'eighty']
Skipping row, could not read marks: ['Karthik Nair', '104', '']
101.25
```

This is real progress — the reader survives Rohit's and Karthik's bad rows and reaches the end of the file. But look closely at that final number: **an average of 101.25, on a scale where the maximum possible mark is 100.** Nothing crashed, yet the answer is impossible. Priya's row, `Priya Singh,105,150`, never raised an exception at all — `int("150")` converts perfectly cleanly to the number `150` — so it sailed straight into the total, dragging the average above the maximum score that should even be achievable.

**Catching an exception only protects you from a conversion that fails outright — it does nothing about a value that converts successfully but is still wrong.** That gap is exactly what the next two stages close.

## Stage 3: sorting rows into two separate piles

Right now, this reader only knows how to skip and print — it has no memory of *what* it kept or *what* it threw away, which makes the impossible average above much harder to track down. The next improvement is to stop mixing outcomes together and give every row exactly one of two homes: a list of records that worked, and a list of records that didn't.

```python
import csv

def read_marks(path):
    valid_records = []
    invalid_records = []
    with open(path, "r", newline="") as file:
        reader = csv.reader(file)
        next(reader)
        for row in reader:
            name, roll_no = row[0], row[1]
            try:
                marks = int(row[2])
                valid_records.append((name, roll_no, marks))
            except ValueError:
                invalid_records.append(row)
    return valid_records, invalid_records

valid, invalid = read_marks("marks.csv")
print("Valid:", valid)
print("Invalid:", invalid)
```

```
Valid: [('Ananya Roy', '101', 88), ('Meera Iyer', '103', 95), ('Priya Singh', '105', 150), ('Arjun Mehta', '106', 72)]
Invalid: [['Rohit Sharma', '102', 'eighty'], ['Karthik Nair', '104', '']]
```

Notice that Priya's row is *still* sitting inside `valid_records`, marks of `150` and all. Separating the two lists is genuinely useful — you can now inspect either group on its own, and any later calculation can deliberately run only over `valid_records` — but it hasn't, by itself, fixed the underlying bug. `int("150")` was never going to raise an exception, no matter how carefully you organise where its result lands.

| | Naive (Stage 1) | Skip-and-print (Stage 2) | Sorted into two lists (Stage 3) |
|---|---|---|---|
| Behaviour on a bad row | Crashes, stops entirely | Prints a message, keeps going | Appends to `invalid_records`, keeps going |
| Rows after the bad one | Never reached | Processed normally | Processed normally |
| Record of what failed | None — program is just dead | Only on screen, gone once scrolled past | Kept in a real list you can inspect afterwards |
| Catches a value like `150` marks | N/A | No | No — not yet |

## The rule Python's own conversion can never enforce

`int("150")` succeeding is not a bug in Python — Python genuinely has no way of knowing that a student's marks are supposed to stay between 0 and 100. That ceiling is a fact about *your* problem, not about numbers in general, and a fact like that is called a **business rule**: a condition specific to what the data is supposed to mean, which no built-in conversion function can ever check for you.

You already have the tool for enforcing a business rule — Unit 5.2's `raise`. The trick that makes this pattern click into place is raising the *same* exception type that a failed conversion would have raised on its own:

```python
try:
    marks = int(row[2])
    if not (0 <= marks <= 100):
        raise ValueError(f"marks {marks} is outside the valid 0-100 range")
    valid_records.append((name, roll_no, marks))
except ValueError:
    invalid_records.append(row)
```

The `if not (0 <= marks <= 100):` line only ever runs *after* the conversion has already succeeded — it is checking meaning, not data type. Because both a failed conversion and a failed business rule raise the exact same `ValueError`, one `except ValueError:` block downstream cleanly catches either kind of problem, without you needing two separate `except` clauses for what is, from the outside, the same outcome: this row cannot be trusted.

**A value that converts without error is not automatically a valid value — only your own business-rule check, enforced with `raise`, catches the difference between "this is a number" and "this is a number that actually makes sense here."**

## Stage 4: logging which row failed, and why

A reader that only sorts rows into two piles still leaves you guessing *why* any particular row landed in the reject pile, especially once a file has dozens of failures instead of two. The fix is small: track each row's position in the file, and capture the exact reason it failed alongside it.

```python
import csv

def read_marks(path):
    valid_records = []
    invalid_records = []
    with open(path, "r", newline="") as file:
        reader = csv.reader(file)
        next(reader)
        for row_number, row in enumerate(reader, start=2):
            name, roll_no = row[0], row[1]
            try:
                marks = int(row[2])
                if not (0 <= marks <= 100):
                    raise ValueError(f"marks {marks} is outside the valid 0-100 range")
                valid_records.append((name, roll_no, marks))
            except ValueError as error:
                invalid_records.append(row)
                print(f"Row {row_number} rejected: {row} -> {error}")
    return valid_records, invalid_records

valid, invalid = read_marks("marks.csv")
```

```
Row 3 rejected: ['Rohit Sharma', '102', 'eighty'] -> invalid literal for int() with base 10: 'eighty'
Row 5 rejected: ['Karthik Nair', '104', ''] -> invalid literal for int() with base 10: ''
Row 6 rejected: ['Priya Singh', '105', '150'] -> marks 150 is outside the valid 0-100 range
```

`enumerate(reader, start=2)` numbers each row starting at 2, matching its actual line number in the file — the header occupies line 1, so the first data row, Ananya's, is genuinely line 2. `except ValueError as error` captures the exception object itself, not just the fact that something went wrong, so `{error}` in the message prints the exact text Python (or your own `raise`) attached to it. Notice the three reject messages now tell three genuinely different stories — bad text, a blank field, an out-of-range value — even though all three end up in `invalid_records` for the same structural reason.

Before checking, predict what would happen if `except ValueError:` here were changed to a bare `except:`. It would still catch all three of these rows correctly — but it would also silently swallow a genuine bug elsewhere in your own code, such as a typo like `row[9]` on a three-column file, and you would have no way of telling a real bug apart from an ordinary bad row.

## Stage 5: the complete, robust reader

Put every stage together — two separate lists, a business-rule check, a per-row reason, and a summary computed only over the rows you can actually trust — and you get the finished pattern:

```python
import csv

def read_marks(path):
    valid_records = []
    invalid_records = []

    with open(path, "r", newline="") as file:
        reader = csv.reader(file)
        next(reader)  # skip header
        for row_number, row in enumerate(reader, start=2):
            name, roll_no = row[0], row[1]
            try:
                marks = int(row[2])
                if not (0 <= marks <= 100):
                    raise ValueError(f"marks {marks} is outside the valid 0-100 range")
                valid_records.append({"name": name, "roll_no": roll_no, "marks": marks})
            except ValueError as error:
                invalid_records.append({"row_number": row_number, "row": row, "reason": str(error)})

    return valid_records, invalid_records


valid_records, invalid_records = read_marks("marks.csv")

print(f"Processed {len(valid_records) + len(invalid_records)} rows: "
      f"{len(valid_records)} valid, {len(invalid_records)} invalid.\n")

for record in invalid_records:
    print(f"Row {record['row_number']} rejected: {record['row']} -> {record['reason']}")

class_average = sum(r["marks"] for r in valid_records) / len(valid_records)
print(f"\nClass average (valid rows only): {class_average:.2f}")
```

```
Processed 6 rows: 3 valid, 3 invalid.

Row 3 rejected: ['Rohit Sharma', '102', 'eighty'] -> invalid literal for int() with base 10: 'eighty'
Row 5 rejected: ['Karthik Nair', '104', ''] -> invalid literal for int() with base 10: ''
Row 6 rejected: ['Priya Singh', '105', '150'] -> marks 150 is outside the valid 0-100 range

Class average (valid rows only): 85.00
```

Trace the average by hand before trusting it: only Ananya (88), Meera (95), and Arjun (72) ever reach `valid_records`, so `(88 + 95 + 72) / 3` is exactly `85.00` — a genuinely believable class average, computed over data you can actually stand behind. Compare that to Stage 2's `101.25`, and you can see precisely what three stages of refinement bought you: not just "didn't crash," but "produced a number worth trusting."

A natural next step, if this program were headed for a real exam cell rather than a classroom, would be writing `invalid_records` out to its own CSV file with `csv.writer()` — the same tool from Unit 5.1 — so a member of staff could open a rejection report later without needing to re-run any code at all. You'll build exactly that in the exercise below.

## Robust readers in the real world

This "skip and log" shape is not a classroom simplification — it is the baseline expectation for any real system that accepts a file from outside itself:

- **Banking and fintech** — an overnight batch job reconciling thousands of transaction rows routes anything malformed into a review queue instead of halting the entire night's processing over one row.
- **UPI payment settlement** — a settlement file listing thousands of transactions logs and sets aside any row with a corrupted or missing amount, so every other transaction still settles on schedule.
- **IRCTC-style railway booking** — a bulk upload of ticket bookings from a travel agent lets every row with a valid fare and passenger count through, while a row with a missing fare goes into a rejection report for staff to chase up later.
- **E-commerce catalog uploads** — a seller uploading five thousand products expects the platform to accept every well-formed row and report back the handful with a missing price or invalid category, not reject the whole file.
- **AI/ML data pipelines** — a training script reading a large dataset skips and logs a handful of corrupted rows rather than aborting a multi-hour training run over data it was never going to use anyway.

## Common mistakes when building a robust reader

- **Wrapping `try`/`except` around the whole `for` loop instead of one row inside it** — this is fail-fast wearing a fail-soft costume; the very first bad row still ends the loop entirely.
- **Catching a bare `except:`** to "handle everything" — this also swallows genuine bugs in your own code, such as a typo in a variable name, and you never find out they were there.
- **Assuming a successful conversion means a valid value** — `int("150")` will never raise an exception on its own; only an explicit business-rule check catches it.
- **Silently discarding a bad row with no record of what happened** — a program that stops crashing but never logs what it rejected has traded one problem (a crash) for another (invisible data loss).
- **Running a summary calculation over a list that mixes valid and invalid rows** — a rejected row that carries no real number can quietly distort a total or an average if it's ever counted.

## Try it yourself

Do this in a Colab cell before moving on. You are handed `orders.csv`, with columns `Customer,Amount` — some rows have a blank amount, one has non-numeric text, and one has a negative amount. Write a reader, following every stage built in this unit, that separates rows into `valid_records` and `invalid_records`, treating an amount as valid only if it converts to a number *and* is strictly greater than 0. Print a summary count of valid versus invalid rows, then print the total of all valid amounts, computed only over `valid_records`. Confirm for yourself that the negative amount is correctly rejected even though it converts to a number without any error, and that your reader still reaches and processes every row that comes after each bad one. As a stretch goal, write `invalid_records` out to a second file, `invalid_orders.csv`, complete with its own header row, using `csv.writer()` from Unit 5.1.

---

### Key Terminology

- **Robust file reader** — a program that reads a file row by row and sorts every row into "worked" or "didn't," instead of stopping at the first failure.
- **Valid record** — a row that converted correctly and passed every check your program cares about.
- **Invalid record (malformed row)** — a row that failed to convert, or converted fine but broke a business rule.
- **Fail-fast** — stopping the program immediately at the first sign of a problem.
- **Fail-soft (skip and log)** — recording a problem and continuing, only finishing once every row has been looked at.
- **Business rule** — a condition specific to what your data is supposed to mean, which no built-in type conversion can check on its own.
- **Graceful degradation** — a program continuing to produce useful, partial results even when part of its input is broken.
- **`enumerate(iterable, start=n)`** — pairs each item from an iterable with a running count, starting from `n`, used here to track a row's actual line number.

### Mastery Checkpoint

Before moving to Unit 6.1, check that you can answer these without looking back:

1. Why does wrapping `try`/`except` around an entire `for` loop fail to produce a robust reader, even though `try`/`except` is technically present?
2. `int("150")` never raises an exception. Why can a row with `marks = 150` still end up in `invalid_records`, and what line of code actually rejects it?
3. What is the difference between what Stage 2 (skip and print) gives you and what Stage 3 (two separate lists) gives you, given that both survive the exact same bad rows?
4. Why does raising `ValueError` yourself for a business-rule failure let one `except ValueError:` block catch both that failure and a genuine conversion failure?
5. A reader is meant to compute a class average, but the number it prints turns out to be mathematically impossible. What is the first thing you should check?

### Summary

You now know how to combine `open()` and `csv.reader()` from Unit 5.1 with `try`/`except` and `raise` from Unit 5.2 into a single, disciplined pattern: two separate lists for valid and invalid records, a `try`/`except` scoped around one row at a time rather than the whole loop, a business-rule check that catches values Python's own conversion would happily accept, and a per-row log of exactly what failed and why. This closes out Module V — you have now covered reading and writing files, handling the exceptions that real files and real users inevitably produce, and, in this unit, the exact professional pattern that ties both skills into a reader you would genuinely trust with real data. From here, the next step moves away from writing individual programs and toward managing how your code changes over time — starting with Unit 6.1, Version Control Basics.

### Additional Resources

- [Python 3 Documentation — `csv`: CSV File Reading and Writing](https://docs.python.org/3/library/csv.html)
- [Python Tutorial — official docs: "Errors and Exceptions"](https://docs.python.org/3/tutorial/errors.html)
- [Python 3 Documentation — Built-in Exceptions (`ValueError`)](https://docs.python.org/3/library/exceptions.html#ValueError)
- [Python 3 Documentation — Built-in Functions (`enumerate()`)](https://docs.python.org/3/library/functions.html#enumerate)
- [W3Schools — Python Try Except](https://www.w3schools.com/python/python_try_except.asp)
- [W3Schools — Python JSON / File Handling](https://www.w3schools.com/python/python_file_handling.asp)
