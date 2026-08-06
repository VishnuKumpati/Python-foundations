# File Handling

**Giving Your Programs a Memory**

By the end of the OOP chapters, our Student Management System had become far more than a handful of simple classes. Students could register and enroll in courses. Teachers could manage their assignments. Administrators could create new users. Every type of account behaved differently, yet all of them shared the same core functionality through inheritance. Sensitive information, like a password, stayed protected through encapsulation. Different accounts responded to the same action — `login()` — in their own way, through polymorphism.

From an Object-Oriented Programming point of view, our application had become genuinely well structured.

And yet it still suffered from one enormous limitation.

**It couldn't remember anything.**

---

## A Problem That Doesn't Appear Immediately

Here is the `User` and `Student` class exactly as we've been building it throughout this book:

```python
class User:
    def __init__(self, name, email, password=None):
        self.name = name
        self.email = email
        self.__password = password

    def login(self):
        print(f"{self.name} logged in.")

    def logout(self):
        print(f"{self.name} logged out.")

    def get_password(self):
        return self.__password

    def set_password(self, new_password):
        self.__password = new_password


class Student(User):

    def __init__(self, name, email, student_id):
        super().__init__(name, email)
        self.student_id = student_id
        self.courses = []

    def enroll_course(self, course):
        self.courses.append(course)
        print(f"{self.name} enrolled in {course}.")
```

Let's use it the ordinary way:

```python
student = Student("Rahul", "rahul@example.com", "S101")

student.enroll_course("Python Programming")
student.enroll_course("Database Systems")

print(student.courses)
```

```text
Rahul enrolled in Python Programming.
Rahul enrolled in Database Systems.
['Python Programming', 'Database Systems']
```

Everything works exactly as expected. Rahul has an account, and he's enrolled in two courses.

Now suppose Rahul finishes studying for the day and closes the program. The next morning, he opens it again. What should happen? Most people would naturally expect Rahul's account to still be there — his two courses still listed, his profile looking exactly the way he left it.

Let's see what actually happens when the program is run again, fresh:

```python
print(student.courses)
```

```text
NameError: name 'student' is not defined
```

Rahul is gone. Not just his courses — his entire account.

---

## The Application Didn't Forget Rahul — Python Did Exactly What It Was Supposed To

At first, this looks like a bug in Python. It isn't. Python behaved exactly as it always has.

Think about every program we've written throughout this entire book. Every single one of them followed the same shape:

```text
Start Program
      │
Create Objects
      │
 Use Objects
      │
 Program Ends
```

When a program ends, **everything it created during that run disappears** — every `Student`, every `Teacher`, every `Admin`, every list, every dictionary, every variable. Not because Python deleted them out of spite, but because the program that created them no longer exists, and nothing exists to hold onto them anymore.

We never noticed this limitation before now, because every example in this book started from scratch, used its objects, and then ended. The next example started from scratch again. Nothing ever needed to survive between one run and the next.

Real applications cannot work that way.

---

## Real Applications Need Memory

Think about any online learning platform you've actually used. You create an account today. You log in again tomorrow. A week later you enroll in another course. A month later you finish an assessment.

If the application forgot everything the moment it closed, you would have to create a brand-new account every single day, and every course you'd ever enrolled in would vanish overnight. That is obviously not how real software behaves.

Real applications remember. The question that actually matters isn't *whether* they remember — it's **how**.

---

## Where Have Our Objects Actually Been Living?

To answer that, we first need to understand where our objects have been sitting all along.

Every object we've ever created in Python — every `Student`, every `Teacher`, every list of enrolled courses — lives inside the computer's main memory, commonly called **RAM** (Random Access Memory).

When we write

```python
student = Student("Rahul", "rahul@example.com", "S101")
```

Python builds a `Student` object and stores it in RAM. You can think of RAM as the application's workspace — everything the program is actively using sits there, because RAM is extremely fast to read from and write to.

```text
RAM
+--------------------------------+
| Student("Rahul")               |
| Teacher("Ananya")              |
| Admin("Priya")                 |
| Lists, dictionaries,           |
| and every other object         |
| the program is using right now |
+--------------------------------+
```

Every method we've called throughout this whole book has worked with objects sitting in exactly this place. When Rahul enrolled in a course, that course was appended to a list already living in RAM. Nothing was written anywhere else.

---

## RAM Is Fast, But It Doesn't Last

RAM is built for speed, not permanence. The moment the application stops running, the operating system takes that memory back and gives it to something else.

```text
Application Starts
        │
        ▼
  Objects Created
        │
        ▼
   Stored in RAM
        │
        ▼
 Application Closes
        │
        ▼
   RAM Released
```

Everything that was stored there is gone. That is exactly what happened to Rahul — his account was never deleted. It simply only ever existed for as long as the application was running.

This property has a name: **volatile memory**. Volatile memory doesn't preserve its contents once the program that's using it stops. RAM is excellent for actively working with data. It is not designed for keeping data around permanently.

---

## So Where Should Permanent Information Actually Live?

If RAM can't do the job, our application needs a second place to keep anything that has to survive between runs. That place is the computer's **storage device** — a hard disk or SSD.

Unlike RAM, storage is designed to hold onto information even after the application that wrote it has closed.

| | RAM | Storage |
|---|---|---|
| Speed | Very fast | Much slower |
| Lifespan | Temporary — cleared automatically | Permanent — preserved |

Real applications use both, moving data between them as needed. When Rahul logs in, his information is loaded from storage into RAM. While the application runs, Python works with that information in RAM, because RAM is faster. When Rahul enrolls in another course, the updated information gets written back to storage. The next time the application starts, that updated information is loaded again.

```text
Storage
   │
   ▼  load
  RAM
   │
   ▼  modify
  RAM
   │
   ▼  save
Storage
```

This load → modify → save cycle is followed, in one form or another, by almost every application you use every single day.

---

## Introducing Files

Applications don't write Python objects directly onto a disk. Instead, they store information inside **files**.

> A **file** is a named collection of data stored permanently on a storage device.

Our Student Management System might eventually contain files like these:

```text
accounts.txt
login_log.txt
attendance.csv
profiles.json
```

Each one stores a different kind of information, and — unlike anything living in RAM — every one of them keeps existing after the application closes. Tomorrow, next week, or next year, the application can open these same files and pick up exactly where it left off.

For the first time in this book, our application has somewhere to actually remember things.

---

## One Question Still Remains

We now know **where** Rahul's information should live: inside a file, on disk, instead of only in RAM.

But knowing a file exists isn't the same as knowing how to talk to it. Our Python program still has no way to open a file, read what's inside it, or save new information into it.

Everything from here begins with a single built-in function.

---

## Teaching Python to Open a File

Python provides exactly one built-in function for this: `open()`.

```python
file = open("accounts.txt")
```

This one line looks simple, but several things are quietly happening behind it:

```text
Student Management System
            │
            ▼
    open("accounts.txt")
            │
            ▼
 Operating system finds the file
            │
            ▼
 Python creates a file object
            │
            ▼
 Program can now work with the file
```

Notice what *hasn't* happened yet. Python has not read Rahul's account, and it has not written anything. It has only opened a connection between our program and the file — and that connection is represented by a **file object**.

> A **file object** is not the file itself. It is a Python object that represents an open connection between your program and the file stored on disk. Every read, write, or close operation you perform happens through this object — never directly on the file.

```text
accounts.txt (on disk)
       ▲
       │
  File Object
       ▲
       │
Python Program
```

From this point on, every single operation we perform — reading, writing, moving through the file, or closing it — happens through this file object. Without it, Python has no way to talk to the file at all.

### Reading Is the Default

Simply writing `open("accounts.txt")` works because Python assumes something on your behalf: that you want to **read** the file. This is exactly the same as writing it out in full:

```python
open("accounts.txt", "r")
```

`"r"` stands for **read mode**, and it tells Python: *"open this file because I want to see what's inside it."* It does not let us modify the file at all. If the file doesn't exist yet, Python raises an error, because there's nothing to read.

---

## Reading the Entire File

Suppose `accounts.txt` already contains three registered names, one per line:

```text
Rahul
Ananya
Priya
```

To pull the whole thing into our program, we use `.read()`:

```python
file = open("accounts.txt", "r")
data = file.read()
print(data)
```

```text
Rahul
Ananya
Priya
```

`data` now holds the entire contents of the file — as a single string:

```python
print(type(data))
```

```text
<class 'str'>
```

The file itself is completely unaffected. Reading only *copies* information into our program; it never removes or changes what's stored on disk. Think of opening a notebook: reading a page never erases what's written on it. Files behave exactly the same way.

### Why Did the Second Read Return Nothing?

Try this:

```python
file = open("accounts.txt")

print(file.read())
print(file.read())
```

```text
Rahul
Ananya
Priya

```

The second call returned an empty string. A beginner's first guess is usually "the file must have become empty" — but it hasn't. Open `accounts.txt` in any editor and Rahul, Ananya, and Priya are all still there.

Something else changed: Python remembers *where it stopped reading*. When the first `.read()` finished, it had already reached the very end of the file. The second `.read()` simply continued from that same spot — and there was nothing left. We'll see exactly how Python tracks this a little later in the chapter, once we've built up enough context to make sense of it. For now, just remember the rule:

> Every call to `.read()` continues from wherever the *previous* read stopped.

---

## Reading Only Part of a File

`.read()` accepts an optional number, telling it exactly how many characters to pull:

```python
file = open("accounts.txt")
print(file.read(5))
```

```text
Rahul
```

Instead of reading the whole file, Python read only the first five characters. Call `.read()` again right after, and it continues from that same point — it does not jump back to the beginning automatically.

---

## Reading One Line at a Time

Our account file stores one name per line. Instead of pulling everything in as one giant block, it's often more useful to process a single line at a time. That's exactly what `.readline()` does:

```python
file = open("accounts.txt")

print(file.readline())
print(file.readline())
print(file.readline())
```

```text
Rahul

Ananya

Priya
```

Each call returns the *next* line in the file. Just like `.read()`, every call moves forward — it never repeats a line you've already seen.

---

## Reading Every Line as a List

Sometimes you want every line kept separately, rather than merged into one long string. `.readlines()` does that:

```python
file = open("accounts.txt")
accounts = file.readlines()
print(accounts)
```

```text
['Rahul\n', 'Ananya\n', 'Priya\n']
```

Every line becomes one element of a Python list. Notice each element still ends in `\n` — that's the actual newline character stored in the file, not something Python added. If you don't want it:

```python
for account in accounts:
    print(account.strip())
```

```text
Rahul
Ananya
Priya
```

---

## The Way You'll Actually See Files Read in Real Code

`.readlines()` is useful, but most real applications don't load an entire file into a list before working with it. Instead, they process one line at a time, straight from the file object:

```python
with open("accounts.txt") as file:
    for account in file:
        print(account.strip())
```

```text
Rahul
Ananya
Priya
```

The `for` loop automatically pulls one line after another. This uses far less memory on a large file, and it's the style you'll see used most often in professional Python code. (We'll come back to that `with` in a moment — for now, just notice the loop itself.)

### Which Reading Method Should You Use?

| Need | Method |
|---|---|
| Read everything at once | `.read()` |
| Read a fixed number of characters | `.read(size)` |
| Read just one line | `.readline()` |
| Read every line into a list | `.readlines()` |
| Process a large file, one line at a time | `for line in file` |

Don't try to memorize this table. Instead, ask one question: **how does my program actually need to process this file?** The right method usually falls out of the answer on its own.

Our application can now load information that already exists. But what happens the very first time Rahul registers, when there's nothing saved about him yet at all? Reading can't help there — the application needs a way to **save** information for the first time.

---

## Saving Information for the First Time

To save data instead of reading it, we open the file in **write mode** — `"w"` instead of `"r"` — and use `.write()`:

```python
student = Student("Rahul", "rahul@example.com", "S101")

file = open("accounts.txt", "w")
file.write(f"{student.name},{student.email},{student.student_id}")
file.close()
```

`accounts.txt` now contains:

```text
Rahul,rahul@example.com,S101
```

Instead of asking the file for information, `.write()` gives the file information to store.

### Where Did the Old Data Go?

Suppose `accounts.txt` already had Rahul's row saved in it from yesterday, and today Kabir registers:

```python
file = open("accounts.txt", "w")
file.write("Kabir,kabir@example.com,S102")
file.close()
```

Open the file again:

```text
Kabir,kabir@example.com,S102
```

Rahul's row is gone completely. Many beginners assume `.write()` *added* Kabir's line to the file. It didn't — the disappearance happened earlier, the moment the file was *opened* in `"w"` mode. `"w"` immediately clears whatever was already in the file, before `.write()` ever runs. Only after that does `.write()` start filling in the now-empty file.

> Think of `"w"` as saying: *"start with a completely empty file."* Everything written afterward goes into that fresh, blank file.

### One File, Many Writes

Write mode only clears the file *once*, at the moment it's opened — not every single time `.write()` is called:

```python
file = open("accounts.txt", "w")
file.write("Rahul\n")
file.write("Ananya\n")
file.write("Priya\n")
file.close()
```

```text
Rahul
Ananya
Priya
```

Every `.write()` call added onto the same file. The file was never re-cleared between calls — only once, right when `open(..., "w")` ran.

### Why Did Everything Appear on One Line?

Remove the newlines and try again:

```python
file = open("accounts.txt", "w")
file.write("Rahul")
file.write("Ananya")
file.write("Priya")
file.close()
```

```text
RahulAnanyaPriya
```

`print()` automatically moves to a new line after each call. `.write()` does not — it stores *exactly* the string you hand it, nothing more. If you want each entry on its own line, you have to include `\n` yourself. This one small difference is responsible for one of the most common beginner mistakes in file handling.

---

## When Write Mode Becomes the Wrong Choice

Every time a user logs in, suppose our application records that event:

```text
login_log.txt

Rahul logged in
Ananya logged in
```

Now Priya logs in too. Should Rahul's earlier login disappear because of it? Obviously not — a login history is supposed to keep growing over time. But watch what happens if we save Priya's login using `"w"`:

```python
file = open("login_log.txt", "w")
file.write("Priya logged in\n")
file.close()
```

```text
Priya logged in
```

Yesterday's history is gone. Write mode solved the problem of *saving* information — but it introduced a new one: every save replaces everything that came before it. A login history clearly needs a different approach — one that keeps the old contents and simply adds to them.

```text
Need to replace old contents
            │
            ▼
      Write Mode ("w")

Need to keep old contents,
and add new information
            │
            ▼
     Append Mode ("a")
```

---

## File Modes — Telling Python What You Intend to Do

We've now opened the same kind of file two different ways — once to read it, once to overwrite it — and each time, changing a single letter completely changed what happened. Clearly, opening a file isn't enough on its own. Python also needs to know **what you intend to do with it**, and that instruction is called the **file mode**.

If Python always assumed you wanted to read, saving would never be possible. If it always assumed you wanted to write, simply opening a file to look at it would erase its contents instantly. Instead of guessing, Python asks you to say so explicitly, every time.

| Mode | Purpose | If the file doesn't exist | If the file already has content |
|---|---|---|---|
| `"r"` | Read an existing file | Raises `FileNotFoundError` | Left completely untouched |
| `"w"` | Replace the contents of a file | Creates it | Erased first — starts from a blank file |
| `"a"` | Keep existing contents, add new data | Creates it | Kept — new writes go after it |
| `"x"` | Create a file only if it doesn't already exist | Creates it | Raises `FileExistsError` — refuses to touch it |

### Append Mode in Action

```python
file = open("login_log.txt", "a")
file.write("Priya logged in\n")
file.close()
```

```text
Rahul logged in
Ananya logged in
Priya logged in
```

Python didn't remove anything this time. It moved straight to the end of the file and continued writing from there. Append mode is exactly right for anything that naturally grows over time — login histories, attendance records, audit logs. It also creates the file automatically if it doesn't exist yet, so it works equally well for the very first login or the thousandth.

### Protecting a File You Never Want Overwritten

Imagine our system generates Rahul's final grade report as `Grade_Report_Rahul.txt`. That file should only ever be created once — if the report generator accidentally runs a second time, silently overwriting it with `"w"` could destroy something important. This is exactly what `"x"` (exclusive create) is for:

```python
file = open("Grade_Report_Rahul.txt", "x")
```

If the file doesn't exist, Python creates it normally. If it already exists, Python refuses outright:

```text
FileExistsError: [Errno 17] File exists: 'Grade_Report_Rahul.txt'
```

Unlike `"w"`, `"x"` will never silently destroy an existing file — it acts as a safety net against exactly that kind of accident.

### Choosing the Right Mode

```text
Need to view information?        →  Read       ("r")
Need to replace everything?      →  Write      ("w")
Need to preserve old data?       →  Append     ("a")
Need to protect an existing file? → Exclusive Create ("x")
```

Start by answering that question, and the correct mode usually becomes obvious on its own.

---

## One Behavior We Still Haven't Explained

We now know how to choose a file, open it, read from it, write to it, and pick the right mode for the job. But one thing from earlier in this chapter is still a mystery.

```python
file = open("accounts.txt")
print(file.read())
print(file.read())
```

The second `.read()` came back empty, even though the file itself was untouched. Why? The answer is something that has been quietly happening the entire time, completely out of sight: every open file keeps track of exactly *where it currently is*.

---

## The File Pointer

Imagine placing your finger on the very first letter of a page before you start reading. As you read, your finger moves forward across the page. Eventually it reaches the last character — and if someone asks you to keep reading, there's simply nothing left. Files behave in exactly this way.

When a file is opened, this invisible marker — the **file pointer** — starts at position zero, the very beginning. As `.read()` pulls characters out, the pointer keeps moving forward. By the time the first `.read()` in our example finished, the pointer had already reached the end of the file. The second `.read()` simply continued from there, found nothing left, and returned an empty string. The file never changed. Only the pointer moved.

### Seeing the Pointer — `.tell()`

`.tell()` reports the pointer's current position, as a count of characters from the start of the file:

```python
file = open("accounts.txt")
print(file.tell())
```

```text
0
```

The pointer starts at 0. Read five characters, and check again:

```python
file.read(5)
print(file.tell())
```

```text
5
```

It moved forward by exactly five. The important idea isn't the specific number — it's that Python always knows exactly where the *next* read will begin.

### Moving the Pointer — `.seek()`

Sometimes we want to read something again — say, Rahul's line, right from the top. Calling `.read()` again won't help, since the pointer is already sitting at the end. Instead, we move the pointer ourselves, using `.seek(position)`:

```python
file.seek(0)
print(file.read())
```

```text
Rahul
Ananya
Priya
```

Unlike `.read()`, which moves the pointer automatically as a side effect, `.seek()` moves it because *we* told it to.

```text
Open file
    │
    ▼
Pointer at beginning
    │
    ▼
Read data ──▶ Pointer moves forward
    │
    ▼
Want to read it again?
    │
    ▼
seek() first, then read again
```

Once you have this mental picture, the behavior of `.read()`, `.readline()`, and `.readlines()` all become far easier to predict.

---

## Letting Go of a File Properly

Every example so far has, sooner or later, needed to end its connection to the file. Once a program is done with a file, that connection should be released — and `.close()` is what does it.

```python
file = open("accounts.txt")
data = file.read()
file.close()
```

`.close()` tells Python: *"I'm finished working with this file."* The file itself keeps existing on disk — only the connection between the program and the file is removed.

Once a file is closed, Python won't let you use it anymore:

```python
file = open("accounts.txt")
file.close()
file.read()
```

```text
ValueError: I/O operation on closed file.
```

That's expected — we already told Python we were done with it.

### Is Placing `.close()` by Hand Always Enough?

Not quite:

```python
file = open("accounts.txt")
data = file.read()
print(10 / 0)
file.close()
```

The program crashes at `10 / 0`, and `.close()` — sitting right below it — never gets a chance to run. The file stays open until Python eventually cleans it up on its own. Most operating systems recover from this eventually, but relying on that is not good practice. There's a safer way.

### Trusting Python to Close It — `with`

```python
with open("accounts.txt") as file:
    data = file.read()
```

The reading code hasn't changed at all. What's different is what happens the moment Python leaves this block — whether it finishes normally, or an exception tears straight through it, **Python closes the file automatically**, every single time. We never have to remember to do it ourselves.

That reliability is exactly why you'll see `with` used in almost every piece of real Python code that touches a file. From here on, every example in the rest of this chapter uses it.

---

## Finding Files Anywhere in the Project

Every file we've opened so far has sat in the very same folder as the program itself:

```python
with open("accounts.txt") as file:
    data = file.read()
```

Python finds it immediately, because it sits right beside the program. That's fine while the application stays small — but real applications don't stay that simple for long. As our Student Management System grows, it starts collecting login histories, attendance records, reports, and profile data, and dumping all of it into one folder becomes hard to manage:

```text
StudentManagementSystem/
│
├── main.py
│
├── data/
│     ├── accounts.csv
│     ├── attendance.csv
│     ├── profiles.json
│     └── login_log.txt
│
└── reports/
      └── monthly_report.txt
```

Now the program still runs from `main.py`, but `accounts.csv` lives one folder over, inside `data/`. Python needs directions to find it — and that's exactly what a **file path** provides.

### Relative Paths

A **relative path** describes where a file is, starting from wherever the current program is running:

```python
with open("data/accounts.csv") as file:
    data = file.read()
```

Read left to right: move into the `data` folder, then open `accounts.csv` sitting inside it. That's the whole idea — a relative path describes how to reach a file *starting from the current project*.

If you copy the entire `StudentManagementSystem` folder onto a different computer, this still works, because the relationship between `main.py` and `data/accounts.csv` never changed — only their shared location did. That's why most Python projects use relative paths whenever they can.

### Absolute Paths

An **absolute path** spells out a file's complete location on the computer, instead of describing it relative to the project:

```text
C:\Users\Priya\projects\StudentManagementSystem\data\accounts.csv
```

This works from anywhere on Priya's machine — but only there, and only inside that exact folder. Hand the same script to someone else, or move the project, and the path is simply wrong. Absolute paths are useful for files that genuinely live somewhere fixed, outside the project itself; for files that travel *with* your code, relative paths are almost always the better choice.

---

## Not Every File Holds Text

Every file we've opened so far — `accounts.txt`, `login_log.txt` — has stored readable text, and every `.read()` we've called has returned a string:

```python
print(type(file.read()))
```

```text
<class 'str'>
```

That makes sense — these files are made of characters. But Rahul's profile picture isn't. Neither is a recorded lecture, or an exported PDF. Those files contain **binary data**: raw bytes that were never meant to be interpreted as characters at all.

Opening one of these as text produces scrambled nonsense, because a text reader is trying to decode data that was never characters to begin with. The fix is adding `"b"` to the mode:

```python
file = open("rahul_photo.png", "rb")
data = file.read()
print(type(data))
```

```text
<class 'bytes'>
```

`"rb"` tells Python not to try decoding the file's contents into characters at all — just hand back the raw bytes, exactly as they are. Writing works the same way, with `"wb"` in place of `"w"`.

| | Text Files | Binary Files |
|---|---|---|
| Store | Characters | Raw bytes |
| Read as | Strings | `bytes` objects |
| Mode | `"r"`, `"w"`, `"a"` | `"rb"`, `"wb"` |
| Examples | `.txt`, `.csv`, `.json` | `.png`, `.pdf`, `.mp3` |

Choosing between them isn't a matter of preference — it's determined entirely by what kind of data the file actually holds.

---

## Organizing Data Inside Files

Everything we've saved so far has been one simple line per record:

```text
Rahul logged in
Ananya logged in
```

That works when each record is just a sentence. A student account isn't just a sentence — it's a name, an email, a student ID, and possibly more, all belonging together. Storing that as plain, unstructured text quickly becomes difficult to manage correctly.

### When Data Naturally Forms a Table

Look at the accounts our system already manages:

```python
accounts = [
    Student("Rahul", "rahul@example.com", "S101"),
    Teacher("Ananya", "ananya@example.com"),
    Admin("Priya", "priya@example.com"),
]
```

Set the Python objects aside for a second and just look at the information they carry — it naturally forms a table:

| Name | Email | Role |
|---|---|---|
| Rahul | rahul@example.com | Student |
| Ananya | ananya@example.com | Teacher |
| Priya | priya@example.com | Admin |

Whenever data naturally fits into rows and columns, **CSV** — Comma-Separated Values — is usually one of the simplest ways to store it. CSV isn't a Python feature; it's just a file format, where each line is a row and commas mark the boundaries between columns:

```text
Rahul,rahul@example.com,Student
Ananya,ananya@example.com,Teacher
Priya,priya@example.com,Admin
```

### Building CSV by Hand — and Where It Breaks

A first attempt might look like this:

```python
with open("accounts.csv", "w") as file:
    for account in accounts:
        role = type(account).__name__
        file.write(f"{account.name},{account.email},{role}\n")
```

```text
Rahul,rahul@example.com,Student
Ananya,ananya@example.com,Teacher
Priya,priya@example.com,Admin
```

This looks perfect — until a field contains a comma of its own. Suppose we add a remarks column, and Priya's remark is `"Handles accounts, billing, and security"`:

```text
Priya,priya@example.com,Admin,Handles accounts, billing, and security
```

Read that line back by splitting on commas, and it falls apart. Which commas separate the columns, and which ones just happen to sit inside a sentence? There's no way to tell — the problem isn't our code, it's that commas are now being used for two completely different jobs at once.

### Let Python Handle CSV Correctly

Python's built-in `csv` module already understands exactly how a CSV file should be written:

```python
import csv

with open("accounts.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["name", "email", "role", "remarks"])

    for account in accounts:
        role = type(account).__name__
        remarks = "Handles accounts, billing, and security" if role == "Admin" else ""
        writer.writerow([account.name, account.email, role, remarks])
```

```text
name,email,role,remarks
Rahul,rahul@example.com,Student,
Ananya,ananya@example.com,Teacher,
Priya,priya@example.com,Admin,"Handles accounts, billing, and security"
```

We never added those quotation marks ourselves — `csv.writer` noticed the remarks contained a comma and automatically quoted the whole field, so it can never be mistaken for three separate columns when it's read back. (`newline=""` on `open()` isn't optional here — without it, the `csv` module and the operating system can both try to insert line endings, producing extra blank rows on some platforms.)

### Reading CSV Files Back

```python
import csv

with open("accounts.csv", newline="") as file:
    reader = csv.reader(file)
    next(reader)
    for row in reader:
        print(row)
```

```text
['Rahul', 'rahul@example.com', 'Student', '']
['Ananya', 'ananya@example.com', 'Teacher', '']
['Priya', 'priya@example.com', 'Admin', 'Handles accounts, billing, and security']
```

Each row comes back as a plain list, and `next(reader)` skips the header row before the loop starts on real data. Remembering that `row[2]` means "role" gets awkward fast — `csv.DictReader` solves that by using the header row as keys instead:

```python
with open("accounts.csv", newline="") as file:
    reader = csv.DictReader(file)
    for row in reader:
        print(row["name"], row["role"])
```

```text
Rahul Student
Ananya Teacher
Priya Admin
```

### When CSV Isn't the Answer

CSV works beautifully as long as every row has the same shape. But look at Rahul's full profile now that `Student` actually remembers its courses:

```python
student_profile = {
    "name": "Rahul",
    "email": "rahul@example.com",
    "student_id": "S101",
    "courses": ["Python Programming", "Database Systems", "Machine Learning"],
}
```

The `"courses"` field isn't one value — it's a *list*, and it can grow to any length. There's no clean way to squeeze a variable-length list into a single CSV cell. CSV is built for tables, not for nested information.

### JSON Feels Natural in Python

Notice that `student_profile` already looks exactly like a Python dictionary. That's the whole reason JSON pairs so naturally with Python — objects convert between the two with almost no effort:

```python
import json

with open("rahul.json", "w") as file:
    json.dump(student_profile, file, indent=4)
```

```json
{
    "name": "Rahul",
    "email": "rahul@example.com",
    "student_id": "S101",
    "courses": [
        "Python Programming",
        "Database Systems",
        "Machine Learning"
    ]
}
```

Loading it back reconstructs the exact same structure, with real types intact:

```python
with open("rahul.json") as file:
    student_profile = json.load(file)

print(student_profile["courses"])
```

```text
['Python Programming', 'Database Systems', 'Machine Learning']
```

`courses` comes back as an actual Python list — nothing needs to be split apart or reconstructed by hand.

| Format | Best used for |
|---|---|
| `.txt` | Simple, unstructured text |
| `.csv` | Rows and columns — a table |
| `.json` | Nested dictionaries and lists |

The shape of the data decides the format — not personal preference.

---

## Mini Project — Giving the Student Management System a Memory

Let's bring everything in this chapter together into one working example.

### Step 1 — A Student Registers

```python
student = Student("Rahul", "rahul@example.com", "S101")

with open("data/accounts.csv", "a", newline="") as file:
    writer = csv.writer(file)
    writer.writerow([student.name, student.email, "Student"])
```

The moment Rahul registers, his row is appended to `accounts.csv` — not overwritten, so every student who registers before or after him keeps their row too.

### Step 2 — Recording Every Login

```python
with open("data/login_log.txt", "a") as file:
    file.write(f"{student.name} logged in\n")
```

After a few days:

```text
Rahul logged in
Ananya logged in
Rahul logged in
Kabir logged in
```

Because we used append mode, every login was added without ever losing the ones before it.

### Step 3 — Saving Rahul's Complete Profile

```python
student.enroll_course("Python Programming")
student.enroll_course("Database Systems")
student.enroll_course("Machine Learning")

student_profile = {
    "name": student.name,
    "email": student.email,
    "student_id": student.student_id,
    "courses": student.courses,
}

with open("data/rahul.json", "w") as file:
    json.dump(student_profile, file, indent=4)
```

Rahul's full profile — including his list of courses, which a CSV row could never hold cleanly — now lives on disk, not just in RAM.

### Step 4 — Starting the Application Tomorrow

Rahul returns the next day. Yesterday's Python objects are gone; RAM is empty again. But this time, that's fine:

```python
with open("data/rahul.json") as file:
    student_profile = json.load(file)

print(student_profile["name"])
print(student_profile["courses"])
```

```text
Rahul
['Python Programming', 'Database Systems', 'Machine Learning']
```

Yesterday's data has become today's data. That is the entire purpose of file handling.

### What Really Happened

```text
Application Starts
        │
        ▼
    Load Files
        │
        ▼
 Create Python Objects
        │
        ▼
User Works With the Application
        │
        ▼
   Objects Change
        │
        ▼
   Save Changes
        │
        ▼
Application Closes
```

Objects only ever exist while a program is running. Files are what preserve information across the gap between one run and the next.

---

## Common Mistakes Beginners Make

- **Forgetting the newline character.** `file.write("Rahul")` followed by `file.write("Ananya")` produces `RahulAnanya` on one line — `.write()` never adds `\n` for you.
- **Using `"w"` when `"a"` was needed.** `"w"` erases the file before anything new is written; anything meant to grow over time — a log, a history — needs `"a"`.
- **Assuming a second `.read()` starts over.** It continues from wherever the pointer already is. Use `.seek(0)` first if you need to read a file again from the top.
- **Forgetting to close a file, or relying on a manual `.close()` that a crash can skip.** Use `with` — it closes the file automatically, no matter how the block ends.
- **Hand-splitting a line on commas instead of using `csv`.** It breaks the instant any field contains a comma of its own.
- **Choosing the wrong storage format.** Plain text for a simple line, `.csv` for a table, `.json` for anything nested — the shape of the data should decide, not habit.

---

## Key Takeaways

- RAM is fast but volatile — everything in it disappears the instant the program that created it ends. A file, written to disk, survives after the program closes.
- `open(path, mode)` returns a file object — a connection between your program and the file, not the file's contents. The mode (`"r"`, `"w"`, `"a"`, `"x"`) decides whether the file is read, overwritten, appended to, or created only if it doesn't already exist.
- `.read()`, `.readline()`, `.readlines()`, and looping directly over a file object all pull data out, differing only in how much comes back and in what shape.
- Every file object tracks an internal pointer; `.tell()` reports its position and `.seek()` moves it — which is exactly why repeated reads never repeat the same data on their own.
- `with` guarantees a file closes no matter how its block ends, success or failure — which is why it's used almost everywhere in real Python code.
- Relative paths describe a file's location starting from the project; absolute paths spell out its complete location on one specific machine.
- Text files hold characters; binary files hold raw bytes and need `"rb"`/`"wb"` to be opened correctly.
- `csv` stores tabular data without breaking on embedded commas; `json` stores nested, structured data — like a list of enrolled courses — that a flat CSV table can't represent.

---

## Looking Ahead

Throughout this chapter, we quietly assumed that everything worked perfectly. Every file existed. Every path was correct. Every row in every CSV was well-formed.

Real applications can't assume that. A user might delete a file. A folder might get renamed. A CSV export might have a row with a missing value in it. When situations like these come up, our program shouldn't simply crash — it should detect the problem, respond to it sensibly, and keep running wherever it reasonably can.

That is exactly what we'll learn in the next chapter: **Exception Handling**.
