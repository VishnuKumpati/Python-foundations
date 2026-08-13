# File Handling: Making Data Outlive the Program

## File Persistence

Everything a program has stored so far lived in memory: a list of posts, a follower count, an object built from a class. All of it existed only while the program was running. When the program ends, the operating system takes that memory back. Run the program again and it begins with nothing.

Think about the social media app. A user writes a post and the program keeps it in a list. Close the program, start it again, and the list is empty. The post is gone, and nothing the user typed survived.

Data that outlives the program is called **persistent** data. It has to be kept somewhere other than memory, and the disk is that somewhere. A **file** is a named block of data stored on the disk. Anything written into a file today can still be read tomorrow, next week, or on another computer.

## File Paths

Before a program can open a file it has to know which file. The text that names the file is called a **path**, and there are two kinds.

A **relative path** is measured from the folder the program is running in.

- `"posts.txt"` is a file sitting beside the script.
- `"data/posts.txt"` is a file inside a `data` folder beside the script.
- `"../posts.txt"` is a file one folder above the script.

An **absolute path** starts at the top of the drive and names every folder down to the file.

- Windows: `"C:/Users/Rahul/posts.txt"`
- Linux and macOS: `"/home/rahul/posts.txt"`

Prefer relative paths. An absolute path stops working as soon as the project moves to another computer. The two systems do not even write absolute paths the same way, so the same line cannot serve both.

One trap on Windows catches almost everyone. Windows shows its paths with backslashes, and inside a Python string a backslash begins an escape sequence. So `"data\notes.txt"` contains `\n`, which is a newline and not a folder separator. Write forward slashes instead, because Python accepts them on Windows too.

When a path is built from pieces, the `pathlib` module joins them for you. It also picks the correct separator for whichever system the program runs on.

```python
from pathlib import Path

path = Path("data") / "posts.txt"

print(path)
print(path.name)
print(path.parent)
```

**Output**

```text
data/posts.txt
posts.txt
data
```

The `/` between the two pieces is doing the joining. That output is from Linux, and on Windows the first line prints `data\posts.txt` instead. Nothing else in the program changes, which is the whole reason to use `pathlib` rather than gluing strings together. A `Path` can be handed to `open()` anywhere a plain string would go.

## Opening Files

`open()` is the built-in function that gives a program access to a file. It takes a path and a mode, and the mode states what you intend to do with the file.

```python
file = open("posts.txt", "w")

print(type(file))

file.close()
```

**Output**

```text
<class '_io.TextIOWrapper'>
```

`open()` does not hand back the file's text. It hands back a **file object**, and every later operation goes through that object. Printing its type shows what it is: an object built for reading and writing text.

Working with a file always follows the same three steps.

1. **Open** the file, stating what you intend to do with it.
2. **Read from it or write to it**, through the file object.
3. **Close** it, so the operating system knows you have finished.

Skipping step three is the mistake beginners make most often. A later section replaces `close()` with something that cannot be forgotten.

## File Modes

The mode is a short string, and it is the most important argument you pass to `open()`.

| Mode | Meaning | If the file exists | If it does not exist |
|---|---|---|---|
| `"r"` | Read only | Opens it | Raises an error |
| `"w"` | Write | **Erases the contents** | Creates it |
| `"a"` | Append | Writes at the end | Creates it |
| `"x"` | Create only | Raises an error | Creates it |

Two letters combine with these. Adding `"+"` allows reading and writing through one file object, as in `"r+"`. Adding `"b"` opens the file in binary mode, as in `"rb"`, which is what images, audio and PDFs need. Plain text files need neither.

Mode `"r"` is the default, so `open("posts.txt")` and `open("posts.txt", "r")` mean the same thing. Writing the mode out is clearer for anyone reading the code later.

## Writing and Appending

Mode `"w"` opens a file for writing, and `write()` puts a string into it.

```python
file = open("posts.txt", "w")
file.write("First day at college!\n")
file.write("Exam over!\n")
file.close()

print("Saved 2 posts")
```

**Output**

```text
Saved 2 posts
```

The program creates `posts.txt` beside the script, and the file stays there after the program ends. `write()` adds no line break of its own. The `\n` at the end of each string is what puts the next post on its own line.

A whole list of strings can go in with a single call to `writelines()`.

```python
posts = ["First day at college!\n", "Exam over!\n", "Fest tomorrow!\n"]

file = open("posts.txt", "w")
file.writelines(posts)
file.close()

file = open("posts.txt", "r")
print(file.read(), end="")
file.close()
```

**Output**

```text
First day at college!
Exam over!
Fest tomorrow!
```

`writelines()` writes every string in the list, one after another. It adds no line breaks either, which is why each string in the list still ends with `\n`.

Now the difference that matters most in this whole chapter. **Mode `"w"` erases everything already in the file.** Open an existing `posts.txt` in `"w"` mode and the old contents are gone. That happens before you write a single character. Mode `"a"` is the safe alternative, because it writes at the end and leaves the existing contents alone.

```python
file = open("posts.txt", "w")
file.write("First day at college!\n")
file.close()

file = open("posts.txt", "a")
file.write("Exam over!\n")
file.close()

file = open("posts.txt", "r")
print(file.read(), end="")
file.close()
```

**Output**

```text
First day at college!
Exam over!
```

The first post went in with `"w"` and the second with `"a"`, and both survived. Change that second mode to `"w"` and the output becomes only `Exam over!`, because the first post would be erased. Mode `"a"` also creates the file when it is missing, so it is safe from the very first run.

## Reading Data

Mode `"r"` opens a file for reading, and `read()` returns the whole file as one string.

```python
file = open("posts.txt", "w")
file.write("First day at college!\n")
file.write("Exam over!\n")
file.close()

file = open("posts.txt", "r")
content = file.read()
file.close()

print(content, end="")
```

**Output**

```text
First day at college!
Exam over!
```

Each reading example writes the file first so that it runs on its own. In a real project the writing and the reading happen in different programs on different days. The variable `content` holds one string with both lines in it, including the `\n` characters. `end=""` stops `print()` from adding another line break, because the text already ends with one.

A file can also be read in pieces, and doing so reveals an important detail.

```python
file = open("posts.txt", "w")
file.write("First day at college!\n")
file.write("Exam over!\n")
file.close()

file = open("posts.txt", "r")
print(file.read(5))
print(file.readline(), end="")
print(file.readlines())
file.close()
```

**Output**

```text
First
 day at college!
['Exam over!\n']
```

Three methods did three different jobs.

- `read(5)` returned the first five characters, `First`.
- `readline()` returned the rest of that line, starting with the space after `First`.
- `readlines()` returned everything still unread, as a list of strings.

Notice that no method started at the beginning of the file. Python keeps a position inside an open file and moves it forward as you read. Each call carries on from where the previous one stopped.

`read()` loads the entire file into memory. That is fine for a few posts and wasteful for a file holding a lakh of them. Looping over the file object reads one line per turn instead.

```python
file = open("posts.txt", "w")
file.write("First day at college!\n")
file.write("Exam over!\n")
file.write("Fest tomorrow!\n")
file.close()

file = open("posts.txt", "r")
for line in file:
    print(line.strip())
file.close()
```

**Output**

```text
First day at college!
Exam over!
Fest tomorrow!
```

Only one line sits in memory at a time, so the size of the file stops mattering. Each line still carries the `\n` it was written with. `strip()` removes it, along with any spaces at either end. Without `strip()`, every printed line would be followed by a blank one.

## Context Managers

A **context manager** is an object that tidies up after itself once you have finished with it. An open file is one, and the `with` statement is how you use it.

Every program so far ended with `close()`, and forgetting that line is easy. Worse, a program that crashes between `open()` and `close()` never reaches the closing line, so the file is left open. The `with` statement removes both risks. Python closes the file when the indented block ends, whether the block finished normally or crashed.

```python
with open("posts.txt", "w") as file:
    file.write("First day at college!\n")

with open("posts.txt", "r") as file:
    print(file.read(), end="")
```

**Output**

```text
First day at college!
```

Read the first line as *open this file, call it `file`, and close it when this block ends*. There is no `close()` anywhere in the program, and none is needed. The file object works only inside the indented block, and outside it the file is already closed.

Use `with` for every file you open from now on. The earlier examples called `close()` by hand only so that you could see what `with` is doing for you.

## Error Handling

Opening a file that is not there, in mode `"r"`, stops the program.

```python
file = open("followers.txt", "r")
```

**Output**

```text
Traceback (most recent call last):
  File "app.py", line 1, in <module>
    file = open("followers.txt", "r")
           ^^^^^^^^^^^^^^^^^^^^^^^^^^
FileNotFoundError: [Errno 2] No such file or directory: 'followers.txt'
```

`FileNotFoundError` is the error you will meet most often while learning files. It means what it says. The file name it prints is the first place to look. Check it for a spelling mistake, or for a path pointing at the wrong folder.

The mode decides whether this can happen at all. Modes `"w"`, `"a"` and `"x"` create the file when it is missing. Only `"r"` and `"r+"` insist that the file already exists. A program that reads should either write the file first, or expect it to be absent.

Stopping the program is not the only option. Python can catch an error like this and carry on instead, which is what exception handling is for. That is a topic of its own, and it comes next.

## Practice

Build a small notes file for the social media app, one step at a time. Each task continues the file from the task before it.

1. **Write a file.** Use `open()` with mode `"w"` to create `notes.txt`, write two lines into it, and close it. Open the file in a text editor afterwards and confirm both lines are there.

2. **Write a list at once.** Replace the two `write()` calls with a list of three strings and a single `writelines()` call. Confirm all three lines appear.

3. **Read it back.** Open `notes.txt` in mode `"r"`, read it with `read()`, and print it using `end=""`. Add a comment explaining why `end=""` is needed.

4. **Prove that `"w"` erases.** Open `notes.txt` in mode `"w"` again and write one different line. Read the file and confirm the earlier lines are gone. Write a comment naming the mode you should have used.

5. **Append safely.** Add one more line using mode `"a"`, then read the file and confirm nothing was lost.

6. **Read in pieces.** In one open file, call `read(4)`, then `readline()`, then `readlines()`, printing each result. Explain in a comment why the second call did not start at the beginning.

7. **Read line by line.** Loop over the file object and print each line with `strip()`. Then print the same loop without `strip()` and describe the difference in a comment.

8. **Switch to `with`.** Rewrite every `open()` in your program as a `with` block and delete all the `close()` calls. Confirm the output is unchanged.

9. **Trigger the error.** Open a file name that does not exist in mode `"r"`. Record the error Python reports. Then change only the mode so the same program runs, and explain why that mode works.


## Reference Links

- [Python Official Docs — Reading and Writing Files](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files)
- [Python Official Docs — `open()`](https://docs.python.org/3/library/functions.html#open)
- [Python Official Docs — `pathlib`](https://docs.python.org/3/library/pathlib.html)
- [W3Schools — Python File Handling](https://www.w3schools.com/python/python_file_handling.asp)
- [GeeksforGeeks — File Handling in Python](https://www.geeksforgeeks.org/python/file-handling-python/)
