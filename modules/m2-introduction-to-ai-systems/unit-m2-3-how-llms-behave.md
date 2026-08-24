# How LLMs Behave

---

It is nine days before her first-year exams, and Maya is revising with the study assistant her college gives every student. She types a question, reads the answer, and types the same question again a minute later to check she copied it down correctly.

The second answer is different. Not wrong — different. Same question, same assistant, same minute, different words.

Most students decide something is broken. Nothing is broken. That difference, and four other behaviours that surprise people, all come from the same mechanism, and once you can predict them you stop being caught out by them.

## Why the Same Question Gives Different Answers

Maya types: **Explain recursion in one line.**

The first answer:

```
Recursion is when a function calls itself to solve a smaller version of the same problem.
```

She asks again:

```
Recursion is a technique where a function solves a problem by calling itself on a smaller input.
```

Both are correct. Neither was stored anywhere. A language model builds each answer one token at a time, and at every token it has a *choice* — several words could reasonably come next. Choosing one of them is called **sampling**, and sampling is why two runs diverge.

The choice is not random in the everyday sense. The model is not throwing dice over the whole dictionary. It works from a ranked list, and understanding that list explains everything else in this file.

## Probability Distributions

Suppose the answer has begun, and the text so far reads *Recursion is when a function calls*. Before writing the next token, the model produces a **probability distribution**: every possible next token with a number saying how likely it is here.

Written out, that list looks roughly like this. The exact figures vary by model — what matters is the shape.

| Candidate next token | Roughly how likely |
|---|---|
| itself | very high |
| another | low |
| back | low |
| upon | very low |
| pizza | vanishingly small |

Three things follow from that shape, and they explain nearly every behaviour students find strange.

**The model always has options.** Even when one candidate dominates, others carry some probability. Nothing forces the top one to be chosen every time.

**Nonsense is not impossible, only unlikely.** *Pizza* has a tiny share rather than a zero share, which is why a model occasionally produces something absurd.

**One different token changes everything after it.** If *another* is chosen instead of *itself*, every token that follows is now predicting a continuation of a different sentence. The two answers Maya received did not differ by one word — they parted company at one token and never rejoined.

What decides how boldly the model picks from this list is a single setting.

## Temperature

**Temperature** is a dial that controls how much the model favours the top of the list over the rest of it.

**Low temperature** flattens the excitement and picks the most likely token. At a temperature of zero the choice becomes deterministic: the highest-probability candidate wins every time, so the same question produces the same answer.

**High temperature** gives the lower-ranked candidates a real chance. Answers become more varied and more surprising, and lower-quality ones become more likely too.

The practical guidance is short: lower temperatures suit tasks with a right answer, and higher temperatures suit tasks where you want range.

| What Maya is doing | Temperature to prefer | Why |
|---|---|---|
| Asking for a definition to memorise | Low | She wants the same reliable wording each time |
| Checking a formula before an exam | Low | Variation here is a risk, not a feature |
| Generating ten project title ideas | Higher | Sameness is the failure she is trying to avoid |
| Rewriting her lab report introduction three ways | Higher | She is choosing between options, not seeking truth |

### Before the Numbers

Maya runs a test. She asks the same definition question ten times at a low setting and ten times at a high one, and counts how many distinct answers come back.

Predict the result before reading on. Low temperature always takes the top candidate, so the answers should barely move; high temperature reaches into the rest of the list, so they should scatter.

```
Low temperature  → 10 responses, 1 distinct wording
High temperature → 10 responses, 7 distinct wordings
```

Was that what you expected? Ten identical answers at the low setting is not the model being lazy. It is the model taking the same top candidate at every token, which is exactly what a definition needs and exactly what brainstorming does not.

One caution worth carrying: with recent models, the default setting is usually the tested one, and pushing the dial away from it can make output worse rather than more creative — including strange failures such as looping. Change it deliberately, for a reason, and check the result.

Temperature explains why answers vary. It does not explain why a confident answer is sometimes untrue.

## Hallucination

Maya needs one citation for her lab report. She asks for a source on damping in simple pendulums, and receives:

```
Kumar, R. & Silva, M. (2019). Damping effects in simple pendulum systems.
Journal of Applied Mechanics, 41(3), pp. 44-57.
```

Author names, a plausible journal, a volume, an issue, a page range. The paper does not exist.

This is a **hallucination**: output that is fluent, confident, and false. It is the single most important behaviour in this file, and its cause is already on the page above.

The model was not looking anything up. Asked for a citation, it produced the tokens that most likely follow a request for a citation — and what follows is a surname, a year in brackets, a title in sentence case, a journal name, page numbers. Every token was a good prediction. The result is a perfectly shaped citation with nothing behind it.

Think of an exam answer written by someone who did not read the chapter but has read many answers to questions like it. They know the shape of a good answer: define the term, give an example, state the limitation. What comes out reads like knowledge and contains none.

Two properties make hallucination dangerous rather than annoying.

**It arrives in the same tone as the truth.** There is no wobble, no hedge, and no signal that this part of the answer is weaker than the rest. Confidence carries no information about accuracy.

**It is most likely exactly where you can check least.** Specific citations, precise statistics, section numbers, quotations, and dates are all cases where a plausible pattern is easy to produce and the truth is hard for you to verify quickly.

Lowering the temperature reduces the variety of hallucinations. It does not remove them, because the most probable answer can be false. What reduces them is giving the model the material to answer from and asking it to answer only from that — which brings up the question of how much material it can hold at once.

## Context Windows

Everything the model can see while producing one answer has to fit into a single space called the **context window**, measured in tokens. Your instructions, any documents you paste, the conversation so far, and the reply being written all share it.

Modern windows are large. A big one today holds something on the order of eight full-length novels. It is still a ceiling, and two consequences follow for Maya.

**Her 40 pages of notes may fit, but that does not make pasting them wise.** Every token costs money and time, and a long prompt takes longer to come back. Sending the three relevant pages beats sending forty.

**Long conversations lose their beginning.** When a chat runs past the window, the earliest turns fall out of view. If Maya said "I am a first-year student, explain simply" an hour ago and the explanations have quietly grown harder since, nothing malfunctioned — that instruction is no longer in the window.

The fix for the second problem is to put standing instructions somewhere they cannot scroll away.

## System and User Prompts

Two different kinds of instruction reach the model, and telling them apart explains a great deal about how AI products behave.

The **system prompt** is set by whoever built the assistant. It defines the role, the rules, and the format, it applies to every message, and the student never sees it. The college's study assistant carries something like this:

```
You are a study tutor for first-year students.
Explain concepts step by step in plain language.
Answer only from the uploaded course notes.
Never provide final answers to graded assignments; guide the student instead.
```

The **user prompt** is what Maya types: the specific question, and any material she attaches for this one task.

That division is why the assistant explains recursion patiently but refuses to hand over her graded assignment answer. She is not being singled out. A rule written by her college sits in front of every message she sends, and no amount of rephrasing on her side changes it.

Two practical points come out of this.

Standing rules belong in the system prompt, where they cannot scroll out of the window. One-off requests belong in the user prompt.

When a large document and a question are sent together, put the document first and the question last. Instructions land better at the end, after the material they refer to.

## Best Practices

- Use a low temperature for anything with a correct answer, and a higher one only when variety is the point
- Leave the temperature at its default unless you have a specific reason, and check the output after changing it
- Treat every specific citation, statistic, date, and quotation as unverified until you have seen the source yourself
- Give the model the material to answer from, and instruct it to use only that material
- Send the pages that matter rather than the whole folder, for cost, speed, and accuracy alike
- Put standing rules in the system prompt and one-off requests in the user prompt
- Place long documents before the question, not after it
- Ask the same important question twice. Two different answers tell you the model is unsure, which is information worth having

## Common Beginner Mistakes

- Deciding the AI is broken because two answers to one question differ
- Raising the temperature to make answers "better", and getting worse answers with more variety
- Copying a citation, statistic, or page number into submitted work without opening the source
- Reading confidence as accuracy, when the two are unrelated
- Pasting an entire folder into the prompt and paying for tokens that do not help the answer
- Expecting a long conversation to remember an instruction given far above the current message
- Trying to argue an assistant out of a restriction that was written into its system prompt

## Key Takeaways

- A model chooses each token from a ranked list of candidates. Choosing among them is **sampling**, and it is why the same question can produce different answers
- That ranked list is a **probability distribution** — every possible next token with a likelihood attached. Unlikely tokens are rare rather than impossible
- A single different token changes every token after it, which is why two answers diverge completely rather than by one word
- **Temperature** controls how much the model favours the top of the list. Low means deterministic and repeatable; zero always takes the most likely token; high means varied and riskier
- Use low for definitions, formulas, and facts, and higher for brainstorming and rewriting — and leave the default alone unless you have a reason
- A **hallucination** is fluent, confident, false output, produced because a well-shaped answer and a true answer are different things
- Hallucinations appear most often where they are hardest to check: citations, statistics, dates, and quotations
- The **context window** is the total space, measured in tokens, for instructions, documents, conversation, and reply — so long chats lose their beginning and long prompts cost more
- The **system prompt** sets the rules for every message and is invisible to the user; the **user prompt** is the individual request

> **Interview tip:** Asked why an AI gave two different answers to one question, do not say "it is random". Walk through the mechanism: the model produces a distribution over next tokens, temperature decides how strictly it takes the top of that distribution, and one different token changes everything after it. Then connect it to hallucination — the most probable text and the true text are not the same thing — and say which temperature you would use for a factual lookup versus a brainstorm. That chain of reasoning is what interviewers are listening for.

## Reference Links

- 📎 [Google for Developers: What is a Large Language Model?](https://developers.google.com/machine-learning/crash-course/llm/transformers)
- 📎 [Google AI for Developers: Prompting strategies and temperature](https://ai.google.dev/gemini-api/docs/prompting-strategies)
- 📎 [Google AI for Developers: Long context and context windows](https://ai.google.dev/gemini-api/docs/long-context)
- 📎 [AWS: What is Generative AI?](https://aws.amazon.com/what-is/generative-ai/)
