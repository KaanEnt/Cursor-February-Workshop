# Test Case Design

## The 5-Case Minimum

Every eval suite needs at least 5 test cases covering distinct scenarios. More is better, but 5 is the floor for meaningful signal.

### Case Type 1: Golden Path (2 cases minimum)

Typical, well-formed inputs that represent the most common usage. These are your baseline — if the prompt can't handle these, nothing else matters.

```yaml
- id: "standard-b2b-headline"
  input: "Write a headline for a CRM tool targeting sales managers"
  context: "Product helps reduce data entry time by 60%"
  category: "golden_path"
```

**Design principle**: Pull these directly from real user requests or production logs. Don't invent idealized inputs.

### Case Type 2: Edge Case (1 case minimum)

Minimal, ambiguous, or unusual inputs that test the prompt's resilience.

```yaml
- id: "minimal-input"
  input: "Write a headline"
  context: ""
  category: "edge_case"
  notes: "Tests behavior when no product context is given"

- id: "ambiguous-audience"
  input: "Write a headline for a tool that helps both developers and designers"
  context: "Collaboration platform"
  category: "edge_case"
  notes: "Tests whether prompt picks an audience or tries to address both"
```

**Design principle**: Think about what a lazy, confused, or new user would send. The prompt should degrade gracefully, not break.

### Case Type 3: Adversarial (1 case minimum)

Inputs that push against the prompt's constraints to see if they hold.

```yaml
- id: "jargon-heavy-input"
  input: "Write a headline for our paradigm-shifting, AI-powered, enterprise-grade synergistic platform"
  context: "User provided input is full of jargon the prompt should NOT propagate"
  category: "adversarial"
  notes: "Tests whether the prompt's no-jargon instruction overrides jargon in the input"

- id: "off-topic-request"
  input: "Write me a poem about cats"
  context: "This is not a headline request — tests boundary enforcement"
  category: "adversarial"
```

**Design principle**: Try to break the prompt. If you can break it in testing, users will break it in production.

### Case Type 4: Domain-Specific (1 case minimum)

Inputs drawn directly from the actual production context where this prompt will be used.

```yaml
- id: "real-product-q4-launch"
  input: "Write a headline for our Q4 product launch — new AI-powered search feature"
  context: "Internal product: Acme Search. Audience: existing customers. Tone: excited but not hypey."
  category: "domain_specific"
  notes: "Mirrors an actual request from the marketing team"
```

**Design principle**: These cases are the most valuable because they represent reality. Update them as the product evolves.

## Test Case Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique slug for this test case |
| `input` | Yes | The user message / request |
| `context` | No | Additional context, constraints, or background |
| `category` | No | `golden_path`, `edge_case`, `adversarial`, `domain_specific` |
| `expected` | No | Reference output for comparison (useful for `exact_match` criteria) |
| `notes` | No | Why this test case exists, what it's meant to surface |

## Scaling Your Test Suite

| Suite Size | When to Use |
|------------|-------------|
| 5 cases | Early exploration, first eval pass |
| 10-15 cases | Active iteration, pre-ship validation |
| 25+ cases | Production prompts with high traffic or high stakes |

**Adding new cases**: Every time the prompt fails in production (user complains, output is wrong), capture that input as a new test case. Your eval suite should grow organically from real failures.
