# image-analysis-skill

## 主题：给文本模型长眼睛

文本模型（DeepSeek、各种 Agent）只能读文字，**看不到图片**——这是它们最大的能力缺口。

这个技能就是给它们**装上一双眼睛**：

> **图片 → 视觉模型转成文字 → 文本模型"看见"了**

![架构图](docs/architecture.png)

你只需要有一个能看图的模型（本地 Ollama 视觉模型，或任意 OpenAI 兼容视觉 API），这双"眼睛"就能把任意图片——截图、照片、插画、角色设定图、AI 生成图——翻译成**结构化文字描述 + 检索标签**，喂给任何文本模型去理解、推理、回答。

## 怎么用：一条命令，图片变文字

```powershell
# 本地"眼睛"（Ollama + 视觉模型，默认 llava:13b）
.\image-analysis.ps1 -ImagePath "D:\pics\photo.webp"

# 云端"眼睛"（OpenAI 兼容视觉 API，如 gpt-4o / qwen-vl）
$env:VISION_API_KEY = "sk-..."
.\image-analysis.ps1 -ImagePath "D:\pics\photo.png" -Provider openai -Model "gpt-4o-mini"
```

输出示例：

```text
这是一张角色设定图……（结构化描述）
检索标签：HARUMIN, 角色设定图, 橙黄条纹泳装, 银色创可贴, 短发女性角色, 声线描述
```

把输出粘进任何文本模型的对话，它就能基于图片内容继续推理了。

**脚本自动预处理**：任意格式（WebP 等）转 PNG、长边缩放到 1080——大图直接喂模型会又慢又糊（token 爆炸），缩放后"眼睛"读得更清、更快、更省。

## 配置（精简）

| 参数 | 默认 | 说明 |
|---|---|---|
| `-Provider` | `local` | `local`（Ollama）/ `openai`（兼容 API） |
| `-Model` | `llava:13b` | 视觉模型（这双"眼睛"） |
| `-Endpoint` | `http://127.0.0.1:11434` | 服务地址 |
| `-ApiKey` | 空 | 仅 `openai` 需要（或用 `$env:VISION_API_KEY`） |
| `-MaxEdge` | `1080` | 图片长边缩放上限（px） |

## 这双"眼睛"好不好用（5080 16GB 实测）

- 读一张图 **5~10 秒**，产出 **300~700 tokens** 文字描述；
- 本地视觉前端 `Qwen3.6-35B-IQ3_S`：**100% GPU、118 tok/s**（唯一满速选项）；
- 预处理收益：大图 token 从 **3647 降到 713**（降 80%），快 24 倍，质量无损失。

![性能对比](docs/benchmark.png)

## 适用于

- **DSH（DeepSeek Harness）**：DeepSeek AI 开源的 agent 框架（[GitHub](https://github.com/deepseek-ai/deepseek-harness)），本身是纯文本模型；把本技能放进 `~/.dsh/skills/`，代理遇到图片自动"睁眼"。
- **Claude Code / 任意 Agent**：放进对应技能目录即可。
- **任何文本 LLM API**：把输出描述直接粘进对话。

## 前提（二选一）

1. **本地视觉模型**：安装 [Ollama](https://ollama.com) 并 `ollama pull llava:13b`（或任意 vision 模型）；
2. **OpenAI 兼容视觉 API**：任意服务商 + API Key。

---

实现细节、全部参数与常见问题见 `SKILL.md` 与 `image-analysis.ps1`。
