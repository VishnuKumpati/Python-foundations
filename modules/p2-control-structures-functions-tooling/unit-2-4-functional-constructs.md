# 2.4 Functional Constructs

---

[← Previous: 2.3 Functions](unit-2-3-functions.md) | [Go back to TOC](../../README.md) | [Next: 2.5 Modules, Packaging & Professional Tooling →](unit-2-5-modules-packaging-tooling.md)


---

## Why Functional Constructs?

You've written functions that take values in and return values out. This unit covers a different way of working with functions: passing them *as values themselves*, writing small throwaway ones inline, wrapping existing functions with extra behavior, and pausing a function midway through instead of running it all at once.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Lambda** | A small, unnamed function written in a single line |
| **Higher-order function** | A function that takes another function as input, or returns one |
| **Decorator** | A function that wraps another function to add behavior, without changing its code |
| **Generator** | A function that produces values one at a time, pausing between each |
| **Lazy evaluation** | Computing a value only when it's actually needed, not ahead of time |

---

## Lambda (Anonymous) Functions

A **lambda** is a small function written in one line, with no `def` and no name:

```python
square = lambda x: x * x
print(square(5))
```
```
25
```

Compare it to the equivalent `def` version:

```python
def square(x):
    return x * x
```

Same thing, just more compact. A lambda can only hold a single expression, no multiple lines, no `if`/`else` blocks, no loops, just one calculation that gets returned automatically.

**Where lambdas are actually useful:** as a quick, throwaway function passed *into* another function, most commonly as a sort key:

```python
words = "apple fig banana".split()
print(sorted(words, key=len))
```
```
['fig', 'apple', 'banana']
```

`key=len` tells `sorted()` to sort by each word's length instead of alphabetically. You'll see `[...]` output like this, that's a **list**, Python's ordered collection type. It gets a full unit starting in 3.1, for now just treat it as a sequence you can loop over or sort.

You could write the sort key as a `lambda` too, for more than a single built-in function:

```python
print(sorted(words, key=lambda w: w[-1]))
```
```
['banana', 'apple', 'fig']
```

This sorts by each word's **last letter**, something `len` alone couldn't do.

---

## Higher-Order Functions

A **higher-order function** takes another function as an argument (or returns one). `sorted(key=...)` above was already one example. Two more common ones:

**`map()`**, applies a function to every item in a sequence:

```python
doubled = map(lambda x: x * 2, range(1, 6))
for value in doubled:
    print(value)
```
```
2
4
6
8
10
```

**`filter()`**, keeps only the items where the function returns `True`:

```python
evens = filter(lambda x: x % 2 == 0, range(1, 11))
for value in evens:
    print(value)
```
```
2
4
6
8
10
```

Both `map()` and `filter()` give back something you loop through, not the finished set of values right away, they compute each result only as you ask for it. That's the same laziness idea you'll see properly in Generators below.

---

## Decorators (a Light Introduction)

A **decorator** wraps a function with extra behavior, without touching the original function's code. It's a higher-order function that takes a function in and returns a new, "upgraded" version of it.

**Start simple: a decorator for a function that takes no arguments.**

```python
def shout(func):
    def wrapper():
        result = func()
        return result.upper() + "!"
    return wrapper

@shout
def greet():
    return "hello"

print(greet())
```
```
HELLO!
```

The `@shout` line right above `def greet():` is exactly the same as writing `greet = shout(greet)`, the `@` syntax is just shorthand for that. `wrapper` calls the original function inside itself and changes what comes back, `greet` itself never had to change.

**Now, what if the function you're decorating takes arguments?** `wrapper()` above has no parameters, so it would break on anything but a zero-argument function. Fix it with `*args, **kwargs` from Unit 2.3, so the decorator works on *any* function, no matter what it takes:

```python
import time

def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    total = 0
    for i in range(1000000):
        total += i
    return total

slow_function()
```
```
slow_function took 0.0421 seconds
```

Same idea as `shout` above, just with `*args, **kwargs` passed straight through to `func`, so `wrapper` can forward whatever arguments the real function needs, without knowing in advance what they'll be.

You won't write decorators often yet, but you'll see `@something` above function definitions constantly in real Python code, and now you know what's actually happening underneath.

---

## Generators

A **generator** is a function that produces a sequence of values one at a time, using `yield` instead of `return`:

```python
def count_up_to(n):
    i = 1
    while i <= n:
        yield i
        i += 1

for number in count_up_to(5):
    print(number)
```
```
1
2
3
4
5
```

**`yield` pauses the function** instead of ending it. Each time the loop asks for the next value, `count_up_to` picks up right where it left off, rather than starting over.

**Calling a generator function doesn't run it immediately:**

```python
gen = count_up_to(5)
print(gen)
```
```
<generator object count_up_to at 0x...>
```

Nothing has run yet, `gen` is a generator object, waiting to be stepped through. The code inside only executes as you actually loop over it (or call `next()` on it).

**This is lazy evaluation**, computing each value only when it's needed, instead of building the entire sequence up front. For a small range like this it barely matters, but for a huge sequence, a generator never holds more than one value in memory at a time, while a function that built and returned a full list of a million items would hold all of them at once.

---

## Try it Yourself

**(a)** Write a lambda that takes a number and returns whether it's negative (`True`/`False`). Test it with `-5` and `5`.

**(b)** Use `filter()` with a lambda to print only the multiples of 3 from `range(1, 30)`.

**(c)** Write a decorator called `announce` that prints `"Calling <function name>..."` before running the wrapped function, then returns its result unchanged. Make it work on functions that take arguments, using `*args, **kwargs`. Apply it to a simple function `add(a, b)` that returns `a + b`.

**Your turn:** write a generator function `even_numbers(n)` that yields even numbers from `2` up to `n`. Loop over `even_numbers(10)` and print each value.

---

## Common Mistakes

- Trying to fit a full `if`/`else` block or multiple statements into a `lambda`, it can only hold one expression
- Forgetting `map()` and `filter()` give back something you need to loop over (or convert), not a ready-made result
- Assuming `@decorator` is special magic, it's just `func = decorator(func)` with nicer syntax
- Forgetting a decorator's inner function needs `*args, **kwargs` to work on functions with different parameters
- Assuming calling a generator function runs its code immediately, it doesn't, nothing happens until you iterate it
- Using `return` instead of `yield` inside a generator, which ends it instead of pausing it

---

## Interview Questions

**Q1: What's the difference between `lambda` and a regular function?**

A: A lambda is a small, unnamed function limited to a single expression, no statements, no loops, no multiple lines. A regular `def` function can hold as much logic as needed and must be given a name.

**Q2: What is a higher-order function?**

A: A function that takes another function as an argument, or returns one. `sorted(key=...)`, `map()`, `filter()`, and decorators are all examples.

**Q3: What does the `@` symbol actually do above a function definition?**

A: It applies a decorator, shorthand for `func = decorator(func)`. The function underneath gets replaced with whatever the decorator returns.

**Q4: What's the difference between `return` and `yield`?**

A: `return` ends a function and hands back one value. `yield` pauses the function, hands back one value, and resumes exactly where it left off the next time a value is requested.

**Q5: What is lazy evaluation, and why does it matter?**

A: Computing a value only when it's actually needed, rather than all at once ahead of time. It matters for large sequences, a generator holds only one value in memory at a time, instead of building an entire result set upfront.

---

## Quick Recap

- A `lambda` is a one-expression, unnamed function, most useful as a quick argument to another function.
- A higher-order function takes a function as input or returns one; `sorted(key=...)`, `map()`, and `filter()` are common examples.
- A decorator wraps a function with extra behavior using `@`, shorthand for `func = decorator(func)`.
- A generator uses `yield` to produce values one at a time, pausing between each, instead of computing everything upfront.
- Calling a generator function doesn't run its code, iterating over it does.

Next up: **Unit 2.5, Modules & Packaging**, organizing code across files and managing external packages.

##  Reference Links

- [Python 3 Documentation — Lambda Expressions](https://docs.python.org/3/reference/expressions.html#lambda)
- [Python 3 Documentation — Functional Programming HOWTO](https://docs.python.org/3/howto/functional.html)
- [Python 3 Documentation — Generators (The Python Tutorial, Classes chapter)](https://docs.python.org/3/tutorial/classes.html#generators)
- [Real Python — How to Use Python Lambda Functions](https://realpython.com/python-lambda/)
- [Real Python — Primer on Python Decorators](https://realpython.com/primer-on-python-decorators/)
- [Real Python — Introduction to Python Generators](https://realpython.com/introduction-to-python-generators/)
- [W3Schools — Python Lambda](https://www.w3schools.com/python/python_lambda.asp)

[← Previous: 2.3 Functions](unit-2-3-functions.md) | [Go back to TOC](../../README.md) | [Next: 2.5 Modules, Packaging & Professional Tooling →](unit-2-5-modules-packaging-tooling.md)

---

*© 2026 Revature · AI Native Engineering — Foundations · Unit 2.4 · Version 2.0*
