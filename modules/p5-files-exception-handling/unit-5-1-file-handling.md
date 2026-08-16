# File Handling

## File Persistence

Variables, lists, and objects live in memory while a program is running. When the program stops, that data is lost, so nothing a program stores survives being closed and reopened.

A file fixes that. A file is data stored on disk, and the disk keeps it after the program ends. A shopping app saves your orders this way, a game saves your scores, and a notes app saves your notes. Data that survives is called **persistent data**, and working with files to store and retrieve it is called **file handling**.

## File Paths

A **path** is the text that names the file you want. It is the first argument you pass when opening a file, and there are two kinds.

### Relative Paths

A relative path is resolved from the **current working directory**. It is written without any drive or root in front of it.

The current working directory is often the project folder when you run a program from there, but it can be different from the folder containing the Python script.

- `"posts.txt"` is a file in the current working directory.
- `"data/posts.txt"` is a file inside the `data` folder.
- `"../posts.txt"` is a file one folder above the current working directory.

### Absolute Paths

An absolute path gives the complete location, starting from the top of the drive.

- Relative: `"data/posts.txt"`
- Absolute: `"C:/Users/Rahul/project/data/posts.txt"`

An absolute path leaves no doubt about which file is meant, but it stops working the moment the project moves. Prefer relative paths in projects for that reason, and keep absolute paths for one-off scripts on your own machine.

## File Modes

Knowing which file you want is not enough. Python also needs to know what you intend to do with it, and that is what the **mode** says. It is a short string passed alongside the path.

| Mode | Use | Existing file | Missing file |
|---|---|---|---|
| `r` | Read | Opens it | Raises an error |
| `w` | Write, replacing everything | Erases the contents | Creates it |
| `a` | Add at the end | Keeps the contents | Creates it |
| `x` | Create only | Raises an error | Creates it |

The most important distinction is between `w` and `a`. Choosing `w` when you meant `a` destroys data silently, because the erase happens before a single character is written.

Two letters combine with these four. Adding `+` allows reading and writing through one file object, as in `r+`. Adding `b` opens the file in binary mode, as in `rb`, which is what images, audio and PDFs need. Plain text files need neither.

## Opening Files

With a path and a mode chosen, the file can be opened. Python provides one built-in function for it.

### open() and the File Object

`open()` takes the path and the mode, and hands back a **file object**. That object is not the file's text. It is the interface your program uses to reach the file, and every read or write goes through it.

```text
open()
   ↓
file object
   ↓
read() / write()
   ↓
close()
```

Close the file when you are finished with it.

```python
file = open("posts.txt", "w")
file.write("First day at college!\n")
file.close()

print("Saved")
```

**Output**

```text
Saved
```

The first line opens `posts.txt` for writing and stores the file object in `file`. The second writes a line through it. The third closes it, and only then is the work finished.

### with open()

The program above works, but it depends on you remembering `close()`. Forget it, or crash before reaching it, and the file is left open. Python has a form that removes both risks.

```python
with open("posts.txt", "w") as file:
    file.write("First day at college!\n")

print("Saved")
```

**Output**

```text
Saved
```

Read the first line as *open this file, call it `file`, and close it when this block ends*. Python closes it for you, so there is no `close()` to write and none to forget. It closes the file even if an exception is raised inside the block, which a hand-written `close()` cannot promise.

An object that `with` cleans up after is called a **context manager**, and `open()` returns one. This is the preferred form, and every example from here on uses it.

## Writing and Appending

Two methods put text into a file, and the mode chosen at `open()` decides whether it replaces or adds.

### write()

`write()` sends one string to the file.

```python
with open("posts.txt", "w") as file:
    file.write("First day at college!\n")
    file.write("Exam over!\n")

print("Saved 2 posts")
```

**Output**

```text
Saved 2 posts
```

Open `posts.txt` in any text editor and both posts are there, one on each line. That is because each string ends with `\n`. `write()` adds no line break of its own, so without it both posts would run together on a single line.

### writelines()

When the lines are already in a list, one call writes all of them.

```python
lines = ["Python\n", "Java\n", "SQL\n"]

with open("languages.txt", "w") as file:
    file.writelines(lines)

print("Saved 3 languages")
```

**Output**

```text
Saved 3 languages
```

`writelines()` does not add `\n`; include the newline in each string if you want separate lines.

### w versus a

The modes table said `w` erases and `a` adds. Here is that difference in a program.

```python
with open("posts.txt", "w") as file:
    file.write("First day at college!\n")

with open("posts.txt", "a") as file:
    file.write("Exam over!\n")

print("Wrote one post, then added another")
```

**Output**

```text
Wrote one post, then added another
```

Open the file and both posts are there. Now change that second `a` to `w` and run it again. Only `Exam over!` remains, because the second open erased the first post before writing.

## Reading Data

Mode `r` opens a file for reading. Four methods take the text back out, and they differ in how much they return at a time.

The examples below read `posts.txt`, which contains:

```text
First day at college!
Exam over!
```

| Method | Returns |
|---|---|
| `read()` | Remaining content as one string |
| `read(n)` | Next `n` characters |
| `readline()` | Next line |
| `readlines()` | Remaining lines as a list |

### read()

The simplest of them returns the whole file as one string.

```python
with open("posts.txt", "r") as file:
    content = file.read()

print(content, end="")
```

**Output**

```text
First day at college!
Exam over!
```

`content` is a single string holding both lines, `\n` characters included. `end=""` stops `print()` from adding another line break, since the text already ends with one.

### The File Position

Look again at the table above. Every row says **remaining** or **next**, never *all* or *first*. The reason is that Python keeps a position inside an open file and moves it forward as you read.

```text
First day at college!
     ↑
current position after read(5)
```

```python
with open("posts.txt", "r") as file:
    print(file.read(5))
    print(file.readline(), end="")
    print(file.readlines())
```

**Output**

```text
First
 day at college!
['Exam over!\n']
```

`read(5)` returned `First` and left the position five characters in. `readline()` carried on from exactly there, which is why it began with a space instead of with `First`. `readlines()` then collected everything still unread. No call started over at the beginning.

### Reading Line by Line

`read()` pulls the entire file into memory at once. For a handful of posts that is fine, and for a file holding a lakh of them it is wasteful. Looping over the file object avoids it.

```python
with open("posts.txt", "r") as file:
    for line in file:
        print(line.strip())
```

**Output**

```text
First day at college!
Exam over!
```

Each turn of the loop hands back the next line of the file as a string, which is what `line` holds. This makes it suitable for processing large files without loading the entire file into memory at once.

`strip()` removes the `\n` that each line still carries, along with any spaces at either end. Without it, every printed line would be followed by a blank one.

## Error Handling

Opening a file that is not there, in mode `r`, stops the program.

```python
with open("followers.txt", "r") as file:
    content = file.read()
```

**Output**

```text
Traceback (most recent call last):
  File "app.py", line 1, in <module>
    with open("followers.txt", "r") as file:
         ^^^^^^^^^^^^^^^^^^^^^^^^^^
FileNotFoundError: [Errno 2] No such file or directory: 'followers.txt'
```

`FileNotFoundError` is the error you will meet most often while learning files. The file name it prints is the first thing to check, either for a spelling mistake or for a path pointing at the wrong folder.

Notice what the error did to the program. It did not warn and carry on. It stopped, and nothing written after that line ever ran.

That is not always what you want. A program can catch an error like this instead and decide what to do next, such as showing a message, asking for another file name, or creating the file. Handling errors that way is the subject of the next chapter, **Exception Handling**.

## Practice

1. **Write and read.** Create `notes.txt`, write two lines into it, then open it in read mode and print the whole file with `read()`.

2. **Append and loop.** Add a third line using mode `a`, then print every line using a `for` loop with `strip()`. Confirm the first two lines are still there.

3. **A small notes program.** Write a program that shows a menu with three choices: add a note, view all notes, or exit. Adding a note appends one line to `notes.txt`, and viewing reads the file line by line. Run the program twice and confirm the notes from the first run are still there.

## Reference Links

- [Python Official Docs — Reading and Writing Files](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)
- [Python Official Docs — `open()`](https://docs.python.org/3/library/functions.html#open)
- [W3Schools — Python File Handling](https://www.w3schools.com/python/python_file_handling.asp)
- [GeeksforGeeks — File Handling in Python](https://www.geeksforgeeks.org/python/file-handling-python/)
