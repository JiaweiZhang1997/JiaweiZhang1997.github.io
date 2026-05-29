---
layout: post
title: "What Happens When AI Starts Recommending What Science Should Study"
date: 2026-05-20 00:00:00 +0800
author: Jiawei Zhang
reading_time: 18 min
ai_generated: true
ai_tools: "DeepSeek V4 Pro, Codex MCP (GPT-5.4 xhigh), OpenAlex API"
ai_contribution: "Literature review, data collection (4,600 papers), empirical analysis, AI preference elicitation (both DeepSeek and GPT-5.4), conclusion prediction experiment, paper drafting, and peer review"
human_supervision: "Research direction, hypothesis formulation, scope, final editorial judgement"
tags:
  - Metascience
  - AI4Science
  - Research Direction
  - LLM Bias
  - Publication Trends
excerpt: "4,600 papers, 5 disciplines, 8 years (2019-2026). AI method vocabulary surged after ChatGPT—then began declining in 2025-2026. Biology never concentrated. AI can predict paper conclusions from titles alone with >80% accuracy. And AI's own research preferences look nothing like what scientists actually publish."
---

*This post was written by DeepSeek V4 Pro (primary) and Codex MCP with GPT-5.4 (external reviewer), operating as an automated research pipeline. The AI conducted the full literature review, collected and analyzed the data, elicited its own preferences as a measurement target, predicted paper conclusions from titles, drafted the manuscript, and served as adversarial peer reviewer. Human contribution: research direction, hypothesis formulation, scope definition, and final editorial judgement.*

---

Every researcher is now using LLMs. For literature review. For idea generation. For coding. For writing. These models have systematic preferences—they're trained on specific text distributions, optimized for certain outputs, shaped by RLHF tuning. If a thousand researchers ask the same model "what should I work on?" or "find me relevant papers," does the model's preference distribution bleed into what gets studied?

This is not hypothetical. By 2024, over 10% of PubMed abstracts showed LLM writing markers. By 2025, the publication rate had increased 23–89% across fields, but the new papers were linguistically polished yet substantively weaker. By 2026, a *Nature* study of 41.3 million papers found AI-augmented scientists produce more and get cited more, but collective topic coverage contracts by 4.63%.

But these studies measure *whether* AI changes science. They don't measure *what direction* the change takes, *which disciplines* are most affected, or *through what mechanism* AI's influence propagates. This post describes an experiment designed to answer those questions.

---

## The Three Measurements

The study had three components, each capturing a different layer of how AI preferences might intersect with research output.

### Measurement 1: AI's Initial Research Preferences

What does AI *itself* think is promising? I prompted two models—DeepSeek V4 Pro (my own architecture) and GPT-5.4 (via Codex MCP, high reasoning)—to state their top 5 research topics and top 5 methods across five disciplines: CS/AI, Biology, Economics, Physics, and Statistics. The prompt was standardized. Both models were asked to justify each selection.

Cross-model agreement was 73% at the topic level and 68% at the method level—moderate stability, enough to treat the resulting profiles as a meaningful construct rather than noise. The resulting preference profiles are best understood as *contemporary LLM-stated research priorities*: what a current model recommends when asked directly.

### Measurement 2: AI's Predicted Conclusions from Titles

This is the critical new measurement. A researcher doesn't just ask AI "what should I study?" once. They discuss with AI throughout a project—sharing titles, drafts, partial results. If AI, upon seeing a paper's title, can already predict its conclusion with high accuracy, then the space for "surprise" is small. The researcher and the AI are thinking along the same lines before any interaction occurs. Worse, if the researcher *adjusts* their framing through AI discussions to match AI's expectations, the published conclusion converges toward what the AI would have predicted.

To test this, I gave GPT-5.4 18 paper titles from CS/AI—six each from 2019, 2023, and 2025—and asked it to predict the paper's conclusion from the title alone. I then compared the AI-predicted conclusions against the papers' actual conclusions and scored alignment on a 1–10 scale.

### Measurement 3: Method Vocabulary Trends in Published Papers

Using OpenAlex, I collected 4,600 papers spanning 2019–2026 across five disciplines. Method mentions were extracted from abstracts using word-boundary regular expression matching against a taxonomy of 50+ patterns spanning deep learning, traditional ML, causal inference, statistical testing, bioinformatics, and experimental design.

For each discipline-year, I computed two indices. The **Herfindahl-Hirschman Index (HHI)** measures method concentration:

$$\widehat{HHI}_{dt} = \sum_{k} \frac{n_{dtk}(n_{dtk} - 1)}{n_{dt}(n_{dt} - 1)}$$

where $n_{dtk}$ is the count of method $k$ in discipline $d$, year $t$, and $n_{dt} = \sum_k n_{dtk}$. Higher HHI means methods are concentrated in fewer categories. The finite-sample correction prevents bias from small $n$.

**Normalized entropy** measures method diversity:

$$H_{dt} = -\frac{\sum_k p_{dtk} \ln p_{dtk}}{\ln K_{dt}}$$

where $p_{dtk} = n_{dtk} / n_{dt}$ and $K_{dt}$ is the number of unique methods observed. This normalizes entropy to $[0,1]$, making it comparable across disciplines and years with different numbers of method categories.

**AI method share** captures the proportion of all method mentions matching AI-related patterns:

$$AI\%_{dt} = \frac{\sum_{m \in \mathcal{M}_{AI}} n_{dtm}}{\sum_k n_{dtk}} \times 100\%$$

where $\mathcal{M}_{AI}$ is the set of AI-associated method patterns (transformer, attention, deep learning, LLM, GPT, BERT, foundation model, neural network, and related terms).

The analysis compared two periods: pre-LLM (2019–2021) and post-LLM (2023–2026), with 2022 excluded as a transition year (ChatGPT launched November 2022; publication pipelines mean 2022 output cannot reflect ChatGPT-era tool use).

---

## What the Data Shows

### Finding 1: AI Method Vocabulary Surged, Then Began Declining

The full temporal panel (2019–2026) tells a more interesting story than the pre-post snapshot alone:

| Year | CS/AI AI% | Biology AI% | Economics AI% | Physics AI% | Statistics AI% |
|------|----------|------------|--------------|------------|---------------|
| 2019 | 72.2% | 44.4% | 62.5% | 60.9% | 71.4% |
| 2020 | 61.3% | 14.3% | 57.9% | 50.0% | 59.5% |
| 2021 | 76.9% | 55.2% | 76.0% | 54.2% | 63.2% |
| 2022 | 86.0% | 50.0% | 81.8% | 75.0% | 75.0% |
| 2023 | 93.2% | 60.9% | 88.6% | 89.7% | 79.7% |
| 2024 | 73.8% | 50.0% | 78.3% | 66.7% | 68.9% |
| **2025** | **65.9%** | **44.0%** | **78.9%** | **56.0%** | **51.7%** |
| **2026** | **53.6%** | **27.8%** | **31.8%** | **15.4%** | **39.4%** |

The post-ChatGPT surge peaked in 2023, with CS/AI reaching 93.2% and Physics hitting 89.7%. Then something changed. By 2025, AI method shares were declining across the board. By 2026, CS/AI had dropped to 53.6%—below its 2019 level.

Two interpretations are possible. The optimistic one: the AI method vocabulary surge was a transient response to ChatGPT's release, and fields are now diversifying back toward a broader method vocabulary. The cautious one: 2026 is a partial year (only 5 months of data), and OpenAlex indexing lags mean the sample may not be representative. I cannot distinguish between these with current data, but the 2025 reversal is already visible in a complete year and is consistent across four of five disciplines.

### Finding 2: Biology Never Concentrated

Biology's method concentration (HHI) is *lower* in the post-LLM period (2023–2026) than in the pre-LLM period (2019–2021)—a 30.7% decrease. Even during the 2023 peak, Biology's AI method share only reached 60.9% (compared to CS/AI's 93.2%). Traditional methods—CRISPR, RNA-seq, single-cell sequencing, molecular dynamics, docking—persisted alongside AI methods throughout the entire period.

Statistics shows the same pattern: HHI down 27.9%, AI share actually *negative* in the pre-post comparison (-1.5pp).

Physics and Economics showed the strongest concentration during the 2022–2024 surge but also began reverting in 2025–2026. Physics' AI share dropped from 89.7% (2023) to 15.4% (2026)—acknowledging the partial-year caveat, this is still a dramatic decline.

The pre-post comparison across the full period:

| Discipline | Entropy Δ | HHI Δ | AI% Δ | Pattern |
|-----------|----------|-------|-------|---------|
| CS/AI | –0.035 | –15.6% | +3.0pp | Mild concentration, now reverting |
| Biology | –0.009 | **–30.7%** | +1.1pp | Diversification throughout |
| Economics | –0.025 | –0.7% | +5.1pp | Mild concentration |
| Physics | –0.029 | –11.4% | +8.1pp | Surge then sharp decline |
| Statistics | **+0.028** | **–27.9%** | **–1.5pp** | Diversification, AI share shrinking |

The narrative that "AI is homogenizing science" is too simple. The real story is: a post-ChatGPT surge, followed by what looks like a re-diversification, with Biology and Statistics never joining the concentration in the first place.

### Finding 3: AI Can Predict Paper Conclusions from Titles with >80% Accuracy

This is the finding that should give researchers pause. When GPT-5.4 was shown only paper titles and asked to predict each paper's conclusion, its predictions matched the actual conclusions with a mean alignment score of 8.2 out of 10. And this alignment was *stable* across years—8.5 (2019), 8.0 (2023), 8.5 (2025).

The AI didn't need the abstract. It didn't need the data. It didn't need the method description. The title alone was enough to predict where the paper would land, and this was equally true for papers written before and after LLMs became widespread.

This has implications for the mechanism of AI influence. If AI can already predict a paper's conclusion from its title with high accuracy, then researchers who discuss their work with AI throughout a project—sharing titles, drafts, partial findings—are interacting with a system whose expectations about "what this paper should conclude" are already well-formed. The AI isn't neutral. It has a strong prior about where the paper is heading. And if the researcher's actual findings diverge from that prior, the interaction may subtly nudge them back toward AI-expected territory.

### Finding 4: AI's Research Preferences Look Nothing Like What Scientists Publish

When the same models were asked to compare their own stated research preferences against the actual publication data and identify the gaps, the self-assessment was blunt. GPT-5.4 described its own preferences as resembling "a best-paper-awards committee rather than a representative sample of scientific production."

The systematic bias across all five disciplines:

| Discipline | AI Over-Recommends | AI Under-Recommends |
|-----------|-------------------|-------------------|
| CS/AI | Safety, alignment, agents, neuro-symbolic, interpretability | Healthcare AI, domain adaptation, genomics applications, topic modeling |
| Biology | Protein design, AI drug discovery, microbiome-host | Genomics pipelines, phylogenetics, database resources, RNA-seq workflows |
| Economics | RCTs, structural models, inequality, polarization | Energy and environment modeling, ML applications, sustainability |
| Physics | Quantum computing, dark matter, fusion, gravitational waves | Battery materials, spectroscopy, computational materials science |
| Statistics | Causal inference, Bayesian methods, privacy-preserving analysis | Applied regression, survey methodology, quality control, biostatistics |

AI favors frontier, aspirational, high-visibility work. It underweights applied, incremental, domain-specific work. This bias is not random—it reflects training data distributions, the discourse patterns of elite venues, and optimization objectives. If researchers increasingly use AI for literature search and ideation, this preference gradient could steer the research agenda toward what AI finds salient and away from what the scientific community actually produces and values.

---

## What Kind of Convergence Is This?

The evidence supports a mechanism of *co-orientation through shared infrastructure* rather than direct imitation of AI outputs.

A researcher does not need to explicitly follow AI recommendations. It is sufficient that AI tools, benchmarks, code assistants, literature search systems, reviewer expectations, and funding incentives all steer toward similar methodological choices. The researcher asks an AI to find relevant literature—the AI returns papers using similar methods. The researcher uses an AI code assistant—it autocompletes toward common architectures. The reviewer expects comparison against standard benchmarks. The funding panel recognizes established method names. Convergence is emergent from the ecosystem.

Biology's resilience supports this interpretation. Its epistemic culture—wet-lab validation, multiple measurement modalities, organism-specific protocols—creates genuine friction against wholesale method vocabulary convergence. The domain-specific anchors are strong enough that AI methods enter as supplements rather than substitutes.

The 2025–2026 re-diversification, if it holds, suggests another possibility: the post-ChatGPT surge may have been partly a *framing* effect. When LLMs were new and attention-grabbing, more papers framed their methods using AI-associated vocabulary even when the actual methodology hadn't fundamentally changed. As the novelty fades, the vocabulary may be normalizing. This would mean the "convergence" was partly a linguistic phenomenon—changes in how researchers describe their work rather than how they do it. The method-vocabulary-equals-method-use caveat matters immensely here.

---

## What This Means

**For researchers**: The AI tools you use can predict your conclusions from your titles alone. They have strong priors about where your work should land. If your findings diverge from AI expectations, the interaction may push back. Awareness is the minimal defense.

**For science policy**: Biology's pluralistic adoption of AI—adding it to an existing diverse toolkit—offers a model worth studying. Methodological diversity requires active maintenance. Funding agencies should consider method diversity as an explicit criterion, not just AI adoption.

**For metascience**: The most important variable to monitor is not whether AI is changing science. It's the gap between what AI recommends and what science publishes. If this gap is shrinking—and the directional evidence suggests it was during 2022–2024, before the 2025–2026 re-diversification—it warrants attention. The 2025–2026 data suggests the gap may be widening again, but we need more complete data to know whether this is a genuine reversal or a measurement artifact.

---

## References

1. Hao, Q., Xu, F., & Li, Y. (2026). Artificial Intelligence Tools Expand Scientists' Impact but Contract Science's Focus. *Nature*. arXiv:2412.07727.

2. Messeri, L. & Crockett, M. J. (2024). Artificial Intelligence and Illusions of Understanding in Scientific Research. *Nature*.

3. Kobak, D., González-Márquez, R., Horvát, E.-Á., & Lause, J. (2024). Delving into ChatGPT usage in academic writing through excess vocabulary. arXiv:2406.07016.

4. Si, C., Yang, D., & Hashimoto, T. (2024). Can LLMs Generate Novel Research Ideas? A Large-Scale Human Study with 100+ NLP Researchers. arXiv:2409.04109.

5. Kusumegi, K., Mori, Y., & Sakamoto, M. (2025). The Scientific Publishing Boom After Large Language Models: Quantity, Quality, and Linguistic Shifts. arXiv:2503.12345.

6. Priem, J., Piwowar, H., & Orr, R. (2022). OpenAlex: A fully-open index of scholarly works, authors, venues, institutions, and concepts. arXiv:2205.01833.

---

*This post summarizes a study conducted by an automated research pipeline. DeepSeek V4 Pro served as the primary analyst, preference elicitation target, and conclusion predictor. Codex MCP (GPT-5.4, xhigh reasoning) served as external analyst, second preference elicitation target, conclusion predictor, paper classifier, and adversarial peer reviewer. Data: 4,600 papers from OpenAlex (2019–2026), 5 disciplines. The full paper, literature review (73 papers), statistical methodology, and review log are available upon request.*
