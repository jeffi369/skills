<div align="center">

# 👁️ 给 DeepSeek 装眼睛 · Give DeepSeek Eyes

**给 DeepSeek 装上一双"眼睛"的 Agent Skill —— 让 DSH 里的文本模型看得见图片。**

Give text models eyes — self-developed Agent Skills for [DeepSeek Harness (DSH)](https://github.com/deepseek-ai/deepseek-harness), organized per the [Agent Skills standard](https://agentskills.io).

**中文版**：[README-cn.md](README-cn.md) · 中文说明书

**DSH 生态关联** · [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) · topic [`dsh-plugin`](https://github.com/topics/dsh-plugin) · [Agent Skills](https://agentskills.io)

---

> **Image → vision model converts it to text → the text model "sees"**

[**image-analysis**](skills/image-analysis/) gives DeepSeek (or any text model) a pair of eyes inside DeepSeek Harness — screenshots, photos, illustrations, character sheets, AI-generated images all become structured text the model can reason about.

</div>

---

## ⚙️ How it works

The Skill calls a **local vision model (Ollama)** or an **OpenAI-compatible vision API** to convert images and graphic files — screenshots, photos, charts, design drafts, illustrations, character sheets, AI-generated images — into structured text. DeepSeek then reads that text, which is exactly how it gains the ability to "read" images and graphic files.

```text
Image / graphic file → Skill → local vision model or vision API → structured text → DeepSeek reads & reasons
```

---

## ✨ The Skill

| Skill | What it does | Manuals |
|---|---|---|
| [**image-analysis**](skills/image-analysis/) | Image → structured Chinese description + retrieval tags. Local Ollama or OpenAI-compatible vision API. | [中文](skills/image-analysis/README-cn.md) · [English](skills/image-analysis/README-en.md) |

### Architecture

![Architecture](skills/image-analysis/docs/architecture.png)

### Measured performance (RTX 5080 16GB)

![Benchmark](skills/image-analysis/docs/benchmark.png)

---

## 🔗 DeepSeek Harness Ecosystem

This repository is part of the **DeepSeek Harness plugin ecosystem** — find it under the [`dsh-plugin`](https://github.com/topics/dsh-plugin) topic, alongside [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) itself. Drop a skill folder into `~/.dsh/skills/`, and the DSH agent automatically "opens its eyes" whenever it meets an image, injecting the image description into the text model's context.

## 🚀 Install into DSH

Put the skill folder into DSH's skill directory:

```text
~/.dsh/skills/image-analysis/
```

Restart the DSH session — the agent automatically "opens its eyes" when it meets an image, and injects the image description into the text model's context.

## ✅ Prerequisites (for image-analysis)

1. **Local vision model**: [Ollama](https://ollama.com) + `ollama pull llava:13b` (or any vision model)
2. or **OpenAI-compatible vision API** + API key

---

<div align="center">

*给 DeepSeek 装眼睛 · Built for the DeepSeek Harness (DSH) ecosystem — topic [`dsh-plugin`](https://github.com/topics/dsh-plugin)*

</div>
