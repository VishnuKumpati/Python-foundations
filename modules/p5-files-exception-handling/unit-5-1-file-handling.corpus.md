---
topic_id: 24960c98-315e-402d-9539-10ed668e7a80
title: Unit 5.1 - File Handling
position_in_module: 1
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 5.1 - File Handling — Topic Corpus

## 2. Prerequisites

- Unit 2.5: module, importing, standard library — file handling in this unit relies on importing Python's built-in `csv` and `json` modules exactly the way Unit 2.5 taught.
- Unit 3.1: list — CSV rows are read back as lists of strings, and this corpus assumes indexing and iteration over a list are already familiar.
- Unit 3.4: dictionary, key, value, key-value pair, mapping, bracket access — JSON data and `csv.DictReader` rows both come back to you as dictionaries, so this corpus assumes that shape is already understood.
- Unit 1.4: f-string, type conversion, `ValueError` — reading real files constantly requires converting text to numbers with `int()`/`float()`, and diagnosing the `ValueError` that results when that conversion is given text that isn't numeric.
- Unit 2.2: for loop, iteration — every example in this unit loops over the rows of a file one at a time.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** the difference between a text file and a binary file, and between an absolute path and a relative path.
- **Identify** the correct file mode (`r`, `w`, `a`, `r+`) for a given task, and describe what happens to existing file content in each case.
- **Apply** the `with` statement as a context manager to open, use, and automatically close a file, including when an error interrupts the block.
- **Differentiate** between reading a file as plain text, as CSV rows via the `csv` module, and as structured data via the `json` module.
- **Apply** `csv.reader`/`csv.DictReader` and `json.load`/`json.dump` to read and write the two most common real-world data formats.
- **Explain** why file operations can fail for reasons outside a program's control, without yet writing code that handles those failures.

## 4. Introduction

Every Python program written up to this point disappears the moment it stops running. Type `balance = 5000` in a notebook cell, close the notebook, reopen it tomorrow, and `balance` is gone — it lived only in the computer's temporary working memory (RAM), which is wiped clean the instant the program that used it ends. That's fine for a five-line practice exercise, but it is not how any real software behaves. A banking app has to remember your account balance after you close it and reopen it a week later. A food delivery app has to keep a record of every order, so a support agent can look one up next month. None of that works if data vanishes when the program stops.

The fix is the **file**: data written to a storage device (like a hard disk), so it survives after the program that created it has ended. This single idea — write data somewhere durable, then read it back later, possibly in a completely different program run — is one of the most universally used skills in software, and it is also the foundation this entire module is building toward. Two of the most common shapes that stored data takes are CSV (spreadsheet-style rows and columns) and JSON (the format most web services and AI tools speak to each other in). By the end of this unit you will be able to open a file safely, read and write plain text, and read and write both CSV and JSON — the exact pattern behind the "robust file reader" this module is building up to, and behind almost every data-driven tool you will build afterward.

Consider a small concrete scenario: a railway booking system needs to write a confirmation note when a ticket is booked, read back a whole day's bookings from a CSV export to check who is confirmed versus waitlisted, and save one passenger's complete record so it can be reloaded later — perhaps by a completely different program, on a completely different day. Each of those three needs uses a different tool covered in this unit: plain-text writing and reading, the `csv` module, and the `json` module. None of it is exotic; it is the same handful of building blocks, `open()`, `with`, `csv`, and `json`, combined in different orders depending on the shape of the data involved.

## 5. Core Concepts

### 5.1 Files and Paths

A **file** is a named collection of data stored permanently on a storage device such as a hard disk or solid-state drive (SSD), in contrast to a variable, which exists only in RAM — the computer's temporary working memory — while a program is running [1]. When the program ends, RAM is cleared; a file on disk is not.

Files come in two broad kinds [1]:

- A **text file** stores human-readable characters — letters, digits, punctuation — using a character **encoding**, a scheme that maps each character to a specific sequence of bytes on disk. The most common encoding today is UTF-8. `.txt` notes, `.csv` spreadsheets, and `.json` data files are all text files, and this entire unit works only with text files.
- A **binary file** stores raw bytes that are not meant to be read as characters at all — images (`.jpg`), audio (`.mp3`), compiled programs (`.exe`). Opening a binary file in a plain text editor shows scrambled symbols, because the editor is trying to interpret non-character data as if it were text.

Every file also has a **path** — its address on the storage device — in one of two forms [1]:

- An **absolute path** gives the complete location starting from the drive's root, for example `C:\Users\Priya\data\marks.csv`. It works no matter where the program that opens it is run from.
- A **relative path** gives a location relative to wherever the program is currently running from, for example `data/marks.csv`. It is shorter, and — unlike an absolute path — it keeps working if the entire project folder is moved to another computer, because it never depends on a specific drive letter or user folder name.

A useful comparison: an absolute path is like a full postal address (country, city, street, house number) that works from anywhere in the world; a relative path is like "two doors down from where you are standing right now" — only meaningful once you already know where "here" is. Most real projects prefer relative paths for exactly that portability [1]. Concretely: a program that opens `"C:\Users\Priya\data\marks.csv"` will fail the moment the project folder is copied to a different computer with a different username, because that exact drive letter and folder no longer exists — whereas a program that opens `"data/marks.csv"` keeps working as long as the `data` folder travels along with the project, regardless of whose computer it lands on.

### 5.2 Opening a File: `open()` and Modes

Python's built-in `open()` function is how a program connects to a file on disk. Calling `open(filename, mode)` returns a **file object** — an object you then call methods on to read from, or write to, the file it represents [1]. `filename` is a string holding the file's path (absolute or relative); `mode` is a short string telling Python what you intend to do with the file, so Python can prepare it correctly before you do anything else.

Four modes cover almost every situation a beginner will meet [1]:

| Mode | Meaning | Effect on existing content |
|---|---|---|
| `"r"` | Read — open an existing file to read from it. This is the default if no mode is given at all. | Nothing is changed; if the file does not exist, Python immediately raises a `FileNotFoundError` rather than creating one. |
| `"w"` | Write — open a file so you can write to it, creating it if it doesn't exist yet. | **Erases everything already in the file the instant it is opened** — before a single new character is written. There is no undo. |
| `"a"` | Append — open a file so new content is added after whatever is already there. | Existing content is left untouched; new writes are added at the end. |
| `"r+"` | Read-and-write — open an existing file for both reading and writing. | Existing content is left untouched, but — like `"r"` — the file must already exist, or Python raises `FileNotFoundError`. |

`"r+"` is the least commonly needed of the four for a beginner, but it exists for situations where a program genuinely needs to both inspect a file's current content and modify it within the same open connection to it, rather than opening it twice under two different modes. For almost everything in this unit — reading a report, saving a record, appending a log line — one of `"r"`, `"w"`, or `"a"` is the right choice, and `"r+"` is mentioned here mainly so its name is recognizable rather than mysterious the first time it appears in someone else's code.

The mode you choose is a decision with real consequences: opening a file that already holds a week's worth of saved data in `"w"` mode by mistake, instead of `"a"`, silently destroys that data before you write anything new. There is no warning and no confirmation prompt — the erasure happens the moment `open()` runs [1].

To see the difference concretely, imagine `log.txt` already contains one line, `"Day 1: system started"`. Opening it with `"a"` and writing `"Day 2: system started\n"` leaves both lines in the file, in order. Opening the very same file with `"w"` and writing that same line leaves only `"Day 2: system started"` — the first line is gone, permanently, with no message telling you it happened. This is precisely why choosing the mode deliberately, rather than by habit, matters as much as any other line in the program.

### 5.3 Reading and Writing Plain Text

Once a file object exists, its `write()` method takes a string and adds it to the file, and its `read()` method returns the file's entire remaining content as one string. One easy-to-miss detail: `write()` never adds a newline character (`\n`) on its own — if you want each write to end a line, you must include `\n` in the string yourself [1]. `\n` is the character sequence that represents "start a new line" inside a text file; it is invisible when a file is displayed as formatted text, but it is a real character stored in the file.

```python
with open("booking_note.txt", "w") as file:
    file.write("Booking confirmed for Priya Nair, PNR 4521067890\n")
```

Here, `"Booking confirmed for Priya Nair, PNR 4521067890\n"` is written into `booking_note.txt`; without the trailing `\n`, any text written immediately afterward would run onto the same line.

A file object also supports looping over it directly, one line at a time, which is often more convenient than calling `read()` and getting the entire file back as a single string:

```python
with open("booking_note.txt", "r") as file:
    for line in file:
        print(line)
```

Here, `for line in file:` visits the file one line at a time, binding each one — still ending in its own `\n` — to `line`. Because both the file's own `\n` and `print()`'s automatic newline are present, each printed line is followed by one blank line; this is the same doubled-newline behavior seen whenever text read from a file, still carrying its original `\n`, is passed straight into `print()`.

### 5.4 Closing Safely: Context Managers and the `with` Statement

Every file that gets opened must eventually get **closed** — the file object's `close()` method, which tells the operating system you are done with the file and releases its handle. Closing matters for a reason that is easy to overlook: when you call `write()`, the text is not necessarily written straight to the disk. It commonly sits first in a **buffer** — a small block of memory used as a temporary holding area — and is only flushed (actually written) to disk when the file is closed [1]. If a program crashes, or simply forgets to call `close()`, that buffered data may never reach the disk at all, and anyone else who tries to read the same file afterward may see nothing, or only part of what was written [1].

Calling `close()` manually works, but it has a serious weakness: if an error occurs anywhere between `open()` and the `close()` line, the `close()` line is never reached, and the file is left open. Python's answer to this is the **context manager** — an object that defines a setup action to run when a block begins and a cleanup action that is *guaranteed* to run when the block ends, no matter how it ends [1]. The `with` statement is the syntax used to invoke a context manager, and the file object returned by `open()` is itself a context manager:

```python
with open("marks.csv", "r", newline="") as file:
    ...
```

`with open(...) as file:` opens the file, binds the returned file object to the name `file` for use inside the indented block, and — the moment that block finishes, whether it finished normally or an error interrupted it partway through — automatically calls `file.close()`. This is why `with` is the standard, expected way to work with files in real Python code: it removes the possibility of a forgotten `close()` entirely [1].

It is worth being able to state precisely what a context manager promises, since it is a frequent interview question for freshers: it defines two steps — a setup action that runs the moment the `with` block is entered (here, the file has already been opened by `open()` before the block starts), and a cleanup action that is *guaranteed* to run the moment the block is exited, regardless of whether that exit was normal or caused by an error partway through [1]. For a file specifically, "cleanup" means closing it and releasing the operating system's file handle on it, and flushing any buffered writes to disk. Manual `open()`/`close()` offers no such guarantee at all — an error raised on any line before the `close()` call simply skips over it, leaving the file open and its buffered writes possibly never reaching disk.

| Aspect | Manual `open()` / `close()` | `with` Context Manager |
|---|---|---|
| Closes on success | Yes, if the `close()` line is reached | Yes, always |
| Closes if an error occurs mid-block | No — the `close()` line is skipped entirely | Yes — cleanup runs automatically as the block is exited |
| Risk of a forgotten `close()` | High | None |
| Recommended for new code | No | Yes |

### 5.5 Storing Tables of Data: CSV

**CSV**, short for Comma-Separated Values, is a plain-text file format that stores rows and columns as text, with the values in each row separated by commas — the shape you get whenever a spreadsheet is exported as plain text [1]. A CSV file with a header row and two data rows looks like this on disk:

```
Passenger,Seats
Priya Nair,2
Arjun Rao,0
```

Python's built-in `csv` module (imported with `import csv`, exactly as Unit 2.5 taught for any standard-library module) understands this format well enough to handle tricky cases correctly, such as a name containing a comma inside quotes, which naive comma-splitting would get wrong [1]. Two tools from that module matter most:

- `csv.reader(file)` wraps an already-open file object so that iterating over it yields each row as a **list** of strings.
- `csv.DictReader(file)` wraps an open file object so that iterating over it yields each row as a **dictionary**, with keys taken automatically from the file's first (header) row.

```python
import csv

with open("bookings.csv", "r", newline="") as file:
    reader = csv.DictReader(file)
    for row in reader:
        seats = int(row["Seats"])
        print(row["Passenger"], seats)
```

`newline=""` is an argument passed to `open()` itself, not to `csv.reader` or `csv.DictReader` — the `csv` module's own documentation requires it so that the module can manage line endings itself, rather than rows splitting incorrectly on some systems [1].

Two rules make CSV behave differently from what a beginner might expect. First, **every value read from a CSV row arrives as a string**, even a value that looks like a whole number — CSV has no concept of numeric types at all, only text — so `row["Seats"]` must be converted with `int()` before it can be used in arithmetic or a numeric comparison [1]. Second, `csv.reader()` does **not** skip the header row automatically: the very first row it yields is the header (`['Passenger', 'Seats']`), and if that row is fed straight into `int()` the program crashes with a `ValueError`, because the text `"Seats"` cannot become a number. `csv.DictReader()` avoids this problem for you, because it consumes that header row itself in order to build its dictionary keys — but plain `csv.reader()` requires you to discard the header row yourself, typically with `next(reader)`, before your own loop begins [1].

The `csv` module also writes files, which matters just as much as reading them — an "export my data" feature in a real application is exactly this in reverse. `csv.writer(file)` wraps an open file object opened in `"w"` mode, and its `writerow(list_of_values)` method writes one row at a time:

```python
import csv

with open("bookings.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["Passenger", "Seats"])       # header row, written first
    writer.writerow(["Priya Nair", 2])
    writer.writerow(["Arjun Rao", 0])
```

`writer.writerow([...])` takes a list and writes its values as one comma-separated line, automatically placing commas between them and a newline at the end — the exact reverse of what `csv.reader()` does when reading a row back. Note that the header row is not automatic here; a `csv.writer` writes exactly the rows you give it, in order, so the header must be written explicitly as the very first `writerow()` call.

### 5.6 Storing Structured Data: JSON

**JSON**, short for JavaScript Object Notation, is a plain-text data format for storing key-value pairs and lists, structurally close to a Python dictionary — and it is the format most web services and AI/ML APIs use to exchange data [1]. Where CSV is naturally shaped like a single flat table, JSON is naturally shaped like the nested dictionaries and lists already familiar from Units 3.1–3.4:

```json
{
  "pnr": "4521067890",
  "passenger": "Priya Nair",
  "seats": 2,
  "status": "Confirmed"
}
```

Python's built-in `json` module provides two pairs of functions:

- `json.dump(data, file)` — **serialization**: takes a Python object (typically a `dict` or `list`) and writes its JSON text representation into an already-open file.
- `json.load(file)` — **deserialization**: reads JSON text from an already-open file and converts it back into a Python object.
- `json.dumps(data)` / `json.loads(text)` — the same two operations, but working on a Python string already sitting in a variable rather than a file on disk. (The trailing `s` stands for "string.") Using the file-based function where the string-based one was needed, or vice versa, raises a `TypeError`.

```python
import json

booking = {"pnr": "4521067890", "passenger": "Priya Nair", "seats": 2, "status": "Confirmed"}

with open("booking.json", "w") as file:
    json.dump(booking, file, indent=2)

with open("booking.json", "r") as file:
    data = json.load(file)
    print(data["passenger"], data["seats"])
```

`indent=2` is a purely cosmetic, optional argument that produces human-readable, indented JSON text instead of one dense unbroken line — it has no effect on the data itself [1]. The single biggest practical difference between JSON and CSV is that **JSON preserves real types**: a JSON number is read back as a Python `int` or `float`, `true`/`false` becomes a Python `bool`, and `null` becomes `None` — so `data["seats"]` above comes back as the integer `2`, not the string `"2"` a CSV row would have produced for the same value [1].

Because JSON's shape is just Python's own dictionaries and lists written out as text, nesting works exactly as it already does with a nested dictionary: a JSON value can itself be another set of key-value pairs, or a list of them. A booking record that needs to store several stops on a passenger's route, for example, naturally becomes a list nested inside the dictionary:

```json
{
  "passenger": "Priya Nair",
  "stops": ["Chennai", "Vijayawada", "Hyderabad"]
}
```

`json.load()` turns this back into an ordinary Python dictionary whose `"stops"` key holds an ordinary Python list — `data["stops"][0]` is the string `"Chennai"`, accessed exactly the way any nested dictionary-inside-a-list-inside-a-dictionary would already be accessed. No new syntax is required to work with nested JSON; it deserializes into exactly the nested dictionaries and lists already familiar from earlier units.

### 5.7 When File Operations Fail

Opening a file that does not exist in `"r"` mode raises a **`FileNotFoundError`** — Python's way of signaling, immediately, that there is nothing at the given path to read [1]. This is only one example of a broader truth: file operations depend on things outside a program's control entirely. A path can be mistyped. A file a program expects to already exist may never have been created, or may have been created in a different folder than the one the program is looking in. A disk can run out of free space partway through a write. A folder can lack the operating-system permission needed to write into it at all. None of these are bugs in the sense of incorrect logic — the code asking to open the file may be perfectly correct — they are failures of the environment the program happens to be running in, and every one of them is entirely realistic in production software, not just a classroom exercise.

This unit's job is only to make these failure modes recognizable: knowing that `FileNotFoundError` specifically means "nothing exists at this path in read mode" is itself useful, because it immediately narrows down what to check first. What this unit deliberately does *not* do is teach a way to catch that error and keep the program running smoothly afterward — that mechanism, `try`/`except`, is the entire subject of the very next topic, Unit 5.2: Errors & Exceptions. For now, a `FileNotFoundError` (or any other file-related failure) still stops the program, exactly like every other unhandled error seen so far in this course.

## 6. Implementation

The pattern behind reliably reading any file — plain text, CSV, or JSON — follows the same sequence of steps every time:

1. **Decide the mode.** Are you reading existing content (`"r"`), replacing a file's content from scratch (`"w"`), or adding to what's already there (`"a"`)? Getting this wrong is the single most common file-handling mistake.
2. **Open with `with`.** Write `with open(path, mode) as file:` rather than a bare `open()` call, so the file closes automatically no matter what happens inside the block.
3. **Pick the right reader for the shape of the data.** Plain text → `file.read()`. Rows and columns → wrap `file` with `csv.reader()` or `csv.DictReader()`. Nested key-value data → pass `file` to `json.load()`.
4. **Skip the header, if using `csv.reader()`.** Call `next(reader)` once before the loop to consume and discard the header row — or use `csv.DictReader()`, which does this automatically.
5. **Convert values to their real types before using them.** Every CSV value is text; convert with `int()` or `float()` as needed before doing arithmetic or comparisons. JSON values are already the right type, so this step is unnecessary there.
6. **Let the `with` block end.** Do not call `close()` yourself — the context manager handles it, even if an error interrupts the block partway through.
7. **When writing back out, mirror the same structure.** A `csv.writer` reproduces a header row and comma-separated rows explicitly, one `writerow()` call at a time; `json.dump()` reproduces the exact nested shape of whatever Python object (typically a `dict` or `list`) is passed to it.
8. **Treat the first failure as a diagnostic clue, not a dead end.** A `FileNotFoundError` almost always means a typo in the path or a file that was never created in the expected folder; a `ValueError` on the very first CSV row almost always means the header was never consumed.

Applied to the railway booking scenario from the introduction, these steps look like: open `bookings.csv` with `with` and `newline=""` (steps 1–2); choose `csv.DictReader` since the file has more than two columns worth naming (step 3); rely on `DictReader`'s automatic header handling rather than calling `next()` (step 4); convert `row["Seats"]` with `int()` before comparing it to a threshold (step 5); let the block close the file automatically (step 6); and, if a confirmed booking's full record needs saving for later, build a dictionary from that row and pass it to `json.dump()` (step 7).

The same steps generalize far beyond railway bookings. A student-marks program reading a CSV export from a college's result portal follows exactly this sequence: choose `"r"` (step 1), open with `with` (step 2), use `csv.DictReader` since a marksheet typically has several named columns (step 3), rely on the automatic header handling (step 4), convert each mark from string to number before averaging (step 5), and let the file close itself (step 6). Swap "railway booking" for "student marks," "e-commerce order," or "patient record," and the steps do not change — only the column names and the business rule applied to the converted values do. This is precisely why file handling, once learned this way, transfers cleanly to almost any dataset a beginner will meet afterward.

## 7. Real-World Patterns

File handling of exactly this shape appears constantly in production systems [1]:

- **Banking and FinTech:** nightly batch jobs read a CSV of a day's transactions to reconcile accounts, while significant events are logged as JSON for audit trails.
- **UPI and payment systems:** each payment attempt is commonly written as a JSON record, so it can later be replayed, audited, or displayed in a user's transaction history.
- **E-commerce:** an "export orders" button on a shopping site is, on the server, a CSV writer running inside a `with` block, guaranteeing the file handle can never be left stuck open even if the download is interrupted.
- **Healthcare:** patient records exported for a lab or insurer are commonly CSV; hospital systems exchanging data with external software increasingly use JSON instead.
- **Education:** a college's result portal generates a student's marksheet as a CSV export, while its own internal APIs answer student queries with JSON.
- **Railway booking systems:** a daily bookings report is a CSV of passenger name, fare, and seat count — precisely the shape used throughout this unit's worked examples.
- **AI/ML:** training pipelines load datasets from CSV or JSON files before a single model is trained, and every request and response to an AI model's API travels as JSON — the same `json.load()`/`json.dump()` pattern taught here.

## 8. Best Practices

- Always use `with open(...) as file:` rather than a manual `open()`/`close()` pair — it guarantees the file closes even when an error occurs, which manual closing cannot [1].
- Pass `encoding="utf-8"` explicitly when opening text files, rather than relying on a computer's default encoding, which can differ between machines and cause subtle bugs when a file is later opened elsewhere [1].
- Pass `newline=""` to `open()` whenever reading or writing CSV, exactly as the `csv` module's own documentation recommends [1].
- Prefer relative paths within a project, so it keeps working after being copied or shared to another computer [1].
- Prefer `csv.DictReader` over plain `csv.reader` once a file has more than two or three columns — named access (`row["Marks"]`) reads far more clearly than positional access (`row[1]`) [1].
- Double-check the mode before opening an existing file — a `"w"` where an `"a"` was meant silently destroys everything already saved, with no warning and no way to undo it.
- Write the header row explicitly and first whenever using `csv.writer`, since — unlike `csv.DictReader` on the reading side — nothing about `csv.writer` adds one automatically.
- Use `json.dump(..., indent=2)` for any file a human might open directly to inspect; the extra whitespace costs nothing functionally and makes debugging far faster.
- Treat a `ValueError` on the very first row processed from a CSV file as a strong hint that the header row was never skipped, rather than assuming the data itself is malformed.

## 9. Hands-On Exercise

Using a file `students.csv` containing a header row `Name,Score` followed by three data rows, write a program that: (1) opens the file with `with` and `csv.DictReader`; (2) converts each `Score` to an `int`; (3) builds one Python dictionary per student combining their name and score; (4) writes the full list of these dictionaries to `students.json` using `json.dump(..., indent=2)`. Then reopen `students.json` and print each student's name and score, confirming that the score comes back as a genuine integer rather than a string, unlike the value read straight from `students.csv`.

As a follow-up, deliberately break the program by opening `students.csv` with plain `csv.reader()` instead of `csv.DictReader()`, and remove the line that skips the header row. Run it, read the resulting `ValueError` message carefully, and confirm it points at the header row's text value — this is the single most common CSV mistake freshers hit, and recognizing its signature quickly is worth practicing deliberately.

## 10. Key Takeaways

- A file lets data outlive the program that created it; a variable does not, because RAM is cleared the moment a program ends.
- `open(filename, mode)` returns a file object, and the mode — `"r"` (read), `"w"` (write, erasing existing content), `"a"` (append), `"r+"` (read and write) — determines exactly what happens to any content already there.
- A context manager, invoked with `with open(...) as file:`, closes a file automatically when its block ends, whether that block finished normally or was interrupted by an error — something manual `open()`/`close()` cannot guarantee.
- CSV stores rows and columns as plain text, and every value read from it is a string regardless of appearance; `csv.reader()` yields lists and requires the header row to be skipped manually, while `csv.DictReader()` yields dictionaries and consumes the header automatically.
- JSON stores key-value data shaped like a Python dictionary and preserves real types (numbers, booleans, `None`); `json.dump()`/`json.load()` serialize to and deserialize from a file, while `json.dumps()`/`json.loads()` do the same for a string already in memory.
- Opening a nonexistent file in read mode raises `FileNotFoundError`; handling such failures gracefully, rather than only recognizing that they occur, is covered next in Unit 5.2.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
