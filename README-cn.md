# Agent Skills（DeepSeek Harness 技能库）

为 [DeepSeek Harness（DSH）](https://github.com/deepseek-ai/deepseek-harness) 自研的 Agent Skills 技能库，按 [Agent Skills 标准](https://agentskills.io) 组织：每个技能是一个独立文件夹，内含 `SKILL.md`（YAML frontmatter + 指令）。

## 主题：给文本模型长眼睛

**image-analysis** —— 在 DeepSeek Harness 里给 DeepSeek（或任意文本模型）装上一双"眼睛"：

> **图片 → 视觉模型转成文字 → 文本模型"看见"**

| 技能 | 说明 | 文档 |
|---|---|---|
| [image-analysis](skills/image-analysis/) | 图片 → 结构化文字描述 + 检索标签（本地 Ollama 或 OpenAI 兼容视觉 API） | [中文](skills/image-analysis/README-cn.md) · [English](skills/image-analysis/README-en.md) |

![架构图](skills/image-analysis/docs/architecture.png)

![性能对比（实测数据）](skills/image-analysis/docs/benchmark.png)

## 安装到 DSH

把技能文件夹放进 DSH 技能目录：

```text
~/.dsh/skills/image-analysis/
```

重启 DSH 会话，技能即被代理自动识别：遇到图片（截图、照片、插画、设定图、AI 生成图……）自动"睁眼"，把图片描述注入文本模型上下文。

## 前提（使用 image-analysis 时）

1. **本地视觉模型**：安装 [Ollama](https://ollama.com) 并 `ollama pull llava:13b`（或任意 vision 模型）；
2. 或 **OpenAI 兼容视觉 API**：任意服务商 + API Key。
