# Prompt Iteration Patterns

## The One-Change Rule

Change exactly one thing between prompt versions. If you change multiple things, you can't attribute score improvements (or regressions) to a specific change.

**Exception**: If the prompt needs a complete restructure (score < 0.50), a rewrite is justified. But this should be v2, not v1.1 — it's a new architecture.

## Iteration Strategies (Ordered by Effectiveness)

### 1. Add Few-Shot Examples

**When**: Output quality is inconsistent — sometimes right, sometimes wrong.
**How**: Add 2-3 input/output pairs that demonstrate the exact behavior you want.
**Why it works**: Examples are unambiguous. They show rather than tell.

```markdown
## Examples

Input: "Write a headline for a password manager"
Output: "Never forget a password again — or worry about getting hacked"

Input: "Write a headline for a time tracking app"  
Output: "Know exactly where your hours go. Reclaim the ones you're wasting."
```

**Effectiveness**: High. Few-shot examples typically improve scores by 0.10-0.25 on the first iteration.

### 2. Add Negative Examples

**When**: The output keeps doing something you don't want.
**How**: Show what NOT to do, with an explanation of why it's wrong.

```markdown
## Don't Do This

❌ "Leverage our cutting-edge AI-powered solution to optimize your workflow"
Why it's bad: Jargon-heavy, product-centric, no emotional hook.

✅ "Spend less time on busywork. Let AI handle the boring stuff."
Why it's good: Customer-centric, concrete, passes the BBQ test.
```

**Effectiveness**: High for constraint enforcement. Reduces jargon/format violations by 50-80%.

### 3. Restructure Information Hierarchy

**When**: The model follows some instructions but ignores others (amnesia).
**How**: Put the most critical instructions at the very end of the prompt, right before the model generates output. Models have strong recency bias.

```markdown
## Your Task
[... task description ...]

## Methodology
[... detailed methodology ...]

## CRITICAL — Before You Write, Remember:
- Every headline must be under 15 words
- Lead with a customer benefit, not a product feature
- No jargon — if you wouldn't say it at a BBQ, don't write it
```

**Effectiveness**: Medium-High. Fixes amnesia-type failures.

### 4. Add Explicit Constraints

**When**: Outputs are vague or inconsistent in quality.
**How**: Replace subjective instructions with measurable constraints.

```markdown
# Before (vague)
Write a concise, engaging headline.

# After (concrete)
Write a headline that:
- Is 5-12 words long
- Starts with a verb or a "you" statement
- Contains no words over 3 syllables
- Would make someone stop scrolling
```

**Effectiveness**: Medium. Improves consistency, may reduce creativity.

### 5. Add a Self-Check Step

**When**: The model produces outputs that violate its own instructions.
**How**: Add a verification step after generation.

```markdown
After writing your output, verify it against this checklist:
□ Under 15 words
□ No jargon (leverage, utilize, synergy, paradigm, holistic)
□ Customer is the subject, not the product
□ Contains an emotional or curiosity hook

If any check fails, revise before outputting.
```

**Effectiveness**: Medium. Adds latency but catches obvious violations.

### 6. Decompose the Task

**When**: The prompt asks for too many things at once and quality suffers across all of them.
**How**: Break into sequential steps with intermediate outputs.

```markdown
# Step 1: Identify the customer's pain point from the input
# Step 2: Write 3 benefit statements addressing that pain
# Step 3: Pick the strongest benefit and craft a headline around it
# Step 4: Edit the headline for punch (under 12 words, active verb, no jargon)
```

**Effectiveness**: Medium-High for complex tasks. Increases token usage.

## Version Changelog Format

Every new prompt version should start with a changelog:

```markdown
# Changelog

## v3 (2026-02-27)
- Added 2 few-shot examples for B2B headlines (addresses low emotional-hook scores)
- Moved word-limit constraint to the end of the prompt (addresses format drift)
- Eval score: 0.88 (up from 0.72 in v2)

## v2 (2026-02-26)
- Added negative examples for jargon (addresses no-jargon criterion failure)
- Replaced "be concise" with "under 15 words" (addresses ambiguity)
- Eval score: 0.72 (up from 0.58 in v1)

## v1 (2026-02-25)
- Initial version
- Eval score: 0.58
```

## When to Stop Iterating

| Signal | Action |
|--------|--------|
| Score > 0.90 with no critical failures | Ship it |
| Score 0.80-0.89, improving each iteration | One more iteration, then ship |
| Score plateaued across 2 iterations | The prompt architecture may be the bottleneck — consider a redesign |
| Score regressed from previous version | Revert to the previous version and try a different iteration strategy |
| 5+ iterations with marginal gains | Diminishing returns — ship the best version and move on |
