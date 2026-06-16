---
tags:
  - papers/adversarial-attacks
aliases:
  - RaPA
  - Random Parameter Pruning Attack
  - 随机参数剪枝攻击
date: 2025-04-24
arxiv_id: "2504.18594"
---

# 通过随机参数剪枝增强可迁移定向攻击

## 核心信息

- 标题: RaPA: Enhancing Transferable Targeted Attacks via Random Parameter Pruning
- 标题翻译: RaPA: 通过随机参数剪枝增强可迁移定向攻击
- 作者: Tongrui Su (苏彤睿), Qingbin Li (李清彬), Shengyu Zhu (朱胜宇), Wei Chen (陈薇), Xueqi Cheng (程学旗)
- 机构: 中国科学院计算技术研究所, 智能算法安全全国重点实验室
- 发表时间: 2025-04-24 (v1), 2026-02-26 (v2), 2026-04-06 (v3, CVPR camera-ready)
- 发表渠道: CVPR 2026
- arXiv: [2504.18594](https://arxiv.org/abs/2504.18594)
- 论文链接: [CVPR 2026 Virtual Poster](https://cvpr.thecvf.com/virtual/2026/poster/36301)
- 代码 / 项目: [github.com/molarsu/RaPA](https://github.com/molarsu/RaPA)
- 论文类型: 方法论文 (对抗攻击, 迁移攻击, 计算机视觉)

## 原文摘要翻译

与非定向攻击相比,定向迁移攻击尽管已通过输入多样化、梯度稳定化和重新训练代理模型等多种手段取得了显著进展,但其攻击成功率仍然低得多。本文发现,==现有方法生成的对抗样本严重依赖代理模型中极少数的参数子集,这反过来限制了它们对不可见目标模型的迁移能力==。受此启发,我们提出了随机参数剪枝攻击 (Random Parameter Pruning Attack, RaPA),在攻击过程中引入参数级别的随机化。==在每个优化步骤中,RaPA 随机剪枝模型参数以生成多样化但在语义上保持一致的代理模型变体==。我们证明,==这种参数级随机化等价于添加一个重要性均衡正则化项,从而缓解了过度依赖问题==。跨 CNN 和 Transformer 架构的大规模实验表明,RaPA 显著增强了可迁移性。在从 CNN 向 Transformer 模型迁移这一最具挑战性的场景中,RaPA 的平均攻击成功率比现有最优基线高 11.7 个百分点 (基线为 33.3%),且完全免训练、跨架构高效并可轻易集成到现有攻击框架中。代码已开源。

## 创新点

- **首次揭示了"捷径参数依赖"这一迁移攻击的深层瓶颈**。通过 Optimal Brain Damage (OBD) 框架对代理模型参数重要性进行二阶量化,==实验证明仅剪掉 Top 0.5% 最重要的参数,攻击成功率就暴跌 46% 以上==——这意味着对抗样本在"走捷径",将攻击信号集中绑定到了极少数的模型参数上,一旦目标模型不具备这些捷径路径,攻击立即失效。
- **在攻击生成过程中引入了参数级的 DropConnect 式随机化**。此前 CFM 在特征空间做随机混合,FTM 在特征空间做可学习扰动,GRA 在梯度方向做邻域平滑——都是在"信号层面"处理迁移问题。RaPA 是第一个将随机化下推到模型参数本身的攻击方法:==每次迭代生成一个 Bernoulli-masked 子模型,对抗样本被迫在多个参数结构不同的模型变体上同时有效,从而无法绑定到任何特定参数。==--> **单一模型内部的参数子空间，就已经包含了足以泛化到其他架构的巨大多样性**。通过动态 DropConnect，在单模型内实现了高效的虚拟集成(self-ensemble)。
- **通过二阶泰勒展开证明随机参数剪枝等价于重要性均衡正则化**。引入 Gini 系数来量化参数重要性的不平等程度:基线情况下 Gini = 0.32,加入 RaPA 后骤降至 0.08,直观验证了"马太效应"被有效抹平。
- **确立了 BN 层和 FC 层是捷径依赖的重灾区**。通过逐层剪枝消融实验发现,==在 BN 和 FC 层上剪枝的收益远大于在 Conv 层上剪枝== (Conv 层权重本身较稀疏,剪枝反而可能破坏模型的基础语义一致性),为后续工作提供了清晰的操作指引。

## 一句话总结

RaPA 发现对抗样本对代理模型参数的"捷径依赖"是定向迁移攻击长期停滞在低成功率的根本原因,通过在每次攻击迭代中随机剪枝 BN/FC 层参数来生成语义一致的多样化子模型,迫使对抗样本将攻击信号均匀散布到全局参数中,从而在不增加任何训练开销的前提下将 CNN→Transformer 跨架构迁移成功率从 33.3% 提升至 45.0%（以RN50为源模型）。

## 问题定义

### 捷径参数依赖——迁移攻击的隐藏瓶颈

作者通过 OBD 框架对现有迁移攻击生成的对抗样本进行了一项关键诊断实验:

OBD 使用损失函数对参数的二阶导数 (Hessian 对角近似) 来衡量每个参数对当前对抗样本的重要性。
$$\mathcal{I}(\theta_i) = \frac{\partial^2 \mathcal{L}(f(x_{\text{adv}}))}{\partial \theta_i^2} \times \theta_i^2$$
This metric reflects how much the loss would change if a parameter θi were removed, and can serve as a proxy for its contribution to the effectiveness of the adversarial example.
根据该重要度指标修剪替代模型参数(ResNet50)的两个不同子集：最重要前0.5%和最不重要0.5%的最低0.5%参数,剪掉最不重要的参数 (bottom k%) 对攻击成功率几乎无影响,但剪掉最重要的 0.5% 参数后,ASR 暴跌超过 46%。
![[top-bottom.jpg]]

这一现象的根因链为:

1. 代理模型的 BN 层和 FC 层中存在少数"关键节点",它们对损失函数的局部曲率贡献最大。
2. 基于梯度的攻击方法天然倾向于利用这些曲率最大方向来快速降低损失,从而将对抗扰动的信号集中编码到这些捷径上。
3. 不同模型架构 (尤其是 CNN vs. Transformer) 的参数结构完全不同,这些捷径在目标模型上不存在对应物,导致对抗样本的语义失效。

RaPA 的目标: 在不增加训练开销、不修改攻击框架的前提下,打破这一捷径依赖,迫使对抗样本学习更"分散"、更具泛化性的攻击模式。
## 方法主线

### 基础框架

RaPA 是免训练的即插即用模块,嵌入在标准 MI-FGSM + Logit Loss 框架中。核心改动仅发生在每次迭代的前向传播阶段。
![[RaPA算法图.jpg]]
### 核心机制: 随机参数剪枝

在第 $t$ 次攻击迭代中,对代理模型 $F$ 的选定层参数施加 Bernoulli 掩码（伯努利分布）:
The corresponding masks for the weight matrix and bias are（5）；Then the masked parameters are computed as（6）
其中 $\mathcal{M}_w \in \{0, 1\}^{d_{\text{in}} \times d_{\text{out}}}$ 和 $\mathcal{M}_b \in \{0, 1\}^{d_{\text{out}}}$ 都是随机生成的,pw、pb are DropConnect probabilities
$$\begin{aligned} \mathcal{M}_w &\sim \mathrm{Bernoulli}(1-p_w), & \mathcal{M}_b &\sim \mathrm{Bernoulli}(1-p_b).... (5)\\ W_{\mathcal{M}} &= \mathcal{M}_w \odot W, & b_{\mathcal{M}} &= \mathcal{M}_b \odot b.............(6) \end{aligned}$$
$$\Theta_t = \{W_k \odot M_k^t \mid k \in \mathcal{S}\}$$

其中 $\mathcal{S}$ 为选定的剪枝层集合 (BN 层和 FC 层), $M_k^t \sim \text{Bernoulli}(p)$ 为逐元素 Bernoulli 掩码, $p$ 为保留率。Conv 层的参数始终完整保留。

每次前向传播都会重新采样 Mask，生成一个在语义上一致但在参数分布上各异的“子模型集合”（Sub-networks）,然后在这个随机剪枝后的子模型 $F(\cdot; \Theta_t)$ 上计算对抗样本的梯度:

$$g_{t+1} = \mu \cdot g_t + \frac{\nabla_x \mathcal{L}(F(x_t^{\text{adv}}; \Theta_t), y_t)}{\|\nabla_x \mathcal{L}(F(x_t^{\text{adv}}; \Theta_t), y_t)\|_1}$$

$$x_{t+1}^{\text{adv}} = \text{Clip}_{x, \epsilon}\{x_t^{\text{adv}} - \alpha \cdot \text{sign}(g_{t+1})\}$$

关键在于: 每次迭代使用不同的随机掩码 $M_k^t$,使得对抗样本在 $T$ 轮迭代中面对 $T$ 个不同的参数化子模型。它必须学会在所有子模型上都有效,而不仅仅是在完整参数的捷径上。

### DropConnect vs. Dropout: 为什么剪枝而不是屏蔽激活

RaPA 选择 DropConnect (对权重置零) 而非 Dropout (对激活置零),原因有二:

- Dropout 屏蔽的是某层的输出激活,代理模型的容量在每个前向传播中整体减少,但参数仍然完整,捷径依然存在。DropConnect 直接切断参数与输出的连接,使捷径本身在每次迭代中不同。
- 实验表明 Dropout 类方法的增益显著小于 DropConnect (ASR 提升约 3-5% vs. 11.7%),验证了参数级随机化才是关键。

### 层级选择: BN 和 FC 是捷径的温床

We thus apply DropConnect to ==the weight and bias parameters of linear layers== as well as ==the transformation parameters of normalization layers==. Both types of layers are widely used in mainstream architectures including Transformer.

作者做了一个消融实验，发现**不要在所有层上都做剪枝**。对卷积层（Conv）做随机剪枝效果很差（因为 Conv 权重本来就稀疏），但对 **Batch Normalization (BN) 层和 Fully Connected (FC) 层** 的参数做随机剪枝，不仅开销极小，效果反而出奇的好！

![[bn-fc.jpg]]
原因: Conv 层由于权重共享和局部连接的特性,参数重要性分布本身已经较为均匀;而 BN 层的缩放/偏移参数和 FC 层的全连接权重天然存在高度的参数重要性集中,是对抗样本"走捷径"的主战场。

### 理论等价性: 重要性均衡正则化

论文通过二阶泰勒展开给出了 RaPA 的理论解释。记 $\theta$ 为完整模型参数, $\hat{\theta} = \theta \odot M$ 为剪枝后参数,期望损失可展开为:
$$\displaystyle \mathbb{E}_{\mathcal{M}} \left[ \mathcal{L}\left(f(x_{\text{adv}}; \mathcal{M} \odot \theta)\right) \right] \approx \mathcal{L}\left(f(x_{\text{adv}}; \theta)\right) + \frac{p(1-p)}{2} \sum_i \frac{\partial^2 \mathcal{L} f(x_{\text{adv}}; \theta)}{\partial \theta_i^2} \theta_i^2$$

$$\mathbb{E}_M[\mathcal{L}(\hat{\theta})] \approx \mathcal{L}(\theta) + \lambda \sum_i H_{ii} \cdot \theta_i^2$$

其中 $H_{ii}$ 为 Hessian 对角元 (参数重要性), $\lambda$ 由剪枝率 $p$ 决定。这意味着: RaPA 在期望上等价于在原始损失函数中添加了一个**权重衰减项,但衰减强度与每个参数的重要性成正比**。高重要性参数 (捷径) 受到更强的正则化惩罚,低重要性参数几乎不受影响。

引入 Gini 系数来量化参数重要性分布的不平等程度:

$$\text{Gini} = \frac{\sum_{i=1}^n \sum_{j=1}^n |H_{ii} - H_{jj}|}{2n \sum_{i=1}^n H_{ii}}$$

其中 Gini = 0 表示完全均等, Gini = 1 表示极端集中。实验显示 RaPA 将 Gini 系数从 0.32 降至 0.08。
RaPA achieves the lowest Gini coefficients among all compared methods, suggesting that ==it effectively flattens the importance distribution and suppresses the over-concentration of sensitivity on a few parameters.== This balanced importance allocation leads to improved robustness and transferability across different architectures.
![[gini.jpg]]

### 机制流程

1. **参数掩码生成**: 对 BN 和 FC 层的每个参数按保留率 $p$ 生成 Bernoulli 掩码 $M_k^t$。
2. **子模型构建**: 将掩码与对应层参数逐元素相乘 $\Theta_t = \theta \odot M^t$, Conv 层参数保持完整。
3. **梯度计算**: 在随机子模型 $F(\cdot; \Theta_t)$ 上执行前向+反向传播,计算 $\nabla_x \mathcal{L}$。
4. **动量累积**: $g_{t+1} = \mu \cdot g_t + \text{normalize}(\nabla_x \mathcal{L})$,按 MI-FGSM 标准流程。
5. **对抗样本更新**: $x_{t+1}^{\text{adv}} = \text{Clip}(x_t^{\text{adv}} - \alpha \cdot \text{sign}(g_{t+1}))$,循环至 $T$ 轮。

## 实验与核心结果

### 实验设置

| 项目     | 配置                                                              |
| ------ | --------------------------------------------------------------- |
| 数据集    | ImageNet-Compatible（经典的 NIPS 2017 攻击挑战赛数据集）                     |
| 扰动预算   | $\ell_{\infty}$, $\epsilon = 16/255$                            |
| 评估指标   | 黑盒条件下的定向攻击成功率 (Targeted ASR)                                    |

### 主要结果

**CNN → Transformer 跨架构迁移 (最核心结果)**:RaPA 相比 FTM 提升 11.7 个百分点,在跨架构这一最困难场景上取得了显著突破。

以 ResNet-50 为代理模型,攻击 ViT-B/16, Swin-B, DeiT-B, CLIP-ViT-B 等 Transformer 目标模型:

| 方法       | 平均 TASR   |
| -------- | --------- |
| FTM      | 33.3%     |
| **RaPA** | **45.0%** |
![[result1.jpg]]




### 消融实验

| 消融方向 | 核心发现 |
|---|---|
| 参数级随机化 vs. 激活级随机化 | DropConnect (权重) >> Dropout (激活) >> 无随机化。证实捷径存在于参数层面而非激活层面 |
| 剪枝层选择 | BN+FC 联合 >> 单独 BN ≈ 单独 FC >> Conv。Conv 层剪枝收益近乎为零 |
| 保留率 $p$ | $p=0.9$ 最优;$p>0.95$ (剪太少) 捷径依赖未充分打破;$p<0.8$ (剪太多) 子模型语义崩溃 |
| 剪枝层数 (BN+FC 的覆盖范围) | 覆盖所有 BN+FC 层最优;仅覆盖最后几层有收益但不足 |
| 迭代数 $T$ 的 scaling | RaPA 的收益随 $T$ 增加而持续扩大 ($T=20$ 时相比 $T=10$ 再提升 3-5%),说明更多子模型变体带来更多泛化性增益 |
| 参数重要性分布的 Gini 系数 | 基线 0.32 → RaPA 0.08,量化验证了均衡化效应 |
| RaPA + 现有方法 (即插即用) | RaPA + Admix, RaPA + CFM, RaPA + DI 等所有组合上均有正向叠加收益,无负互斥 |

## 消融实验关键数据

![[images/table1_obd_diagnosis.png]]
*Table 1 及 OBD 诊断可视化：左侧子表展示剪掉 top-0.5% (最重要) 和 bottom-99.5% (最不重要) 参数后的 ASR 对比；右侧曲线展示不同剪枝比例下 ASR 的衰减趋势。剪掉 Top 0.5% 最重要参数后 ASR 暴跌 46%+，而剪掉 Bottom 99.5% 几乎无影响，直接验证了"捷径参数依赖"假说。*

![[images/fig1_framework.png]]
*Fig. 1: RaPA 方法框架。每次攻击迭代中，对 BN 层和 FC 层参数施加 Bernoulli 掩码生成随机子模型，在此子模型上计算梯度并累积动量更新对抗样本。多次迭代使用不同掩码，迫使对抗样本在所有子模型变体上同时有效。*

![[images/table2_gini.png]]
*Table 2: 参数重要性的 Gini 系数对比 (越低越均匀)。基线 MI-FGSM 在 BN 层的 Gini ≈ 0.32，RaPA 下降至 0.08，直观量化了重要性均衡化效应。逐层分析显示 BN 层和 FC 层是参数重要性集中的重灾区，Conv 层本身已较均匀。*

![[images/table3_cnn_to_transformer.png]]
*Table 3: 以 ResNet-50 为代理模型，攻击五个 ViT/DeiT/Swin/CLIP Transformer 目标模型的定向攻击成功率 (TASR, %)。RaPA 平均 45.0%，相比 CFM 的 33.3% 提升 11.7 个百分点，在跨架构这一最具挑战性的场景上取得显著突破。*

![[images/table4_full_results.png]]
*Table 4 及 Figure 2: 左表展示 RaPA 在 CNN→CNN、CNN→Transformer、Transformer→CNN 等全部代理-目标模型配对上的 TASR，以及对抗训练防御模型 (Inc-v3ens3/ens4, IncRes-v2ens) 上的表现 (~88%)。右图展示不同推理次数 S 下各方法的 ASR scaling 曲线，RaPA 的增益随 S 增加持续扩大。*

![[images/table567_ablations.png]]
*Tables 5-7 及 Figure 3: Table 5 对比不同剪枝层类型 (BN/FC/Conv) 对 ASR 的影响，BN+FC 联合最优。Table 6 展示多种防御方法 (JPEG压缩、随机化、降噪、扩散模型) 下的鲁棒性。Table 7 展示 RaPA 与 DI/TI/SI/Admix/CFM 的即插即用兼容性。Figure 3 展示不同迭代数 T 和保留率 p 下的 ASR 变化曲面。*

## 讨论与局限

### 已被实验证明的结论

- **捷径参数依赖确实存在且是迁移性的主要瓶颈**。OBD 诊断 + Gini 系数 + 消融实验三重证据收敛到同一结论。
- **参数级的 DropConnect 随机化是打破捷径依赖的有效手段**。BN+FC 联合剪枝带来了超过 11 个百分点的跨架构迁移提升,远超 Dropout、特征扰动和梯度平滑等方法。
- **BN 和 FC 层是捷径的主要载体**。Conv 层剪枝效果几乎为零,为后续方法提供了精准的操作目标。
- **RaPA 是免训练的和即插即用的**。可直接嵌入 MI-FGSM, Admix, CFM, DI 等现有框架,无额外训练开销。
- **重要性均衡化的理论模型成立**。二阶泰勒展开 + Gini 系数验证为 RaPA 提供了坚实的理论支撑。

### 尚未被证明或论文未覆盖的问题

- **保留率 $p$ 的最优值是否跨架构稳定**。实验固定 $p=0.9$,BN 层和 FC 层用同一保留率。不同架构 (如 ViT 的 MLP 层 vs. ResNet 的 FC 层) 可能需要不同的保留率,这一维度的消融缺失。
- **剪枝策略的随机性是否最优**。Bernoulli 掩码是完全无结构的随机剪枝,是否存在更优的结构化剪枝策略 (如按参数重要性加权采样、通道级剪枝、或基于 Hessian 的定向剪枝)?
- **定向攻击之外的泛化性**。论文聚焦定向攻击,对非定向攻击和通用对抗扰动 (UAP) 场景中捷径依赖是否同样存在、RaPA 是否同样有效,未做实验。
- **与 FTM/GRA 的叠加上限未知**。RaPA、FTM (特征层扰动) 和 GRA (梯度方向修正) 分别从参数、特征、梯度三个不同粒度处理迁移性问题,三者理论上可以级联叠加,但论文未探索这些组合 (仅测试了与 DI/Admix 的叠加)。
- **在更大模型上的 scaling 行为**。实验中的 ViT-B 和 Swin-B 均为 Base 规模,未经 ViT-L/H 或 LLM 等更大规模模型的验证。随着模型规模增大,捷径参数依赖的行为是否存在质变,未讨论。
- **批量剪枝的效率优化**。每次迭代生成新的 Bernoulli 掩码并在子模型上前向反向传播,在工程实现上是否可以用批量并行技巧 (类似 multi-crop) 来减少开销,论文虽提及但未展开。

### 与同类方法的定位差异

RaPA 是**参数级随机化**在迁移攻击中的首次系统性应用,与此前工作在操作粒度上构成了明确的层次关系:

| 粒度 | 方法 | 核心操作 |
|---|---|---|
| 输入空间 | DI, SI, Admix | 变换/混合输入图像 |
| 梯度方向 | GRA | 邻域梯度余弦相似度修正 |
| 特征空间 | CFM, FTM | 在中间层混合/扰动特征 |
| **模型参数** | **RaPA** | **DropConnect 剪枝 BN/FC 层** |

这种粒度分层意味着 RaPA 与所有上级方法在理论上互补——==输入变换、梯度修正和特征扰动处理的是"信号的多样性",而 RaPA 处理的是"信号接收器的多样性"==。这也解释了为什么 RaPA 与 Admix/CFM/DI 叠加时持续有正向增益:它们在修改攻击流程的不同环节。

### 复现与工程关注点

- **保留率 $p$ 是最关键的超参数**。$p=0.9$ 是论文推荐的通用值,但在不同代理模型上可能需要微调 ($p$ 过低导致子模型语义崩溃,过高则捷径依赖缓解不足)。
- **剪枝层的手动指定**。需要根据代理模型架构手动标识 BN 层和 FC 层,对于自定义或非标准架构 (如 EfficientNet, ConvNeXt) 可能需要额外适配。
- **内存开销可忽略**。Bernoulli 掩码与参数同维度但仅需 1 bit 存储 (可用位图),对显存的影响基本为零。
- **单次迭代的计算开销轻微增加**。额外的掩码生成和前向传播均在子模型上执行,计算量与完整模型相同。唯一增加的是掩码生成的随机数开销 ($O(d)$ 其中 $d$ 为总参数维数),可忽略。

## 个人笔记与追问

### 待验证的复现问题

- ResNet-50 中 BN 层和 FC 层的参数总量远小于 Conv 层,只在 BN+FC 上做剪枝的"子模型语义一致性"是否真的成立? 如果 BN 的参数被大幅扰动,是否会影响整个网络的归一化统计量,导致子模型输出出现系统性偏移而非仅仅是多样性?
- 论文推荐的 $p=0.9$ 意味着每次迭代保留 90% 的 BN+FC 参数,但 BN 层的 $\gamma, \beta$ 参数数量极小 (每个通道仅 2 个标量),随机剪枝的统计意义是什么? 是否本质上等同于在 BN 参数上做了较大的随机扰动 ($\pm 10\%$ 量级)?
- Vit/DeiT 架构中 LN (LayerNorm) 替代了 BN,论文是否对不同归一化层的剪枝效果做了对比? LN 参数的语义与 BN 不同 (逐样本归一化 vs. 逐批次归一化),剪枝 LN 参数的效果是否等同于剪枝 BN?

### 潜在的研究延伸

- **结构化剪枝替代 Bernoulli 随机剪枝**。按 Hessian 重要性加权的 DropConnect (高重要性参数以更高概率被剪枝),理论上等价于更精准的重要性均衡正则化,可能进一步降低 Gini 系数并提升迁移性。
- **RaPA + FTM + GRA 三合一**。三个方法分别操作参数 + 特征 + 梯度三个粒度,理论上完全正交,应该存在乘法叠加效应。如果实验证实这一点,可能将 CNN→Transformer 的定向攻击 TASR 推上 50% 以上的台阶。
- **RaPA 用于对抗训练**。如果在对抗训练中引入 RaPA 式的参数剪枝,能否迫使防御模型在训练时就被迫分散其鲁棒性来源,生成更强的防御? 这是一个自然的对偶方向。

### 跟读文献建议

- CFM: Byun et al., "Introducing Competition to Boost the Transferability of Targeted Adversarial Examples through Clean Feature Mixup", 2023 — RaPA 的直接对比基线
- FTM: Liang et al., "Improving Transferable Targeted Attacks with Feature Tuning Mixup", CVPR 2025 — 同领域同期工作,与 RaPA 存在正交互补关系
- GRA: Zhu et al., "Boosting Adversarial Transferability via Gradient Relevance Attack", ICCV 2023 — 梯度方向的迁移增强方法,与 RaPA 有正交叠加潜力
- DropConnect: Wan et al., "Regularization of Neural Networks using DropConnect", ICML 2013 — RaPA 使用的核心正则化技术来源
- OBD: LeCun et al., "Optimal Brain Damage", NeurIPS 1989 — RaPA 参数重要性诊断的理论基础
- 论文阅读笔记： https://mp.weixin.qq.com/s?__biz=Mzg2MjMyODgzNA==&mid=2247485187&idx=1&sn=53a5f2b58bb9978a94435c1d9aba7f54&poc_token=HDDlKGqjUAsFLJINyYbj-McMa5Wa5xm7eV9WrsDZ
