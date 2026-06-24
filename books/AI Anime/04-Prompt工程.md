# 第 4 章：Prompt 工程

> Prompt 是 AI 动漫创作的"剧本"。好的 prompt 是高质量动漫视频的基础。

## 4.1 LTX-2 Prompt 的核心原则

LTX-2 的文本编码器是 Gemma 3 (12B)，具备强大的多语言理解能力。其 prompt 设计与 Stable Diffusion 系列有显著不同：

### LTX-2 vs Stable Diffusion Prompt 差异

| 维度 | SD/Flux 风格 | LTX-2 风格 |
|------|-------------|-----------|
| 结构 | 标签列表（tag1, tag2, tag3...） | 自然语言段落 |
| 长度 | 短（20-80 tokens） | 中等（50-200 词） |
| 语法 | 词组/标签 | 完整句子 |
| 时序 | 不适用 | 按时间顺序描述 |
| 否定词 | 常用 | 少用（CFG 更有效） |

### LTX-2 推荐的 Prompt 结构

```
[主要动作的一句话描述]
+ [具体的动作和姿态细节]
+ [角色/物体的外观描述]
+ [背景和环境细节]
+ [摄影机角度和运动]
+ [光线和色彩描述]
+ [变化或突发事件]

保持在 200 词以内，像一个连续的自然段。
```

## 4.2 动漫 Prompt 模板

### 4.2.1 单人角色动画

```
[动作描述]。[角色名] is a [年龄/性别] anime character with [发型],
[眼睛颜色], wearing [服装风格]。[角色] [具体动作], [表情变化]。
The scene is set in [背景环境], with [光线/氛围]。
[摄影机描述: close-up/wide shot/panning/tracking]。
The art style is [动漫风格描述], with [色彩方案/特效]。

示例:
A young female anime character with long flowing silver hair and bright blue eyes
gently turns her head toward the camera, her expression shifting from contemplation
to a soft smile. She wears a traditional white and red shrine maiden outfit with
flowing sleeves that ripple in the breeze. Cherry blossom petals drift across the
scene. The camera slowly dollies in from a medium shot to a close-up, focusing on
her face. Warm golden hour sunlight filters through the sakura trees, creating soft
lens flares. Art style reminiscent of Makoto Shinkai films with vibrant colors and
atmospheric lighting.
```

### 4.2.2 动作场景

```
[快速动作描述]。The character [战斗/运动动作] with [力度/速度]。
[特效描述: impact frames/speed lines/energy effects]。
[背景变化: 快速移动/碎片/冲击波]。

示例:
An intense battle sequence unfolding at high speed. The spiky-haired protagonist
in orange gi launches a powerful energy blast from his cupped hands, the blue
energy sphere crackling with electricity and growing larger as he thrusts forward.
Speed lines streak across the frame emphasizing the explosive motion. Dust and
debris erupt from the ground in concentric rings. The camera shakes with each
energy pulse, following the action with dynamic tracking shots. High-contrast
lighting with dramatic rim lights, cel-shaded style with bold black outlines
typical of shonen anime fight scenes.
```

### 4.2.3 情感/日常场景

```
[安静/温馨的氛围描述]。The character [日常动作] with [情绪/节奏]。
[环境音暗示]，[光线变化]。[摄影机: 缓慢/固定]。

示例:
A serene slice-of-life moment in a sunlit classroom after school. A female
student with short brown hair and gentle hazel eyes sits by the window, slowly
turning the pages of a book, her finger tracing the text. Golden afternoon light
streams through the window, dust motes floating in the air. Her expression is
peaceful and absorbed. The camera remains static in a medium shot, letting the
quiet atmosphere speak. Soft pastel color palette with warm tones, watercolor-style
backgrounds blending softly. Studio Ghibli-inspired gentle animation style.
```

### 4.2.4 场景/空镜

```
[场景的详细描述]。The camera [运动方式] across [空间]。
[光线/天气/时间]。[色彩/风格]。

示例:
An establishing shot of a futuristic anime city at dusk. Towering neon-lit
skyscrapers with holographic advertisements flickering in the purple twilight sky.
Flying vehicles weave between buildings leaving light trails. The camera slowly
pans from left to right across the sprawling metropolis, revealing layers of
elevated highways and floating platforms. The city gradually lights up as day
turns to night, thousands of windows illuminating like stars. Cyberpunk aesthetic
with deep purples, electric blues, and warm orange neon accents. Clean linework
and detailed mechanical design in the style of Ghost in the Shell.
```

## 4.3 风格关键词词汇表

### 日本动漫风格

```
画风关键词:

# 经典风格
- "Makoto Shinkai style" — 新海诚（你的名字/天气之子）
- "Studio Ghibli style" — 吉卜力（千与千寻/龙猫）
- "Mamoru Hosoda style" — 细田守（穿越时空的少女）
- "Satoshi Kon style" — 今敏（千年女优/红辣椒）
- "KyoAni style" — 京都动画（日常/细腻）
- "Trigger style" — 扳机社（天元突破/赛博朋克）
- "Ufotable style" — Ufotable（鬼灭之刃/Fate）

# 技术风格
- "cel-shaded anime style" — 赛璐珞风格
- "hand-drawn animation style" — 手绘动画风格
- "90s retro anime aesthetic" — 90年代复古
- "clean linework, bold outlines" — 干净线条+粗轮廓
- "limited animation style" — 有限动画风格
- "sakuga quality animation" — 作画级质量

# 色彩方案
- "vibrant and saturated colors" — 鲜艳饱和
- "soft pastel color palette" — 柔和粉彩
- "muted earth tones" — 低调大地色
- "high-contrast with deep shadows" — 高对比深阴影
- "monochrome with color accents" — 黑白+色彩点缀

# 特效
- "speed lines and motion blur" — 速度线+动态模糊
- "impact frames with dramatic lighting" — 冲击帧效果
- "sparkle and glow effects" — 闪光特效
- "ink wash painting effect" — 水墨效果
```

### 中国动漫风格

```
# 国漫风格
- "Chinese donghua style" — 国漫风格
- "ink painting anime style" — 水墨动画风格
- "xianxia aesthetic, flowing robes" — 仙侠风/飘逸长袍
- "wuxia martial arts cinematic" — 武侠电影感

# 具体参考
- "Big Fish and Begonia style" — 大鱼海棠
- "White Snake aesthetic" — 白蛇缘起
- "Nezha dynamic action style" — 哪吒之魔童降世
- "The Legend of Hei clean style" — 罗小黑战记
```

## 4.4 摄影机语言

### 镜头运动

```
静态镜头:
- "static camera, fixed frame" — 固定机位
- "locked-off shot" — 锁定镜头
- "tripod shot, stable composition" — 三脚架稳定构图

推拉:
- "camera slowly dollies in" — 缓慢推进
- "zoom in to close-up" — 变焦至特写
- "camera pulls back to reveal" — 后拉展示全景

横摇/竖摇:
- "camera pans left to right" — 从左到右横摇
- "tilt up slowly" — 缓慢上摇
- "whip pan transition" — 快速甩镜头

跟拍:
- "tracking shot following the character" — 跟拍角色
- "steadicam smooth following" — 稳定器平滑跟拍
- "dolly zoom effect" — 希区柯克变焦
```

### 镜头景别

```
- "extreme wide shot, establishing" — 大远景/定场
- "wide shot, full body" — 全景
- "medium shot, waist up" — 中景/腰部以上
- "medium close-up, chest up" — 近景/胸部以上
- "close-up on face" — 面部特写
- "extreme close-up on eyes" — 眼睛大特写
- "over-the-shoulder shot" — 过肩镜头
- "POV shot from character's perspective" — 角色主观视角
- "low angle, looking up" — 低角度仰拍
- "high angle, looking down" — 高角度俯拍
- "dutch angle, tilted frame" — 倾斜构图
```

## 4.5 动漫动作词汇库

### 角色动作

```
头部/面部:
- "turns head slowly" — 缓缓转头
- "tilts head curiously" — 好奇歪头
- "nods gently" — 轻轻点头
- "shakes head in disbelief" — 难以置信摇头
- "hair flowing in the wind" — 发丝飘动
- "eyes widening in surprise" — 惊讶睁大眼睛
- "tears streaming down face" — 泪流满面
- "blushing and looking away" — 脸红转头

身体动作:
- "walking gracefully" — 优雅行走
- "running with determination" — 坚定奔跑
- "jumping and spinning in mid-air" — 空中跳跃旋转
- "drawing a sword in one fluid motion" — 拔刀流暢动作
- "striking a dynamic pose" — 摆出动态姿势
- "stumbling and catching balance" — 踉跄保持平衡
- "sitting down slowly" — 缓缓坐下
- "standing up with resolve" — 坚定站起

手势:
- "reaching out with an open hand" — 伸出手
- "clenching fist tightly" — 紧握拳头
- "pointing forward dramatically" — 戏剧性指向前方
- "wiping away tears" — 擦去眼泪
- "adjusting glasses" — 推眼镜
- "hair flip over shoulder" — 甩头发到肩后
```

### 环境特效

```
自然:
- "cherry blossom petals falling" — 樱花飘落
- "autumn leaves swirling" — 秋叶飞舞
- "snow gently falling" — 雪花缓缓飘落
- "rain streaking down window" — 雨水沿窗滑落
- "wind rippling through grass" — 风吹草地
- "waves crashing on rocks" — 海浪拍打岩石

光线:
- "sunlight filtering through trees" — 阳光透过树林
- "lens flare from setting sun" — 夕阳镜头光晕
- "soft rim light on character" — 角色边缘柔光
- "volumetric god rays" — 体积光/丁达尔效应
- "flickering candlelight" — 烛光摇曳
- "neon reflections on wet pavement" — 霓虹在湿路面反射

魔法/能量:
- "glowing energy aura surrounding body" — 身体周围能量光晕
- "magic circle forming beneath feet" — 脚下魔法阵形成
- "sparks and particles emanating" — 火花粒子散发
- "transformation sequence with light ribbons" — 变身光带序列
- "energy beam charging and firing" — 能量束充能发射
```

## 4.6 Prompt 增强

LTX-2 支持自动 prompt 增强（`enhance_prompt` 参数），会使用 Gemma 的系统提示模板优化你的 prompt：

```python
pipeline(
    prompt="anime girl walking in park",
    enhance_prompt=True,  # 自动扩展为基础 prompt
    ...
)
```

系统提示模板定义在：
- `ltx_core/text_encoders/gemma/encoders/prompts/gemma_t2v_system_prompt.txt`
- `ltx_core/text_encoders/gemma/encoders/prompts/gemma_i2v_system_prompt.txt`

## 4.7 动漫场景 Prompt 速查表

```
场景类型        | 关键元素                         | 推荐帧数
───────────────┼──────────────────────────────────┼──────────
角色登场        | 特写→中景, 动作揭示, 风/光效      | 49-73
对话场景        | 面部表情, 唇部动作, 平稳镜头       | 73-97
战斗高潮        | 速度线, 能量特效, 动态机位         | 97-121
变身/觉醒       | 光效包裹, 服装变化, 上升气流       | 73-97
感人告别        | 泪水, 夕阳, 慢动作, 特写          | 73-97
日常蒙太奇      | 快速剪辑感, 多场景切换描述         | 121-161
城市空镜        | 全景, 摇镜, 昼夜变化               | 97-121
片尾/ED 风      | 角色行走, 风景, 胶片质感           | 73-97
```

## 4.8 常见错误与修正

| 错误 | 问题 | 修正 |
|------|------|------|
| 标签式 prompt | "1girl, anime, sakura, wind" | 写自然句子 |
| prompt 过短 | <20 词 | 补充动作、背景、光线细节 |
| prompt 过长 | >250 词 | 精简，保留关键动作和视觉 |
| 缺乏时序 | 只描述静态画面 | 描述"从...到..."的变化过程 |
| 矛盾描述 | "static" + "running" | 检查逻辑一致性 |
| 抽象概念 | "feeling of nostalgia" | 转化为具体视觉：warm sepia tones, slow fading transitions |

---

**下一章：[第 5 章：图生视频](05-图生视频.md)**
