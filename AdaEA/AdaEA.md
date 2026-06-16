---
tags:
  - papers/adversarial-attacks
aliases:
  - AdaEA
  - 自适应模型集成对抗攻击
date: 2023-10
arxiv_id: "2308.02897"
---

# An Adaptive Model Ensemble Adversarial Attack for Boosting Adversarial Transferability

## 核心信息

- 标题: An Adaptive Model Ensemble Adversarial Attack for Boosting Adversarial Transferability
- 标题翻译: 一种用于提升对抗迁移性的自适应模型集成对抗攻击
- 作者: Bin Chen, Jiali Yin, Shukai Chen, Bohao Chen, Ximeng Liu
- 机构: 福州大学 (Fujian, China); 元智大学 (Taipei, Taiwan)
- 发表时间: 2023 年 10 月
- 发表渠道: ICCV 2023 (pp. 4489–4498)
- arXiv: 2308.02897
- 论文链接: https://openaccess.thecvf.com/content/ICCV2023/html/Chen_An_Adaptive_Model_Ensemble_Adversarial_Attack_for_Boosting_Adversarial_Transferability_ICCV_2023_paper.html
- 代码 / 项目: https://github.com/CHENBIN99/AdaEA
- 论文类型: 方法 (method)

## 原文摘要翻译

对抗样本的迁移性使得攻击者能够在黑盒场景下（即攻击者对目标模型一无所知）实施攻击，因此基于迁移的对抗攻击受到广泛关注。已有工作大多通过梯度变化或图像变换来放大输入中关键区域的扰动。这些方法可以在差异有限的模型间实现迁移（如从 CNN 到 CNN），但面对架构差异较大的模型间迁移（如从 CNN 到 ViT）时往往失效。另一条路线是模型集成对抗攻击：通过融合多个架构多样的代理模型输出获得集成损失，使得生成的对抗样本能同时欺骗多个模型，从而提高迁移到其他模型的可能性。然而，现有集成攻击仅简单地等权融合各代理模型输出，未能有效捕获和放大对抗样本的内在迁移信息。本文提出一种自适应集成攻击方法 ==AdaEA，通过监测各模型对对抗目标的贡献差异比来自适应控制输出融合==。此外，==引入一个额外的差异削减滤波器以进一步同步梯度更新方向==。在多个数据集上，AdaEA 相较于现有集成攻击取得了显著提升，并且能够增强现有的基于迁移的攻击方法，进一步证明了其有效性和通用性。

## 创新点

1. **对抗比率 (adversarial ratio)**：定义了代理模型间的对抗贡献差异度量——每个模型生成的对抗样本对其他模型的攻击成功率。这是将集成权重分配与迁移性直接建立因果联系的关键，而不是凭经验或启发式地设定权重。

2. **自适应梯度调制 (AGM)**：根据对抗比率自适应地为各代理模型分配集成权重，使得含有更多可迁移对抗信息的模型得到更大的梯度贡献。相比 SVRE 的方差缩减思路，AGM 是主动的信息放大策略。

3. **差异削减滤波器 (DRF)**：通过通道级余弦相似度构建梯度差异图，并以二值化掩码过滤掉梯度差异过大的分量。这在机制层面直接缓解了 CNN 与 ViT 梯度方向冲突导致的过拟合问题，是对集成优化过程的显式正则化。

4. **正交的集成包装器设计**：AdaEA 不是一种新的独立攻击算法，而是一个可以无缝嵌入到任何基于迭代梯度的攻击方法（I-FGSM、MI-FGSM、DI²-FGSM 等）中的集成包装器。这种设计使其具有很高的实用复用价值。

## 一句话总结

AdaEA 通过自适应梯度调制 (AGM) 和差异削减滤波器 (DRF) 两个模块，将模型集成对抗攻击从简单的等权平均提升为基于迁移贡献的自适应融合，在 CNN↔ViT 跨架构迁移场景下取得了相比 Ens 和 SVRE 的大幅提升，并且可以作为通用包装器增强现有迁移攻击。

## 研究背景与问题

### 对抗迁移性的核心瓶颈

对抗攻击的白盒场景（攻击者知晓目标模型的全部参数和梯度）已经相当成熟，但黑盒场景（对目标模型一无所知）才是实际威胁建模的核心。利用对抗样本的迁移性——即针对代理模型 (surrogate model) 生成的对抗样本也能欺骗目标模型——是黑盒攻击的主要路线。

然而迁移成功率高度依赖代理模型与目标模型之间的架构相似性。当目标模型与代理模型同属 CNN 家族时，现有的迁移增强策略（如 MI-FGSM、DI²-FGSM、PI-FGSM、NAA 等）可以取得不错的效果。但当目标模型切换到 ViT 家族时，这些方法几乎完全失效——如 Fig. 1(a) 所示，CNN→ViT 的平均攻击成功率仅约 2%。
![[1.jpg]]

*Fig. 1: Overview of different attack schemes and performance. (a) Transfer-based methods fail across CNNs→ViTs. (b) Equal-weight ensemble attack. (c) AdaEA adaptively ensembles outputs.*

### 模型集成攻击的现状与不足

模型集成攻击的基本思路是同时攻击多个架构多样的代理模型，希望生成的对抗样本能捕获跨架构的通用对抗模式。其数学形式为：

$$\arg\max_{x^{adv}} \sum_{k=1}^{K} w_k L(f_k(x^{adv}), y)$$

其中 $w_k$ 是第 $k$ 个模型的权重，$L$ 是损失函数（通常为交叉熵），$f_k$ 是代理模型的输出。

已有工作的关键局限：

- **Ens (Liu et al., 2017)**：直接等权平均所有模型的 logits 或 losses，$w_k = 1/K$。
- **SVRE (Xiong et al., 2022)**：注意到不同代理模型的梯度方差问题，通过随机方差缩减来稳定集成梯度。

但这些方法的共同问题是：它们将每个代理模型视为同等重要。而从图 2 的梯度余弦相似度热力图可以直观看出，CNN 内部相似度高、ViT 内部相似度高、但跨 CNN-ViT 的梯度相似度很低（甚至接近于 0 或负）。==在这种情况下等权平均会使得一个模型捕获的可迁移信息被另一个模型"平滑"掉，导致次优的集成结果==。
![[2.jpg]]
*Fig. 2: Cosine similarity between gradients produced from different models. Gradients are closer when model architectures are more similar.*

### AdaEA 的核心动机

AdaEA 的核心洞察是：不是所有代理模型对迁移性的贡献都相同。应该根据每个模型生成的对抗样本在其他模型上的攻击表现来自适应调整其在集成中的权重，同时显式抑制梯度方向冲突的模型分量。这与传统等权集成思路有本质区别——==问题从"如何减少集成方差"变成了"如何放大可迁移信息并抑制冲突"==。

## 方法主线

### 机制流程

AdaEA 的整体流程可分为四个阶段：

![[3.jpg]]
*Fig. 3: An overview of AdaEA. The gradients obtained from CNNs and ViTs are fed into AGM and DRF to get the ensemble gradient for generating adversarial examples with gradient-based attack.*

**步骤 1：获取各代理模型梯度。** 给定当前对抗样本 $x^{adv}_t$，对 $K$ 个代理模型分别计算交叉熵损失关于输入的梯度 $g_i = \nabla_{x^{adv}_t} L(f_i(x^{adv}_t), y)$。

**步骤 2：自适应梯度调制 (AGM)。** 计算各模型的对抗比率 $\rho_i$ 和自适应权重 $w_i^*$，输出加权后的梯度集合。

**步骤 3：差异削减滤波 (DRF)。** 计算各模型梯度间的通道级余弦相似度，构建差异图 $d$，通过二值掩码 $B$ 过滤不一致的梯度分量。

**步骤 4：生成集成梯度并更新对抗样本。** 将 AGM 加权后的梯度与 DRF 掩码进行逐元素乘法得到最终集成梯度，以此更新 $x^{adv}_{t+1}$。

### 自适应梯度调制 (AGM)

AGM 的核心是定义**对抗比率 (adversarial ratio)**——衡量代理模型 $f_i$ 生成的对抗样本对其他模型 $f_k$ 的攻击效果。
先计算 sk,i​  可看作第 k 个模型在使用第 i 个模型梯度生成的对抗样本上的损失

$$s_{k,i} = -\mathbf{1}_y \cdot \log\left(\text{softmax}\left(p_k[x^{adv}_t + \alpha \cdot \text{sign}(g_i)]\right)\right)$$

其中 $p_k(\cdot)$ 表示模型 $f_k$ 的 logits 输出，$\alpha$ 为攻击步长，$1_y$ is the ground truth logits。这个式子的含义是：用模型 $f_i$ 的梯度方向生成一个临时的单步对抗样本，然后测试它在模型 $f_k$ 上的交叉熵损失。对抗比率 $\rho_i$ 定义为：

$$\rho_i = \frac{\beta}{K-1} \sum_{\substack{k=1, \\ k \neq i}}^{K} \frac{s_{k,i}}{s_{k,k}}$$

具体来说，对于每一个其他模型 $k$，计算从 $i$ 迁移到 $k$ 的攻击效果与 $k$ 自攻击效果的比值，然后对所有 $k≠i$ 取平均（除以 $K−1$），最后乘以 $β$。$ρi​$ 越大，说明从模型 $i$ 迁移到 $k$ 的攻击效果相对越强，即 $i$ 模型提供了更多通用的对抗信息。最后通过$softmax$ 归一化得到自适应权重：

$$\displaystyle w_1^*, w_2^*, \dots, w_K^* = \mathrm{softmax}(\rho_1, \rho_2, \dots, \rho_K)$$

其中 $\beta$ 控制权重分布的超参数。

或
$$\displaystyle \rho_i = \frac{\sum_{k \neq i} s_{k,i}}{\sum_{j=1}^{K} \sum_{k \neq j} s_{k,j}}$$
分子：所有目标模型 $k（k≠i）$被模型 $i$ 生成的对抗样本攻击的损失之和。即模型 $i$ 作为源的总迁移攻击效果;
分母：所有可能的**跨模型攻击**（源 $j$ ≠ 目标 $k$）的损失总和。即全局迁移攻击总效果
$ρi​$是模型 $i$ 在**绝对迁移效果**中的份额。份额越大，说明模型 $i$ 的对抗样本最能广泛地欺骗其他模型。
然后通过指数尺度变换得到权重:
$$\displaystyle w_i = \frac{\exp(\beta \cdot \rho_i)}{\sum_{j=1}^{K} \exp(\beta \cdot \rho_j)}$$

### 差异削减滤波器 (DRF)

DRF 解决的是另一个问题：即使加权后的梯度仍然可能存在方向冲突。DRF 首先计算任意两个代理模型 $f_i$ 和 $f_k$ 在空间位置 $(p,q)$ 处的梯度余弦相似度：

$$d_i(p,q) = \frac{1}{K-1} \sum_{k \neq i} \frac{g_i(p,q) \cdot g_k(p,q)}{\|g_i(p,q)\| \cdot \|g_k(p,q)\|}$$

即模型 $f_i$ 在位置 $(p,q)$ 的梯度与所有其他模型同位置梯度的平均余弦相似度。对所有 $i$ 取平均得到最终的差异图 $d(p,q)$:
$$\displaystyle d(p, q) = \frac{1}{K} \sum_{i=1}^{K} d_i(p, q)$$

然后通过二值化阈值 $\eta$ 构建滤波器：

$$B(p,q) = \begin{cases} 0, & \text{if } d(p,q) \leq \eta \\ 1, & \text{otherwise} \end{cases}$$

余弦相似度低于 $\eta$ 的位置被视为"差异过大"的分量，直接置零。最终的集成梯度为：

$$g_{t+1} = \nabla_{x^{adv}_t} L\left(\sum_{k=1}^{K} w_k^* f_k(x^{adv}_t), y\right) \odot B$$

其中 $\odot$ 表示逐元素乘法，$\eta = -0.3$ 为默认阈值。注意这里 AGM 的输出权重 $w_k^*$ 已经融入集成损失的加权聚合中，DRF 的掩码再对梯度做空间维度的同步。
![[adaea.jpg]]
### 与已有方法的关键差异

SVRE 的方差缩减操作在**随机采样**的代理模型子集上估计梯度，试图降低不同子集间的方差；AdaEA 的 DRF 则在**所有代理模型**的梯度上做空间维度的显式同步。前者是统计层面的去偏，后者是几何层面的方向对齐。两者并不冲突，但 AdaEA 的 DRF 对 CNN↔ViT 这种架构差异特别大的场景效果更显著（也确实是 DRF 在 ViT 目标上带来了最大的提升）。

## 实验设计

### 数据集与模型

实验覆盖三个数据集：CIFAR-10、CIFAR-100 和 ImageNet。

**目标模型（黑盒，共 8 个）：**
- CNN 分支：ResNet-50、WideResNet-101-2、BiT-M-R50×1、BiT-M-R101
- ViT 分支：ViT-Base、DeiT-Base、Swin-Base、Swin-Small

**代理模型（白盒，默认 4 个）：** ResNet-18、Inception v3、ViT-Tiny、DeiT-Tiny。这个选择刻意涵盖 CNN 和 ViT 两个架构分支，且使用小模型攻击大模型。

### 攻击设置与超参数

- 基础攻击：I-FGSM，20 次迭代
- 扰动约束：$l_\infty$，$\epsilon = 8/255$，步长 $\alpha = 2/255$
- AGM 权重尺度：$\beta = 10$
- DRF 阈值：$\eta = -0.3$
- SVRE 内部更新次数：4（默认设置）

### 基线方法

两个直接可比的集成攻击基线：Ens (Liu et al., 2017) 和 SVRE (Xiong et al., CVPR 2022)。所有方法使用完全相同的代理模型设置。还测试了 AdaEA 包装在不同基础攻击（FGSM、I-FGSM、MI-FGSM、DI²-FGSM）上的效果。

## 核心实验结果

### 主实验：黑盒攻击成功率

下表浓缩了 Table 1 的核心结果——AdaEA 在三个数据集上对所有 8 个目标模型的平均攻击成功率：

| 数据集 | Ens | SVRE | AdaEA | Δ over Ens |
|--------|-----|------|-------|------------|
| CIFAR-10 | 26.56 | 29.38 | **45.09** | +18.53 |
| CIFAR-100 | 63.76 | 66.23 | **71.10** | +7.34 |
| ImageNet | 46.38 | 46.39 | **49.36** | +2.98 |

AdaEA 在 CIFAR-10 上的提升（+18.53%）远大于 CIFAR-100（+7.34%）和 ImageNet（+2.98%），部分原因是 CIFAR-10 的基线成功率较低、提升空间更大，同时也因为小分辨率的 CIFAR 图像上梯度差异对最终攻击效果的影响更显著。值得注意的是 SVRE 在 ImageNet 上增量几乎为零（+0.01%），这表明方差缩减思路在更复杂的任务上可能遇到了瓶颈，而 AdaEA 仍然保持了约 3 个点的增益。

![[4.jpg]]
*Table 1: Black-box attack success rate (%) against eight naturally trained models. Bolded numbers indicate best results.*

### AdaEA 作为攻击包装器的通用性

Table 2 的结果证明了 AdaEA 的通用性——将其包装在四种不同的基础攻击方法上，均带来一致的提升（CIFAR-10 上的平均 ASR）：

| 基础攻击 | Ens | SVRE | AdaEA | Δ(AdaEA vs Ens) |
|----------|-----|------|-------|------------------|
| FGSM | 12.77 | 21.31 | **36.89** | +24.12 |
| I-FGSM | 28.00 | 30.06 | **46.27** | +18.27 |
| MI-FGSM | 37.07 | 23.04 | **54.81** | +17.74 |
| DI²-FGSM | 72.44 | 33.49 | **80.32** | +7.88 |

有两个值得注意的异常现象：SVRE 在 MI-FGSM 和 DI²-FGSM 下相比 Ens 反而出现了大幅退化（分别为 -14.03% 和 -38.95%）。这可能是因为 SVRE 的随机方差缩减与这些攻击已有的动量或输入变换机制产生了冲突，导致梯度估计不稳定。相反，AdaEA 的 AGM 和 DRF 是确定性的自适应调权与方向同步，与这些增强策略天然兼容。

![[5.jpg]]
*Table 2: Attack success rate (%) of adversarial examples generated by ensemble attacks based on different attack methods on CIFAR-10.*

### 攻击高级防御模型

Table 3 报告了对三组对抗训练模型 (Inc-v3ens3, Inc-v3ens4, Inc-v2ens) 和六种输入变换防御 (R&P, Bit-Red, JPEG, ComDefend, RS, NRP) 的攻击结果。以 DI²-FGSM 为基础攻击时：

- 对抗训练防御：AdaEA 的平均 Robust Accuracy 仅剩 2.42%，而 Ens 为 1.66%、SVRE 为 0.96%（数值越低攻击越成功——这里 AdaEA 比 Ens 多降低了约 0.76 个百分点，考虑到对抗训练模型的基线精度本来就大幅压缩，这是一种有意义的改善）。
- 输入变换防御：AdaEA 的平均 ASR 为 65.37%，相比 Ens 的 60.44% 提升了约 5 个点。这表明 AdaEA 生成的对抗样本对输入预处理类防御具有更强的鲁棒性。

![[6.jpg]]
*Table 3: Robust accuracy (%) against adversarial training models and advanced defense methods on CIFAR-10.*

### 组件消融：AGM 与 DRF 各自的贡献

Table 4 清晰展示了两个组件的互补性（默认 4 个代理模型，CIFAR-10 场景）：

| 方法配置 | CNN 目标平均 | ViT 目标平均 | 全部目标平均 |
|----------|-------------|-------------|-------------|
| 基础集成 (Ens) | 29.96 | 23.94 | 27.29 |
| + AGM | 39.48 | 40.18 | 39.79 (+12.50) |
| + DRF | 38.31 | 47.57 | 42.42 (+15.13) |
| AdaEA (AGM+DRF) | 40.85 | 49.69 | 44.78 (+17.49) |

AGM 在 CNN 和 ViT 上的提升相对均衡（+9.52 / +16.24），说明自适应权重策略本身是通用的。DRF 在 ViT 上的提升尤为突出（+23.63），这与 DRF 的设计动机一致——CNN 与 ViT 之间的梯度差异远大于 CNN 内部或 ViT 内部的差异，显式过滤这些差异分量可以大幅改善跨架构迁移。

![[7.jpg]]
*Table 4: Experimental results of average attack success rate (%) on the component ablations in AdaEA.*

### 代理模型组合的影响

Table 5 系统考察了代理模型数量、CNN/ViT 比例等因素对迁移性的影响。主要发现：

- **更多代理模型 → 更高迁移性**：从 2 个代理模型增加到 4 个，全部目标的平均 ASR 提升约 20 个百分点。
- **CNN 比例高 → CNN 目标高、ViT 目标低**：当代理模型中 CNN 占主导时（3CNNs:1ViT），CNN 目标的平均 ASR 达到 63.82%、ViT 目标仅 30.56%。反之为 1CNN:3ViTs 时，ViT 目标高达 72.83%、CNN 目标为 49.66%。
- **仅用同架构代理模型攻击异架构目标几乎无效**：3CNNs 攻击 ViTs 仅 13.58%、3ViTs 攻击 CNNs 为 36.53%。

AdaEA 在所有组合下均优于对应的 Ens 基线，表明其自适应机制对不同代理模型构成具有鲁棒性。

![[8.jpg]]
*Table 5: Comparison of average attack success rate (%) between ensemble attack and AdaEA under different ensemble models on CIFAR-10.*

### 超参数敏感性

Fig. 5 展示了两个关键超参数的敏感性分析（CIFAR-10 场景）：
- $\beta$（AGM 的权重锐度）：在 0 到 20 范围内，平均 ASR 先升后降，$\beta=10$ 附近取得最优。过小的 $\beta$ 退化为近似等权，过大的 $\beta$ 使权重过于集中在单一模型上。
- $\eta$（DRF 的差异过滤阈值）：在 -0.8 到 0.4 之间变化，All models 的 ASR 在 $\eta = -0.3$ 附近达到峰值。当 $\eta$ 过小（接近 -0.8）时几乎不进行过滤，当 $\eta$ 过大（接近 0.4）时过滤了过多梯度分量，性能反而下降。

![[10.jpg]]
*Fig. 5: Ablation study on (a) weighting scale β in AGM and (b) binarization threshold η in DRF.*

### 热力图定性分析

Fig. 4 的可视化热力图对比了干净图像、Ens、SVRE 和 AdaEA 生成的对抗样本在代理模型 (Res-18, ViT-T) 和黑盒模型 (WRN50-2, Swin-T) 上的注意力分布。AdaEA 生成的对抗样本在跨架构模型上的关键区域激活更集中、更一致，从直观层面支持了 AGM+DRF 可以更好地捕获跨模型共享的对抗语义这一论断。

![[9.jpg]]
*Fig. 4: Heatmaps of different inputs in the surrogate models and black-box models. (a) Clean image and adversarial examples. (b)-(e) Heatmaps on surrogate models (Res-18, ViT-T) and black-box models (WRN50-2, Swin-T).*

## 讨论与局限

### 论文证明了什么

这篇论文比较扎实地证明了三个核心命题：第一，等权集成在代理模型架构多样性较强时确实是次优的——不仅有直觉的论证（梯度相似度热力图），还有量化的消融实验作为支撑。第二，AGM 和 DRF 两个模块各自有效且互补：AGM 解决"谁贡献更多"的问题，DRF 解决"方向不一致怎么办"的问题。第三，AdaEA 是一个通用的集成包装器，可以与多种现有攻击方法无缝组合。

### 论文没有证明什么

1. **对抗比率是否是最优的迁移性代理指标**：AGM 用每个模型"攻击其他模型的效果"作为权重依据，这有循环论证的嫌疑——本质上是用"在当前代理模型集合上的表现"来近似"对未知目标模型的迁移性"。论文没有对比其他可能的权重分配策略（如基于模型不确定性、基于梯度范数等）。

2. **DRF 阈值的普适性**：$\eta = -0.3$ 的选择仅在 CIFAR-10 上做了敏感性分析。在不同数据集、不同代理模型组合下该阈值是否需要重新调整，论文没有讨论。

3. **对抗训练代理模型下的表现**：所有代理模型均使用标准训练权重。如果代理模型本身经过对抗训练，AGM 和 DRF 的行为会如何变化是未知的。

4. **非分类任务的迁移性**：实验局限于图像分类。目标检测、语义分割等下游任务上的对抗迁移性未涉及。

### 值得关注的局限

- **计算开销**：AGM 需要 $O(K^2)$ 次额外的前向传播来计算对抗比率矩阵（每步迭代对每对 $(i,k)$ 都要走一次 $p_k[x + \alpha \cdot \text{sign}(g_i)]$）。当代理模型数量 $K$ 较大时这会成为训练/攻击效率的瓶颈。论文没有报告 AdaEA 相比 Ens 的实际时间开销倍数。

- **ImageNet 上的增益缩小**：从 CIFAR-10 的 +18.53% 到 ImageNet 的 +2.98%，AdaEA 的优势在大规模数据集上明显收窄。一种解释是 ImageNet 本身的高分辨率和大类别数使得基础攻击已经能捕获足够的可迁移信息，留给 AGM+DRF 改善的空间变小；另一种解释是默认的超参数在 ImageNet 上不是最优的。

- **SVRE 在特定攻击下的退化**：SVRE 在 MI-FGSM 和 DI²-FGSM 下的严重退化（-14.03% 和 -38.95%）虽然反衬了 AdaEA 的鲁棒性，但也暗示集成攻击的行为对基础攻击的特性非常敏感。论文没有深入分析这种敏感性的根源。

## 总结与启发

### 方法论层面的启发

AdaEA 最值得借鉴的设计思路是：将集成加权问题从一个"每个模型都是一样的，我们需要平均它们"的被动融合视角，转变为一个"每个模型携带不同质量的迁移信息，我们需要主动筛选和放大"的主动调控视角。这种思路可能适用于其他涉及多模型协同的场景，如多教师知识蒸馏、联邦学习中的模型聚合、多模态融合等。

### 工程复用指南

- 如果要在自己的工作中使用 AdaEA：按照论文的默认设置（$\beta=10, \eta=-0.3$），将 AGM 和 DRF 包裹在现有的 I-FGSM 迭代循环中即可。
- 代理模型选择建议：确保 CNN 和 ViT 各占一定比例，完全同架构的代理模型集合会严重限制对异架构目标的迁移性。
- 基础攻击的选择：DI²-FGSM + AdaEA 的组合在实验中取得了最高的绝对 ASR (80.32% on CIFAR-10, 65.37% against defenses)，是当前最优的迁移攻击组合之一。

### 后续研究可以关注的方向

1. 如何降低 AGM 的 $O(K^2)$ 计算复杂度？是否可以用代理指标近似对抗比率？
2. DRF 的阈值 $\eta$ 是否可以自适应学习，而非手工设定？
3. AdaEA 在目标攻击 (targeted attack) 和物理世界攻击场景下的表现如何？
4. AdaEA 的权重分布是否可解释？能否从中挖掘出哪些模型架构对迁移性贡献最大的结构性规律？

## 参考文献与后续阅读

核心被引文献：

- **Ensemble attack baseline**: Liu et al., "Delving into Transferable Adversarial Examples and Black-box Attacks", ICLR 2017 — 首次提出模型集成对抗攻击
- **SVRE baseline**: Xiong et al., "Stochastic Variance Reduced Ensemble Adversarial Attack for Boosting the Adversarial Transferability", CVPR 2022 — 引入方差缩减的集成攻击
- **I-FGSM**: Kurakin et al., "Adversarial Examples in the Physical World", ICLR 2017
- **MI-FGSM**: Dong et al., "Boosting Adversarial Attacks with Momentum", CVPR 2018
- **DI²-FGSM**: Xie et al., "Improving Transferability of Adversarial Examples with Input Diversity", CVPR 2019
- **ViT**: Dosovitskiy et al., "An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale", ICLR 2021
- **DeiT**: Touvron et al., "Training Data-Efficient Image Transformers & Distillation through Attention", ICML 2021
- **Swin Transformer**: Liu et al., "Swin Transformer: Hierarchical Vision Transformer using Shifted Windows", ICCV 2021
- **Adversarial training**: Madry et al., "Towards Deep Learning Models Resistant to Adversarial Attacks", ICLR 2018

后续可跟踪的扩展工作方向：目标攻击中的自适应集成、基于 AdaEA 的对抗训练增强、跨模态（图像→文本/VLM）迁移中的梯度同步策略。

---

*笔记生成日期：2026-06-12*
                                                                                                      