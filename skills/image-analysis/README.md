# image-analysis-skill

本地视觉模型图片分析技能（DSH / Claude Code Skill）。对任意图片：**统一预处理（WebP 等格式转 PNG、长边缩放到 1080px）→ 调用本地 Ollama 视觉模型 → 输出结构化中文描述 + 检索标签**。全程离线、免费、无 API 费用，单张图 5~10 秒出结果。

在 **RTX 5080 16GB / 32GB 内存 / Windows** 上实测通过（Ollama 端口 11598）。

---

## 关于 DSH（DeepSeek Harness）

[DeepSeek Harness（`dsh`）](https://github.com/deepseek-ai/deepseek-harness) 是 [DeepSeek AI](https://deepseek.com) 开源的智能体（Agent）工具框架：**一切皆插件**（基于 [Cordis](https://github.com/cordiverse/cordis) 架构），通过 Web UI（默认 `http://127.0.0.1:3080`，`npx @deepseek-ai/dsh web` 启动）或 CLI 驱动 AI 代理完成编程、资产管理、视觉分析等任务。

本技能就是为 DSH 的**技能（Skill）机制**设计的：把文件夹放进 `~/.dsh/skills/` 后，DSH 代理在遇到读图任务（资产入库、提示词管理、截图理解等）时，会自动按本流程「预处理 → 调本地视觉模型 → 结构化输出」。目前 DSH 处于 developer preview，迭代很快，存在破坏性变更，请关注 [GitHub Discussions](https://github.com/deepseek-ai/deepseek-harness/discussions)。

---

## 为什么需要它

直接用视觉模型读图会遇到三个现实问题，本技能全部解决：

| 问题 | 现象 | 解决 |
|---|---|---|
| **格式不兼容** | WebP 等格式在 llama.cpp/Ollama 直读报错 `Failed to load image or audio file` | 统一转 **PNG**（无损） |
| **分辨率过大** | 2560×1440 直喂吃掉 **3647 tokens、119s**，还易撑爆上下文 | 长边缩放到 **1080**，token 降 80%、**快 24 倍**，质量无感知损失 |
| **思考模式干扰** | Qwen 系列默认开思考，token 全花在推理上，正文为空 | 强制 `think: false` |

## 特性

- 🖼️ **自动预处理**：WebP/GIF/BMP/TIFF → PNG；长边 >1080 → 等比缩放（Windows 内置 WIC 实现，零依赖，失败自动回退 ffmpeg）
- 🚀 **本地推理**：Ollama + 本地视觉模型，离线可用，无 API 费用
- 📝 **结构化输出**：内容描述 + 元素细节 + 文字全文读出 + 3-6 个检索标签
- 🛠️ **开箱即用**：单文件 PowerShell 脚本，无需安装任何 Python 包
- 🔧 **可配置**：模型、端口、缩放阈值、输出长度均可调

## 环境要求

| 项目 | 要求 | 说明 |
|---|---|---|
| 操作系统 | Windows 10/11 | WIC 编解码（系统自带） |
| Ollama | ≥ 0.8（实测 0.32.13） | 默认端口 **11598**（本机配置），可在脚本中改 |
| 视觉模型 | Qwen3.6-35B-IQ3_S（推荐）等 | 需具备 `vision` 能力（`ollama show <模型> \| grep vision`） |
| 显存 | ≥ 16GB（27B 档模型） | 大模型量化档位需能进显存 |
| PowerShell | 5.1 / 7+ | 脚本含 Windows PowerShell 5.1 兼容写法 |

## 安装

把整个 `image-analysis` 文件夹放到对应技能目录：

```text
DSH:          ~/.dsh/skills/image-analysis/
Claude Code:  ~/.claude/skills/image-analysis/
通用（任意）:  任意目录，手动执行脚本亦可
```

重启会话后技能即可被代理识别（描述中含触发词：图片分析 / 读图 / 资产入库等）。

## 使用方法

### 方式 A：作为 Skill 自动调用

代理检测到图片分析任务时自动执行：预处理 → 分析 → 输出。

### 方式 B：手动执行（复制即用）

```powershell
# 1. 预处理
Add-Type -AssemblyName PresentationCore
function Convert-ImageForVision($src, $dst) {
  $fs = [System.IO.File]::OpenRead($src)
  $dec = [System.Windows.Media.Imaging.BitmapDecoder]::Create($fs, [System.Windows.Media.Imaging.BitmapCreateOptions]::None, [System.Windows.Media.Imaging.BitmapCacheOption]::OnLoad)
  $frame = $dec.Frames[0]
  $w = [double]$frame.PixelWidth; $h = [double]$frame.PixelHeight
  $long = [Math]::Max($w, $h)
  if ($long -gt 1080) { $scale = 1080.0 / $long; $tw = [int][Math]::Round($w * $scale); $th = [int][Math]::Round($h * $scale) } else { $tw = [int]$w; $th = [int]$h }
  if ($tw -ne [int]$w -or $th -ne [int]$h) {
    $t = New-Object System.Windows.Media.Imaging.TransformedBitmap
    $t.BeginInit(); $t.Source = $frame; $t.Transform = New-Object System.Windows.Media.ScaleTransform ($tw/$w), ($th/$h); $t.EndInit()
    $frame = $t
  }
  $enc = New-Object System.Windows.Media.Imaging.PngBitmapEncoder
  $enc.Frames.Add([System.Windows.Media.Imaging.BitmapFrame]::Create($frame))
  $out = [System.IO.File]::Create($dst); $enc.Save($out); $out.Close(); $fs.Close()
  Write-Host "OK: $($frame.PixelWidth)x$($frame.PixelHeight) -> $dst"
}
Convert-ImageForVision 'D:\图片\原图.webp' 'D:\图片\处理后.png'

# 2. 分析
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$img = [Convert]::ToBase64String([System.IO.File]::ReadAllBytes('D:\图片\处理后.png'))
$prompt = '请用中文详细描述这张图片的内容和画面元素（人物/动作/文字/形状/颜色/布局）。如果图中有文字请完整读出。最后给出 3-6 个检索标签（逗号分隔）。请具体、准确。'
$body = @{ model = 'Qwen3.6-35B-IQ3_S:latest'; stream = $false; think = $false; options = @{ num_predict = 700 }; messages = @(@{ role='user'; content=$prompt; images = @($img) }) } | ConvertTo-Json -Depth 8
# 注意：必须用 UTF-8 字节发送（见 FAQ 第 2 条）
$bytes = [System.Text.Encoding]::UTF8.GetBytes($body)
$r = Invoke-RestMethod -Uri 'http://127.0.0.1:11598/api/chat' -Method Post -ContentType 'application/json; charset=utf-8' -Body $bytes -TimeoutSec 600
$r.message.content
```

## 配置项

| 参数 | 默认 | 说明 |
|---|---|---|
| 缩放长边阈值 | 1080px | 需要极致细节（小字号文字/密插画）可调 1536 |
| 模型 | `Qwen3.6-35B-IQ3_S:latest` | 换模型需确认其 `vision` 能力 |
| Ollama 端口 | 11598 | 非默认 11434，按本机实际修改 |
| `think` | `false` | 必须关思考，否则正文为空 |
| `num_predict` | 700 | 输出上限，长图描述可调大 |

## 实测结果（本机 5080 16GB）

| 测试图 | 预处理 | 耗时 | 结果 |
|---|---|---|---|
| WebP 角色设定图 1280×714 | →PNG 1080×602 | 7.1s | 文字/服装/特征全读出 + 6 标签 ✅ |
| 竖屏截图 335×773 | 无需缩放 | 19.1s | 界面元素 + 提示词原文逐字读出 ✅ |
| 9.85MB 大图 5404×3040 | →1080×608 | 40.8s | 角色设定全读出 + 6 标签 ✅ |
| 天宫大图 2560×1440（原图直喂） | 无 | 119s / 3647 tokens | token 爆炸、慢 |
| 天宫大图（缩放后） | →1080×608 | 4.9s / 713 tokens | 质量无感知损失 ✅ |

## FAQ

**1. 为什么必须转 PNG？**
llama.cpp/Ollama 对 WebP 解码兼容性差，实测直读报 `Failed to load image or audio file`；PNG 无损且稳定。

**2. 为什么中文请求体要显式 UTF-8 编码？**
Windows PowerShell 5.1 的 `Invoke-RestMethod -Body <string>` 默认按 ASCII 编码，中文会变成 `?`，模型会把提示词误判为"乱码/问号"并跑偏。必须 `[System.Text.Encoding]::UTF8.GetBytes($body)` 后以字节发送。

**3. 为什么长边 1080？**
视觉模型按 patch 编码图像，分辨率过高时 token 数爆炸（2560 宽约 3647 tokens）、上下文被图片吃满、速度骤降；1080 是"信息保留 / 上下文占用 / 速度"三者的平衡点，实测缩放后读图质量无感知损失。

**4. 模型很慢怎么办？**
先看 `ollama ps` 的 PROCESSOR 列。若含 CPU 百分比，说明显存不够、部分层跑在 CPU：把模型的 `num_ctx` 压到 16384~32768（128K 上下文是显存杀手，16GB 显存下会导致 ~24% 层进 CPU、速度掉到 ~9 tok/s）。

**5. 想用别的模型？**
确认模型支持视觉（`ollama show <模型>` 的 Capabilities 含 `vision`），并调整 `model` 字段即可；建议显存能完整容纳的量化档位。

## License

MIT
