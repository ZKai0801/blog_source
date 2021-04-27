---
title: 从AUC到DCA
date: 2021-04-25
categories: ['Statistics']
tags: ['DCA', 'ROC']
---

DCA全名 Decision Curve Analysis，即决策曲线分析法，在 2006 年由 MSKCC 的 Vickers 和 Elkin 首次提出 <sup>1, 2, 3</sup>，用于评估临床中的诊断模型、预后模型等。相较于在二战时期被发明的ROC曲线，DCA还非常年轻，但已在临床模型评估中展现出其独有的优势。因为此分析法并不重点关注于灵敏度及特异性，而是从患者的净收益情况、及其自身意愿的角度来评估模型的优劣。

在本文中，我将从二元分类问题入手，系统介绍ROC曲线以及DCA曲线

<!--more-->

# 二元分类问题

无论是ROC曲线还是DCA曲线，都是用于评估二元分类 (binary classification) 模型的一种分析方法。所谓二元分类问题，就是结局仅有两种情况的问题，在临床之中尤为常见。例如，诊断一位患者是否得了某种疾病（结局仅可能为：“有” 或者 “没有”）。

对于评估一个模型是否能准确的进行分类来说，常见有如下几个指标：

1. 灵敏度  TPR

   又被称为召回率，英文：Sensitivity、Recall Rate、True Positive Rate

   即 实际为阳性中检出阳性 的占比。

   假设我们在评估一种癌症的检测方法，则该指标评估的是 该检测方法对患者的检出能力水平。

   

2. 特异性 TNR

   英文 Specificity、True Negative Rate 

   即 实际为阴性中检出阴性 的占比。

   同样假设我们在评估一种癌症的检测方法，则该指标评估的是 该检测方法对健康人群的排除能力水平。

   

3. 精确度 PPV

   又称为阳性预测值，英文 Precision、Positive Predictive Value

   所指为在 所有检出为阳性之中真正为阳性 的占比

   同样假设下，该指标评估的是 所有检测为阳性的人群中，真正患癌的人数占比

   此值低，则意味着假阳性的人数多，则会对健康受试者造成不必要的恐慌，以及浪费之后的检测、治疗。

   

4. 阴性预测值 NPV

   英文 Negative Predictive Value

   所指为在 所有检测为阴性之中真正为阴性 的占比

   同样假设下，该指标评估的是 所有检测为阴性的人群中，真正健康人的人数占比

   此值低，则意味着假阴性的人数多，这些被误诊为健康的癌症患者则会被耽误治疗、甚至错过最佳治疗时机。

   

5. F1 score

   不知道中文名叫啥，又称为 F score 或 F measure

   其定义为 灵敏度 和 精确度 的调和平均数，公式如下：

   F1-score = 2 / ( (1/ TPR ) + (1 / PPV) )

   该值在 0 ~ 1 范围内浮动，1 则表示该值具有完美的灵敏度和精确度；0 则意味着 灵敏度 或 精确度 中一值为 0



上述五类指标在临床中最为常见，建议熟读并背诵 :) 

详情见下图：

![binary_classification](https://raw.githubusercontent.com/ZKai0801/bioitland.io/master/imgs/dca_bc.png)



# ROC曲线

ROC 全名 Receiver operating characteristics，之所以有这么个古怪的名字，是因为它是在二战时期被发明，一开始是用作评估雷达的信号接受能力。

这一分析方法非常简单，是通过对模型设置不同的阈值来计算出其对应的 TPR 以及 FPR。通过上图可知，TPR 便是灵敏度（即收益）；FPR 便是 1 - 特异性（即代价）。如此一来，每一个模型便都能计算出其对应的一条 ROC 曲线。如下图，曲线越接近图中的左上角，则意味着模型越准确；反之，如果曲线越接近图中的对角线，则意味着模型对结果的预测越随机。

如何更为直观的表达模型的好坏呢？答案便是使用 曲线下面积 -- AUC面积（Area Under Curve）。AUC 这一数值取值在 0.5 - 1 之间。0.5 所对应的便是ROC曲线与对角线重叠的情况，表明模型对结果的预测完全随机；AUC = 1，则是面积最大，意味着模型能完全精准的预测出结果的情况。

![roc_curve](https://glassboxmedicine.files.wordpress.com/2019/02/roc-curve-v2.png)

对于如何应用，这里不详细展开，网上有很多写的不错的博客可以参考，例如：https://rviews.rstudio.com/2019/03/01/some-r-packages-for-roc-curves/

唯一一点需要提一下，就是在生信类的文献中，最常见的模型是预后模型，ROC实际上并不适合评估此类模型。毕竟对于预后来说，因变量实际上是连续变量。因此如果想要用ROC来进行评估，必须先将其因变量转变为二元类变量。这便是我们经常看到的 三年生存、五年生存 等等，指的是模型用于判断，患者在三年后（确切的来说是指 刚过完三年的这个当下）是否生存的状态。这一方法被称为 time-dependent ROC curve analysis<sup>4</sup>，推荐使用 R 包 timeROC 完成



# DCA 曲线

上述传统的统计学评估方法（灵敏度、特异性、AUC等等），并不能很好的应用于临床。这是因为，在某些场景下，我们会希望使用灵敏度更高的模型，但有些情况下，我们更在意模型的特异性。虽然世界上不存在完美的模型，但我们可以尽可能的使患者的净收益最大化，这便是DCA分析的初衷。

自2006年 DCA分析首次提出之后，短短几年内，被引数已上千。并被多篇顶级医学杂志，如 JAMA、JCO 所推荐。然而真正了解其原因、懂得如何去解读它的人并不多（作者在后续文章中自己说的，不是我瞎写的）。



## DCA 的解读及原理

简而言之，DCA曲线所展示的是 **某一模型，相较于 “干预所有患者” 和 “对所有患者均不干预” 这两种策略 在不同的 Threshold probability 下的 Net Benefit "净收益”情况**。因此常规DCA曲线中，会出现三条线：1）干预所有患者的收益情况；2）完全不干预的收益情况；3）根据模型结果进行干预的收益情况。这三条线，每条均是一个**策略**。

DCA图中，Y轴为净收益，指的是患者可从该策略中获得的收益大小；X轴为 Threshold probability，指的是医生及患者对于”干预措施“的意愿（接受还是抵触）。

DCA曲线的解读方法，大致可分为两步：1）首先根据临床因素挑选一个合适的 threshold  probability，2）随后在该 Threshold 下 便可以选择对应着最高净收益的策略。下面我们通过一个例子来理解这个过程。

在前列腺癌筛查的研究中，若受检者被查出其前列腺特异性抗原（prostate-specific anigen PSA）升高的情况，往往会被要求去做活体检测以确认是否真实患有恶性前列腺癌。然而实际中，PSA升高的受检者中，仅有很小一部分为恶性肿瘤。那么对于那些为患病的健康人来说，他们去做活检，便是过度治疗，需承担不必要的伤害。在这一例子中， “干预手段” 指的是 “活检” ；”净收益“ 指的是 受检者中可被准确检出前列腺癌的比例；而”Threshold Probability“ 指的是患者及医生对于活检的接受度。

如下图所示，Intervention for all / Intervention for none 这两条线，分别表示对所有患者进行活检、以及对所有患者均不采取干预。Test 这一条虚线，指的是 PSA筛查 这一手段。这一筛查方法具有固定的灵敏度及特异性，故成直线，而非曲线。假设现在我们研发了一款全新的预测方法，可以更好的预测患者是否患有恶性肿瘤。那么这一预测方法，便是图中所画的 "Model"。从图中可以看的是，**Model 这一曲线，在绝大情况下均有最高的收益值**，也就是说按此模型结果来决定是否进行活检收益最高。除非患者、医生特别希望进行活检 -- Preference非常低 -- 只有此时 “全干预” 的策略收益高于模型。

![dca](https://media.springernature.com/lw685/springer-static/image/art%3A10.1186%2Fs41512-019-0064-7/MediaObjects/41512_2019_64_Fig1_HTML.png?as=webp)



理解了Benefit，接下来我们聊聊 Preference。即使在同样的病例面前，不同的患者、不同的医生，最后做出的决定可能都不一样。比如对于一个处于壮年的患者来说，尽可能的确诊才是他的诉求，如果能在可治愈时期发现癌症、并治愈癌症，那么他的收益才是最大的；但是对于一位体弱多病的老人来说，尽量避免活检、维持现在的生活才是他的诉求，因为活检会对患者产生一定的伤害，甚至导致感染。另一个重要的考量是价格。 如果干预手段价格太高，那么便需尽可能减少干预（在案例中便是指活检）。

具体到x轴的实际内容上，是由 Threshold probability 这一值来反映 Preference 的情况。所谓 Threshold probability，可理解为模型预测的患癌可能性。假设医生选择 10% 作为模型的阈值，则若患者使用该模型检测出 10% + 的值，则医生就会对其进行活检；反之则不会。以 10% 为阈值，还意味着医生认为 漏掉一个恶性肿瘤，比做一个不必要的活检要糟糕 9 倍。这个比例（1：9），也被形象的称之为 "exchange rate" 交换率。

Net benefit 净收益的定义为：

​	Net Benefit = Benefit - harm * exchange rate

在刚刚那个示例中，如果该模型在一个100人的队列中进行验证，这个队列中所有人均检测出了PSA水平升高，且无其他明显表型。其中，有 25 个是真正的癌症患者，剩余75个为健康人。那么对于 10% 的exchange rate 来说：

​	Net Benefit = 25% - 75% * (1 / 9) = 16.7%

同理，其他所有的点均可靠此方法计算出来。

![dca_preference](https://media.springernature.com/lw685/springer-static/image/art%3A10.1186%2Fs41512-019-0064-7/MediaObjects/41512_2019_64_Fig2_HTML.png?as=webp)



当然，DCA这种评估方式也带来了明显的问题，因为 threshold probability 的纯主观的，各个医生会有不同的评估。假设一个医生完全没有主观意见的话，那么DCA曲线变对他**毫无帮助**。只有在考虑到了临床中各种因素之后，才能将其转换为较为具体的 threshold probability，并根据此来选择模型、策略。



## DCA的实现方式

在理解了DCA的原理之后，可以发现实现起来并不复杂。但我们并不需要从头计算，很多R包都可以实现这DCA曲线，这里推荐使用Y叔叔的R包 -- `ggDCA` <sup>5</sup>。Y叔自己写了个详细的文章，这里就不再冗述了：[首发！决策曲线分析R包：ggDCA](https://mp.weixin.qq.com/s/dcN1BvmuSO7osWFPPq3pYg)





---

参考文献：

1. Vickers AJ, Elkin EB. Decision curve analysis: a novel method for evaluating prediction models. Med Decis Making. 2006;26(6):565–74
2. Vickers AJ, Van Calster B, Steyerberg EW. Net benefit approaches to the evaluation of prediction models, molecular markers, and diagnostic tests. BMJ. 2016;352:i6
3. Vickers AJ, van Calster B, Steyerberg EW. A simple, step-by-step guide to interpreting decision curve analysis. Diagn Progn Res. 2019 Oct 4;3:18. doi: 10.1186/s41512-019-0064-7. PMID: 31592444; PMCID: PMC6777022.
4. Kamarudin AN, Cox T, Kolamunnage-Dona R. Time-dependent ROC curve analysis in medical research: current methods and applications. BMC Med Res Methodol. 2017 Apr 7;17(1):53. doi: 10.1186/s12874-017-0332-6. PMID: 28388943; PMCID: PMC5384160.
5. https://cran.r-project.org/web/packages/ggDCA/index.html