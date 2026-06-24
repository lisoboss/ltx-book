# 第 7 章：IC-LoRA 风格控制

> IC-LoRA（In-Context LoRA）是 LTX-2 独有的能力——通过一张或多张参考图像/视频，精确控制生成内容的风格、角色外观和运动模式。这是实现"统一动漫风格"的核心技术。

## 7.1 IC-LoRA 原理

### 7.1.1 与传统 LoRA 的区别

```
传统 LoRA:
  训练时: 学习特定风格/角色的低秩适配器权重
  推理时: 加载 LoRA → 风格固化在权重中
  切换风格: 需要卸载当前 LoRA，加载另一个

IC-LoRA:
  训练时: 学习"如何根据参考图像调整输出"的能力（元学习）
  推理时: 提供参考图像/视频 → 实时风格迁移
  切换风格: 只需更换参考图像，无需重新加载模型
```

### 7.1.2 IC-LoRA 的工作机制

```
┌─────────────────────────────────────────────────────┐
│                  IC-LoRA 推理流程                      │
├─────────────────────────────────────────────────────┤
│                                                       │
│  [参考视频/图像]              [噪声 Latent]            │
│       │                           │                   │
│       ├─ VAE Encode ──→ [Ref Latent]                  │
│       │                           │                   │
│       │              ┌────────────┴──────────┐        │
│       └──────────────┤  Token 序列拼接        │        │
│                      │  [Target | Reference]  │        │
│                      └────────────┬──────────┘        │
│                                   │                   │
│                      ┌────────────┴──────────┐        │
│                      │  Transformer 处理      │        │
│                      │  (48 Blocks)           │        │
│                      │                        │        │
│                      │  Reference tokens:     │        │
│                      │  - sigma=0 (无噪声)     │        │
│                      │  - 参与 Self-Attention │        │
│                      │  - 无 loss / 不更新    │        │
│                      │                        │        │
│                      │  Target tokens:        │        │
│                      │  - 正常去噪             │        │
│                      │  - Attention 关注 Ref  │        │
│                      └────────────┬──────────┘        │
│                                   │                   │
│                          只取 Target 部分              │
│                          → VAE Decode → 输出          │
│                                                       │
└─────────────────────────────────────────────────────┘
```

## 7.2 可用的预训练 IC-LoRA

| IC-LoRA | 功能 | 适用动漫场景 |
|---------|------|-------------|
| **Union Control** | 综合控制（边缘+深度+姿态） | 线稿→上色动画、草稿→成品 |
| **Motion Track** | 运动轨迹控制 | 精确控制角色运动路径 |
| **Pose Control** | 人体姿态控制 | 角色动作复刻 |
| **Detailer** | 细节增强 | 提升画面精细度 |
| **HDR** | HDR 视频输出 | 高品质后期调色 |
| **LipDub** | 唇形同步 | 角色配音对口型 |

## 7.3 ICLoraPipeline 使用

### 7.3.1 基础视频风格迁移

```python
from ltx_core.loader import LTXV_LORA_COMFY_RENAMING_MAP, LoraPathStrengthAndSDOps
from ltx_pipelines.ic_lora import ICLoraPipeline
from ltx_pipelines.utils.args import VideoConditioningAction
from ltx_pipelines.utils.media_io import decode_video_by_frame, encode_video

# IC-LoRA 配置
ic_lora = LoraPathStrengthAndSDOps(
    "models/loras/ic-lora-union-control.safetensors",
    strength=0.5,  # 参考强度: 0.0=忽略参考, 1.0=完全跟随参考
    sd_ops=LTXV_LORA_COMFY_RENAMING_MAP,
)

pipeline = ICLoraPipeline(
    distilled_checkpoint_path="models/ltx-2.3-22b-distilled-1.1.safetensors",
    spatial_upsampler_path="models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors",
    gemma_root="models/gemma-3-12b",
    loras=[ic_lora],
)

# 参考视频: 可以是线条动画、3D 预览、或任何结构参考
reference_frames = decode_video_by_frame("reference/animation_draft.mp4")

# 视频条件: 参考视频 + 首帧图像
video_condition = VideoConditioningAction(
    reference_frames=reference_frames,
    image_path="keyframes/first_frame.jpg",  # 可选首帧
    image_frame_idx=0,
    conditioning_frame_idxs=None,  # None = 所有帧都受参考约束
    conditioning_attention_mask_path=None,  # 可选: 空间注意力遮罩
)

video, audio = pipeline(
    prompt="High quality anime style, vibrant colors, clean linework, "
           "detailed shading, cel-shaded, professional animation quality, "
           "character walking through a magical forest",
    negative_prompt="worst quality, low quality, blurry, deformed",
    seed=42,
    height=512,
    width=768,
    num_frames=73,
    frame_rate=24.0,
    video_condition=video_condition,
)

encode_video(video, 24.0, audio, "outputs/stylized_anime.mp4")
```

### 7.3.2 CLI 方式

```bash
python -m ltx_pipelines.ic_lora \
    --checkpoint-path models/ltx-2.3-22b-distilled-1.1.safetensors \
    --spatial-upsampler-path models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors \
    --gemma-root models/gemma-3-12b \
    --lora models/loras/ic-lora-union-control.safetensors 0.5 \
    --prompt "High quality anime, vibrant colors, cel-shaded..." \
    --video-condition reference/draft.mp4 \
    --image-condition keyframes/first.jpg 0 1.0 33 \
    --height 512 --width 768 --num-frames 73 \
    --seed 42 \
    --output-path outputs/ic_lora_result.mp4
```

## 7.4 动漫风格参考策略

### 7.4.1 参考视频制作

IC-LoRA 的参考视频不需要是高质量的。关键是**结构信息**：

```python
# 方案 1: Canny 边缘检测参考
# 使用 compute_reference.py 生成 Canny 边缘图
# 这些边缘图作为结构参考，IC-LoRA 将其转化为动漫风格
"""
bash:
uv run python scripts/compute_reference.py my_videos/ \
    --output my_videos/dataset.json
"""

# 方案 2: 深度图参考
# 用深度估计模型生成深度图作为结构参考

# 方案 3: 姿态骨架参考
# 用 OpenPose/DWPose 等提取姿态骨架 → IC-LoRA 生成带肌肉/服装的角色

# 方案 4: 3D 渲染低模参考
# 用 Blender 的低模+简单材质渲染 → IC-LoRA 转化为高精度动漫
```

### 7.4.2 参考强度调优

```python
# 不同场景的推荐参考强度

strength_presets = {
    "strict_follow": 0.8,   # 严格遵循参考结构（适合精确的线稿→成品）
    "moderate": 0.5,        # 默认，平衡参考和创意
    "loose_guide": 0.3,     # 松散引导（适合风格迁移但允许姿态变化）
    "minimal_hint": 0.1,    # 微弱的风格提示
}

# 在使用不同 strength 时观察效果差异
for name, strength in strength_presets.items():
    lora = LoraPathStrengthAndSDOps(
        "models/loras/ic-lora-union-control.safetensors",
        strength=strength,
        sd_ops=LTXV_LORA_COMFY_RENAMING_MAP,
    )
    # ... 运行 pipeline ...
```

### 7.4.3 Downscale Factor

IC-LoRA 支持参考视频使用较低分辨率，节省显存和加速推理：

```python
# read_lora_reference_downscale_factor() 自动从 LoRA 元数据读取
# 训练时的 downscale_factor 应和推理时一致

# 如果训练时用了 downscale_factor=2:
# 参考视频可以是 256×384，目标视频是 512×768
# IC-LoRA 自动处理分辨率差异
```

## 7.5 实战场景

### 7.5.1 线稿→成品动画

```python
# 场景: 你已经有了手绘/板绘的线稿动画（粗稿）
# 目标: 将线稿转化为精美的成品动漫

ic_lora_union = LoraPathStrengthAndSDOps(
    "models/loras/ic-lora-union-control.safetensors",
    strength=0.6,  # 稍高 = 更精确跟随线稿
    sd_ops=LTXV_LORA_COMFY_RENAMING_MAP,
)

pipeline = ICLoraPipeline(
    distilled_checkpoint_path="models/ltx-2.3-22b-distilled-1.1.safetensors",
    spatial_upsampler_path="models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors",
    gemma_root="models/gemma-3-12b",
    loras=[ic_lora_union],
)

# 线稿作为参考
lineart_frames = decode_video_by_frame("animation/lineart_draft.mp4")

video_condition = VideoConditioningAction(
    reference_frames=lineart_frames,
    image_path=None,  # 无需额外首帧
    conditioning_frame_idxs=None,
)

video, _ = pipeline(
    prompt="Professional anime production quality, rich colors, detailed "
           "character shading with soft gradients and sharp highlights, "
           "vibrant background with depth of field, Makoto Shinkai film "
           "aesthetic, 4K quality",
    negative_prompt="sketch, rough, unfinished, dull colors",
    seed=42,
    height=512, width=768, num_frames=len(lineart_frames),
    frame_rate=24.0,
    video_condition=video_condition,
)
```

### 7.5.2 3D→2D 动漫渲染

```python
# 场景: 用 3D 软件做了角色动画
# 目标: 将所有镜头统一转化为手绘动漫风格

# 使用 IC-LoRA 将 3D 渲染转化为动漫
# 3D 渲染的优势: 可以精确控制摄像机、光照、角色比例
# IC-LoRA 的优势: 一次处理将整段 3D 动画转化为统一动漫风格

ic_lora_detailer = LoraPathStrengthAndSDOps(
    "models/loras/ic-lora-detailer.safetensors",
    strength=0.5,
    sd_ops=LTXV_LORA_COMFY_RENAMING_MAP,
)

# 可叠加使用: Union Control (结构) + Detailer (细节)
pipeline = ICLoraPipeline(
    distilled_checkpoint_path="models/ltx-2.3-22b-distilled-1.1.safetensors",
    spatial_upsampler_path="models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors",
    gemma_root="models/gemma-3-12b",
    loras=[ic_lora_union, ic_lora_detailer],  # 多个 IC-LoRA 可叠加
)
```

### 7.5.3 姿态驱动的角色动画

```python
# 场景: 你有目标动作的姿态序列（如从视频中提取的 pose）
# 目标: 让动漫角色做出相同的动作

ic_lora_pose = LoraPathStrengthAndSDOps(
    "models/loras/ic-lora-pose-control.safetensors",
    strength=0.7,
    sd_ops=LTXV_LORA_COMFY_RENAMING_MAP,
)

# 参考视频: OpenPose/DWPose 提取的骨骼动画
pose_reference = decode_video_by_frame("reference/pose_sequence.mp4")

video_condition = VideoConditioningAction(
    reference_frames=pose_reference,
    image_path="character/anime_girl_standing.jpg",  # 角色参考（首帧）
    image_frame_idx=0,
)

video, _ = pipeline(
    prompt="The character performs the exact same movements as the reference, "
           "anime art style, detailed clothing with fabric physics, flowing hair",
    ...
    video_condition=video_condition,
)
```

## 7.6 IC-LoRA 训练要点

如果你想训练自己的动漫风格 IC-LoRA：

```yaml
# 训练配置关键点
training_strategy:
  name: "flexible"
  video:
    is_generated: true
    latents_dir: "latents"
    conditions:
      - type: reference
        latents_dir: "reference_latents"
        probability: 1.0

# 参考视频可以是:
# - Canny 边缘: 对目标视频运行 Canny 边缘检测
# - 深度图: 对目标视频运行深度估计
# - 姿态图: OpenPose 骨骼提取
# - 简化版: 对目标视频降采样/模糊

# 数据集结构:
# dataset.json:
# [{"video": "anime_clip_001.mp4", "reference_video": "canny_001.mp4", "caption": "..."}]
```

### 生成参考视频

```bash
# LTX-2 提供了 compute_reference.py 脚本
# 默认生成 Canny 边缘图

uv run python scripts/compute_reference.py anime_clips/ \
    --output anime_clips/dataset.json

# 预处理（含参考视频）
uv run python scripts/process_dataset.py anime_clips/dataset.json \
    --resolution-buckets "768x512x49" \
    --model-path models/ltx-2.3-22b-dev.safetensors \
    --text-encoder-path models/gemma-3-12b \
    --reference-downscale-factor 2
```

## 7.7 IC-LoRA 与普通 LoRA 的协同

最强大的用法是将两者结合：

```python
# 角色 LoRA: 确保角色外观一致
# IC-LoRA: 确保动作/结构精确
# 两者叠加: 角色正确 + 动作精确 + 风格统一

character_lora = LoraPathStrengthAndSDOps(
    "loras/my_anime_character.safetensors",
    strength=0.8,  # 角色 LoRA
    sd_ops=LTXV_LORA_COMFY_RENAMING_MAP,
)

ic_lora_union = LoraPathStrengthAndSDOps(
    "models/loras/ic-lora-union-control.safetensors",
    strength=0.5,  # IC-LoRA 结构控制
    sd_ops=LTXV_LORA_COMFY_RENAMING_MAP,
)

pipeline = ICLoraPipeline(
    distilled_checkpoint_path="models/ltx-2.3-22b-distilled-1.1.safetensors",
    spatial_upsampler_path="models/ltx-2.3-spatial-upscaler-x2-1.1.safetensors",
    gemma_root="models/gemma-3-12b",
    loras=[character_lora, ic_lora_union],  # 两者叠加！
)
```

---

**下一章：[第 8 章：LoRA 角色训练](08-LoRA角色训练.md)**
