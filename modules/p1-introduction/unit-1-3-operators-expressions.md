# 1.3 Operators & Expressions

---

[← Previous: 1.2 Variables, Identifiers & Types](unit-1-2-variables-identifiers-types.md) | [Go back to TOC](../../README.md) | [Next: 1.4 Statements, Conversion & Output →](unit-1-4-statements-conversion-output.md)


---

## What is an Operator?

An **operator** is a symbol (or keyword) that tells Python to do something with values — add them, compare them, combine them.

An **expression** is any combination of values and operators that reduces to one result.

```python
total = 200 + 50
```

`200 + 50` is the expression, `+` is the operator, `200` and `50` are the **operands**. Python works out `250` first, *then* stores it in `total`.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Operator** | A symbol/keyword that acts on values, e.g. `+`, `==`, `and` |
| **Operand** | A value the operator works on |
| **Expression** | Values + operators that reduce to one result |
| **Truthy / Falsy** | How Python treats any value as true-ish or false-ish |
| **Short-circuit** | Python stops checking once the answer is already known |

---

## Arithmetic Operators

| Operator | Name | Example | Result |
|---|---|---|---|
| `+` | addition | `7 + 3` | `10` |
| `-` | subtraction | `7 - 3` | `4` |
| `*` | multiplication | `7 * 3` | `21` |
| `/` | true division | `7 / 2` | `3.5` |
| `//` | floor division | `7 // 2` | `3` |
| `%` | modulo (remainder) | `7 % 2` | `1` |
| `**` | exponentiation | `7 ** 2` | `49` |

**Two things trip people up:**

- `/` **always** returns a `float`, even `6 / 2` → `3.0`, not `3`.
- `//` rounds *down*, not toward zero. So `-7 // 2` is `-4`, not `-3`.

**With negative numbers, `%` also behaves differently than you'd expect** — it takes the sign of the *divisor*, not the number being divided:

```python
print(-7 % 2)   # 1, not -1
```

`//` and `%` are actually a pair — this always holds true:

```
(a // b) * b + (a % b) == a
```

Check it: `(-7 // 2) * 2 + (-7 % 2)` = `(-4 * 2) + 1` = `-8 + 1` = `-7` ✅

---

## Comparison Operators

Always produce a `bool` — `True` or `False`.

| Operator | Name | Example | Result |
|---|---|---|---|
| `==` | equal to | `5 == 5` | `True` |
| `!=` | not equal to | `5 != 3` | `True` |
| `<` | less than | `3 < 5` | `True` |
| `>` | greater than | `3 > 5` | `False` |
| `<=` | less or equal | `5 <= 5` | `True` |
| `>=` | greater or equal | `3 >= 5` | `False` |

**`=` vs `==` — the #1 beginner mistake:**

| | `=` | `==` |
|---|---|---|
| Does what | Stores a value | Asks a question |
| Result | Nothing — it's an action | Always `True` or `False` |
| Example | `x = 5` | `x == 5` |

You can also chain comparisons: `1 < x < 10` reads exactly like "is x between 1 and 10?"

**One quiet gotcha:** comparing different types is legal, but always `False` — `5 == "5"` is `False`, even though they look the same. This bites people right after Unit 1.2, since `input()` always returns a string.

---

## Logical Operators

Combine or flip `bool` values.

| Operator | Meaning | Example | Result |
|---|---|---|---|
| `and` | true only if both sides are true | `True and False` | `False` |
| `or` | true if at least one side is true | `True or False` | `True` |
| `not` | flips a value | `not True` | `False` |

**Short-circuit evaluation:** Python stops the instant it already knows the answer.

- `False and anything` → skips the right side, already `False`
- `True or anything` → skips the right side, already `True`

This matters: if the right side would crash (like a division by zero), it might never even run.

---

## Truthiness

Python treats every value as true-ish or false-ish, not just actual `True`/`False`.

**Falsy** (short, fixed list): `False`, `0`, `0.0`, `""` (empty string)

**Truthy:** everything else — including the string `"False"` (it's non-empty text, so it's truthy).

```python
coupon_code = ""
has_coupon = bool(coupon_code)
print(has_coupon)   # False — empty string is falsy
```

---

## Operator Precedence

When an expression has more than one operator, Python follows a fixed order — it doesn't just read left to right.

**Highest to lowest:**

```
1. ()              parentheses
2. **               exponentiation
3. -x               unary minus
4. * / // %         multiply/divide family
5. + -              add/subtract
6. == != < > <= >=  comparisons
7. not              logical not
8. and              logical and
9. or               logical or
```

This means:

- `-2 ** 2` is `-4`, not `4` — `**` binds tighter than the minus sign
- `2 + 3 > 4 and 1 < 2` reads as `((2 + 3) > 4) and (1 < 2)`
- Same-level operators go left to right: `20 / 4 * 2` is `(20 / 4) * 2 = 10.0`

**Don't memorize the ladder.** If the order isn't obvious, just add parentheses — they always win and never make an expression wrong.

---

## Try it Yourself

A snack stall order: 4 samosas at ₹15, 2 cold drinks at ₹20, plus a ₹10 packing fee, split between 3 friends.

```python
samosa_price = 15
samosa_qty = 4
cold_drink_price = 20
cold_drink_qty = 2
packing_fee = 10
friends_sharing = 3

item_total = samosa_price * samosa_qty + cold_drink_price * cold_drink_qty
final_bill = item_total + packing_fee
share_per_friend = final_bill // friends_sharing
leftover_rupees = final_bill % friends_sharing

print(final_bill)
print(share_per_friend)
print(leftover_rupees)
```

Output:

```
110
36
2
```

`item_total` works out because `*` runs before `+` — no parentheses needed. `//` splits the bill into whole rupees each; `%` gives what's left over. Check: `(36 × 3) + 2 = 110` ✅

**Your turn:** now add a discount check. The order qualifies for a discount if the bill is at least ₹100, **and** the customer has a student card, **and** the order isn't cancelled:

```python
has_student_card = True
order_cancelled = False

qualifies_by_amount = final_bill >= 100
discount_eligible = qualifies_by_amount and has_student_card and not order_cancelled
print(discount_eligible)
```

Try changing `has_student_card` to `False` and see what happens — and why.

---

## Common Mistakes

- Confusing `=` with `==` — the single most common bug in this unit
- Expecting `/` to behave like `//` — `/` always gives a float
- Misreading `-2 ** 2` as `4` instead of `-4`
- Assuming Python reads strictly left to right, ignoring precedence
- Treating the string `"False"` as falsy — it isn't, only the actual `False` value is
- Forgetting that short-circuit evaluation can skip code entirely on the right side of `and`/`or`

---

## Interview Questions

**Q: What's the difference between `/` and `//`?**
A: `/` is true division — always returns a `float`. `//` is floor division — rounds down toward negative infinity, and returns an `int` if both operands are `int`.

**Q: What does `-7 % 2` give you?**
A: `1`, not `-1` — modulo takes the sign of the divisor, not the number being divided. It pairs with `//`: `(a // b) * b + (a % b)` always reconstructs `a`.

**Q: What is short-circuit evaluation?**
A: Python stops evaluating `and`/`or` the moment the final result is already known — it won't bother checking the right side if the left side already decides the answer.

**Q: What values are falsy in Python?**
A: A short fixed list — `False`, `0`, `0.0`, and `""`. Everything else is truthy.

**Q: Is `=` the same as `==`?**
A: No. `=` assigns a value. `==` compares two values and returns `True` or `False`.

---

## Quick Recap

- An operator acts on operands; an expression always reduces to one value.
- Arithmetic: `+ - * / // % **` — remember `/` gives a float, `//` rounds down.
- Comparisons (`==`, `!=`, `<`, `>`, `<=`, `>=`) always return a `bool`.
- `=` assigns, `==` compares — don't mix them up.
- `and`, `or`, `not` combine conditions, with short-circuit evaluation.
- Truthiness: `False`, `0`, `0.0`, `""` are falsy — everything else is truthy.
- Precedence has a fixed order; parentheses always override it.


## Reference Links

- [Python 3 Documentation — Expressions](https://docs.python.org/3/reference/expressions.html)
- [Python 3 Documentation — Built-in Types (Truth Value Testing)](https://docs.python.org/3/library/stdtypes.html#truth-value-testing)
- [Real Python — Operators and Expressions in Python](https://realpython.com/python-operators-expressions/)
- [W3Schools — Python Operators](https://www.w3schools.com/python/python_operators.asp)

[← Previous: 1.2 Variables, Identifiers & Types](unit-1-2-variables-identifiers-types.md) | [Go back to TOC](../../README.md) | [Next up: **1.4 Statements, Conversion & Output** — turning these values and conditions into clean, readable results. →](unit-1-4-statements-conversion-output.md)

---

*© 2026 Revature · AI Native Engineering — Foundations · Unit 1.3 · Version 2.0*
