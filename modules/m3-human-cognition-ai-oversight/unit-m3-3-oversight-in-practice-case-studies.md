# Unit M3.3 — Oversight in Practice: Case Studies

*Week 15*

## Topics

### Case study 1 — college admissions

- Where the human override point was missed

### Case study 2 — automated medical triage

- What failure mode was tolerated that should not have been

### Case study 3 — AI loan approval at scale

- Who was accountable and why it matters

### Automation complacency

- Accurate systems making humans less vigilant

### Human-in-the-loop checkpoints

- When to require sign-off vs allow autonomous action

### Connecting oversight to your domain

- Which components need a mandatory human checkpoint

## Labs

**Lab M3.2a: Case study data** — Store the three case studies as a list of dicts (`name`, `failure_point`, `oversight_missed`, `accountable_party`).

**Lab M3.2b: Post-mortem function** — Write `post_mortem(case)` printing what failed, who was accountable, and the preventing control; run on all three.

**Lab M3.2c: Checkpoint designer** — Write `checkpoint(component, error_threshold, review_frequency)` printing a human-in-the-loop plan for three components.

**Lab M3.2d: Complacency risk scorer** — Write `complacency_risk(accuracy, human_review_rate)` returning High/Medium/Low; test four scenarios.

**Lab M3.2e: Full oversight report** — Combine the above into `oversight_report(system, components, cases)` and run on your domain.
