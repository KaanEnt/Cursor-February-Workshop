# Version Comparison Methodology

## Generating a Comparison Report

After running evals on two or more prompt versions, produce a structured comparison that makes the decision to ship (or iterate further) obvious.

## Comparison Table Format

### Per-Criterion Comparison

```
| Criterion           | v1    | v2    | v3    | Trend     |
|---------------------|-------|-------|-------|-----------|
| follows-framework   | 0.60  | 0.72  | 0.88  | ▲ +0.28   |
| under-word-limit    | 0.80  | 0.90  | 0.95  | ▲ +0.15   |
| no-jargon           | 0.70  | 0.95  | 0.95  | ▲ +0.25   |
| emotional-hook      | 0.40  | 0.60  | 0.78  | ▲ +0.38   |
| **OVERALL**         | 0.58  | 0.72  | 0.88  | ▲ +0.30   |
```

### Per-Test-Case Comparison

```
| Test Case            | v1    | v2    | v3    | Notes                        |
|----------------------|-------|-------|-------|------------------------------|
| standard-b2b         | 0.70  | 0.85  | 0.92  | Steady improvement           |
| minimal-input        | 0.40  | 0.55  | 0.80  | Big jump in v3 (few-shot)    |
| jargon-heavy-input   | 0.50  | 0.90  | 0.92  | Fixed by negative examples   |
| real-product-launch  | 0.65  | 0.70  | 0.88  | Domain example helped         |
| off-topic-request    | 0.45  | 0.60  | 0.82  | Boundary enforcement improved |
```

## Regression Detection

A **regression** is any criterion or test case where the score dropped by more than 0.05 between versions.

Regressions are more important than improvements — they indicate the new version broke something.

### Handling Regressions

1. **Identify what changed** between versions that could cause the regression
2. **Determine if the regression is acceptable** — sometimes improving one criterion trades off against another
3. **If unacceptable**, either:
   - Revert the specific change that caused the regression
   - Add a constraint to preserve the behavior that regressed
   - Merge the best parts of both versions

## Statistical Confidence

With small test suites (5-10 cases), score differences under 0.05 are noise. Don't iterate based on tiny fluctuations.

| Score Difference | Interpretation |
|-----------------|----------------|
| < 0.05 | Noise — not a real change |
| 0.05 - 0.10 | Probably real, but could be variance |
| 0.10 - 0.20 | Meaningful improvement |
| > 0.20 | Significant improvement (or regression) |

To increase confidence:
- Add more test cases (more data points = more signal)
- Run the same eval 3 times and average (controls for LLM output variance)
- Use automated criteria where possible (deterministic = no variance)

## The Ship Decision

### Ship Checklist

Before shipping a prompt version:
- [ ] Overall score ≥ 0.85
- [ ] No individual criterion below 0.60
- [ ] No regressions > 0.05 from best previous version
- [ ] Adversarial test cases all score ≥ 0.70
- [ ] Domain-specific test cases all score ≥ 0.80
- [ ] At least 2 eval runs to confirm consistency

### Version History Hygiene

Keep all versions and results forever. The eval history is documentation:
- It shows *why* the prompt looks the way it does
- It prevents re-introducing patterns that previously failed
- It provides a rollback target if production performance degrades

### Post-Ship Monitoring

After shipping a prompt:
1. Collect real user inputs that produce bad outputs
2. Add those as new test cases to the eval suite
3. Re-run the eval suite monthly (or after any prompt change)
4. If the score drops below 0.80, trigger a new iteration cycle

This creates a flywheel: production failures feed the eval suite, which drives prompt improvements, which reduces production failures.
