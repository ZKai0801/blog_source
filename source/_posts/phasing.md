---
title: Phasing
date: 2020-05-18
categories: ['Bioinformatics']
tags: ['Phasing']
---

这一篇拖了挺久，一来是五一的时候出去玩了一趟，就偷了会懒，结果五一回来一直忙于工作，就又拖了一周，等我缓过神来才惊觉已经过去了半个多月，因此赶忙过来补上一篇。

上文说到，phasing 的根本目的，是将测序所得的碱基序列，重新正确的划分至他们本身的起源。而这一方法，在绝大数语境之中，指的便是将序列归为父亲或者母亲的染色体之中。目前大多数 phasing 的方法都是基于这一目的展开的，在肿瘤之中并不适用，由于肿瘤异质性的存在，测得的碱基序列可被归到不同的亚克隆之中，这使得传统 phasing 方法无法被使用。

<!--more-->

抛开实验方法不谈，纯生信的 phasing 方法主要分为三种：family-based 通过家系分型；population-based 通过族群分型；以及 read-based 从测序序列进行分型。下面让我一一道来

# 1. Family-based phasing

家系分型，可谓是所有 phasing 方法中最为精准的一种，通过对一家三口的基因组进行测序及比较，便能准确的将被测者（子女）的两个单倍体区分开来，如下图：

![family_phasing](https://pic4.zhimg.com/80/v2-6218beff25542ed442876b05d812b4fb_1440w.jpg)

比如 父亲为AA，母亲为BB，则子女必为AB，其中 A 必定来自父亲，而 B 必定来自于母亲。当然这种方法并没办法完全区分所有的位点。如图中倒数第四个位点，当父母子女均为 AB 杂合子时，则无法区分

# 2. Population-based phasing

从理论上来看，通过减数分裂中发生的基因重组，后代个体的染色体将会被完全的重组一遍，当然这一过程需要持续足够长的时间。而人类诞生至今所经历的时间远远达不到染色体的完全重组，因此便有了 linkage disequilibrium (LD) 的概念，即在人群之中，相邻的多个位点会出现共同遗传的特征，这些共同遗传的位点，就被称为 LD block。这种现象的产生，便是因为基因重组在历史上并未在这些位点之间产生。由此我们可以推断，一个更古老的族群，会比年轻的族群中包含更短的 LD block。事实也确实如此，非洲人群的 LD block 相对亚洲人、欧洲人确实更短。

通过人群中的这一特征，我们便可以利用统计学方法建立起一套群体特异的 reference map，然后便能从我们测序所得的序列中找出最有可能的单倍型排列方式。当然，可想而知的是，这一方法最大的局限性在于它无法实现个体特性性突变以及那些罕见突变，只能对人群中的常见突变做到区分。

# 3. Read-based phasing

Read-based phasing 是一种仅依赖于测序所得序列的 phasing 方法，与以上两种 phasing 方式不同，它不需要借助其他人的测序结果，仅需要本身样本进行比对、甚至组装，来判断突变是否存在于同一单倍型上。显而易见的是，这种方法最受测序长度的影响，二代测序由于读长的限制，只能检测到 150bp 以内的单倍型；当然，以此代价换来的是可以精确的发现 de novo 的单倍型，这让其在肿瘤方面的应用变得尤为突出。

支持read-based phasing的软件很多，如MAC，但这里我们重点谈一下 GATK 里面对单倍型的处理。GATK 流程的第一步是从 BAM 文件中进行常规的 call 突变，然后使用 ReadBackPhasing tool 从得到的 vcf 文件中重新考虑所有的 reads 以找出最有可能的单倍型

这一工具在现在版本的 gatk 中默认是自动开启的，然而需要注意的是，这一算法是针对 germline 研发的。而在肿瘤当中，不同的亚克隆相当不同的单倍型，这些过多的单倍型会使得能检测的信号变弱，从而无法检测到单倍型

虽然 GATK 团队认为他们的算法有很大的提升空间，但与其他方法相比， GATK的算法已经能达到非常高的灵敏度和特异性了

![mnv_gatk](https://raw.githubusercontent.com/ZKai0801/bioitland.io/master/imgs/mnv_gatk.png)

在 `mutect2` 当中，单倍型也同样会被检测出来，可通过 `--max-mnp-distance` 参数来指定最长的合并距离，默认下会将1bp以内的自动合并。但需要注意的是，GATK只会合并PASS的突变，也就是它们认为是真阳性的突变，如果你拥有一套自己的真假突变过滤逻辑，就需要自行合并

合并的逻辑并不难，我们仅需要找出GATK已经为我们判断好的单倍型即可，在VCF文件中，这个信息被收录在 PGT 和 PID 之中，下文则是 VCF header中对其的描述：

> PGT: "Physical phasing haplotype information, describing how the alternate alleles are phased in relation to one another"
> PID: "Physical phasing ID information, where each unique ID within a given sample (but not across samples) connects records within a phasing group"

简单的来说，就是PGT表示的是突变之间的关系（顺势还是反式），共有两种表示方式：0|1，1|0 

由于目前单倍型主要考虑的是胚系，因此仅有父系和母系之间的区别，0|1，1|0则是分别表示来自于父母之中的某一方，当然具体是父方还是，母方这个未知

而 PID 则是GATK赋予同一单倍型的编号，故在同一单倍型中的突变拥有相同PID。

理解了这个，合并起来就容易多了，具体的合并逻辑可参考我github仓库中的 [merge_mnv.py](https://github.com/ZKai0801/tertiary_analysis/blob/master/merge_mnv.py) ，这个脚本需要你提前将假阳性的突变从vcf中去掉，再将剩下的mnp合并起来，距离默认为5bp内







