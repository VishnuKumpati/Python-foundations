# Operators & Expressions

In Unit 1.2 you learned how to give a value a name, and you finished by collecting a value straight from a person using `input()` — only to discover the one gotcha that trips up nearly everyone the first time: whatever the user types comes back as a `str`, even if it looks exactly like a number. That single discovery hints at something bigger. So far, every value you have created has just sat there, stored under a name, waiting. A variable that only ever gets printed back out is not doing much work yet.

Real programs do not just store values — they act on them. A food-delivery app has to add up your cart. A banking app has to check whether your balance covers a withdrawal. A railway-booking site has to decide, instantly, whether a seat is still available. All three are doing the same kind of thing: taking values you already know how to store, and combining or comparing them to produce a new answer. That "combining and comparing" is exactly what this chapter is about — arithmetic, comparisons, and logic, and the fixed order Python follows when more than one of them appears in the same line.

By the time you finish this chapter, an expression like `final_bill >= 50 and has_student_card and not order_cancelled` will not look like a wall of symbols — it will read almost like a sentence, and you will know exactly which piece Python works out first.

---

## From storing values to doing something with them

Picture a college snack stall. You already know the price of a samosa and how many you bought — that's just data sitting in variables, exactly as in Unit 1.2. But the moment you ask "how much do I owe?" or "do I have enough cash?", you have stopped merely storing numbers and started *doing something* with them.

An **operator** is a symbol or short keyword that tells Python to perform an action on one or more values — add them, compare them, or combine two yes/no questions into one. The values an operator acts on are called its **operands**. In `200 + 50`, the operator is `+` and the operands are `200` and `50`. String several operands and operators together and you get an **expression** — something Python works through step by step until it collapses down to exactly one answer. `4 * 15` is an expression. So is `age >= 18`. So is `is_member and not is_banned`. However long an expression grows, Python always reduces it to a single value in the end — that value is what gets stored, printed, or compared next.

Say out loud, in one sentence, what makes `total = 200 + 50` different from `total = 250` in terms of the *work* Python does to get there — both end with the same value in `total`, but only one of them involved an expression Python had to evaluate.

## Arithmetic operators: the calculations underneath everything

**Arithmetic operators** work on the `int` and `float` values from Unit 1.2, and most of them behave exactly the way your school maths already taught you.

| Operator | Name | Example | Result |
|---|---|---|---|
| `+` | addition | `7 + 3` | `10` |
| `-` | subtraction | `7 - 3` | `4` |
| `*` | multiplication | `7 * 3` | `21` |
| `/` | true division | `7 / 2` | `3.5` |
| `//` | floor division | `7 // 2` | `3` |
| `%` | modulo (remainder) | `7 % 2` | `1` |
| `**` | exponentiation | `7 ** 2` | `49` |

Three of these deserve a slower look, because they are where beginners most often get surprised.

**True division (`/`) always produces a `float`, even when the numbers divide evenly.** Before checking, predict what `6 / 2` gives you — most learners guess `3`, but Python hands back `3.0`.

```python
print(6 / 2)
print(7 / 2)
```

```
3.0
3.5
```

**Floor division (`//`)** rounds the result *down*, toward negative infinity, discarding the leftover. For positive numbers this feels like "just chop off the decimal part" — `7 // 2` is `3`. But it is not the same as rounding toward zero once negative numbers get involved: `-7 // 2` gives `-4`, not `-3`, because the true answer `-3.5` gets rounded *down*, and down from `-3.5` is `-4`.

**Modulo (`%`)** gives you whatever floor division left over, and it takes the sign of the number on the right (the divisor) — so `-7 % 2` is `1`, not `-1`. These two operators always fit together: `(a // b) * b + (a % b)` reconstructs `a` exactly, which is a handy way to sanity-check your own arithmetic.

The type of an arithmetic result depends on what you started with: if both operands are `int`, then `+`, `-`, and `*` hand back an `int`. The instant even one operand is a `float`, the result "promotes" to `float` — `7 + 3` is `10`, but `7 + 3.0` is `10.0`. And dividing by zero with `/`, `//`, or `%` does not quietly give you `0` or "infinity" the way a calculator display might — it raises a **`ZeroDivisionError`**, the same kind of halt-and-report behaviour you already saw from `NameError` and `SyntaxError` in Unit 1.2.

**Trying to divide anything by zero stops your program immediately with a `ZeroDivisionError` — Python never silently invents an answer.**

## Operator precedence: which one runs first

The moment an expression contains more than one operator, a natural question appears: does Python just work left to right? Before reading on, try predicting the result of `2 + 3 * 4` yourself.

```python
print(2 + 3 * 4)
```

```
14
```

If you guessed `20` (reading strictly left to right, `2 + 3` first), that is exactly the trap. Python follows a fixed ranking called **operator precedence**, and multiplication outranks addition, so `3 * 4` is computed first, giving `12`, then `2 + 12` gives `14`. From highest priority (evaluated first) to lowest (evaluated last), the ladder looks like this:

1. `()` — parentheses, for grouping
2. `**` — exponentiation
3. `-x` — unary minus (negating a single value)
4. `*`, `/`, `//`, `%` — the multiplication/division family
5. `+`, `-` — addition/subtraction
6. `<`, `<=`, `>`, `>=`, `==`, `!=` — comparisons
7. `not`
8. `and`
9. `or` — evaluated last

Two consequences fall straight out of this ladder. First, `**` binds *tighter* than unary minus, so `-2 ** 2` is read as "negate the result of `2 ** 2`," giving `-4`, not `4` — to actually square `-2` you must write `(-2) ** 2`. Second, `2 + 3 > 4 and 1 < 2` is read as `((2 + 3) > 4) and (1 < 2)`, because arithmetic outranks comparison, and comparison outranks logical operators.

When two operators share the *same* rung on the ladder, Python breaks the tie with **associativity** — working left to right. So `20 / 4 * 2` is `(20 / 4) * 2 = 10.0`, not `20 / (4 * 2) = 2.5`.

You never actually need to memorise this whole ladder. **Whenever the intended order isn't obvious at a glance, wrap the part you want computed first in parentheses `()`.** Parentheses always override precedence, they cost nothing to add, and they can never turn a correct expression into a wrong one — treat them as free insurance for your own readability, not as a last resort.

## Comparison operators: asking a true/false question

A **comparison operator** compares two operands and always hands back a `bool` — `True` or `False`, never anything else.

| Operator | Name | Example | Result |
|---|---|---|---|
| `==` | equal to | `5 == 5` | `True` |
| `!=` | not equal to | `5 != 3` | `True` |
| `<` | less than | `3 < 5` | `True` |
| `>` | greater than | `3 > 5` | `False` |
| `<=` | less than or equal to | `5 <= 5` | `True` |
| `>=` | greater than or equal to | `3 >= 5` | `False` |

Python also lets you **chain** comparisons in a way that reads close to plain English: `1 < x < 10` evaluates as a single combined condition meaning "is `x` between 1 and 10?" — rather than forcing you to write two separate comparisons joined by `and`.

It is legal Python to compare values of different types, such as `5 == "5"` — a number against text. Try predicting the result before you check: it does not crash, and it does not come out `True` just because they look alike on the page. It evaluates to `False`, because a number and a string are never considered equal, no matter how similar they print.

## `=` versus `==` — the single most common beginner mistake

This distinction earns its own section because it causes more early bugs than anything else in this topic, and it directly builds on the assignment operator you met back in Unit 1.2.

| | `=` (Assignment) | `==` (Equality comparison) |
|---|---|---|
| Purpose | Binds a name to a value | Asks whether two values are equal |
| Result | No result value — it performs an action | Always a `bool`: `True` or `False` |
| Example | `x = 5` — stores `5` in `x` | `x == 5` — asks "does `x` currently equal `5`?" |
| Where it's used | Only to create or update a variable | Anywhere an expression can appear |

**A single `=` stores a value; a double `==` asks a question and hands back an answer — mixing the two up is one of the most common bugs a beginner will ever write.** A habit that prevents most of the confusion: when reading code aloud, say "gets" for `=` (`x gets 5`) and "equals?" for `==` (`x equals 5?`).

## Logical operators and short-circuit evaluation

A **logical operator** — `and`, `or`, or `not` — combines or inverts `bool` values (and, as the next section shows, not only `bool` values).

| Operator | Name | Example | Result |
|---|---|---|---|
| `and` | true only if both sides are true | `True and False` | `False` |
| `or` | true if at least one side is true | `True or False` | `True` |
| `not` | flips a single value | `not True` | `False` |

`and` and `or` share a behaviour called **short-circuit evaluation**: Python stops the instant the final answer is already certain, without ever looking at the remaining side. For `A and B`, if `A` is already falsy, the whole thing must be `False`, so `B` never runs at all. For `A or B`, if `A` is already truthy, the whole thing must be `True`, so `B` never runs either.

This is not just trivia — it has a real, sometimes surprising consequence. Code sitting on the right-hand side of `and` or `or` that would normally raise an error may simply never execute, because the left side already decided the outcome. `False and expensive_check()` never calls `expensive_check()` at all. Say out loud, in one sentence, why a PIN check placed before a balance check in a payment app (`pin_correct and balance_sufficient`) means the balance is never even looked up once the PIN has already failed.

## Truthiness: how Python judges *any* value as true-ish or false-ish

**Truthiness** is Python's rule for treating *any* value — not just the literal `True`/`False` from the `bool` type — as true-ish or false-ish whenever it turns up as an operand of `and`, `or`, or `not`.

Python's list of **falsy** values is short and fixed: `False`, `0`, `0.0`, and `""` (the empty string). Every other value is **truthy** — including negative numbers and, importantly, the *text* `"False"`, which is a non-empty string and therefore truthy, not falsy. The built-in `bool()` function from Unit 1.2 reveals a value's truthiness directly:

```python
print(bool(""))
print(bool("hello"))
print(bool(0))
```

```
False
True
False
```

**The empty string `""` is falsy, but the string `"False"` is truthy — it is easy to assume otherwise, and that assumption is a real source of bugs.** Relying on truthiness directly (`if coupon_code:`) rather than spelling it out longhand (`if coupon_code != "":`) is considered cleaner Python style, and you will see it constantly once you reach conditionals in a later unit.

## Putting it all together: a snack-stall bill

Every operator family from this chapter shows up naturally in one small, realistic example — a snack stall working out a shared bill.

```python
samosa_price = 15
samosa_qty = 4
packing_fee = 10
friends_sharing = 3
has_student_card = True
coupon_code = ""
order_cancelled = False

final_bill = samosa_price * samosa_qty + packing_fee
share_per_friend = final_bill // friends_sharing
leftover_rupees = final_bill % friends_sharing

qualifies_by_amount = final_bill >= 50
has_coupon = bool(coupon_code)
not_cancelled = not order_cancelled
discount_eligible = qualifies_by_amount and has_student_card and not_cancelled
free_packing = discount_eligible or has_coupon

print(final_bill)
print(share_per_friend, leftover_rupees)
print(discount_eligible, free_packing)
```

```
70
23 1
True True
```

`final_bill` needs no parentheses at all, because `*` already outranks `+` on the precedence ladder: Python computes `15 * 4 = 60` first, then adds `10`, giving `70`. Floor division then splits that bill into whole rupees per friend (`70 // 3` is `23`), and modulo hands back whatever floor division couldn't split evenly (`70 % 3` is `1`) — check the identity from earlier and `(23 * 3) + 1` does reconstruct `70` exactly. `has_coupon` leans on truthiness alone: an empty string is falsy, so this comes out `False` with no need to write `coupon_code != ""`. And `free_packing` is the short-circuit moment — because `discount_eligible` is already `True`, Python never even bothers looking at `has_coupon` to decide the `or`.

A few mistakes are worth watching for deliberately as you start writing your own expressions:

- Writing `=` where you meant `==`, or the reverse — one stores, the other asks a question, and Python will not warn you which one you meant.
- Assuming `/` and `//` are interchangeable — pick the one that matches whether you actually want a fractional answer or a whole-number count.
- Forgetting that `%` follows the sign of the divisor, not the number being divided, when negative numbers are involved.
- Comparing values of very different types (`5 == "5"`) and being surprised it's `False` rather than an error.
- Assuming any non-empty-looking value is truthy the same way — remember only `False`, `0`, `0.0`, and `""` are falsy; everything else, including `"False"` as text, is truthy.

## Try it yourself

Do this in a Colab cell before moving on. A snack stall sells `5` samosas at `₹15` each and `2` cold drinks at `₹20` each, plus a `₹10` packing fee. Compute `final_bill` using arithmetic operators. Store `qualifies = final_bill >= 100`, and `has_coupon = bool("")`. Combine them with `discount = qualifies and has_coupon`. Print all four values, then work out for yourself, in one sentence, why `discount` comes out `False` even when `qualifies` is `True` — the answer lives entirely in how `and` and truthiness interact.

---

### Key Terminology

- **Operator** — a symbol or keyword that tells Python to perform an operation on one or more values.
- **Operand** — a value that an operator acts on.
- **Expression** — any combination of values, variables, and operators that Python reduces to exactly one result.
- **True division (`/`)** — division that always produces a `float`, even when it divides evenly.
- **Floor division (`//`)** — division that rounds its result down toward negative infinity, discarding the remainder.
- **Modulo (`%`)** — the remainder left over after floor division; takes the sign of the divisor.
- **`ZeroDivisionError`** — raised when `/`, `//`, or `%` attempts to divide by zero.
- **Operator precedence** — the fixed ranking that decides which operator in an expression is evaluated first.
- **Associativity** — the left-to-right tie-breaking rule Python uses between operators of equal precedence.
- **Comparison operator** — an operator (`==`, `!=`, `<`, `>`, `<=`, `>=`) that always produces a `bool`.
- **Chained comparison** — writing something like `1 < x < 10` as a single combined condition.
- **Logical operator** — `and`, `or`, or `not`; combines or inverts values, often `bool` values.
- **Short-circuit evaluation** — Python stopping evaluation of `and`/`or` the instant the final result is already certain.
- **Truthiness** — Python's rule for treating any value as true-ish or false-ish in a logical context.
- **Falsy** — one of Python's fixed set of "false-ish" values: `False`, `0`, `0.0`, `""`.
- **Truthy** — any value that is not falsy.

### Mastery Checkpoint

Before moving to Unit 1.4, check that you can answer these without looking back:

1. Why does `6 / 2` give you `3.0` instead of `3`, and how does that differ from what `6 // 2` gives you?
2. Given `-7 // 2` and `-7 % 2`, what does each evaluate to, and why doesn't the modulo result come out negative?
3. Without evaluating it fully in your head first, explain why `2 + 3 > 4 and 1 < 2` evaluates to `True` — which operator runs first, and why?
4. What is the difference between what `x = 5` does and what `x == 5` does, and why does mixing them up rarely cause an error message that clearly points to the mistake?
5. In `False and some_function()`, why is it guaranteed that `some_function()` never actually runs?

### Summary

You now know how to move beyond simply storing values to actually doing something with them: the seven arithmetic operators and the three that most often surprise beginners (`/`, `//`, `%`), the fixed precedence ladder Python follows whenever an expression mixes several operators, and the parentheses that let you override that ladder whenever you want to be certain. You have seen that every comparison produces a `bool`, that `=` and `==` are entirely different operations wearing similar-looking symbols, and that `and`/`or`/`not` combine conditions using short-circuit evaluation — a behaviour that can silently skip code on the right-hand side. Finally, you have met truthiness, Python's rule for judging any value, not just `True` and `False`, as true-ish or false-ish. From here, the next step is learning how to turn these individual expressions into full statements, convert values between types on purpose, and control exactly how your output is displayed — starting with Unit 1.4, Statements, Conversion & Output.

### Additional Resources

- [Python Tutorial — official docs: "Using Python as a Calculator"](https://docs.python.org/3/tutorial/introduction.html#using-python-as-a-calculator)
- [Python 3 Documentation — Operator Precedence](https://docs.python.org/3/reference/expressions.html#operator-precedence)
- [Python 3 Documentation — Comparisons](https://docs.python.org/3/reference/expressions.html#comparisons)
- [Python 3 Documentation — Boolean Operations (`and`, `or`, `not`)](https://docs.python.org/3/library/stdtypes.html#boolean-operations-and-or-not)
- [Python 3 Documentation — Truth Value Testing](https://docs.python.org/3/library/stdtypes.html#truth-value-testing)
- [W3Schools — Python Operators](https://www.w3schools.com/python/python_operators.asp)
- [W3Schools — Python Booleans](https://www.w3schools.com/python/python_booleans.asp)
