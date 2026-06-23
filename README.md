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
- Analysis: 67
- Opinion: 67
- Fan Commentary: 66
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


*The model was trained with default hyperparameters: 3 epochs, learning rate 2e-5, batch size 16. Training for only 3 epochs was appropriate given the small dataset size — more epochs risked overfitting on 140 training examples.*

```
You are a classifier for r/soccer posts.
Classify the following post into exactly one of these labels:

analysis - The post explains a football-related claim using specific 
verifiable evidence such as statistics, historical examples, match 
events, or tactical observations.

opinion - The post makes a football-related argument or prediction 
based on reasoning and intuition but provides no verifiable evidence.

fan_commentary - The post primarily reacts, celebrates, complains, 
jokes, or expresses support without making a reasoned argument.

Respond with only the label name. Do not explain your answer.
Post: {text}
```


## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot Llama 70B (baseline) | 83.9% |
| Fine-tuned DistilBERT | 71.0% |

The fine-tuned model performed worse than the zero-shot baseline by 12.9 percentage points.

### Per-Class Metrics

**Zero-shot Baseline (Llama 70B)**

| Label | Precision | Recall | F1 |
|---|---|---|---|
| Analysis | 1.00 | 0.60 | 0.75 |
| Opinion | 0.82 | 0.82 | 0.82 |
| Fan Commentary | 0.82 | 0.93 | 0.88 |
| Macro Average | 0.88 | 0.78 | 0.81 |

**Fine-tuned DistilBERT**

| Label | Precision | Recall | F1 |
|---|---|---|---|
| Analysis | 0.00 | 0.00 | 0.00 |
| Opinion | 1.00 | 0.64 | 0.78 |
| Fan Commentary | 0.62 | 1.00 | 0.77 |
| Macro Average | 0.54 | 0.55 | 0.52 |

### Confusion Matrix (Fine-tuned DistilBERT)

| True \ Predicted | analysis | opinion | fan_commentary |
|---|---|---|---|
| analysis | 0 | 0 | 5 |
| opinion | 0 | 7 | 4 |
| fan_commentary | 0 | 0 | 15 |

Key observations:
- The model correctly classified all 15 Fan Commentary examples.
- The model correctly classified 7 of 11 Opinion examples.
- The model failed completely on Analysis, predicting fan_commentary for all 5 examples.


Planned success criteria:
- Overall accuracy >= 80%
- Analysis F1 >= 0.75
- Opinion F1 >= 0.75
- Fan Commentary F1 >= 0.85
- Macro F1 >= 0.80

Deployment requirement:
- Macro F1 >= 0.80 and no individual label below 0.70 F1

### Wrong Predictions Analysis

**Wrong Prediction 1**
Text: "How referee calls shaped the Premier League: What the table 
could have looked like with correct decisions"
True label: analysis
Predicted: fan_commentary (confidence: 0.37)

Analysis: This post is a statistics-based counterfactual argument 
about referee decisions, which clearly qualifies as Analysis. The 
model predicted fan_commentary, likely because the headline framing 
("How referee calls shaped...") resembles reactive fan language 
rather than structured evidence. This is a data problem — the 
training set likely did not contain enough Analysis examples with 
this kind of headline-style framing.

**Wrong Prediction 2**
Text: "In Italy, the fans sing non-stop. In England it's different 
but the whole stadium is involved."
True label: opinion
Predicted: fan_commentary (confidence: 0.37)

Analysis: This post makes a comparative cultural argument about 
football atmospheres in two countries, which qualifies as Opinion. 
The model predicted fan_commentary, likely because the post mentions 
fans and stadiums — surface-level features associated with Fan 
Commentary. This is a model problem — the model is relying on 
topic keywords rather than the post's argumentative structure.

**Wrong Prediction 3**
Text: "Hydration breaks at World Cup add nothing but take away 
a lot, Says Bielsa"
True label: opinion
Predicted: fan_commentary (confidence: 0.37)

Analysis: This post quotes a manager making a reasoned argument 
about a competition rule, which qualifies as Opinion. The model 
predicted fan_commentary, likely because quote posts often appear 
in reaction threads where Fan Commentary is dominant. This reveals 
a systematic pattern — the model struggles with quote-style posts 
regardless of whether the quoted content contains an argument.

### Sample Classifications

| Text | True Label | Predicted Label | Confidence |
|---|---|---|---|
| "Arsenal are the first ever team in Premier League history to go the whole season without receiving a red card or conceding a penalty" | analysis | analysis | 0.89 |
| "I'm furious we sold him - he was our best creative player and now we have no one who can unlock a low block" | opinion | opinion | 0.76 |
| "Unbelievable, what a performance from Arsenal!" | fan_commentary | fan_commentary | 0.94 |
| "How referee calls shaped the Premier League: What the table could have looked like with correct decisions" | analysis | fan_commentary | 0.37 |
| "In Italy, the fans sing non-stop. In England it's different but the whole stadium is involved." | opinion | fan_commentary | 0.37 |

**Correct prediction explanation:**
"Unbelievable, what a performance from Arsenal!" was correctly 
classified as Fan Commentary with 0.94 confidence. This prediction 
is reasonable because the post contains no football argument — 
it is purely an emotional exclamation. The high confidence score 
reflects that this is an unambiguous example of the label.

### Reflection: What the Model Learned vs What I Intended

I intended the model to learn the distinction between evidence-based 
reasoning (Analysis), intuition-based argumentation (Opinion), and 
emotional reaction (Fan Commentary). The central challenge I designed 
into the task was the Analysis vs Opinion boundary — whether a post 
supports its claims with verifiable evidence or relies on reasoning 
alone.

What the model actually learned was simpler and less useful: it 
learned to identify Fan Commentary reliably and defaulted to that 
label when uncertain. The model achieved perfect recall on Fan 
Commentary (1.00) but completely failed on Analysis (F1: 0.00), 
predicting fan_commentary for every Analysis example in the test set.

The gap between intention and outcome has three likely causes:

1. Surface-level features dominated. The model appears to have 
learned surface patterns associated with Fan Commentary — short 
posts, emotional language, exclamations — rather than the 
structural distinction between evidence and argument. Posts that 
looked like headlines or quotes were misclassified as Fan Commentary 
regardless of their content.

2. Analysis examples were underrepresented in training. With only 
~47 Analysis examples in the training set, the model did not see 
enough variety to learn what Analysis looks like across different 
post formats — headlines, statistics posts, tactical breakdowns, 
and counterfactual arguments all belong to Analysis but look very 
different on the surface.

3. DistilBERT is too small for this task at this data size. The 
zero-shot Llama 70B baseline outperformed the fine-tuned model 
by 12.9 percentage points, suggesting that a larger model with 
general language understanding handles subtle argumentative 
distinctions better than a small fine-tuned model trained on 
200 examples.

## Spec Reflection

**One way the spec helped:**
The spec's emphasis on label design before data collection was the 
most valuable constraint in the project. Being required to define 
decision rules and stress-test edge cases before annotating 200 
examples forced me to think carefully about the Analysis vs Opinion 
boundary early. Without that structure, I would have started 
collecting data with vague labels and produced an inconsistent 
dataset.

**One way implementation diverged from the spec:**
The spec assumes the fine-tuned model will outperform the zero-shot 
baseline, framing the baseline as a reference point to beat. In 
practice, the fine-tuned DistilBERT underperformed the Llama 70B 
baseline by 12.9 percentage points. This divergence was not a 
pipeline error — it reflects a genuine limitation of fine-tuning 
a small model on a small dataset for a subtly defined task. The 
baseline comparison ended up being more informative than expected, 
revealing that task-specific training does not always outperform 
a capable zero-shot model at this data scale.

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

