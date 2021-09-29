# Transformer
## 1. An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale.

## 2. DeepViT: Towards Deeper Vision Transformer.
ViTs has been successfully applied in image classification tasks recently. Can we improve the performance of ViTs when scaled to be deeper.
### 2.1 Attention collapse
Motivated by the success of deep CNNs, will the performance of ViTs increase as depth increases.When the hidden dimension and the number of heads are fixed, use different number of transformer blocks to build multiple ViT models. The overall performances for image classification are summarized
<img src='/Users/lengguangjie/paper-list/figures/2_accuracy_for_different-blocks.png' alt='accuracy for different numbers of transformer blocks'></img>
