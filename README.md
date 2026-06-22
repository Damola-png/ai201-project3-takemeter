# TakeMeter

## Project Overview
TakeMeter is a text classifier for discourse quality in r/soccer, Reddit's largest football discussion community. The model is a fine-tuned `distilbert-base-uncased` classifier that predicts one of three labels:
- Analysis
- Opinion
- Fan Commentary

This project also includes a zero-shot baseline using Groq `llama-3.3-70b-versatile` for comparison.These labels capture a meaningful distinction in football discourse: whether a post explains a claim with evidence, argues from intuition, or primarily expresses emotion and reaction.



## Community Choice and Reasoning
r/soccer was selected because it contains high-variance football discourse in a single community. Posts often look similar at a glance but differ in important ways — some are evidence-based and analytical, some are reasoned arguments without concrete evidence, and others are primarily emotional reactions. This variety makes it a strong fit for a classification task where the boundaries between labels are subtle and meaningful.



That makes r/soccer a strong environment for testing whether a classifier can separate subtle discourse boundaries.

## Label Taxonomy
The classifier predicts three labels.

### Analysis
Posts that explain a football-related claim using specific verifiable evidence, including statistics, historical examples, match events, or tactical observations.

Examples from the dataset:
- *"Arsenal are the first ever team in Premier League history to go the whole season without receiving a red card or conceding a penalty"* → Analysis because it cites a specific, verifiable record.
- *"Erling Haaland has scored 2 or more goals on his Bundesliga, Premier League, Champions League and World Cup debuts"* → Analysis because it makes a comparative factual claim across multiple competitions.

### Opinion
Posts that make a football-related argument or prediction based on reasoning and intuition, without specific verifiable evidence.

Decision rule: reasoning alone is not sufficient for Analysis. A detailed and logical argument can still be Opinion if it is unsupported by verifiable evidence.

Examples from the dataset:
- *"I'm furious we sold him - he was our best creative player and now we have no one who can unlock a low block"* → Opinion because it makes a football argument about squad composition even though the tone is emotional.
- *"I disagree with him going to Germany. Winning the Bundesliga with Bayern Munich is like winning the Scottish Premiership with Celtic."* → Opinion because it argues a comparative judgment about league prestige without data.

### Fan Commentary
Posts whose primary purpose is reacting, celebrating, complaining, joking, venting, or expressing support, rather than making a reasoned football argument.

Examples from the dataset:
- *"Full time scenes as Leicester City are relegated to League One"* → Fan Commentary because the post documents a reaction moment; no argument is being made.
- *"Unbelievable, what a performance from Arsenal!"* → Fan Commentary because the primary purpose is emotional expression, not a football claim.

## Hard Edge Cases
### Analysis vs Opinion
A post is Analysis only if it includes specific verifiable evidence.

Example boundary:
- "The Champions League should return to a champions-only format" without data -> Opinion.
- The same claim with historical performance data, attendance figures, or competitive-balance metrics -> Analysis.

### Opinion vs Fan Commentary
If the primary purpose is to make a football argument, classify as Opinion even when tone is emotional.

If the primary purpose is emotional reaction, classify as Fan Commentary even if there is a brief football reason.

Example boundary:
- "I'm furious we sold him - he was our best creative player and now we have no one who can unlock a low block" -> Opinion (argument is primary).
- "I'm absolutely devastated we sold him. Worst transfer window ever." -> Fan Commentary (emotion is primary).

### Opinion vs Fan Commentary
If the primary purpose is to make a football argument, classify as Opinion even when tone is emotional.

If the primary purpose is emotional reaction, classify as Fan Commentary even if there is a brief football reason.

Example boundary:
- "I'm furious we sold him - he was our best creative player and now we have no one who can unlock a low block" -> Opinion.
- "I'm absolutely devastated we sold him. Worst transfer window ever." -> Fan Commentary.

## Data Collection
Examples are collected manually from r/soccer using:
- Match threads
- Post-match threads
- Discussion threads
- Statistics posts
- Quote posts
- Original discussion posts

Dataset format is a CSV with columns:
- `id`
- `text`
- `label`
- `llm_label`
- `notes`

Target class counts:
- Analysis: 65
- Opinion: 65
- Fan Commentary: 70

If a class is underrepresented, targeted sampling is used. Analysis examples are most common in statistics and tactical discussions; Fan Commentary is most common in match-thread reactions.

## Fine-tuning Approach
- Base model: `distilbert-base-uncased`
- Libraries: `transformers`, `datasets`, `scikit-learn`
- Training environment: Google Colab free T4 GPU
- Split strategy: 70% train, 15% validation, 15% test (handled by starter notebook)

## Baseline Description
Baseline model: Groq `llama-3.3-70b-versatile` in zero-shot classification mode.

Purpose of baseline:
- Establish a non-fine-tuned reference point.
- Compare fine-tuned DistilBERT performance against a strong general-purpose LLM baseline.

## Evaluation Report
To be completed after model training.

### Metrics to Report
To be completed after model training.

### Overall Performance
To be completed after model training.

### Per-class Performance
To be completed after model training.

### Confusion Matrix Analysis
To be completed after model training.

### Success Criteria Check
To be completed after model training.

Planned success criteria:
- Overall accuracy >= 80%
- Analysis F1 >= 0.75
- Opinion F1 >= 0.75
- Fan Commentary F1 >= 0.85
- Macro F1 >= 0.80

Deployment requirement:
- Macro F1 >= 0.80 and no individual label below 0.70 F1

## Sample Classifications
To be completed after model training.

## Reflection
To be completed after model training.

## Spec Reflection
To be completed after model training.

## AI Usage
AI tools were used in three roles:

1. Label stress-testing
- An LLM generated 5-10 synthetic examples for each difficult label boundary.
- This was used to validate whether decision rules were consistent before full annotation.

2. Annotation assistance
- An LLM produced initial label suggestions stored in `llm_label`.
- Every label was manually reviewed before inclusion in the final dataset.
- Final training labels always come from reviewed human decisions.

3. Failure analysis support (post-training)
- Misclassified examples will be exported and reviewed with an LLM to identify recurring error patterns.
- Any LLM-suggested pattern will be verified against confusion matrix evidence.

