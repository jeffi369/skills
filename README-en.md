# Agent Skills (DeepSeek Harness Skill Library)

Self-developed Agent Skills for [DeepSeek Harness (DSH)](https://github.com/deepseek-ai/deepseek-harness), organized per the [Agent Skills standard](https://agentskills.io): each skill is a self-contained folder with a `SKILL.md` (YAML frontmatter + instructions).

## Theme: Give Text Models Eyes

**image-analysis** — give DeepSeek (or any text model) a pair of eyes inside DeepSeek Harness:

> **Image → vision model converts it to text → the text model "sees"**

| Skill | Description | Docs |
|---|---|---|
| [image-analysis](skills/image-analysis/) | Image → structured text description + retrieval tags (local Ollama or OpenAI-compatible vision API) | [中文](skills/image-analysis/README-cn.md) · [English](skills/image-analysis/README-en.md) |

![architecture](skills/image-analysis/docs/architecture.png)

![benchmark (measured)](skills/image-analysis/docs/benchmark.png)

## Install into DSH

Put the skill folder into DSH's skill directory:

```text
~/.dsh/skills/image-analysis/
```

Restart the DSH session and the skill is picked up automatically: whenever the agent encounters an image (screenshot, photo, illustration, character sheet, AI-generated image...), it "opens its eyes" and injects the image description into the text model's context.

## Prerequisites (for image-analysis)

1. **Local vision model**: install [Ollama](https://ollama.com) and `ollama pull llava:13b` (or any vision model);
2. or **OpenAI-compatible vision API**: any provider + API key.
