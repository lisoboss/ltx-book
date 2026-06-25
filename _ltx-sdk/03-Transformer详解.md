---
layout: chapter
title: "Transformer 详解"
chapter_number: 3
subtitle: "双流块架构、自注意力、文本交叉注意力、音视频交叉注意力、RoPE"
permalink: /books/ltx-sdk/03-Transformer详解/
---

# 第 3 章：Transformer 详解

> 双流块架构、自注意力、文本交叉注意力、音视频交叉注意力、RoPE

## 3.1 双流架构

LTX-2 Transformer 由多个 **BasicAVTransformerBlock** 组成。每个块同时处理视频和音频两个模态：

```
┌─────────── BasicAVTransformerBlock ───────────┐
│                                                │
│  Video Flow:                                   │
│  ┌──────────────────────────────────────┐      │
│  │ Self-Attention (3D RoPE)             │      │
│  │  ↓                                   │      │
│  │ Text Cross-Attention                 │      │
│  │  ↓                                   │      │
│  │ Audio-Video Cross-Attention ────────┐│      │
│  │  ↓                                  ││      │
│  │ Feed-Forward Network (SwiGLU)       ││      │
│  └──────────────────────────────────────┘│     │
│                                          │      │
│  Audio Flow:                             │      │
│  ┌──────────────────────────────────────┐│      │
│  │ Self-Attention (1D RoPE)             ││      │
│  │  ↓                                  ││      │
│  │ Text Cross-Attention                ││      │
│  │  ↓                                  ││      │
│  │ Audio-Video Cross-Attention ←───────┘│      │
│  │  ↓                                   │      │
│  │ Feed-Forward Network (SwiGLU)        │      │
│  └──────────────────────────────────────┘      │
│                                                │
└────────────────────────────────────────────────┘
```

## 3.2 注意力机制

### 自注意力 (Self-Attention)

- **视频**：使用 **3D RoPE**（旋转位置编码），编码 `(时间, 高度, 宽度)` 三维位置信息
- **音频**：使用 **1D RoPE**，编码时间维度的位置信息
- 两个模态的自注意力是**独立的**——视频 tokens 只注意视频 tokens，音频 tokens 只注意音频 tokens

### 文本交叉注意力 (Text Cross-Attention)

- **所有 Transformer 块**都包含文本交叉注意力
- 文本 embeddings 来自 Gemma 文本编码器（4096-dim 视频，2048-dim 音频）
- LTX-2.3 中新增 **Prompt AdaLN**：用噪声水平 `sigma` 调制交叉注意力强度

### 音视频交叉注意力 (Audio-Video Cross-Attention)

- 仅在**部分块**中存在（非所有块都做跨模态交互）
- 视频流可以关注音频流，反之亦然
- 这是 LTX-2 实现音视频同步生成的核心机制

## 3.3 FFN

每个流使用 **SwiGLU** 激活的前馈网络，独立处理。

## 3.4 扰动 (Perturbations)

Transformer 支持在自注意力中注入**扰动**，用于：

- **STG (Spatio-Temporal Guidance)**：选择性扰动某些块的自注意力
- 默认 STG 块：`[28]`（LTX-2.3）/ `[29]`（LTX-2）

## 3.5 模态接口 (Modality API)

训练器中使用 `Modality` 数据类来描述每个模态的输入：

```python
from ltx_core.model.transformer.modality import Modality

video = Modality(
    enabled=True,
    latent=video_latents,       # [B, seq_len, 128] 分块化的潜在 tokens
    sigma=sigma,                # [B,] 当前噪声水平（每批次）
    timesteps=video_timesteps,  # [B, seq_len] 逐 token 时间步嵌入
    positions=video_positions,  # [B, 3, seq_len, 2] 位置坐标
    context=video_embeds,       # 文本条件嵌入
    context_mask=None,          # 文本上下文的可选注意力掩码
)

# 前向传播
video_pred, audio_pred = model(video=video, audio=audio, perturbations=None)
```

> **注意**：`Modality` 是不可变的（frozen dataclass）。修改时使用 `dataclasses.replace()`。

## 3.6 `sigma` 与 `timesteps` 的区别

| 参数 | 粒度 | 用途 |
|------|------|------|
| `sigma` | 每批次 `[B,]` | Prompt AdaLN 条件控制（LTX-2.3）、跨模态注意力条件控制（两个版本） |
| `timesteps` | 逐 token `[B, seq_len]` | 条件 tokens 取 0，噪声 tokens 取 sigma（`sigma * denoise_mask`） |

---

**下一章：[第 4 章：VAE 与 Vocoder](04-VAE与Vocoder.md)**
