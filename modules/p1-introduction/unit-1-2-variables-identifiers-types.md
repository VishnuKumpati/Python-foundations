# Variables, Identifiers & Types

---

## What is a Variable?

A **variable** is a name that holds a value, so you can reuse it or change it later, instead of retyping the value everywhere.

You create one with `=`:

```python
x = 5
```

Read this as "`x` now refers to `5`," not "x equals 5" like in maths. In Python, `=` is an action, not a fact.

Open a new cell in your Colab notebook from Unit 1.1, and try each example below as you go.

**Quick glossary:**

| Term | Meaning |
|---|---|
| **Variable** | A name that refers to a value |
| **Assignment (`=`)** | Binds a name to a value |
| **Reassignment** | Giving an existing variable a new value |
| **Identifier** | The name you choose for a variable |
| **Type** | What kind of value it is: number, text, true/false |
| **Dynamic typing** | Python figures out the type for you, automatically |

---

## Why Variables Matter

Without them, every program can only `print()` fixed text once and forget it. Real apps need to:

- **Remember** a value (a cart total while you keep shopping)
- **Reuse** a value (a tax rate applied to every item)
- **Update** a value (a bank balance after a transaction)

One variable, one name, handles all three.

---

## Reassignment

The same name can point to a new value later. The old one is just gone:

```python
score = 100
print(score)

score = 150
print(score)
```

Output:

```
100
150
```

---

## Naming Rules

- Letters, digits, and `_` only
- Can't start with a digit: `age2` is fine, `2age` is not
- No spaces or symbols like `-`, `!`, `$`
- Can't use a **keyword** (also called a reserved word)

**What's a keyword?** It's a word Python has already reserved for its own use, things like `if`, `for`, `class`, `True`, `and`, `return`. Python uses these words to understand the *structure* of your code, so it won't let you reuse them as a variable name. There'd be no way to tell "the `if` that means a decision" apart from "a variable I happened to call `if`."

```python
class = "10A"   # SyntaxError: class is a keyword, not available as a name
class_name = "10A"   # fine, just add a word to make it unique
```

You don't need to memorize the full list. Python (and your code editor) will tell you immediately with a `SyntaxError` if you accidentally use one.

**Convention (not a rule, just good practice):** use `snake_case`, like `total_price`, not `TotalPrice`.

And Python is **case sensitive**: `total` and `Total` are two different variables. This trips people up constantly.

---

## The Four Basic Types

```python
customer_name = "Ananya Roy"   # str: text, in quotes
delivery_fee  = 29.50          # float: has a decimal point
order_id      = "SWG10234"     # str: quoted, even though it has digits
is_paid       = False          # bool: True or False only
```

`type()` is a built-in function. Give it any value or variable, and it tells you what kind of data it is. You'll use this constantly whenever you're not 100% sure what you're working with.

```python
print(type(customer_name))
print(type(delivery_fee))
print(type(order_id))
print(type(is_paid))
```

Output:

```
<class 'str'>
<class 'float'>
<class 'str'>
<class 'bool'>
```

Read `<class 'str'>` as simply "this is a `str`." The `class` wording is just how Python labels its built-in types internally; it doesn't mean anything special for you at this stage.

---

## Getting Input from the User

`input()` pauses your program, shows a message, and waits for the person to type something and press Enter:

```python
name = input("Enter your name: ")
print("Hello,", name)
```

```
Enter your name: Ada
Hello, Ada
```

**The one thing to remember:** `input()` always returns a **string**, even if the user types a number.

```python
age = input("Enter your age: ")
print(type(age))
```

```
Enter your age: 20
<class 'str'>
```

Try `age + 5` right now and Python throws a `TypeError`: you can't add a number to text.

The fix is to convert it first, using `int()`:

```python
age = input("Enter your age: ")   # "20", a string
age = int(age)                     # now 20, an actual number
print(age + 5)
```

```
Enter your age: 20
25
```

`int(age)` doesn't change what `age` was. It creates a new number value from it, which you then store back into `age`. This same idea, converting text into a real number before using it, comes up constantly, and gets its own proper unit later (1.4).

---

## Static vs Dynamic Typing

| | Static (Java, C) | Dynamic (Python) |
|---|---|---|
| Declare type up front? | Yes: `int age = 30;` | No, Python figures it out |
| When is type checked? | Before running | While running |
| Can a variable change type? | No | Yes, same name, different type later |

```python
data = 100
print(type(data))   # int

data = "one hundred"
print(type(data))   # str
```

Same name, different type. That's because the type belongs to the **value**, not the name.

---

## Different Ways to Print

You've already seen `print()` take one string. Now that you've got several variables of different types sitting around, here's how to print them together cleanly.

**Comma-separated:** `print()` joins them with a single space automatically.

```python
print("Name:", customer_name, "Fee:", delivery_fee)
```
```
Name: Ananya Roy Fee: 29.5
```

**Controlling the ending with `end`:** by default, `print()` adds a new line after each call. Change that with `end`.

```python
print("Loading", end="")
print("...")
```
```
Loading...
```

**Controlling the separator with `sep`:** changes what goes *between* comma-separated values, instead of the default single space.

```python
print("2026", "07", "28", sep="-")
```
```
2026-07-28
```

---

## Try it Yourself

**(a)** Create `restaurant_name = "Spice Route"`, print it, then print its type.

```python
restaurant_name = "Spice Route"
print(restaurant_name)
print(type(restaurant_name))
```
```
Spice Route
<class 'str'>
```

**(b)** Now add `quantity = 3` and `packing_charge = 15.0`. Print all three types.

```python
print(type(restaurant_name))
print(type(quantity))
print(type(packing_charge))
```
```
<class 'str'>
<class 'int'>
<class 'float'>
```

**Your turn:** using `input()`, ask for the customer's name, print `"Order for:"` followed by it. Then create `order_confirmed = False`, reassign it to `True`, and print the result.

---

## Common Mistakes

- Using a variable before assigning it: `NameError: name '...' is not defined`
- Starting a name with a digit (`2age = 30`): `SyntaxError`
- Naming something after a reserved word (`class = "10A"`): `SyntaxError`
- Assuming `total` and `Total` are the same variable. They're not.
- Confusing `"5"` (a string) with `5` (a number). Check with `type()` if unsure.

---

## Interview Questions

**Q1: Is Python statically or dynamically typed?**

A: Dynamically typed. You never declare a type up front. Python figures it out from the value, and the same name can hold different types at different points.

**Q2: What's the difference between a rule and a convention here?**

A: Identifier rules (no leading digit, no reserved words) are enforced: break them and your code won't run. `snake_case` is a convention, a team agreement for readability, not something Python checks.

**Q3: Is `=` the same as `==`?**

A: No. `=` assigns a value. `==` compares two values (covered in the next unit).

---

## Quick Recap

- A variable is a name bound to a value with `=`; reassignment replaces the old value.
- Identifiers follow strict rules: no leading digit, no keywords (reserved words like `if`, `class`). `snake_case` is the convention.
- Python is case sensitive.
- The four basic types: `int`, `float`, `str`, `bool`. Use `type()` to find out what type any value is.
- `input()` always returns a string, no matter what's typed.
- Python is dynamically typed: the type belongs to the value, not the name.
- `print()` can take multiple values (comma-separated) and control spacing with `sep` and `end`.
