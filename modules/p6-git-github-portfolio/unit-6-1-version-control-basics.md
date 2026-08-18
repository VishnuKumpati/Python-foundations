# Version Control: Working on the Same Code Together

## Sharing Code and Working Together

Every program so far lived in one file on one computer. That works while you are alone, and breaks the moment someone else joins.

You and a friend both edit the marks program on the same day. Now there are two versions of one file, and copying either over the other loses somebody's work. You also cannot say what changed since yesterday, or who wrote a line and why.

**Version control** solves this. It is a system that records every change to your files, keeps the full history, and merges work from several people into one copy.

## Git and GitHub

These two names are used together so often that beginners think they are one thing. They are not.

**Git** is the version control program. It runs on your computer, records your changes, and keeps the history in a hidden folder named `.git` inside your project. Git works perfectly well with no internet connection.

**GitHub** is a website that stores a copy of a Git project online. It gives your team one shared address to send work to and take work from. It also adds things Git does not have, such as issues, pull requests and a web page for your project.

| | Git | GitHub |
|---|---|---|
| What it is | A program | A website |
| Where it runs | On your computer | On the internet |
| Needs the other one | No | Yes, it stores Git projects |
| Used for | Recording your own changes | Sharing changes with a team |

A folder that Git is tracking is called a **repository**, or repo for short. The copy on your computer is the **local repository**, and the copy on GitHub is the **remote repository**.

## Starting a Repository

### git init

`git init` turns the current folder into a Git repository. Run it once, inside your project folder.

```text
$ git init
Initialized empty Git repository in /home/rahul/marks/.git/
```

Git created the hidden `.git` folder. Your files are untouched, and Git is now watching the folder for changes.

### git status

`git status` tells you what Git can see. Run it whenever you are unsure what state your work is in.

```text
$ git status
On branch main

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
    average.py

nothing added to commit but untracked files present (use "git add" to track)
```

`average.py` is **untracked**, which means Git has noticed the file but is not recording it yet. Every new file starts this way.

## Saving Changes

Saving work in Git takes two steps. First you choose which files to save, then you save them with a message. Git separates the two so that you can record part of your work without recording all of it.

### git add

`git add` marks a file to be included in the next save. The marked files are said to be **staged**.

```text
$ git add average.py
```

`git add` prints nothing when it works. Run `git status` to confirm.

```text
$ git status
On branch main

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
    new file:   average.py
```

To stage every changed file at once, use a dot instead of a file name.

```text
$ git add .
```

Use `git add .` when everything you changed belongs together. Name the files individually when it does not.

### git commit

`git commit` saves the staged files into the history as one entry, called a **commit**. The `-m` option carries the message that explains what changed.

```text
$ git commit -m "Add average marks program"
[main (root-commit) 0f28341] Add average marks program
 1 file changed, 4 insertions(+)
 create mode 100644 average.py
```

`0f28341` is the commit's id. Git gives every commit a unique id, so yours will be different.

Write the message as what the change does, not what you did. "Add average marks program" is useful six months later. "update" and "changes" are not.

## Sending Work to GitHub

### git remote add

A new repository knows nothing about GitHub. `git remote add` tells it the address of the remote copy. The name `origin` is the usual name for the main remote.

```text
$ git remote add origin https://github.com/rahul/marks.git
```

### git push

`git push` sends your commits to the remote repository. The first push needs `-u origin main`, which sends the branch and remembers the connection.

```text
$ git push -u origin main
To https://github.com/rahul/marks.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

After that first time, `git push` on its own is enough. Your commits are now on GitHub, where your friend can see them.

## Getting Others' Work

Your friend has been committing and pushing too. Their commits are on GitHub, but not yet on your computer. Two commands bring them over, and the difference between them matters.

### git fetch

`git fetch` downloads the new commits but does not change any of your files. It lets you see what has arrived before you accept it.

```text
$ git fetch
From https://github.com/rahul/marks
   f4886f7..f2a3ae8  main       -> origin/main
```

`git status` now shows that you are behind.

```text
$ git status
On branch main
Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean
```

### git pull

`git pull` downloads the new commits and applies them to your files in one step. It is `git fetch` followed by a merge.

```text
$ git pull
Updating f4886f7..f2a3ae8
Fast-forward
 average.py | 1 +
 1 file changed, 1 insertion(+)
```

Your copy of `average.py` now contains your friend's line as well as your own. Pull before you start work each day, so you are building on the latest version.

## Ignoring Files

Some files should never go into a repository. Python creates a `__pycache__` folder, editors leave behind their own files, and log files change on every run. None of them belong in the history.

A file named `.gitignore` lists the things Git should skip. Create it in the project folder.

```text
__pycache__/
*.log
.env
```

Each line is a pattern. `__pycache__/` ignores the whole folder, `*.log` ignores every file ending in `.log`, and `.env` ignores that one file. Once the file exists, `git status` stops showing them.

Commit `.gitignore` itself, so everyone on the team ignores the same things.

## Branches

A **branch** is a separate line of work inside the same repository. It lets you build something new without touching the working version, and Git keeps both until you decide to combine them.

Every repository starts with one branch, normally called `main`.

### git branch

`git branch` with a name creates a branch. With no name, it lists the branches and marks the one you are on with `*`.

```text
$ git branch feature-average
$ git branch
  feature-average
* main
```

### git switch

`git switch` moves you onto another branch. Your files change to match that branch, and commits you make now belong to it alone.

```text
$ git switch feature-average
Switched to branch 'feature-average'
```

Work here as normal, using `git add` and `git commit`. Nothing you do touches `main`.

### git merge

`git merge` brings another branch's commits into the branch you are on. Switch to the branch that should receive the work first, then merge.

```text
$ git switch main
$ git merge feature-average
Updating 02feee3..1073fa5
Fast-forward
 average.py | 2 ++
 1 file changed, 2 insertions(+)
```

`main` now contains the work from `feature-average`. If both branches changed the same lines of the same file, Git cannot decide which to keep. It reports a **merge conflict**, which you resolve by editing the file yourself.

## Practice

1. **Make your first commit.** Create a folder, put one Python file in it, and run `git init`. Check `git status`, then stage the file with `git add` and save it with `git commit -m`. Run `git status` again and note that it is now clean.

2. **Send it to GitHub.** Create an empty repository on GitHub and connect it with `git remote add origin`. Push your commit with `git push -u origin main`, then refresh the GitHub page and confirm your file is there.

3. **Work on a branch.** Create a branch with `git branch` and switch to it. Add a new function to your file and commit it. Switch back to `main`, confirm the new function is not there, then merge the branch and confirm it is.

## Resources

- [Git Official Documentation](https://git-scm.com/doc)
- [GitHub Docs — Get started](https://docs.github.com/en/get-started)
- [W3Schools — Git Tutorial](https://www.w3schools.com/git/)
