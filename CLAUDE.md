# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ltx-book** is a Chinese-language technical guidebook for AI anime (动漫) creation using **LTX-2**, the open-source DiT-based audio-video foundation model from Lightricks. The book covers the full pipeline from environment setup to production-quality anime generation, including character training, style control, keyframe interpolation, and audio-video synchronization.

## Repository Structure

```
ltx-book/
├── _chapters/                 # Jekyll collection: chapter files with frontmatter
├── _layouts/                  # Jekyll layouts (default, chapter, home)
├── _includes/                 # Jekyll includes (head, header, footer, chapter-nav)
├── assets/css/                # SCSS styles (CJK typography customizations)
├── books/AI Anime/            # Original source chapters (excluded from Jekyll build)
├── .github/workflows/         # GitHub Actions Pages deployment
├── LTX-2/                     # Git submodule: Lightricks/LTX-2 official repo
│   ├── packages/
│   │   ├── ltx-core/          # Core model (transformer, VAE, text encoder, scheduler)
│   │   ├── ltx-pipelines/     # Inference pipelines (10 pipeline variants)
│   │   └── ltx-trainer/       # Training toolkit (LoRA, full fine-tuning, IC-LoRA)
│   └── .claude/skills/
│       └── train-model/       # Skill: end-to-end LTX-2 model training
├── _config.yml                # Jekyll site configuration
├── Gemfile                    # Ruby dependencies (github-pages gem)
├── index.md                   # Home page / table of contents
└── .gitignore                 # Standard Jekyll/Pages gitignore
```

## Book Structure

The `books/AI Anime/` chapters form a progressive learning path:

| Chapter | Topic | Role in Pipeline |
|---------|-------|------------------|
| 00 | Foreword | Project overview, why LTX-2 for anime |
| 01 | AI Anime Overview | Technical roadmap: static → dynamic |
| 02 | Environment Setup | Python 3.12+, CUDA 12.4+, uv, model downloads |
| 03 | LTX-2 Architecture | DiT transformer, VAE, prompt encoder, scheduler |
| 04 | Prompt Engineering | Chronological action descriptions, camera/cinematography language |
| 05 | Image-to-Video | TI2Vid pipeline workflow, first-frame conditioning |
| 06 | Keyframe Interpolation | Smooth motion generation between keyframes |
| 07 | IC-LoRA Style Control | Unified anime style via in-context LoRA |
| 08 | LoRA Character Training | Custom character fine-tuning |
| 09 | Video Extension & Repair | Retake, LipDub, video extension |
| 10 | Complete Workflow | End-to-end pipeline: pre-production → post-production |
| 11 | Advanced Tips | Performance optimization, quality tuning |
| 12 | Appendix | Command reference, FAQ, resource index |

## Working with the Book Content

The book chapters are standalone markdown files covering both conceptual explanations and practical CLI/code examples. Chapters reference specific LTX-2 pipelines and commands. When editing book content:

- All chapters use Chinese with English technical terms preserved inline
- Code blocks use `bash`, `python`, and `yaml` language tags
- Pipeline names (e.g., `TI2VidTwoStagesPipeline`, `ICLoraPipeline`) should match the LTX-2 submodule exactly
- Command examples should reference paths relative to the LTX-2 repo root (e.g., `uv run python scripts/train.py configs/t2v_lora.yaml`)

## GitHub Pages

The site is built with **Jekyll + minima theme** and deployed via GitHub Actions from `main`. Chapter content is authored in `books/AI Anime/` (original source) and replicated to `_chapters/` (Jekyll collection with frontmatter). When editing chapters, update both locations.

## LTX-2 Submodule

The submodule at `LTX-2` tracks the official Lightricks repository. Key submodule facts:

- **Python version**: 3.12+ managed with `uv`
- **Setup**: `cd LTX-2 && uv sync --frozen && source .venv/bin/activate`
- **Linting (ltx-trainer)**: `cd LTX-2/packages/ltx-trainer && uv run ruff check .`
- **Tests (ltx-trainer)**: `cd LTX-2/packages/ltx-trainer && uv run pytest`
- **Training**: `cd LTX-2/packages/ltx-trainer && uv run python scripts/train.py configs/t2v_lora.yaml`
- **Submodule update**: `git submodule update --remote LTX-2`

### Pipeline Quick Reference

The LTX-2 submodule provides these inference pipelines (all in `packages/ltx-pipelines/src/ltx_pipelines/`):

- **`TI2VidTwoStagesPipeline`** — Production-quality text/image-to-video (recommended default)
- **`TI2VidTwoStagesHQPipeline`** — Same but with Res2s second-order sampler (fewer steps, better quality)
- **`TI2VidOneStagePipeline`** — Single-stage for quick prototyping
- **`DistilledPipeline`** — Fastest (8-step distilled model)
- **`ICLoraPipeline`** — Video-to-video / image-to-video transformations
- **`KeyframeInterpolationPipeline`** — Interpolate between keyframe images
- **`RetakePipeline`** — Regenerate specific time regions of existing video
- **`LipDubPipeline`** — Lip dubbing with speaker identity matching
- **`A2VidPipelineTwoStage`** — Audio-to-video generation
- **`HDRICLoraPipeline`** — Video-to-video with HDR output

### Model Requirements

- **GPU**: 24GB+ VRAM for inference, 80GB+ for training (32GB possible with low-VRAM config)
- **CUDA**: 12.4+
- **OS**: Linux required for training (Triton dependency); inference works on Windows/WSL
- **Checkpoints**: Download from [Lightricks/LTX-2.3 on HuggingFace](https://huggingface.co/Lightricks/LTX-2.3)

### Frame & Resolution Constraints

LTX-2 latents have strict constraints: frames must satisfy `frames % 8 == 1` (valid: 1, 9, 17, 25, 33, 41...); width and height must be divisible by 32.

## Train-Model Skill

The repository includes a skill at `LTX-2/.claude/skills/train-model/` for end-to-end LTX-2 model training. It handles dataset probing, mode selection, preprocessing, training launch, monitoring, and post-train validation. Reference phases are in `phases/` and supporting docs in `references/`. Use this skill when asked to train, fine-tune, LoRA, or produce custom LTX-2 models.

## When Book Content References Submodule Code

When updating book chapters with code examples or pipeline references, verify against the actual submodule code. Pipeline files are at `LTX-2/packages/ltx-pipelines/src/ltx_pipelines/*.py`. Training configs are at `LTX-2/packages/ltx-trainer/configs/*.yaml`. The ltx-trainer CLAUDE.md (`LTX-2/packages/ltx-trainer/AGENTS.md`) is the authoritative reference for training internals.
