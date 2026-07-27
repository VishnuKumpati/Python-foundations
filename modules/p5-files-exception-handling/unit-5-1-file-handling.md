# File Handling

Unit 4.3 closed out Module IV by showing you how a class can define exactly how its own objects print, compare, and represent themselves — the last piece of polish on everything you had learned about bundling data and behaviour together. But every one of those objects, however elegant, shares one quiet limitation: the moment your Colab runtime restarts, or you simply close the browser tab, every variable, every list, every dictionary, every object you built is gone. RAM — the memory a running program uses — is wiped clean the instant that program stops, and nothing you have written so far survives past the session that created it.

Real software cannot live with that limitation. A banking app has to remember your account balance next week, not just for the ten seconds you were looking at your phone. A food-delivery app has to keep a record of every order ever placed, so a support agent can pull one up months later. And a huge amount of the data your programs will ever touch does not even start out as something you typed — it arrives from somewhere else entirely: a bank statement exported as a spreadsheet, a response from a payment gateway's API, a configuration file somebody else wrote. None of that works with memory alone.

This unit hands you the fix: the **file** — data written to a storage device, where it outlives the program that created it — and the two data formats you will meet constantly once you start reading and writing real files, CSV and JSON. By the end, you will be able to open a file safely, read it back in several different ways, close it correctly every single time without having to remember to, and move data in and out of the two formats that most of the software industry already speaks.

---

## From memory to disk: what a file actually is

A **file** is a named collection of data stored permanently on a storage device — a hard disk, an SSD, or in Colab's case, the temporary disk attached to your notebook's runtime — in contrast to a variable, which exists only in RAM while your program is actually running. Close the program, and RAM is cleared; a file left on disk is not.

Files come in two broad kinds. A **text file** stores human-readable characters — letters, digits, punctuation — using a character **encoding**, a scheme that maps each character to a specific sequence of bytes. UTF-8 is by far the most common encoding today, and `.txt` notes, `.csv` spreadsheets, and `.json` data files are all text files — everything this unit works with. A **binary file** stores raw bytes never meant to be read as characters at all — a photo (`.jpg`), an audio clip (`.mp3`), a compiled program (`.exe`). Open a binary file in a plain text editor and you get scrambled symbols, because the editor is stubbornly trying to interpret non-character data as if it were text.

Every file also has a **path** — its address on disk — written one of two ways:

| Path type | What it looks like | Works after moving the project? |
|---|---|---|
| **Absolute path** | `C:\Users\Priya\data\marks.csv` | No — breaks the moment the drive letter or folder no longer exists |
| **Relative path** | `data/marks.csv` | Yes — keeps working as long as `data/` travels with the project |

Think of an absolute path as a full postal address — country, city, street, house number — meaningful from anywhere in the world. A relative path is closer to "two doors down from where you're standing right now" — it only means something once you already know where "here" is. That is precisely why most real projects, and every example in this unit, favour relative paths: `open("marks.csv", "r")` keeps working wherever the notebook happens to run, as long as `marks.csv` sits alongside it.

**One Colab-specific catch worth knowing now: a file you write to your notebook's default working directory lives only as long as that runtime does — restart the runtime, and the file is gone exactly like every variable was, unless you had saved it to Google Drive or downloaded it first.** Files genuinely outlive a single *program run*, but on Colab's free, temporary runtime they do not automatically outlive a full runtime reset — persistence and "not stored in Google Drive" are two different things, and it is worth holding both facts in mind at once.

## Opening a file: `open()` and telling Python your intention

Python's built-in `open()` function is how a program connects to a file on disk. Calling `open(filename, mode)` hands back a **file object** — an object you then call methods on to read from it or write to it. `filename` is a string holding the path; `mode` is a short string telling Python exactly what you plan to do with the file, so it can prepare the connection correctly before you touch anything.

| Mode | Meaning | What happens to existing content |
|---|---|---|
| `"r"` | Read — open an existing file to read from it (the default if you give no mode at all) | Nothing changes; if the file doesn't exist, Python raises `FileNotFoundError` immediately |
| `"w"` | Write — open a file to write to it, creating it if it doesn't exist | **Erases everything already there, the instant it's opened** — before a single new character is written |
| `"a"` | Append — open a file so new content is added after what's already there | Existing content is left completely untouched |
| `"x"` | Exclusive creation — create a brand-new file, only if one doesn't already exist at that path | If the file already exists, Python refuses and raises `FileExistsError` instead of touching it |

Before reading on, predict for yourself: if `statement.csv` already holds three months of exported bank transactions and you open it with `"w"` by mistake instead of `"a"`, what happens to those three months of data the moment `open()` runs? It is erased — completely, silently, with no confirmation prompt and no undo — before you have written a single new line. `"x"` exists precisely to guard against this kind of accident: opening a UPI transaction log with `"x"` instead of `"w"` means Python itself refuses to overwrite a file that turns out to already exist, forcing you to notice the collision rather than silently destroying it.

```python
file = open("transaction_log.txt", "x")
file.write("UPI-TXN-88213: Rs.500 to ananya@upi\n")
file.close()
```

```
FileExistsError: [Errno 17] File exists: 'transaction_log.txt'
```

**Choosing a file's mode is a decision with real consequences, not a formality — mixing up `"w"` and `"a"` is one of the most common, and most damaging, file-handling mistakes a beginner makes.** There is no dialog box asking "are you sure?" — Python trusts that you meant exactly what you typed.

## Why a file must always be closed

Every file you open must eventually be **closed**, using the file object's `close()` method — the signal that tells the operating system you're done with the file and releases its handle on it. This matters for a reason that is easy to overlook: when you call `write()`, the text you hand over does not necessarily land on disk immediately. It typically sits first inside a **buffer** — a small block of memory used as a temporary holding area — and is only flushed, meaning actually written to disk, when the file is closed. If your program crashes, or you simply forget to call `close()`, that buffered text may never reach the disk at all. Somebody else — a colleague's script, a second `open()` call, even you tomorrow — reading that same file afterward may find it empty, or holding only part of what you thought you wrote.

```python
file = open("order_summary.txt", "w")
file.write("Order #4471: 3 items, Rs.899 total\n")
# forgot file.close() here
```

Run only this much and reopen `order_summary.txt` from a separate cell, and there is a real chance the line you just wrote is not there yet — the write is still sitting in the buffer, waiting for a `close()` that never came.

**Calling `close()` manually has a serious weakness: if an error is raised anywhere between `open()` and the `close()` line, that line is never reached, and the file is left open with its buffer possibly never flushed.** That single weakness is exactly what the next section fixes.

## `with`: a library book that returns itself

Think of borrowing a library book. You could take it home and rely purely on your own memory to bring it back by the due date — but if you get distracted, fall ill, or simply lose track of the date, the book stays out. Now imagine a library where every book you borrow is automatically, unfailingly returned the moment you finish reading it — even if you got distracted halfway through, even if something interrupted you entirely. That second library is what Python's **context manager** gives you for files.

A context manager is an object that defines a setup action to run when a block begins, and a cleanup action that is *guaranteed* to run when the block ends — no matter how it ends, cleanly or via an error partway through. The `with` statement is the syntax that invokes one, and the file object `open()` returns is itself a context manager:

```python
with open("order_summary.txt", "w") as file:
    file.write("Order #4471: 3 items, Rs.899 total\n")
```

`with open(...) as file:` opens the file, binds the returned file object to `file` for use inside the indented block, and — the instant that block finishes, whether normally or because an error interrupted it — automatically calls `file.close()` for you. You never write `file.close()` yourself inside a `with` block; the guarantee is built in.

| Aspect | Manual `open()` / `close()` | `with` context manager |
|---|---|---|
| Closes on success | Yes, if the `close()` line is actually reached | Yes, always |
| Closes if an error interrupts the block | No — the `close()` line is skipped entirely | Yes — cleanup runs regardless |
| Risk of a forgotten `close()` | High | None |
| Recommended for new code | No | Yes |

**Use `with open(...) as file:` for every single file you open in this course, and in any real Python code you write afterward — there is no situation where manual `open()`/`close()` is genuinely the better choice.** Try it yourself: rewrite the `order_summary.txt` example from the previous section using `with`, then try to reopen the file in a fresh cell immediately afterward — the line you wrote is there this time, because the block already closed and flushed it before that first cell even finished running.

## Reading a file back

Once a file is open for reading, a file object gives you several ways to pull its content back out, each suited to a different situation.

```python
with open("order_summary.txt", "r") as file:
    whole_thing = file.read()
    print(whole_thing)
```

```
Order #4471: 3 items, Rs.899 total

```

- `.read()` returns the file's entire remaining content as one single string. Notice the blank line after the output above: `whole_thing` still carries the `\n` that was written into the file, and `print()` adds its own newline on top of that — this doubled newline is one of the first small surprises beginners run into when reading files back.
- `.readline()` returns just the next single line, including its trailing `\n`, and moves forward one line each time you call it again — useful when you want to peek at only the first line or two without pulling in the whole file.
- `.readlines()` returns every line as a list of strings, each one still ending in `\n` — handy when you specifically need list operations, like counting lines or indexing into a particular one.

Often, though, the most natural way to read a file is to loop over it directly, one line at a time, rather than calling any of the three methods above:

```python
with open("order_summary.txt", "r") as file:
    for line in file:
        print(line)
```

`for line in file:` visits the file one line at a time, binding each one — again, still carrying its own `\n` — to `line`. This is the style you will reach for constantly: it never loads more of the file into memory at once than it needs to, which matters once a file grows from three lines to three million.

Before checking, predict what would print if `order_summary.txt` held three separate lines and you ran this loop — you would see three lines of actual text, each followed by one blank line, for exactly the same doubled-newline reason as `.read()` above.

## Writing to a file

You have already seen the tool: a file object's `write()` method takes a single string and appends it to the file. **`write()` never adds a newline character on its own — if you want each write to end its own line, you must include `\n` in the string yourself.** Forget it, and everything you write keeps running onto the same line, no matter how many separate `write()` calls you make.

```python
with open("cart_summary.txt", "w") as file:
    file.write("Item: Wireless Mouse, Qty: 1, Rs.799\n")
    file.write("Item: USB Cable, Qty: 2, Rs.198\n")

with open("cart_summary.txt", "r") as file:
    print(file.read())
```

```
Item: Wireless Mouse, Qty: 1, Rs.799
Item: USB Cable, Qty: 2, Rs.198

```

Say out loud, before scrolling back up, what this same output would have looked like with both `\n` characters removed — every item would have run together onto a single unreadable line, `Item: Wireless Mouse, Qty: 1, Rs.799Item: USB Cable, Qty: 2, Rs.198`, which is exactly the kind of bug a missing `\n` causes in practice.

## Rows and columns as text: the `csv` module

**CSV**, short for Comma-Separated Values, is a plain-text file format that stores rows and columns as text, with the values in each row separated by commas — precisely the shape you get whenever a spreadsheet, a bank statement, or an order history is exported as plain text. A small CSV file, with a header row followed by two data rows, looks like this on disk:

```
Passenger,Seats
Priya Nair,2
Arjun Rao,0
```

Python's built-in `csv` module, loaded with `import csv`, understands this format well enough to handle tricky cases correctly — a name containing a comma inside quotes, for instance, which naive comma-splitting would get badly wrong. Two tools from it matter most:

- `csv.reader(file)` wraps an already-open file object so that iterating over it yields each row as a **list** of strings.
- `csv.DictReader(file)` wraps an open file object so that iterating over it yields each row as a **dictionary**, with keys taken automatically from the file's first, header row.

```python
import csv

with open("bookings.csv", "r", newline="") as file:
    reader = csv.DictReader(file)
    for row in reader:
        seats = int(row["Seats"])
        status = "Confirmed" if seats >= 2 else "Waitlisted"
        print(f"{row['Passenger']}: {status}")
```

```
Priya Nair: Confirmed
Arjun Rao: Waitlisted
```

`newline=""` is an argument passed to `open()` itself, not to `csv.DictReader` — the `csv` module's own documentation requires it, so that the module can manage line endings itself rather than rows occasionally splitting incorrectly on some systems.

**Every single value read out of a CSV row arrives as a string, even one that looks exactly like a whole number — CSV has no concept of numeric types at all, only text.** That is precisely why `row["Seats"]` is wrapped in `int()` above before it is compared with `>=`; skip that conversion and you would be comparing the text `"2"` against the number `2` in a way that never behaves the way arithmetic should.

`csv.reader()`, unlike `csv.DictReader()`, does **not** skip the header row automatically — the very first row it yields is the header itself, `['Passenger', 'Seats']`. Feed that straight into `int()` and Python raises a `ValueError`, because the text `"Seats"` obviously cannot become a number:

```python
import csv

with open("bookings.csv", "r", newline="") as file:
    reader = csv.reader(file)
    for row in reader:
        seats = int(row[1])
        print(row[0], seats)
```

```
ValueError: invalid literal for int() with base 10: 'Seats'
```

The fix is to consume that header row once, with `next(reader)`, before your own loop begins:

```python
import csv

with open("bookings.csv", "r", newline="") as file:
    reader = csv.reader(file)
    header = next(reader)          # reads and discards the header row
    for row in reader:
        seats = int(row[1])
        print(row[0], seats)
```

```
Priya Nair 2
Arjun Rao 0
```

**Whenever a CSV read crashes with a `ValueError` on its very first row, an un-consumed header row is almost always the cause — check that before assuming your data itself is broken.**

The `csv` module writes files too, which matters just as much: an "export my transactions" button on a real banking app's website is exactly this, running on a server. `csv.writer(file)` wraps a file opened in `"w"` mode, and its `writerow(list_of_values)` method writes one row at a time:

```python
import csv

with open("bookings.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["Passenger", "Seats"])   # header row, written first, by hand
    writer.writerow(["Priya Nair", 2])
    writer.writerow(["Arjun Rao", 0])
```

Note that the header row here is not automatic — a `csv.writer` writes exactly the rows you hand it, in order, so if you want a header at all, you write it yourself, first.

## Structured data across the web: JSON

**JSON**, short for JavaScript Object Notation, is a plain-text data format for storing key-value pairs and lists, structurally close to a Python dictionary — and it is the format most web services, banking APIs, and AI/ML tools use to exchange data with each other. Where CSV is naturally shaped like one flat table, JSON is naturally shaped like the nested dictionaries and lists you already know from Module III:

```json
{
  "upi_id": "ananya@upi",
  "amount": 500,
  "status": "Success"
}
```

Python's built-in `json` module gives you two matched pairs of functions:

| Function | Direction | Works on |
|---|---|---|
| `json.dump(data, file)` | Python object → JSON | an already-open file |
| `json.load(file)` | JSON → Python object | an already-open file |
| `json.dumps(data)` | Python object → JSON | a string sitting in memory |
| `json.loads(text)` | JSON → Python object | a string sitting in memory |

Converting a Python object into JSON is called **serialization**; converting JSON back into a Python object is called **deserialization**. The trailing `s` on `dumps`/`loads` stands for "string" — reach for the plain versions when a file is involved, and the `s` versions when you already have JSON text sitting in a variable, perhaps because it came back from a UPI payment gateway's API response rather than a file on disk.

```python
import json

payment = {"upi_id": "ananya@upi", "amount": 500, "status": "Success"}

with open("payment.json", "w") as file:
    json.dump(payment, file, indent=2)

with open("payment.json", "r") as file:
    data = json.load(file)
    print(data["upi_id"], data["amount"], data["status"])
```

```
ananya@upi 500 Success
```

`indent=2` is a purely cosmetic, optional argument that produces human-readable, indented JSON text rather than one dense unbroken line — it has no effect whatsoever on the data itself.

**The single biggest practical difference between JSON and CSV: JSON preserves real types.** A JSON number is read back as a genuine Python `int` or `float`, `true`/`false` becomes a Python `bool`, and `null` becomes `None` — so `data["amount"]` above comes back as the integer `500`, not the string `"500"` a CSV row would have handed you for the identical value.

| | CSV | JSON |
|---|---|---|
| Natural shape | One flat table — rows and columns | Nested key-value pairs and lists |
| Value types on read | Always strings, regardless of appearance | Real types preserved — `int`, `float`, `bool`, `None` |
| Typical source | A spreadsheet or report export | A web/API request or response, a config file |
| Python tools | `csv.reader`, `csv.DictReader`, `csv.writer` | `json.load`, `json.dump`, `json.loads`, `json.dumps` |

Because JSON's shape is just Python's own dictionaries and lists written out as text, nesting works exactly as it already does elsewhere in this course — a JSON value can itself be another set of key-value pairs, or a list of them:

```python
import json

order = {
    "order_id": "SWG10234",
    "items": ["Butter Naan", "Paneer Tikka"],
    "total": 398
}

text = json.dumps(order)
print(text)

restored = json.loads(text)
print(restored["items"][0])
```

```
{"order_id": "SWG10234", "items": ["Butter Naan", "Paneer Tikka"], "total": 398}
Butter Naan
```

`json.dumps(order)` turns the whole dictionary into one JSON string, with the nested list preserved intact; `json.loads(text)` turns that same string back into an ordinary Python dictionary, and `restored["items"][0]` is accessed exactly the way any list nested inside a dictionary already would be.

A handful of mistakes are worth watching for deliberately while these two modules are still new:

- Forgetting to consume the header row with `csv.reader()`, causing a `ValueError` on the very first row processed.
- Treating a CSV value as a number without converting it first, since every CSV value is a string regardless of how it looks.
- Confusing `dump`/`load` with `dumps`/`loads` — the plain pair expects an open file; the `s` pair expects a string. Handing a file object to `json.loads()`, or a string to `json.load()`, raises a `TypeError`.
- Opening a file that doesn't exist in `"r"` mode, raising `FileNotFoundError` — a mistyped path or a file created in the wrong folder is almost always the reason.
- Opening an existing file in `"w"` mode by accident, silently erasing everything it held, with no warning and no way to undo it.

## Try it yourself

Do this in a Colab cell before moving on. Model a small e-commerce order export end to end. First, write a plain-text file `order_note.txt` containing one line confirming an order number and total, using `with` and `write()` — remembering the `\n`. Then, given an `orders.csv` file with a header row `OrderID,Total` followed by three data rows, use `csv.DictReader` to print each order's ID together with `"High Value"` if its total, converted with `int()`, is 500 or more, and `"Standard"` otherwise. Finally, build a Python dictionary describing one specific order — an ID, a list of item names, and a total — and save it as `order.json` using `json.dump(..., indent=2)`, then read it back with `json.load()` and print the first item in the list, confirming the total comes back as a genuine integer rather than a string.

---

### Key Terminology

- **File** — a named collection of data stored permanently on disk, unlike a variable, which exists only in RAM while a program runs.
- **Encoding** — a scheme, such as UTF-8, mapping each character in a text file to a specific sequence of bytes.
- **Absolute path** — a file's complete location starting from the drive's root; breaks if the project moves.
- **Relative path** — a file's location relative to wherever the program is currently running; portable across machines.
- **File object** — the object `open()` returns, used to read from or write to the file it represents.
- **Mode** — the string passed to `open()` — `"r"`, `"w"`, `"a"`, or `"x"` — telling Python what you intend to do with the file.
- **Buffer** — a temporary block of memory holding written data before it is flushed to disk on `close()`.
- **Context manager** — an object defining a setup action and a guaranteed cleanup action around a block of code.
- **`with` statement** — the syntax that invokes a context manager, guaranteeing a file closes when its block ends, error or not.
- **CSV (Comma-Separated Values)** — a plain-text format storing rows and columns as text, with commas separating each row's values.
- **JSON (JavaScript Object Notation)** — a plain-text format storing key-value data close to Python's own dictionaries and lists.
- **Serialization** — converting a Python object into a stored format such as JSON text (`json.dump`/`json.dumps`).
- **Deserialization** — converting stored data, such as JSON text, back into a Python object (`json.load`/`json.loads`).
- **`FileNotFoundError`** — raised when opening, in read mode, a file that doesn't exist at the given path.
- **`FileExistsError`** — raised when opening, in `"x"` mode, a file that already exists.

### Mastery Checkpoint

Before moving to Unit 5.2, check that you can answer these without looking back:

1. What actually happens to a file's existing content the instant you open it in `"w"` mode, versus `"a"` mode?
2. Why does a forgotten `close()` on a manually opened file sometimes leave written data missing entirely, even though `write()` was called successfully?
3. What does `with open(...) as file:` guarantee that manual `open()`/`close()` cannot, and what specific scenario makes that guarantee matter?
4. A CSV read crashes with `ValueError: invalid literal for int() with base 10: 'Seats'` on its very first row. What almost certainly caused this, and how do you fix it?
5. What is the single biggest practical difference between how CSV and JSON hand back a value like the number `500`?

### Summary

You now know why a file, unlike a variable, survives past the program that created it — with the Colab-specific caveat that a runtime reset can still take an unsaved file with it — and how `open()`, its four modes, and the `with` statement work together to read and write one safely, always closing it even when something goes wrong partway through. You've read a file back three different ways, written to one while remembering the newline `write()` never adds for you, and used the `csv` and `json` modules to move data in and out of the two formats you will meet constantly in real software: rows-and-columns exports, and the nested, type-preserving structure of a web API response. From here, the next step is learning what to do when one of these operations fails outright — starting with Unit 5.2, Errors & Exceptions.

### Additional Resources

- [Python Tutorial — official docs: "Reading and Writing Files"](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)
- [Python 3 Documentation — Built-in Functions: `open()`](https://docs.python.org/3/library/functions.html#open)
- [Python 3 Documentation — `csv` Module](https://docs.python.org/3/library/csv.html)
- [Python 3 Documentation — `json` Module](https://docs.python.org/3/library/json.html)
- [Python 3 Documentation — Built-in Exceptions (`FileNotFoundError`, `FileExistsError`)](https://docs.python.org/3/library/exceptions.html)
- [W3Schools — Python File Handling](https://www.w3schools.com/python/python_file_handling.asp)
- [W3Schools — Python JSON](https://www.w3schools.com/python/python_json.asp)
