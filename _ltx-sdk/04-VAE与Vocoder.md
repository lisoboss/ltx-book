---
layout: chapter
title: "VAE 与 Vocoder"
chapter_number: 4
subtitle: "Video VAE 压缩率与形状、Audio VAE、HiFi-GAN vs BigVGAN v2"
permalink: /books/ltx-sdk/04-VAE与Vocoder/
---

# 第 4 章：VAE 与 Vocoder

> Video VAE 压缩率与形状、Audio VAE、HiFi-GAN vs BigVGAN v2

## 4.1 Video VAE

LTX-2 的 Video VAE 负责在像素空间和潜在空间之间转换视频数据。

### 编码器 (VideoEncoder)

- 输入：原始视频帧
- 输出：压缩的潜在表示
- **空间压缩**：32×（高度和宽度各压缩 32 倍）
- **时间压缩**：8×
- **潜在通道数**：128

在 ltx-core 中的加载方式：

```python
from ltx_core.model.video_vae import VideoEncoder, VideoDecoder

encoder = VideoEncoder(...)
decoder = VideoDecoder(...)
```

### 形状计算

给定输入张量 `[B, C, F, H, W]`：
- 编码后潜在形状：`[B, 128, ceil(F/8), ceil(H/32), ceil(W/32)]`
- 帧数约束：`F % 8 == 1`（有效值：1, 9, 17, 25, 33, 41, 49, 57, 65, 73, 81, 89, 97, 121）
- 分辨率约束：`H % 32 == 0`, `W % 32 == 0`

### 分块化 (Patchification)

潜在特征被进一步分块化为 Transformer 可以处理的 token 序列：

```python
from ltx_core.components.patchifiers import VideoLatentPatchifier

patchifier = VideoLatentPatchifier()
# 输入: [B, 128, F_lat, H_lat, W_lat]
# 输出: [B, seq_len, 128]  (每个 token 的 dim = 128 × 1 × 1 × 1 = 128)
```

## 4.2 Audio VAE + Vocoder

### Audio VAE

- 将音频波形压缩到潜在空间
- **潜在通道数**：8
- **梅尔频谱 bin 数**：16
- 分块化：`AudioPatchifier` → token dim = `8 × 16 = 128`

### Vocoder

将音频潜在表示转换为波形。版本自动检测：

| 版本 | Vocoder | 特点 |
|------|---------|------|
| LTX-2 (19B) | HiFi-GAN | 标准声码器 |
| LTX-2.3 (22B) | BigVGAN v2 + BWE | 带宽扩展，更高音质 |

检测逻辑在 `VocoderConfigurator` 中：如果 checkpoint config 中有 `config["vocoder"]["bwe"]`，则使用 `VocoderWithBWE`；否则使用 `Vocoder`。

```python
from ltx_core.model.audio_vae import AudioDecoder
from ltx_core.model.audio_vae.vocoder import Vocoder, VocoderWithBWE

decoder = AudioDecoder(...)
# 类型自动检测，无需手动指定版本
```

### 采样率

`VocoderWithBWE` 通过 `output_sampling_rate` 属性暴露输出采样率。

## 4.3 实践要点

1. **帧数约束是最常见的错误来源**：生成时必须确保帧数满足 `F % 8 == 1`
2. **空间上采样器**（x2 或 x1.5）在推理管道中可提升输出分辨率
3. **音频和视频的解码是独立的**：可以只解码视频或只解码音频

---

**下一章：[第 5 章：文本编码器](05-文本编码器.md)**
