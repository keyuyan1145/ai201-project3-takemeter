# TakeMeter — r/nba Post Classifier

A fine-tuned text classifier that categorizes r/nba Reddit posts into four intent-based labels: `news`, `analysis`, `discussion`, and `entertainment`. Built as part of AI 201 Project 3.

---

## Project Overview

**Community:** r/nba — chosen because the same subreddit hosts four functionally distinct post types that coexist daily. Reporters drop trade news, analysts build arguments from stats, fans debate hypotheticals, and others post jokes and celebrations. The NBA Finals recently wrapped up, generating a dense snapshot of all four post types in a short window — making it a practical source for a balanced, timely dataset.

**Labels:**

| Label | Definition |
|-------|------------|
| `news` | A post whose value is a new, falsifiable fact that would be rendered false or meaningless if the underlying event turned out to be wrong. |
| `analysis` | A post where the poster's original synthesis or structured argument is the contribution — the facts are known, and the post has value only because the poster connected or reframed them non-obviously. |
| `discussion` | A post that exists to solicit the community's opinion — an open question, hypothetical, or bare assertion designed to attract pushback — whose value lives in the replies. |
| `entertainment` | A post whose value is entirely emotional or aesthetic — humor, awe, nostalgia — where a different outcome would not meaningfully change its appeal. |

**Label examples:**

- **news** — "[Charania] Just in: The Portland Trail Blazers are hiring Minnesota Timberwolves lead assistant Micah Nori as the franchise's next head coach, sources tell ESPN." / "[Charania] BLOCKBUSTER: The Milwaukee Bucks are trading franchise icon Giannis Antetokounmpo and Bobby Portis to the Miami Heat for Tyler Herro, Kel'el Ware, Jaime Jaquez Jr., Kasparas Jakucionis, 3 first-round picks..."
- **analysis** — "The Timberwolves have turned Karl-Anthony Towns, their 2024 first round pick, 2026 first round pick, and 2031 first round pick into Donte DiVincenzo, Ayo Dosunmu, and Joan Beringer." / "[Noh] My salary model has Trae Young at $115.1 million in value over the next four years, assuming an average of 70 games played / 34 mpg. That makes his $212 million contract underwater by $97 million."
- **discussion** — "Do you think LeBron and Curry were more important to the NBA than Ronaldo and Messi are to soccer?" / "Based on their current cores, which top prospects (2026) fit the Wizards and Jazz? If you are the Wizards, will you pick Dybantsa or Peterson?"
- **entertainment** — "Why doesn't Wemby simply grow a 3-foot flattop to help block shots without the risk of fouling?" / "Chuck watching Cardi B's halftime performance: 'I don't know if those are B's. They might be Cardi D's... She should change her name.'"

**Success criteria:** ≥70% overall accuracy and ~70% per-class accuracy (±5% tolerance) on a balanced test set.

---

## Hard Edge Cases

Three boundary pairs where the label is genuinely ambiguous, with the tiebreaker rule applied.

**News vs. Analysis:** A post that compiles known facts into a structured comparison. Individual facts are news, but original assembly and framing make it analysis. Tiebreaker: could it run as a wire headline with no byline? If yes → `news`. If the poster's reasoning is what makes it worth reading → `analysis`.
> *Resolved example:* "Giannis Antetokounmpo leaves Milwaukee as the Bucks' all-time leader in points, rebounds, assists, and blocks. He and Kevin Garnett with the Timberwolves are the only players in NBA history to lead a franchise in all four categories." — Strip the Garnett comparison and the stat list still stands alone as a headline. → **`news`**

**Analysis vs. Discussion:** A post that states a confident verdict with one supporting stat but no reasoning chain. The stat is bait, not evidence; the post wants argument, not agreement. Tiebreaker: does the post build a chain from evidence to conclusion? If yes → `analysis`. If it asserts without showing work → `discussion`.
> *Resolved example:* "Giannis has 1 playoff series win over the last 5 years. This trade will be a disaster for Miami. Not a fan!" — No reasoning chain; designed to attract pushback. → **`discussion`**

**News vs. Entertainment:** A post that contains a new fact but where the appeal comes entirely from framing or charm. Tiebreaker: would a different outcome or quote change the post's value? If yes → `news`. If the feeling works regardless of the specific result → `entertainment`.
> *Resolved example:* "Jalen Brunson to release children's book: 'I've been thinking a lot about how I can show kids who might not be the tallest or the fastest, that they, too, have greatness in them.'" — A different quote would change the appeal entirely; the feeling is the value. → **`entertainment`**

---

## Dataset

- **Source:** r/nba, collected by saving Reddit pages as PDF or HTML-tagged PDF and extracting post text via Claude. Text-only posts; image, video, and media-first posts were excluded.
- **Size:** 279 total examples. Final per-label distribution: `news` 74, `analysis` 55, `discussion` 84, `entertainment` 66.
- **Balancing:** Over-represented labels were trimmed to match the floor of smaller labels. The `analysis` label was supplemented with 20 AI-generated examples (10 unambiguous, 10 boundary cases) due to limited organic posts.
- **Annotation:** Dataset was pre-labeled by Claude using the finalized label definitions as a system prompt. A random sample was reviewed by a human annotator for agreement; ambiguous cases were labeled directly by hand. All examples carry an `llm_labeled` flag for auditability.

---

## Classification System Prompt

Used as the system prompt for baseline evaluation (zero-shot `llama-3.3-70b-versatile`, no fine-tuning) and dataset annotation pre-labeling (Claude).

```
You are classifying posts from r/nba.
Assign each post to exactly one of the following categories.

news: A post whose entire value is a new, falsifiable fact about a real event — a trade, signing, result, or statement — that would be rendered false or meaningless if the underlying fact turned out to be wrong.
Example: "[Charania] Just in: The Portland Trail Blazers are hiring Minnesota Timberwolves lead assistant Micah Nori as the franchise's next head coach, sources tell ESPN."

analysis: A post where the poster's original synthesis, reasoning, or structured argument is the contribution — the underlying facts are already known, and the post has value only because the poster connected, quantified, or reframed them in a non-obvious way.
Example: "[Noh] My salary model has Trae Young at $115.1 million in value over the next four years, assuming an average of 70 games played / 34 mpg. That makes his $212 million contract underwater by $97 million."

discussion: A post that exists primarily to solicit the community's opinion — an open question, a hypothetical, or a bare assertion designed to attract pushback — whose value lives in the thread it generates, not the post itself.
Example: "Do you think Lebron and Curry were more important to NBA than Ronaldo and Messi is to soccer? Obviously soccer is more popular even though it is a more boring sport. But IMO, Lebron and Curry meant more..."

entertainment: A post that contains no falsifiable claim, no structured argument, and no open question — its entire content is exhausted by the emotional or aesthetic experience of reading it.
Example: "Chuck watching Cardi B's halftime performance: 'I don't know if those are B's. They might be Cardi D's... She should change her name.'"

Respond with ONLY the label name.
Do not explain your reasoning.

Valid labels:
news
analysis
discussion
entertainment
```

---

## Model & Training

**Model:** `distilbert-base-uncased` fine-tuned on the labeled dataset.

**Hyperparameter adjustments:**

| Parameter | Default | Adjusted | Reason |
|-----------|:-------:|:--------:|--------|
| Epochs | 3 | 6 | At 3 epochs the loss barely moved (~1.4 throughout), indicating minimal learning. Increasing to 6 brought loss down to ~0.7 by the final epoch, showing meaningful convergence. |

---

## Evaluation Results

### Overall Accuracy

| Model | Overall Accuracy |
|-------|:---------------:|
| Baseline (`llama-3.3-70b-versatile`, zero-shot) | 71% |
| Fine-tuned (DistilBERT, 6 epochs) | **86%** (36/42) |

### Per-Class Accuracy

| Label | Baseline (Llama-3.3-70B) | Fine-Tuned (DistilBERT) |
|-------|:--------:|:----------:|
| `news` | <70% | 100% (11/11) |
| `analysis` | >70% | 89% (8/9) |
| `discussion` | >70% | 92% (11/12) |
| `entertainment` | >70% | 60% (6/10) |

*Baseline per-class figures are directional — exact numbers were not captured, only that `news` fell below 70% while the other three exceeded it.*

The fine-tuned model resolved the baseline's primary weakness (`news`) entirely, but introduced a new one: `entertainment` dropped to 60%. The two models effectively swapped their problem labels.

### Confusion Matrix (Fine-Tuned, Test Set)

|  | Predicted: news | Predicted: analysis | Predicted: discussion | Predicted: entertainment |
|--|:-:|:-:|:-:|:-:|
| **True: news** | **11** | 0 | 0 | 0 |
| **True: analysis** | 1 | **8** | 0 | 0 |
| **True: discussion** | 0 | 1 | **11** | 0 |
| **True: entertainment** | 2 | 1 | 1 | **6** |

Diagonal = correct predictions. `news` and `discussion` are near-perfect. `entertainment` accounts for 4 of the 6 total errors, spread across all three other labels.

Supplementary image: ![Confusion matrix](confusion_matrix.png)

### Sample Classifications

4 examples from the test set — 2 correct predictions, 2 wrong predictions, selected to show the model at its best and at its most common failure mode.

| # | Post (truncated) | True | Predicted | Confidence | Correct? |
|---|-----------------|:----:|:---------:|:----------:|:--------:|
| 1 | "[Fischer] Sacramento has already had conversations with Charlotte and Toronto, trying to see if there was an opportunity to move Domantas Sabonis..." | `news` | `news` | 0.68 | ✓ |
| 2 | "Something nobody is talking about: the Knicks' bench outscored the Spurs' bench 87-31 across the Finals. That's not a star gap — Wemby and Brunson basically cancelled out. It's a depth gap..." | `analysis` | `analysis` | 0.42 | ✓ |
| 3 | "[OC]: The New York/Dayton Renaissance - The little-known story of the all-Black, Black-owned team that tried to overcome corruption to integrate the NBA..." | `analysis` | `news` | 0.45 | ✗ |
| 4 | "Chris Jent (Knicks assistant) ALL-TIME NBA stats: 37 points, 16 rebounds, 8 assists. Jokic against the Timberwolves on 4/1/2022: 38 points, 19 rebounds, 8 assists." | `entertainment` | `analysis` | 0.44 | ✗ |

**Example 1 (correct — news):** The journalist attribution prefix `[Fischer]` and the direct report of team-to-team trade conversations are the clearest possible signal for `news` — the post has no interpretation, no argument, and would be rendered meaningless if the underlying conversations did not occur. The model's confidence of 0.68 is the highest among the correct sample, consistent with how unambiguous the form is.

**Example 2 (correct — analysis):** The post opens with "Something nobody is talking about," signals an original argument, then builds a quantified claim (87-31 bench differential) and draws a non-obvious conclusion (it's a depth gap, not a star gap). The model correctly identifies the poster's synthesis as the value, even at moderate confidence (0.42) — reflecting that the analytical structure is present but the post reads conversationally rather than formally.

**Example 3 (wrong — analysis → news):** The `[OC]` tag signals original research, and the post is a historical deep-dive with an editorial argument. But the narrative style — telling a chronological story about a real team — reads like reporting rather than synthesis. The model predicts `news` at 0.45 confidence, reflecting genuine uncertainty. This is the one non-`entertainment` failure in the set and shows the model struggles with long-form analytical posts that adopt a journalistic voice.

**Example 4 (wrong — entertainment → analysis):** The two-row stat table format triggers an `analysis` prediction at 0.44 confidence. The joke is the absurdity of the comparison (a journeyman's entire career vs. one Jokic game), but nothing in the text signals humor. This and Example 3 together illustrate the core `entertainment` failure mode: the model reads structure, not intent.

---

## Error Analysis

6 total errors across 42 test examples. 4 of 6 involve `entertainment`, making it the dominant failure mode. Three representative cases are analyzed below.

---

**Example 1 — entertainment predicted as news**
> *"Jalen Brunson and Josh Hart toss out first pitches at Yankee Stadium after securing the New York Knicks' first NBA championship in 53 years"*
> True: `entertainment` · Predicted: `news` · Confidence: 0.56

**Which labels are confused?** Entertainment → news (2 of 4 entertainment failures follow this pattern).

**Why is the boundary hard?** The post contains a real, falsifiable event in plain declarative language — structurally identical to a wire headline. The humor (NBA players at a baseball stadium post-title) is implicit and carries no linguistic marker the model can detect.

**Labeling or data problem?** Data problem. The labeling is correct — a different quote about the same championship would not carry the same charm. The model learned "real event + named players = news" as a shortcut because the training set lacks enough entertainment examples that are factually grounded.

**What would fix it?** More training examples of celebration and anniversary posts framed as plain facts — ironic juxtapositions, milestone cameos — so the model sees the factually-grounded entertainment pattern explicitly.

---

**Example 2 — entertainment predicted as analysis**
> *"Chris Jent (Knicks assistant) ALL-TIME NBA stats: 37 points, 16 rebounds, 8 assists. Jokic against the Timberwolves on 4/1/2022: 38 points, 19 rebounds, 8 assists."*
> True: `entertainment` · Predicted: `analysis` · Confidence: 0.44

**Which labels are confused?** Entertainment → analysis (1 case; low confidence signals the model is uncertain).

**Why is the boundary hard?** The post uses a two-column stat comparison — the structural format of genuine analysis. The joke is the absurdity of comparing a journeyman's entire career to one Jokic game, but nothing in the text signals humor.

**Labeling or data problem?** Data and definition problem. The labeling is correct. The training data likely has few examples of humor-through-stat-comparison, and the label definition of entertainment does not explicitly name absurdist comparisons as a sub-type.

**What would fix it?** Add entertainment examples that use stat or table structure as a vehicle for humor, and expand the label definition to name absurdist comparisons explicitly.

---

**Example 3 — entertainment predicted as discussion**
> *"Carmelo is the most unlucky athlete ever. Melo lost his nick name, his number, his city, his top10 scorer and his full name now. What else can he lose? It's just a joke guys."*
> True: `entertainment` · Predicted: `discussion` · Confidence: 0.69

**Which labels are confused?** Entertainment → discussion (high confidence — the model committed strongly to the wrong label).

**Why is the boundary hard?** The post ends with a question ("What else can he lose?"), the strongest syntactic signal for discussion. The joke disclaimer appears after the question, so the model has already committed. Rhetorical questions are indistinguishable from genuine discussion openers at the surface level.

**Labeling or data problem?** Partially data, partially definition. The labeling is correct — the post wants a reaction, not a reply. The tiebreaker rule ("does the post want a reply or a reaction?") captures this distinction conceptually but is not represented in enough training examples for the model to learn it.

**What would fix it?** Training examples of entertainment posts that use rhetorical question syntax — jokes ending with "right?", "anyone else?", or mock solicitations — so the model learns that a trailing question alone does not make a post a discussion.

---

## Model Reflection

**What the model captured:** The model learned the structural and lexical signals of `news`, `analysis`, and `discussion` reliably. Wire-style declarative sentences, stat-driven arguments, and question syntax are each consistent enough in form that fine-tuning on 50+ examples per label was sufficient to hit 89–100% per-class accuracy on those three. The model internalized the surface shape of those categories well.

**What the model missed:** The `entertainment` label was intended to capture posts whose value is emotional or aesthetic — which often means the entire meaning hinges on tone, irony, sarcasm, or a single phrase at the end that reframes everything before it. A post can spend three sentences presenting what looks like a factual comparison or a discussion prompt, and then one closing line ("It's just a joke guys", or an absurdist punchline) flips the entire intent. Humans process that reversal naturally; the model commits to a prediction based on the dominant structure of the first few sentences and never recovers.

This is a harder problem than the other label boundaries because entertainment does not have a consistent surface form — it borrows the structure of every other label as a vehicle for humor or celebration. The model learned to predict by form; the `entertainment` label requires predicting by intent, which the training data did not represent diversely enough to teach.

**What the model overfit to:** For `entertainment`, the model overfit to the absence of factual language, question syntax, or structured arguments as the signal for the label. When entertainment posts included any of those — a real event stated plainly, a rhetorical question, a stat comparison — the model defaulted to the label whose form matched, regardless of tone. The decision boundary it learned is essentially "if it looks like news/analysis/discussion, it is one" rather than "is the emotional register the point?"

**Data vs. labeling:** For `news`, `analysis`, and `discussion`, the wrong predictions are arguably borderline cases where the label could have gone either way — the model's prediction is defensible. Since the full dataset was LLM-pre-labeled and only a sample was human-reviewed, some of those labels may reflect annotation inconsistency rather than model error. For `entertainment`, the failures are more clearly a data coverage problem: the training examples did not include enough posts that use irony, sarcasm, rhetorical questions as punchlines, or factual framing for comedic effect. More diverse entertainment examples — explicitly covering plays on language and structural mimicry — are the primary fix.

---

## Spec Reflection

**Where the spec helped:** Defining the dataset size target and the evaluation criteria upfront — 200+ examples, roughly even per-class distribution, 70% accuracy floor — simplified every downstream decision. When extracting posts from the saved Reddit PDFs, it was immediately apparent that the `analysis` label was underrepresented because there was already a concrete target to measure against. Without that distribution requirement in the spec, the imbalance might not have surfaced until training, where it would have been much harder to diagnose and fix.

**Where the implementation diverged:** The spec assumed all examples would come from organic Reddit posts. In practice, the `analysis` label had too few organic examples to meet the 50-example floor even after exhausting the collected pages. The fallback — generating 20 synthetic examples with Claude — was not the original plan, but it was consistent with the spec's intent (balanced dataset, disclosed AI usage) even if the mechanism differed. The spec's early decision to require roughly even distribution is what made the gap visible quickly enough to address it before annotation rather than after.

---

## AI Usage Disclosure

### Instance 1 — Data Extraction and Dataset Balancing

**What I directed the AI to do:** Reddit r/nba pages were saved as PDFs and shared with Claude to extract individual post texts for the initial dataset. After extraction, some labels had significantly more examples than others. I directed Claude to identify and remove excess examples from the over-represented labels to bring per-class counts closer to the 50-example target.

**What it produced:** A trimmed, more balanced dataset.

**What I changed or overrode:** No individual examples were overridden — removals were accepted as-is. The decision to trim rather than augment was a deliberate choice to preserve the organic quality of collected posts.

---

### Instance 2 — Synthetic Example Generation for Underrepresented Label

**What I directed the AI to do:** The `analysis` label had too few organic examples. I directed Claude to generate 20 synthetic r/nba posts for the label — 10 unambiguous and 10 boundary cases sitting on the edge with `news` or `discussion`.

**What it produced:** 20 generated posts in the analysis format, mixing clear and boundary cases.

**What I changed or overrode:** A sample was manually reviewed for quality. No examples were rejected — all 20 were accepted. Generated examples are flagged `llm_labeled = True` in the dataset.

---

### Instance 3 — Annotation Pre-Labeling

**What I directed the AI to do:** After finalizing the label definitions and tiebreaker rules, Claude pre-labeled the full dataset using the complete definitions as a system prompt, assigning one label per post with a confidence signal for ambiguous cases.

**What it produced:** A first-pass label for each example, stored in the `ai_prelabel` column.

**What I changed or overrode:** A random sample was reviewed against a human label for agreement. Ambiguous cases flagged by Claude were labeled directly by hand. The `ai_prelabel` column is retained in the dataset so the agreement rate between Claude and human annotation is fully auditable.
