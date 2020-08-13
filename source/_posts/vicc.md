---
title: 癌症突变解读协会 -- VICC
date: 2020-05-28
categories: ['Cancer', 'Bioinformatics']
tags: ['Variant Classification']
---

今天要介绍的是Variant Interpretation for Cancer Consortium (VICC) 癌症突变解读协会，这一协会隶属 Global Alliance for Genomic and Health 组织之下，旨在促进肿瘤界中的突变的解读。虽然 AMP/ASCO/CAP 在2017年就发布了这方面的指南，但细读之，就能发现里面留下了很多待解决的问题。而 VICC 项目的发起，就是为了解决这些问题，因此 VICC 的原文非常值得一读，文献地址请见文末

<!--more-->

# 胚系与体系突变解读

实际上，这里说的胚系与体系突变解读，强调的更是是遗传病和癌症领域的突变解读，这是因为遗传病必然由胚系突变引起，而癌症中大数致癌突变都是后天产生的。遗传病方面的最权威突变解读指南是由 ACMG/AMP 于2015年发布，这一套体系可以说非常成熟；相反 AMP/ASCO/CAP 于2017年发表的肿瘤方面的突变解读，可实行性要弱的多。下面来说说这两者之间的两大大不同之处

## 1. 等级划分标准之差

由于遗传病往往是由单个或者少量突变引起，这使得遗传病产生的原因相对可循，ACMG/AMP 便可以引入“致病性”这一概念来研究这些突变对蛋白功能的影响。然而对于肿瘤来说，基因组已经发生了巨变，我们很难预测某一个突变对于肿瘤的影响，因此 AMP/ASCO/CAP 并没有提供一个系统的步骤去评定一个突变的 'oncogenicity' （致癌性），相反，它建立了一套基于**临床证据**的评断标准，并且，这些标准非常模糊，实践者很难用它去严格的建立自己的规则

## 2. 突变表达形式之差

对于胚系来说，突变的表达形式非常简单明了，就是单一的，在DNA层面上的表现，可以用基因组位置+突变碱基的方式进行描述，很少有复杂的情况。

然而在肿瘤之中，突变的表达形式真的是多种多样，最常见的便是用 HGVS 方式进行表述，例如：`NP_004295.2:p.F1174L (ALK F1174L)` 但这个突变可由`NC_000002.11:g.29443695G>T` 或者`NC_000002.11:g.29443695G>C` 造成；或者使用突变类型对突变进行表达，例如： BRCA1 中的 oncogenic mutation。这便对癌症中突变的注释造成了巨大的挑战。

# 肿瘤突变分类的挑战

由于肿瘤突变分类的基础是寻找尽可能多临床证据，这考验的实际上还是数据挖掘能力。

在胚系突变方面，基本上就是 ClinVar 、HGMD 一统天下的局面，并且数据库之间的表达差异并不大。

然而对于体系突变来说，尤其是肿瘤方面，数据库就完全是各自为营的状态，它们之间几乎**没有任何共同点**。下图是VICC对六大肿瘤数据库的条目进行比对的情况，可以看到的是，绝大数条目中的引用的文献都是独一的（占83%），能被六个数据库共同引用的仅由一例。

![](https://media.springernature.com/full/springer-static/esm/art%3A10.1038%2Fs41588-020-0603-8/MediaObjects/41588_2020_603_Fig6_ESM.jpg?as=webp)

这六大数据库分别是：

- [PMKB](https://pmkb.weill.cornell.edu/)
- [CGI](https://www.cancergenomeinterpreter.org/)
- [CIViC](https://civicdb.org/home)
- [OncoKB](https://www.oncokb.org/)
- [MMatch](https://www.molecularmatch.com/)
- [Jax-CKB](https://ckb.jax.org/)

但若仅仅是如此，我们只需要找到足够多的数据库即可，可实际情况并非如此，在找齐足够多的数据库之后，我们还面临着五大挑战：

1. 基因名的表述差异
2. 突变的表述差异
3. 癌症分型的差异
4. 药名的差异
5. 证据等级的划分差异

其中，第一条和第四条相对容易解决。

对于1) 来说，虽然 gene symbol 有所区分，但其中的对应关系都是已知的，统一到 HGNC gene symbols 就可以了

对于4) 来说，VICC 的做法是利用将所有药物在 Mychem.info、 PubChem、 ChEMBL 上进行检索，最后归一

证据等级的划分，这个虽然各大数据库都有自己的划分方式，但是根本原则相似，因此 VICC 在它们之间确立了一套映射关系，统一划分至 AMP/ASCO/CAP 标准之中，如下表：

| Evidence level    | Defining characteristics                                     | CIViC                                 | OncoKB             | JAX-CKB                         | CGI                                | MMatch   | PMKB   |
| :---------------- | :----------------------------------------------------------- | :------------------------------------ | :----------------- | :------------------------------ | :--------------------------------- | :------- | :----- |
| Level A (tier I)  | Evidence from professional guidelines or FDA-approved therapies relating to a biomarker and disease. | Level A                               | Level 1/2A /R1     | Guideline/FDA approved          | Clinical practice                  | Level 1A | Tier 1 |
| Level B (tier I)  | Evidence from clinical trials or other well-powered studies in clinical populations, with expert consensus. | Level B                               | Level 3A           | Phase III                       | Clinical trials III–IV             | Level 1B |        |
| Level C (tier II) | Evidence for therapeutic predictive markers from case studies, or other biomarkers from several small studies. Also, evidence for biomarker therapeutic predictions for established drugs for different indications. | Predictive level C                    | Level 2B, level 3B | Clinical study/phase I/phase II | Clinical trials I–II, case reports | Level 2C | Tier 2 |
| Level D (tier II) | Preclinical findings or case studies of prognostic or diagnostic biomarkers. Also includes indirect findings. | Nonpredictive level C/level D/level E | Level 4            | Phase 0, preclinical            | Preclinical data                   | Level 2D |        |



而最后癌症分型的区别，以及突变描述的区别，是所有差异里面最难以处理的。这里，重点说明一下VICC对于这两者的处理方式

## 肿瘤分型的匹配

先将所有的肿瘤分型在 EBI OLS 上检索一遍，找到其匹对的DOID，这能匹配到98.7%的分型，剩下无法匹配出的分型统一规划到 DOID: 162 (cancer) 这一大类中。

然后将能找到的分型归一到 TopNode 之中。这里归一的形式有所区分，一是精准匹配，就是匹配得到最接近的那一分型；还有一种是模糊匹配，便是将这一分型的上下级均匹配出来，但不匹配它的姊妹级分型。如下图 b，蓝色字体表示的是匹配出的分型，红色的字体便是无法匹配出的，也就是它的姊妹级

![](https://media.springernature.com/full/springer-static/esm/art%3A10.1038%2Fs41588-020-0603-8/MediaObjects/41588_2020_603_Fig8_ESM.jpg?as=webp)

## 突变的匹配

突变的归一步骤，在VICC中主要分为三步：

1. 将所有突变均转换为基因组位置的描述方式:

   chrom, start, end, ref, alt

   对于已有这些参数的突变来说，这个固然简单。然而由于大多数突变没有一个清晰的基因组位置，因此VICC 首先在 COSMIC 中检索了这些突变的名称，来获取上述参数。对于检索不出的突变，VICC 手动制定了各种规则，来解决这之中的冲突

2. 将突变转换至 HGVS 命名

3. 将这串 HGVS 命名的突变提交至 ClinGen Allele Registry 以获取唯一的突变编码

对于突变匹配来说，VICC 又建立了四套不同的规则，如上图中 a 所示，分别是：1) exact match 突变需要两者完全的匹配; 2) positional match 基因组位置必须一致，但碱基可以不同; 3) focal match 两者必须在存在10%以上的匹配; 4) regional match 只要突变有交集，就匹配

之所以建立起如此复杂的匹配模式，是因为数据库中很多突变的描述是基因某一密码子，某一外显子，甚至仅仅是某一基因。因此如果按照一般的匹配方式，即 exact match，那么这一部分突变将永远无法匹配上。

---

VICC 这一项研究最大功劳便是整合了各数据库之间的差异，最后还设计了一个交互式网站，可供大家免费使用：https://search.cancervariants.org/

不仅如此，这篇文献文末还提供了研究中所用的所有代码：http://git.io/vicckb 并且告诉了大家各大数据库的下载方式。

Variant Interpretation for Cancer Consortium (VICC) 

---

参考文献：

Wagner A H, Walsh B, Mayfield G, et al. A harmonized meta-knowledgebase of clinical interpretations of somatic genomic variants in cancer[J]. Nature genetics, 2020, 52(4): 448-457.

Li M M, Datto M, Duncavage E J, et al. Standards and guidelines for the interpretation and reporting of sequence variants in cancer: a joint consensus recommendation of the Association for Molecular Pathology, American Society of Clinical Oncology, and College of American Pathologists[J]. The Journal of molecular diagnostics, 2017, 19(1): 4-23.

Richards S, Aziz N, Bale S, et al. Standards and guidelines for the interpretation of sequence variants: a joint consensus recommendation of the American College of Medical Genetics and Genomics and the Association for Molecular Pathology[J]. Genetics in medicine, 2015, 17(5): 405-423.