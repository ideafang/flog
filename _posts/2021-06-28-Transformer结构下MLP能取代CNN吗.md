---
layout: post
comments: false
title: "Transformer结构下MLP能取代CNN吗"
date: 2021-6-28
tags: study-log
---

> 2021年5月的第一周，Google，Facebook，清华，牛津四个不同的机构分别公开了四种纯MLP结构的CV模型，并声称在测试中都取得了一定的效果。这在业界引起了不小的讨论，深度学习模型的研究热点从MLP->CNN/RNN->深层CNN->MLP，转了一圈最终又回到MLP上来，这种现象是否意味着MLP将成为深度学习领域下一个热门研究点？

<!--more-->

本文取自清华大学Meng-Hao Gao分享在Arxiv上的一篇小短文[《Can Attention Enable MLPs To Catch Up With CNNs?》](https://arxiv.org/abs/2105.15078) 本文为读后所做的一些记录以供分享。

这篇文章根据深度学习模型近些年来的发展过程选取了Linear，CNN，Transformer三种经典深度学习结构进行简要分析，最后对上述四个机构提出的四种MLP模型进行研究，并给出了作者认为的下一步研究方向。

# 模型结构（learning architectures）演变

- MLP

  MLP是全连接神经网络，在输入和输出层中间包含一层或多层hidden layer，层与层之间由线性层（linear layers）与激活函数（activation functions）相连。在CNN大规模应用之前，MLP一直都是深度学习领域最主要的模型架构。

  深层的MLP加上激活函数在理论上可以拟合任何的曲线，但由于MLP全连接结构包含巨大的参数量，因此训练MLP的成本很高而且MLP很容易过拟合。

  同时，由于MLP直接将上一层的输出作为本层的输入，因此MLP难以感知（capture）输入的局部特征（local structure），同时也更容易受到无用特征的干扰。

  但近年来随着计算能力的提升以及海量训练数据的构建，我们发现线性层的表达能力远超出我们的想象，线性层还有很多未被发现的优势值得探索。

- CNN

  1. [LeNet](https://www.researchgate.net/publication/2985446_Gradient-Based_Learning_Applied_to_Document_Recognition)[1]: CNN的目的是学习输入的局部特征并降低计算开销。1998年LeCun提出了CNN计算结构并设计了5层CNN网络，提升了手写数字识别的准确率。
  2. [AlexNet](https://proceedings.neurips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf)[2]: 一种大型CNN网络，以显著优势登顶[大型图像识别竞赛（ImageNet Large Scale Visual Recognition Challenge, 2012）](https://image-net.org/challenges/LSVRC/2012/)，CNN开始广泛应用在深度学习的各个领域。
  3. 抛开计算能力和训练数据量的提升，CNN成功的关键在于其引入的归纳偏差（inductive bias）：假设信息具有空间局部性（spatial locality），因此可以通过使用具有共享权重（shared weights）的滑动卷积来减少网络参数的数量。
  4. 但CNN的感受野很有限，学习远程依赖（long-range dependencies）的能力很差。为了增加CNN捕获远程依赖的能力，需要使用更大的卷积核或者特殊结构的卷积。
  5. 由几个小卷积拼成的“大内核”卷积并不是扩大CNN感受野的合适方法[3]

- Transformer

  1. 注意力机制是TransFormer模型的核心，通过注意图（attention map）的形式能够学习输入数据中任意两个位置间的远程依赖关系。
  2. 但是TransFormer模型相比CNN多出的灵活性和降低的局部偏差意味着TransFormer需要更大量的数据才能训练，因此大型TransFormer模型都需要先在超大型数据集上进行预训练，例如GPT-3[4]和ViT[5].

# 基于线性层的模型结构

为了避免上述模型的缺点，降低计算量并获得更好的结果，最近提出了四种很相似的MLP结构。它们的目的都是为了充分发挥linear的优势。在这四种模型里，为了获得稳定的训练效果，都添加了残差链接和正则化。

- [MLP-Mixer](https://arxiv.org/pdf/2105.01601.pdf)[6]
- [External Attention](https://arxiv.org/pdf/2105.02358)[7]
- [Feed-forward-only Model](https://arxiv.org/pdf/2105.02723)[8]
- [ResMLP](https://arxiv.org/pdf/2105.03404.pdf)[9]

![image-20210702173048615](/assets/images/image-20210702173048615.png)

## 共同特点

- long distance interactions

  自注意力机制在上述四种MLP模型中均有体现。MLP-Mixer，ResMLP，Feed-Forward-Only Model三种模型通过在token维度使用线性层来实现不同patches之间的通信。External attention分别在token和feature维度使用softmax加l1_norm来实现。与CNN相比，这些模型能学到不同patches之间的关联以及自动选择合适且不规则的（suitable and irregular）感受野。

- Local semantic information(CV)

  与自然语言中的词token不同，图像中每个像素只包含很少的语义信息并且相邻像素之间很多时候并没有直接的关系（directly informative）。因此在使用MLP结构之前需要从图像中提取出有价值的语义信息，MLP-Mixer，ResMLP，Feed-Forward-Only Model模型选择将图像划分为16x16的局部具有语音信息的小块，External attention使用T2T模型或者CNN模型为MLP提供丰富的语义信息。

- Residual connections

  残差链接用来解决梯度消失问题并使训练过程更稳定，因此这种方法在深度卷积神经网络中十分常用。同样残差链接在MLP网络中也是一种有效的结构，四种模型都有使用。

- Reduced inductive bias

  CNN的局部处理会造成归纳偏差（inductive bias），这会在训练数据充足时降低准确性。上述四种模型在单个token中独立使用linear或者所有token使用一样的linear，降低了归纳偏差。（存疑：所有token使用同样的linear不也会造成归纳偏差吗？）

# 总结和展望

## 总结

上述四种基于MLP的变种TransFormer在ImageNet上具有更快的推理速度，但距离效果最好的CNN和TransFormer模型仍具有5-10%的差距。在准确度和速度的权衡中，它们也没有明显优于轻量级网络。因此需要继续研究去挖掘这些架构的潜力。

## 展望

- 所有的linear都是直接或间接处理图像块，提取局部特征并降低计算复杂度。但是将图片切分为不重叠图像块的过程又引入了归纳偏差。下一步应该将CNN与MLP合理结合起来，利用CNN良好的局部结构提取能力以及MLP处理远程依赖的能力。
- 这四种模型的一个目标是避免直接使用自注意力机制。因此可以参考CV领域成功应用的TransFormer模型进行改进。例如。TransFormer中的多头注意力（multi-head attention）机制，可以尝试应用在这些模型中。
- 这些MLP模型结构可以感知不同结构的数据，如点云（point cloud， graph），可以将其应用到不同的领域。

# Reference paper

[1]. LeCun, Y., Bottou, L., Bengio, Y., & Haffner, P. (1998). Gradient-based learning applied to document recognition. *Proceedings of the IEEE*, *86*(11), 2278-2324.

[2]. Krizhevsky, A., Sutskever, I., & Hinton, G. E. (2012). Imagenet classification with deep convolutional neural networks. *Advances in neural information processing systems*, *25*, 1097-1105.

[3]. Peng, C., Zhang, X., Yu, G., Luo, G., & Sun, J. (2017). Large kernel matters--improve semantic segmentation by global convolutional network. In *Proceedings of the IEEE conference on computer vision and pattern recognition* (pp. 4353-4361).

[4]. Brown, T. B., Mann, B., Ryder, N., Subbiah, M., Kaplan, J., Dhariwal, P., ... & Amodei, D. (2020). Language models are few-shot learners. *arXiv preprint arXiv:2005.14165*.

[5].Dosovitskiy, A., Beyer, L., Kolesnikov, A., Weissenborn, D., Zhai, X., Unterthiner, T., ... & Houlsby, N. (2020). An image is worth 16x16 words: Transformers for image recognition at scale. *arXiv preprint arXiv:2010.11929*.

[6].Tolstikhin, I., Houlsby, N., Kolesnikov, A., Beyer, L., Zhai, X., Unterthiner, T., ... & Dosovitskiy, A. (2021). Mlp-mixer: An all-mlp architecture for vision. *arXiv preprint arXiv:2105.01601*.

[7].Guo, M. H., Liu, Z. N., Mu, T. J., & Hu, S. M. (2021). Beyond self-attention: External attention using two linear layers for visual tasks. *arXiv preprint arXiv:2105.02358*.

[8].Melas-Kyriazi, L. (2021). Do you even need attention? a stack of feed-forward layers does surprisingly well on imagenet. *arXiv preprint arXiv:2105.02723*.

[9].Touvron, H., Bojanowski, P., Caron, M., Cord, M., El-Nouby, A., Grave, E., ... & Jégou, H. (2021). Resmlp: Feedforward networks for image classification with data-efficient training. *arXiv preprint arXiv:2105.03404*.

