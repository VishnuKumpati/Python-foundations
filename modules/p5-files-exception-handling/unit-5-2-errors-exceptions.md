# Errors & Exceptions

Unit 5.1 taught you to open a file, read it line by line, parse it as CSV, or load it as JSON — genuinely useful skills. But look back at every example in that chapter and notice a shared, unstated assumption: the file was always exactly where you expected it, and every value inside it was always exactly the type you expected. Real files, and real users, rarely cooperate that reliably. A teammate deletes `sales.csv` an hour before your script runs. A customer at a shop counter types "five hundred" into a payment app instead of `500`. A railway booking system is asked to look up a PNR that was never actually booked. The moment your code depends on something outside its own control — a file that might not exist, a user who might type garbage, a dictionary key nobody ever added — your program is one bad input away from crashing outright with a raw traceback, unless you plan for that failure deliberately.

That planning is exactly what this chapter equips you to do. You will learn to tell apart two very different kinds of "something went wrong" — a mistake in the code itself versus a problem that only surfaces while otherwise-valid code is running — and then meet the four keywords Python gives you to detect a problem, respond to it on your own terms, and keep the program alive afterward: `try`, `except`, `else`, and `finally`. You will see how to handle more than one kind of failure from the same risky block, meet the handful of built-in exceptions you will run into constantly for the rest of your career, and learn to `raise` a problem yourself — including one Python has no built-in name for — with a custom exception class.

By the time you finish, the very `open()` calls from Unit 5.1 will survive a missing file instead of crashing on it, and every risky line you write from here on will fail on your terms, not by accident.

---

## Two kinds of "something went wrong"

Picture a shop's card-payment terminal. A customer taps their card, and the terminal's network connection drops for half a second. What should happen next? The worst possible outcome is that the entire terminal freezes and needs a hard restart, locking out every customer waiting in line behind that one unlucky tap. The right outcome is that the terminal notices this one specific problem, shows a clear message such as "Please try again," and is completely ready for the very next customer a second later. That difference — crash everything, versus notice the problem and keep going — is what this whole chapter is about, and it starts with a distinction Python draws sharply between two very different kinds of failure.

The word **error** is the general umbrella term for anything that stops your program from doing what it's supposed to do. Underneath that umbrella, Python splits errors into two categories that behave completely differently, and knowing which one you're facing changes what you can actually do about it.

A **syntax error** happens *before* your program runs at all. Python reads through your code first, checking that it obeys the grammar rules of the language — a colon after an `if` line, correctly spelled keywords — and if that check fails, Python refuses to run even the very first line. You already met this back in Units 1.1–1.4, every time a missing colon or a stray typo produced a `SyntaxError`.

```python
if 5 > 3
    print("It is warm.")
```

```
  File "<cell>", line 1
    if 5 > 3
           ^
SyntaxError: expected ':'
```

A **runtime exception** — usually just called an **exception** — is a completely different animal. The code itself is valid Python; it starts running successfully, but partway through, something goes wrong: a file isn't where it's expected, a number is divided by zero, a piece of text can't be turned into a number. You have already met several of these without necessarily naming the category — `ZeroDivisionError` in Unit 1.3, `ValueError` and `TypeError` in Unit 1.4, `IndexError` in Unit 3.1, `KeyError` in Unit 3.4, and `FileNotFoundError` in Unit 5.1. Every single one of those is a runtime exception.

```python
print(10 / 0)
```

```
ZeroDivisionError: division by zero
```

| | Syntax error | Runtime exception |
|---|---|---|
| When it happens | Before the program runs at all | While otherwise-valid code is executing |
| What triggers it | Broken grammar — a missing colon, a typo in a keyword | A value or situation the code meets while running — bad input, a missing file, a zero divisor |
| Can your code catch it? | No — the program never started | Yes — this is exactly what this chapter teaches |
| Where you met it before | A missing `:` or misspelled `if` in Units 1.1–1.4 | `ZeroDivisionError`, `ValueError`, `KeyError`, `FileNotFoundError`, and more |

**Only a runtime exception can be anticipated and handled in your own code — a syntax error means your program never started running, so there is nothing left to "catch."** That single fact is the reason this entire chapter is about exceptions, not syntax errors: the tools you're about to learn only ever apply once code is genuinely running.

## `try` and `except`: a safety net under the tightrope walker

Think of risky code the way you'd think of a tightrope walker. The walker still attempts the crossing — that's the whole point of the act — but a safety net is stretched underneath first, so a fall doesn't end the show. **Exception handling** is Python's version of that net: it lets you attempt something that might fail, while having a planned response ready the instant it does, so a single bad input doesn't end your entire program.

The core tool is the **`try` block**: the section of code you are deliberately attempting, because you know it *might* fail.

```python
amount_text = "499.00"

try:
    amount = float(amount_text)
    print("Amount entered: Rs.", amount)
except ValueError:
    print("Please enter a valid number.")
```

```
Amount entered: Rs. 499.0
```

`"499.00"` converts cleanly to `499.0`, so every line inside `try` succeeds, and the matching `except ValueError:` block underneath is simply skipped — it never runs at all when nothing went wrong. Before checking, predict what happens if `amount_text` were `"five hundred"` instead: `float()` cannot parse that text into a number, so it raises `ValueError` right there, Python immediately jumps to the matching `except ValueError:` block, prints "Please enter a valid number," and the program carries on to whatever comes after the whole `try`/`except` — it does not crash, and it does not re-attempt the line that failed.

**Nothing is "protected" against an exception until it is written inside a `try` block — wrapping the risky line is what activates the safety net, not merely knowing an exception is possible.** A `try` with no matching `except` for the exception that actually occurs offers no protection at all; the exception still escapes and crashes the program exactly as if there had been no `try` there.

## Catching more than one kind of failure

A single `try` block often protects more than one risky line, and different lines can fail in completely different ways. Picture a shop accepting a UPI payment: converting the typed amount to a number can fail one way, and looking up the customer's account balance can fail in an entirely unrelated way. Python lets you list several `except` clauses after one `try`, each naming its own exception type, so each kind of failure gets its own specific response.

```python
balances = {"Rohit Verma": 300, "Asha Singh": 1000, "Karan Mehta": 5000}

def process_payment(payer, amount_text):
    try:
        amount = float(amount_text)
        current_balance = balances[payer]
        print(f"{payer} wants to pay Rs.{amount}; balance is Rs.{current_balance}.")
    except ValueError:
        print("Please enter a valid number.")
    except KeyError:
        print(f"No account found for {payer}.")

process_payment("Rohit Verma", "150")
process_payment("Unknown User", "200")
process_payment("Asha Singh", "five hundred")
```

```
Rohit Verma wants to pay Rs.150.0; balance is Rs.300.
No account found for Unknown User.
Please enter a valid number.
```

`float(amount_text)` can fail with `ValueError` if the text isn't numeric; `balances[payer]` can fail with `KeyError` if `payer` was never added to the dictionary — two unrelated problems, each caught by its own clause. **Python checks `except` clauses top to bottom and runs only the first one that matches** — it never runs more than one `except` block for a single exception, and it never checks the clauses below the one that already matched.

When two or more exception types genuinely deserve an identical response, you don't need a separate clause for each — group them in a tuple instead:

```python
try:
    amount = float("five hundred")
except (ValueError, TypeError) as e:
    print("Conversion failed:", e)
```

```
Conversion failed: could not convert string to float: 'five hundred'
```

`except (ValueError, TypeError) as e:` catches either type with one shared block, and `as e` binds whichever exception object actually occurred to the local name `e`. That object is real — Python builds it the moment the exception fires, and it carries the underlying message, so you can print it, log it, or inspect it, rather than only reacting to the bare fact that *something* went wrong. Try predicting, before you check, what `e` would hold if the failure had instead come from an incompatible type operation rather than a bad conversion — whatever message Python normally shows for that exception is exactly what prints.

## `else` and `finally`: the rest of the safety net

Two more pieces complete the toolkit, and neither is optional decoration — each guarantees something the other three parts cannot. The **`else` block** runs only if the entire `try` block finished with *zero* exceptions, and it exists so "only do this once everything above succeeded" logic can be kept visibly separate from the risky attempt itself. The **`finally` block** runs unconditionally — whether `try` succeeded, an `except` fired, or even a brand-new exception occurred that nothing caught — which makes it the one place you can always rely on for cleanup or logging.

```python
def process_payment(payer, amount_text):
    try:
        amount = float(amount_text)
        current_balance = balances[payer]
    except ValueError:
        print("Please enter a valid number.")
    except KeyError:
        print(f"No account found for {payer}.")
    else:
        if amount <= current_balance:
            print(f"Payment of Rs.{amount} by {payer} approved.")
        else:
            print(f"Insufficient balance: Rs.{current_balance} available.")
    finally:
        print(f"Payment attempt for {payer} logged.")

process_payment("Rohit Verma", "150")
process_payment("Unknown User", "200")
process_payment("Karan Mehta", "99999")
```

```
Payment of Rs.150.0 by Rohit Verma approved.
Payment attempt for Rohit Verma logged.
No account found for Unknown User.
Payment attempt for Unknown User logged.
Insufficient balance: Rs.5000 available.
Payment attempt for Karan Mehta logged.
```

Trace the three calls by hand before trusting that output. For Rohit Verma, both lines inside `try` succeed, so `else` runs and approves the payment — then `finally` logs the attempt regardless. For the unknown user, `balances[payer]` raises `KeyError`, so its `except` runs instead of `else` — and `finally` still logs the attempt afterward. For Karan Mehta, both conversions succeed, `else` runs, but `99999` exceeds his balance of `5000`, so the inner `if`/`else` inside the `else` block itself prints the rejection — and once again, `finally` logs the attempt no matter which of these three outcomes actually happened.

**`finally` is the one guarantee in this entire chapter that holds no matter what — it runs after a clean success, after a caught exception, and even after an exception nothing caught, right before that exception is allowed to keep propagating.** This is exactly why it's the right place for logging "an attempt was made," closing a resource, or anything else that must happen on every single path, not just the successful one.

## Catching a parent also catches the child: the exception family tree

Every exception in Python — built-in or your own — is built from a class, exactly the concept from Unit 4.1 onward: a class defines a blueprint, and an exception object is one instance of it. Exception classes form a tree, all ultimately inheriting from a common ancestor called `BaseException`, using the same inheritance mechanism you'll formalise in Unit 4.2. `Exception` is a direct child of `BaseException`, and nearly everything you will ever catch — `ValueError`, `TypeError`, `ZeroDivisionError`, `KeyError`, `IndexError`, `FileNotFoundError` — descends from `Exception`. A relevant slice of that tree looks like this:

```
BaseException
 └── Exception
      ├── ArithmeticError
      │     └── ZeroDivisionError
      ├── LookupError
      │     ├── IndexError
      │     └── KeyError
      ├── OSError
      │     └── FileNotFoundError
      ├── ValueError
      └── TypeError
```

Why this matters in practice: **catching a parent class in an `except` clause also catches every one of its children.** `FileNotFoundError` inherits from `OSError`, so `except OSError:` would also catch a `FileNotFoundError` — the exact same "catch the parent, catch the child too" idea you'll see formally with class inheritance in Unit 4.2.

```python
try:
    with open("sales.csv") as file:
        data = file.read()
except OSError:
    print("Could not open sales.csv - check that the file exists.")
```

```
Could not open sales.csv - check that the file exists.
```

This directly revisits the exact `open()` calls from Unit 5.1, now protected: a missing file no longer crashes the whole script, it prints a clear message and moves on. This also explains why **order matters** across multiple `except` clauses: if you list a broader parent exception before a more specific child, the parent's clause matches first, top to bottom, and the child's clause never runs at all — even though it would have matched too. Before writing several related `except` clauses, ask yourself which one is the specific case and which is the broader parent, and list the specific one first.

## Six exceptions worth knowing exactly

A handful of exceptions come up constantly enough across real Python code to be worth memorising precisely what triggers each one — and you've already met every one of them by name in an earlier unit.

| Exception | Triggered by | Example | First met in |
|---|---|---|---|
| `ValueError` | A value has the *right type* but an *inappropriate value* | `int("abc")` — a string, just not a numeric one | Unit 1.4 |
| `TypeError` | An operation is applied to a value of the *wrong type entirely* | `"5" + 5` — you cannot add text and a number directly | Unit 1.4 |
| `ZeroDivisionError` | A number is divided by zero | `10 / 0` | Unit 1.3 |
| `FileNotFoundError` | `open()` is called on a file that doesn't exist at that path | `open("sales.csv")` after the file was deleted | Unit 5.1 |
| `KeyError` | A dictionary is accessed with a key that isn't in it | `balances["Unknown User"]` | Unit 3.4 |
| `IndexError` | A list or tuple is accessed with a position outside its range | `marks[10]` on a 3-item list | Unit 3.1 |

A distinction worth keeping precise, because it's easy to blur the two: **`ValueError` means right type, wrong content; `TypeError` means wrong type entirely.** `int("cat")` raises `ValueError` because a string was expected and given, but its content isn't numeric text; `len(5)` raises `TypeError` because an integer has no length concept at all — no amount of "fixing the content" of `5` would ever make `len()` work on it. Before moving on, try predicting which of the two `"5" + 5` raises — it's `TypeError`, because text and numbers can never be combined with `+` no matter what either one contains.

## Raising your own exceptions with `raise`

Sometimes the problem your code detects isn't something a built-in exception already describes. A negative payment amount, or one that exceeds a customer's balance, isn't a `ValueError` or a `KeyError` — it's a rule specific to *your* program. The **`raise`** keyword lets your own code deliberately trigger an exception the moment it detects exactly this kind of problem.

```python
raise ValueError("amount cannot be negative")
```

For a problem with no good built-in match at all, define a **custom exception**: a class that inherits from `Exception`, using precisely the class-inheritance syntax you'll formalise in Unit 4.2.

```python
class InvalidAmountError(Exception):
    pass

def validate_amount(amount):
    if amount <= 0:
        raise InvalidAmountError(f"Rs.{amount} is not a valid payment amount.")

try:
    validate_amount(-150)
except InvalidAmountError as e:
    print("Payment rejected:", e)
```

```
Payment rejected: Rs.-150 is not a valid payment amount.
```

Raising `InvalidAmountError` works exactly like raising any built-in exception, and any code that calls `validate_amount()` can catch it specifically, by name, with `except InvalidAmountError:` — exactly as it would catch a `ValueError` or `KeyError`. **Always inherit your own custom exceptions from `Exception`, not `BaseException`** — `BaseException` also covers a couple of special signals, such as the program being told to exit, that you almost never want to accidentally catch alongside your own errors.

Say out loud, in one sentence, why `InvalidAmountError` needed to be its own class at all, rather than just raising a plain `ValueError` with a descriptive message — the answer is that a custom class lets a caller catch *this specific rule violation* with its own dedicated `except` clause, distinct from every other reason a `ValueError` might occur elsewhere in the same program.

One further form is worth knowing, even though you'll use it rarely at this stage: `raise` used with no arguments at all, written bare inside an `except` block, re-raises whichever exception is currently being handled — useful when you want to log that a problem occurred but still let it propagate upward afterward. The everyday form you will reach for constantly is `raise SomeException("message")`, exactly as used above.

## The bare `except:` anti-pattern

A **bare `except:`** is an `except` clause with no exception type named at all. It matches literally anything that goes wrong inside `try` — including problems you never anticipated and never intended to hide.

```python
try:
    total = blances[payer]   # a typo: this should read "balances"
except:
    print("Something went wrong.")
```

```
Something went wrong.
```

The danger here is not stylistic — it is that a genuine bug, in this case a simple typo (`blances` instead of `balances`), actually raises a `NameError`, the same kind of exception you met back in Unit 1.2 whenever code refers to a variable that was never assigned. A bare `except:` swallows that `NameError` silently and reports it as if it were an expected, planned-for situation, exactly the same as any deliberate `KeyError`. The fix is always to name the specific exception you actually expect:

```python
try:
    total = blances[payer]   # the same typo, left in on purpose
except KeyError:
    print("No account found.")
```

```
NameError: name 'blances' is not defined
```

| | Bare `except:` | `except KeyError:` |
|---|---|---|
| What it catches | Absolutely everything, planned for or not | Only a missing dictionary key |
| A typo (`blances`) inside `try` | Silently reported as "something went wrong" | Surfaces as an uncaught `NameError` — a visible crash you can actually investigate |
| Right for | Almost nothing in real code | Any situation where you know exactly what can fail |

**Never write a bare `except:` — always name the specific exception type you actually expect, so anything else still surfaces as a visible crash you can investigate.** Catching `Exception` broadly is only a marginal improvement over this and should be reserved for genuinely last-resort logging, not everyday use.

## Exception handling in the real world

The same toolkit protects real, working software everywhere data meets the unpredictable outside world:

- **UPI and payment systems** — validate the entered amount (`ValueError` if it isn't numeric), look up the account (`KeyError` if it's unknown), and reject an over-the-limit amount with a custom exception raised via `raise` — logging every single attempt in `finally`, regardless of the outcome.
- **Banking and fintech** — a funds-transfer service rejects a non-existent destination account or an invalid amount without ever crashing the entire banking application for every other customer being served at that same moment.
- **E-commerce checkout** — reading a discount-coupon configuration file gracefully handles the exact `FileNotFoundError` from Unit 5.1's `open()` calls, instead of blocking every customer's checkout because one file happened to go missing.
- **IRCTC-style railway booking** — a lookup for a PNR or seat that doesn't exist is caught with `KeyError`, exactly as it would be for a missing account balance, so one bad lookup never takes down the whole booking page.
- **Data processing and AI/ML pipelines** — a script processing a large file of rows catches one row's specific bad value, logs it, and continues to the next row rather than crashing on the first malformed line partway through a million-row file. Building a complete version of exactly this pattern — reading a file, skipping and logging bad rows, and continuing — is what Unit 5.3 covers next.

A short list of mistakes worth watching for deliberately while this is still new:

- Writing a bare `except:` instead of naming the specific exception you expect, silently hiding genuine bugs alongside planned-for failures.
- Wrapping an entire program in one giant `try` block instead of just the risky line, making it impossible to tell which operation actually failed.
- Listing a broader parent exception (`OSError`) before a more specific child (`FileNotFoundError`) across separate `except` clauses, so the specific block never runs.
- Assuming `finally` is optional cleanup rather than a genuine guarantee — it runs on every path out of the block, not only when nothing failed.
- Reaching for a custom exception when a built-in one, such as `ValueError`, already describes the situation perfectly well.

## Try it yourself

Do this in a Colab cell before moving on. Using `already_spent_today = {"Rohit Verma": 15000, "Asha Singh": 500}` and a `DAILY_LIMIT = 20000`, write `check_daily_limit(payer, amount_text)`. Inside one `try` block: convert `amount_text` to a `float`, look up `already_spent_today[payer]`, add the two together into `total`, and — if `total` exceeds `DAILY_LIMIT` — `raise` a custom `DailyLimitExceededError` carrying a message stating how far over the limit it is. Catch `ValueError` (bad text), `KeyError` (unknown payer), and your new `DailyLimitExceededError`, each with its own message. Use `else` to print `"Amount accepted. Total spent today would be Rs.<total>."` only when nothing above failed, and `finally` to print `"Checked limit for <payer>."` on every single call. Test it with `("Rohit Verma", "6000")`, `("Asha Singh", "1000")`, `("Unknown User", "500")`, and `("Asha Singh", "five hundred")`, predicting each outcome before you run it.

---

### Key Terminology

- **Error** — the general umbrella term for anything that stops a program from doing what it's supposed to do.
- **Syntax error** — an error caught before the program runs at all, from broken Python grammar such as a missing colon.
- **Runtime exception (exception)** — a problem that arises while otherwise-valid code is executing, and — unlike a syntax error — can be anticipated and handled.
- **Exception handling** — the set of tools (`try`, `except`, `else`, `finally`, `raise`) used to detect and respond to a runtime exception instead of letting the program crash.
- **`try` block** — the section of code you deliberately attempt, knowing it might fail; nothing is protected until it is written here.
- **`except` block** — runs only if the specific exception type named after `except` was raised somewhere inside the `try` block above it.
- **Exception object** — the real object Python constructs when an exception occurs, carrying the error message; bound to a name with `as e`.
- **`else` block** — runs only if the entire `try` block finished with zero exceptions.
- **`finally` block** — runs unconditionally on every path — success, a caught exception, or even an uncaught one.
- **Exception class hierarchy** — the tree of exception classes, all inheriting from `BaseException`; catching a parent class also catches every child beneath it.
- **`BaseException` / `Exception`** — the root of every exception class; custom exceptions should inherit from `Exception`, not `BaseException`.
- **Custom exception** — a user-defined class inheriting from `Exception`, used to signal a problem specific to your own program.
- **`raise`** — the keyword that deliberately triggers an exception, built-in or custom, from your own code.
- **Bare `except:`** — an `except` clause naming no exception type, which matches literally anything, including bugs you never anticipated.
- **`ValueError`** — raised when a value has the right type but an inappropriate value, such as `int("abc")`.
- **`TypeError`** — raised when an operation is applied to a value of the wrong type entirely, such as `"5" + 5`.
- **`KeyError`** / **`IndexError`** — raised for a missing dictionary key, or a list/tuple position outside its valid range, respectively.

### Mastery Checkpoint

Before moving to Unit 5.3, check that you can answer these without looking back:

1. Why can a runtime exception be caught and handled in your own code, while a syntax error cannot?
2. A `try` block has two lines that can each fail differently. What happens if you write one `except` clause for a type not raised by either line, and the actual exception raised is a different type entirely?
3. `FileNotFoundError` inherits from `OSError`. If you write `except OSError:` before `except FileNotFoundError:` in the same `try`, which clause actually runs when a file is missing — and why does the second one become unreachable?
4. What is the one guarantee `finally` gives you that neither `except` nor `else` can, and why does that make it the right place for logging or cleanup?
5. A bare `except:` silently reports a `NameError` from a typo as "something went wrong." What should the code have written instead, and what would that change have revealed?

### Summary

You now know how to tell a syntax error, caught before your program ever runs, apart from a runtime exception, which arises during otherwise-valid code and — unlike a syntax error — can be planned for. You've used `try` and `except` as a safety net around risky code, handled several distinct failure types from one block with multiple `except` clauses or a shared tuple, and used `else` and `finally` to separate success-only logic from cleanup that must run on every path. You've seen how Python's exception classes form a family tree where catching a parent also catches its children, memorised exactly what triggers `ValueError`, `TypeError`, `ZeroDivisionError`, `FileNotFoundError`, `KeyError`, and `IndexError`, and learned to `raise` your own exceptions — including a custom class — for problems no built-in exception describes. You've also seen precisely why a bare `except:` is a real bug risk, not just a style preference. From here, the next step is putting file handling and exception handling together into one complete program — Unit 5.3, a case study building a robust file reader that survives real, messy data instead of crashing on the first bad line.

### Additional Resources

- [Python Tutorial — official docs: "Errors and Exceptions"](https://docs.python.org/3/tutorial/errors.html)
- [Python Tutorial — official docs: "Handling Exceptions"](https://docs.python.org/3/tutorial/errors.html#handling-exceptions)
- [Python Tutorial — official docs: "User-defined Exceptions"](https://docs.python.org/3/tutorial/errors.html#user-defined-exceptions)
- [Python 3 Documentation — Built-in Exceptions](https://docs.python.org/3/library/exceptions.html)
- [W3Schools — Python Try Except](https://www.w3schools.com/python/python_try_except.asp)
