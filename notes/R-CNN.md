# Rich feature hierarchies for accurate object detection and semantic segmentation Tech report (v5)


## 产生候选框（Selective search)
* 聚类产生初始的分割区域
* 根据颜色、纹理、大小、形状、相似度加权合并产生不同层次的候选框

```flow
s=start:开始
e=end:结束
o=operation:操作项

s-o-e
```

## 候选框压缩(Warped region)
* 等比例缩放（保留长宽比例，连带临近像素）
* 等比例缩放（保留长宽比例，`不`连带临近像素）
* 非等比例缩放（`不`保留长宽比例，`不`连带临近像素）

* 文章采用连带临近像素的非等比例缩放（p=16，连带临近16个像素）
  
## 计算特征（Compute CNN features）
* AlexNet 取掉全连接层，只保留卷积层和池化层
* 对训练的网络在相应的压缩得到的候选框进行微调
  
## 区域分类（Classify regions）

### 支撑向量机（SVM）
* 利用 `CNN` 提取的固定长度的特征，训练 `N+1` (样本 + 背景) 个 `SVM` 分类器。

### Bounding box regression
得到固定长度的特征之后，一边进行分类，一边进行回归，`使用回归来矫正候选框的位置`，得到预测框。
* 输入：训练的pair:$\{(P^{i}, G^{i})\}_{i=1,...,N}$， 其中$P^{i} = (P_{x}^{i},, P_{y}^{i}, P_{w}^{i}, P_{h}^{i})$f分别代表着第 i 个候选样本的中心位置以及候选样本的长和宽。
* 我们希望回归四个参数来表示候选样本与真实样本的变换 $d_{x}(P), d_{y}(P), d_{h}(P), d_{w}(P)$
$$
\hat{G}_{x} = P_{w}d_{x}(P) + P_{x}
\\
\hat{G}_{y} = P_{h}d_{y}(P) + P_{y}
\\
\hat{G}_{w} = P_{w}\exp{d_{w}(P)}
\\
\hat{G}_{h} = P_{h}\exp{d_{h}(P)}
 $$

* 使用 $d_{\star}(P) = w_{\star}^{\rm T}\phi_{5}(P)$， $\phi_{5}(P)$ 表示原始的候选样本经过CNN提取样本后得到的位置参数
* 使用 `岭回归` （Ridge Regression）

$$
\mathbf{w}_{\star}=\underset{\hat{\mathbf{w}}_{\star}}{\operatorname{argmin}} \sum_{i}^{N}\left(t_{\star}^{i}-\hat{\mathbf{w}}_{\star}^{\mathrm{T}} \boldsymbol{\phi}_{5}\left(P^{i}\right)\right)^{2}+\lambda\left\|\hat{\mathbf{w}}_{\star}\right\|^{2}
$$

$$
\begin{aligned}
t_{x} &=\left(G_{x}-P_{x}\right) / P_{w} \\
t_{y} &=\left(G_{y}-P_{y}\right) / P_{h} \\
t_{w} &=\log \left(G_{w} / P_{w}\right) \\
t_{h} &=\log \left(G_{h} / P_{h}\right)
\end{aligned}
$$

### 正负样本

* CNN微调时的正负样本
  * 与 `ground-truth` 的IOU最大且IOU大于0.5的候选框为证正样本，
  * 其余候选框为负样本

* SVM训练时的正样本
  * `ground-truth` 为正样本
  * 与 `ground-truth` 的 `IOU` 小于0.3 的候选框为负样本
  * 忽略 `IOU` 大于 0.3 的样本
  * 0.3 是通过搜素确定的

* 使用 `SVM` 而不是直接用 `Softmax`
  * 微调的时候，得到的候选框并不是真实的物体，而是与物体重合度较高的候选框
  * `SVM` 使用的真实的人工标注的正样本