---
layout: post
title: "Getting Started with Quantitative Trading from Scratch"
subtitle: "A single post that exercises headings, prose, callouts, code blocks, tables, lists, math, and tags."
date: 2026-05-11 00:30:00 +0800
author: Jiawei Zhang
reading_time: 6 min
tags:
  - Demo
  - AI4Science
  - Protein Modeling
  - Engineering
excerpt: "A visual test post for the blog theme, including the common elements used in technical and research writing."
---

This article is a living style sample for the blog. It is meant to make the theme easy to inspect before real posts start accumulating.

The target feeling is simple: readable enough for long research notes, structured enough for engineering logs, and calm enough that the content gets the attention.

## Section Heading

A good post often starts with a short framing paragraph. For example, a protein modeling note might explain the dataset, the model checkpoint, the evaluation metric, and the result that changed your mind.

### Smaller Heading

Use smaller headings for local arguments, implementation notes, ablation results, or paper-reading observations.

## Callout

<div class="blog-callout">
  <strong>Working note.</strong>
  Use callouts for assumptions, warnings, experiment settings, or small conclusions that should survive a quick skim.
</div>

## Quote

> Good research notes should preserve the path of thought, not only the polished conclusion.

Quotes are useful when reviewing papers, recording a claim, or separating an interpretation from the original source.

## Lists

Unordered lists are useful for observations:

- Tokenization can hide important biochemical structure.
- Retrieval helps only when the retrieved evidence is specific enough.
- A small evaluation set is better than no evaluation set, but it should be labeled honestly.

Ordered lists are useful for procedures:

1. Define the question.
2. Collect the smallest useful dataset.
3. Run a baseline.
4. Record the failure modes before tuning.

## Code

Inline code like `model.eval()` should feel quiet inside a sentence. Larger blocks should be easy to scan.

```python
from pathlib import Path

def load_sequences(path: str) -> list[str]:
    records = Path(path).read_text().splitlines()
    return [line.strip() for line in records if line and not line.startswith(">")]

sequences = load_sequences("examples.fasta")
print(f"Loaded {len(sequences)} sequences")
```

## Table

Tables should stay compact and readable.

| Experiment | Input | Metric | Result |
| --- | --- | ---: | ---: |
| Baseline encoder | sequence | AUROC | 0.781 |
| Structure-aware model | sequence + structure | AUROC | 0.824 |
| Retrieval-augmented note | sequence + motifs | AUROC | 0.836 |

## Math

Inline math such as $p(y \mid x)$ should render naturally. Display math can be used for short derivations:

$$
L = - \sum_i y_i \log p_i
$$

## Divider

---

The rest of this paragraph exists to check spacing after a divider. The page should still feel calm, with enough contrast for technical reading and enough whitespace to avoid becoming a wall of text.
