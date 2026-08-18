# Unit M3.2 — The Judgment Framework

*Week 14*

## Topics

### Three questions before acting on AI output

- What is the cost of this being wrong?
- Can I verify this without the AI?
- Who is accountable if this fails?

### Acceptable error

- Defining the tolerable failure threshold

### High-stakes domains

- Medical
- Legal
- Safety — where AI must not be the final word

## Labs

**Lab M3.1a: Judgment function** — Write `judgment(cost_of_wrong, can_verify, accountable)` returning Automate / Needs Oversight / Human Only.

**Lab M3.1b: Component list** — Store your domain system's components as a list of dicts (`name`, `cost_of_wrong`, `can_verify`, `accountable`).

**Lab M3.1c: Apply judgment** — Loop through components, call `judgment()` for each, and print the verdict.

**Lab M3.1d: Override detector** — Count Human Only components; warn if more than two — reconsider scope.
