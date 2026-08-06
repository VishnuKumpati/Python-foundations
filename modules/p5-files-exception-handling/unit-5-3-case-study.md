# Case Study — Building a Robust File Reader

No new syntax appears in this chapter. Everything here is `with open()`, `csv`, and `try`/`except` — the exact tools from the last two chapters — used together, the way a real data-loading step actually has to.

---

## A Roster With Problems In It

`accounts.csv` comes back from an export with five rows in it:

```text
name,email,role,student_id
Rahul,rahul@example.com,Student,S101
Vikram,,Student,S104
Ananya,ananya@example.com,Teacher,
Sourav,sourav@example.com,Principal,
Priya,priya@example.com,Admin,
```

Two rows are broken: Vikram's `email` column is empty, and Sourav's `role` is `"Principal"` — not one of `Student`, `Teacher`, or `Admin`. Everything else about them is fine.

---

## Attempt One — Fail-Fast

The direct approach: read every row, build the matching account from it, trust the data.

```python
import csv

accounts = []
with open("accounts.csv", newline="") as file:
    reader = csv.DictReader(file)
    for row in reader:
        if not row["email"]:
            raise ValueError(f"{row['name']} is missing an email address")
        if row["role"] == "Student":
            accounts.append(Student(row["name"], row["email"], row["student_id"]))
        elif row["role"] == "Teacher":
            accounts.append(Teacher(row["name"], row["email"]))
        elif row["role"] == "Admin":
            accounts.append(Admin(row["name"], row["email"]))

print(len(accounts))
```

```text
ValueError: Vikram is missing an email address
```

The program stops on row two. Not just row two's data — *everything after it too*. Rahul, who was already added, is fine; Ananya and Priya, both sitting below the bad row with nothing wrong with them, are never even reached. One missing email took the rest of the roster down with it.

For a one-off script, that might be tolerable. For anything that runs unattended — a nightly import, a batch job processing thousands of rows — it isn't. A single mistake made by whoever exported this CSV shouldn't be able to erase every valid record after it.

---

## Attempt Two — Fail-Soft, the Wrong Way

The instinct to fix this is usually: wrap it in `try`/`except`, and just move past whatever breaks.

```python
accounts = []
with open("accounts.csv", newline="") as file:
    reader = csv.DictReader(file)
    for row in reader:
        try:
            if not row["email"]:
                raise ValueError(f"{row['name']} is missing an email address")
            if row["role"] == "Student":
                accounts.append(Student(row["name"], row["email"], row["student_id"]))
            elif row["role"] == "Teacher":
                accounts.append(Teacher(row["name"], row["email"]))
            elif row["role"] == "Admin":
                accounts.append(Admin(row["name"], row["email"]))
        except:
            pass

print(len(accounts))
```

This runs to completion, no crash — but look at what it actually produced:

```text
3
```

Three accounts out of five, and nothing anywhere says which two vanished, or why. `except: pass` catches the error and throws it away in the same breath. Six months from now, when someone notices Vikram is missing from a report and asks why, there is no answer anywhere in this code — the evidence was deleted the moment it was caught. This is worse than the crash: the crash was at least loud.

A **robust** file reader has to keep both properties at once:

- Don't let one bad row stop rows after it from being processed.
- Don't silently erase the fact that a row failed, or why.

---

## Attempt Three — Fail-Soft, Correctly

The fix is small: instead of `pass`, keep a second list, and put something useful in it.

```python
valid_accounts = []
rejected_rows = []

with open("accounts.csv", newline="") as file:
    reader = csv.DictReader(file)
    for row in reader:
        try:
            if not row["email"]:
                raise ValueError(f"{row['name']} is missing an email address")
            if row["role"] == "Student":
                valid_accounts.append(Student(row["name"], row["email"], row["student_id"]))
            elif row["role"] == "Teacher":
                valid_accounts.append(Teacher(row["name"], row["email"]))
            elif row["role"] == "Admin":
                valid_accounts.append(Admin(row["name"], row["email"]))
            else:
                raise ValueError(f"{row['name']} has an unrecognized role: {row['role']!r}")
        except ValueError as e:
            rejected_rows.append((row, str(e)))

print(f"Loaded {len(valid_accounts)} accounts, rejected {len(rejected_rows)}.")
for row, reason in rejected_rows:
    print(f"  Skipped {row['name']}: {reason}")
```

```text
Loaded 3 accounts, rejected 2.
  Skipped Vikram: Vikram is missing an email address
  Skipped Sourav: Sourav has an unrecognized role: 'Principal'
```

Nothing conceptually new happened between this version and the last one — the `try`/`except` sits in the same place, catching the same kind of problem. The entire improvement is that the `except` block now *does something* with the failure instead of discarding it: it records the offending row alongside the exact reason, in a second list that sits right next to the list of successes.

That one change is the whole difference between a script that hides its own mistakes and one that reports them honestly.

---

## Giving the Failure Its Own Name

Reusing `ValueError` for two very different problems — a missing email and an unrecognized role — works, but it blurs them together. A CSV with a structurally wrong column is a different kind of problem from one bad value in an otherwise fine row. Distinguishing them means giving this platform's own validation failure its own identity, the way the last chapter introduced `DuplicateAccountError`:

```python
class InvalidRecordError(Exception):
    pass


def build_account(row):
    name = row["name"]
    email = row["email"]
    role = row["role"]

    if not email:
        raise InvalidRecordError(f"{name} is missing an email address")

    if role == "Student":
        return Student(name, email, row["student_id"])
    elif role == "Teacher":
        return Teacher(name, email)
    elif role == "Admin":
        return Admin(name, email)
    else:
        raise InvalidRecordError(f"{name} has an unrecognized role: {role!r}")
```

`build_account()` narrows every possible way a row can be wrong — a missing email, an unrecognized role — down to one single exception type its caller has to think about: `InvalidRecordError`. The caller's loop gets simpler, not more complicated, because it only has one thing to catch:

```python
valid_accounts = []
rejected_rows = []

with open("accounts.csv", newline="") as file:
    reader = csv.DictReader(file)
    for row in reader:
        try:
            valid_accounts.append(build_account(row))
        except InvalidRecordError as e:
            rejected_rows.append((row, str(e)))
```

---

## The Missing File Itself

Every version above still assumes `accounts.csv` exists at all. It should assume nothing:

```python
try:
    with open("accounts.csv", newline="") as file:
        rows = list(csv.DictReader(file))
except FileNotFoundError:
    print("accounts.csv not found — nothing to load.")
    rows = []

valid_accounts = []
rejected_rows = []
for row in rows:
    try:
        valid_accounts.append(build_account(row))
    except InvalidRecordError as e:
        rejected_rows.append((row, str(e)))
```

Notice this `try` sits *outside* the loop, wrapping only the file open itself — `FileNotFoundError` is a problem with the whole file, not with any individual row, so it's handled at the level that matches: once, before row-processing ever starts.

---

## Putting It Together

```python
import csv


class InvalidRecordError(Exception):
    pass


def build_account(row):
    name = row["name"]
    email = row["email"]
    role = row["role"]

    if not email:
        raise InvalidRecordError(f"{name} is missing an email address")

    if role == "Student":
        return Student(name, email, row["student_id"])
    elif role == "Teacher":
        return Teacher(name, email)
    elif role == "Admin":
        return Admin(name, email)
    else:
        raise InvalidRecordError(f"{name} has an unrecognized role: {role!r}")


def load_accounts(path):
    try:
        with open(path, newline="") as file:
            rows = list(csv.DictReader(file))
    except FileNotFoundError:
        print(f"{path} not found — nothing to load.")
        return [], []

    valid_accounts, rejected_rows = [], []
    for row in rows:
        try:
            valid_accounts.append(build_account(row))
        except InvalidRecordError as e:
            rejected_rows.append((row, str(e)))

    return valid_accounts, rejected_rows


accounts, rejected = load_accounts("accounts.csv")

print(f"Loaded {len(accounts)} of {len(accounts) + len(rejected)} rows.")
for row, reason in rejected:
    print(f"  Rejected {row.get('name', '?')}: {reason}")
```

```text
Loaded 3 of 5 rows.
  Rejected Vikram: Vikram is missing an email address
  Rejected Sourav: Sourav has an unrecognized role: 'Principal'
```

Every piece inside `load_accounts()` is a technique this course has already covered on its own — `with open()`, `csv.DictReader`, `try`/`except`, a custom exception, the `Student`/`Teacher`/`Admin` classes from the OOP chapters. What makes it a *robust* file reader isn't new syntax; it's the shape of the decisions: one bad row can't take down the rows around it, and no failure disappears without a trace.

```text
Read the file
     │
     ▼
File missing? ──▶ report it, return an empty roster
     │ no
     ▼
For each row:
     │
     ├─▶ valid   ──▶ add to valid_accounts
     └─▶ invalid ──▶ add (row, reason) to rejected_rows
     │
     ▼
Report both counts, and every rejection's reason
```

That shape — separate the valid from the invalid, and always say why something was rejected — is the same one behind real data-loading code in production systems, at any scale.

---

## Try It Yourself

1. Add a sixth row to `accounts.csv` with a valid `Teacher`, and confirm `load_accounts()` picks it up as a fourth valid account without touching the two rejections already there.
2. Extend `build_account()` to also reject a `Student` row whose `student_id` is empty, using the same `InvalidRecordError`.
3. The function below is meant to work exactly like `load_accounts()`, but it silently returns an empty roster on *any* problem, including a genuine bug in `build_account()`. Find the issue and fix it so only `InvalidRecordError` is swallowed:

   ```python
   def load_accounts(path):
       accounts = []
       with open(path, newline="") as file:
           for row in csv.DictReader(file):
               try:
                   accounts.append(build_account(row))
               except:
                   pass
       return accounts
   ```

4. Change `load_accounts()` so that instead of just printing the rejected rows, it also writes them to a `rejected_accounts.csv` file, one row per rejection, including the reason.
