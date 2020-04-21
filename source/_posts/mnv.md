---
title: Multi-nucleotide variants
date: 2020-04-18
categories: ["Bioinformatics"]
tags: ["MNV", "Phasing"]
---

MNV (multi-nucleotide variants) 即多个碱基发生变化的突变，此问题跟 phasing 紧密相关，然而目前为止，绝大数关于 phasing 和 MNV 的研究都是基于人类胚系展开的，在肿瘤方面，仍鲜有文章发表。

<!--more-->

# MNV

MNV名称跟 SNV 类似，只不过把 single 变成了 multiple，顾名思义，便是多个碱基发生变化的突变。说到这里，稍微会思考的同学可能就会奇怪，为什么需要把这种突变单独归为一类，而不是分开成多个 snvs / indels 呢？确实，一般的 snp calling 软件如 GATK 都会将这种突变视为一个个分开的突变。我们之所以将其归为一类突变，是因为它在临床上、生物学上具有特殊的意义。

MNV的定义是：在同一个单倍型 （haplotype）上，同时发生的两个甚至多个突变。对于什么是单倍型，维基百科上描述了对它的四种不同定义：

1. 从单个父母中遗传下来的一组等位基因
2. 在染色体之中的一组紧密相连的等位基因，保守性高且在多代中遗传下来
3. 多个同时发生的SNPs，这一理论被用来进行 genotype imputation
4. （用于很多基因检测公司）表示在一个特定基因区间的所有突变的综合

说实话这些个描述都非常片面，只是从四个角度对单倍型的功能等进行了描述，如果是初学者，相信看完之后还是很难理解。个人认为，单倍型这一概念，是对现有测序方法在空间层面上的一种补充。现有的测序方法，如二代、三代测序，虽然在碱基层面上达到了极高的精密度，但是却丢失了基因组在细胞内的空间分布信息，因为DNA经过打断这一过程，已无法追溯它的起源。而单倍型的概念，便是对于这一缺陷的补充，即试图分辨出，基因序列究竟来源于哪里。对于人类胚系细胞来说，当然这个来源只有两种可能：父系、母系；在进化学当中，我们也可以通过这个来研究共同祖先等问题。而在肿瘤的研究当中，由于肿瘤异质性的存在，并非所有细胞的基因组都完全一致，因此不同单倍型可能来自于不同的亚克隆之中。

Fig 1a. 中显示了两个相邻突变在BAM中的reads真实分布，可见，MNV中的两个突变均发生于相同的reads之上，这种情况我们称之为顺势 (in-cis )，而处于不同reads之上即称为反式 (in trans). 

Fig 1b 则清晰的显示出了这两种情况会导致两种不同的突变类型，它的生物学意义不言而喻

![mnv_examples](https://www.biorxiv.org/content/biorxiv/early/2019/03/10/573378.1/F1.large.jpg)

MNV在临床上的意义则更为重要，如 EGFR 的 T790M 和 C797S 突变，若两者位于反式，则对 EGFR-TKI 敏感，反之若两者位于顺势，则对其耐药 (Wang et al., 2017)

# MNV判定方法

如何确定突变是否在同一个单倍型上，这一过程就被称之为 phasing 

目前 phasing 的方式分为四种：

1. read-based phasing
2. family-based phasing
3. population-based phasing
4. laboratory-based phasing

read-based phasing 又被称为 read backed method ，是通过判断相邻突变是否共同发生在同样的 reads 上面。显而易见的是，这种方式虽更容易发现 de novo 的单倍型，但非常受限于 reads 的长度，而常见的二代测序平均长度仅为 150bp 左右，这导致大于这个长度的突变距离很难被归为同一个单倍型

family-based phasing 则是判断相同对的突变是否能在家系中同时遗传下来

population-based phasing 也被称为 statistical phasing，是通过已知序列的人群进行统计学上的推论的 phasing 方式  

这里我们重点谈一下 GATK 里面对单倍型的处理。它流程的第一步是从 BAM 文件中进行常规的 call 突变，然后使用 ReadBackPhasing tool 从得到的 vcf 文件中重新考虑所有的 reads 以找出最有可能的单倍型

这一工具在现在版本的 gatk 中默认是自动开启的，然而需要注意的是，这一算法是针对 germline 研发的，而二倍体的胚系当中，仅存在父系母系这两种可能。而在肿瘤当中，不同的亚克隆可以相当不同的单倍型，这些过多的单倍型会使得能检测的信号变弱，从而无法检测到单倍型

虽然 GATK 团队认为他们的算法有很大的提升空间，但与其他方法相比， GATK的算法已经能达到非常高的灵敏度和特异性了（见 Fig 1c）

![mnv_gatk](https://raw.githubusercontent.com/ZKai0801/bioitland.io/master/images/mnv_gatk.png)

在 `mutect2` 当中，单倍型也同样会被检测出来，可通过 `--max-mnp-distance` 参数来指定最长的合并距离，默认下会将1bp以内的自动合并。但需要注意的是，GATK只会合并PASS的突变，也就是它们认为是真阳性的突变，如果你拥有一套自己的真假突变过滤逻辑，就需要自行合并

合并的逻辑并不难，我们仅需要找出GATK已经为我们判断好的单倍型即可，在VCF文件中，这个信息被收录在 PGT 和 PID 之中，下文则是 VCF header中对其的描述：

> PGT: "Physical phasing haplotype information, describing how the alternate alleles are phased in relation to one another"
> PID: "Physical phasing ID information, where each unique ID within a given sample (but not across samples) connects records within a phasing group"

简单的来说，就是PGT表示的是突变之间的关系（顺势还是反式），共有两种表示方式：0|1，1|0 

由于目前单倍型主要考虑的是胚系，因此仅有父系和母系之间的区别，0|1，1|0则是分别表示来自于父母之中的某一方，当然具体是父方还是，母方这个未知

而 PID 则是GATK赋予同一单倍型的编号，故在同一单倍型中的突变拥有相同PID。

理解了这个，合并起来就容易多了，具体的合并逻辑可参考我github仓库中的 [merge_mnv.py](https://github.com/ZKai0801/tertiary_analysis/blob/master/merge_mnv.py) ，这个脚本需要你提前将假阳性的突变从vcf中去掉，再将剩下的mnp合并起来，距离默认为5bp内



------

参考文献：

1. Wang, Zhen, et al. “Lung Adenocarcinoma Harboring EGFR T790M and In Trans C797S Responds to Combination Therapy of First- and Third-Generation EGFR TKIs and Shifts Allelic Configuration at Resistance.” *Journal of Thoracic Oncology*, vol. 12, no. 11, 2017, pp. 1723–1727., doi:10.1016/j.jtho.2017.06.017.

   https://www.ncbi.nlm.nih.gov/pubmed/28662863

2. Choi, Y., Chan, A., Kirkness, E., Telenti, A. and Schork, N., 2018. Comparison of phasing strategies for whole human genomes. *PLOS Genetics*, 14(4), p.e1007308.

3. Kaplanis, J., Akawi, N., Gallone, G., McRae, J., Prigmore, E., Wright, C., Fitzpatrick, D., Firth, H., Barrett, J. and Hurles, M., 2019. Exome-wide assessment of the functional impact and pathogenicity of multinucleotide mutations. *Genome Research*, 29(7), pp.1047-1056.

4. Landscape of multi-nucleotide variants in 125,748 human exomes and 15,708 genomes 

   https://www.biorxiv.org/content/10.1101/573378v2.full

5. GATK Purpose and operation of Read-backed Phasing

   https://gatkforums.broadinstitute.org/gatk/discussion/45/purpose-and-operation-of-read-backed-phasing
