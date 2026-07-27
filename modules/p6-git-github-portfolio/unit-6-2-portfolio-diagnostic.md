# Portfolio & Diagnostic

Unit 6.1 taught you the basic Git/GitHub loop — repository, commit, push — through the web interface. That loop is only useful if you actually have somewhere of your own to keep using it. Everything you practised in Unit 6.1 lived in disposable, one-off repositories: created for a single exercise, used once, and quietly forgotten the moment the exercise ended. Nobody builds a career, or even a job application, on a folder of throwaway practice repos with names like `hw3` and `test`.

This unit closes that gap, and in doing so it closes out the entire Python Foundations course. You are not about to learn a new Python keyword or a new Git command — every single skill this unit needs, you already have. What you are about to do is take everything you built since Unit 1.1 — variables, conditionals, loops, functions, lists, dictionaries, classes, file handling, exceptions, and now version control — and organise it somewhere a stranger can actually see it, trust it, and judge it in under a minute. That "somewhere" is your **portfolio repository**: one permanent, public GitHub repository you create today and keep adding to for the rest of the AI Native Engineering programme, not just for this course.

This unit also introduces the one checkpoint standing between you and Part B: the **Part A diagnostic**, a short script combining a Python skill and a Git skill, reviewed by your instructor directly on GitHub before you're signed off to continue. Think of everything that follows as a single afternoon's work that turns six modules of practice into one thing you can point to and say, with evidence behind it, "I built this."

---

## Why a scattered GitHub profile costs you more than you think

Picture two students finishing this exact course. Both can write a class without hesitation, both can read a CSV file and handle a bad row gracefully, both know the add-commit-push loop cold. One of them has a GitHub profile that is empty except for a handful of practice repositories with vague names and no README. The other has a single, well-organised repository: a clear title, a short description of what it holds, a handful of sensibly named project folders, and a commit history stretching back weeks with specific, readable messages.

Now picture a recruiter at a TCS-style service company, or a hiring manager at a smaller product startup, screening a stack of fresher resumes. They do not have time to interview everyone in depth, and they know that marks and certificates only say so much. What they often do instead — quickly, before the interview even starts — is open the GitHub link on the resume. Thirty seconds later, one of these two students has visibly proven their skill. The other's skill is exactly as real, but it is invisible, because there was nowhere for it to be seen.

That is the entire problem a portfolio repository solves. A resume is a static claim about what you know. **A portfolio repository is evidence** — an always-growing, always-checkable record that a recruiter, a mentor, or an instructor can open and verify for themselves, without ever having to take your word for it. Ask yourself honestly, before reading on: if someone opened your GitHub profile right now, would it show them anything at all?

## The portfolio repository: a toolbox, not a resume

A resume freezes the moment you save it as a PDF — it describes a snapshot of you, and it stops changing the second you stop editing it. A portfolio repository behaves completely differently: it is a living toolbox that keeps growing every time you build something new, the same way a working carpenter's toolbox gains a new tool with every project rather than being packed once and sealed shut. You will keep committing to this exact repository long after this course ends — through Part B and beyond — so it becomes, over months, a genuine record of your entire journey rather than a one-time submission.

This is the key difference from every repository you touched in Unit 6.1. Those were disposable — created for one exercise, then abandoned the moment it was graded. A portfolio repository is created exactly once, using the same "New repository" workflow you already know, and it is never thrown away or started over. You will not create a second one for Part B, or a third for whatever comes after that — you keep adding to this same one.

**Set the repository to public the moment you create it, not "once it's finished" — a portfolio is never really finished, and a private repository cannot be reviewed by an instructor or shown to a recruiter at all.** Give it a clear, permanent name — something like `yourname-ai-native-portfolio` — and resist the urge to rename it every time you have a better idea; a stable name is part of what makes it findable and linkable from a resume.

## What makes a portfolio repository readable in under a minute

A good portfolio repository is not complicated — it is *predictable*. A visitor should be able to guess where something lives just from a folder's name, without opening every file to check. That predictability rests on two things: a clear structure, and a README that actually does its job.

A sensible top-level shape looks like this:

| Path | Purpose |
|---|---|
| `README.md` | The overview: who you are, what the repository holds, and where to find specific work. |
| `python-foundations/` | Practice work from this course, organised by unit or module. |
| `<project-name>/` | One folder per stand-alone project, named for what it does — `csv_summary_tool/`, never `week5` or `misc`. |
| `<project-name>/README.md` | A short, project-specific README explaining that one project and how to run it. |
| `diagnostics/` | Checkpoint submissions, including the one this unit produces. |

Keep this flat rather than deeply nested. A handful of clearly named folders sitting at the top level is far easier for a visitor to scan in a few seconds than several layers they have to click through one at a time just to find the thing they were looking for.

The **README** deserves special attention, because Unit 6.1 already told you it is usually the first, and sometimes only, thing a visitor reads — and in a portfolio, it has to do more work than it did in a one-off practice repo. It needs a clear title naming who you are, a one-line description of what the repository collects, and a short "What's Inside" list acting as an index, so a visitor can jump straight to the project that interests them instead of opening every folder in turn. A vague opening line that could describe literally anyone's repository defeats the entire purpose of writing one at all.

```markdown
# Priya Nair's AI-Native Engineering Portfolio

Python projects and exercises from the Python Foundations modules (P1-P6).

## What's Inside
- `csv_summary_tool/` — reads and summarizes transaction CSVs
- `student_records/` — class-based student record manager
- `diagnostics/` — checkpoint submissions reviewed by faculty

## Quick Start
Run the CSV summary tool:
`python csv_summary_tool/summary.py transactions.csv`
```

Notice what this short README already accomplishes: the title answers "whose repository is this," the description answers "what does it hold," the index turns a visitor into someone who knows exactly which folder to open, and the "Quick Start" line turns a passive reader into someone who could actually run your code themselves in the next thirty seconds.

## Weak repository, strong repository — the same skill, two very different first impressions

The gap between a messy portfolio and a well-organised one is not a matter of taste. It is checkable, item by item, against specific criteria:

| Aspect | Weak Repository | Strong Repository |
|---|---|---|
| README | Missing, or one vague line | Clear title, description, and index of contents |
| Folder names | `misc/`, `week3/`, `stuff/` | `csv_summary_tool/`, `student_records/` |
| Commit history | Few commits, vague messages like `update` | Frequent commits with clear, specific messages |
| File organisation | Unrelated files mixed together | One folder per project, related files kept together |
| Visibility | Private, or forgotten after creation | Public and actively maintained |

Two repositories built by students with identical skills can sit at opposite ends of this table. A repository holding `week2.py`, `week5_final.py`, and a `stuff/` folder full of loose notes and an old test file tells a reader nothing at a glance — nothing in those names hints at what the code actually does. Rename and regroup that exact same code into a `grade_calculator/` folder with its own short README, sitting alongside a `python-foundations/` folder for practice work, and you have moved into the strong column without changing a single line of the underlying Python. Presentation, here, is not decoration — it is the only thing standing between skill that gets noticed and skill that gets scrolled past.

Notice, too, that "commit history" appears in this table as its own row, not a footnote. Unit 6.1 taught you that a commit history is the full, ordered sequence of every commit ever made. In a portfolio, that history stops being a mere technical record and starts acting as *evidence*. A trail of frequent, specific commit messages — "Add Part A diagnostic — reads marks.csv and prints summary stats" — tells a reader that real work happened steadily, over time. One giant commit dumped in the night before an interview, or a string of messages reading `update`, `update2`, `final final v2`, tells a very different story, even if the code sitting at the end of both histories is identical.

## The Part A diagnostic: proving the pieces fit together

Everything above gets your existing work organised and visible. The diagnostic is the one genuinely new thing this unit asks of you — although "new" is generous, since every skill it requires, you have already practised. A **diagnostic**, also called a **checkpoint assessment**, is a small, deliberately easy task used to confirm you have reached a required skill level before being allowed to move forward. It is not designed to teach you anything, and it is not designed to be hard. Its only job is to verify readiness, the way a driving test doesn't teach you to drive — it simply confirms, before you're handed the keys to a real road, that you already can.

The **Part A diagnostic** is the specific checkpoint that closes this entire course. It asks for one short script that does two things at once, rather than proving each in isolation:

- **A Python skill** — reading a CSV file (Unit 5.1) and producing a clear summary from it, while handling malformed rows the way Unit 5.2 and Unit 5.3 taught you: skip-and-log, never crash-on-first-bad-row.
- **A Git skill** — committing that finished script to your portfolio repository with a clear message, and pushing it to GitHub (Unit 6.1), so it exists somewhere your instructor can actually open and read.

**Commit the diagnostic to your portfolio repository, not a throwaway one** — a diagnostic sitting in some disposable practice repo, however correct the code inside it, cannot be the thing your instructor reviews for sign-off. The whole point of this unit was building one permanent home for exactly this kind of submission.

A handful of exact rules govern what counts as a passing submission, and they are worth reading literally rather than loosely:

- The submission must be **working code** — it has to actually run without errors against sample input, not just look plausible on a skim.
- It must show the Python skill and the Git skill **together**, in one submission.
- It must be **pushed to your public portfolio repository** — a commit sitting only on your own machine, or hidden behind a private repository, cannot be reviewed by anyone but you.
- **Sign-off** — your instructor's confirmation that you're ready for Part B — is given only after they open the real, pushed commit on GitHub themselves. Not a description of what you did. Not a screenshot. Not a zip file emailed across. The actual commit, reviewed directly.

## Walking through what the diagnostic script might look like

There is no single correct diagnostic script — this is illustrative, not a template to copy line for line — but it helps to see the shape one might take, because it is almost exactly the robust CSV reader pattern from Unit 5.3, aimed at a fresh dataset.

Say your instructor hands you `marks.csv`, a file of student names and marks, with one row deliberately malformed to check that your script doesn't fall over the moment it meets bad data:

```python
import csv

valid_records = []
invalid_records = []

with open("marks.csv", "r", newline="") as file:
    reader = csv.reader(file)
    next(reader)                       # skip header row
    for row in reader:
        name = row[0]
        try:
            mark = int(row[1])
            if not (0 <= mark <= 100):
                raise ValueError(f"mark {mark} out of range")
            valid_records.append((name, mark))
        except ValueError:
            invalid_records.append(row)

marks_only = [mark for name, mark in valid_records]

print(f"Rows processed: {len(valid_records) + len(invalid_records)}")
print(f"Malformed rows skipped: {len(invalid_records)}")
if marks_only:
    print(f"Average mark: {sum(marks_only) / len(marks_only):.1f}")
    print(f"Highest mark: {max(marks_only)}")
    print(f"Lowest mark: {min(marks_only)}")
```

```
Rows processed: 43
Malformed rows skipped: 1
Average mark: 74.2
Highest mark: 98
Lowest mark: 31
```

Trace it the way you traced the robust readers in Unit 5.3: the `try`/`except` sits inside the loop, wrapping only the one row currently being processed, so a single bad row — a blank mark, or text where a number should be — lands in `invalid_records` without taking the rest of the file down with it. The two summary lists stay strictly separate, and every statistic at the bottom is computed only over `valid_records`, never over rows that never actually held a usable mark. Before you build your own version, ask yourself: which earlier unit taught you each individual line here? If you can answer that for every line, you already understand why this diagnostic is described as a synthesis rather than something new.

Once this script runs cleanly against the sample file, the only remaining step is the one you already know cold from Unit 6.1: commit it into your portfolio repository's `diagnostics/` folder with a specific message — something like "Add Part A diagnostic — reads marks.csv and prints summary stats" — and push it.

## Common mistakes worth avoiding deliberately

- A portfolio repository holding one giant, uncommented script instead of organised, named project folders.
- No README at all, or a one-line placeholder that could describe anyone's repository.
- A diagnostic that crashes the instant it meets the first malformed row, instead of skipping and logging it the way Unit 5.3 taught.
- Writing the diagnostic locally, testing it, and forgetting the actual commit-and-push step — leaving nothing for an instructor to review at all.
- Treating the repository as "done" the moment the diagnostic is pushed, rather than something you keep adding to for the rest of the programme.

## Try it yourself

Before moving on, actually do this rather than only reading about it. Create your permanent portfolio repository on GitHub now, set to public, with a name like `yourname-ai-native-portfolio`. Write a first-draft README with a title, a one-line description, and a "What's Inside" list — even if some of the projects it names don't exist yet. Create a `python-foundations/` folder and move one piece of practice work into it, committing with a specific message. Then create a `diagnostics/` folder and start your Part A diagnostic script: read a sample CSV of your own making, print a row count, and add the skip-and-log pattern from Unit 5.3 before you attempt any summary statistics. Commit what you have so far, even if it isn't finished — a portfolio's commit history is meant to show a process, not just a finished product that appeared from nowhere.

---

### Key Terminology

- **Portfolio repository** — a single, permanent GitHub repository created once and updated for the entire programme, rather than abandoned after one exercise.
- **Project structure** — how folders and files inside a repository are organised so their purpose is clear without opening every file.
- **README** — the file shown automatically on a repository's main page, usually the first thing a visitor reads.
- **Diagnostic / checkpoint assessment** — a small, focused task confirming a learner has reached a required skill level before advancing, without teaching anything new.
- **Part A diagnostic** — the specific checkpoint closing this course: a script combining a Python skill (reading and summarising CSV data) and a Git skill (commit and push), reviewed for sign-off.
- **Sign-off** — an instructor's confirmation, given only after reviewing your actual pushed commit on GitHub, that you may proceed to Part B.
- **Commit history as evidence** — a repository's commit log read not just as a technical record but as proof of steady, genuine effort over time.
- **Public repository** — a repository anyone with the link can view; required for both instructor review and later recruiter visibility.

### Mastery Checkpoint

Before you submit your Part A diagnostic, check that you can answer these without looking back:

1. Why is a portfolio repository created once and never abandoned, unlike the practice repositories you built in Unit 6.1?
2. A repository has no README and folders named `week3/` and `stuff/`. Using the weak-vs-strong criteria from this unit, what specifically is wrong with it, and how would you fix each problem?
3. What two distinct skills — one Python, one Git — must the Part A diagnostic demonstrate together, and why must they appear together rather than separately?
4. Why does a diagnostic script that crashes on the first malformed row fail the diagnostic, even if it produces correct output on every clean row?
5. Why can sign-off never be given based on a description, a screenshot, or a zip file of your diagnostic, no matter how convincing that description is?

### Summary

This unit closes out Python Foundations (Part A) by asking you to do something no earlier unit did: gather everything you have built — variables and types, control flow and functions, the four core data structures, object-oriented classes, robust file handling, and now version control — into one permanent, public portfolio repository with a structure and README a stranger can trust in under a minute. You have seen why a portfolio behaves like a growing toolbox rather than a static resume, what separates a weak repository from a strong one, and exactly what the Part A diagnostic demands: a script that reads a CSV, handles bad rows gracefully, and reports a clear summary, committed and pushed to that same repository for your instructor to review and sign off on directly. With that sign-off, Part A is complete — the entire Python foundation this programme is built on is now in place, and Part B is where you start putting it to work.

### Additional Resources

- [Python 3 Documentation — csv: CSV File Reading and Writing](https://docs.python.org/3/library/csv.html)
- [Python Tutorial — official docs: "Reading and Writing Files"](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)
- [Python 3 Documentation — Errors and Exceptions](https://docs.python.org/3/tutorial/errors.html)
- [W3Schools — Git and GitHub Introduction](https://www.w3schools.com/git/git_intro.asp?remote=github)
- [W3Schools — Git Push (GitHub)](https://www.w3schools.com/git/git_push.asp?remote=github)
- [W3Schools — Python JSON](https://www.w3schools.com/python/python_json.asp)
