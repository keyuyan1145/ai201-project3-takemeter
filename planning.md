# TakeMeter — planning.md

> Complete this document before writing any implementation code.
> Your label definitions, edge case analysis, and data plan are what you'll use to guide annotation and model evaluation — the more specific they are, the more reliable your classifier will be.
> Your planning.md will be reviewed as part of your submission.

---

## Community

- What community did you choose and why?
- Why is this community a good fit for a classification task?
- What makes the discourse varied enough to be interesting?

---

## Labels

- What are your 2–4 labels?
- Define each label in a complete sentence.
- Include 2 example posts per label.

---

## Hard Edge Cases

- What type of post will be genuinely ambiguous between two labels?
- How will you handle it when you encounter it during annotation?

---

## Data Collection Plan

- Where will you collect examples?
- How many per label?
- What will you do if a label is underrepresented after 200 examples?

---

## Evaluation Metrics

- Which metrics will you use to evaluate your model?
- Why are those the right metrics for this specific task?
- Accuracy alone is not enough — explain what else you need and why.

---

## Definition of Success

- What performance would make this classifier genuinely useful?
- What would you accept as "good enough" for deployment in a real community tool?

---

## AI Tool Plan

### Label Stress-Testing

- Which AI tool will you use and what will you give it as input?
- What will you ask it to generate, and at which label boundaries?
- How will you use the output to tighten your definitions before annotating?

### Annotation Assistance

- Will you use an LLM to pre-label examples before reviewing them yourself?
- If yes, which tool, and how will you track which examples were pre-labeled?

### Failure Analysis

- How will you use an AI tool to identify patterns in your wrong predictions?
- What will you look for, and how will you verify the patterns yourself?
