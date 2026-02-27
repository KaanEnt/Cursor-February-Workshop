# Failure Diagnosis Framework

## The Diagnostic Sequence

When an eval run produces low scores, diagnose failures in this order:

### 1. Is the Test Case Valid?

Before blaming the prompt, verify the test case itself:
- Is the `expected` output actually correct?
- Is the input realistic, or is it an impossible request?
- Are the scoring criteria appropriate for this input?

**Fix**: Adjust the test case if it's unfair. Don't tune prompts to pass bad tests.

### 2. Is the Criterion Calibrated?

Check if the scoring mechanism is producing accurate scores:
- For `regex`: Does the pattern actually match what you think it matches? Test it on known good/bad outputs.
- For `llm_judge`: Is the judge prompt clear and unambiguous? Run the judge on 3 manually-scored outputs to verify alignment.
- For `rubric`: Are the score levels (0.0, 0.4, 0.7, 1.0) well-defined with clear boundaries?

**Fix**: Recalibrate the criterion before iterating the prompt.

### 3. Identify the Failure Pattern

Categorize each failure into one of these patterns:

#### Instruction Amnesia
**Symptom**: The prompt has a clear instruction, but the output ignores it.
**Cause**: The instruction is buried in a long prompt, or conflicting instructions override it.
**Fix**: Move the instruction to the end of the prompt (recency bias) or repeat it in a "Remember:" section. Alternatively, shorten the prompt — cut what isn't earning its weight.

#### Ambiguity
**Symptom**: The output partially follows the instruction but interprets it differently than intended.
**Cause**: The instruction uses vague language ("be concise", "be professional", "use a good tone").
**Fix**: Replace vague instructions with concrete constraints: "Under 15 words", "No sentences starting with 'We'", "Use the word swap list in section 3".

#### Missing Negative Example
**Symptom**: The output does something the prompt didn't explicitly forbid.
**Cause**: The prompt tells the model what to do but not what to avoid.
**Fix**: Add a "Do NOT" section with concrete bad examples. Models learn more from negative examples than positive instructions for constraint enforcement.

#### Conflicting Instructions
**Symptom**: The output oscillates between two styles or partially follows two contradictory rules.
**Cause**: Two parts of the prompt give incompatible guidance (e.g., "be creative and surprising" + "follow the exact template").
**Fix**: Resolve the conflict by adding priority ("when in doubt, prefer X over Y") or scoping each instruction to specific parts of the output.

#### Insufficient Context
**Symptom**: The output is generic or misses domain-specific nuances.
**Cause**: The prompt doesn't provide enough background, examples, or domain knowledge.
**Fix**: Add 2-3 few-shot examples that demonstrate the desired output style. Examples are the single most effective way to improve prompt quality.

#### Format Drift
**Symptom**: The output starts in the right format but drifts partway through.
**Cause**: Long outputs lose format discipline. The model "forgets" the format constraint.
**Fix**: Add format reminders mid-prompt, or break the task into smaller steps with explicit format requirements at each step.

## Diagnostic Decision Tree

```
Output is wrong
├── Is the test case fair?
│   ├── No → Fix the test case
│   └── Yes ↓
├── Is the criterion calibrated?
│   ├── No → Recalibrate the criterion
│   └── Yes ↓
├── Does the prompt address this at all?
│   ├── No → Instruction is MISSING → Add it
│   └── Yes ↓
├── Is the instruction clear and specific?
│   ├── No → Instruction is AMBIGUOUS → Make concrete
│   └── Yes ↓
├── Is there a conflicting instruction?
│   ├── Yes → CONFLICT → Resolve with priority rules
│   └── No ↓
├── Are there examples of the desired behavior?
│   ├── No → INSUFFICIENT CONTEXT → Add few-shot examples
│   └── Yes ↓
└── Is the output partially right?
    ├── Yes → FORMAT DRIFT or AMNESIA → Repeat/reinforce constraint
    └── No → Major architecture issue → Consider prompt redesign
```

## Severity Classification

| Severity | Description | Action |
|----------|-------------|--------|
| **Critical** | Output is factually wrong, harmful, or completely off-task | Block shipping. Fix immediately. |
| **High** | Output violates a primary criterion (framework structure, core constraint) | Must fix before shipping. |
| **Medium** | Output partially follows instructions but has quality gaps | Fix if possible within 1-2 iterations. |
| **Low** | Stylistic imperfection, minor format issue | Note for future improvement. Don't block shipping. |
