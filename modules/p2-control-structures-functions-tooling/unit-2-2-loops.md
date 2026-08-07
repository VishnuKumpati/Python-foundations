# 2.2 Loops

---

[← Previous: 2.1 Conditionals](unit-2-1-conditionals.md) | [Go back to TOC](../../README.md) | [Next: 2.3 Functions →](unit-2-3-functions.md)

---

## Why Loops?

Without a loop, printing something 10 times means writing `print()` 10 times. Loops let you repeat code without repeating yourself, run the same block again and again, either a fixed number of times or until a condition changes.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Loop** | Code that repeats |
| **Iteration** | One pass through the loop body |
| **Iterable** | Something you can loop over, one item at a time (a string, a range, later a list) |
| **Infinite loop** | A loop whose condition never becomes `False`, runs forever |
| **`break`** | Exits the loop immediately |
| **`continue`** | Skips the rest of this iteration, moves to the next one |

---

## `while` Loop

Repeats **as long as** a condition stays `True`:

```python
count = 1

while count <= 5:
    print(count)
    count += 1
```
```
1
2
3
4
5
```

Every `while` loop needs something that eventually makes the condition `False`, here, `count += 1`. Miss that step, and you get an **infinite loop**:

```python
count = 1

while count <= 5:
    print(count)   # count never changes, this runs forever
```

If you ever run one by accident in Colab, use **Runtime → Interrupt execution** to stop it.

**A genuinely useful pattern:** `while True` with a `break` inside, for loops that should run until something specific happens, not a fixed number of times:

```python
while True:
    answer = input("Type 'quit' to stop: ")
    if answer == "quit":
        break
    print("You typed:", answer)
```

---

## `for` Loop

Repeats **once for each item** in something iterable, a string, a `range()`, and later, lists:

```python
for letter in "Python":
    print(letter)
```
```
P
y
t
h
o
n
```

`letter` is the **loop variable**, it takes on each value in turn. You can name it anything, but a descriptive name helps: `for student in ...` reads better than `for x in ...`.

---

## `range()`

The most common thing to loop over when you just need numbers, not text:

```python
for i in range(5):
    print(i)
```
```
0
1
2
3
4
```

`range()` counts from `0` up to, but **not including**, the number you give it, same "stop is excluded" rule you already know from slicing.

**`range()` takes up to three arguments:** `range(start, stop, step)`

```python
for i in range(2, 10, 2):
    print(i)
```
```
2
4
6
8
```

```python
for i in range(10, 0, -1):
    print(i)
```
```
10
9
8
7
6
5
4
3
2
1
```

**Just need to repeat something N times, and don't care about the number itself?** Use `_` as a throwaway variable name, a common Python convention:

```python
for _ in range(3):
    print("Hello!")
```
```
Hello!
Hello!
Hello!
```

---

## `enumerate()`, Index and Value Together

Sometimes you need both the position and the value while looping:

```python
for index, letter in enumerate("abc"):
    print(index, letter)
```
```
0 a
1 b
2 c
```

Without `enumerate()`, you'd have to track the index yourself with a separate counter variable, `enumerate()` does it for you.

---

## `zip()`, Pairing Up Two Sequences

`zip()` walks through two (or more) iterables **at the same time**, pairing up matching positions:

```python
names = "AB"
scores = "12"

for name, score in zip(names, scores):
    print(name, score)
```
```
A 1
B 2
```

If the two are different lengths, `zip()` stops at the **shorter** one, it doesn't raise an error or fill in gaps.

---

## `break` and `continue`

**`break`** exits the loop immediately, skipping everything left, even future iterations:

```python
for i in range(10):
    if i == 5:
        break
    print(i)
```
```
0
1
2
3
4
```

**`continue`** skips only the rest of the *current* iteration, then moves on to the next one:

```python
for i in range(6):
    if i % 2 == 0:
        continue
    print(i)
```
```
1
3
5
```

---

## Nested Loops

A loop inside another loop, the inner loop runs completely for every single pass of the outer one:

```python
for row in range(3):
    for col in range(2):
        print(f"row {row}, col {col}")
```
```
row 0, col 0
row 0, col 1
row 1, col 0
row 1, col 1
row 2, col 0
row 2, col 1
```

Same trade-off as nested conditionals from Unit 2.1, useful for grids and tables, but readability drops fast past two levels.

**The loop `else` clause (brief):** a loop's `else` block runs only if the loop finished normally, without hitting a `break`:

```python
for i in range(5):
    if i == 10:
        break
else:
    print("Loop completed without a break.")
```
```
Loop completed without a break.
```

This is a Python-specific feature, not common in other languages, and easy to skip past for now. It's worth recognizing if you see it in someone else's code, but you won't need it often.

---

## Try it Yourself

**(a)** Print all even numbers from `1` to `20` using `range()` with a step.

**(b)** Loop through the string `"Revature"` with `enumerate()`, printing each letter's position and the letter itself.

**(c)** You have two strings, `subjects = "MEH"` and `marks = "789"`. Use `zip()` to print each subject paired with its mark.

**Your turn:** write a `while` loop that keeps asking `"Enter a number (0 to stop): "` and adds up everything entered, until the user types `0`. Print the total at the end. (Hint: you'll need `int()` to convert the input before adding it.)

---

## Common Mistakes

- Forgetting to update the loop variable in a `while` loop, causing an infinite loop
- Confusing `break` (stops the whole loop) with `continue` (skips just this iteration)
- Assuming `range(5)` includes `5`, it stops right before it, same as slicing
- Changing the loop variable inside a `for` loop and expecting it to affect which items come next, it doesn't, Python already queued up the next value
- Assuming `zip()` raises an error on mismatched lengths, it just stops at the shorter one silently
- Over-nesting loops when the logic could be simpler with a single loop and a condition

---

## Interview Questions

**Q1: What's the difference between `break` and `continue`?**

A: `break` exits the loop entirely. `continue` skips only the rest of the current iteration and moves on to the next one.

**Q2: How do you avoid an infinite `while` loop?**

A: Make sure something inside the loop eventually makes the condition `False`, usually by updating a counter or checking for a specific exit condition (like `while True` paired with a `break`).

**Q3: What does `enumerate()` do, and why use it instead of a manual counter?**

A: It gives you both the index and the value together while looping, without needing to create and increment a separate counter variable yourself.

**Q4: What happens if you `zip()` two sequences of different lengths?**

A: It stops as soon as the shorter one runs out, no error, the extra items in the longer one are simply ignored.

---

## Quick Recap

- `while` repeats as long as a condition is `True`; always make sure something inside it can end the loop.
- `for` repeats once per item in something iterable, a string, `range()`, and later, lists.
- `range(start, stop, step)` counts numbers, stop is always excluded.
- `enumerate()` gives you index and value together; `zip()` pairs up two sequences at once, stopping at the shorter one.
- `break` exits a loop entirely; `continue` skips just the current iteration.
- Nested loops run the inner loop fully for every pass of the outer one.


## Reference Links

- [The Python Tutorial — More Control Flow Tools (for, range, break, continue)](https://docs.python.org/3/tutorial/controlflow.html)
- [Python 3 Language Reference — The `while` Statement](https://docs.python.org/3/reference/compound_stmts.html#the-while-statement)
- [Python 3 Documentation — Built-in Functions: `enumerate()` and `zip()`](https://docs.python.org/3/library/functions.html#enumerate)
- [Real Python — Python "for" Loops](https://realpython.com/python-for-loop/)
- [Real Python — Python "while" Loops](https://realpython.com/python-while-loop/)
- [W3Schools — Python For Loops](https://www.w3schools.com/python/python_for_loops.asp)
- [W3Schools — Python While Loops](https://www.w3schools.com/python/python_while_loops.asp)

[← Previous: 2.1 Conditionals](unit-2-1-conditionals.md) | [Go back to TOC](../../README.md) | [Next: 2.3 Functions →](unit-2-3-functions.md)

---

*© 2026 Revature · AI Native Engineering — Foundations · Unit 2.2 · Version 2.0*
