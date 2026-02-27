# Eval Criteria Types

## Automated Criteria

### `exact_match`
The output must exactly match a reference string. Use sparingly — only for highly deterministic prompts (e.g., classification labels, structured data extraction).

```yaml
- id: "correct-label"
  type: "exact_match"
  expected: "positive"
```

### `contains`
The output must contain a specific substring. Useful for checking that required elements are present.

```yaml
- id: "includes-cta"
  type: "contains"
  expected: "Get started"
```

### `regex`
The output must match (or not match) a regular expression. Useful for format validation, word limits, and jargon detection.

```yaml
# Must match — output should be a single sentence
- id: "single-sentence"
  type: "regex"
  pattern: "^[^.!?]*[.!?]$"

# Must NOT match — no corporate jargon
- id: "no-jargon"
  type: "regex"
  pattern: "(?i)\\b(leverage|synergy|paradigm|utilize|holistic)\\b"
  expect_no_match: true
```

### `length_check`
Validates output length in words, characters, or lines.

```yaml
- id: "under-20-words"
  type: "length_check"
  metric: "words"
  max: 20

- id: "at-least-3-lines"
  type: "length_check"
  metric: "lines"
  min: 3
```

## Human/LLM-Judged Criteria

### `rubric`
A qualitative scoring rubric. The evaluator (human or LLM) reads the output and assigns a score from 0.0 to 1.0 based on the description.

```yaml
- id: "tone-matches-brand"
  type: "rubric"
  description: |
    Score 1.0: Output perfectly matches the brand's casual, confident tone
    Score 0.7: Mostly matches but has some formal/stiff phrasing
    Score 0.4: Mixed tone — some parts casual, some corporate
    Score 0.0: Entirely wrong tone — too formal, too casual, or off-brand
```

### `llm_judge`
Delegates scoring to an LLM with a specific judge prompt. The judge prompt should ask a YES/NO question or request a score.

```yaml
- id: "emotionally-engaging"
  type: "llm_judge"
  judge_prompt: |
    Read this headline and score its emotional engagement from 0 to 1.
    - 1.0: Creates strong curiosity, urgency, or excitement
    - 0.5: Mildly interesting but forgettable
    - 0.0: Completely flat, no emotional hook
    Output only the numeric score.
```

## Choosing Criteria Types

| Need | Use |
|------|-----|
| Output must be a specific value | `exact_match` |
| Output must include key phrases | `contains` |
| Output must follow a format | `regex` |
| Output must be within length bounds | `length_check` |
| Quality is subjective but definable | `rubric` |
| Need nuanced judgment at scale | `llm_judge` |

**Rule of thumb**: Use automated criteria for constraints (length, format, forbidden words) and judged criteria for quality (tone, relevance, creativity). Aim for at least 50% automated criteria to keep evals reproducible.
