---
topic_id: 3dd1881f-f6ac-41d2-b529-d4cd278d5498
title: Unit 6.1 - Version Control Basics
position_in_module: 1
generated_at: 2026-07-21
resource_count: 1
---

# 1. Unit 6.1 - Version Control Basics — Topic Corpus

## 2. Prerequisites

_No formal prerequisites._

This unit is a deliberate domain shift away from Python syntax. The only baseline it assumes is ordinary computer literacy — you know what a file is, what a folder is, and you can sign in to a website with an account (the same kind of account you'd use for a Google or email login). No command-line knowledge and no prior Python topic is required to understand what follows.

## 3. Learning Objectives

By the end of this topic, you will be able to:

- **Explain** what problem version control solves, and why silently losing an earlier version of a file is a real, recurring risk without it.
- **Differentiate** between Git and GitHub — what each one actually is, and how the two work together.
- **Describe** the conceptual add-commit-push loop and how it maps onto a series of clicks in the GitHub website.
- **Identify** the purpose of a commit message and recognize the difference between a vague one and a professional one.
- **Apply** the habit of committing small, logical, well-labeled changes rather than one large unexplained change.

## 4. Introduction

Imagine you are writing a short essay in a text editor. You save it. Ten minutes later you delete a paragraph you thought was bad, save again, and immediately realize the paragraph was actually the best part. It is gone. There is no way to get it back — your save simply overwrote it, and your editor does not remember what came before.

This exact situation happens to every single person who writes code, not just essays. You edit a file, you save it, and if that save turns out to be a mistake, the old version is gone unless you happened to keep a separate copy somewhere. Across a real software project — one with dozens or hundreds of files, changed every day for months or years — that risk is not a rare accident, it is a certainty waiting to happen.

**Version control** is the general name for the category of tools built specifically to make that risk disappear. This topic introduces the problem version control solves, the two tools this course uses to solve it — Git and GitHub — and the everyday habits (small commits, clear messages) that turn a basic user of these tools into someone who works the way a professional engineering team expects.

## 5. Core Concepts

### 5.1 The problem: work that can silently disappear

Every file on a computer can be overwritten. When you save a change, the previous content of that file is replaced — not kept somewhere as a backup. If the new version turns out to be wrong, broken, or simply worse than what was there before, there is normally no way back to the old version, because nothing was recording it. On a single small script, that might cost you an afternoon of redone work. On a real project touched by a whole team over months, it can mean losing work nobody can reconstruct, or — just as commonly — nobody being able to explain why a piece of code looks the way it does or when a mistake first appeared [1].

### 5.2 Version control

**Version control** is a category of tool that solves this problem by keeping a permanent, ordered history of every change you choose to save, instead of just keeping whatever the file looks like right now [1]. Instead of one file that gets silently overwritten, a version control system keeps every labeled snapshot you have ever asked it to keep, in order, forever. You can always go back and look at exactly what a file looked like at any earlier saved point, and compare it to what it looks like today.

### 5.3 Git

**Git** is the specific version control tool that the professional software industry has, by a huge margin, standardized on [1]. Git does not stop you from editing or deleting things in your working files — you can still change anything you like. What it does instead is keep a permanent, labeled snapshot of your project every time you deliberately tell it to, so that even after you have changed a file a hundred times since, that earlier snapshot is never truly lost. Git can run entirely on a single computer with no internet connection at all — the history it keeps lives wherever your project lives.

### 5.4 Repository

A **repository** (often shortened to "repo") is a project's folder together with its entire saved history, as tracked by Git [1]. It is the unit Git operates on: one repository per project, containing both the current files and the full record of every past checkpoint for those files.

### 5.5 Commit

A **commit** is one saved checkpoint — a snapshot of the project's files at one specific moment, permanently labeled with a message that explains what changed and why [1]. Once something is committed, that snapshot exists in the repository's history forever; committing again never erases or rewrites an earlier commit, it only adds a new one on top. This is the mechanism that directly solves the disappearing-work problem from §5.1: an earlier version is always still there, sitting in the history, even long after the file has changed many more times.

The **commit message** is the short piece of text attached to a commit describing what changed and why. It is the single biggest difference between a repository that is useful to read later and one that is not — a message like `"update"` tells a future reader nothing, while `"Fix rounding error in average marks calculation"` tells them exactly what that checkpoint was for, without needing to open a single file [1].

### 5.6 Add (staging)

**Add**, sometimes called **staging**, is the step of choosing which changes are ready to be bundled into the next commit [1]. Conceptually, it exists because you might have several changes sitting in your files at once, and you may only want some of them included in the next labeled checkpoint. On the GitHub website specifically, you do not perform this as a separate action — it happens automatically the moment you edit or upload a file through the web editor, so in practice this course treats it as an invisible step rather than a click you make yourself.

### 5.7 Push

**Push** is the act of sending your committed checkpoints to GitHub, so the online copy of your repository matches your latest committed work [1]. On the GitHub website, committing and pushing are not two separate actions you perform — they happen together the instant you click **Commit changes**, so the online repository updates immediately with no separate upload step.

### 5.8 Pull / fetch

**Pull** (sometimes called **fetch**) is the reverse of push: bringing changes made somewhere else — by you on a different device, or by a teammate — down into the copy of the project you are currently viewing or working on [1]. In a course delivered entirely through the GitHub website, you mostly experience this passively: every time you open a repository page, you are looking at its current, most up-to-date state.

### 5.9 Branch

A **branch** is a separate line of commits that does not affect the main project until it is deliberately combined back into it [1]. Every repository has a default branch, conventionally named `main`, which represents the project's stable, always-working line of work. This unit works exclusively with the `main` branch through GitHub's **Commit directly to the `main` branch** option; the deeper mechanics of creating and combining multiple branches are a professional-level skill covered later in your development, not something this topic teaches.

### 5.10 GitHub

**GitHub** is a hosting platform for projects that use Git — a website, github.com, that stores a copy of your Git history online [1]. Because the copy lives on GitHub's servers rather than only on your own device, it survives even if your laptop is lost, damaged, or replaced, and it can be seen by anyone you grant access to: an instructor, a teammate, or later in your career, a prospective employer.

The relationship to hold onto: **Git is the underlying save-checkpoint system; GitHub is a website that hosts and shares those checkpoints online** [1]. Git works completely on its own with no GitHub involved at all. GitHub, on the other hand, has nothing to host without Git underneath it. Treating the two as interchangeable — "GitHub" as just another word for version control — is one of the most common beginner mix-ups in this area, and confusing them is often read as a red flag in a technical interview.

### 5.11 GitHub Classroom

**GitHub Classroom** is a tool built on top of GitHub that instructors use to distribute and collect coursework [1]. When a student accepts a GitHub Classroom assignment, GitHub automatically creates a private repository that belongs to that student, visible only to the student and the instructor by default. This is the mechanism this course uses to hand out and receive every assignment — you will always be working inside your own private, individually created repository rather than a shared or public one.

### 5.12 Commit history

The **commit history** is the complete, ordered list of every commit ever made to a repository, viewable on GitHub's **Commits** tab [1]. Reading it answers exactly the question that opened this topic — "what did this file look like earlier, and what changed since then?" — because every past checkpoint remains individually visible and individually labeled, in the order it was made.

### 5.13 README and .gitignore

A **README** is a file — conventionally named `README.md` — that explains what a repository contains and how to use it. It is normally the first thing anyone visiting a repository sees, so its purpose is to give a stranger enough context to understand the project without reading every file inside it.

A **`.gitignore`** is a file listing items that Git should never track as part of the project's history — most importantly, secrets such as passwords or private keys. Anything left out of Git this way never becomes part of any commit, and therefore never appears anywhere in the permanent history described in §5.12.

### 5.14 Putting the terms together

It helps to walk through one small scenario using every term from this section in order, so they stop feeling like a list of separate definitions. Suppose you have a **repository** called `average-calculator`. You open it on GitHub and edit `average_calculator.py` to fix a bug — that edit is automatically **added (staged)** the moment you make it in the web editor. You scroll down, write a message explaining the fix, and click **Commit changes**: this creates a **commit**, a permanent snapshot labeled with that **commit message**, sitting in the repository's **main branch**. Because you used the website, that commit is **pushed** to GitHub in the same click — there is no separate upload to perform. A teammate working on their own copy of the same project would later **pull** that commit down to see your fix. Months later, anyone can open the **commit history** on the **Commits** tab and see this exact checkpoint, in order, alongside every other change ever made — which is the entire point of using version control in the first place (§5.1-§5.2).

## 6. Implementation

Because this course works entirely through the GitHub website, there is no command-line syntax to learn here — the "procedure" is a sequence of clicks. The conceptual loop from §5.5-§5.7 — **add, commit, push** — maps onto the browser as follows:

1. **Open or create a repository.** Accept a GitHub Classroom assignment (see §5.11), or open an existing repository you already have access to.
2. **Make a change.** Click **Add file** to create or upload a file, or click the pencil (edit) icon on a file that already exists.
3. **Add (stage) the change.** On the web interface this happens automatically the moment you edit or upload — there is no separate button to press for this step (see §5.6).
4. **Commit the change.** Scroll to the commit box at the bottom of the page, replace the default text with a clear message describing what changed and why, and choose **Commit directly to the `main` branch**.
5. **Push the change.** This happens automatically the instant you click **Commit changes** — the online repository is updated immediately, with no separate upload step to perform (see §5.7).
6. **Review the history.** Open the repository's **Commits** tab to see every checkpoint, in order, with its message, author, and timestamp (see §5.12).

The detail worth remembering: on the GitHub web interface, steps 3 through 5 collapse into a single click of **Commit changes**. Someone working from a local computer using Git's command line would perform staging, committing, and pushing as three visibly separate actions — a distinction that becomes relevant later in your career if you move to command-line Git, but is not required anywhere in this course.

## 7. Real-World Patterns

Every real engineering team — a small startup, a bank's software division, a hospital's patient-records team — relies on Git and GitHub, or an equivalent tool, for every change that reaches a live product. In most serious organizations, a change is also reviewed by a teammate before it becomes part of the main project, through a mechanism called a **pull request**; a pull request is named here only so you recognize the term — how to create and use one is a professional-level skill covered later, not part of this topic.

This is also why a group project where one teammate's "fix" quietly breaks someone else's work is usually solvable in minutes on a real team: opening the Commits tab shows exactly which checkpoint introduced the problem, who made it, and what they wrote as the reason [1]. In regulated industries such as banking and healthcare, this permanent, attributable history is frequently not just convenient but a compliance requirement, since regulators expect a clear record of what changed in a production system and when.

The same reasoning is why GitHub Classroom (§5.11) distributes coursework the way it does: each student gets a real, private repository with a real commit history behind their work, rather than a single zip file with no story of how it was built. Recruiters and interviewers looking at a candidate's GitHub profile read commit history the same way — as evidence of genuine, dated, incremental work, not a finished result that appeared from nowhere.

Job descriptions across the software industry list "Git and GitHub" as a baseline expectation regardless of programming language or company size, and technical interviews at every experience level, including entry level, routinely include at least one question that assumes daily familiarity with both [1]. Two questions come up often enough to be worth anticipating directly: "what is the difference between Git and GitHub?" (answered in §5.10) and "describe your commit habits" — for which a strong, concrete answer sounds like "I commit each time a small, working piece of a task is done, and I write a message describing what changed and why."

## 8. Best Practices

- **Commit often, in small logical chunks.** A commit that changes one thing is far easier to review, understand, and undo later than one giant commit bundling ten unrelated edits together.
- **Write commit messages for your future self.** Assume the person reading it later — often you, weeks afterward — has no memory of writing the code and only has the message to go on.
- **Use the imperative mood.** Write "Fix rounding error in average calculation," not "Fixed" or "Fixing" — this matches the convention used across the professional Git community.
- **Keep a clear README.** Even a few lines explaining what a project does makes a repository usable by someone who lands on it with zero context (§5.13).
- **Never commit secrets.** Passwords, API keys, and similar values must stay out of every commit — once something is committed, it exists permanently in the history described in §5.12, even if a later commit appears to remove it.

Common anti-patterns to avoid: vague commit messages such as `"update"` or `"fix"`; going long stretches of work without committing anything, which risks losing hours of progress to a single mistake; and accepting GitHub's pre-filled default commit message instead of writing your own explanation.

## 9. Hands-On Exercise

Open a repository containing one script you have already written, or create a new one and add a short Python file to it through the web editor. Make one small, real improvement to the file (for example, a clearer variable name, an added comment, or a small bug fix). Commit that single change with a message that states what changed and why, following the imperative-mood style from §8, rather than accepting GitHub's default pre-filled message. Then make one more small, separate improvement and commit it on its own, with its own clear message, instead of bundling it into the first commit. Finally, open the **Commits** tab and confirm both checkpoints appear in order, each with its own distinct, readable message.

## 10. Key Takeaways

- Version control keeps a permanent, ordered history of saved changes so that an earlier version of a file is never silently lost, no matter how many times it is edited afterward.
- Git is the version control system that keeps this history; GitHub is a separate website that hosts and shares that history online — the two are not the same thing.
- The conceptual loop is add, commit, push; on GitHub's web interface these collapse into a single "Commit changes" click.
- A commit message must state what changed and why, in the imperative mood — vague messages like "update" are the most common beginner mistake in this area.
- GitHub Classroom gives each student their own private repository for coursework, and never commit secrets, since a repository's history is permanent.

## 11. Next Steps

_System-derived from the next entry in curriculum/sequence.md._
