# TakeMeter Planning Notes

## Community
- Target community: r/soccer (largest football discussion community on Reddit).
- Why this community: strong variety of discourse styles in one place.
- Typical post types seen during review:
  - Tactical breakdowns
  - Transfer reactions
  - Statistical arguments
  - General fan commentary
- Core challenge: many posts look similar at first glance but differ in discourse quality and evidentiary support.

## Labels
Three-label taxonomy:

1. Analysis
- Definition: post explains a football-related claim using specific verifiable evidence.
- Acceptable evidence examples: statistics, historical examples, match events, tactical observations.

2. Opinion
- Definition: post makes a football-related argument or prediction using reasoning/intuition but no verifiable evidence.
- Decision rule: reasoning alone is not enough for Analysis.
- If argument is detailed/logical but unsupported by verifiable evidence, still label as Opinion.

3. Fan Commentary
- Definition: post is mainly reactive (celebrating, complaining, joking, venting, support).
- Primary purpose is emotional reaction, not reasoned argument.

## Hard Edge Cases
### Analysis vs Opinion
- Analysis requires specific verifiable evidence.
- Opinion can still be detailed, coherent, and persuasive, but if it is based only on intuition/reasoning, it remains Opinion.
- Example rule test:
  - "Champions League should return to champions-only format" without data -> Opinion.
  - Same claim with historical performance data, attendance figures, or competitive-balance metrics -> Analysis.

### Opinion vs Fan Commentary
- If main purpose is making a football argument, label as Opinion, even if emotional.
- If main purpose is reaction/venting/celebration, label as Fan Commentary, even if a short football reason appears.
- Example rule test:
  - "I'm furious we sold him - he was our best creative player and now we have no one who can unlock a low block" -> Opinion (argument is primary).
  - "I'm absolutely devastated we sold him. Worst transfer window ever." -> Fan Commentary (emotion is primary).

## Data Collection Plan
- Collection method: manual sampling from r/soccer.
- Sources:
  - Match threads
  - Post-match threads
  - Discussion threads
  - Statistics posts
  - Quote posts
  - Original discussion posts
- Dataset format: CSV with columns:
  - id
  - text
  - label
  - llm_label
  - notes
- Class targets:
  - Analysis: 65
  - Opinion: 65
  - Fan Commentary: 70
- Sampling strategy for imbalance:
  - If Analysis is low, sample more statistics/tactical posts.
  - If Fan Commentary is low, sample more match-thread reactions.

## Evaluation Metrics
- Primary metrics:
  - Accuracy
  - Precision
  - Recall
  - F1-score
- Also report per-class metrics for all three labels.
- Rationale:
  - Classes are conceptually close.
  - Edge boundaries are intentionally difficult.
  - Accuracy alone can hide overprediction of a dominant class.

## Definition of Success
- Overall accuracy >= 80%
- Analysis F1 >= 0.75
- Opinion F1 >= 0.75
- Fan Commentary F1 >= 0.85
- Macro-averaged F1 >= 0.80

Deployment gate:
- Macro F1 >= 0.80
- No individual class below 0.70 F1

## AI Tool Plan
- Label stress-testing:
  - Use an LLM to generate 5-10 synthetic examples at hard boundaries.
  - Goal: pressure-test decision rules before annotation.
  - Current result: generated boundary examples consistently fell on one side, suggesting rules are stable.

- Annotation assistance:
  - Use LLM suggested labels in `llm_label`.
  - Every example is manually reviewed before finalizing labels.
  - Final dataset uses reviewed labels only.

- Failure analysis (post-training):
  - Export misclassified examples.
  - Use LLM to detect recurring error patterns.
  - Verify those hypotheses against confusion matrix evidence.
