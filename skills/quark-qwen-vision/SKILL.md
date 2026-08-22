---
name: quark-qwen-vision
description: 夸克浏览器专用视觉 skill——通过 CDP 网页操作调用夸克内置千问（qwen-vl）看图。当需要分析图片内容（识图/描述/OCR/反推提示词/图像对比）且本机已装夸克浏览器时使用。原理：夸克以调试模式（--remote-debugging-port=9222）运行 → CDP 控制千问对话页（p.quark.cn/pcquark-chat/sidebar）→ 模拟粘贴图片 + 提问 → 读取回复。零依赖：无需 Ollama、无需 API Key。前置条件：夸克浏览器正在调试模式运行且千问对话页已打开。
user-invocable: true
---

# quark-qwen-vision — 夸克浏览器专用视觉 Skill

> 用**夸克浏览器内置的千问（qwen-vl）模型**看图，全程网页操作，零本地算力、零 API Key。
> 本机（笔记本）视觉方案：`deepseek-eyes` 的替代通道（无 Ollama 时走这个）。

## 前置条件（使用前必须确认）

1. **夸克浏览器以调试模式运行**：
   ```powershell
   # 若夸克未运行：
   Start-Process 'C:\Users\82604\AppData\Local\Programs\Quark\quark.exe' -ArgumentList '--remote-debugging-port=9222'
   # 若已运行：先关闭全部 quark 进程，再以上述方式启动
   ```
2. **千问对话页已打开**：访问 `https://p.quark.cn/pcquark-chat/sidebar`（或点夸克 AI 侧边栏）。
   - 验证：`http://127.0.0.1:9222/json/list` 返回 200 且列表中存在 `pcquark-chat` 页面。

## 调用方式

```powershell
node "<skill 目录>\scripts\qwen-vision.js" <图片路径> ["问题"]
```

示例：
```powershell
node "D:\AI\Dsh_Data\skills\quark-qwen-vision\scripts\qwen-vision.js" "D:\Download\包1.png" "这张图是什么？反推提示词"
```

## 工作原理（网页操作四步）

```
1. CDP 连接夸克千问对话页（p.quark.cn/pcquark-chat/sidebar）
2. 读取本地图片 → base64 → File → 构造 ClipboardEvent paste 模拟粘贴
3. 输入问题 → 点击 .submit-button 发送
4. 轮询页面文本直至回复稳定 → 输出千问回答
```

## 关键经验（踩坑记录）

- **launcher 页不是对话页**：`chrome://ai-desktop-plugin/launcher/` 只是快速输入入口；真正的对话页是 `https://p.quark.cn/pcquark-chat/sidebar`，回复渲染在这里
- **发送按钮**：`.submit-button`（输入文字后激活为 `.submit-button.active`）；模拟 Enter 键无效，必须点击按钮
- **图片上传**：点 addButton 走 OS 原生文件对话框（CDP 无法操作）；正确姿势是**模拟 paste 事件**（ClipboardEvent + DataTransfer + File）
- **读取回复**：轮询 `document.body.innerText`，连续 3-4 轮无变化视为回复完成
- **回复可能很长**：输出截断至 8000 字符，必要时分段读

## 故障排查

| 现象 | 处理 |
|---|---|
| 脚本报"未找到千问对话页" | 夸克未带调试端口启动，或侧边栏未打开 → 按前置条件处理 |
| 发送后无回复 | 等待更久（流式输出）；检查千问页面是否有错误提示 |
| 图片没挂上 | 确认文件存在、格式为 png/jpg/webp；大图建议先压缩到 1080px 内 |

## 关联

- 工具脚本目录：`D:\AI\Harness\quark-cdp\`（cdp.js 客户端 / send-chat.js 文字对话 / qwen-vision.js 看图 / 诊断脚本）
- 本机文字视觉备选：`deepseek-eyes` skill 的 openai 通道（需 DashScope Key，未配置）
