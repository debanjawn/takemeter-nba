# TakeMeter NBA

A fine-tuned text classifier that evaluates discourse quality in NBA online 
communities. Built for AI201 Project 3.

## Community

I chose r/NBATalk because I'm an NBA fan and read the comments constantly. 
The discourse is extremely varied — you get detailed statistical breakdowns 
sitting right next to pure emotional reactions and obvious ragebait. The 
community is text-heavy and opinionated, which makes it a strong fit for a 
classification task. I added a `ragebait` label specifically because it's a 
real and distinct phenomenon I see constantly — people post exaggerated 
claims not because they believe them but to provoke a reaction.

## Label Taxonomy

**`analysis`** — The post makes a structured argument backed by statistics, 
historical comparison, or tactical observation. Evidence is specific and 
verifiable.
- "Boston attempted the most threes in the league this season, which is why 
they're so difficult to come back against — even a small defensive lapse can 
turn into a 12-point swing in two minutes."
- "Denver's offense doesn't rely on isolation because Jokić's passing creates 
advantages before defenders can even set, which is why role players shoot so 
efficiently there."

**`hot_take`** — A bold, confident opinion stated without supporting evidence. 
The claim might be true, but the post asserts rather than argues.
- "They mightve been able to win that chip without Tatum."
- "LeBron in the 90s wouldve had 7+ rings, Kobe was a better MJ."

**`reaction`** — An immediate emotional response to a specific event. Little 
to no argument — the post is expressing a feeling in the moment.
- "NO WAY HE HIT THAT LMAOOOO"
- "GIANNIS JUST SENT THAT INTO THE THIRD ROW"

**`ragebait`** — A post that expresses a reaction but the claim is so 
exaggerated it couldn't be a sincere belief — it exists purely to provoke a 
response or just be outrageous.
- "Jalen Brunson is the size of a traffic cone and somehow gets superstar calls."
- "Trade Chet for an air fryer, him not being here is better."

## Data Collection

**Source:** AI-generated examples based on real r/NBATalk discourse patterns.

**Why AI-generated:** Three of the four labels (analysis, hot_take, reaction) 
are easy to collect from real Reddit threads using the .json trick. Ragebait 
is the exception. Real ragebait posts are context-dependent — they only read 
as ragebait if you know the player, the situation, and the community norms 
around that moment. Scraped out of context, they look identical to hot_takes 
or reactions. After attempting to collect real ragebait examples and finding 
consistent labeling nearly impossible without heavy context, AI generation 
was the only practical path to getting 50 cleanly labeled ragebait examples. 
Because the other three labels also needed to be AI-generated to maintain 
consistent labeling style across the dataset, all 200 examples were generated 
this way.

**Labeling process:** 50 examples were generated per label (200 total) using 
Claude, prompted with each label definition from planning.md. Examples were 
reviewed manually to confirm they fit the label definitions before inclusion.

**Label distribution:**
| Label | Count |
|-------|-------|
| analysis | 50 |
| hot_take | 50 |
| reaction | 50 |
| ragebait | 50 |

**Difficult-to-label examples:**

1. "Paul George has more podcast episodes than memorable playoff performances."
— Could be `hot_take` (bold opinion) or `ragebait` (too exaggerated to be 
sincere). Labeled `ragebait` because the framing is designed to provoke rather 
than argue a basketball point.

2. "The midrange shot is still essential for becoming an elite playoff scorer."
— Could be `analysis` (tactical claim) or `hot_take` (no evidence given). 
Labeled `hot_take` because no specific evidence is provided to support it.

3. "Chet Holmgren is more important to Oklahoma City's ceiling than most fans 
realize." — Could be `hot_take` or `analysis`. Labeled `hot_take` because 
the claim is stated without supporting statistics or tactical reasoning.

## Fine-Tuning Approach

**Base model:** `distilbert-base-uncased` from HuggingFace

**Training setup:** Fine-tuned on Google Colab using a free T4 GPU. Training 
took approximately 23 seconds for 3 epochs on 140 examples.

**Hyperparameters:**
- Epochs: 3 (default — appropriate for small datasets, more risks overfitting)
- Learning rate: 2e-5 (standard starting point for BERT-family models)
- Batch size: 16 (fits T4 GPU comfortably)

**Key hyperparameter decision:** I kept all defaults because the dataset size 
(200 examples) is within the range the notebook recommends for these settings. 
Increasing epochs beyond 3 on 200 examples risks overfitting.

## Baseline

**Model:** Groq `llama-3.3-70b-versatile` with zero-shot prompting

**Prompt approach:** The system prompt defined all 4 labels with one-sentence 
definitions and one example per label, instructing the model to output only 
the label name. The prompt was based directly on the label definitions in 
`planning.md`.

**How results were collected:** The notebook ran the prompt against all 30 
test set examples with a 0.1s delay between requests to respect free-tier 
rate limits. All 30 responses were parseable.

## Deployed Interface

The repo includes `index.html` — a single-file web app that runs locally in 
any browser with no installation required.

**How it works:** The interface uses the Groq API directly (the same 
zero-shot baseline that scored 80% on the test set) rather than the 
fine-tuned DistilBERT model. This was a deliberate choice — the fine-tuned 
model scored 43.3% on the test set and never once predicted `ragebait`, 
making it not useful in practice. The Groq baseline with a well-crafted 
prompt significantly outperformed it.

**To run it:**
1. Open `index.html` in your browser
2. Enter your Groq API key in the field at the top
3. Paste any NBA comment and hit Classify
4. The app returns the label (analysis, hot_take, reaction, or ragebait)

The API key is entered at runtime and never stored or committed to the repo.

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|-------|----------|
| Zero-shot baseline (Groq) | 80.0% |
| Fine-tuned DistilBERT | 43.3% |
| Change | -36.7% (regression) |

### Per-Class Metrics

**Baseline (Groq):**
| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| analysis | 1.00 | 1.00 | 1.00 |
| hot_take | 0.54 | 1.00 | 0.70 |
| reaction | 1.00 | 1.00 | 1.00 |
| ragebait | 1.00 | 0.14 | 0.25 |

**Fine-tuned DistilBERT:**
| Label | Precision | Recall | F1 |
|-------|-----------|--------|----|
| analysis | 0.38 | 1.00 | 0.55 |
| hot_take | 0.20 | 0.14 | 0.17 |
| reaction | 1.00 | 0.50 | 0.67 |
| ragebait | 0.00 | 0.00 | 0.00 |

### Confusion Matrix

| True \ Predicted | analysis | hot_take | reaction | ragebait |
|-----------------|----------|----------|----------|----------|
| analysis | 8 | 0 | 0 | 0 |
| hot_take | 6 | 1 | 0 | 0 |
| reaction | 0 | 4 | 4 | 0 |
| ragebait | 7 | 0 | 0 | 0 |

### Wrong Predictions Analysis

**#1 — "THIS ENDING IS UNBELIEVABLE."**
- True: `reaction` | Predicted: `hot_take` | Confidence: 0.27
- Short all-caps emotional text with no specific content. The model likely 
couldn't distinguish it from a bold opinion because both are short and lack 
evidence. The model had no event context to anchor it as a reaction.

**#2 — "Paul George has more podcast episodes than memorable playoff performances."**
- True: `ragebait` | Predicted: `analysis` | Confidence: 0.27
- Written in a calm declarative style that resembles an analytical observation. 
The model read the structure as analysis rather than catching that the claim 
is an exaggerated dig with no real basketball argument.

**#3 — "Jokic is just a tall guy who throws fancy passes to wide-open teammates."**
- True: `ragebait` | Predicted: `analysis` | Confidence: 0.30
- Same pattern as #2 — ragebait written in a declarative sentence gets 
classified as analysis. The model never learned to identify ragebait at all, 
predicting it 0 times across the entire test set.

### Sample Classifications

| Text | True Label | Predicted | Confidence | Correct? |
|------|-----------|-----------|------------|---------|
| "Boston attempted the most threes..." | analysis | analysis | 0.85 | ✅ |
| "THIS ENDING IS UNBELIEVABLE." | reaction | hot_take | 0.27 | ❌ |
| "NO WAY HE HIT THAT LMAOOOO" | reaction | reaction | 0.91 | ✅ |
| "Trade Chet for an air fryer..." | ragebait | analysis | 0.28 | ❌ |
| "They mightve been able to win that chip without Tatum." | hot_take | analysis | 0.27 | ❌ |

## Reflection

**What the model learned vs. what I intended:**

I intended the model to learn the conceptual distinction between a genuine 
argument, a bold opinion, an emotional reaction, and an exaggerated 
provocation. What it actually learned was surface-level formatting patterns.

The root cause was a data collection problem, not a modeling problem. 
Ragebait is genuinely difficult to collect from Reddit — out of context, 
a ragebait post looks identical to a hot_take or reaction. You need to know 
the player, the moment, and the community norms to recognize that a post is 
designed to provoke rather than express a genuine opinion. This made it 
impossible to build a cleanly labeled real-data ragebait set. AI generation 
was the only practical way to get 50 examples that were unambiguously ragebait 
by construction.

The tradeoff was that AI-generated text has a consistent, clean style that 
real Reddit posts don't have. All four label categories ended up with 
structurally similar writing, so the model learned formatting patterns 
instead of conceptual distinctions. It collapsed to predicting `analysis` 
for almost everything and never predicted `ragebait` once.

The zero-shot Groq baseline, which had access to explicit label definitions 
at inference time, significantly outperformed the fine-tuned model (80% vs 
43.3%). This is the honest result: when your training data doesn't reflect 
real discourse, a well-prompted LLM with no training beats a fine-tuned model.

## Spec Reflection

**One way the spec helped:** Writing `planning.md` before collecting data 
forced me to define the hard edge case between `ragebait` and `reaction` 
before annotating 200 examples. Having that decision rule written down made 
labeling faster and more consistent.

**One way implementation diverged:** The spec assumed data would be collected 
from real community posts. The ragebait label made this impractical — it's 
a label that only makes sense in context, which makes it nearly impossible 
to collect and label reliably at scale from raw Reddit data. Choosing to 
include ragebait as a label was the right creative decision, but it forced 
a data collection approach that hurt model performance.

## AI Usage

1. **Label stress-testing:** Used Claude to generate boundary cases between 
`ragebait` and `hot_take` to tighten label definitions before annotation. 
Claude generated examples I couldn't cleanly classify, which revealed that 
my ragebait definition needed a more explicit decision rule about sincerity 
of belief.

2. **Data generation:** Used Claude to generate all 200 training examples 
(50 per label) based on label definitions from `planning.md`. Every example 
was reviewed manually before inclusion. AI generation was chosen because 
ragebait is context-dependent in real Reddit posts and nearly impossible to 
label reliably without knowing the specific moment and community norms. This 
is disclosed as the primary cause of the fine-tuning regression — AI-generated 
data lacks the textural diversity of real community posts.

3. **Failure analysis:** Used Claude to identify patterns in wrong predictions 
after training. Claude identified that ragebait written in declarative 
sentences was consistently misclassified as analysis, which was verified by 
reviewing the confusion matrix.
