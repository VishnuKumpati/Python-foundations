# Conditionals

By the end of Unit 1.4 you could take a raw value, convert it with `int()` or `float()`, and print it back out in a neat, readable f-string. Your programs, by this point, look genuinely polished — but notice something they still cannot do. Every line you have written so far runs no matter what the data actually is. `print()` fires whether the cart is empty or full. A conversion happens whether the number is sensible or nonsensical. Run the same notebook twice with two different inputs and you get two different-looking outputs, sure, but the exact same sequence of steps runs both times, in the exact same order, top to bottom.

Real software cannot work that way. A food-delivery app has to decide, on the spot, whether your order qualifies for free delivery. A bank has to decide whether your account actually holds enough money before it lets a withdrawal go through. A railway-booking site has to decide, the instant you click "book," whether a seat is even still available. None of that is "run every line in order" — it is "look at a fact, and choose a path based on it." You already have the exact tool for producing that fact: the boolean expressions from Unit 1.3, things like `cart_value >= 500` or `age >= 18 and has_id`, which collapse down to a single `True` or `False`. What has been missing is a way to say "and if that's `True`, do *this* — otherwise, do *that* instead."

This unit supplies exactly that missing piece: the **conditional**. By the time you finish, you will be able to write a program that reacts differently depending on what it's given — sorting a value into one of several categories, checking two facts at once, nesting one decision inside another when that is genuinely the shape of the problem, and collapsing a simple either/or choice into a single line. This is the first real decision-making tool in the entire course, and almost everything you write from here on — loops, functions, whole applications — assumes you can already make your program choose.

---

## From "always the same script" to a program that decides

Picture a Swiggy-style food-delivery app at the exact moment you tap "Place Order." Somewhere inside, the app has to answer a genuine either/or question: is your cart big enough to waive the delivery fee, or not? There is no version of this app that just always waives the fee, and no version that always charges it — the answer depends entirely on the value sitting in your cart at that moment, and the program has to look at that value and pick one of two outcomes.

A **conditional** is a control structure that runs one block of code, or a different block, depending on whether a **condition** — a boolean expression, exactly the kind you built in Unit 1.3 — evaluates to `True` or `False`. Python's basic conditional keyword is `if`:

```python
temperature = 30

if temperature > 25:
    print("It is warm.")
```

```
It is warm.
```

Read this as "if the condition `temperature > 25` is `True`, run the indented line below it." Since `30 > 25` is `True`, the `print()` line runs. Before checking, predict what happens if you change `temperature` to `20` and rerun the cell — the condition becomes `False`, and Python simply skips that line and moves on to whatever comes next, without raising any error at all. Nothing crashes; the line is just never reached.

A condition can be anything that reduces to a `bool`: a comparison such as `age >= 18`, a variable that already holds `True` or `False` (`if is_logged_in:`), or a compound expression built with `and`, `or`, and `not`, which you'll return to shortly. Say out loud, in one sentence, what `if` actually adds on top of a plain boolean expression like `cart_value >= 500` — the expression alone only produces an answer; `if` is what lets that answer decide whether a block of code runs at all.

## Blocks and indentation: how Python knows what belongs where

The **block** is the group of statements that belong to one branch of a conditional — the lines that run together when that branch's condition holds. Some languages mark a block with punctuation, such as curly braces `{ }`. Python does not. It uses **indentation** — the whitespace at the start of a line — and the standard is 4 spaces per level.

```python
if temperature > 25:
    print("It is warm.")    # indented -- inside the block
print("Done checking.")     # not indented -- outside the block, always runs
```

```
It is warm.
Done checking.
```

Notice that `print("Done checking.")` runs regardless of the temperature, because it sits outside the indented block entirely — it belongs to the program's main flow, not to the `if`. Every `if`, `elif`, or `else` line ends with a colon `:`, which is Python's own signal that "the indented block right after this line is what belongs to it."

**Indentation in Python is grammar, not decoration — get it wrong and Python refuses to run the program at all, raising an `IndentationError`.** Mixing 2 spaces and 4 spaces, or tabs and spaces, within what should be the same block is enough to trigger it. Try it yourself: type the `if` example above into a Colab cell, then deliberately indent `print("It is warm.")` by only 2 spaces instead of 4, and read the error Python gives you — recognising that message on sight will save you real time later.

## Multi-branch decisions: `elif` and `else`

A bare `if` can only decide between "run this block" and "don't." Most real decisions have more than two outcomes — think of an IRCTC-style booking status that could be Confirmed, Waitlisted, or Cancelled, not just yes-or-no. That is exactly where `elif` (short for "else if") and `else` come in:

```python
cart_value = 350.0

if cart_value >= 500:
    delivery_fee = 0.0
elif cart_value >= 200:
    delivery_fee = 20.0
else:
    delivery_fee = 40.0

print(delivery_fee)
```

```
20.0
```

A **branch** is one of these possible paths — one `if`, `elif`, or `else` block. Trace it by hand before trusting the output: `cart_value >= 500` is `350.0 >= 500`, which is `False`, so Python moves on. `cart_value >= 200` is `350.0 >= 200`, which is `True`, so `delivery_fee` becomes `20.0` — and the `else` is skipped entirely, even though Python never actually evaluated it.

A handful of strict rules govern this chain:

- Exactly one `if` starts a conditional. `elif` and `else` are both optional.
- There can be any number of `elif` branches, but at most one `else`, and it must come last.
- Conditions are checked **in order, top to bottom**, and Python runs the block belonging to the *first* condition that is `True`, then skips every remaining `elif`/`else` — even one that would also have matched.
- If nothing matches and there is no `else`, Python runs nothing from the conditional at all and simply continues. This is silent: no error, no message. That silence is precisely why a missing `else` can hide a real bug — an unexpected input gets handled by doing absolutely nothing, and nobody is told.

**Because only the first matching branch runs, branch order is not a style choice — it can change your program's actual behaviour.** Suppose the two `elif` branches above had been written in the opposite order:

```python
cart_value = 350.0

if cart_value >= 200:
    delivery_fee = 20.0
elif cart_value >= 500:
    delivery_fee = 0.0
else:
    delivery_fee = 40.0

print(delivery_fee)
```

```
20.0
```

The output looks identical here, but something has quietly broken: a cart worth `600` would now also match `cart_value >= 200` first and get charged `20.0`, never reaching the `cart_value >= 500` branch it should have qualified for — because every value that satisfies "at least 500" also satisfies "at least 200." Before checking, predict what `delivery_fee` this second version produces for `cart_value = 600` — if you guessed `20.0` instead of the intended `0.0`, you've just found exactly the bug that wrong ordering causes. Ordering an `elif` chain from most specific to least specific is what keeps it correct.

## Compound conditions inside an `if`

A **compound condition** combines two or more boolean expressions using `and`, `or`, or `not` — the same logical operators from Unit 1.3 — directly inside an `if` or `elif`:

```python
cart_value = 350.0
is_premium_member = True

if cart_value >= 500 or is_premium_member:
    delivery_fee = 0.0
else:
    delivery_fee = 40.0

print(delivery_fee)
```

```
0.0
```

`cart_value >= 500` is `False` on its own, but `is_premium_member` is `True`, and because the operator is `or`, only one side needs to hold for the whole condition to be `True`. Had the operator been `and`, both sides would have needed to be `True`, and `delivery_fee` would have landed on `40.0` instead. A **chained comparison** — the range-check shorthand from Unit 1.3, such as `0 <= score <= 100` — works exactly the same way sitting inside an `if`, and reads closer to plain English than writing `score >= 0 and score <= 100` out longhand.

**When `and` and `or` appear together in the same condition, wrap the parts in parentheses so the grouping is unambiguous at a glance** — for example, `if (age >= 18 and has_id) or is_guest:` — rather than relying on precedence rules from Unit 1.3 that are easy to misremember under pressure.

## Nesting: when one decision genuinely depends on another

**Nesting** means placing one conditional entirely inside the block of another. It earns its place only when an inner decision only makes sense *after* an outer question has already been settled — not simply because two facts both happen to matter at once.

```python
restaurant_open = True
cart_value = 350.0
is_premium_member = True

if restaurant_open:
    if cart_value >= 500 or is_premium_member:
        delivery_fee = 0.0
    else:
        delivery_fee = 40.0
    print("Delivery fee: Rs.", delivery_fee)
else:
    print("Restaurant is currently closed.")
```

```
Delivery fee: Rs. 0.0
```

Asking "what's the delivery fee?" for a restaurant that is closed is not a smaller version of the same question — it is a meaningless one. That dependency is exactly what justifies nesting here: the outer `if restaurant_open:` is checked first, and the inner fee logic is only reached, and only evaluated, once the outer condition is confirmed `True`. Set `restaurant_open = False` yourself and rerun this cell — notice the inner `if`/`else` never runs at all, not even to check `cart_value`, because Python never even steps inside the outer block.

| | Flattened compound condition | Nested conditional |
|---|---|---|
| Use when | Two facts are simply both required together | An inner decision only makes sense once an outer one is already settled |
| Example | `if age >= 18 and has_id:` | `if restaurant_open:` then `if cart_value >= 500:` inside it |
| Readability at 2 levels | Usually clearer | Still fine |
| Readability at 3–4+ levels | Stays flat and scannable | Becomes hard to trace by eye |

**Stacking three or four levels of nested `if` statements when a single `and`/`or` expression would say exactly the same thing is a common, avoidable source of hard-to-follow code.** Before writing a nested `if`, ask yourself the one-sentence test: does the inner question only make sense because the outer one was answered a certain way? If the two conditions are really just both required at once, with no such dependency, flatten them into one compound condition instead.

## The conditional (ternary) expression

Everything so far is a **statement**: it directs which lines run, but the `if`/`elif`/`else` construct itself does not hand back a value you could store or print directly. Python also offers a **conditional expression** — informally called a **ternary expression** — a compact, one-line form that *does* produce a value:

```
value_if_true if condition else value_if_false
```

```python
age = 20
status = "adult" if age >= 18 else "minor"
print(status)
```

```
adult
```

Python evaluates the condition first: `20 >= 18` is `True`, so the whole expression collapses to `value_if_true`, the string `"adult"` — `"minor"` is never even looked at. That resulting value is then stored in `status` exactly as any ordinary assignment would store it. Before checking, predict what `status` becomes if `age` is `15` instead — the condition turns `False`, and the entire expression produces `"minor"` this time, with `"adult"` skipped.

**The defining difference from a plain `if`/`else` statement: a ternary expression produces a value you can assign, print, or pass along; an `if`/`else` statement only directs control flow and produces nothing of its own.** Keep the ternary form for simple, single-line, two-outcome choices — the moment you need more than two outcomes, chaining ternaries together is technically legal Python but genuinely hard to read, and a full `if`/`elif`/`else` chain is the better tool.

Conditionals of every shape covered here show up constantly outside the classroom: a bank checks `if amount <= balance:` before releasing a withdrawal, where getting the branch order wrong could allow an overdraft; a healthcare dashboard classifies a temperature reading into "Normal," "Fever," or "High Fever" with an `elif` chain identical in shape to the delivery-fee example above; and an AI/ML system often turns a model's confidence score into an accept/reject decision — `if confidence >= 0.8:` accept automatically, otherwise flag it for a human to review.

A short list of mistakes worth watching for deliberately while this topic is still new:

- Writing `if age = 18:` instead of `if age == 18:` — a single `=` is the assignment operator from Unit 1.2, not a comparison, and Python raises a `SyntaxError` rather than silently doing the wrong thing.
- Ordering an `elif` chain from broad to narrow, which can make a later, narrower branch unreachable.
- Leaving out `else` when an unexpected input is genuinely possible, letting your program silently do nothing instead of surfacing the problem.
- Mixing indentation levels within what should be one block, triggering an `IndentationError`.
- Nesting three or four `if` levels deep when one compound condition with `and`/`or` would say the same thing far more clearly.

## Try it yourself

Do this in a Colab cell before moving on. Model the Swiggy-style delivery scenario end to end: start with `restaurant_open = True`, `cart_value = 620.0`, and `is_premium_member = False`. Write an outer `if restaurant_open:` / `else` that prints `"Restaurant is currently closed."` when closed. Inside the open branch, nest an `elif` chain with three tiers — free delivery at `cart_value >= 500` or when `is_premium_member` is `True`, a `20.0` fee at `cart_value >= 200`, and a `40.0` fee otherwise — and print the resulting fee. Then add one line using a ternary expression that turns that fee into a label: `"Free Delivery"` if the fee is `0.0`, otherwise `"Delivery Charges Apply"`. Run it once with your original values, then change `cart_value` to `50.0` and `restaurant_open` to `False` in turn, predicting the output each time before you rerun the cell.

---

### Key Terminology

- **Conditional** — a control structure that runs one block of code, or a different block, depending on whether a condition is `True` or `False`.
- **Condition** — a boolean expression attached to an `if`, `elif`, or ternary expression.
- **Block** — the group of statements belonging to one branch, marked in Python by consistent indentation rather than punctuation.
- **`IndentationError`** — raised when a block's lines are not indented consistently.
- **Branch** — one possible path through a conditional: one `if`, `elif`, or `else`.
- **`elif`** — short for "else if"; adds an additional condition checked only if every condition above it was `False`.
- **`else`** — the branch that runs only if every `if`/`elif` condition above it was `False`.
- **Compound condition** — two or more boolean expressions joined with `and`, `or`, or `not` inside a single condition.
- **Nesting** — placing one conditional entirely inside the block of another, justified when the inner decision depends on the outer one already being settled.
- **Conditional (ternary) expression** — the one-line form `value_if_true if condition else value_if_false`, which produces a value rather than merely directing control flow.

### Mastery Checkpoint

Before moving to Unit 2.2, check that you can answer these without looking back:

1. Why does Python raise no error at all when an `if` condition is `False` and there is no matching `elif` or `else` — and why can that silence itself be a source of bugs?
2. Two `elif` branches check `cart_value >= 200` and `cart_value >= 500`. Why does the order you write them in change which branch actually runs for `cart_value = 600`?
3. What single question should you ask yourself to decide whether two related conditions should be nested or flattened into one compound condition?
4. What does a ternary expression produce that a plain `if`/`else` statement does not, and why does that difference matter?
5. `if age = 18:` fails immediately with a `SyntaxError`, before the program even runs. What is actually wrong with that line, and what should it say instead?

### Summary

You now know how to make a Python program choose between paths instead of always running the same lines in the same order: the `if` statement that attaches an action to a boolean expression, the indentation rules that define a block, and the `elif`/`else` chain that handles more than two outcomes while checking conditions top to bottom and running only the first match. You've combined conditions with `and`/`or`/`not`, nested one conditional inside another only when the inner decision genuinely depends on the outer one, and used a ternary expression to collapse a simple two-outcome choice into a single value-producing line. From here, the next step is learning how to run a block of code not just once, based on a decision, but repeatedly — starting with Unit 2.2, Loops.

### Additional Resources

- [Python Tutorial — official docs: "More Control Flow Tools" (if statements)](https://docs.python.org/3/tutorial/controlflow.html#if-statements)
- [Python 3 Documentation — Compound Statements (`if`, `elif`, `else` grammar)](https://docs.python.org/3/reference/compound_stmts.html#the-if-statement)
- [Python 3 Documentation — Conditional Expressions](https://docs.python.org/3/reference/expressions.html#conditional-expressions)
- [PEP 8 — Style Guide for Python Code](https://peps.python.org/pep-0008/)
- [W3Schools — Python Conditions and If Statements](https://www.w3schools.com/python/python_conditions.asp)
- [W3Schools — Python Ternary Operator](https://www.w3schools.com/python/python_conditions_shorthand.asp)
