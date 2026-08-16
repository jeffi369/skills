<div align="center">

# 👁️ 给 DeepSeek 装眼睛 · DeepSeek Harness 技能库

**给文本模型长眼睛。** 为 [DeepSeek Harness（DSH）](https://github.com/deepseek-ai/deepseek-harness) 自研的 Agent Skills，按 [Agent Skills 标准](https://agentskills.io) 组织。

**English**：[README.md](README.md) · English README

---

> **图片 → 视觉模型转成文字 → 文本模型"看见"**

[**image-analysis**](skills/image-analysis/) 在 DeepSeek Harness 里给 DeepSeek（或任意文本模型）装上一双"眼睛"——截图、照片、插画、角色设定图、AI 生成图，统统变成模型能理解的结构化文字。

</div>

---

## ✨ 技能清单

| 技能 | 功能 | 文档 |
|---|---|---|
| [**image-analysis**](skills/image-analysis/) | 图片 → 结构化中文描述 + 检索标签（本地 Ollama 或 OpenAI 兼容视觉 API） | [中文](skills/image-analysis/README-cn.md) · [English](skills/image-analysis/README-en.md) |

### 架构

![架构图](skills/image-analysis/docs/architecture.png)

### 实测数据（RTX 5080 16GB）

![性能对比](skills/image-analysis/docs/benchmark.png)

---

## 🚀 安装到 DSH

把技能文件夹放进 DSH 技能目录：

```text
~/.dsh/skills/image-analysis/
```

重启 DSH 会话——代理遇到图片会自动"睁眼"，把图片描述注入文本模型的上下文。

## ✅ 前提（使用 image-analysis 时）

1. **本地视觉模型**：[Ollama](https://ollama.com) + `ollama pull llava:13b`（或任意 vision 模型）
2. 或 **OpenAI 兼容视觉 API** + API Key

---

<div align="center">

*为 DeepSeek Harness 生态而建 · DeepSeek Harness Skill Library*

</div>
