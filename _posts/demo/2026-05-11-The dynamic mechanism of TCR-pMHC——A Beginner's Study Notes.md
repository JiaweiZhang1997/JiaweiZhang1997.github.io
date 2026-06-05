---
layout: post
title: "The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes"
subtitle: "T细胞免疫学常识以及激活机制学习笔记."
date: 2026-05-11 00:30:00 +0800
author: Jiawei Zhang
reading_time: 55 min
tags:
  - Biology
  - TCR-pMHC
  - AI4Science
excerpt: "T细胞到底是如何被激活的呢？这篇笔记从TCR-pMHC背景知识、抗原呈递、公开数据集的数据构成、Tcell激活机制和建模需求几个层面做一个尽量完整的入门梳理。"
---

T细胞（T cell），是人体免疫系统中至关重要的一部分，属于适应性免疫系统的核心主力，它可以识别由MHC呈递的表位，进而杀死对应的有害细胞。它们就像是人体内的“特种部队”，可以精准识别并摧毁被病毒感染的细胞和肿瘤细胞。TCR-T，TILs等免疫疗法以及肿瘤疫苗等均是基于T细胞可以杀伤肿瘤细胞的特性开发的治疗方案。

**TCR-T 细胞疗法**：提取患者外周血中的普通 T 细胞，在体外通过基因工程技术导入编码特异性 T 细胞受体（TCR）的基因，使其能够特异性识别并结合肿瘤细胞呈递的细胞内部抗原（pMHC 复合体），在体外大量扩增后回输给患者，以此实现对特定实体瘤细胞的靶向杀伤。

**TIL 疗法**：通过手术切除获取患者的肿瘤组织，在体外分离出其中天然已经浸润到肿瘤内部、具备肿瘤抗原识别能力的淋巴细胞（TIL），利用细胞因子在体外进行大规模扩增和活化，再将其回输到经过清淋处理的患者体内，利用其天然的多克隆靶点识别能力杀灭肿瘤细胞。

**治疗性肿瘤疫苗**：通过对患者肿瘤进行基因测序和算法筛选，找出肿瘤特异性的突变新抗原表位，将其制成 mRNA、多肽或细胞疫苗注射入患者体内，通过激活体内的抗原呈递细胞来诱导、扩增患者自身体内具备肿瘤特异性的 T 细胞，从而达到清除残留肿瘤病灶和预防复发的目的。

这些方法大都需要筛选出可以互相识别的TCR-pMHC对，所以TCR-pMHC pair（这里说binding不太准确，所以改用了pair，因为核心目标是激活T-cell，后续会做具体的说明）预测模型也就应运而生。在两年前我拿到这个任务时，对免疫几乎一无所知。所以当时的我选择使用了时下最普遍的策略，使用对序列的mask learning任务做预训练，让模型学习一些分布规律，后通过data-driven的方式做微调，直接学习TCR-pMHC的pair模式。虽然顺利发了文章，但在训练时我们就发现，TCR-pMHC的训练数据有非常多问题：数据少，测量标准不统一，抗原的分布严重有偏；TCR由于vdj重排，导致TCR-pMHC pair的特征空间是十分巨大的；又加之TCR-pMHC 能否pair对突变十分敏感。导致在现有的数据量下，难以训练一个准确且可以泛化的TCR-pMHC pair预测模型，大部分文章都是在自己的测试集上表现还不错，一旦迁移，就基本等效于随机了。

也是从这开始，我越来越强烈地感觉到理解课题的重要性，特别是在数据不足的领域。如果想要真正解决问题，必须理解其中的机制，通过先验知识约束可能的搜索空间，完成更优雅的工作，得到更优的结果。T cell activation 看似是简单的“是否激活”标签，背后却混合了抗原加工、HLA 呈递、TCR 序列、三维结构、细胞状态、共刺激、实验体系和微环境等多层因素。因此，人类先验不是“可有可无的解释”，而是帮助模型缩小搜索空间、定义合理输入输出、发现标签偏差的重要工具。为了更好地处理 TCR-pMHC pair，更准确地说，是 **T cell activation** 任务，我开始尝试理解其中的背景知识和关键机制，于是有了这篇学习笔记，在帮助自己梳理过去生物知识的同时了解最新的免疫学前沿理论，以便更好的构建T cell 激活预测模型。

<div class="blog-callout">
  <strong>本文定位。</strong>
  这不是正式的综述论文，而是一份面向建模问题的入门级机制科普。重点在于理解：T cell的各种背景知识以及 T cell激活的机制（本文主要涉及的是$\alpha\beta$ $\text{CD8}^+$ T 细胞），以及哪些生物学变量可能成为机器学习模型的关键特征、混杂因素或标签来源。
</div>

## 关键名词说明

### 1. MHC / HLA（主要组织相容性复合体 / 人类白细胞抗原）

在人类中，HLA 主要分为 **I 类（Class I）** 和 **II 类（Class II）**，它们的分子结构和多聚体状态截然不同：

#### HLA-I 类分子（所有受检细胞表面，具体结构见Figure 1a）

* **多聚体状态**：**异二聚体（Heterodimer）**。由一条重链（$\alpha$ 链）和一条轻链（$\beta_2$-微球蛋白，$\beta_2$m）通过非共价键连接组成。
* **分子量与长度**：$\alpha$ 链约 45 kDa（约 340 个氨基酸）；$\beta_2$m 约 12 kDa（约 99 个氨基酸）。
* **功能区划分**：
  * **抗原结合槽（Antigen-binding cleft）**：由 $\alpha_1$ 和 $\alpha_2$ 结构域共同构成。这是一个**封闭式**的槽状结构，两端闭合。
  * **免疫球蛋白样区**：$\alpha_3$ 结构域（负责结合 T 细胞表面的 CD8 共受体）和独立的 $\beta_2$m。
  * **跨膜区与胞质尾区**：仅由 $\alpha$ 链延伸出跨膜螺旋锚定在细胞膜上。


#### HLA-II 类分子（专职抗原呈递细胞表面，如 DC、B细胞、巨噬细胞，具体结构见Figure 1b）

* **多聚体状态**：**异二聚体（Heterodimer）**。由一条 $\alpha$ 链和一条 $\beta$ 链通过非共价键连接组成。
* **分子量与长度**：$\alpha$ 链约 33-35 kDa（约 230-250 个氨基酸）；$\beta$ 链约 28-30 kDa（约 230-240 个氨基酸）。
* **功能区划分**：
  * **抗原结合槽**：由 $\alpha_1$ 和 $\beta_1$ 结构域共同构成。与 I 类不同，其结合槽是**开放式**的（两端敞开）。
  * **免疫球蛋白样区**：$\alpha_2$ 和 $\beta_2$ 结构域（其中 $\beta_2$ 负责结合 T 细胞表面的 CD4 共受体）。
  * **跨膜区与胞质尾区**：$\alpha$ 链和 $\beta$ 链均拥有各自独立的跨膜区和胞质短尾。

<figure style="text-align: center;">
  <img src="/_posts/assets/2026-05-11-The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes/MHC_strcuture.png" alt="MHC 结构" style="max-width: 100%; height: auto;">
  <figcaption style="color: #777; font-weight: bold; text-align: center;">
    Figure 1: MHC 结构
  </figcaption>
</figure>

#### 每个人体内的 HLA 数量与组成情况

HLA（人类白细胞抗原）基因复合体位于人类 **6号染色体短臂（6p21.3）**。由于人类是二倍体（父母各给一套），每个人体内表达的 HLA 分子数量有着严格的遗传学边界。

##### 核心经典 HLA 分子的分类

临床与生物信息学中最核心关注的是**经典 HLA 分子（Classical HLA）**：

* **HLA-I 类**：包括 **HLA-A、HLA-B、HLA-C** 三个主要座位。
* **HLA-II 类**：包括 **HLA-DP、HLA-DQ、HLA-DR** 三个主要座位。

##### 每个人体内到底有多少种不同的 HLA 蛋白？

因为 HLA 基因是**共显性（Codominant）表达**，即来自父亲和母亲的等位基因会同时在细胞表面表达。

* **理论最大数量（完全异质结）**：
  * **I 类分子**：A、B、C 三个座位，父母各给一套不同的，即 $3 \times 2 = \mathbf{6}$ **种**不同的 HLA-I 类重链蛋白。
  * **II 类分子**：DP、DQ、DR 的 $\alpha$ 链和 $\beta$ 链父母各给一套。由于 II 类分子是 $\alpha/\beta$ 异二聚体，**来自父亲的 $\alpha$ 链可以和来自母亲的 $\beta$ 链错配组装**。因此，每个座位理论上可以衍生出 4 种组合，总计最大可表达约 **12–16 种**不同的 HLA-II 类复合物。
* **理论最小数量（极端纯合子）**：如果父母高度近亲结婚，给的 HLA 基因完全一模一样，那这个人人体内就只有 **3种 I 类** 和 **3种 II 类** 分子。

---

##### HLA 的遗传情况：单倍型与连锁不平衡

HLA 复合体是人类基因组中**多态性最高、最复杂**的区域，其遗传遵循非常独特的规律：

###### 单倍型遗传（Haplotype Inheritance）

因为 HLA 的各个座位（A, B, C, DR, DQ, DP）在 6 号染色体上排列得极其紧密，它们在减数分裂产生生殖细胞时，**极少发生同源染色体交叉重组**（重组率 $< 1\%$）。

* 父亲或母亲通常是将一整条染色体上的 HLA 基因群作为一个整体（称为单倍型，Haplotype）原封不动地传给下一代。
* **亲代与子代**：任意一个子女，必然有一半的 HLA 单倍型与父亲完全相同，另一半与母亲完全相同。
* **同胞手足之间**：根据孟德尔遗传定律，亲兄弟姐妹之间，HLA 完全相同的概率是 **25%**，完全不同的概率是 **25%**，一半相同的概率是 **50%**。这也是骨髓移植常在亲兄弟姐妹中寻找供体的原因。


### 2. 抗原（Antigen / 肽段 Peptide）

这里特指被 HLA 呈递的**内源性/外源性抗原短肽（Epitope）**。

* **多聚体状态**：**单链线性的寡肽（Oligopeptide）**。
* **多肽长度（核心差异）**：
  * **I 类 HLA 呈递的肽段**：通常非常固定，精确在 **8–11 个氨基酸**。因为 I 类结合槽两端闭合，肽段必须像一条弓起来的弦一样卡在槽内。
  * **II 类 HLA 呈递的肽段**：长度较长且不固定，通常在 **13–25 个氨基酸**（核心结合序列通常为 9 个氨基酸，两端延伸）。因为 II 类结合槽两端开放，长肽可以像长面条一样悬垂在槽外。
* **功能区划分**：
  * **锚定位点（Anchor residues）**：通常是肽段的特定位置（如 I 类肽的第 2 位和第 9 位氨基酸），其侧链深埋进 HLA 结合槽的口袋（Pockets）中，决定结合的稳固度。
  * **TCR 接触位点（TCR-contacting residues）**：肽段中部朝向外侧、暴露于溶剂中的氨基酸侧链，直接与 TCR 发生物理接触并决定特异性识别。

<figure style="text-align: center;">
  <img src="/_posts/assets/2026-05-11-The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes/MHC2.png" alt="在MHC结合槽中的peptide" style="max-width: 100%; height: auto;">
  <figcaption style="color: #777; font-weight: bold; text-align: center;">
    Figure 2: 在MHC结合槽中的peptide
  </figcaption>
</figure>

### 3. T cell

#### TCR（T 细胞受体，见Figure 3）

目前人体内 95% 以上的常规 T 细胞表达的都是 **$\alpha\beta$ TCR**（少部分为 $\gamma\delta$ TCR）。

* **多聚体状态**：**异二聚体（Heterodimer）**。由一条 $\alpha$ 链和一条 $\beta$ 链通过**二硫键**共价连接。
* **分子量与长度**：总分子量约为 90 kDa。$\alpha$ 链约 40-45 kDa（约 250-270 个氨基酸）；$\beta$ 链约 45-50 kDa（约 260-290 个氨基酸）。
* **功能区划分**：
  两条链的胞外部分均由两个高度保守的球状结构域构成（类似于抗体的 Fab 段）：
  * **可变区（Variable region, V$\alpha$ / V$\beta$）**：位于最外端。
  * 包含核心功能区——**CDR 环（互补决定区，CDR1, CDR2, CDR3）**。每条链有 3 个 CDR。
  * **CDR1 和 CDR2** 主要负责识别和绑定 HLA 分子的 $\alpha$-螺旋（保守基质）。
  * **CDR3**（由基因重排决定，多样性最高）集中在中间，直接与**抗原短肽**接触，是决定 T 细胞特异性的绝对核心。
  * **恒定区（Constant region, C$\alpha$ / C$\beta$）**：紧邻细胞膜，结构相对固定，起到支撑可变区和传递活化信号的作用。
  * **连接区、跨膜区与胞质尾区**：TCR 拥有带正电荷的跨膜区，这使得它必须在细胞膜上与带负电荷的 **CD3 复合物**（由 $\gamma, \delta, \epsilon, \zeta$ 链组成的寡聚体）结合，形成 **TCR-CD3 复合物（八聚体高阶大分子）**，才能真正把激活信号传导给 T 细胞内部。

<figure style="text-align: center;">
  <img src="/_posts/assets/2026-05-11-The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes/tcr_on_member2.png" alt="在细胞膜上的TCR-cd3复合物" style="max-width: 100%; height: auto;">
  <figcaption style="color: #777; font-weight: bold; text-align: center;">
    Figure 3: 在细胞膜上的TCR-cd3复合物
  </figcaption>
</figure>


#### V(D)J 重排：TCR 多样性的物理演操

T 细胞要识别海量的外源病原体和肿瘤新抗原，其核心依赖的是 TCR 可变区（特别是 CDR3 环）的超高多样性。人体内每天能产生多达 $10^{15}$–$10^{18}$ 种不同序列的 TCR，但人类基因组一共只有约 2 万个蛋白质编码基因。这种“少基因产生多抗体/受体”的机制，正是通过 **V(D)J 重排（V(D)J Recombination）** 实现的。

##### 基因片段的“大乐透”组合

在 T 细胞的发育阶段（胸腺内），TCR 的编码基因片段是断裂、分散排列的。这些片段主要分为三类：

* **V 片段（Variable，可变区）**：数量最多，决定受体的主要结构。
* **D 片段（Diversity，多样性区）**：**仅存在于 TCR $\beta$ 链和 $\delta$ 链中**，$\alpha$ 链和 $\gamma$ 链没有 D 片段。
* **J 片段（Joining，连接区）**：负责将 V/D 片段连接到恒定区（C区）。

重排时，淋巴细胞特异性重组酶（**RAG-1 和 RAG-2**）会介入，随机挑选一个 V 片段、一个 D 片段（如果是 $\beta$ 链）和一个 J 片段，将它们之间的内含子序列切除，然后将选中的片段拼接在一起。

$$\text{TCR }\alpha\text{ 链重排} = V_\alpha + J_\alpha$$

$$\text{TCR }\beta\text{ 链重排} = V_\beta + D_\beta + J_\beta$$

##### 多样性的三大来源

* **组合多样性（Combinatorial Diversity）**：不同 V、D、J 片段的随机组合。例如 $\beta$ 链有约 40 个 $V_\beta$、2 个 $D_\beta$、13 个 $J_\beta$，组合方式就有 $40 \times 2 \times 13 = 1040$ 种。加上 $\alpha$ 链的随机组合，数量呈几何级数增长。
* **连接多样性（Junctional Diversity）【关键】**：在片段拼接的断裂点上，末端脱氧核苷酸转移酶（**TdT**）会**随机删去或凭空插入**几个核苷酸（N区插入）。这直接导致了氨基酸序列的改变和移码，使得即使选中了完全相同的 V、D、J 片段，最终生成的 TCR 编码也完全不同。这也是 **CDR3 环（直接接触抗原短肽的区域）** 极其多变的核心原因。
* **$\alpha$ 链与 $\beta$ 链的配对多样性**：体外独立重排完毕的 $\alpha$ 链与 $\beta$ 链，在空间上随机结合成异二聚体。

<figure style="text-align: center;">
  <img src="/_posts/assets/2026-05-11-The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes/vdj.png" alt="vdj重排" style="max-width: 100%; height: auto;">
  <figcaption style="color: #777; font-weight: bold; text-align: center;">
    Figure 4: vdj重排
  </figcaption>
</figure>


#### CD3 的家族组成与结构

正如我们之前在冷冻电镜结构中看到的，一个完整的 **TCR-CD3 复合物**（Figure 3） 是一个高阶的八聚体（Octamer）大分子机器，由 1 个 TCR 二聚体和 3 个 CD3 二聚体紧密铰链而成。

* **$\text{CD3}\epsilon\gamma$ 异二聚体**：由 $\epsilon$ 链（绿色）和 $\gamma$ 链（灰色）组成，贴在 TCR 的一侧。
* **$\text{CD3}\epsilon\delta$ 异二聚体**：由 $\epsilon$ 链（绿色）和 $\delta$ 链（粉色）组成，贴在 TCR 的另一侧。
* **$\text{CD3}\zeta\zeta$ 同二聚体**：由两条完全相同的 $\zeta$ 链（青色）组成，深埋在复合物的内部核心。



## Tcell 激活
影响Tcell激活的受体有很多（如CD4，CD8，CD28，CD25），以下主要讨论的是$\alpha\beta$ $\text{CD8}^+$ T 细胞以及通过TCR传达的信号，因为多样性主要来自TCR。

### MHC呈递表位（Figure 5）
首先，TCR只能识别由MHC呈递的表位。
HLA-I 类分子负责呈递内源性抗原，因此是杀伤肿瘤细胞的主力军，也是本文的重点。正常的有核细胞，内部会不断降解旧蛋白并合成新蛋白。细胞内的胞质溶胶会把这些蛋白切成 8-11 个氨基酸的短肽，在内质网处由 HLA-I 类分子包裹形成pMHC复合物，运输到细胞表面。$\text{CD8}^+$ 杀伤性T 细胞发现被呈递的抗原不是自体的蛋白分解的正常短肽（细胞发生癌变产生了肿瘤新抗原，或被病毒感染）时，会直接释放穿孔素和颗粒酶，启动细胞凋亡程序。

HLA-II 类分子负责呈递外来的细菌、外源毒素等。它们会通过内吞或胞吞作用把敌人“吃”进细胞内，在溶酶体里将其消化成 13-25 个氨基酸的长肽，同样在内质网中将其放进 HLA-II 类分子的开放式结合槽中，运输到膜表面展示出来，等待被$\text{CD4}^+$ 辅助性 T 细胞（Helper T cells）。$\text{CD4}^+$ T 细胞识别后不会亲自去杀敌，而是开始疯狂合成分泌各种细胞因子（如 IL-2、IFN-$\gamma$ 等），指挥 B 细胞开始批量生产抗体，或者召集巨噬细胞杀死有害的细胞。
速记小技巧：HLA-I与$\text{CD8}^+$ T 细胞，HLA-II与$\text{CD4}^+$ T 细胞，相乘都是8（1*8=2*4=8）。

<figure style="text-align: center;">
  <img src="/_posts/assets/2026-05-11-The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes/MHC_present_peptide.png" alt="MHC 呈递peptide流程" style="max-width: 100%; height: auto;">
  <figcaption style="color: #777; font-weight: bold; text-align: center;">
    Figure 5: MHC 呈递peptide流程
  </figcaption>
</figure>

### TCR识别epitope-MHC复合物

T 细胞受体（TCR）是 T 细胞表面负责特异性识别抗原的关键分子。与抗体不同，TCR 的识别遵循严格的MHC限制性（双重识别）原则。

也就是说，TCR 绝对不会直接识别游离的外源抗原，它必须同时识别“抗原表位”和“MHC分子”。如果被呈递上来的是非自体的 MHC 分子，或者自体 MHC 分子呈递的是正常的自体蛋白短肽，TCR 均不会被激活。只有当自体的 MHC 分子呈递了异常抗原（如肿瘤新抗原或病毒多肽）时，TCR 才能真正启动特异性免疫反应。

在微观结构上，绝大多数 TCR 由 $\alpha$ 和 $\beta$ 两条链组成，其顶端带有高度可变的**互补决定区（CDR）**。在识别 pMHC（多肽-MHC）复合物时，这些区域有着精密的分工：

* **CDR1 和 CDR2**：主要负责识别 MHC 分子顶部的 $\alpha$ 螺旋，确认呈递抗原的载体是自体分子。
* **CDR3**：具有极高的序列多样性，它会直接深入 MHC 的抗原结合槽中央，精准识别并结合抗原短肽上的特异性核心氨基酸序列（表位）。

<figure style="text-align: center;">
  <img src="/_posts/assets/2026-05-11-The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes/tcr_detail.png" alt="TCR-pMHC 复合物" style="max-width: 100%; height: auto;">
  <figcaption style="color: #777; font-weight: bold; text-align: center;">
    Figure 6: TCR-pMHC 复合物
  </figcaption>
</figure>

### TCR-CD3复合物激活下游信号 

然而，TCR 虽然负责精准识别，但其胞内区极短，无法独立将激活信号传达给细胞内部。因此，它必须与 **CD3 分子** 结合，形成完整的 TCR-CD3 复合体。当 TCR 在胞外成功结合 pMHC 后，CD3 分子会利用其胞内段的 ITAM 基团，将激活信号级联传递至细胞核内，从而唤醒 T 细胞的杀伤或辅助功能。

CD3 的每一条链都长着一条常常的拖入细胞质内部的“尾巴”，这些尾巴上密布着信号传导的核心元件——**ITAM（免疫受体酪氨酸活化基序，Immunoreceptor Tyrosine-based Activation Motif）**。

* $\gamma$ 链、$\delta$ 链、$\epsilon$ 链的胞质尾区：各含有 **1 个** ITAM。
* $\zeta$ 链的胞质尾区：异乎寻常的长，每条 $\zeta$ 链单独含有 **3 个** ITAM（$\zeta\zeta$ 二聚体共含有 **6 个**）。

因此，一个完整的 TCR-CD3 八聚体，在细胞质内部一共悬挂了 **10 个 ITAM 信号弹**。这种高密度的信号位点设计，是 T 细胞能够产生级联放大、对极微量抗原产生“触之即发”反应的物理基础。

<figure style="text-align: center;">
  <img src="/_posts/assets/2026-05-11-The dynamic mechanism of TCR-pMHC——A Beginner's Study Notes/tcr_detail.png" alt="TCR-CD3 Complex Bound to peptide-MHC I" style="max-width: 100%; height: auto;">
  <figcaption style="color: #777; font-weight: bold; text-align: center;">
    Figure 7: TCR-CD3 Complex Bound to peptide-MHC I
  </figcaption>
</figure>

#### 阶段一：引爆信号弹（受体近端激酶的招募与磷酸化）

这是物理识别转化为化学信号的关键第一步。

**Lck 激酶的靠近与激活**  
当 TCR 与抗原提呈细胞（APC）上的 pMHC 结合时，T 细胞的共受体（CD4 或 CD8）也会结合到 MHC 分子的保守区域。CD4/CD8 的胞内段携带着一种名为 Lck（一种 Src 家族酪氨酸激酶）。共受体的结合直接将 Lck 拉到了 CD3 复合体那长长的胞质尾巴附近。

**ITAM 的磷酸化**  
靠近后的 Lck 迅速对 CD3 各条链上的 ITAM 进行磷酸化（Phosphorylation）修饰。每个 ITAM 包含两个关键的酪氨酸（Tyrosine）残基（序列特征为 $YxxL/I(x)_{6-8}YxxL/I$）。Lck 会将这两个酪氨酸双双磷酸化。$\zeta\zeta$ 同二聚体上的 6 个 ITAM 尤为关键，它们的高效磷酸化是后续信号放大的绝对核心。

#### 阶段二：招募信号放大器（ZAP-70 的结合与激活）

磷酸化的 ITAM 改变了自身的构象和化学性质，成为了完美的“停机坪”。

**ZAP-70 的招募**  
细胞质中游离着一种名为 ZAP-70（Syk 家族激酶）的关键蛋白。ZAP-70 包含两个串联的 SH2 结构域。这两个 SH2 结构域会精准、高亲和力地结合到双磷酸化的 ITAM 上。此时，ZAP-70 被锚定到了细胞膜内侧的 TCR-CD3 复合物上。

**ZAP-70 的激活**  
结合在 ITAM 上的 ZAP-70 恰好暴露在了 Lck 激酶的火力范围内。Lck 顺势对 ZAP-70 进行磷酸化，将其彻底激活。至此，原始的 TCR 结合信号已经成功转移给了 ZAP-70，ZAP-70 将作为超级放大器开启下游反应。

在此过程中，前面提到的 $\text{CD8}^+$ 或 $\text{CD4}^+$ 分子作为辅助受体也发挥着不可或缺的作用。它们会结合在 MHC 分子侧面的非多态区，进一步稳定 TCR 与 pMHC 之间的空间结合结构，使激活信号的传导更加强烈且持久。

## Tcell 激活机制的各种假说

前面已经说明了一个相对清晰的下游通路：TCR 识别 pMHC，CD3 的 ITAM 被 Lck 磷酸化，ZAP-70 被招募并激活，随后信号进入 LAT/SLP-76、Ca$^{2+}$-NFAT、MAPK/AP-1、NF-$\kappa$B 等经典通路。

但真正困难的问题其实在更前面：**TCR 到底是如何判断一个 pMHC 值不值得点火的？** 这个问题非常反直觉。T 细胞一方面要有极高的特异性，能够在大量正常 self-pMHC 背景里识别出只有一两个氨基酸差异的异常抗原；另一方面又要有极高的敏感性，有时靶细胞表面只有很少量的 agonist pMHC，也足以触发 T 细胞响应。

所以，T cell activation 并不是一个单一开关，而更像一套多层安检系统：分子结合动力学、膜表面空间组织、机械力、受体聚集、细胞骨架、共受体和反馈调控都在参与判断。下面这些假说并不互相排斥，它们更像是从不同尺度解释同一件事。

### 1. 动力学校对假说（Kinetic Proofreading）

**动力学校对（Kinetic Proofreading）** 是解释 TCR 特异性的经典模型。它的核心思想很简单：TCR-pMHC 结合以后，并不是马上激活 T 细胞，而是要按顺序完成一串信号步骤，比如 CD3 ITAM 磷酸化、ZAP-70 招募、ZAP-70 激活、LAT signalosome 形成等。

如果一个 pMHC 只是很弱地、很短暂地碰了一下 TCR，那么它可能在第 1 步或第 2 步之前就已经解离了，信号链条来不及走完，T 细胞就不会真正被激活。只有那些能停留足够久的 pMHC，才有机会把这串“校验流程”走到最后。

可以把它想象成一个多级密码锁：

$$
TCR\text{-}pMHC \rightarrow C_1 \rightarrow C_2 \rightarrow C_3 \rightarrow \cdots \rightarrow C_n \rightarrow \text{Activation}
$$

每一级都需要 TCR-pMHC 复合物没有提前散开。这样一来，哪怕两个肽段的结合寿命只差一点点，经过多步放大后，也可能变成“强激活”和“无响应”的巨大差异。

这个模型对建模很重要，因为它提醒我们：不能只看 TCR 和 peptide 是否能结合，还要关心 **off-rate、dwell time、信号步骤数量、反馈强度和 readout 时间点**。一个 pair 在 binding assay 里看起来是 positive，并不等于它一定能完成完整 activation。

### 2. 连续触发假说（Serial Triggering）

**连续触发（Serial Triggering）** 试图解释另一个现象：为什么极少量的 pMHC 也能诱导 T 细胞产生明显反应？

这个假说认为，一个 agonist pMHC 并不是只服务一个 TCR。它可以先结合一个 TCR，触发一部分早期信号，然后解离出来，再去结合下一个 TCR。也就是说，一个 pMHC 像一个可以反复点火的火柴头，在膜表面连续触发多个 TCR，从而把非常稀少的抗原信号放大。

这也解释了为什么 TCR-pMHC 的亲和力并不是越高越好。如果结合太弱，信号来不及积累；但如果结合太强，pMHC 被一个 TCR 长时间占住，反而不利于它去连续触发更多 TCR。真正有效的激活可能存在一个动力学窗口：结合要足够久，可以完成早期校对；但又不能久到完全失去周转能力。

### 3. 动力学隔离假说（Kinetic Segregation）

**动力学隔离（Kinetic Segregation）** 从膜表面空间尺寸的角度解释 TCR 如何启动信号。

在静息状态下，T 细胞膜上同时存在点火方和灭火方。点火方是 Lck 这类激酶，负责给 CD3 ITAM 加磷酸；灭火方是 CD45 这类磷酸酶，负责把磷酸基团去掉。CD45 的胞外段很大，像一个巨大的拖把，平时可以不断清除偶发的弱磷酸化信号，让 T 细胞保持安静。

当 TCR 与 pMHC 结合时，T 细胞膜和 APC/靶细胞膜会在局部被拉得很近。TCR-pMHC 复合物本身比较短，形成的是一个狭窄的 close contact zone。这个空间足够容纳 TCR、pMHC、CD3 和 Lck，却容不下胞外段很大的 CD45。

结果就是：CD45 被物理排挤出局，局部区域从“激酶和磷酸酶互相拉扯”变成“激酶占优”。Lck 终于获得了一个相对干净的点火环境，可以高效磷酸化 CD3 ITAM。

### 4. 机械感应与逆境键假说（Mechanosensing / Catch Bond）

T 细胞并不是静静地等待抗原飘过来。它会在 APC 或靶细胞表面扫描、爬行、伸出微绒毛，并通过细胞骨架给 TCR-pMHC 复合物施加微小的机械力。

传统直觉认为，两个分子结合后越拉越容易分开，这叫 **slip bond**。但一些 TCR-pMHC 相互作用表现出类似 **catch bond（逆境键）** 的行为：在一定范围内，外力不是让它更快散开，反而让它咬得更紧、寿命更长。

这给 TCR 的抗原判别提供了一个非常漂亮的物理机制：

* 如果遇到的是普通 self-pMHC，外力一拉，复合物很快滑脱，信号来不及积累。
* 如果遇到的是真正的 agonist pMHC，外力会让 TCR-pMHC 的结合构象进入更稳定状态，延长 bond lifetime，并把机械变化传递到 TCR-CD3 复合物。

结合结构来看，这种力可能通过 TCR 的可变区、恒定区和跨膜区向下传递，改变 CD3 胞外段和胞质尾区的状态，让原本贴近膜的 ITAM 更容易暴露给 Lck，已经有一些文章在尝试通过这个机制优化TCR，并取得了一些进展。


### 5. 构象改变假说（Conformational Change）

**构象改变假说** 认为，TCR 与 pMHC 结合不仅仅是两个表面贴在一起，还可能引发 TCR-CD3 复合物内部的结构变化。

例如，TCR 的 CDR loops、peptide、MHC $\alpha$ 螺旋、TCR 恒定区或跨膜螺旋，都可能在结合后发生微小但关键的位移。这些变化会进一步影响 CD3 胞外段的位置、跨膜螺旋束的排列，甚至让 CD3 胞质尾区从膜内侧松开，使 ITAM 更容易被 Lck 接触。

这个假说的价值在于，它把“能不能结合”推进到了“以什么姿势结合”。两个 TCR-pMHC pair 可能有相似的亲和力，但如果 docking geometry、接触残基、受力路径和构象变化不同，最终信号能力可能完全不一样。

### 6. 受体聚集与微集群假说（Clustering / Microclusters）

单个 TCR 的信号很弱，真正的激活往往依赖许多 TCR 在膜表面形成局部聚集。**TCR microclusters（微集群）** 可以看作早期信号的小型指挥所。

当 TCR 接触 agonist pMHC 后，局部会快速聚集 TCR-CD3、ZAP-70、LAT、SLP-76 等信号分子。这些微集群通常在免疫突触成熟之前就已经出现，并且能在几秒到几十秒内启动早期信号。随后，它们会随着 actin retrograde flow 向细胞接触面的中心移动。

这个模型解释了为什么 T cell response 不只是单个 pair 的事情。pMHC 在膜上的分布、TCR 的表达量、局部扩散速度、细胞骨架流动、LFA-1/ICAM-1 黏附强度，都会影响微集群能否形成、维持和放大。

### 7. 免疫突触假说（Immunological Synapse）

如果说 microclusters 是早期的小火点，那么 **免疫突触（Immunological Synapse）** 就是 T 细胞和靶细胞之间形成的成熟作战界面。

当 T 细胞与 APC 或靶细胞稳定接触后，接触面上的分子会重新排布，形成类似靶盘的结构，常被分成几个区域：

* **cSMAC（central SMAC）**：中心区域，富集 TCR-CD3 以及部分信号和受体回收相关分子。
* **pSMAC（peripheral SMAC）**：外围区域，富集 LFA-1/ICAM-1 等黏附分子，像一圈密封圈一样稳定两个细胞的接触。
* **dSMAC（distal SMAC）**：更外侧区域，常见较大的调节分子和动态边界结构。

早期人们认为免疫突触是激活的核心起点。后来发现，很多早期信号在 microclusters 阶段就已经完成，成熟免疫突触更像是一个整合平台：它维持长时间信号、稳定细胞接触、组织受体回收，并在 CD8 T 细胞杀伤时帮助穿孔素和颗粒酶定向释放到靶细胞上，减少误伤旁边的正常细胞。

所以，免疫突触不仅是“细胞粘在一起”，而是 T 细胞把识别、信号、黏附和杀伤方向性组织到同一个界面上的结果。

真实的 T cell activation 大概率不是某一个假说单独完成的，而是这些机制在不同时间和空间尺度上的接力：TCR 先在膜表面扫描 pMHC；合适的 ligand 在力学和动力学窗口里停留足够久；局部 CD45 被排斥，Lck 磷酸化 ITAM；TCR microclusters 形成并放大信号；最后免疫突触成熟，把信号、黏附和杀伤输出组织成一个稳定界面。

因此，TCR-pMHC pair 预测如果只问“能不能 binding”，往往是不够的。更接近生物学的问题应该是：这个 pair 在特定 HLA、抗原密度、细胞状态、共刺激环境和实验 readout 下，能不能跨过 T cell activation threshold。

从静态 Docking 到动态分布：既然 Kinetic Proofreading（动力学校对）和 Catch Bond（逆境键）假说都强调了结合寿命（Dwell time）和构象变化，这意味着只考虑传统的静态结构（如 AlphaFold 直接预测出的单一构象）是不够的。模型需要捕捉多肽在结合槽中的柔性（Flexibility），以及 TCR 结合后引起的构象空间变化，这样才能在给定有限的数据下优化出更好的模型。

## 数据
### [VDJdb](https://vdjdb.com/)
目前各种机器学习模型使用最广泛的数据集是VDJdb，其中的数据来源主要分为以下三大流派：
1. pMHC 多聚体分选（Multimer Sorting）：数据的绝对主力这是 VDJdb 中占比最高的数据来源。由于单个 TCR 与单个 pMHC 之间的亲和力非常弱（解离很快），实验中无法直接用单个 pMHC 去“钓”出特异性 T 细胞。实验原理与流程：科学家将多个相同的 pMHC 复合体组装在同一个骨架上，最常见的是四聚体（Tetramer）或右旋糖酐多聚体（Dextramer，可包含几十个 pMHC），并带上荧光标签。将这些多聚体与外周血单核细胞（PBMC）混合。由于多聚体能同时结合 T 细胞表面的多个 TCR，产生了高强度的“亲和力（Avidity）”，从而将复合物稳定固定在细胞表面。通过流式细胞术（FACS），将带有特定荧光的 T 细胞分选出来。最后对这些细胞进行 TCR 测序（早年是 Bulk 测序，现在多为单细胞测序）。数据特性：这种数据代表的是纯粹的物理结合（Binding）。正如你之前笔记中提到的，结合了并不等于能激活。这类数据假阳性较高，且在 Bulk 测序时代，往往只能测到 TCR $\beta$ 链（因为 $\alpha$ 和 $\beta$ 在裂解细胞时配对信息丢失了）。

2. 单细胞测序与抗原条形码（scTCR-seq + Feature Barcoding）：高通量的新星随着 10x Genomics 等单细胞技术的普及，近几年的 VDJdb 新增数据大量来源于此。实验原理与流程：实验依然依赖 pMHC 多聚体（通常是 Dextramer），但这次不仅带荧光标签，还带有一段特异性的 DNA 条形码（DNA Barcode）。不同的 pMHC 对应不同的 DNA 序列。将细胞与多种不同的 pMHC 混合孵育后，直接打入单细胞液滴中。在同一个液滴内，同时测出该细胞的 TCR $\alpha$ 链、TCR $\beta$ 链以及它所结合的 pMHC DNA 条形码。数据特性：这是目前获取 双链（$\alpha\beta$ 配对） 数据的最高效方式。但因为系统非常灵敏，非特异性的粘附也会产生微弱的 DNA 信号，因此数据中经常混杂着背景噪声，需要严苛的生物信息学过滤。

3. 功能活化实验（Functional Activation Assays）：最符合生理意义的少数派这是在 VDJdb 中占比相对较少，但质量极高、最接近真实“激活”的一类数据。实验原理与流程：不再依赖人工合成的 pMHC 多聚体，而是将真实的 T 细胞与抗原呈递细胞（APC）在培养皿中混合，并加入目标多肽进行共培养。如果 T 细胞的 TCR 识别了 pMHC 并完成了完整的下游信号传导（经历了动力学校对、微集群形成等），T 细胞就会被真实激活。研究人员通过检测细胞表面的活化标志物（如 CD69、CD137/4-1BB 上调），或者通过检测细胞因子分泌（胞内细胞因子染色 ICS，或 ELISPOT 实验），把真正发生活化的 T 细胞分选出来，再进行测序。数据特性：这代表了真实的细胞激活（Activation），经历了严格的生理学串联验证，假阳性极低。但实验通量低，成本高，难以大规模应用。

### [McPAS-TCR](https://friedmanlab.weizmann.ac.il/McPAS-TCR/)

McPAS-TCR 的定位不是严格的“亲和力数据库”，而是 **pathology-associated TCR catalog**。它把文献中和感染、自免、肿瘤、过敏等病理状态相关的 TCR 序列整理在一起，因此它的标签语义比 VDJdb 更杂。

它背后测到的现象大致分为两层：

1. **抗原/表位相关的 TCR**：如果记录中同时给出了 peptide、MHC restriction、antigen source 等信息，这类数据通常来自上面已经说过的 pMHC multimer 分选或功能活化实验。这里不再重复实验流程，只强调标签含义：它测到的是“某个 TCR 在特定实验体系下和某个 epitope 有关联”，可能是结合阳性，也可能是激活阳性，具体要看原始记录的 assay type。

2. **病理状态相关的 TCR**：很多记录并不是直接证明了 TCR 识别某个 pMHC，而是在疾病样本、肿瘤浸润淋巴细胞、感染组织、外周血扩增克隆中观察到某些 TCR clonotype 富集。这里测到的是 **clonal expansion / disease association**，也就是“这个 TCR 在某种病理环境里出现或扩增”，不等于它一定识别了目标抗原，更不等于能激活 T cell。

所以 McPAS-TCR 更适合作为疾病相关 TCR 的线索库。如果用于训练 TCR-pMHC pair 模型，必须筛选带有明确 epitope、HLA/MHC 和实验类型的子集，否则很容易把 bystander T cell、炎症背景扩增和真正的 cognate TCR 混在一起。

### [PIRD](https://db.cngb.org/pird/)

PIRD 的核心是 **population-scale immune repertoire data**。它主要测到的是免疫组库中 TCR/BCR clonotype 的组成和丰度，而不是 TCR-pMHC 亲和力，也不是 T cell activation。

更具体地说，PIRD 里的原始数据通常来自 bulk TCR-seq/BCR-seq 或 single-cell immune repertoire sequencing。实验读出的主要是：

* CDR3 nucleotide / amino acid sequence
* V、D、J gene assignment
* clone count 和 clone frequency
* repertoire diversity
* 样本来源、疾病状态、组织类型、人群信息
* 在 single-cell 数据中，可能还有配对的 TCR $\alpha/\beta$ 或 BCR heavy/light chain

因此 PIRD 的标签更接近 **repertoire abundance / immune repertoire composition**。它回答的问题是“某个个体、组织或疾病状态下有哪些 TCR/BCR、丰度是多少”，而不是“这个 TCR 是否能识别某个 pMHC”。

如果 PIRD 中某些子库或关联表给出了抗原、疾病或表位信息，也需要回到原始文献看它是怎么来的：是 multimer 分选、功能刺激、疾病富集，还是单纯的 repertoire association。只有前两类才比较接近 TCR-pMHC specificity 或 T cell activation 标签。

### [IEDB](https://www.iedb.org/)

IEDB 是最容易被误用的数据源之一，因为它不是单一任务数据库，而是把 epitope 相关的多种实验结果都放在一起。使用 IEDB 时，关键不是看它叫不叫 positive，而是看 **assay type**。

它里面常见的测量现象包括：

1. **MHC-peptide binding**：测的是 peptide 和 MHC/HLA 的结合能力，例如 IC50、KD、stability score 等。这里的亲和力通常是 **peptide-MHC 亲和力**，不是 TCR-pMHC 亲和力。它能说明肽能不能被 HLA 装载和呈递，但不能直接说明 TCR 会不会识别。

2. **MHC ligand elution / mass spectrometry**：从细胞表面的 HLA/MHC 复合物中洗脱天然呈递的肽，再用 LC-MS/MS 鉴定。这里测到的是 **天然呈递 presentation**，也就是“这个 peptide 确实出现在 MHC 上”。它不测 TCR binding，也不测 T cell activation。

3. **T cell assay**：肽刺激 T cell、PBMC、T cell clone 或 T cell line 后，检测 IFN-$\gamma$、IL-2、TNF、ELISpot、intracellular cytokine staining、proliferation、cytotoxicity、CD69、CD137/4-1BB、CD154 等 readout。这里测到的是 **T cell activation / functional response**，是最接近本文目标的标签。

4. **MHC multimer / tetramer staining**：测的是 T cell 表面的 TCR 是否能被 pMHC 多聚体捕获。这里更接近 **pMHC binding / antigen-specific enrichment**，不是单分子亲和力，也不保证下游激活。

5. **B cell / antibody assay**：测的是抗体或 BCR 对 epitope 的识别，与 TCR-pMHC 建模不是同一个问题。

因此 IEDB 必须按 assay type 拆开使用：做 antigen presentation 可以用 ligand elution 和 MHC binding；做 TCR-pMHC recognition 可以用 multimer 或 T cell assay；做 T cell activation prediction 则应该优先使用 T cell functional assay。

### [ImmRep23](https://github.com/justin-barton/IMMREP23)

ImmRep23 是 receptor-antigen prediction 的 benchmark。它的 positive 样本主要来自 pMHC multimer/dextramer 阳性 T cell 的 single-cell TCR 数据；实验原理和 VDJdb 的前两类相同，这里不重复展开。需要注意的是，它测到的主要现象是 **pMHC binding / antigen-specific enrichment**，不是 KD，也不是细胞激活。

ImmRep23 中不同来源实验的共同点是：先用带荧光或 barcode 的 pMHC multimer/dextramer 捕获候选 antigen-specific T cell，再用 Smart-seq2、10x Genomics、BD Rhapsody 或 ImmunoScape 流程读出配对的 TCR $\alpha/\beta$ 链。它的优势是 paired-chain 信息比较完整，适合训练 TCR-pMHC recognition 模型。

但它的 negative 需要格外小心：很多 negative 是通过 TCR-peptide swap 构造出来的，也就是把某个 TCR 和别的 peptide 组合成 presumed negative。这类 negative 不是逐一湿实验测出来的 non-binder/non-activator，因此更适合作为 benchmark 的对照样本，而不是严格的生物学阴性标签。

### [BATCAVE](https://github.com/meyer-lab-cshl/BATMAN)

BATCAVE 是这里最接近 **T cell activation / cross-reactivity** 的 benchmark。它不是简单收集“某个 TCR 见过某个抗原”，而是围绕已知 index peptide 做单氨基酸突变扫描：把 peptide 的某个位置换成其他氨基酸，再观察同一个 TCR 对这些 mutant peptide 的反应强弱。

它测到的核心现象是功能输出，例如：

* IFN-$\gamma$ secretion
* NFAT-GFP reporter signal
* 其他 T cell activation reporter 或 cytokine readout

所以 BATCAVE 的标签通常更接近连续型的 activation strength，而不是单纯的 binding yes/no。它特别适合研究“一个 TCR 对 peptide 突变有多敏感”，也就是 cross-reactivity 和突变逃逸问题。

需要注意的是，BATCAVE 里也可能整合少量 TCR-pMHC affinity 数据，但这不是它最主要的标签来源。它的主轴仍然是 mutant peptide 引起的 T cell activation response。

### [TCR3d](https://tcr3d.ibbr.umd.edu/) / [STCRDab](https://opig.stats.ox.ac.uk/webapps/stcrdab-stcrpred) / [histo.fyi](https://www.histo.fyi/)

这三个资源都属于结构资源，不能和 VDJdb、IEDB T cell assay、BATCAVE 这类功能标签混为一谈。

**TCR3d** 和 **STCRDab** 收集的是 PDB 中已经解析的 TCR、pMHC、TCR-pMHC 或 TCR-CD3 结构。底层实验主要是 X-ray crystallography，近年也包括部分 cryo-EM。它们测到的是：

* 三维原子坐标
* TCR-pMHC docking angle
* CDR loop conformation
* interface buried surface area
* TCR 与 peptide/MHC 的接触残基
* apo/holo 构象差异

这些信息回答的是“这个复合物长什么样、怎么接触、构象如何变化”，而不是“这个 T cell 是否被激活”。少数条目会带 affinity metadata，例如 SPR、ITC 或 BLI 测得的 KD、kon、koff，但那是额外整合的物理化学测量，不是结构库本身的主标签。

**histo.fyi** 更偏向 peptide-MHC 结构资源。它整理的是 MHC class I 及其结合 peptide 的结构，包括 apo pMHC、receptor-bound pMHC 等。它测到的是 peptide 在 MHC groove 中如何摆放、哪些位置暴露给 TCR、不同 HLA allele 的槽结构差异。它不直接测 TCR specificity，也不测 T cell activation。

### [Observed TCR Space（OTS）](https://opig.stats.ox.ac.uk/webapps/ots)

OTS 是 paired-chain repertoire background resource。它收集 public single-cell TCR repertoire 中真实观察到的 TCR $\alpha/\beta$ 配对。这里测到的是 **某个 TCR pair 在真实人群或样本中出现过**，而不是它识别哪个 pMHC。

OTS 的核心读出包括：

* paired TRA/TRB CDR3
* V/J gene usage
* clonotype occurrence
* study、sample、donor 等 metadata

所以 OTS 适合做 background TCR distribution、negative sampling、语言模型预训练或 repertoire coherence analysis。它不应该被直接当作 antigen-specific positive 数据。

### [Adaptive Biotechnologies immuneACCESS](https://www.adaptivebiotech.com/)

immuneACCESS 是 Adaptive Biotechnologies 的公开 immune repertoire 数据入口。它的主体是 immunoSEQ 输出，测到的是 TCR/BCR clonotype 的丰度和频率，尤其常见的是 unpaired TCR $\beta$ chain。

常见读出包括：

* TCR $\beta$ CDR3 sequence
* V/D/J gene call
* clone count
* clone frequency
* 样本、疾病、时间点、组织来源等 metadata

因此，大部分 immuneACCESS 数据和 PIRD 类似，属于 **repertoire abundance**，不是 TCR-pMHC binding 或 activation。

不过 immuneACCESS 中有一类重要子集是 MIRA 或类似 antigen mapping 数据。它通过 peptide pool 刺激、富集或分选 antigen-associated T cell，再对富集前后 TCR $\beta$ clonotype 做 immunoSEQ，推断某些 TCR $\beta$ 和 antigen 的关联。这里测到的是 **antigen enrichment / antigen association**，仍然通常不是 paired $\alpha\beta$，也不是直接 KD 或完整 T cell activation strength。


读完上文的朋友应该已经很清楚了，这些数据实际上对应的是不同的生物学过程，在训练模型时是绝对不能混用的，如何合理的应用这些数据集仍是亟待解决的问题。

当然，T 细胞的激活绝非仅仅取决于 TCR 与 pMHC 的匹配模式（第一信号）。在真实的生理与病理环境中，这更像是一场多维度的动态博弈：T 细胞还必须接收到如 CD28 介导的共刺激信号（第二信号）才能避免失能，而肿瘤细胞往往通过表达 PD-L1 等免疫检查点强行‘踩刹车’。此外，局部的细胞因子信号（第三信号）、恶劣的肿瘤微环境（TME）（如缺氧、高酸性、代谢竞争与物理基质屏障），以及 T 细胞自身的表观耗竭状态和靶细胞表面的抗原真实丰度，都会直接决定激活的成败。因此，TCR-pMHC 的完美匹配只是拿到了攻击的‘入场券’，真正扣动扳机还需跨越重重微环境与细胞状态的壁垒。希望在不远的将来，我们可以通过更巧妙的方法建模更完整的过程（虚拟细胞？），
