# TakeMeter — planning.md

> Complete this document before writing any implementation code.
> Your label definitions, edge case analysis, and data plan are what you'll use to guide annotation and model evaluation — the more specific they are, the more reliable your classifier will be.
> Your planning.md will be reviewed as part of your submission.

---

## Community
- What community did you choose and why?
- Why is this community a good fit for a classification task?
- What makes the discourse varied enough to be interesting?

**Community chosen: r/nba**

As a basketball fan, r/nba was a natural choice — it's a community I already read and understand well enough to annotate accurately without needing to look up context. The NBA Finals just wrapped up, which spiked a surge of post-season recap posts, trade speculation, and early 2026–27 offseason questions — giving a rich and timely snapshot of all four post types in a short window.

It is a strong fit for a classification task because the same subreddit hosts four functionally distinct post types that coexist daily: reporters drop trade news, analysts build arguments from stats, fans debate hypotheticals, and others post highlight clips or jokes. The intent behind each post is meaningfully different, making the labels reflect real behavioral categories rather than arbitrary splits.

The discourse is varied enough to be interesting because the NBA calendar creates natural topic clusters — trade deadline, playoffs, offseason free agency, draft — but the post *type* cuts across all of them. A trade rumor, a cap-space breakdown, a "who wins this series" debate, and a Shaq meme can all appear in the same hour on the same topic. That overlap between topic and intent is what makes the classification non-trivial and worth building a model for.

---

## Labels

- What are your 2–4 labels?
- Define each label in a complete sentence.
- Include 2 example posts per label.
  
Four labels are used. Each is defined by the primary communicative intent of the post, not its topic.

### Label 1: News

A post whose entire value is a new, falsifiable fact about a real event — a trade, signing, result, or statement — that would be rendered false or meaningless if the underlying fact turned out to be wrong.

**Example post A:**
> "[Charania] Just in: The Portland Trail Blazers are hiring Minnesota Timberwolves lead assistant Micah Nori as the franchise's next head coach, sources tell ESPN."

**Example post B:**
> "[Charania] BLOCKBUSTER: The Milwaukee Bucks are trading franchise icon Giannis Antetokounmpo and Bobby Portis to the Miami Heat for Tyler Herro, Kel'el Ware, Jaime Jaquez Jr., Kasparas Jakucionis, 3 first-round picks..."

---

### Label 2: Analysis

A post where the poster's original synthesis, reasoning, or structured argument is the contribution — the underlying facts are already known, and the post has value only because the poster connected, quantified, or reframed them in a non-obvious way.

**Example post A:**
> "The Timberwolves have turned Karl-Anthony Towns, their 2024 first round pick, 2026 first round pick, and 2031 first round pick into Donte DiVincenzo, Ayo Dosunmu, and Joan Beringer."

**Example post B:**
> "[Noh] My salary model has Trae Young at $115.1 million in value over the next four years, assuming an average of 70 games played / 34 mpg. That makes his $212 million contract underwater by $97 million."

---

### Label 3: Discussion

A post that exists primarily to solicit the community's opinion — an open question, a hypothetical, or a bare assertion designed to attract pushback — whose value lives in the thread it generates, not the post itself.

**Example post A:**
> "Do you think Lebron and Curry were more important to NBA than Ronaldo and Messi is to soccer? Obviously soccer is more popular even though it is a more boring sport. But IMO, Lebron and Curry meant more..."

**Example post B:**
> "Based on their current cores. Which Top Prospects (2026) fit Wizards and Jazz?... If you are the Wizards. Will you pick Dybantsa or Peterson? They have Trae and AD. But both always have injury concerns..."

---

### Label 4: Entertainment

A post that contains no falsifiable claim, no structured argument, and no open question — its entire content is exhausted by the emotional or aesthetic experience of reading it.

**Example post A:**
> "Why doesn't Wemby simply grow a 3-foot flattop to help block shots without the risk of fouling? When standing flat on his feet, Wemby is about 32 inches from the rim..."

**Example post B:**
> "Chuck watching Cardi B's halftime performance: 'I don't know if those are B's. They might be Cardi D's... She should change her name.'"

---

## Hard Edge Cases

- What type of post will be genuinely ambiguous between two labels?
- How will you handle it when you encounter it during annotation?

**News vs. Analysis:** A post that compiles known facts into a structured comparison or roster view. Individual facts are News, but original assembly and framing is the poster's contribution. Ask: could it run as a wire headline with no byline? If yes → News. If the poster's reasoning is what makes it worth reading → Analysis.

> *Example:* "Giannis Antetokounmpo leaves Milwaukee as the Bucks' all-time leader in points, rebounds, assists, and blocks. He and Kevin Garnett with the Timberwolves are the only players in NBA history to lead a franchise in all four categories." — Strip the Garnett comparison and the stat list still stands alone. → **News**

**Analysis vs. Discussion:** A post that states a confident verdict with one supporting stat but no reasoning chain connecting them. The stat is bait, not evidence; the post wants argument, not agreement. Ask: does the post build a chain from evidence to conclusion? If yes → Analysis. If it asserts without showing work → Discussion.

> *Example:* "Giannis has 1 playoff series win over the last 5 years. This trade will be a disaster for Miami. I can't stress how bad this Giannis trade is for the Heat.. This is a playin team at best with a weak bench. Not a fan!" → **Discussion**

**News vs. Entertainment:** A post that contains a new fact but where the fact is trivial and the appeal comes entirely from its framing or the person's charm. Ask: would a different outcome or quote change the post's value? If yes → News. If the feeling works regardless of the specific result → Entertainment.

> *Example:* "Jalen Brunson to release children's book: 'I've been thinking a lot about how I can show kids who might not be the tallest or the fastest, that they, too, have greatness in them.'" — A different quote would change the appeal entirely. The feeling is the value. → **Entertainment**

**Handling ambiguous cases during annotation:** Apply the tiebreaker rules in order — News/Analysis first, then Analysis/Discussion, then News/Entertainment, then Discussion/Entertainment. When two tiebreakers conflict, default to the category whose definition the post satisfies most completely. Force a single label every time; no "both" or "other." Any post that takes more than 30 seconds to resolve gets logged as an edge case with a note on which rule was applied.

---

## Data Collection Plan

- Where will you collect examples?
- How many per label?
- What will you do if a label is underrepresented after 200 examples?

All examples will be collected from r/nba. No web scraping tool will be used — instead, Reddit pages will be saved as PDF or HTML-tagged PDF and shared directly with Claude for text extraction. Only text-based posts are included — any post that leads with an image, video, or embedded media is excluded, since the classifier will operate on text alone and media-first posts have a different signal profile.

The target is 200+ examples with at least 50 per label, keeping the distribution roughly even across all four categories. If any label remains underrepresented after exhausting available organic posts, AI-generated examples will fill the gap — a mix of unambiguous cases (to round out the count) and ambiguous boundary cases (to stress-test the annotator and the eventual model). Generated examples will be flagged in the dataset so their origin is transparent.

---

## Evaluation Metrics

- Which metrics will you use to evaluate your model?
- Why are those the right metrics for this specific task?
- Accuracy alone is not enough — explain what else you need and why.

**Primary metric: overall accuracy.** The target is 70% — meaning the model assigns the correct label on at least 7 out of 10 posts across all four categories combined.

**Secondary metric: per-class accuracy.** Overall accuracy alone is not enough because a model could hit 70% by doing well on whichever label happens to be most frequent while completely failing on a rarer one. Per-class accuracy is reported for each label individually, with a target of ~70% per class and a tolerance of ±5% — so any individual label should fall between 65% and 75%. A model whose per-class accuracy swings wider than that is learning the majority class distribution, not the actual distinctions between labels, and would be unreliable in practice.

---

## Definition of Success

- What performance would make this classifier genuinely useful?
- What would you accept as "good enough" for deployment in a real community tool?

The classifier is considered successful if it reaches 70% overall accuracy and approximately 70% per-class accuracy on a held-out test set drawn from the same distribution as training — with no individual label falling below 65% or exceeding 75%. Both the train and test splits will be kept roughly even across all four labels so that neither accuracy figure is inflated by class imbalance.

At this threshold the model is making a correct call 7 out of 10 times on every category, not just the easy ones, which is useful enough to drive a soft community tool — such as a suggested auto-flair or a post-routing filter — where a human can review or override the prediction. Below this threshold, the per-class variance would be too high to trust the model on any single label consistently.

---

## AI Tool Plan

### Label Stress-Testing

- Which AI tool will you use and what will you give it as input?
- What will you ask it to generate, and at which label boundaries?
- How will you use the output to tighten your definitions before annotating?

**Tool:** Claude only.

Example posts will be fed to Claude along with the four label definitions. Claude is asked to assign a label for each post and, if the post sits at a boundary, to explain why it is ambiguous and what reasoning it used to land on a final label. Posts Claude flags as ambiguous are then handed to a human annotator for independent labeling. When the human and Claude disagree — or when Claude's reasoning reveals a gap in the definition — the label description is updated to close that gap. This loop runs before any large-scale annotation begins, so ambiguous examples sharpen the definitions rather than pollute the dataset.

### Annotation Assistance

- Will you use an LLM to pre-label examples before reviewing them yourself?
- If yes, which tool, and how will you track which examples were pre-labeled?

---

**Tool:** Claude only. All 200 examples will be pre-labeled by Claude before any human review begins.

Each example in the dataset will carry an `llm_labeled` flag (boolean) indicating whether the label was assigned by Claude. Pre-labeled examples are not all reviewed individually — instead, a random sample is pulled from the `llm_labeled = True` group to check agreement between Claude's label and a human label, giving an estimate of Claude's reliability on this task. Examples that Claude marked as ambiguous during pre-labeling are not sampled randomly — they are handed directly to a human annotator for labeling, since those are the cases where Claude's label is least trustworthy.

### Failure Analysis

- How will you use an AI tool to identify patterns in your wrong predictions?
- What will you look for, and how will you verify the patterns yourself?

---

**Tool:** Claude only.

After evaluation, all misclassified examples — with their predicted label and expected label — will be passed to Claude at once. Claude will compare the two labels across the full set of wrong predictions and summarize the directional patterns it finds: which label pairs are being confused most often, whether the errors run consistently in one direction (e.g., Analysis predicted as Discussion more than the reverse), and any surface-level linguistic or structural features shared by the misclassified posts. The summary will be used to inform the label analysis section of the writeup and, if patterns are strong enough, to revisit label definitions or annotator guidance before any retraining.
