# image-analysis-skill

**给文本模型"长眼睛"。** 一个通用的本地图片分析技能：把任意图片变成**结构化中文描述 + 检索标签**，让纯文本 LLM 代理（[DSH](#关于-dshdeepseek-harness)、Claude Code、自定义 Agent）也能"看见"图片内容。

**通用性**：不绑定特定硬件、路径、端口或模型——任何机器开箱即用。

> **前提条件（二选一）**：用户须具备其一——
> ① **本地视觉模型**：已安装并运行 [Ollama](https://ollama.com)，且拉取过一个具备 vision 能力的模型（如 `ollama pull llava:13b`）；
> ② **OpenAI 兼容的视觉 API**：任意服务商（OpenAI GPT-4o、通义千问 qwen-vl、Gemini、vLLM/LM Studio 自建等），需要 API Key。
>
> 两种通道脚本都支持：`-Provider local`（默认）或 `-Provider openai`。全程离线免费（本地）或按 API 计费（云端）。

---

## 关于 DSH（DeepSeek Harness）

[DeepSeek Harness（`dsh`）](https://github.com/deepseek-ai/deepseek-harness) 是 [DeepSeek AI](https://deepseek.com) 开源的智能体（Agent）工具框架：**一切皆插件**（基于 [Cordis](https://github.com/cordiverse/cordis) 架构），通过 Web UI（默认 `http://127.0.0.1:3080`，`npx @deepseek-ai/dsh web` 启动）或 CLI 驱动 AI 代理完成编程、资产管理、视觉分析等任务。

本技能专为 DSH 的**技能（Skill）机制**设计：把文件夹放进 `~/.dsh/skills/` 后，DSH 代理遇到读图任务（资产入库、提示词管理、截图理解等）时，会自动按「预处理 → 本地视觉模型 → 结构化输出」执行。它同时兼容任何 Claude Code / 自定义 Agent 技能目录，也可作为独立脚本直接使用。目前 DSH 处于 developer preview，存在破坏性变更，请关注 [GitHub Discussions](https://github.com/deepseek-ai/deepseek-harness/discussions)。

---

## 它解决什么

直接用视觉模型读图有三个现实问题，本技能全部解决：

| 问题 | 现象 | 解决 |
|---|---|---|
| **格式不兼容** | WebP 等格式在 llama.cpp/Ollama 直读报错 `Failed to load image or audio file` | 统一转 **PNG**（无损） |
| **分辨率过大** | 2560×1440 直喂吃掉 **3647 tokens**、119s，还易撑爆上下文 | 长边缩放到 **1080**，token 降 80%、快 24 倍，质量无感知损失 |
| **思考模式干扰** | Qwen 系默认开思考，token 全花在推理上，正文为空 | 强制 `think: false` |

## 典型架构：给文本模型长眼睛（DeepSeek API 视觉搭档）

![image-analysis 架构图](docs/architecture.png)

**痛点**：DeepSeek API 等文本模型**没有视觉能力**，无法直接"看"图片。

**方案**：用一张本地显卡部署视觉模型当"眼睛"——图片 → 本地视觉模型转成文字描述 → 注入 DeepSeek（或其他文本 LLM）上下文，让文本模型获得视觉上下文后完成推理回答。

**部署示例（RTX 5080 16GB 实测）**：

```text
图片(任意格式) → image-analysis.ps1 → Qwen3.6-35B-IQ3_S(本地,100% GPU)
                                    → 中文描述+标签(约300-700 tokens)
                                    → DeepSeek API 文本模型 → 最终回答
```

- **视觉模型**：`Qwen3.6-35B-IQ3_S`（27B 档建议 16GB 显存；已实测 100% GPU、~115 tok/s）
- **一张图端到端成本**：预处理 + 读图约 **5~10s**，产出约 300~700 tokens 文本描述
- **注入 DeepSeek 的成本**：几百 token 的描述文本对上下文几乎无压力，DeepSeek 据此完成带图推理（识图、OCR、理解截图、资产打标等）

这样，任何纯文本模型/API 都能"看见"图片——不需要换模型，不需要云视觉 API，本地一次部署长期免费。

## 环境要求（前提条件）

| 通道 | 必备 | 说明 |
|---|---|---|
| **local（默认）** | [Ollama](https://ollama.com) 已运行 + 一个视觉模型 | `ollama pull llava:13b`（通用默认）；推荐更强模型如 qwen3-vl 系列、Qwen3.6-35B-IQ3_S 等（`ollama show <模型>` 的 Capabilities 需含 `vision`） |
| **openai** | OpenAI 兼容视觉 API + API Key | OpenAI / 通义千问 / Gemini / vLLM / LM Studio 等；Key 用 `-ApiKey` 或 `$env:VISION_API_KEY` |
| 操作系统 | Windows（WIC 内置解码）；Linux/macOS 走 ffmpeg（见下） | — |
| 内存/显存 | 本地通道需能容纳所选模型（27B 档建议 ≥16GB 显存）；API 通道无要求 | — |

## 快速开始

```powershell
# 通道 A：本地 Ollama（默认）
ollama pull llava:13b                          # 只需一次
.\image-analysis.ps1 -ImagePath "D:\pics\photo.webp"

# 通道 B：OpenAI 兼容视觉 API
$env:VISION_API_KEY = "sk-..."                 # 或 -ApiKey 参数
.\image-analysis.ps1 -ImagePath "D:\pics\photo.png" -Provider openai -Model "gpt-4o-mini"

# 批量分析一个文件夹
Get-ChildItem "D:\pics" -Include *.png,*.jpg,*.webp -Recurse | ForEach-Object {
  .\image-analysis.ps1 -ImagePath $_.FullName
}
```

输出示例（角色设定图）：

```
预处理 OK: 1280x714 -> 1080x602 -> D:\pics\photo_analysis.png
模型: llava:13b | 图片 token: 738 | 输出 token: 512
── 分析结果 ──────────────────────────────────────────
这是一张角色设定图……（结构化描述）
检索标签：HARUMIN, 角色设定图, 橙黄条纹泳装, 银色创可贴, 短发女性角色, 声线描述
```

## 参数与配置

```powershell
.\image-analysis.ps1 -ImagePath <必填> `
  [-Provider local|openai] [-Model <视觉模型>] [-Endpoint <服务地址>] [-ApiKey <Key>] `
  [-MaxEdge <长边阈值>] [-Prompt <自定义提示词>] [-MaxTokens <输出上限>] [-OutputPng <输出路径>]
```

| 参数 | 默认 | 说明 |
|---|---|---|
| `-ImagePath` | 必填 | 输入图片 |
| `-Provider` | `local` | `local`（Ollama）或 `openai`（OpenAI 兼容 API） |
| `-Model` | `llava:13b`（local）/ `gpt-4o-mini`（openai） | 视觉模型名 |
| `-Endpoint` | `http://127.0.0.1:11434`（local）/ `https://api.openai.com/v1`（openai） | 服务地址；其他兼容服务自行指定 |
| `-ApiKey` | 空 | 仅 `openai` 需要 |
| `-MaxEdge` | `1080` | 长边缩放阈值（px）；细节优先可调 `1536` |
| `-Prompt` | 内置 | 自定义分析提示词 |
| `-MaxTokens` | `700` | 输出 token 上限 |
| `-OutputPng` | 自动 | 预处理产物路径（默认 `<原名>_analysis.png`） |

环境变量覆盖（便于 DSH/CI 统一配置）：

```powershell
$env:VISION_PROVIDER = "local"            # local | openai
$env:VISION_MODEL = "qwen3-vl:7b"
$env:VISION_ENDPOINT = "http://127.0.0.1:11598"
$env:VISION_API_KEY = "sk-..."            # openai 通道
```

优先级：**命令行参数 > 环境变量 > 默认值**。

常见 OpenAI 兼容 Endpoint 示例：

| 服务商 | Endpoint | 示例模型 |
|---|---|---|
| OpenAI | `https://api.openai.com/v1` | `gpt-4o-mini` / `gpt-4o` |
| 通义千问（DashScope） | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-vl-max` / `qwen-vl-plus` |
| Gemini（OpenAI 兼容） | `https://generativelanguage.googleapis.com/v1beta/openai` | `gemini-2.0-flash` |
| 自建 vLLM / LM Studio | `http://127.0.0.1:8000/v1` 等 | 按部署配置 |

## 工作原理

1. **预处理**：WIC（Windows 内置）解码任意格式 → 输出无损 PNG；长边 > 阈值时等比缩放（默认 1080）。WIC 不可用时自动回退 `ffmpeg`。
2. **分析（双通道）**：
   - `local`：base64 图片 → Ollama `/api/chat`（`think:false` 关思考）；
   - `openai`：data URI 图片 → OpenAI 兼容 `/chat/completions`（`Authorization: Bearer <Key>`）。
3. **编码修复**：请求体以 **UTF-8 字节**发送（Windows PowerShell 5.1 的 `Invoke-RestMethod -Body <string>` 默认 ASCII 编码，中文会被替换成 `?`，导致模型误判"乱码"——这是本技能踩过并已修复的坑，脚本内已注释）。

## 跨平台说明

- **Windows**：WIC 内置，零依赖；ffmpeg 兜底（`winget install ffmpeg`）。
- **Linux/macOS**：无 WIC，直接用 ffmpeg 预处理：`ffmpeg -i <src> -vf "scale='min(1080,iw)':-2" <dst>.png`，再执行脚本的分析部分（或把脚本里的 `Convert-ImageForVision` 替换为 ffmpeg 调用）。

## 数据分析（RTX 5080 16GB 实测）

![解码速度对比](docs/benchmark.png)

**视觉模型解码速度**（同一张 1080px 图，关闭思考）：

| 模型配置 | 解码速度 | GPU 利用率 |
|---|---|---|
| **Qwen3.6-35B-IQ3_S @ 131K** | **118 tok/s** | 100% |
| qwen3.8-iq3xxs @ 16K | 71 tok/s | 100% |
| qwen3.8-iq3xxs @ 32K | 46 tok/s | 94% |
| qwen3.8-q3km @ 128K | 15 tok/s | 67% |
| qwen3.8-iq3xxs @ 128K | 9 tok/s | 76% |

> 结论：16GB 显存下，**Qwen3.6-35B-IQ3_S（100% GPU、118 tok/s）是最优视觉前端**；qwen3.8 系列须把上下文压到 16K~32K 才能接近满显存，128K 上下文是显存杀手（部分层被挤到 CPU，速度骤降）。

**预处理收益**（天宫大图 2560×1440）：

| 输入 | 图片 token | 总耗时 |
|---|---|---|
| 原图直喂 | **3647 tokens** | 119s |
| 缩放 1080 后 | **713 tokens** | 4.9s |

> token 降 80%、快 24 倍，读图质量无感知损失——1080 长边缩放是本技能的核心优化。

**端到端工作流成本**（DeepSeek 视觉搭档场景）：

| 环节 | 成本 |
|---|---|
| 预处理（转 PNG + 缩放） | < 1s（本地，零依赖） |
| 本地视觉模型读图 | 5~10s / 张（热加载 ~4s） |
| 描述文本注入 DeepSeek | 300~700 tokens，几乎可忽略 |
| 合计 | **单张图 5~10s，之后文本模型自由推理** |

## 常见问题（FAQ）

**1. 为什么长边 1080？**
视觉模型按 patch 编码图像，分辨率过高时 token 爆炸（2560 宽约 3647 tokens）、上下文被图片吃满、速度骤降；1080 是信息保留/上下文占用/速度的平衡点。需要极致细节（小字号文字、密插画）可调 1536（token 约翻倍）。

**2. 为什么必须转 PNG？**
llama.cpp/Ollama 对 WebP 解码兼容性差，实测直读报 `Failed to load image or audio file`；PNG 无损且稳定。

**3. 为什么请求体要显式 UTF-8？**
Windows PowerShell 5.1 `Invoke-RestMethod -Body <string>` 默认 ASCII 编码，中文变 `?`，模型把提示词误判为"乱码/问号"并跑偏。必须 `[System.Text.Encoding]::UTF8.GetBytes($body)` 后以字节发送（脚本已内置）。

**4. 模型很慢怎么办？**
`ollama ps` 看 PROCESSOR 列：含 CPU 百分比说明显存不够、部分层跑在 CPU。把模型的 `num_ctx` 压到 16384~32768（128K 上下文是显存杀手），或换更小的量化档（如 Q3_K_M/IQ3_XXS）。

**5. 想用别的模型？**
确认模型支持视觉（`ollama show <模型>` Capabilities 含 `vision`），`-Model` 指定即可。

**6. 脚本报"意外的标记/字符串未结束"？**
脚本内包含中文，Windows PowerShell 5.1 要求 **.ps1 以 UTF-8 带 BOM 保存**，否则中文按 GBK 解析会乱码并破坏语法。本仓库的 `image-analysis.ps1` 已带 BOM；自行编辑后请保持 BOM（PowerShell 7 无此限制）。

## 实测记录（开发者机器：RTX 5080 16GB / 32GB，Ollama 11598）

| 测试图 | 预处理 | 耗时 | 结果 |
|---|---|---|---|
| WebP 角色设定图 1280×714 | →PNG 1080×602 | 7.1s | 文字/服装/特征全读出 + 6 标签 ✅ |
| 竖屏截图 335×773 | 无需缩放 | 19.1s | 界面元素 + 提示词原文逐字读出 ✅ |
| 9.85MB 大图 5404×3040 | →1080×608 | 40.8s | 角色设定全读出 + 6 标签 ✅ |
| 天宫大图 2560×1440（直喂） | 无 | 119s / 3647 tokens | token 爆炸、慢 |
| 天宫大图（缩放后） | →1080×608 | 4.9s / 713 tokens | 质量无感知损失 ✅ |

## 目录结构

```text
image-analysis/
├── SKILL.md              技能定义（DSH / Claude Code 技能元数据 + 流程）
├── README.md             本文档
└── image-analysis.ps1    可执行脚本（参数化，跨机器通用）
```

## License

MIT
