# Unit M2.3 — How LLMs Behave

*Week 12*

## Topics

### Why the same question gives different answers

- Temperature and sampling

### Temperature

- Low picks the most likely token
- High introduces variation

### Probability distributions

- Output as a distribution over tokens (intuition, no maths)

### Hallucination

- Why AI states falsehoods confidently

### Context and prompting basics

- Context windows (intro)
- System vs user prompt (intro)

## Labs

**Lab M2.3a: Temperature results store** — Build `temp0_results` and `temp1_results` (10 responses each) and print both lists.

**Lab M2.3b: Unique response counter** — Write `count_unique(results)` and compare the two temperature lists.

**Lab M2.3c: Temperature accuracy check** — Count matches against a known answer and print accuracy for temp0 vs temp1.

**Lab M2.3d: Variance analyser** — Write `response_variance(results)` and interpret which temperature is more consistent.
