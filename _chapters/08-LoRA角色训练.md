---
layout: chapter
title: "LoRA 角色训练"
book: "AI 动漫"
chapter_number: 8
subtitle: "训练专属的动漫角色 LoRA，让你在不同镜头和场景中保持角色一致性"
permalink: /chapters/LoRA角色训练/
---

# 第 8 章：LoRA 角色训练

> 训练专属的动漫角色 LoRA，让你的角色在所有镜头中保持一致的外貌、服装和风格。

## 8.1 为什么需要角色 LoRA

基础 LTX-2 模型虽然强大，但无法精确控制角色外观。每次生成，同一段 prompt 可能产生外观差异巨大的角色。LoRA 角色训练解决了这个问题：

```
无 LoRA:
  Prompt: "anime girl with silver hair"
  → 第1次: 短发银发少女 A
  → 第2次: 长发银发少女 B
  → 第3次: 完全不相关的角色 C

有 LoRA:
  Prompt: "MYTRIGGER anime girl with silver hair"
  → 每次生成: 外观一致的特定角色 ★
```

## 8.2 训练环境准备

### 8.2.1 硬件要求

| 训练类型 | 最低 GPU | 推荐 GPU | 预计时间 |
|----------|----------|----------|----------|
| T2V LoRA (标准) | A100 80GB | H100 80GB | 4-12 小时 |
| T2V LoRA (低显存) | RTX 5090 32GB | A6000 48GB | 8-24 小时 |
| IC-LoRA | A100 80GB | H100 80GB | 8-24 小时 |
| Full Fine-tune | 4× H100 80GB | 8× H100 80GB | 数天 |

> **注意**：训练**必须**在 Linux 上进行（Triton 依赖）。

### 8.2.2 训练数据要求

```yaml
# 数据质量金字塔
         ┌──────────┐
         │  多样性   │ ← 不同角度、光照、背景、动作
         ├──────────┤
         │  一致性   │ ← 同一角色/风格的所有片段
         ├──────────┤
         │  清晰度   │ ← 1080p+, 无压缩伪影, 良好光照
         ├──────────┤
         │  数量     │ ← 推荐 50-200 个片段（5-30秒/片段）
         └──────────┘

# 动漫角色训练的数据集建议:
- 最少: 30 个片段，每个 5-10 秒
- 推荐: 100 个片段，每个 5-15 秒
- 理想: 200+ 个片段，覆盖多种姿势、角度、表情、光照
```

## 8.3 完整训练流程

### Step 1: 准备训练视频

```bash
# 收集角色在各角度/动作/光照下的视频片段
# 放在统一目录下
ls anime_character_clips/
# char_front_walk.mp4
# char_side_run.mp4
# char_closeup_talk.mp4
# char_action_fight.mp4
# ...
```

### Step 2: 分割场景

```bash
# 将长视频按场景切分
uv run python scripts/split_scenes.py anime_character_clips/ scenes_output/ \
    --filter-shorter-than 3s \
    --filter-longer-than 30s
```

### Step 3: 生成 Caption

```bash
# 启动 caption 服务（在另一个终端）
uv run python scripts/serve_captioner.py

# 为视频生成描述
uv run python scripts/caption_videos.py scenes_output/ \
    --output scenes_output/dataset.json
```

生成的 `dataset.json` 格式：

```json
[
  {
    "video": "scenes_output/char_front_walk_scene_001.mp4",
    "caption": "The silver-haired anime character walks forward confidently, her long hair flowing behind her, wearing a white and blue school uniform, bright daylight outdoor setting"
  },
  {
    "video": "scenes_output/char_side_run_scene_002.mp4",
    "caption": "Side profile of the silver-haired girl running with determination, her uniform skirt fluttering, wind blowing through her hair, athletic track background"
  }
]
```

> **重要**：检查并修正自动生成的 caption！确保描述准确反映了角色外观和动作。不一致的 caption 会导致训练失败。

### Step 4: 预处理数据

```bash
# 预处理: 编码视频 latent + 文本嵌入
uv run python scripts/process_dataset.py scenes_output/dataset.json \
    --resolution-buckets "768x512x49" \
    --model-path /path/to/ltx-2.3-22b-dev.safetensors \
    --text-encoder-path /path/to/gemma-3-12b \
    --lora-trigger "MYANIMECHAR" \
    --output-dir ./precomputed

# 多分辨率 bucket（更好的泛化性）
uv run python scripts/process_dataset.py scenes_output/dataset.json \
    --resolution-buckets "768x512x49;512x512x49;512x768x49" \
    --model-path /path/to/ltx-2.3-22b-dev.safetensors \
    --text-encoder-path /path/to/gemma-3-12b \
    --lora-trigger "MYANIMECHAR"
```

> `--lora-trigger "MYANIMECHAR"` 会将触发词自动添加到所有 caption 前面。推理时在 prompt 中包含这个触发词即可激活 LoRA。

### Step 5: 配置训练参数

创建 `configs/my_anime_character_lora.yaml`：

```yaml
# ===== 模型配置 =====
model:
  model_path: "/path/to/ltx-2.3-22b-dev.safetensors"
  text_encoder_path: "/path/to/gemma-3-12b"
  training_mode: "lora"

# ===== LoRA 配置 =====
lora:
  rank: 32          # 动漫角色建议 32-64
  alpha: 32         # 通常 = rank
  dropout: 0.0      # 数据少时可设 0.05-0.1 防过拟合
  target_modules:   # 视频+音频注意力
    - "to_k"
    - "to_q"
    - "to_v"
    - "to_out.0"

# ===== 训练策略 =====
training_strategy:
  name: "flexible"
  video:
    is_generated: true
    latents_dir: "latents"
  audio:
    is_generated: true
    latents_dir: "audio_latents"

# ===== 优化器 =====
optimization:
  learning_rate: 1.0e-4
  batch_size: 1
  gradient_accumulation_steps: 4
  max_train_steps: 2000
  lr_scheduler: "cosine"
  warmup_steps: 200

# ===== 加速 =====
acceleration:
  mixed_precision: "bf16"
  # 对于 A100/H100 标准配置不需要额外设置
  # 对于 32GB GPU: 参考 t2v_lora_low_vram.yaml

# ===== 数据 =====
data:
  preprocessed_data_root: "./precomputed"
  num_workers: 4

# ===== 验证（训练过程中检查效果）=====
validation:
  every_n_steps: 500
  generate_audio: false  # 纯视频可以先关掉，节省时间
  samples:
    - prompt: "MYANIMECHAR standing in a cherry blossom garden, soft wind, "
              "sunset lighting, anime style, high quality"
    - prompt: "MYANIMECHAR walking through a futuristic city at night, "
              "neon reflections, cyberpunk aesthetic"
    - prompt: "MYANIMECHAR sitting in a classroom by the window, afternoon "
              "sunlight, peaceful atmosphere"

# ===== Checkpoint =====
checkpoints:
  save_every_n_steps: 500
  keep_last_n: 3

# ===== 输出 =====
output_dir: "outputs/my_anime_character"
```

### Step 6: 启动训练

```bash
# 单 GPU
uv run python scripts/train.py configs/my_anime_character_lora.yaml

# 多 GPU (accelerate)
uv run accelerate launch scripts/train.py configs/my_anime_character_lora.yaml

# 低显存配置 (RTX 5090 32GB)
uv run python scripts/train.py configs/t2v_lora_low_vram.yaml
```

### Step 7: 监控训练

```bash
# 训练日志会打印到控制台
# Weights & Biases (如果配置了 wandb)
# 验证输出保存到 output_dir/validation/

# 关键指标:
# - loss 平稳下降 → 正常
# - loss 剧烈震荡 → LR 过高
# - loss 不变 → LR 过低或数据问题
# - 验证视频中角色一致性越来越好 → 训练有效
```

## 8.4 动漫角色训练最佳实践

### 8.4.1 数据收集策略

```
动漫角色的训练数据应覆盖:

1. 多角度 (至少):
   ├── 正面 (Front) — 最重要
   ├── 45度 (Three-quarter)
   ├── 侧面 (Profile)
   └── 背面 (Back) — 可选

2. 多距离:
   ├── 特写 (面部细节)
   ├── 中景 (上半身)
   └── 全景 (全身+环境)

3. 多表情 (至少 5 种):
   ├── 中性/平静
   ├── 开心/微笑
   ├── 愤怒/坚定
   ├── 悲伤/沉思
   └── 惊讶

4. 多光照:
   ├── 正面光 (清晰)
   ├── 侧光 (立体感)
   └── 背光 (轮廓)

5. 多动作:
   ├── 静止站立
   ├── 行走
   ├── 跑步
   └── 特定动作 (拔刀/跳跃/挥手等)
```

### 8.4.2 数据质量检查清单

```
[ ] 角色在所有片段中是同一个人（相同发型、服装、体型）
[ ] 视频分辨率 ≥ 720p（推荐 1080p）
[ ] 没有严重的压缩伪影
[ ] 没有水印覆盖角色面部
[ ] 没有剧烈的镜头抖动
[ ] 光照基本一致或覆盖多变
[ ] Caption 准确描述了角色和场景
[ ] 触发词统一（如 MYANIMECHAR）
```

### 8.4.3 LoRA Rank 选择

```python
lora_rank_guide = {
    8:  "极轻量，仅捕获基本色调 + 发型（推荐用于风格 LoRA）",
    16: "轻量，捕获角色基本外观（数据少时的安全选择）",
    32: "标准，推荐起点（捕获角色+服装+基本动作风格）",
    64: "较高，更多细节（200+ 片段时使用）",
    128:"最高，需要大量数据防过拟合",
}
```

### 8.4.4 避免的常见错误

| 错误 | 后果 | 预防 |
|------|------|------|
| 数据中混入多个角色 | LoRA 学习模糊的"平均脸" | 严格筛选，每角色单独训练 |
| 所有片段在相同背景 | LoRA 学会背景而非角色 | 在多种背景中拍摄/合成 |
| Caption 不一致 | 不同片段学不同特征 | 检查并手动修正所有 caption |
| 片段过长（>30秒） | 场景切换时 loss 混乱 | 用 split_scenes.py 切分 |
| Trigger 词用错 | 推理时无法激活 LoRA | 训练和推理使用相同 trigger |

## 8.5 使用训练好的 LoRA

### 8.5.1 推理

```python
from ltx_core.loader import LTXV_LORA_COMFY_RENAMING_MAP, LoraPathStrengthAndSDOps

# 加载你的角色 LoRA
my_character_lora = LoraPathStrengthAndSDOps(
    "outputs/my_anime_character/checkpoint-2000/lora.safetensors",
    strength=0.8,  # 0.5-1.0，越高越接近训练数据
    sd_ops=LTXV_LORA_COMFY_RENAMING_MAP,
)

# 在任意 Pipeline 中使用
pipeline = TI2VidTwoStagesPipeline(
    checkpoint_path="models/ltx-2.3-22b-dev.safetensors",
    distilled_lora=distilled_lora,
    spatial_upsampler_path="models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors",
    gemma_root="models/gemma-3-12b",
    loras=[my_character_lora],  # ← 你的角色 LoRA
)

# Prompt 中必须包含 trigger 词
video, _ = pipeline(
    prompt="MYANIMECHAR walking through a bamboo forest at sunrise, "
           "soft golden light filtering through leaves, peaceful atmosphere",
    ...
)
```

### 8.5.2 CLI 方式

```bash
python -m ltx_pipelines.ti2vid_two_stages \
    --checkpoint-path models/ltx-2.3-22b-dev.safetensors \
    --distilled-lora models/ltx-2.3-22b-distilled-lora-384-1.1.safetensors 0.6 \
    --spatial-upsampler-path models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors \
    --gemma-root models/gemma-3-12b \
    --lora outputs/my_anime_character/checkpoint-2000/lora.safetensors 0.8 \
    --prompt "MYANIMECHAR standing heroically on a cliff edge..." \
    --height 512 --width 768 --num-frames 73 \
    --output-path outputs/character_test.mp4
```

### 8.5.3 多 LoRA 叠加

```python
# 可以叠加多个 LoRA: 角色 + 风格 + 摄影机运动
character_lora = LoraPathStrengthAndSDOps("loras/char.safetensors", 0.8, sd_ops)
style_lora = LoraPathStrengthAndSDOps("loras/anime_style.safetensors", 0.5, sd_ops)
camera_lora = LoraPathStrengthAndSDOps("loras/camera_dolly_in.safetensors", 0.3, sd_ops)

pipeline = TI2VidTwoStagesPipeline(
    ...
    loras=[character_lora, style_lora, camera_lora],
)
```

## 8.6 IC-LoRA 角色训练（进阶）

对于需要更强的角色一致性控制，可以训练 IC-LoRA：

```yaml
# v2v_ic_lora.yaml 风格的角色 IC-LoRA 配置
model:
  model_path: "/path/to/ltx-2.3-22b-dev.safetensors"
  text_encoder_path: "/path/to/gemma-3-12b"
  training_mode: "lora"

lora:
  rank: 64
  target_modules:  # 视频专用模块
    - "attn1.to_k"
    - "attn1.to_q"
    - "attn1.to_v"
    - "attn1.to_out.0"
    - "attn2.to_k"
    - "attn2.to_q"
    - "attn2.to_v"
    - "attn2.to_out.0"
    - "ff.net.0.proj"
    - "ff.net.2"

training_strategy:
  name: "flexible"
  video:
    is_generated: true
    latents_dir: "latents"
    conditions:
      - type: reference
        latents_dir: "reference_latents"
        probability: 1.0
      - type: first_frame
        probability: 0.2
  # 音频可选
```

IC-LoRA 训练的数据集需要**配对视频**（目标视频 + 参考视频）：

```json
[
  {
    "video": "anime_final/clip_001.mp4",
    "reference_video": "anime_lineart/clip_001.mp4",
    "caption": "Finished anime clip with colors and shading"
  },
  {
    "video": "anime_final/clip_002.mp4",
    "reference_video": "anime_pose/clip_002.mp4",
    "caption": "Character in action pose, dynamic movement"
  }
]
```

---

**下一章：[第 9 章：视频扩展、修复与 LipDub 配音](09-视频扩展与修复.md)**
