# Unit M2.4 — Ethics, Safety & Governance

*Week 13*

## Topics

### Real failure cases

- Healthcare misdiagnosis
- Hiring bias
- Deepfake harm

### Data bias

- Biased training data produces biased output

### The four pillars

- Fairness
- Transparency
- Accountability
- Harm prevention

### Red-teaming

- Systematically breaking your own system before deploying

### Prompt injection

- How attackers manipulate AI through crafted inputs

### Governance frameworks

- EU AI Act
- NIST AI RMF — Govern, Map, Measure, Manage
- White House Executive Order on AI (2023)
- India AI governance

### Engineering takeaway

- Knowing which framework applies is part of the job, not just an ethics discussion

## Labs

**Lab M2.4a: Risk keyword checker** — Write `risk_level(use_case)` returning High/Medium/Low based on domain keywords.

**Lab M2.4b: Governance matcher** — Store frameworks and focus areas in a dict; given a domain, print the most relevant framework.

**Lab M2.4c: Accountability checker** — Write `who_is_accountable(use_case, is_human_in_loop)` returning Developer / Deployer / User.

**Lab M2.4d: Ethics report generator** — Combine the above into `ethics_report(use_case)` printing risk level, framework, and accountable party; run on three real systems.
