# Version Control Basics

Unit 5.3 closed out Module V with a file reader robust enough to survive a missing file, a malformed line, and a dozen other things that could go wrong — real, working software, built entirely by you. Every program across all five modules has lived the same way, though: as a file, or a handful of files, sitting on your own machine or inside your own Colab session. Nobody else has ever seen how any of it came together. If you had rewritten a working function into a broken one last week, there would be no way back — the good version simply isn't there anymore, overwritten the moment you saved. And if a teammate ever asked "wait, who changed this, and why?", the honest answer would be "I have no idea."

This unit, opening Module VI, is different in kind from everything before it. There is no new Python syntax here at all — not one keyword, not one function. What you're about to learn instead is a professional habit that every single software team on earth relies on, from a five-person startup to a bank's core engineering division: **version control**. You'll meet Git, the tool that keeps a permanent history of your work, and GitHub, the website that hosts that history online so it survives past your laptop and can be shown to anyone. And because this entire module is taught through the GitHub website rather than any command line, everything you learn here you'll practise with nothing more than a browser and a few careful clicks.

By the end, you'll understand exactly what a repository, a commit, and a commit message are, how GitHub Classroom hands you your assignments in this course, and the habits that separate a messy history from one a recruiter would actually want to read.

---

## The problem hiding in plain sight

Think about how you've probably already been "saving versions" of your own work, without ever calling it that. You finish a script, and before making a risky change you copy the file and rename it — `average_calculator_v2.py`. A week later you tweak it again — `average_calculator_v2_final.py`. Then you fix one more bug the night before a deadline — `average_calculator_v2_final_REALLY_final.py`. If this already sounds painfully familiar, that's because nearly everyone who has ever written code has done exactly this at least once.

This approach quietly fails in several ways at once:

- You end up with a folder full of near-duplicate files, and no clean way to tell, months later, what actually changed between `v2` and `v2_final`.
- If you delete or overwrite a file by mistake, whatever you hadn't separately copied is simply gone — there is no safety net underneath a normal save.
- On a real team, two people editing "the same" file under different names have no way of knowing whose copy is the current one, or of combining both sets of changes without a lot of manual, error-prone work.

**A file that gets silently overwritten every time you save it offers no way back to what it looked like a moment ago — and on a real project, that isn't a rare accident, it's a certainty waiting to happen.** Picture a five-person startup in Bengaluru shipping a payments feature, or a much larger team at an IT services company like TCS or Infosys maintaining a banking system used by millions — in both cases, dozens of people touch the same files, every single day, for years. Naming conventions and manual copies cannot survive that scale. Something has to actually keep the history for you.

Before reading on, think of one piece of work you've personally lost or nearly lost — an assignment, an essay, a photo edit — because you saved over the only copy you had. That's precisely the risk version control exists to remove.

## Version control: history that never gets thrown away

**Version control** is the general name for tools built specifically to solve this problem. Instead of a file that only ever shows you its current state, a version control system keeps every snapshot you deliberately ask it to remember, in order, permanently. You can always go back and see exactly what a file looked like at any earlier saved point, and set that against what it looks like today.

The analogy that makes this click fastest is a video game's save-point system. In a game with proper save points, you don't lose your progress just because you walked into a dangerous room — you can always reload an earlier save and try again from there, with the earlier save still sitting there untouched. Version control gives your code the exact same safety net: no matter how many times you've changed a file since, every earlier "save point" is still sitting in the project's history, waiting to be looked at, compared, or restored.

**Git** is the specific version control tool the software industry has standardised on, almost without exception. Git does not stop you from editing or deleting anything in your working files — you're free to change whatever you like, exactly as before. What it adds is the ability to permanently label a snapshot of your entire project the moment you ask it to, so that even after a hundred more changes, that earlier snapshot is never actually lost. Git can, in principle, run entirely on one computer with no internet connection at all; the history lives wherever the project itself lives.

## The basic loop: repository, commit, push

Three ideas carry almost all of the conceptual weight in this unit, and they build on each other in one continuous loop.

A **repository** — nearly everyone shortens it to "repo" — is a project's folder together with its entire saved history. It's the one unit Git actually operates on: a single repository holds both your current files and the complete record of every past checkpoint for those files. Every course project, and every real software project you'll ever work on, lives inside exactly one repository.

A **commit** is one saved checkpoint — a snapshot of your project's files at one specific moment, permanently labelled with a short message explaining what changed and why. Once you commit, that snapshot exists in the repository's history forever. Committing again never erases or edits an earlier commit — it only adds a new one on top, the same way a new save point in a game never deletes the one before it. This is precisely the mechanism that solves the disappearing-work problem from the opening section: an earlier version is always still there, sitting in the history, however many more times the file changes afterwards.

**Push** is the act of sending your committed checkpoints up to GitHub, so the copy of your repository living online matches your latest committed work. Think of push as the moment your local save points get backed up somewhere that survives even if your own machine doesn't.

Put the three together and you get the loop every Git user repeats, over and over, for the life of a project: make a change, **commit** it as a labelled snapshot, then **push** it so the online copy is up to date. On the GitHub website specifically — which is exactly how this course works — you'll see in a moment that committing and pushing collapse into a single click, which makes the loop even easier to hold in your head.

There's a fourth motion worth knowing by name, even though you'll mostly experience it passively in this course: **pull** (sometimes called fetch) is the reverse of push — bringing changes made somewhere else, by you on another device or by a teammate, down into the copy of the project you're currently looking at. Because this module works entirely through the GitHub website, every repository page you open is already showing you the current, most up-to-date state, so pulling happens for you automatically rather than as a deliberate step you trigger.

## What a commit message actually buys you

A commit without a message would just be an anonymous snapshot — technically useful, practically useless, because nobody (including future you) would know why it exists. The **commit message** is the short piece of text attached to every commit, stating what changed and why.

Compare these two commit messages for the exact same underlying change:

| Commit message | What a future reader learns |
|---|---|
| `update` | Nothing. Something changed, at some point, for some reason. |
| `fix` | Slightly more, but still no idea what was broken or how. |
| `Fix rounding error in average marks calculation` | Exactly what was wrong, and exactly what this checkpoint addresses — without opening a single file. |

**A commit message is written for a reader who has no memory of writing the code — often that reader is you, three weeks later, having completely forgotten the details.** The convention professional teams use, and the one you should build into muscle memory now, is the imperative mood: "Fix rounding error," not "Fixed rounding error" or "Fixing rounding error." It reads like an instruction the commit itself is carrying out, and it's the style you'll see across virtually every serious open-source project on GitHub.

Try this the next time you make any change, however small: before you write the commit message, finish the sentence "This commit will..." out loud — whatever comes after "will" is very close to the message you should actually type.

## GitHub: where your repository lives online

**GitHub** is a hosting platform for projects that use Git — a website, github.com, that stores a copy of your Git history online. Because that copy lives on GitHub's servers rather than only on your own laptop, it survives even if your device is lost, damaged, or simply replaced next year, and it can be seen by anyone you grant access to — an instructor grading your work, a teammate on a group project, or, later in your career, a recruiter deciding whether to call you in for an interview.

The relationship between the two tools is worth pinning down precisely, because mixing them up is one of the most common beginner mistakes in this whole area — and one that shows up as a genuine red flag in technical interviews.

| | Git | GitHub |
|---|---|---|
| What it is | The version-control tool itself | A website that hosts Git repositories online |
| Where it runs | On whichever machine holds the project | On GitHub's own servers, accessed through a browser |
| Needs the other to exist? | No — Git works completely on its own | Yes — GitHub has nothing to host without Git underneath it |
| What it gives you | The commit history, the save-point mechanism | A shared, backed-up, visible home for that history |

**Git is the underlying save-checkpoint system; GitHub is a website that hosts and shares those checkpoints online — treating the two names as interchangeable is a mix-up worth correcting for yourself right now, before it becomes a habit.**

## GitHub Classroom: how your assignments actually arrive

**GitHub Classroom** is a tool built on top of GitHub that instructors use to distribute and collect coursework, and it is exactly how every assignment in this programme reaches you. When you accept a GitHub Classroom assignment link, GitHub automatically creates a brand-new, private repository that belongs to you alone — visible only to you and your instructor by default, nobody else.

This matters for two reasons. First, you are never editing a shared, public copy that other students could also be touching — your repository is genuinely yours. Second, because it is a real repository with a real commit history behind it, the finished assignment you submit is never just a final file that appeared from nowhere — it carries a dated, attributable record of how you actually built it, which is precisely what a recruiter or interviewer looking at your GitHub profile later will be reading.

## Working entirely through the browser

Because this module deliberately avoids the command line, the "procedure" for every action below is a sequence of clicks rather than a line you type. Once you've done this once, it becomes as automatic as opening a Colab notebook was after Unit 1.1.

**Creating a repository:**

1. Sign in to `github.com` with your account.
2. Click the **+** icon near the top-right corner, then **New repository**.
3. Give it a short, meaningful name — `average-calculator`, not `project1`.
4. Choose **Public** or **Private** visibility, optionally tick **Add a README file**, and click **Create repository**.

**Editing or uploading a file:**

1. Open the repository and click **Add file**, choosing either **Create new file** or **Upload files**.
2. If creating a new file, type its name (including the `.py` extension) and write its contents directly in the browser's editor.
3. If a file already exists and you want to change it, open it and click the pencil (edit) icon instead.

**Committing your change:**

1. Scroll down to the box labelled **Commit changes** at the bottom of the page.
2. Replace GitHub's default pre-filled text with your own clear, imperative-mood message describing what changed and why.
3. Confirm **Commit directly to the `main` branch** is selected — `main` is every repository's default branch, its stable, always-working line of work — and click **Commit changes**.

That single click is doing more work than it looks like. On a local computer using Git's command line, staging, committing, and pushing would be three separate, visible actions. On the GitHub website, all three collapse into this one click: the moment you press **Commit changes**, your change is saved as a labelled snapshot *and* sent up to GitHub's servers, with nothing left for you to do afterwards.

**Viewing your history:**

1. Open your repository and click the **Commits** tab (sometimes shown as a small clock icon with a number beside it).
2. Every commit you've ever made appears here, in order, each with its message, its author, and exactly when it was made.

This is where the entire promise of version control becomes visible and concrete: click into any past commit and you can see precisely what the project looked like at that exact checkpoint, long before whatever changed afterwards.

Do this once for yourself before continuing: open GitHub, create a small test repository, add one short file with a sentence or two inside it, and commit it with a real, descriptive message — then open the Commits tab and confirm your one checkpoint is sitting there, labelled exactly the way you wrote it.

## Good habits: small commits, honest messages

The single biggest difference between a repository that's pleasant to read later and one that isn't has nothing to do with the code itself — it's how the commits were made along the way.

| One giant commit | Several small commits |
|---|---|
| Bundles a week's worth of unrelated changes into one snapshot | Each commit represents one logical, self-contained change |
| A bug introduced anywhere in that week is hard to trace to a specific line | Opening the Commits tab shows exactly which checkpoint introduced a problem |
| The commit message, if there even is a good one, has to summarise ten different things at once | Each message can be short, specific, and genuinely useful |
| Undoing one bad change usually means undoing everything alongside it | A single bad change can be reviewed or reverted on its own |

This is also exactly why a group project where one teammate's "fix" quietly breaks somebody else's work is usually solvable in minutes on a real team: opening the Commits tab reveals precisely which checkpoint introduced the problem, who made it, and what they wrote as the reason for it. That visibility only works, though, if commits were actually kept small and honestly labelled in the first place.

A short list of habits worth avoiding deliberately while this is still new:

- Writing vague, one-word commit messages such as `"update"` or `"fix"` that tell a future reader nothing.
- Going long stretches of real work without committing anything at all, risking hours of progress to a single mistake.
- Accepting GitHub's pre-filled default commit message instead of writing your own honest description.
- Bundling several unrelated changes into one commit because it feels faster than committing each one separately.
- Committing secrets — passwords, API keys, tokens — directly into a file; once committed, they exist permanently in the history, even if a later commit appears to delete them.

## Try it yourself

Before moving on, create one small repository of your own through the GitHub website, exactly as described above. Add a short Python file — even a single `print()` line is enough — and commit it with a clear, imperative-mood message stating what the file does. Then make one more small, separate change to the same file — add a comment, or improve a variable name — and commit that on its own, with its own distinct message, rather than folding it into the first commit. Finally, open the **Commits** tab and confirm both checkpoints appear in order, each individually labelled and individually readable.

---

### Key Terminology

- **Version control** — a category of tool that keeps a permanent, ordered history of saved changes, instead of only ever showing a file's current state.
- **Git** — the version control tool the software industry has standardised on; keeps a labelled history of a project's snapshots.
- **Repository (repo)** — a project's folder together with its entire saved history, as tracked by Git.
- **Commit** — one saved, permanently labelled checkpoint of a project's files at a specific moment.
- **Commit message** — the short piece of text attached to a commit, describing what changed and why.
- **Push** — sending committed checkpoints up to GitHub, so the online copy matches your latest work.
- **Pull (fetch)** — bringing changes made elsewhere down into the copy of the project you're viewing.
- **Branch** — a separate line of commits; `main` is every repository's default, stable branch.
- **GitHub** — a website that hosts and shares Git repositories online.
- **GitHub Classroom** — a tool built on GitHub that distributes and collects coursework as private, individual repositories.
- **Commit history** — the complete, ordered list of every commit ever made to a repository, viewable on GitHub's Commits tab.
- **README** — a file explaining what a repository contains and how to use it, usually the first thing a visitor sees.

### Mastery Checkpoint

Before moving to Unit 6.2, check that you can answer these without looking back:

1. Why is renaming files `v2`, `v2_final`, `v2_final_REALLY_final` a fundamentally weaker safety net than real version control, even though both are attempts at the same goal?
2. What is the precise relationship between Git and GitHub — why does calling them interchangeable count as a mix-up?
3. What three actions make up the basic commit loop, and which single click on the GitHub website performs all three at once?
4. Why does a commit message like `"Fix rounding error in average marks calculation"` do more useful work than `"update"`, even though both attach to an identical code change?
5. How does GitHub Classroom's private, individual repository model change what actually gets submitted, compared with handing in a single finished file?

### Summary

You now know why every file you've ever saved has been one accidental overwrite away from losing real work, and how version control removes that risk by keeping a permanent, ordered history of labelled snapshots instead. You've learned what Git actually is, how a repository, a commit, and a commit message fit together into the basic commit-push-pull loop, and precisely how Git and GitHub differ despite being constantly confused for one another. You've walked through creating a repository, editing and committing a file, and reading a commit history entirely through the GitHub website, and picked up the habit — small, honestly labelled commits over one giant unexplained change — that separates a professional history from a messy one. From here, the next step is turning this habit into something visible: Unit 6.2, Portfolio & Diagnostic, where you'll start shaping your own GitHub presence into a record of real, demonstrable work.

### Additional Resources

- [W3Schools — Git Introduction](https://www.w3schools.com/git/git_intro.asp)
- [W3Schools — Git Getting Started](https://www.w3schools.com/git/git_getstarted.asp)
- [W3Schools — Git Repository](https://www.w3schools.com/git/git_repo.asp)
- [W3Schools — Git Commit](https://www.w3schools.com/git/git_commit.asp)
- [W3Schools — Git GitHub](https://www.w3schools.com/git/git_github.asp)
- [W3Schools — Git Pull](https://www.w3schools.com/git/git_pull.asp)
