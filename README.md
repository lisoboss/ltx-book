# LTX-2 技术指南

基于 **[LTX-2](https://github.com/Lightricks/LTX-2)** 开源音视频生成模型的系列技术电子书。涵盖 AI 动漫创作、视频生成、模型训练等主题，从入门到进阶。

📖 **在线阅读**: [lisoboss.github.io/ltx-book](https://lisoboss.github.io/ltx-book/)

---

## 系列书目

### AI 动漫

从零开始的 AI 动漫创作完整教程：

| 章节 | 内容 |
|------|------|
| 前言 | 项目概述，为什么选择 LTX-2 做 AI 动漫 |
| 第 1 章 | AI 动漫概述：从静态到动态的技术路线 |
| 第 2 章 | 环境搭建：Python 3.12+, CUDA 12.4+, uv, 模型下载 |
| 第 3 章 | LTX-2 架构深度解析：DiT, VAE, prompt encoder, scheduler |
| 第 4 章 | Prompt 工程：动漫提示词的系统方法 |
| 第 5 章 | 图生视频：让动漫角色动起来 |
| 第 6 章 | 关键帧插值：流畅的动作生成 |
| 第 7 章 | IC-LoRA 风格控制：统一动漫风格 |
| 第 8 章 | LoRA 角色训练：打造专属角色 |
| 第 9 章 | 视频扩展、修复与 LipDub 配音 |
| 第 10 章 | 完整 AI 动漫制作工作流 |
| 第 11 章 | 高级技巧与性能优化 |
| 第 12 章 | 附录：命令速查、FAQ 与资源索引 |

---

## 技术栈

- **内容**: Markdown 书写
- **发布**: GitHub Pages + Jekyll + minima 主题
- **字体**: Noto Sans SC (正文), Noto Sans Mono (代码)
- **CI/CD**: GitHub Actions

## 本地预览

```bash
bundle install
bundle exec jekyll serve
```

## 许可

[LICENSE](LICENSE)
