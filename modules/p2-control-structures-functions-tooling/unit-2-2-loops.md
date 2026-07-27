# Loops

Now that your program can choose between paths with `if`/`elif`/`else`, the next question is how to repeat an action without writing it out by hand every time. Every program you have written so far runs top to bottom, once, and stops. That is fine for a handful of lines, but think about what a bank clerk actually has to do with today's transaction statement: check every single row for anything suspicious. There might be ten transactions today, or ten thousand tomorrow — nobody is going to write ten thousand separate `if` checks, one per line of code.

Python's answer to "do this again, and again, and again" is the **loop**. A loop takes one block of code and repeats it — either for as long as some condition stays true, or once for every item in a group of items — without you writing that block out more than once. This chapter covers the two loop tools Python gives you, `while` and `for`, along with the small toolkit that makes them genuinely useful in real programs: `range()`, `enumerate()`, `zip()`, and the `break`/`continue` keywords that let you redirect a loop mid-flight.

By the end of this chapter, you will be able to look at a repeating task — checking every transaction, searching every seat on a train, printing every row of a report — and know exactly which loop to reach for, and how to stop it at precisely the right moment.

---

## Why "copy the line and change the number" doesn't scale

Picture yourself writing a program that prints the seat number for every seat in a single train coach with 5 seats. Without loops, you would write `print("Seat 1")`, then `print("Seat 2")`, and so on, five times. Now imagine that coach has 72 seats instead of 5 — or the program needs to work for *any* coach, without you knowing the seat count in advance. Copy-pasting a line and changing one number stops being a strategy the moment the count is large or unknown, and it is exactly the wrong habit to build now.

A **loop** is a control structure that repeats a block of code — the **loop body** — multiple times, either until a condition becomes `False` or until a group of items is used up. The loop body is the indented block underneath the loop's first line, governed by the exact same indentation-as-grammar rule you already used for `if` blocks. Each single pass through the loop body is called an **iteration** — run the body five times, and the loop has completed five iterations.

Before reading on, think of one more repeating task from your own life outside code — attendance being called out row by row, or every item in a shopping cart being scanned at a billing counter — and notice that both share the same shape: one action, repeated, until something runs out.

## The `while` loop: repeat as long as a condition holds

A **`while` loop** repeats its body for as long as a given condition stays `True`. Its shape is:

```python
while condition:
    # loop body
```

`condition` is any expression that evaluates to `True` or `False` — the same kind of boolean expression you already write inside an `if`. Python checks `condition` *before* every iteration, including the very first one, so if `condition` is already `False` when the loop starts, the body never runs at all.

```python
orders_pending = 3

while orders_pending > 0:
    print("Processing order... orders left:", orders_pending)
    orders_pending = orders_pending - 1

print("All orders processed.")
```

```
Processing order... orders left: 3
Processing order... orders left: 2
Processing order... orders left: 1
All orders processed.
```

Before checking, try predicting how many times "Processing order..." prints if `orders_pending` starts at `0` instead of `3`. It prints zero times — the condition `0 > 0` is already `False`, so the body never runs even once, exactly as the rule above states.

Notice the line `orders_pending = orders_pending - 1`. It is not decoration — it is the entire reason this loop ever stops. It is what moves the condition, iteration by iteration, toward becoming `False`.

**A `while` loop that never changes anything its condition depends on will keep running forever — this is called an infinite loop, and it is the single most common mistake beginners make with `while`.** Before you ever run a `while` loop, get in the habit of reading its body back to yourself and confirming there is a line that pushes the condition toward `False`. If you ever do get stuck with a cell that seems to hang forever, that is almost always exactly what has happened — go back and check what the condition depends on, and whether the body actually changes it.

## The `for` loop: repeat once per item, no counting required

A **`for` loop** takes a different approach entirely: instead of checking a condition, it repeats its body once for each item in a sequence, and needs no condition of your own to write or maintain. Its shape is:

```python
for item in sequence:
    # loop body
```

`item` is the **loop variable** — a name you choose, which takes on each item's value in turn, one per iteration. `sequence` is anything Python can walk through one item at a time — this general category is called an **iterable**, and a string is the simplest example you already know. Recall `order_id = "SWG10234"` from an earlier chapter — you can loop straight over its characters:

```python
order_id = "SWG10234"
for character in order_id:
    print(character)
```

```
S
W
G
1
0
2
3
4
```

Try it yourself before checking: predict how many lines this prints, and confirm it against the length of `order_id`. Unlike a `while` loop, a `for` loop's number of iterations is decided entirely by how many items are in `sequence` — there is no separate stop condition to write, and no risk of an infinite loop, because the loop ends the instant the sequence runs out of items.

So which do you reach for? The choice comes down to what you already know before the loop starts.

| | `while` loop | `for` loop |
|---|---|---|
| Use it when | You're repeating until a condition changes, and you can't know the count in advance | You already know what you're iterating over — a fixed range, a string, a ready-made group of values |
| Stopping mechanism | A condition you write and must remember to update | Automatic — ends when the sequence is exhausted |
| Risk of infinite loop | Yes, if the condition never changes | No |
| Typical example | "Keep asking for a PIN until it's correct" | "Print every character in this order ID" |

## `range()`: generating numbers to loop over

Most of the time you don't have a ready-made string or group of values sitting around — you just want to repeat something a fixed number of times, or count through a sequence of numbers, such as seat numbers on a train. That is exactly what the built-in **`range()`** function produces: a sequence of whole numbers, controlled by up to three values — `start`, `stop`, and `step`.

`range(stop)` produces integers starting at `0`, up to but **not including** `stop`:

```python
for seat in range(5):
    print("Seat", seat)
```

```
Seat 0
Seat 1
Seat 2
Seat 3
Seat 4
```

Notice that `range(5)` never produces `5` itself — five numbers come out, `0` through `4`. This "stop is exclusive" behaviour is one of the most common sources of confusion for beginners, and misreading it produces an **off-by-one error** — a bug where a loop runs one time too many, or one time too few. If you actually wanted seats numbered `1` through `5`, you would need `range(1, 6)`:

```python
for seat in range(1, 6):
    print("Seat", seat)
```

```
Seat 1
Seat 2
Seat 3
Seat 4
Seat 5
```

`range(start, stop)` begins counting from `start` instead of `0`. Add a third value and `range(start, stop, step)` counts by `step` instead of by ones — `range(0, 10, 2)` produces `0, 2, 4, 6, 8`, stopping before `10` for the same exclusive-stop reason as before.

**Whenever you use `range()`, say the first and last number it will actually produce out loud before running the cell — that one habit catches almost every off-by-one mistake before it happens.**

## `enumerate()`: knowing your position while you loop

Sometimes the item itself isn't enough — you also need to know *where* you are in the sequence. Printing every digit of a PNR is useful, but printing "position 3 is the digit 7" is more useful still. That is what the built-in **`enumerate()`** function adds: it pairs each item in a sequence with an index (a position number), starting at `0` by default.

```python
pnr = "4Q7K9"
for position, character in enumerate(pnr, 1):
    print("Character", position, "is", character)
```

```
Character 1 is 4
Character 2 is Q
Character 3 is 7
Character 4 is K
Character 5 is 9
```

The second argument to `enumerate()` — here, `1` — changes where the counting starts; leave it out and Python starts counting from `0` instead. Notice how the loop line unpacks each pair directly into two names, `position` and `character`, in one go. Reach for `enumerate()` any time the *position* of an item matters as much as the item itself — a roll number alongside a name, a line number alongside a log entry.

## `zip()`: walking two sequences together

A different need comes up just as often: two related sequences that you want to step through in lockstep, one item from each per pass. The built-in **`zip()`** function does exactly that.

```python
coaches = "ABC"
seat_counts = range(20, 23)

for coach, seats in zip(coaches, seat_counts):
    print("Coach", coach, "has", seats, "seats")
```

```
Coach A has 20 seats
Coach B has 21 seats
Coach C has 22 seats
```

Before checking, predict what happens if `coaches` had a fourth letter, `"ABCD"`, while `seat_counts` still only produced three numbers. `zip()` stops the instant its *shorter* sequence runs out — there is no error and no warning, so `"D"` would simply never appear in the loop at all. Pairing two sequences of mismatched length and being surprised that some items from the longer one silently never show up is a genuinely common mistake — the fix is always to check both lengths match before relying on `zip()`.

## `break` and `continue`: redirecting a loop from the inside

Sometimes you want to leave a loop the moment you've found what you were looking for, or skip just one iteration without ending the whole loop. Python gives you two keywords for exactly this, and both are only valid *inside* a loop body — write either one outside a loop and Python raises a `SyntaxError`.

**`break`** ends the loop immediately: no further iterations run, not even the rest of the current one. **`continue`** skips only the rest of the *current* iteration and jumps straight to the next one — it does not end the loop the way `break` does.

```python
for transaction_id in range(1, 11):
    if transaction_id == 4:
        continue
    if transaction_id == 7:
        print("Suspicious transaction found at ID", transaction_id)
        break
    print("Transaction", transaction_id, "looks fine")
```

```
Transaction 1 looks fine
Transaction 2 looks fine
Transaction 3 looks fine
Transaction 5 looks fine
Transaction 6 looks fine
Suspicious transaction found at ID 7
```

Trace it yourself before reading on: transaction `4` never prints anything at all — `continue` sends the loop straight to `5` without finishing that iteration's body. Transaction `7` prints its message and then `break` fires, so `8`, `9`, and `10` are never even checked. This is exactly the bank-clerk scenario from the start of this chapter: skip the ones already reviewed, stop the instant something suspicious turns up.

Both keywords are normally written directly behind an `if` check, since you only want to break or skip under a specific condition — scattering bare `break`s or `continue`s through a loop body with no condition attached makes the flow far harder to trace.

## Nested loops: loops inside loops

A **nested loop** is a loop placed inside the body of another loop, and it is the standard tool for anything shaped like a grid — a seat chart with rows and columns, a report with sections and line items inside each section. Searching a train for a free seat across multiple coaches is a textbook example: the outer loop walks coaches, the inner loop walks seats within a coach.

```python
for coach in range(1, 3):
    for seat in range(1, 4):
        print("Checking Coach", coach, "Seat", seat)
```

```
Checking Coach 1 Seat 1
Checking Coach 1 Seat 2
Checking Coach 1 Seat 3
Checking Coach 2 Seat 1
Checking Coach 2 Seat 2
Checking Coach 2 Seat 3
```

For every single iteration of the outer loop, the *entire* inner loop runs from start to finish before the outer loop moves on — here, that means 2 outer iterations × 3 inner iterations, six checks in total. Working out that multiplication by hand, before you run a nested loop, is a cheap way to sanity-check that it behaves the way you expect.

**Inside a nested loop, `break` and `continue` only affect the innermost loop they are written in — neither one ever reaches out to stop or skip an outer loop.** If you want a free-seat search to stop *both* loops the moment a seat is found, `break` on its own only exits the inner `for seat in ...` loop; the outer `for coach in ...` loop would still move on to its next coach. The standard fix is a flag variable — set it to `True` the moment you find what you're looking for, `break` out of the inner loop as usual, then check the flag immediately after the inner loop ends and `break` the outer loop too if it's set.

A few mistakes are worth watching for deliberately as you start writing your own loops:

- Writing a `while` loop and forgetting the line that moves its condition toward `False` — the classic infinite loop.
- Misreading `range()`'s exclusive stop and running a loop one time too many or too few.
- Assuming `zip()` will complain about mismatched sequence lengths — it never does, it just stops early and silently.
- Expecting `break` inside a nested loop to stop the outer loop too, when it only ever exits the loop it's written directly inside.
- Scattering `break`/`continue` without a clear `if` guarding each one, making the loop's flow hard to trace later.

## Try it yourself

Do this in a Colab cell before moving on. A cinema has 4 rows, numbered `1` to `4`, with 3 seats each, numbered `1` to `3`. Using two `range()` calls in a nested `for` loop, print every seat in the format `"Row 1, Seat 1"` through `"Row 4, Seat 3"`. Once that works, modify it two ways: add a `continue` that silently skips Row `2` entirely, and add a flag variable together with `break` in both loops so the whole printout stops the moment `"Row 3, Seat 2"` is reached. Trace through the expected output by hand first, then run it and check you predicted correctly.

---

### Key Terminology

- **Loop** — a control structure that repeats a block of code multiple times, either until a condition becomes `False` or until a sequence is exhausted.
- **Loop body** — the indented block of code that gets repeated.
- **Iteration** — one single pass through a loop body.
- **`while` loop** — repeats its body for as long as a condition stays `True`, checked before every iteration.
- **Infinite loop** — a `while` loop whose condition never becomes `False`, so it never stops on its own.
- **`for` loop** — repeats its body once for each item in a sequence, with no condition of your own to maintain.
- **Loop variable** — the name in a `for` loop that takes on each item's value in turn.
- **Iterable** — anything Python can walk through one item at a time, such as a string or the output of `range()`.
- **`range()`** — a built-in function producing a sequence of whole numbers from `start` up to, but not including, `stop`, counting by `step`.
- **Off-by-one error** — a bug where a loop runs one time too many or too few, often from misreading `range()`'s exclusive stop.
- **`enumerate()`** — a built-in function pairing each item in a sequence with a position index.
- **`zip()`** — a built-in function that walks two or more sequences together, stopping when the shortest one runs out.
- **`break`** — ends a loop immediately, skipping all remaining iterations.
- **`continue`** — skips the rest of the current iteration only, then moves to the next one.
- **Nested loop** — a loop placed inside the body of another loop, used for grid-shaped problems.

### Mastery Checkpoint

Before moving to Unit 2.3, check that you can answer these without looking back:

1. What single line of code is missing from a `while` loop when it turns out to be an infinite loop, and why does adding it fix things?
2. Why does `range(5)` produce five numbers, `0` through `4`, rather than the numbers `1` through `5`?
3. What is the difference between what `break` does and what `continue` does inside a loop body?
4. Two sequences passed to `zip()` have different lengths. What happens, and does Python warn you about it?
5. Inside a nested loop, if `break` runs inside the inner loop, does the outer loop also stop? What would you add if you wanted it to?

### Summary

You now know how to repeat a block of code without writing it out by hand every time: a `while` loop that repeats for as long as a condition holds, and a `for` loop that repeats once per item in a sequence with no condition to maintain yourself. You've used `range()` to generate numeric sequences and seen exactly how its exclusive stop causes off-by-one errors, paired `enumerate()` and `zip()` onto a basic `for` loop to track position and walk two sequences together, and used `break` and `continue` to redirect a loop's flow from the inside — including the crucial rule that both only ever affect the innermost loop of a nested pair. From here, the next step is packaging up the code you've written so far into reusable, named pieces you can call again and again — starting with Unit 2.3, Functions.

### Additional Resources

- [Python Tutorial — official docs: "More Control Flow Tools" (while and for statements)](https://docs.python.org/3/tutorial/controlflow.html)
- [Python 3 Documentation — `range()` built-in function](https://docs.python.org/3/library/functions.html#func-range)
- [Python 3 Documentation — `enumerate()` built-in function](https://docs.python.org/3/library/functions.html#enumerate)
- [Python 3 Documentation — `zip()` built-in function](https://docs.python.org/3/library/functions.html#zip)
- [W3Schools — Python While Loops](https://www.w3schools.com/python/python_while_loops.asp)
- [W3Schools — Python For Loops](https://www.w3schools.com/python/python_for_loops.asp)
