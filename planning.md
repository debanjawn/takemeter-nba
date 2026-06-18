# TakeMeter NBA — Planning Document

## Community
I chose r/nba because I'm an NBA fan and read the comments constantly. 
The discourse is extremely varied — you get detailed statistical breakdowns 
sitting right next to pure emotional reactions and obvious ragebait. I added 
a `ragebait` label specifically because it's a real and distinct phenomenon 
on r/nba that I see constantly (people know they can't seriously argue Jalen 
Brunson can't compete, but they post it anyway to get a rise out of people).

## Labels

**`analysis`** — The post makes a structured argument backed by statistics, 
historical comparison, or tactical observation. Evidence is specific and verifiable.
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

**`reaction`** — An immediate emotional response to a specific event. 
Little to no argument — the post is expressing a feeling in the moment.
- "NO WAY HE HIT THAT LMAOOOO"
- "GIANNIS JUST SENT THAT INTO THE THIRD ROW"

**`ragebait`** — A post that expresses a reaction but the claim is so 
exaggerated it couldn't be a sincere belief — it exists purely to provoke 
a response or just be outrageous.
- "Jalen Brunson looks like he's in second grade."
- "Trade Chet for an air fryer, him not being here is better."

## Hard Edge Cases
The hardest boundary is `ragebait` vs `reaction`. Both can be emotional and 
lack argument.

**Decision rule:** If the post expresses a genuine feeling about something 
that happened, label it `reaction`. If the claim is so exaggerated it 
couldn't be a sincere belief and exists purely to provoke or be outrageous, 
label it `ragebait`.

## Data Collection Plan
- Source: r/nba comments and posts via Reddit (append .json to any Reddit 
URL to get raw data)
- Target: 50 examples per label (200 total)
- If a label is underrepresented after 200 examples, I will specifically 
search for posts that fit that label (e.g. filter by game threads for 
more reactions, search controversial posts for ragebait)

## Evaluation Metrics
I will use per-class F1 score in addition to overall accuracy. Accuracy 
alone is not enough because if one label dominates the dataset, a model 
can score high just by predicting that label constantly. F1 balances 
precision and recall for each class, which matters here because all 4 
labels should be detected reliably, not just the most common one.

Success threshold: 85% overall accuracy on the test set, with no single 
class F1 below 0.70.

## Definition of Success
A classifier that is genuinely useful and fun — someone should be able to 
paste any r/nba comment and get a label that feels right. Given that 
`ragebait` is somewhat subjective, I accept that this label may have lower 
F1 than the others. The product doesn't need to be perfect, it needs to be 
right often enough that it feels accurate to someone who reads r/nba regularly.

## AI Tool Plan

**Label stress-testing:** Give Claude my label definitions and edge case 
rules and ask it to generate boundary cases I can't cleanly label — use 
this to tighten definitions before annotating 200 examples.

**Annotation assistance:** May use an LLM to pre-label a batch of examples 
before reviewing them myself. Will review and correct every label. Will 
disclose in AI usage section if used.

**Failure analysis:** After training, paste misclassified examples into 
Claude and ask it to identify common patterns. Will verify patterns myself 
before including in the evaluation report.
