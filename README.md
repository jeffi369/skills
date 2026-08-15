<div align="center">

# 👁️ Agent Skills — DeepSeek Harness Skill Library

**Give text models eyes.** Self-developed Agent Skills for [DeepSeek Harness (DSH)](https://github.com/deepseek-ai/deepseek-harness), organized per the [Agent Skills standard](https://agentskills.io).

**中文版**：[README-cn.md](README-cn.md) · 中文说明书

---

> **Image → vision model converts it to text → the text model "sees"**

[**image-analysis**](skills/image-analysis/) gives DeepSeek (or any text model) a pair of eyes inside DeepSeek Harness — screenshots, photos, illustrations, character sheets, AI-generated images all become structured text the model can reason about.

</div>

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

*Built for the DeepSeek Harness ecosystem · DeepSeek Harness 技能库*

</div>
