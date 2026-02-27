---
name: prompt-eval-framework
description: Systematic framework for evaluating AI-facing prompts, tracking success rates across versions, and iterating prompts based on measured performance. Covers eval suite design, scoring criteria, version management, and data-driven prompt improvement.
---

# Prompt Eval Framework

A methodology for treating AI prompts as testable, measurable artifacts — not static text blobs.

## Core Principle

Every AI-facing prompt is a function: it takes an input and produces an output. Like any function, it should be tested against expected behaviors, scored, and improved when it underperforms.

## When to Use This Skill

Activate this skill when:
- Building or modifying a system prompt, agent command, or skill file
- A prompt is producing inconsistent or low-quality outputs
- You need to compare two prompt versions objectively
- Shipping a prompt to production and need confidence it works

## The Eval Loop

```
┌─────────────────┐
│  1. Define       │  What does "good output" look like?
│     Criteria     │  Write rubrics, regex checks, judge prompts.
└────────┬────────┘
         ▼
┌─────────────────┐
│  2. Write        │  Real-world inputs spanning easy, hard,
│     Test Cases   │  and adversarial scenarios.
└────────┬────────┘
         ▼
┌─────────────────┐
│  3. Run          │  Feed each test case through the prompt,
│     Evaluation   │  capture raw output, score each criterion.
└────────┬────────┘
         ▼
┌─────────────────┐
│  4. Analyze      │  Find patterns in failures. Which criteria
│     Results      │  are weakest? Which test cases break?
└────────┬────────┘
         ▼
┌─────────────────┐
│  5. Iterate      │  Revise the prompt to address failures.
│     Prompt       │  Never overwrite — version and re-eval.
└────────┬────────┘
         ▼
┌─────────────────┐
│  6. Compare      │  Side-by-side scoring across versions.
│     Versions     │  Ship when score > threshold.
└────────┘────────┘
         │
         └──→ Repeat from step 3 if score < threshold
```

## Step 1: Define Criteria

Read `references/01-eval-criteria-types.md` for the full taxonomy of criteria types.

Every eval suite needs 3-7 weighted criteria. Each criterion has:

| Field | Description |
|-------|-------------|
| `id` | Short identifier (`no-jargon`, `follows-format`) |
| `description` | What "passing" looks like in plain English |
| `weight` | Relative importance (0.0 - 1.0, all weights should sum to 1.0) |
| `type` | How to score it: `exact_match`, `contains`, `regex`, `rubric`, `llm_judge` |

**Weight assignment heuristic**: Criteria that affect user-facing quality or correctness get 2x the weight of stylistic/formatting criteria.

## Step 2: Write Test Cases

Read `references/02-test-case-design.md` for test case design patterns.

Minimum viable test suite: **5 test cases** spanning:
- 2 "golden path" cases (typical, well-formed input)
- 1 edge case (minimal input, ambiguous request)
- 1 adversarial case (input that tries to break the prompt)
- 1 domain-specific case (input from the actual production context)

Each test case has:
- `id`: descriptive slug
- `input`: the user message / context fed to the prompt
- `context`: additional context or constraints
- `expected` (optional): a reference output for comparison

## Step 3: Run the Evaluation

For each test case:
1. Construct the full prompt (system prompt + test case input + context)
2. Run it and capture the raw output verbatim
3. Score each criterion against the output
4. Record: `{ test_id, output, scores, weighted_score }`

Compute overall suite score: weighted average across all test cases.

## Step 4: Analyze Results

Read `references/03-failure-diagnosis.md` for the diagnostic framework.

After a run, answer these questions:
1. Which criterion has the **lowest average score**?
2. Which test case has the **lowest weighted score**?
3. Is there a **pattern** across failures? (same type of mistake repeating)
4. Is the failure caused by the **prompt** or the **test case expectations**?

Common failure modes:
- **Instruction amnesia**: prompt is too long, model forgets early instructions → move critical instructions to the end
- **Ambiguous constraint**: prompt says "be concise" but doesn't define what concise means → add word/sentence limits
- **Missing negative example**: prompt says what to do but not what to avoid → add "Don't do X" with examples
- **Conflicting instructions**: two parts of the prompt contradict → resolve the conflict explicitly

## Step 5: Iterate the Prompt

Read `references/04-prompt-iteration-patterns.md` for revision strategies.

Rules for iteration:
1. Change **one thing at a time** — don't rewrite the whole prompt between versions
2. Version every change: `v1.md` → `v2.md` → `v3.md`
3. Write a **changelog** at the top of each version explaining what changed and why
4. Re-run the **full eval suite** on the new version (don't cherry-pick test cases)

## Step 6: Compare and Ship

Read `references/05-version-comparison.md` for comparison methodology.

Ship decision matrix:

| Overall Score | Action |
|---------------|--------|
| ≥ 0.90 | Ship with confidence |
| 0.80 - 0.89 | Ship if no critical criteria are below 0.70 |
| 0.70 - 0.79 | Iterate one more time, focus on lowest criteria |
| < 0.70 | Major revision needed — reconsider the prompt's architecture |

## File Structure Convention

```
evals/
  <prompt-name>/
    eval-suite.yaml
    prompts/
      v1.md
      v2.md
      changelog.md
    results/
      run-v1-2026-02-27T14-30.yaml
      run-v2-2026-02-27T15-00.yaml
    analysis/
      comparison-v1-v2.md
```
