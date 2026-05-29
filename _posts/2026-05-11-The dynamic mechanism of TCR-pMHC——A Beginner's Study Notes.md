---
layout: post
title: "The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes"
subtitle: "T细胞免疫学常识以及激活机制学习笔记."
date: 2026-05-11 00:30:00 +0800
author: Jiawei Zhang
reading_time: 55 min
tags:
  - Biology
  - 免疫
  - TCR-pMHC
  - AI4Science
excerpt: "T细胞到底是如何被激活的呢？这篇笔记从TCR-pMHC结构、抗原呈递、激活实验、动态机制、杀伤流程、肿瘤微环境和建模需求几个层面做一个尽量完整的入门梳理。"
---

T细胞（T cell），是人体免疫系统中至关重要的一部分，属于适应性免疫系统的核心主力。它们就像是人体内的“特种部队”，不仅能精准识别并摧毁被病毒感染的细胞或肿瘤细胞，还能指挥其他免疫细胞协同作战。

误打误撞开始做 AI4Science 之后，我越来越强烈地感觉到理解课题的重要性，特别是在数据不足的领域。如果想要真正解决问题，必须理解其中的机制，通过先验知识约束可能的搜索空间，完成更优雅的工作，得到更优的结果。

和文本、图像、视频、具身智能等领域相比，免疫学和蛋白相关任务的数据通常更贵、更小、更异质，测量噪声也更高。T cell activation 这类问题尤其明显：一个看似简单的“是否激活”标签，背后混合了抗原加工、HLA 呈递、TCR 序列、三维结构、细胞状态、共刺激、实验体系和微环境等多层因素。

因此，人类先验不是“可有可无的解释”，而是帮助模型缩小搜索空间、定义合理输入输出、发现标签偏差的重要工具。为了更好地处理 TCR-pMHC binding，更准确地说，是 **T cell activation** 任务，我必须理解其中的背景知识和关键机制，于是有了这篇学习笔记，在帮助自己梳理过去生物知识的同时了解最新的免疫学前沿理论，以便更好的构建T cell 激活预测模型。

<div class="blog-callout">
  <strong>本文定位。</strong>
  这不是综述论文，而是一份面向建模问题的入门级机制地图。重点不是记住每一个分子名，而是理解：T cell的各种背景知识以及 T cell激活的机制（可能的假设），以及哪些生物学变量可能成为机器学习模型的关键特征、混杂因素或标签来源。
</div>

## One-Sentence Summary

T 细胞通过 TCR 识别抗原呈递细胞或靶细胞表面的 peptide-MHC 复合物，但真正的 T cell activation 不是单个 binding event 的结果，而是 TCR-pMHC 结合动力学、CD3 信号转导、共受体/共刺激、膜空间组织、机械力、细胞状态和组织微环境共同积分后的功能输出。

![TCR-pMHC and CD3 signaling architecture](/_posts/assets/2026-05-11-The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes/figure1.png)

## Glossary for This Note

| Term | Meaning in this note |
| --- | --- |
| TCR | T cell receptor, usually alpha-beta TCR unless explicitly stated |
| pMHC | peptide-MHC complex, the actual ligand seen by conventional TCRs |
| HLA | human MHC molecules; HLA-A/B/C are class I, HLA-DR/DQ/DP are class II |
| CDR | complementarity-determining region of TCR; CDR3 is most diverse |
| ITAM | immunoreceptor tyrosine-based activation motif in CD3 chains |
| APC | antigen-presenting cell, e.g. dendritic cell, macrophage, B cell |
| Avidity | multivalent/cellular binding strength, not identical to molecular affinity |
| Potency | ligand dose needed to elicit a given response |
| Cross-reactivity | one TCR recognizing multiple pMHC ligands |
| Immunogenicity | ability of an antigen to induce an immune response in a biological context |

## Why This Matters

T 细胞是适应性免疫的核心执行者。它们需要在体内完成一个困难的判别任务：在大量 self peptide-MHC 背景中识别极少数 foreign、tumor-derived 或 otherwise abnormal peptide-MHC。这个判别一旦出错，可能导致感染控制失败、肿瘤免疫逃逸、自身免疫或免疫治疗毒性。

从建模角度看，TCR-pMHC 任务重要，至少有四个原因：

1. **个体化免疫治疗。** Neoantigen vaccine、TCR-T、TIL therapy、checkpoint blockade 的效果都与 TCR 能否识别有效抗原有关。
2. **抗原特异性解释。** 单细胞 TCR-seq 给了大量 TCR 序列，但多数 TCR 的抗原特异性未知。
3. **安全性评估。** 工程化 TCR 需要预测交叉反应，避免识别正常组织抗原。
4. **机制引导建模。** 如果标签是 tetramer binding、cytokine secretion、killing 或 expansion，它们对应的是不同层级的生物学事件，不能简单混为同一个“positive”。

还可以从“任务终点”来理解它的重要性：

| Scenario | Biological question | Model output one might want |
| --- | --- | --- |
| Neoantigen vaccine | 哪些突变肽能被患者 HLA 呈递并诱导 T 细胞反应？ | peptide presentation, immunogenicity, response ranking |
| TCR-T design | 一个 TCR 是否能识别肿瘤 pMHC，且不会强烈交叉识别正常组织？ | specificity, cross-reactivity risk, activation potency |
| TIL therapy | 扩增出来的 T cell clone 是否 tumor-reactive？ | clone annotation, antigen assignment, functional state |
| Checkpoint blockade | 患者是否已有可恢复的 tumor-reactive exhausted T cells？ | response likelihood, clonal expansion, exhaustion/reinvigoration state |
| Autoimmunity / infection | 某些 TCR clone 是否与特定病原体或自身抗原相关？ | antigen specificity, disease association |

这里的关键难点是：临床或实验上最关心的通常是 function，但数据里最容易获得的往往是 sequence 或 binding proxy。中间每一层都可能引入偏差。

### The Core Computational Challenge

TCR-pMHC modeling 难，不只是因为数据少，而是因为它同时具有以下特征：

- **Combinatorial explosion.** TCR repertoire、peptide space 和 HLA polymorphism 三者相乘，空间巨大。
- **Sparse labels.** 已知 specificity 只是全部 TCR-antigen 空间中极小一部分。
- **Weak negatives.** 很多 negative 是未观察或随机配对，不是真正实验阴性。
- **Context-dependent output.** 同一 TCR-pMHC 可以在不同 cell state、dose、APC 和 cytokine 环境下输出不同。
- **Many-to-many mapping.** 一个 TCR 可以识别多个 peptide，一个 peptide 也可以被多个不同 TCR clonotype 识别。
- **Mechanistic hierarchy.** presentation、binding、signaling、killing、clinical response 是不同层级。

因此，好的模型不应只追求一个 binary accuracy，而要尽量回答：模型到底学到了哪一层生物学？

## The Cast: Molecules and Cells

### Major T Cell Types

T cell 不是一个单一群体。不同细胞类型的 TCR signaling threshold、功能输出和建模目标都不一样。

| Cell type | Main recognition | Main function | Modeling notes |
| --- | --- | --- | --- |
| Naive CD8 T cell | pMHC I | differentiate into cytotoxic effector/memory | strongly depends on priming and co-stimulation |
| Effector CD8 T cell | pMHC I | killing infected/tumor cells | cytotoxicity readouts most relevant |
| Memory CD8 T cell | pMHC I | rapid recall response | lower activation threshold than naive cells |
| Naive CD4 T cell | pMHC II | differentiate into helper subsets | cytokine milieu shapes outcome |
| Th1 | pMHC II | IFN-gamma, macrophage help, CD8 support | relevant to anti-tumor and antiviral immunity |
| Tfh | pMHC II | B cell help | not usually captured by TCR-pMHC binding datasets |
| Treg | pMHC II, often self-biased | suppression and tolerance | high self-reactivity can be functional, not simply harmful |
| Exhausted T cell | chronic antigen context | reduced but not absent function | checkpoint response depends on subset |

Most TCR-pMHC prediction datasets ignore this cellular layer, but activation labels cannot be interpreted cleanly without it.

### TCR-CD3 Complex

TCR 通常指 αβ TCR 的可变区识别模块。它由 TCRα 和 TCRβ 链组成，负责识别 pMHC。真正把识别转成细胞内信号的是 CD3 复合物，包括 CD3γε、CD3δε 和 ζζ 链；这些链的胞内段含有 ITAMs，可以被 Src family kinase Lck 磷酸化，进而招募 ZAP-70 并启动下游信号。现代结构生物学研究显示，TCR-CD3 不是一个松散集合，而是具有特定组装方式的跨膜受体复合物。[^dong2019tcrcd3]

一句话：**TCR 负责看见 pMHC，CD3 负责把“看见”变成信号。**

更具体地说，TCR-CD3 复合物有几个对建模很重要的特点：

- **TCRα/TCRβ 的胞外可变区决定主要识别界面。** CDR1/CDR2 通常更多接触 MHC helices，CDR3 尤其是 CDR3β 常常深入 peptide 暴露区域，但不同 TCR-pMHC 结构差异很大。
- **TCR 本身胞内尾部很短。** 它不是经典 receptor tyrosine kinase；信号能力主要来自 CD3 的 ITAMs。
- **CD3 ITAM 数量提供信号放大潜力。** CD3γε、CD3δε 和 ζζ 链提供多个 ITAM phosphorylation sites，使早期信号能被多步放大。
- **Lck 的定位很关键。** CD4/CD8 co-receptor 结合 MHC 后可把 Lck 带到 TCR-CD3 附近，提高 ITAM phosphorylation 概率。
- **受体复合物处于膜环境中。** TCR 不是在三维溶液里自由漂浮，而是在二维膜上扩散、聚集、受力并与 actin cytoskeleton 耦合。

### Early TCR Signaling Cascade

一个简化但实用的早期信号流程可以写成：

1. TCR binds pMHC.
2. CD4/CD8 co-receptor helps recruit Lck.
3. Lck phosphorylates CD3 ITAMs.
4. ZAP-70 binds phosphorylated ITAMs and becomes activated.
5. ZAP-70 phosphorylates adaptor proteins such as LAT and SLP-76.
6. LAT/SLP-76 signalosome recruits PLC-gamma1, Grb2, Gads, Vav and other molecules.
7. PLC-gamma1 generates IP3 and DAG.
8. IP3 triggers Ca2+ release and store-operated calcium entry; DAG activates PKC-theta and RasGRP.
9. Major transcriptional programs converge on NFAT, NF-kB and AP-1.
10. The cell changes gene expression, metabolism, survival, cytokine secretion and proliferation.

这条路径不是线性电路，而是高度反馈调控的 network。phosphatases、ubiquitin ligases、membrane microdomains、actin dynamics 和 co-stimulatory/checkpoint receptors 都会改变信号幅度和持续时间。[^courtney2018tcrsignaling]

### Positive and Negative Regulation

早期 TCR signaling 中既有正调控，也有负调控：

| Regulator | Rough role |
| --- | --- |
| Lck | phosphorylates CD3 ITAMs; also regulated by phosphorylation state |
| Fyn | Src family kinase, can contribute to early signaling |
| ZAP-70 | binds phosphorylated ITAMs and phosphorylates LAT/SLP-76 |
| LAT | scaffold for signalosome assembly |
| SLP-76 | adaptor linking ZAP-70 to downstream pathways |
| PLC-gamma1 | generates IP3 and DAG |
| CD45 | phosphatase with complex positive/negative roles; spatial exclusion is important |
| Csk | inhibits Src family kinases via inhibitory phosphorylation |
| SHP-1/SHP-2 | phosphatases involved in inhibitory signaling; SHP-2 is linked to PD-1 pathway |
| Cbl family | ubiquitin ligases that can downregulate signaling components |

这张表的用途不是背分子，而是提醒：activation 是 kinase/phosphatase/adaptor/feedback 的动态平衡。

### pMHC: Peptide plus MHC/HLA

MHC 是抗原呈递分子。人类的 MHC 通常称为 HLA。TCR 识别的不是裸 peptide，而是 peptide 和 HLA 形成的复合界面。

| Feature | MHC class I | MHC class II |
| --- | --- | --- |
| Typical peptide length | 8-10 aa, often 9 aa | longer, often 13-18 aa or more |
| Expressed by | most nucleated cells | professional antigen-presenting cells |
| Recognized by | CD8 T cells | CD4 T cells |
| Antigen source | mostly endogenous/cytosolic proteins | mostly exogenous/endosomal proteins |
| Main role | detect infected/tumor cells and enable cytotoxic killing | coordinate helper responses |

HLA 极度多态，这意味着同一个 peptide 是否能被呈递，取决于个体 HLA allele。HLA 的 peptide-binding groove 决定了 anchor residues 和 peptide conformation，而 TCR 看到的是 peptide 与 HLA 表面共同组成的三维化学图案。[^rock2016mhc]

对 TCR 来说，pMHC 表面可以粗略拆成三类信息：

- **Peptide central exposed residues.** 这些残基经常直接决定 antigen specificity。
- **HLA helices and pockets.** 它们限制 peptide 的形状，也与 TCR CDR1/CDR2 接触。
- **Peptide conformation.** 同一个序列在不同 HLA 上可能呈现不同 backbone bulge、register 或 side-chain exposure。

因此，建模时不能只把 peptide 当作普通短文本，也不能只把 HLA 当作类别标签。更合理的表示应考虑 peptide-HLA pair 的 joint conformation。结构免疫学研究表明，TCR 对 antigen-presenting molecules 的识别具有一定 docking bias，但不是一个固定几何模板。[^rossjohn2015tcrrecognition]

### HLA Nomenclature and Polymorphism

HLA allele 名称通常写作 `HLA-A*02:01`、`HLA-B*07:02`、`HLA-DRB1*15:01`。粗略理解：

- `A`, `B`, `C`: class I heavy-chain loci.
- `DR`, `DQ`, `DP`: class II loci.
- `02:01` 这类数字代表 allele group 和具体 protein sequence。

建模时不能把 `HLA-A*02:01` 和 `HLA-A*02:06` 随便合并。它们可能有相似 binding motif，但具体 peptide repertoire 和 TCR-facing surface 可以不同。另一方面，完全把每个 allele 当作独立类别也会造成数据稀疏。更好的方案可能是使用 HLA pseudo-sequence、binding groove residues 或结构表示。

### Structural Variables for TCR-pMHC

TCR-pMHC 结构建模时可以关注：

- TCR docking angle and crossing angle.
- TCR footprint over peptide and MHC helices.
- CDR loop contacts, especially CDR3α and CDR3β.
- peptide buried/exposed residues.
- hydrogen bonds, salt bridges, hydrophobic contacts.
- interface buried surface area.
- shape complementarity.
- electrostatic complementarity.
- water-mediated contacts.
- conformational changes upon binding.

这些变量很多不能从序列直接看出，但可以通过结构预测、docking 或已知 TCR-pMHC structures 构建 proxy。

### Co-Receptors and Co-Stimulation

CD8 或 CD4 不是简单的“细胞类型标签”。它们能与 MHC I 或 MHC II 结合，并把 Lck 带到 TCR-CD3 附近，提高早期信号效率。与此同时，CD28、ICOS、4-1BB 等共刺激分子，以及 PD-1、CTLA-4、LAG-3、TIM-3 等抑制性受体，会强烈影响同一个 TCR-pMHC 结合事件最终是否转化成功能性激活。[^chen2013cosignaling]

T cell priming 常被概括为 signal 1、signal 2、signal 3：

| Signal | Main source | Meaning |
| --- | --- | --- |
| Signal 1 | TCR-pMHC | antigen specificity |
| Signal 2 | CD28/B7 and other co-stimulation | permission, survival, proliferation |
| Signal 3 | cytokines such as IL-2, IL-12, type I IFN, TGF-beta | differentiation direction and functional polarization |

Naive T cell 通常比 effector/memory T cell 更依赖强 co-stimulation。肿瘤或慢性感染环境中，PD-1、CTLA-4 等 inhibitory receptors 会降低 TCR signaling 或改变 activation threshold。因此，同一个 TCR-pMHC interaction 在不同细胞状态下可能产生完全不同的输出。

### Adhesion Molecules

T cell 与 APC/target cell 的接触也依赖 adhesion molecules，例如 LFA-1/ICAM-1。adhesion 不仅让两个细胞“贴住”，也影响免疫突触稳定性、force transmission 和 signal duration。对于 killing assay，target engagement time 与 degranulation probability 也可能相关。

## V(D)J Recombination: Why TCR Repertoires Are So Large

TCR 的多样性主要来自 V(D)J recombination。TCRβ 链由 V、D、J 基因片段重排形成，TCRα 链由 V、J 片段重排形成。重排过程中还会发生 nucleotide deletion、N/P nucleotide addition 等 junctional diversity，最终在 CDR3 区域形成高度多样的序列。CDR3 通常最接近 peptide 中央区域，因此在抗原特异性预测中经常被重点建模。

但需要谨慎：CDR3 很重要，不等于 CDR3 足够。TCR 的 CDR1/CDR2 往往与 MHC helices 接触，TCRα/TCRβ pairing、V/J gene、TCR docking geometry、pMHC conformation 都可能影响识别。只用 CDR3β 预测 antigen specificity，通常会丢掉大量信息。

更细一点看，TCR repertoire 的多样性来自几个来源：

1. **Combinatorial diversity.** 不同 V、D、J gene segment 的组合。
2. **Junctional diversity.** RAG-mediated cleavage 后，coding ends 可被修剪，并由 TdT 添加非模板 N nucleotides；hairpin opening 可产生 P nucleotides。
3. **Chain pairing diversity.** 一个 TCRα 与一个 TCRβ 配对，组合空间进一步扩大。
4. **Thymic selection.** 胸腺正选择保留能弱识别 self-MHC 的 TCR；负选择删除强 self-reactive TCR，但删除不完全。
5. **Peripheral expansion.** 感染、肿瘤、自身免疫、年龄和组织环境会选择性扩增某些 clone。

V(D)J recombination 由 RAG1/RAG2 识别 recombination signal sequences 并启动 DNA double-strand break，随后通过 non-homologous end joining 修复。这一过程带来极大多样性，也意味着 TCR 序列空间高度稀疏。[^schatz2011vdj]

对模型来说，这里有几个直接后果：

- **public TCR 和 private TCR 不同。** 某些 TCR 在多人中反复出现，可能因为重排概率高或抗原选择强；大量 TCR 只在个体内出现。
- **generation probability 是混杂因素。** 一个 TCR 在数据库中常见，不一定代表它对某抗原强特异，也可能是更容易由重排生成。
- **paired-chain 数据更珍贵。** bulk TCRβ-seq 丢失 αβ pairing，会限制 specificity 推断。
- **negative sampling 很危险。** 随机配对 TCR 和 peptide 当作 negative，可能混入未知 true positive，也会让模型学到 antigen frequency 或 HLA bias。

## From Peptide to pMHC

一个 peptide 能否被 T 细胞识别，至少要经过下面几层筛选：

1. **Antigen generation.** 蛋白是否表达、降解、被 proteasome 或 endosomal pathway 加工。
2. **Peptide transport and loading.** MHC I 路径中，肽段通常通过 TAP 进入 ER 并装载到 MHC I；MHC II 路径通常涉及 endosomal/lysosomal processing。
3. **HLA binding stability.** peptide-HLA 复合物是否足够稳定，能否到达细胞表面。
4. **Surface abundance.** 细胞表面 pMHC copy number 是否足够。
5. **TCR recognition.** TCR 是否以合适几何和动力学识别该 pMHC。
6. **Cellular context.** 是否有共刺激、炎症因子、代谢支持，是否存在 checkpoint suppression。

这也是为什么 neoantigen prediction 不能只预测 peptide-MHC binding。HLA binding 是必要但不充分条件；TCR recognition 和 T cell activation 又是更后面的层级。

### MHC Class I Pathway

MHC I 路径主要呈递细胞内蛋白来源的 peptide，典型流程如下：

1. Cytosolic proteins are degraded by proteasome or immunoproteasome.
2. Peptide fragments are transported into the ER by TAP.
3. ER aminopeptidases such as ERAP can trim peptides to suitable length.
4. The peptide loading complex helps load peptide onto MHC I heavy chain plus beta-2 microglobulin.
5. Stable peptide-MHC I complexes pass through Golgi and reach the cell surface.
6. CD8 T cells inspect these pMHC I complexes.

关键变量包括 source protein expression、protein turnover、proteasomal cleavage preference、TAP transport efficiency、ERAP trimming、HLA binding affinity/stability、surface pMHC abundance。mass spectrometry immunopeptidomics 可以直接观测一部分 naturally presented peptides，但覆盖度和定量仍有限。[^blum2013antigenprocessing]

### MHC Class II Pathway

MHC II 路径主要呈递外源或 vesicular compartment 中的 antigen，典型流程如下：

1. Extracellular proteins are endocytosed or phagocytosed by professional APCs.
2. MHC II alpha/beta chains assemble in ER with invariant chain.
3. Invariant chain guides MHC II to endosomal/lysosomal compartments and leaves CLIP in binding groove.
4. HLA-DM promotes CLIP exchange and peptide editing.
5. Stable peptide-MHC II complexes traffic to cell surface.
6. CD4 T cells inspect these pMHC II complexes.

MHC II peptide 通常更长，binding register 更复杂，而且不同 APC type 的 processing environment 不同。对 CD4 T cell 任务来说，简单套用 MHC I 的 9-mer 思维会遗漏很多信息。

### Cross-Presentation

Dendritic cells 可以把外源 antigen 通过 MHC I 呈递给 CD8 T cells，这叫 cross-presentation。它对抗肿瘤免疫和病毒免疫很重要，因为 naive CD8 T cell priming 常常需要专业 APC，而不是直接由肿瘤细胞完成。cross-presentation 的存在提醒我们：靶细胞上的 pMHC 和 priming APC 上的 pMHC 可能来自不同细胞、不同处理环境。

### Presentation Variables as Model Features

如果目标是预测 tumor antigen immunogenicity，可以把 presentation 拆成几个可建模变量：

| Variable | Why it matters | Possible proxy |
| --- | --- | --- |
| RNA expression | antigen source abundance | TPM, single-cell expression |
| protein abundance | peptide source abundance | proteomics, inferred expression |
| protein turnover | degradation supply | half-life, ubiquitination proxy |
| proteasome cleavage | peptide generation | NetChop-like score |
| TAP transport | ER entry for MHC I | TAP transport predictor |
| ERAP trimming | final peptide length/register | sequence motif, ERAP expression |
| HLA binding | complex formation | NetMHCpan/MHCflurry score |
| pMHC stability | surface lifetime | stability predictor, dissociation assay |
| HLA expression | display capacity | HLA RNA/protein, B2M status |
| IFN-gamma response | antigen presentation upregulation | IFN pathway gene expression |

这张表说明：presentation prediction 本身已经是多因素问题。TCR recognition 只是它之后的下一层。

## Binding Is Not Activation

TCR-pMHC binding 常被当作建模目标，但 binding 和 activation 不是同一件事。

**Binding** 更接近分子层面的事件：TCR 是否与某个 pMHC 结合，结合亲和力是多少，dwell time 多长，是否形成稳定 tetramer staining。

**Activation** 是细胞层面的输出：CD3 phosphorylation、Ca2+ flux、NFAT/NF-kB/AP-1 activation、CD69/CD25 upregulation、IL-2/IFN-gamma secretion、proliferation、cytotoxicity 等。

一个 pMHC 可以 weakly bind 但触发有效信号，也可以在 tetramer assay 中看起来能 bind，却不产生强功能输出。TCR 的 ligand discrimination 被认为依赖 binding kinetics、kinetic proofreading、mechanical force、receptor organization 和 feedback regulation 的组合。[^dushek2012kineticproofreading][^huang2010tcrtriggering]

<div class="blog-callout">
  <strong>建模提醒。</strong>
  如果数据集 label 来自 tetramer binding，它更像 binding/avidity 标签；如果 label 来自 cytokine、killing 或 proliferation，它更像 activation/function 标签。把它们直接混成一个二分类任务，会把机制差异压缩成噪声。
</div>

可以把常见概念分成几层：

| Term | Rough level | Typical measurement | Common pitfall |
| --- | --- | --- | --- |
| Affinity | molecular equilibrium | SPR 3D KD | does not encode cell context |
| Off-rate / dwell time | molecular kinetics | SPR koff, 2D assays | force and membrane context can change it |
| Avidity | multivalent/cellular binding | tetramer staining | depends on TCR expression and reagent valency |
| Potency | dose needed for response | EC50 in cytokine/killing assay | assay-specific and time-dependent |
| Efficacy | maximal response | plateau response | high affinity may not guarantee high maximal function |
| Immunogenicity | ability to induce immune response | expansion, cytokine, clinical response | includes presentation, priming and host context |

一个非常常见的误解是：KD 越低，T cell response 越强。实际情况更像“窗口函数”：过弱的 interaction 不能完成信号积累；过强的 interaction 可能影响 serial triggering、TCR downmodulation、negative feedback 或 thymic/peripheral tolerance；不同 T cell clone 和 assay 下的最优区间也不同。

### Practical Difference Between Binding Dataset and Activation Dataset

如果一个数据集来自 tetramer sorting，positive cell 的 TCR 往往具有足够 multimer avidity；negative 可能只是没有被该 multimer 捕获。  
如果一个数据集来自 co-culture cytokine assay，positive 说明在特定 APC、peptide dose、time point 和 cytokine readout 下产生功能响应。  
如果一个数据集来自 tumor killing，positive 还要求 target cell 表达/呈递 antigen，并且 T cell 具备 cytotoxic machinery。

这三者可以重叠，但不是同义词。

### Biophysical Quantities

常见 biophysical terms：

| Quantity | Symbol / unit | Interpretation |
| --- | --- | --- |
| association rate | kon | how fast TCR and pMHC bind |
| dissociation rate | koff | how fast complex falls apart |
| equilibrium affinity | KD = koff / kon | lower KD means stronger equilibrium binding |
| half-life | t1/2 | related to koff; longer half-life often means longer dwell |
| 2D affinity | membrane context | effective binding on cell surfaces |
| bond lifetime under force | force-dependent | relevant to mechanosensing |

SPR 常用于测 3D kinetics，但 TCR signaling 发生在 2D cell-cell interface。2D binding parameters sometimes correlate better with T cell responsiveness than conventional 3D parameters. 这也是为什么结构和力学模型会被提出。[^huang2010tcrtriggering]

## How Do We Measure T Cell Activation?

T cell activation 没有唯一“金标准”。不同实验读数对应不同时间尺度和生物学层级。

| Readout | Typical meaning | Time scale | Notes for modeling |
| --- | --- | --- | --- |
| Tetramer/multimer staining | TCR-pMHC binding or avidity | minutes | high avidity bias; may miss weak but functional TCRs |
| CD69 | early activation marker | hours | sensitive but not necessarily functional |
| CD25 | IL-2 receptor alpha, activation/proliferation readiness | hours to day | depends on cytokine environment |
| IL-2 | autocrine growth signal | hours | common activation output |
| IFN-gamma/TNF/IL-2 ELISA or ICS | effector cytokine response | hours | functional, but assay-dependent |
| CD107a degranulation | cytotoxic granule release | hours | closer to killing potential |
| Chromium release / flow killing / Incucyte killing | target-cell death | hours to days | strongest functional output, but harder and noisier |
| Expansion / clonal enrichment | antigen-driven proliferation in vivo or ex vivo | days to weeks | influenced by many context variables |
| scRNA-seq activation/exhaustion states | cell state programs | snapshot | powerful but label definition is nontrivial |

Flow cytometry, ELISA/ELISpot, intracellular cytokine staining, reporter cell lines, cytotoxicity assays and single-cell multi-omics can all be used，但它们测到的不是同一层信号。对 AI 模型来说，最重要的是把 label provenance 记录清楚。

### Experimental Design Variables

同一种 readout 也会因实验设计不同而产生差异。记录数据时最好保留：

- T cell type: naive, effector, memory, exhausted, TCR-transduced cell line, primary T cell.
- Species: human or mouse.
- TCR format: endogenous TCR, cloned TCR, engineered affinity-enhanced TCR, single-chain construct.
- APC or target cell: dendritic cell, B cell, T2 cell, K562 artificial APC, tumor cell line, primary tumor cell.
- Antigen format: peptide pulsing, minigene, full-length protein, endogenous tumor antigen, mRNA expression.
- Peptide dose and time point.
- HLA allele and expression level.
- Co-stimulation: CD28/B7, anti-CD28 antibody, cytokines, artificial APC beads.
- Readout: marker, cytokine, killing, proliferation, reporter.
- Threshold definition: what counts as positive, relative to which control.

如果这些元数据缺失，模型可能把 assay-specific artifacts 当作 biological rule。

### Example Protocol Logic: Peptide-Pulsed APC Assay

一个简化的 peptide-pulsed APC activation assay：

1. Select APC expressing known HLA allele.
2. Pulse APC with peptide at serial dilutions.
3. Wash or keep peptide depending on protocol.
4. Co-culture APC with TCR-expressing T cells.
5. Incubate for defined time, e.g. 4 h, 16 h, 24 h.
6. Measure CD69/CD25, cytokines, reporter signal or killing.
7. Fit dose-response curve and estimate EC50 / maximum response.

这种 assay 的优点是控制 peptide 和 HLA；缺点是 peptide pulsing 可能绕过自然 antigen processing，且高 peptide dose 可能产生非生理 pMHC density。

### Example Protocol Logic: Endogenous Antigen Recognition

更接近真实情况的 assay 可能使用表达 full-length antigen 或天然肿瘤抗原的 target cells：

1. Confirm target cell HLA and antigen expression.
2. Co-culture target cells with T cells.
3. Measure cytokine, degranulation or killing.
4. Use HLA blocking, antigen knockout or peptide rescue as specificity control.

这种 assay 更接近真实 presentation，但变量更多：antigen expression、processing machinery、HLA expression、target susceptibility to killing 都会影响结果。

### Common Controls

实验上常见 control 包括：

- no peptide / irrelevant peptide control
- known agonist peptide positive control
- HLA-mismatched APC control
- TCR-negative or mock-transduced cell control
- blocking antibody against MHC or TCR
- dose-response curve instead of one fixed dose
- replicate and batch controls

对于机器学习，negative control 的质量尤其重要。很多 public datasets 的 negative 并不是实验验证的 non-binder/non-activator，而是 computationally paired negatives。

### Label Granularity

不要只记录 `positive/negative`，更好的记录方式是：

```text
assay_type: intracellular cytokine staining
readout: IFN-gamma positive fraction
peptide_dose: 1 uM
time_point: 6 h
APC: autologous dendritic cell
HLA: HLA-A*02:01
positive_threshold: >2x no-peptide control and >0.5% IFN-gamma+
replicates: 3
effect_size: 4.8x over control
```

这类元数据越完整，后续越容易把不同论文的数据合并成可解释训练集。

## Data Resources

### VDJdb

VDJdb 是一个汇集已知 antigen specificity TCR 序列的数据库，包含 TCR 序列、抗原、MHC restriction、物种、实验方法等信息。它常用于 TCR specificity prediction benchmark，但存在 antigen coverage skew、public TCR bias、重复记录、实验方法不一致等问题。[^vdjdb]

使用时建议检查：

- 是否有 paired alpha-beta chain。
- antigen epitope 是否标准化。
- MHC allele 是否明确。
- record confidence score 或 curation quality。
- 同一 TCR 是否重复出现在 train/test。
- 是否按 antigen split，而不是随机 record split。

### McPAS-TCR

McPAS-TCR 是一个 manually curated catalogue，收集与病原体、癌症、自身免疫等病理状态相关的 TCR 序列。它对疾病关联分析有用，但“pathology associated”不总等价于精确 pMHC specificity。[^mcpas]

它更适合做 disease-associated repertoire exploration，不应无脑当作 high-confidence TCR-pMHC binding benchmark。

### 10x Genomics Antigen Capture / Immune Profiling

10x Genomics 的单细胞 V(D)J + gene expression + feature barcode / antigen capture 数据可以同时提供 TCR pairing、转录状态和 antigen binding 信息。优点是信息维度高；缺点是 antigen panel 有限，multimer binding 仍然不是完整 activation。[^tenximmune]

### IEDB, MHCflurry, NetMHCpan and Related Resources

IEDB 收集 epitope 和免疫实验信息；NetMHCpan、MHCflurry 等工具主要用于 peptide-MHC binding / presentation prediction。它们对 upstream antigen presentation 很有帮助，但不能直接替代 TCR recognition 或 T cell activation prediction。[^iedb][^netmhcpan][^mhcflurry]

### Dataset Leakage and Benchmark Design

TCR specificity benchmark 容易出现几类 leakage：

- 同一 CDR3 或高度相似 CDR3 同时出现在 train/test。
- 同一 epitope 的 public TCR pattern 被随机 split 泄漏。
- negative sampling 太简单，模型只学会区分真实 epitope 分布和随机 peptide。
- HLA、species、assay type 与 label 强相关。
- 同一 paper 或 batch 的记录同时分布在 train/test。

更严格的评估可以考虑：

- epitope-level split
- donor-level split
- TCR clonotype split
- HLA-held-out split
- pathogen/tumor antigen family held-out split
- assay-held-out transfer

这些 split 会显著降低分数，但更接近真实泛化。

### Data Cleaning Checklist

整理 TCR-pMHC 数据时可以逐项检查：

- 标准化 amino acid sequences，去除 stop codon、ambiguous residues 和非法字符。
- 分开 TRA/TRB，不要把 chain 信息混掉。
- 保留 nucleotide sequence if available，用于 clonotype definition。
- 标准化 V/J gene names，例如 IMGT nomenclature。
- 标准化 HLA allele 到相同 resolution。
- 标准化 peptide sequence 和 source antigen name。
- 记录 species，不要混合 human/mouse 后直接训练。
- 合并重复记录时保留 assay disagreement，而不是简单去重。
- 对同一 TCR-pMHC 多 assay 结果，保留多标签。
- 对 negative samples 标记来源：experimental negative vs sampled negative。
- 对 literature-curated entries 保留 paper ID。

### Useful Derived Features

可以派生的特征包括：

- CDR3 length, charge, hydrophobicity, aromatic content.
- V/J gene family.
- TCR generation probability.
- peptide length, anchor residue pattern.
- HLA supertypes or pocket residues.
- edit distance to known epitope-specific TCR motifs.
- clonotype expansion.
- tissue origin.
- expression of cytotoxic/exhaustion markers if single-cell data available.

## Mechanistic Hypotheses of TCR Triggering

TCR triggering 不是单一机制已经被完全证明的故事，而是多个模型从不同尺度解释同一个现象。下面这些模型经常同时出现，彼此并不一定互斥。

![From TCR-pMHC recognition to T cell activation and killing](/_posts/assets/tcr-pmhc/t-cell-activation-flow.svg)

### 1. Kinetic Proofreading

Kinetic proofreading 最早来自 Hopfield/Ninio 对分子识别准确性的解释，后来被用于 TCR ligand discrimination。核心思想是：TCR-pMHC 结合后，需要按顺序完成多个 signaling steps；如果 ligand dissociates too fast，信号链条来不及走完，就不会触发完整激活。这样 T 细胞可以放大不同 dwell time ligand 的差异。[^mckeithan1995kinetic][^dushek2012kineticproofreading]

优点：解释为什么微小的 off-rate 差异可以带来巨大的功能差异。  
局限：单纯 dwell time 不能解释所有现象，例如极高亲和力有时反而不一定带来更好功能，且细胞内反馈、受体空间组织和机械力也很重要。

从模型语言看，kinetic proofreading 像一个多步门控过程：

$$
TCR\text{-}pMHC \rightarrow C_1 \rightarrow C_2 \rightarrow ... \rightarrow C_n \rightarrow Response
$$

每一步都需要 ligand 没有提前解离。步数越多，系统越能区分快速解离和慢速解离 ligand，但响应速度也会变慢。真实 T cell 可能通过 feedback 和 co-receptor tuning 在 sensitivity 与 specificity 之间折中。

### 2. Serial Triggering

Serial triggering 模型认为，一个 pMHC 可以连续触发多个 TCR，从而解释为什么极低数量的 agonist pMHC 也可能诱导 T cell response。短到适中的 dwell time 可能允许 ligand 在不同 TCR 间周转；如果结合太稳定，反而可能降低 serial engagement。[^valitutti1995serial]

这对建模有启发：affinity 并不总是越高越好，最佳激活可能依赖 dwell time、pMHC density 和 receptor availability 的组合。

这个模型对低 antigen density 尤其重要。许多病毒或肿瘤抗原在细胞表面 copy number 可能很低，T 细胞仍能响应，说明系统必须有很强的 signal amplification。

### 3. Kinetic Segregation

Kinetic segregation 模型强调膜间距离和 phosphatase exclusion。TCR-pMHC 结合使 T cell 与 APC 局部膜间距变短，大型 phosphatase CD45 被排斥在接触区之外，从而让 Lck-mediated phosphorylation 在局部占优，触发 CD3 ITAM phosphorylation。[^davis2006kineticsegregation]

这个模型提醒我们，TCR triggering 不是孤立的溶液结合事件，而是发生在二维膜界面上的空间组织过程。

TCR-pMHC complex 的 ectodomain 尺寸较短，而 CD45 ectodomain 很大。局部 close contact 可能改变 kinase/phosphatase balance。这里的核心不是某个 binding site，而是膜间距、分子尺寸和排斥效应。

### 4. Receptor Clustering and Microclusters

TCR engagement 后，TCR microclusters 会形成并向免疫突触中心移动。早期 signaling molecules 如 ZAP-70、LAT、SLP-76 等在 microclusters 中富集。免疫突触不是简单的“细胞粘住”，而是动态组织信号、黏附和分泌方向性的界面。[^grakoui1999synapse][^yokosuka2005microclusters]

聚集模型对机器学习的启发是：单个 TCR-pMHC pair 的序列/结构特征可能不足以解释 response；pMHC density、膜上分布、TCR copy number 和细胞接触几何也可能是关键变量。

TCR microclusters 可以看作 early signaling hubs。它们与 actin retrograde flow、integrin-mediated adhesion 和 LAT signalosome 动态耦合。随着细胞-细胞接触成熟，受体和信号分子会重新分布到免疫突触不同区域。

### 5. Conformational Change and Induced Fit

TCR 与 pMHC 结合时，TCR CDR loops、peptide 或 MHC helices 可能发生 conformational adjustment。某些研究认为，ligand binding 不仅提供 affinity，也可能通过 docking geometry 或 conformational change 改变 CD3/TCR complex 的受力与信号状态。[^adams2016tcrrecognition]

对结构建模来说，这意味着静态 docking score 可能不够；需要关注 interface flexibility、water-mediated contacts、loop dynamics、electrostatics 和 conformational ensembles。

很多 TCR-pMHC 结构显示，TCR docking 可以具有保守倾向，但每个复合物的 CDR loop usage、peptide contact pattern 和 interface chemistry 都有差异。对于深度学习模型，可能需要同时学习“可迁移的结构规则”和“抗原特异的局部接触”。

### 6. Mechanosensing and Catch Bonds

T 细胞在扫描 APC 或靶细胞时会施加机械力。部分 TCR-pMHC 相互作用在 force 下表现出 catch bond-like behavior，即一定范围内受力反而延长 bond lifetime。研究显示，agonist pMHC 更可能在机械力条件下稳定触发信号，而非激动剂 ligand 则不容易产生同样效果。[^liu2014catchbond][^zhu2019mechanosensing]

这说明常规溶液中的 affinity 或 off-rate 只是一部分信息；二维膜、细胞骨架、actin flow、力加载方向和速度都可能改变有效 binding lifetime。

mechanosensing 的难点在于实验测量更复杂。常规 SPR 给的是三维溶液或表面固定条件下的 binding kinetics；而 T 细胞真实识别发生在两个柔性膜之间，并伴随 actomyosin force、microvilli scanning 和 receptor transport。

### 7. Immunological Synapse

免疫突触通常包含 central supramolecular activation cluster (cSMAC)、peripheral SMAC (pSMAC) 和 distal SMAC (dSMAC) 等区域。它参与信号组织、黏附稳定、胞吐方向化和 cytotoxic granule delivery。[^grakoui1999synapse]

在 CD8 T cell killing 中，免疫突触不仅是信号平台，也是将 perforin/granzyme 精准递送到靶细胞的结构基础。

免疫突触还可以帮助解释为什么 T cell response 具有空间方向性。helper T cell 可以把 cytokine secretion 定向到 APC；cytotoxic T cell 可以把 lytic granules 定向到 target cell，减少旁观者细胞损伤。

### Putting the Models Together

这些模型可以按尺度理解：

| Scale | Mechanism | Main idea |
| --- | --- | --- |
| molecular kinetics | kinetic proofreading, serial triggering | dwell time and ligand turnover shape discrimination |
| membrane organization | kinetic segregation, microclusters | local kinase/phosphatase balance and signal hubs |
| mechanics | catch bonds, mechanosensing | force changes bond lifetime and receptor state |
| structural dynamics | induced fit, docking geometry | binding conformation may affect signaling competence |
| cellular interface | immunological synapse | stable contact, signal organization, directional secretion |

真实 T cell 可能同时利用这些机制。对模型来说，最稳妥的表述是：TCR-pMHC specificity is constrained by sequence and structure, but activation is filtered by kinetics, membrane context, force and cell state.

## How T Cells Kill Target Cells

对于 CD8 cytotoxic T lymphocytes，识别 pMHC 后的杀伤大致包括：

1. **Target recognition.** TCR 识别靶细胞表面特异 pMHC。
2. **Stable contact and synapse formation.** LFA-1/ICAM-1 等黏附分子帮助形成稳定接触。
3. **Polarization.** microtubule organizing center 和 lytic granules 向免疫突触极化。
4. **Degranulation.** perforin 和 granzymes 被释放到突触间隙。
5. **Target apoptosis.** perforin 帮助 granzymes 进入靶细胞，granzyme B 等诱导 apoptosis；FasL-Fas 也可参与杀伤。
6. **Detachment and serial killing.** CTL 可脱离并继续杀伤其他靶细胞。

Perforin/granzyme pathway 和 Fas/FasL pathway 是 cytotoxic lymphocyte killing 的经典机制。[^voskoboinik2015perforin][^martinezlostao2015death]

### Perforin/Granzyme Pathway

Perforin 是 pore-forming protein。CTL 或 NK cell degranulation 后，perforin 有助于 granzymes 进入靶细胞。Granzyme B 可以激活 caspase cascade，也可以通过 Bid/mitochondrial pathway 推动 apoptosis。Granzyme A、K、M 等也有不同底物和功能，但 Granzyme B 最常被作为 cytotoxicity marker。

常见实验读数：

- CD107a surface mobilization: degranulation proxy.
- Granzyme B / perforin expression: cytotoxic potential.
- Annexin V / PI or caspase reporter: target apoptosis.
- live-cell imaging: killing kinetics and serial killing.
- chromium release or LDH release: bulk target-cell lysis.

### Fas/FasL Pathway

Activated T cells 可表达 Fas ligand，与靶细胞 Fas/CD95 结合，触发 death receptor-mediated apoptosis。这个通路不一定需要 granule exocytosis，但依赖靶细胞 death receptor pathway 是否完整。

### Cytokine-Mediated Effects

CD8 T cells 和 Th1-like CD4 T cells 可以分泌 IFN-gamma、TNF 等细胞因子。它们不一定直接杀死靶细胞，但可以：

- 提高 antigen presentation machinery。
- 抑制肿瘤细胞增殖。
- 激活 macrophage 或其他免疫细胞。
- 改变局部 chemokine environment。

因此，在某些 assay 中 IFN-gamma positive 不等于直接 cytotoxic killing，但它是功能性激活的重要读数。

### CD4 T Cells Are Not Just Helpers

CD4 T cells 主要识别 MHC II，但它们的功能很广：

- Th1: IFN-gamma, macrophage activation, support CD8 response.
- Th2: IL-4/IL-5/IL-13, humoral and barrier immunity.
- Th17: IL-17/IL-22, mucosal inflammation.
- Tfh: B cell help and germinal center response.
- Treg: immune suppression and tolerance.

在肿瘤免疫中，CD4 T cells 既可能帮助抗肿瘤反应，也可能通过 Treg 程序抑制免疫。对于模型来说，CD4/CD8 lineage、cell state 和 cytokine profile 都是重要上下文。

### Killing Is Also Quantitative

Killing 不是简单 yes/no。至少有几个可量化维度：

- time to first kill
- probability of killing after contact
- number of targets killed per T cell
- dwell time with target
- degranulation frequency
- cytokine polyfunctionality
- exhaustion after repeated stimulation
- recovery and serial killing capacity

因此，如果模型目标是 cytotoxic function，最好不要只用单一终点标签。live-cell imaging 或 longitudinal killing assay 能提供更丰富的动力学标签。

## Tumor Microenvironment: Why Recognition May Still Fail

即使 TCR 能识别 tumor antigen，肿瘤微环境也可能让 T cell response 失败。常见机制包括：

- **Antigen loss or HLA loss.** 肿瘤细胞下调抗原、B2M 或 HLA，降低 pMHC 呈递。
- **Checkpoint inhibition.** PD-1/PD-L1、CTLA-4 等通路抑制 T cell function。
- **T cell exhaustion.** 长期抗原刺激和抑制性信号导致功能下降，伴随特定转录状态。
- **Suppressive cells.** Treg、MDSC、tumor-associated macrophage 等抑制效应 T 细胞。
- **Metabolic stress.** hypoxia、低葡萄糖、乳酸积累、腺苷等影响 T cell metabolism。
- **Physical exclusion.** stromal barrier 或异常血管导致 T cell 难以进入 tumor nest。

这也是为什么体外 binding/activation 与体内疗效之间常有落差。Checkpoint blockade 的成功部分来自解除这些抑制轴，而不是改变 TCR 本身的 binding specificity。[^pardoll2012checkpoint][^thommen2018exhaustion]

### Antigen and Presentation Escape

肿瘤可以通过多种方式逃避免疫识别：

- mutation or deletion of antigenic clone
- transcriptional silencing of antigen
- loss of heterozygosity in HLA locus
- B2M mutation causing MHC I surface expression loss
- defects in TAP, tapasin, immunoproteasome or IFN-gamma pathway

这会让一个在体外能识别 peptide-pulsed target 的 TCR，在真实肿瘤中找不到足够 pMHC。

### T Cell Exclusion and Suppressive Myeloid Cells

有些肿瘤不是没有 T cells，而是 T cells 停留在 tumor margin 或 stroma，无法进入 tumor nest。CAF、abnormal vasculature、TGF-beta signaling、myeloid inflammation 等都可能参与。MDSC 和 tumor-associated macrophage 可以通过 arginase、ROS、IL-10、TGF-beta、PD-L1 等机制抑制 T cell function。

### Exhaustion Is a Spectrum

T cell exhaustion 不是简单的“坏掉”。慢性抗原刺激下，T cells 会进入一系列状态，包括 progenitor exhausted-like、transitory、terminal exhausted-like states。部分 exhausted T cells 仍可被 PD-1 blockade reinvigorate，部分则更固定。单细胞转录组常用 TOX、PDCD1、LAG3、HAVCR2、TIGIT、CXCL13、GZMB 等基因组合描述不同状态，但不同癌种和组织中含义可能不同。

### Why This Matters for Modeling

一个 TCR specificity model 即使很准，也只能回答“能不能识别”。治疗效果还取决于：

- antigen 是否真的在肿瘤细胞上呈递。
- T cell 是否进入肿瘤。
- T cell 是否处于可恢复状态。
- 是否存在强抑制性 myeloid/stromal environment。
- 是否有足够 clonal expansion 和 persistence。

### Cold, Excluded and Inflamed Tumors

肿瘤免疫状态常粗略分为：

| Phenotype | Rough description | Possible bottleneck |
| --- | --- | --- |
| immune desert / cold | few T cells in tumor | poor priming, low antigenicity, poor recruitment |
| immune excluded | T cells around tumor but not inside tumor nests | stroma, vasculature, TGF-beta, myeloid barriers |
| inflamed / hot | T cells infiltrate tumor | checkpoint suppression, exhaustion, antigen escape |

不同 phenotype 对模型需求不同。cold tumor 中 specificity model 可能不是主要瓶颈；inflamed tumor 中 checkpoint/exhaustion 和 antigen escape 可能更重要。

### Tumor Antigen Classes

常见 tumor antigen 类型：

- **Neoantigens.** Somatic mutation-derived peptides; more tumor-specific but patient-specific.
- **Cancer-testis antigens.** Normally restricted expression, reactivated in tumors.
- **Differentiation antigens.** Tissue-lineage antigens, e.g. melanocyte antigens; risk of on-target off-tumor toxicity.
- **Viral antigens.** Virus-associated cancers, often strong foreign antigen signal.
- **Overexpressed self antigens.** Broad but tolerance and toxicity concerns.

不同 antigen class 的 TCR repertoire、tolerance boundary 和 safety risk 不同。

## What Does This Mean for Deep Learning Models?

如果目标是构建 TCR-pMHC 或 T cell activation 模型，我认为至少要区分下面几类任务。

### Task 1: Peptide-MHC Presentation

输入可能包括 peptide sequence、HLA allele、source protein expression、proteasomal cleavage、TAP transport、mass-spec evidence。输出是 peptide 是否被呈递或 pMHC abundance。

这类任务更接近 antigen presentation prediction，不需要 TCR 序列。

可能的标签：

- HLA binding affinity assay.
- monoallelic cell line ligandome MS.
- multi-allelic immunopeptidomics.
- cell surface pMHC abundance.
- stability assay.

注意：binding affinity 和 natural presentation 不是同一标签。presentation 还受蛋白表达和加工过程影响。

### Task 2: TCR-pMHC Binding

输入包括 TCRα/TCRβ sequence、V/J genes、peptide、HLA allele，最好还有结构或 predicted structure。输出可以是 tetramer binding、SPR affinity、dwell time 或 binary specificity。

这类任务应明确 label 是否来自 multimer staining、literature curation、binding assay 或 functional assay。

可能的建模层级：

- sequence-only model: TCR + peptide + HLA tokens.
- paired-chain model: TRA/TRB with V/J genes and CDR annotations.
- structure-aware model: predicted TCR-pMHC complex, contact map, interface graph.
- retrieval-augmented model: similar known TCRs, public motif, epitope family.
- uncertainty-aware model: flag unseen antigen/HLA/TCR regions.

### Task 3: T Cell Activation

输入除了 TCR/pMHC，还应尽量纳入 cell state、co-stimulation、antigen dose、APC type、assay protocol、time point 和 readout type。输出可以是 CD69/CD25、cytokine、proliferation、killing 等。

这类任务最有临床意义，但也最难，因为 label 混杂更多。

对 activation prediction，建议把输出拆成多任务：

- early marker: CD69, Nur77 reporter, phospho-ZAP70.
- cytokine: IL-2, IFN-gamma, TNF.
- cytotoxicity: CD107a, Granzyme B release, killing.
- proliferation: CFSE dilution, clone expansion.
- exhaustion/dysfunction: PD-1 high, TOX program, reduced cytokine polyfunctionality.

多任务设计可以让模型学到不同 readout 之间的关系，而不是把所有 positive 压成一个标签。

### Task 4: In Vivo Response or Therapy Outcome

输入还要包括 tumor microenvironment、HLA loss、tumor antigen expression、T cell infiltration、checkpoint status、prior treatment、patient-specific immune state。输出可能是 clonal expansion、tumor regression 或 survival。

这已经不是单纯 TCR-pMHC 任务，而是系统免疫学预测。

## Modeling Features Worth Trying

从机制角度看，下面这些特征或 inductive bias 可能有用：

- **TCR paired chain modeling.** 同时建模 TCRα 和 TCRβ，不只用 CDR3β。
- **V/J gene and CDR loop encoding.** CDR1/CDR2 与 MHC 接触，V gene 不应简单丢弃。
- **HLA-aware peptide representation.** peptide 的 anchor residues 和 exposed residues 对 MHC 与 TCR 分别重要。
- **Structure-informed interface features.** TCR-pMHC docking geometry、contact map、distance、electrostatics、buried surface area。
- **Kinetic proxy.** 如果没有实验 off-rate，可以尝试从结构稳定性、interface complementarity、docking ensemble 推断 proxy。
- **Assay-aware labels.** 把 tetramer、cytokine、killing、expansion 分开建模，或把 assay type 作为条件变量。
- **Dose and density.** pMHC abundance、antigen dose、TCR expression、APC type 会影响 activation threshold。
- **Uncertainty estimation.** TCR antigen space 极稀疏，OOD detection 比单点 accuracy 更重要。
- **Negative data hygiene.** 未观察到 binding 不等于 true negative；很多数据集的 negative 是 sampled negative。
- **Cross-reactivity modeling.** 一个 TCR 可识别多个相关或不相关 peptide，模型不能默认 one receptor-one antigen。

### Thymic Selection and Self-Restriction

TCR repertoire 在胸腺中经历 positive selection 和 negative selection。positive selection 保留能够识别 self-MHC 的 TCR，使成熟 T cells 具有 MHC restriction；negative selection 删除强 self-reactive TCR，建立 central tolerance。这个过程不是完美删除所有 self-reactivity，而是塑造一个对 self-pMHC 具有适中反应分布的 repertoire。[^klein2014selection]

这对 TCR-pMHC 模型很重要：

- Mature TCRs are biased to recognize host MHC-like surfaces.
- Self-reactivity is not zero; it is tuned.
- Treg selection can favor certain self-reactive TCRs.
- Tumor antigens often resemble self, so anti-tumor recognition sits close to tolerance boundary.
- Engineered high-affinity TCRs may escape natural thymic selection constraints, increasing cross-reactivity risk.

### Cross-Reactivity Is Required, Not an Accident

理论上，人体可能遇到的 peptide-MHC 空间远大于 T cell clone 数量。因此每个 TCR 必须具有一定 cross-reactivity，否则 repertoire coverage 不足。cross-reactivity 是免疫系统的必要性质，但也是 off-target toxicity 的来源。[^wooldridge2012crossreactivity][^moris2020crossreactivity]

对建模来说，目标不应该是把 TCR 映射到唯一 peptide，而应估计它可能识别的 ligand neighborhood。

### Minimal Data Schema

如果要为这个方向整理数据，我会至少保留下列字段：

| Category | Fields |
| --- | --- |
| TCR | TRA CDR1/2/3, TRB CDR1/2/3, V/J genes, full V(D)J nucleotide if available, clone size |
| Ligand | peptide sequence, source protein, mutation/wildtype, organism, antigen class |
| HLA/MHC | allele, class I/II, expression system |
| Assay | binding/activation/killing, method, threshold, time point, peptide dose |
| Cell context | T cell type, APC/target type, co-stimulation, cytokines, species |
| Outcome | binary label, continuous response, replicate statistics |
| Provenance | paper, database, donor, batch, confidence score |

### Candidate Model Designs

1. **Baseline sequence encoder.** Encode TCRα, TCRβ, peptide and HLA separately, then fuse with cross-attention.
2. **HLA-conditioned peptide encoder.** Model peptide residues in the context of HLA pockets, not as isolated peptide text.
3. **Interface graph model.** Build residue-level graph from predicted TCR-pMHC structure and learn contact patterns.
4. **Assay-conditioned predictor.** Add assay type, dose and readout as conditioning variables.
5. **Hierarchical model.** First predict presentation, then binding, then activation; propagate uncertainty.
6. **Contrastive retrieval.** Retrieve similar epitopes/TCR motifs and let model reason over neighbors.
7. **Calibration layer.** Calibrate scores within antigen family or HLA group to avoid overconfident OOD predictions.

### Loss Functions and Training Objectives

可以考虑的训练目标：

- binary cross entropy for curated positive/negative specificity.
- contrastive loss for matching TCR and cognate pMHC.
- metric learning to cluster TCRs recognizing same or similar epitopes.
- ranking loss for dose-response or potency data.
- regression loss for affinity, cytokine amount or killing fraction.
- multi-task loss across binding, activation and killing labels.
- positive-unlabeled learning when negatives are unreliable.
- survival/time-to-event loss for killing kinetics.

对于 public datasets，positive-unlabeled 或 weak-supervision 思路可能比强行构造大量 sampled negatives 更合理。

### Structure-Aware Modeling Details

结构模型可以有几种输入粒度：

| Level | Example representation |
| --- | --- |
| sequence | amino acid tokens, V/J gene IDs |
| annotated sequence | CDR boundaries, framework regions, chain labels |
| residue graph | nodes as residues, edges by distance/contact |
| interface graph | only TCR-pMHC interface residues |
| geometric features | distances, angles, normals, solvent accessibility |
| energy-like features | hydrogen bond, salt bridge, hydrophobic contacts |
| ensemble | multiple docked conformations or predicted structures |

关键问题是 TCR-pMHC structure prediction 本身也有误差。下游模型要么显式处理结构不确定性，要么使用 ensemble/uncertainty features。

### Evaluation Checklist

- Report performance by epitope, HLA, species and assay type.
- Use epitope-held-out and TCR-held-out splits.
- Remove exact and near-duplicate clonotypes across splits.
- Compare against simple baselines: CDR3 motif, edit distance, V gene frequency, HLA frequency.
- Test robustness to sampled negatives.
- Report calibration, not only AUROC.
- Inspect false positives for plausible cross-reactivity.
- Inspect false negatives for weak/tetramer-negative but functional cases.

### Metrics

不同任务应使用不同指标：

| Task | Useful metrics |
| --- | --- |
| binary specificity | AUROC, AUPRC, balanced accuracy, calibration |
| rare positive screening | precision@k, recall@k, enrichment factor |
| ranking candidate antigens | top-k hit rate, mean reciprocal rank |
| potency regression | Spearman correlation, RMSE, calibration by dose |
| multi-label antigen assignment | macro/micro F1, per-antigen AUPRC |
| OOD detection | AUROC for OOD, risk-coverage curve |
| clinical response | C-index, time-dependent AUC, decision curve |

因为 positive 很稀少，AUPRC 和 top-k enrichment 往往比 AUROC 更接近实际使用场景。

### Baselines That Should Not Be Skipped

在复杂模型前，应该先跑：

- nearest-neighbor by CDR3 edit distance.
- GLIPH-like motif clustering baseline.
- TCRdist-style distance baseline.
- peptide-HLA binding predictor baseline.
- V gene / J gene frequency baseline.
- antigen frequency baseline.
- simple logistic regression over hand-crafted features.

如果深度模型不能稳定超过这些 baseline，说明它可能主要学到了数据偏差。

## A Practical Mental Model

可以把 T cell activation 看成一个多级函数：

$$
Activation = f(TCR, peptide, HLA, structure, kinetics, dose, cell\ state, co\ signals, mechanics, microenvironment, assay)
$$

其中 TCR-pMHC binding 是必要变量之一，但不是最终函数。

对于当前阶段的建模，如果数据不足，我会优先做三件事：

1. **明确标签层级。** 先区分 binding、activation、killing、in vivo response。
2. **构建机制分层任务。** 先预测 peptide-HLA，再预测 TCR-pMHC，再预测 activation，避免端到端黑箱吞掉所有混杂。
3. **用结构和实验元数据减少噪声。** 把 HLA、assay type、species、cell type、antigen source、TCR chain pairing 放进数据 schema。

## Common Misconceptions

1. **“TCR binds peptide.”** 更准确地说，TCR recognizes peptide-MHC as a composite surface.
2. **“High affinity is always better.”** T cell function often depends on kinetic window, force, density and feedback.
3. **“Tetramer positive means functional.”** Tetramer binding is useful but not equal to cytokine production or killing.
4. **“Random TCR-peptide pairs are true negatives.”** Many are unlabeled unknowns, not experimentally verified negatives.
5. **“CDR3β is enough.”** It is informative but incomplete without TCRα, V/J, HLA and peptide context.
6. **“Neoantigen prediction ends at HLA binding.”** Presentation, TCR recognition and immune context are additional filters.
7. **“A single universal TCR-pMHC model should solve everything.”** Binding, activation and clinical response are different tasks.

## Open Questions

这些问题目前仍然很难：

- 如何从 sequence-only data 中可靠推断 TCR-pMHC cross-reactivity？
- 如何构造真正可信的 negative examples？
- 如何把 weak tetramer binding 与 strong functional activation 区分开？
- 如何在 HLA-held-out 或 epitope-held-out 场景中泛化？
- 如何把 TCR specificity 与 tumor microenvironment context 联合建模？
- 如何评价模型是否学到 mechanism，而不是 dataset shortcut？
- 如何将 structure prediction uncertainty 传递到 specificity prediction？
- 如何从单细胞多组学中定义高质量 activation labels？

这些 open questions 本身就可以拆成后续文章。

## A Possible Article Outline After Pruning

后续如果要把这篇素材库压缩成正式文章，可以保留这条主线：

1. Why TCR-pMHC matters for AI4Science.
2. TCR-pMHC structure and antigen presentation.
3. Binding vs activation.
4. Mechanistic models of TCR triggering.
5. Experimental readouts and datasets.
6. Implications for deep learning.

现在这版刻意偏“全”，之后可以按目标读者删掉过细的免疫学机制或建模工程细节。

## Keywords to Search Next

- TCR triggering
- TCR-pMHC kinetics
- kinetic proofreading
- serial triggering
- kinetic segregation
- TCR microclusters
- immunological synapse
- TCR catch bond
- TCR mechanotransduction
- peptide-MHC presentation
- HLA restriction
- T cell exhaustion
- cytotoxic T lymphocyte killing
- tetramer staining
- antigen-specific TCR database
- TCR cross-reactivity

## References

[^dong2019tcrcd3]: Dong, D. et al. Structural basis of assembly of the human T cell receptor-CD3 complex. *Nature*, 2019. [https://doi.org/10.1038/s41586-019-1537-0](https://doi.org/10.1038/s41586-019-1537-0)

[^rock2016mhc]: Rock, K. L., Reits, E., & Neefjes, J. Present Yourself! By MHC Class I and MHC Class II Molecules. *Trends in Immunology*, 2016. [https://doi.org/10.1016/j.it.2016.08.010](https://doi.org/10.1016/j.it.2016.08.010)

[^chen2013cosignaling]: Chen, L. & Flies, D. B. Molecular mechanisms of T cell co-stimulation and co-inhibition. *Nature Reviews Immunology*, 2013. [https://doi.org/10.1038/nri3405](https://doi.org/10.1038/nri3405)

[^courtney2018tcrsignaling]: Courtney, A. H., Lo, W.-L., & Weiss, A. TCR signaling: mechanisms of initiation and propagation. *Trends in Biochemical Sciences*, 2018. [https://doi.org/10.1016/j.tibs.2018.02.005](https://doi.org/10.1016/j.tibs.2018.02.005)

[^rossjohn2015tcrrecognition]: Rossjohn, J., Gras, S., Miles, J. J., Turner, S. J., Godfrey, D. I., & McCluskey, J. T cell antigen receptor recognition of antigen-presenting molecules. *Annual Review of Immunology*, 2015. [https://doi.org/10.1146/annurev-immunol-032414-112334](https://doi.org/10.1146/annurev-immunol-032414-112334)

[^schatz2011vdj]: Schatz, D. G. & Swanson, P. C. V(D)J recombination: mechanisms of initiation. *Annual Review of Genetics*, 2011. [https://doi.org/10.1146/annurev-genet-110410-132552](https://doi.org/10.1146/annurev-genet-110410-132552)

[^klein2014selection]: Klein, L., Kyewski, B., Allen, P. M., & Hogquist, K. A. Positive and negative selection of the T cell repertoire: what thymocytes see and do not see. *Nature Reviews Immunology*, 2014. [https://doi.org/10.1038/nri3667](https://doi.org/10.1038/nri3667)

[^wooldridge2012crossreactivity]: Wooldridge, L., Ekeruche-Makinde, J., van den Berg, H. A., Skowera, A., Miles, J. J., Tan, M. P., Dolton, G., Clement, M., Llewellyn-Lacey, S., Price, D. A., Peakman, M., & Sewell, A. K. A single autoimmune T cell receptor recognizes more than a million different peptides. *Journal of Biological Chemistry*, 2012. [https://doi.org/10.1074/jbc.M111.289488](https://doi.org/10.1074/jbc.M111.289488)

[^moris2020crossreactivity]: Moris, P., De Pauw, J., Postovskaya, A., Gielis, S., De Neuter, N., Bittremieux, W., Ogunjimi, B., Laukens, K., & Meysman, P. Current challenges for unseen-epitope TCR interaction prediction and a new perspective derived from image classification. *Briefings in Bioinformatics*, 2020. [https://doi.org/10.1093/bib/bbaa318](https://doi.org/10.1093/bib/bbaa318)

[^blum2013antigenprocessing]: Blum, J. S., Wearsch, P. A., & Cresswell, P. Pathways of antigen processing. *Annual Review of Immunology*, 2013. [https://doi.org/10.1146/annurev-immunol-032712-095910](https://doi.org/10.1146/annurev-immunol-032712-095910)

[^dushek2012kineticproofreading]: Dushek, O., Aleksic, M., Wheeler, R. J., Zhang, H., Cordoba, S.-P., Peng, Y.-C., Chen, J.-L., Cerundolo, V., Dong, T., Coombs, D., & van der Merwe, P. A. Antigen potency and maximal efficacy reveal a mechanism of efficient T cell activation. *Science Signaling*, 2011. [https://doi.org/10.1126/scisignal.2001430](https://doi.org/10.1126/scisignal.2001430)

[^huang2010tcrtriggering]: Huang, J. et al. The kinetics of two-dimensional TCR and pMHC interactions determine T-cell responsiveness. *Nature*, 2010. [https://doi.org/10.1038/nature08944](https://doi.org/10.1038/nature08944)

[^vdjdb]: Bagaev, D. V. et al. VDJdb in 2019: database extension, new analysis infrastructure and a T-cell receptor motif compendium. *Nucleic Acids Research*, 2020. [https://doi.org/10.1093/nar/gkz874](https://doi.org/10.1093/nar/gkz874)

[^mcpas]: Tickotsky, N., Sagiv, T., Prilusky, J., Shifrut, E., & Friedman, N. McPAS-TCR: a manually curated catalogue of pathology-associated T cell receptor sequences. *Bioinformatics*, 2017. [https://doi.org/10.1093/bioinformatics/btx286](https://doi.org/10.1093/bioinformatics/btx286)

[^tenximmune]: 10x Genomics. Chromium Single Cell Immune Profiling and Feature Barcode technology documentation. [https://www.10xgenomics.com/support/single-cell-vdj](https://www.10xgenomics.com/support/single-cell-vdj)

[^iedb]: Vita, R. et al. The Immune Epitope Database (IEDB): 2018 update. *Nucleic Acids Research*, 2019. [https://doi.org/10.1093/nar/gky1006](https://doi.org/10.1093/nar/gky1006)

[^netmhcpan]: Reynisson, B. et al. NetMHCpan-4.1 and NetMHCIIpan-4.0: improved predictions of MHC antigen presentation by concurrent motif deconvolution and integration of MS MHC eluted ligand data. *Nucleic Acids Research*, 2020. [https://doi.org/10.1093/nar/gkaa379](https://doi.org/10.1093/nar/gkaa379)

[^mhcflurry]: O'Donnell, T. J. et al. MHCflurry: Open-source class I MHC binding affinity prediction. *Cell Systems*, 2018. [https://doi.org/10.1016/j.cels.2018.05.014](https://doi.org/10.1016/j.cels.2018.05.014)

[^mckeithan1995kinetic]: McKeithan, T. W. Kinetic proofreading in T-cell receptor signal transduction. *Proceedings of the National Academy of Sciences*, 1995. [https://doi.org/10.1073/pnas.92.11.5042](https://doi.org/10.1073/pnas.92.11.5042)

[^valitutti1995serial]: Valitutti, S., Müller, S., Cella, M., Padovan, E., & Lanzavecchia, A. Serial triggering of many T-cell receptors by a few peptide-MHC complexes. *Nature*, 1995. [https://doi.org/10.1038/375148a0](https://doi.org/10.1038/375148a0)

[^davis2006kineticsegregation]: Davis, S. J. & van der Merwe, P. A. The kinetic-segregation model: TCR triggering and beyond. *Nature Immunology*, 2006. [https://doi.org/10.1038/ni1369](https://doi.org/10.1038/ni1369)

[^grakoui1999synapse]: Grakoui, A. et al. The immunological synapse: a molecular machine controlling T cell activation. *Science*, 1999. [https://doi.org/10.1126/science.285.5425.221](https://doi.org/10.1126/science.285.5425.221)

[^yokosuka2005microclusters]: Yokosuka, T. et al. Newly generated T cell receptor microclusters initiate and sustain T cell activation by recruitment of Zap70 and SLP-76. *Nature Immunology*, 2005. [https://doi.org/10.1038/ni1242](https://doi.org/10.1038/ni1242)

[^adams2016tcrrecognition]: Adams, J. J., Narayanan, S., Birnbaum, M. E., Sidhu, S. S., Blevins, S. J., Gee, M. H., Sibener, L. V., Baker, B. M., Kranz, D. M., & Garcia, K. C. Structural interplay between germline interactions and adaptive recognition determines the bandwidth of TCR-peptide-MHC cross-reactivity. *Nature Immunology*, 2016. [https://doi.org/10.1038/ni.3310](https://doi.org/10.1038/ni.3310)

[^liu2014catchbond]: Liu, B., Chen, W., Evavold, B. D., & Zhu, C. Accumulation of dynamic catch bonds between TCR and agonist peptide-MHC triggers T cell signaling. *Cell*, 2014. [https://doi.org/10.1016/j.cell.2014.02.053](https://doi.org/10.1016/j.cell.2014.02.053)

[^zhu2019mechanosensing]: Zhu, C., Chen, W., Lou, J., Rittase, W., & Li, K. Mechanosensing through immunoreceptors. *Nature Immunology*, 2019. [https://doi.org/10.1038/s41590-019-0491-1](https://doi.org/10.1038/s41590-019-0491-1)

[^voskoboinik2015perforin]: Voskoboinik, I., Whisstock, J. C., & Trapani, J. A. Perforin and granzymes: function, dysfunction and human pathology. *Nature Reviews Immunology*, 2015. [https://doi.org/10.1038/nri3839](https://doi.org/10.1038/nri3839)

[^martinezlostao2015death]: Martínez-Lostao, L., Anel, A., & Pardo, J. How Do Cytotoxic Lymphocytes Kill Cancer Cells? *Clinical Cancer Research*, 2015. [https://doi.org/10.1158/1078-0432.CCR-15-0685](https://doi.org/10.1158/1078-0432.CCR-15-0685)

[^pardoll2012checkpoint]: Pardoll, D. M. The blockade of immune checkpoints in cancer immunotherapy. *Nature Reviews Cancer*, 2012. [https://doi.org/10.1038/nrc3239](https://doi.org/10.1038/nrc3239)

[^thommen2018exhaustion]: Thommen, D. S. & Schumacher, T. N. T cell dysfunction in cancer. *Cancer Cell*, 2018. [https://doi.org/10.1016/j.ccell.2018.03.012](https://doi.org/10.1016/j.ccell.2018.03.012)
