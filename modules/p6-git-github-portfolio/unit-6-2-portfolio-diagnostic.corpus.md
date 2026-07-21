---
topic_id: 586c0ed1-3d2a-43da-bb9d-50585d9e8cdb
title: Unit 6.2 - Portfolio & Diagnostic
position_in_module: 2
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 6.2 - Portfolio & Diagnostic — Topic Corpus

## 2. Prerequisites

This topic builds directly on **Unit 6.1 - Version Control Basics** (id `3dd1881f-f6ac-41d2-b529-d4cd278d5498`). That unit introduced version control, Git, GitHub, repositories, the add/commit/push loop through the GitHub web interface, commit messages, GitHub Classroom, READMEs, commit history, the difference between public and private repositories, never committing secrets, and `.gitignore`. Everything in this unit assumes you already know what those words mean and can already create a repository, commit a change, and push it through the GitHub web interface. This unit also assumes, as baseline, the full run of Python skills built across Units 1.1 through 5.3 — variables, control flow, data structures, functions, classes, and file handling — without redefining any of that syntax again.

## 3. Learning Objectives

- **Explain** what a portfolio repository is and how it differs from the temporary, disposable practice repositories created in Unit 6.1.
- **Describe** what the Part A diagnostic checks and why it exists as a gate before Part B.
- **Apply** a clear folder structure and a README pattern to build a portfolio repository that a stranger can understand in under a minute.
- **Differentiate** between a disorganized repository and a well-organized one using specific, checkable criteria rather than a vague impression.
- **Apply** the commit discipline learned in Unit 6.1 so that your commit history itself becomes evidence of consistent, ongoing work.

## 4. Introduction

Picture two students finishing the exact same Python course. Both know the same syntax, both solved the same exercises, both can write a function or a class without hesitation. One of them has a GitHub profile that is empty except for a few practice repositories with names like `hw3` and `test`. The other has a single, tidy repository with a clear README, a handful of well-named project folders, and a steady trail of commits stretching back weeks. If a recruiter or interviewer looks at both profiles for thirty seconds each — which they often do — only one of these two students will have actually shown their skill. The other one's skill is real, but invisible.

That is the problem this unit solves. Every unit before this one taught you something new — a Python concept, or, in Unit 6.1, the basic mechanics of Git and GitHub. This unit does not teach a new skill. It asks you to take everything you have already built and organize it somewhere a stranger can see it and trust it. This unit also marks the very end of Part A of the course. Before you can move on to Part B, you will complete a small **diagnostic** — a short, deliberately easy task that checks whether your Python and Git foundations are solid enough to build on, and your instructor will review it directly in your portfolio repository before signing off.

By the end of this unit you will have a real, public, permanent repository representing your work, and a clear understanding of what the diagnostic checkpoint expects from you before you're allowed to continue.

It helps to think of this unit as a change in what "finishing" a piece of work means. In every earlier unit, finishing meant getting your code to run correctly. From here on, finishing also means making sure someone who was not in the room while you wrote it can open it, understand what it does, and trust that you wrote it yourself, over time, rather than in one rushed sitting. That second kind of finishing is what a portfolio repository and a diagnostic checkpoint are built to demonstrate.

## 5. Core Concepts

### Portfolio repository

A **portfolio repository** is a single GitHub repository that you create once and then keep updating for as long as you are learning and building software — in this case, for the rest of the entire AI Native Engineering programme, not just this course [1]. This is the key difference from the repositories you made in Unit 6.1: those were disposable practice repos, created for one exercise and then effectively abandoned. A portfolio repository is permanent. It grows, unit after unit and module after module, into a running record of everything you have built. You create it exactly once, using the same "New repository" workflow from Unit 6.1, but you never throw it away or start over.

### Project structure (folder structure)

**Project structure** refers to how the folders and files inside a repository are organized so that their purpose is clear without anyone having to open every file to figure it out [1]. A predictable structure lets a visitor guess where something lives just from a folder's name. For a portfolio repository, the recommended shape is: a top-level `README.md` giving an overview, one folder per module's practice work (for example `python-foundations/`), one folder per stand-alone project named for what that project actually does (for example `csv_summary_tool/`, not `week5` or `misc`), and a `diagnostics/` folder to hold checkpoint submissions like the one this unit produces. The structure should stay flat rather than deeply nested — a handful of clearly named top-level folders is easier for a reader to scan than several layers a visitor has to click through.

The difference between a messy structure and a well-organized one is not a matter of taste — it can be checked against specific criteria [1]:

| Aspect | Messy Repository | Well-Organized Repository |
|---|---|---|
| README | Missing, or one vague line | Clear title, description, and index of contents |
| Folder names | `misc/`, `week3/`, `stuff/` | `csv_summary_tool/`, `student_records/` |
| Commit history | Few commits, vague messages like `update` | Frequent commits with clear, specific messages |
| File organization | Unrelated files mixed together | One folder per project, related files kept together |
| Visibility | Private, or forgotten after creation | Public and actively maintained |

A repository with `week2.py`, `week5_final.py`, and a `stuff/` folder containing loose notes and an old test file is an example of the messy column: nothing in those names tells a reader what the code does, and unrelated files sit together with no explanation. Renaming and regrouping the same code into a `grade_calculator/` folder with its own short README, alongside a `python-foundations/` folder for practice work, moves it into the well-organized column — same code, completely different first impression.

### README, revisited for a portfolio

Unit 6.1 introduced the **README** as the file shown automatically on a repository's main page — usually the first, and sometimes only, thing a visitor reads. In a portfolio repository, the README does more work than in a one-off practice repo: it needs a clear title (who you are), a one-line description of what the repository collects, and a short index of contents so a visitor can jump straight to the project that interests them rather than opening every folder in turn [1]. A vague, generic README that could describe anyone's repository defeats the entire purpose of having one.

### Diagnostic / checkpoint assessment

A **diagnostic**, also called a **checkpoint assessment**, is a small, focused task used to confirm that a learner has reached a required level of skill before being allowed to move to the next stage [1]. It is not meant to teach anything new, and it is not meant to be difficult — every skill it requires has already been practised in an earlier unit. Its only purpose is to verify readiness before building further on top of it. The **Part A diagnostic** is the specific instance of this idea that closes out this course: one short task combining a Python skill (reading a file and computing a summary from it) with a Git skill (committing and pushing that result to your portfolio repository), together rather than in isolation [1].

### Sign-off

**Sign-off** is your instructor's confirmation, given only after reviewing your actual diagnostic submission, that you are ready to proceed to the next stage of the programme [1]. Sign-off is not granted for a description of what you did, a screenshot of your code, or a zip file emailed across — it requires the instructor to open your public portfolio repository on GitHub and review the real commit itself. This is why the repository must be public and the diagnostic must actually be pushed, not just written locally.

### Public repository, revisited

Unit 6.1 already introduced the distinction between public and private repositories. For a portfolio repository specifically, public visibility is not optional — a private repository cannot be reviewed by an instructor for sign-off, and it cannot be shown to a recruiter later either. The portfolio repository should be set to public from the moment it is created, not switched over "once it's finished," because a portfolio is never really finished — it is meant to be watched growing.

### Commit history as evidence

Unit 6.1 introduced **commit history** as the full sequence of commits in a repository, visible to anyone who opens it. In the context of a portfolio, commit history is not just a technical record — it functions as *evidence*. A repository with frequent, specific commit messages ("Add Part A diagnostic — reads marks.csv and prints summary stats") tells a reader that work happened steadily over time. A repository with one giant commit the night before an interview, or vague messages like `update` or `final final v2`, tells a very different story even if the underlying code is identical.

## 6. Implementation

Setting up a portfolio repository and completing the Part A diagnostic follows a concrete sequence of steps [1]:

1. **Create the repository once.** On GitHub, click "New repository" and give it a clear, permanent name, such as `yourname-ai-native-portfolio`. This is the same repository-creation step you practised in Unit 6.1, but this time it is meant to last for the whole programme, not one assignment.
2. **Set visibility to Public.** Exactly as in Unit 6.1's public-vs-private distinction — a private repository cannot later be reviewed by an instructor or shown to a recruiter.
3. **Add a first README draft.** Create `README.md` (directly on GitHub, or through the clone/commit/push loop from Unit 6.1) with a title, a one-line description, and a short "What's Inside" list acting as an index.
4. **Create one folder per project you want to showcase**, named for its contents rather than a date or a vague label — `csv_summary_tool/`, not `week5`. Move your strongest Part A work into these folders one at a time, committing after each move rather than in one giant dump, so the commit history itself shows steady progress.
5. **Create a `diagnostics/` folder** and add a short script, for example `part-a-diagnostic.py`, that reads a CSV of sample data (such as student marks) and prints a summary — row count, and simple statistics like average, highest, and lowest values — while skipping or flagging any malformed rows instead of crashing on them.
6. **Commit the diagnostic with a clear, specific message** (for example, "Add Part A diagnostic — reads marks.csv and prints summary stats") and push it to GitHub, using the exact add/commit/push loop from Unit 6.1.
7. **Notify your instructor** and share the repository link so they can open the actual commit on GitHub and review it directly.
8. **Wait for sign-off.** Only after the instructor has reviewed the real, pushed commit — not a description of it — is the diagnostic considered complete and Part A closed out.

A concrete example of a finished diagnostic script's expected console output, when run against a sample `marks.csv` with one deliberately malformed row, might look like this:

```
Rows processed: 42
Malformed rows skipped: 1
Average mark: 74.2
Highest mark: 98
Lowest mark: 31
```

This output is the tangible proof that a Python skill (reading and summarizing file data, handling bad rows gracefully) and a Git skill (commit and push to a public repository) worked together, which is exactly what the diagnostic is designed to confirm [1].

Each step in this sequence maps directly back to a Core Concept above, which is worth making explicit. Steps 1–2 create the permanent, public portfolio repository itself. Step 3 produces the README that acts as the entry point for any future reader. Step 4 applies the project-structure guidance — one project per folder, committed incrementally so the commit history shows steady progress rather than a single dump. Steps 5–6 produce the actual diagnostic submission: a script combining a Python skill with a Git skill, exactly as the diagnostic definition requires. Steps 7–8 complete the process by putting the result in front of an instructor and waiting for sign-off on the real commit, not a description of it.

### Diagnostic rules, spelled out

Because the Part A diagnostic is a gate rather than a normal exercise, what it demands is deliberately narrow but exact [1]. The submission must be working code — it has to actually run without errors against the sample input provided, not just look plausible. It must demonstrate one Python skill and one Git skill together, in the same submission, rather than proving each separately. It must be pushed to the public portfolio repository, since a commit sitting only on your own computer, or hidden behind a private repository, cannot be opened and reviewed by anyone else. And sign-off follows only from an instructor actually opening that commit on GitHub — never from a verbal description, a screenshot pasted into a chat, or a zip file sent by email.

## 7. Real-World Patterns

- **Recruitment screening.** In the Indian IT job market — across large service firms, product companies, and startups alike — recruiters and technical interviewers routinely ask for a GitHub link alongside a fresher's resume, and many open that link before the interview even begins [1]. A portfolio with clear READMEs and organized projects tends to generate follow-up questions and genuine interest; an empty or disorganized one rarely does.
- **Hiring signals beyond marks and certificates.** Banking, e-commerce, and startup employers increasingly look for evidence of hands-on coding ability, not just academic results — a well-kept portfolio is treated as a real signal of initiative and consistency, and startups in particular often weigh it heavily because they may lack the bandwidth for lengthy technical interviews [1].
- **Review by commit, not by description.** Just as your instructor will review your Part A diagnostic directly on GitHub rather than through a description or screenshot, many technical teams review a new hire's or intern's early work the same way — by reading actual commits, not summaries of them [1].
- **A record that keeps growing.** This portfolio repository does not stop mattering once Part A ends. Every later part of the programme keeps adding to the same repository, so it becomes a continuously growing record of your whole journey rather than a one-time exercise [1].

## 8. Best Practices

- Write a specific, honest top-level README — who you are, what the repository collects, and a short index of contents — and avoid a vague opening that could describe anyone's repository.
- Organize projects logically, one folder per project, named for what that project does rather than for a date or a vague label like `misc` or `week4`.
- Keep commit messages meaningful and specific, describing what changed and why, never just `update` or `final final v2`.
- Update the portfolio regularly as new work is completed, rather than doing one large dump right before it needs to be reviewed — a steady commit history is itself evidence of consistent effort.
- Keep the repository public from the very start; a private repository cannot be reviewed by an instructor or shown to a recruiter later.
- Avoid an empty or near-empty repository that is created once and never touched again — it communicates nothing about actual ability.
- Avoid dumping unrelated files (screenshots, half-finished scripts, random notes) together with no folder structure or explanation.
- Be ready to explain what is in your portfolio and why — an interviewer may open a project and ask you to walk through it live, and a repository you cannot explain is worse than no repository at all.

## 9. Hands-On Exercise

**Part 1 — Write a project entry.** Imagine a new project folder, `grade_calculator/`, containing a script that reads a CSV of student marks and prints each student's weighted average grade. Write a short "What's Inside" style README entry for it — a project name, a one-line description of what it does, and a "Quick Start" line showing the exact command to run it — such that a stranger reading only those three lines would understand the project and could try it themselves.

**Part 2 — Spot the problems.** A classmate's portfolio looks like this: `week2.py`, `week5_final.py`, and a `stuff/` folder holding a notes file and an old test script, with no top-level README. List at least three specific problems with this layout, using the messy-vs-well-organized criteria from Core Concepts.

**Part 3 — Redesign it.** Using your answer to Part 2, sketch a well-organized replacement layout: a top-level README, a `python-foundations/` folder for practice work, a sensibly named project folder with its own README, and a `diagnostics/` folder for the checkpoint submission.

## 10. Key Takeaways

- A **portfolio repository** is created once and grows for the entire programme, unlike the disposable per-assignment repos made in Unit 6.1.
- A clear, specific **README** is usually the first — and sometimes only — thing a visitor reads, so it should name who you are, what the repository contains, and where to find specific work.
- **Project structure** should let a reader predict where something lives from a folder's name alone, without needing to open every file.
- The **Part A diagnostic** combines one Python skill and one Git skill in a small, deliberately easy task — it exists only to confirm readiness, not to teach anything new.
- **Sign-off** requires an instructor to review the actual pushed commit on GitHub, not a description, screenshot, or zip file, and the repository must stay public for that review to be possible at all.

## 11. Next Steps

_This is the final topic of Python Foundations (Part A). There is no next entry in curriculum/sequence.md — the next step is Part B of the AI Native Engineering programme._
