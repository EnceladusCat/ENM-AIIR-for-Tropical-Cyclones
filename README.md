# ENM-AIIR for Tropical Cyclones

[English](#english) | [中文](#中文)

A deep learning model for estimating tropical cyclone intensity (Dvorak T-number) from single-band satellite infrared imagery.

**Live Demo:** [https://enm-aiir.pages.dev/](https://enm-aiir.pages.dev/)

---

<a name="english"></a>
## English

### Overview

ENM-AIIR is a deep learning model for estimating tropical cyclone intensity from satellite imagery, based on the Dvorak technique's T-number scale. The model is trained on over 70,000 single-band satellite images of storms from the Northwest Pacific and Southwest Pacific basins.

### Dataset

- **Size:** 3,100+ images
- **Coverage:** Tropical cyclones from the Northwest Pacific and Southwest Pacific basins
- **Imagery:** Single-band long-wave infrared (IR Band-13), which enhances cloud-top brightness and helps capture the spatial structure of the storm (e.g. spiral banding, eye clarity, central dense overcast)
- **Labels:** Each image is paired with a corresponding T-number (Dvorak technique) as the intensity label

### Model Architecture

The model is built on a YOLOv8m backbone, repurposed for ordinal regression rather than standard object detection or classification:

- **Backbone:** YOLOv8m (pretrained, used as a feature extractor)
- **Head:** The original classification head is trained over T-number bins (T1.5 through T8, in 0.5 increments), and predictions are aggregated into a single continuous value via confidence-weighted averaging across class probabilities, rather than taking a hard argmax. This produces smoother, more precise intensity estimates than discrete classification alone.

### Training Configuration

| Parameter | Value |
|---|---|
| Base model | yolov8m-cls.pt |
| Image size | 512×512 |
| Batch size | 24 |
| Initial learning rate | 0.0015 |
| Optimizer | AdamW |
| Hardware | Apple Silicon (MPS) |

Checkpoint selection departs from the Ultralytics default: rather than relying on the framework's built-in fitness metric (top-1 accuracy), every saved epoch checkpoint was re-evaluated on the validation set using the weighted-average regression output, and the checkpoint with the lowest RMSE on continuous T-value prediction was selected as the final model. This better reflects the model's real task, which is continuous intensity estimation rather than discrete bin classification.

### Evaluation Results

Evaluated on the held-out validation split, restricted to the T2.5–T8.0 intensity range:

| Metric | Value |
|---|---|
| RMSE | 0.31 (T-number scale) |
| MAE | 0.24 (T-number scale) |
| Accuracy (±0.5 T-number) | 90.40% |

*Note: RMSE and MAE are reported on the T-number scale (e.g. a value of 3.5 represents T3.5). Internally, T-numbers are scaled by ×10 for class indexing during training/inference; the figures above have been converted back to true T-number units.*

### Usage

```bash
git clone https://github.com/EnceladusCat/ENM-AIIR-for-Tropical-Cyclones.git
cd ENM-AIIR-for-Tropical-Cyclones
```

A hosted, browser-based version of the model is available at [https://enm-aiir.pages.dev/](https://enm-aiir.pages.dev/) — no local setup required.

### Limitations

- The model was mostly trained and validated on Northwest/Southwest Pacific basin storms only; performance on storms from other basins (Atlantic, Indian Ocean, etc.) has not been evaluated and may differ.
- Evaluation metrics above are reported for the T1.5–T8.0 range; performance at the extremes of the intensity scale (very weak or very intense systems) may be less reliable due to lower sample density in those bins.
- This model is intended as a research and decision-support tool. It is not a substitute for official tropical cyclone intensity estimates issued by national meteorological agencies (e.g. JMA, JTWC, PAGASA).

### License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

Copyright (c) 2026 MFI

### Citation

If you use this model or dataset in your work, please cite:

```
ENM-AIIR: A Deep Learning Model for Tropical Cyclone Intensity Estimation
MFI, 2026
https://github.com/EnceladusCat/ENM-AIIR-for-Tropical-Cyclones
```

---

<a name="中文"></a>
## 中文

### 项目简介

ENM-AIIR 是一个基于卫星图像、依照德沃夏克分析法（Dvorak Technique）T值标准来估计热带气旋强度的深度学习模型。模型训练数据集包含超过7万张来自西北太平洋和西南太平洋风暴的单波段卫星图像。

### 数据集

- **样本数量：** 3100张以上
- **覆盖范围：** 西北太平洋与西南太平洋海域的热带气旋
- **图像类型：** 单波段卫星长波红外（IR Band-13）波段，该波段能增强云层亮度，有助于更好地捕捉风暴的空间结构（如螺旋云带、风眼清晰度、中心密闭云区等特征）
- **标签：** 每张图像对应一个独立的T值（依据德沃夏克分析法）作为强度标签

### 模型架构

模型以 YOLOv8m 作为骨干网络，但用途并非常规的目标检测或图像分类，而是改造为序数回归任务：

- **骨干网络：** YOLOv8m（预训练权重，用作特征提取器）
- **输出层：** 原始分类头按T值区间（T1.5至T8，以0.5为间隔）进行训练，最终预测结果并非直接取概率最高的类别（argmax），而是通过对各类别置信度加权平均得到一个连续数值，相比纯离散分类能得到更平滑、更精细的强度估计结果。

### 训练配置

| 参数 | 数值 |
|---|---|
| 基础模型 | yolov8m-cls.pt |
| 输入图像尺寸 | 512×512 |
| Batch Size | 24 |
| 初始学习率 | 0.0015 |
| 优化器 | AdamW |
| 训练硬件 | Apple Silicon（MPS） |

权重挑选方式与Ultralytics默认机制不同：并未直接采用框架内置的fitness指标（即top-1分类准确率），而是对每一轮保存下来的checkpoint，分别在验证集上用加权平均回归输出重新评估，最终选取在连续T值预测上RMSE最低的那一轮权重作为最终模型。这样选出的权重更符合模型实际要完成的任务——连续强度估计，而非离散区间分类。

### 评估结果

在保留验证集（T2.5–T8.0强度区间）上的评估结果：

| 指标 | 数值 |
|---|---|
| RMSE | 0.31（T值尺度） |
| MAE | 0.24（T值尺度） |
| 准确率（误差≤0.5个T值） | 90.40% |

*说明：RMSE与MAE均以T值原始尺度呈现（如数值3.5代表T3.5）。模型内部训练/推理时T值会乘以10用于类别索引，上表数字已换算回真实T值单位。*

### 使用方式

```bash
git clone https://github.com/EnceladusCat/ENM-AIIR-for-Tropical-Cyclones.git
cd ENM-AIIR-for-Tropical-Cyclones
```

也可直接通过浏览器访问在线版本，无需本地部署：[https://enm-aiir.pages.dev/](https://enm-aiir.pages.dev/)

### 局限性

- 模型大部分基于西北/西南太平洋海域的风暴数据训练与验证，在其他海域（如大西洋、印度洋等）风暴上的表现尚未评估，可能存在差异。
- 上述评估指标基于T2.5–T8.0区间统计，在强度量表两端（极弱或极强系统）由于该区间样本数量较少，模型表现的可靠性可能有所下降。
- 本模型定位为科研与辅助决策工具，不能替代各国气象机构（如日本气象厅JMA、美国联合台风警报中心JTWC、菲律宾大气地球物理和天文管理局PAGASA等）发布的官方热带气旋强度评估。

### 许可协议

本项目采用 MIT 许可证开源，详见 [LICENSE](LICENSE) 文件。

Copyright (c) 2026 MFI

### 引用方式

如在您的工作中使用本模型或数据集，请按以下格式引用：

```
ENM-AIIR: A Deep Learning Model for Tropical Cyclone Intensity Estimation
MFI, 2026
https://github.com/EnceladusCat/ENM-AIIR-for-Tropical-Cyclones
```