---
title: Multi-nucleotide variants
date: 2020-04-28
categories: ["Bioinformatics"]
tags: ["MNV", "Phasing", "Haplotype"]
---

之前写过一篇关于MNV的文章，但感觉写的逻辑混乱，故推翻重写一篇

MNV (multi-nucleotide variants) 即多个碱基发生变化的突变，是有两个甚至多个 SNVs / InDels 合并而成，至于为何不将这些 SNVs / InDels 分别对待，则由于它在临床上、生物学上具有特殊的意义，因此特意归为一类突变。

至今为止，关于MNV并没有一个准确的定义，虽然大家都明确同一单倍型上的突变需要被合并，但无人指出具体**多近**的突变应该被合并。

<!--more-->

# MNV, Phasing 与 Haplotype 的概念

MNV 与 Haplotype (单倍型) 和 phasing problem 密切相关，我们先来直观的感受一下什么是 MNV:

Fig 1a. 中显示了两个相邻突变在BAM中的reads真实分布，可见，MNV中的两个突变均发生于相同的reads之上，这种情况我们称之为顺势 (in-cis )，而处于不同reads之上即称为反式 (in trans).  处于顺势的相邻突变即是 MNV

Fig 1b 则清晰的显示出了这两种情况（MNV 和 非MNV）会导致两种不同的突变类型，它的生物学意义不言而喻

![mnv_examples](https://www.biorxiv.org/content/biorxiv/early/2019/03/10/573378.1/F1.large.jpg)

在刚刚一图之中，突变和未突变的reads很明显的区分成了两份，这两份的区别，便是单倍型这一概念

单倍型这一概念，是对现有测序方法在**空间层面**上的一种补充。现有的测序方法，如二代、三代测序，虽然在碱基层面上达到了极高的精密度，但是却丢失了基因组在细胞内的空间分布信息，因为DNA经过打断这一过程，已无法追溯它的起源。而单倍型的概念，便是对于这一缺陷的补充，即试图分辨出，基因序列究竟来源于哪里。对于人类胚系细胞来说，当然这个来源只有两种可能：父系、母系；而在肿瘤的研究当中，由于**肿瘤异质性**以及 **非整倍体** 的存在，并非所有细胞的基因组都完全一致，因此不同单倍型可能来自于不同的亚克隆之中。

而如何判断碱基序列来源于同一个单倍型，这一过程就被称之为 phasing，因此可以说，phasing问题是整个单倍型的发现、MNV的判断的基础，我将会新写一篇博文来专门整理 phasing 目前的主流方法

# MNV 的“分类”

很多研究中，都会根据 MNV 的影响、产生的方式对 MNV 进行一个分类

根据 MNV 导致的氨基酸的改变，会将 missense MNVs 分为:

- one-step missense MNVs

  即一个碱基发生改变就能变成的某种氨基酸

- two-step missense MNVs

  需要两个碱基改变才能变成的某种氨基酸

- *exclusive missense SNV

  罕见，为只能由一个碱基改变才能变成的某种氨基酸

根据 MNV 的产生方式，MNV 又可被分为：

- sim-MNV (simultaneous)

  由单一突变过程导致的MNV。这种 MNV 的各单一突变往往具有完全相同的突变频率

- con-MNV (consecutive)

  由两种或以上突变过程导致的MNV，并非同一时间产生，而是之后 phasing 的时候被合并到一起

![mnv_sim_con](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6633265/bin/1047f01.jpg)

# MNV的发生机理

## 1. DNA polymerase zeta

最早被发现可以导致MNV的便是 DNA polymerase zeta，这是一种 translesion polymerase，与变异特征 GC -> AA 和 GA -> TT 紧密相关

> 在现实生活中，细胞必然会不断地收到损伤，其中部分损伤会一直维持在DNA之中，并且严重影响细胞的正常复制
>
> translesion polymerase，或者叫 translesion DNA synthesis (TLS) 便是细胞用来修复这一类损伤的重要武器，以防止危害被进一步扩大。但可想而知，这种快速修复必然是 error-prone ，并且会让未来 mutagenesis 和 carcinogenesis 的可能性进一步加大

在 de novo sim-MNV (1bp) 中，这种特征占到了约20%

## 2. APOBEC 

APOBEC 是属于 cytidine deaminase 的一个家族，主要功能是进行 C -> U 的 RNA 编辑，但也会在暴露的单链DNA上造成 clustered mutations。目前在COSMIC中，跟APOBEC 家族相关的突变特征共有三种：SBS13, SBS2 以及 DBS11。

在胚系突变当中，APOBEC signature 展现出 C..C -> T..T 的突变特征，并且两个突变碱基之间的距离甚远。这一特征在 sim-MNVs 和 con-MNVs (3-20bp) 的所有特征中占比最高

## 3. CpG island

CpG island 对 con-MNV 起到了最大的影响。在人类细胞中，CpG 岛中的 5'C 通常会被甲基化，并且突变率比正常序列高10倍。在 con-MNVs (2-30bp) 中，最常见的突变是：C..C -> T..T，这是由两处CpG岛中的 CG..CG -> TG.. TG 导致的。即使在 con-MNVs (1bp) 中，CG -> TG 占比也高达24%

## 4. Polymerase slippage

又被称为 slipped strand mispairing (SSM) 或者 replication slippage，是发生在 DNA 复制过程中的一种突变过程。在DNA复制的过程中，DNA polymerase 有时会从 DNA 单链上脱落下来，而新生成的 DNA 单链也会从模板链上脱落，并且再次比对上去。 而如果此时正好发生在重复区域，新生成的 DNA 单链便有可能比对错误（毕竟重复区域内，序列非常相似），DNA polymerase 再次回归开始复制时，便可能导致新生链的延长或者缩短

> ![](https://ars.els-cdn.com/content/image/1-s2.0-S1876162310790027-f02-06-9780123812780.jpg)

Wang et al., 2019 为研究 polymerase slippage 对 MNV 产生的影响，统计了重复区域内的 MNV ，发现重复区域中的 MNV 的频率远高于两个SNV同时发生的频率。并且，30%以上的MNVs 均为 AT -> TA 的突变；50% 的 AA -> TT 和 AA -> CC 的突变都发生在重复区域之中

但由于目前无法从重复区域中准确的算出 indel 的质量，该研究并没有计算indel在重复区域的表现，该研究也表明了，未来可通过 long-reads sequencing 的数据来解决这一问题

# MNV 的影响

## 1. 分子层面

Wong 早在1975 年即指出密码子的结果并非随机，单碱基的改变即使导致了氨基酸的改变，改变的氨基酸的生化特性也是相近的。而 MNV 则会能导致同一密码子中多个碱基的改变，由此可见由此改变的氨基酸在生化特性上也会跟原氨基酸相差较远。

Kaplanis et al., 2019 的研究中，定义了一个基于237种生化特质的**氨基酸距离**，在分析了 MNV 所导致的 missense variants 之后，确定了 two-step missense MNVs 在氨基酸距离上会导致更大差异。（two-step missense MNVs 这里定义为，必须由两个碱基改变才会导致的氨基酸）

为了证明更大的生化特性差异会造成更严重的影响，该研究研究了 constraint gene 中各类突变的数量。若MNV missense variants 确实有着更严重的影响，那么在 constraint gene 中应会发现更少的 missense MNVs.  研究结果也证明了我们的猜想，在高度保守基因当中 (pLI > 0.9)，无论是密码子间还是密码子内的 missense MNVs 数量都显著小于 missense SNVs 。见下图

> pLI -- the probability of loss-of-function intolerance score
>
> Samocha et al., 2014

该研究继而分析了所有 missense MNVs / SNVs 的 CADD值，发现 missense MNVs 的 CADD值显著高于 missense SNVs 。见下图

> CADD -- combined annotation-dependent depletion score
>
> 是一个用于预测 missense variants 的有害性的指标
>
> Kircher et al. 2014

分析这些missense MNVs / SNVs 的singleton占比，也得到了同样的结果 。见下图

> singleton 为在某一人群中，仅发生一次的突变；
>
> doubleton 则为发生两次的突变
>
> singleton的比值，能很好的反映出人群中的  purifying selection 的强度，也就意味着越会造成严重后果的突变，singleton 占比越大
>
> Lek et al., 2016

![missense_mnvs](https://genome.cshlp.org/content/29/7/1047/F3.large.jpg)



Wang et al., 2019 研究了 MNV 在 gnomAD 2.1 中的影响，如下图：

![mnv_impact](https://www.biorxiv.org/content/biorxiv/early/2019/03/10/573378.1/F2.large.jpg?width=800&height=600&carousel=1)

## 2. 临床层面

Kaplanis et al., 2019 的研究为研究 MNV 的临床意义，从DDDS (Deciphering Developmental Disorder Study) 研究中的已知致病基因中，找出了10个 de novo MNVs。与预算出的正常 MNV 突变率相比，这些基因中的 MNV 的突变率高了 3.6 倍 （de novo SNV 的频率也同样增高），即使去除了 polymerase zeta (sequence context bias) 的影响后，MNV 的发生率依然较高于正常值

同年的 Wang et al., 2019 的实验也研究了4275份罕见病的病人样本，虽发现了16 个 gained nonsense mutations 以及 110 个 missense MNVs ( 高 CADD，且在gnomAD 中低频 )，但经过人工审核，并未找到一个确定的 causal MNV 



---

参考文献：

Kaplanis J, Akawi N, Gallone G, et al. Exome-wide assessment of the functional impact and pathogenicity of multinucleotide mutations[J]. Genome research, 2019, 29(7): 1047-1056.

Wang Q, Pierce-Hoffman E, Cummings B B, et al. Landscape of multi-nucleotide variants in 125,748 human exomes and 15,708 genomes[J]. BioRxiv, 2019: 573378.

Wang Z, Yang J J, Huang J, et al. Lung adenocarcinoma harboring EGFR T790M and in trans C797S responds to combination therapy of first-and third-generation EGFR TKIs and shifts allelic configuration at resistance[J]. Journal of Thoracic Oncology, 2017, 12(11): 1723-1727.