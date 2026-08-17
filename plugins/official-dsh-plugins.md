# 官方基本插件索引（dsh-plugin · 只选官方）

> 来源：**官方唯一** = [deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)（DSH 框架本体，`packages/` 内置核心模块）。
> 本页仅整理官方插件的名称/功能/来源，供 SuperMate Harness System 插件层对接与复用；不收录第三方。

## 官方核心插件（packages/）

| 插件 | 功能 | 官方位置 |
|------|------|----------|
| `llm` | 模型接入（云端/本地文本与多模态） | packages/llm |
| `skill` | 技能系统（SKILL.md 发现/加载/执行） | packages/skill |
| `mcp` | MCP 工具接入（外部工具协议） | packages/mcp |
| `sandbox` | 沙箱（隔离执行环境） | packages/sandbox |
| `schedule` | 调度（任务/循环调度） | packages/schedule |
| `storage` | 存储（记忆/资产持久化） | packages/storage |
| `session` | 会话管理 | packages/session |
| `workflow` | 工作流编排 | packages/workflow |
| `fs` | 文件系统访问 | packages/fs |
| `shell` | 命令执行 | packages/shell |
| `guard` | 安全守卫/权限 | packages/guard |
| `credentials` | 凭据管理 | packages/credentials |
| `goal` | 目标管理 | packages/goal |
| `subagent` | 子智能体 | packages/subagent |
| `terminal` / `web` / `ui` | 终端/网页/界面 | packages/terminal · web · interaction |

## SuperMate 插件层 ↔ 官方插件对应

| SuperMate 插件层 | 对应官方插件 |
|------------------|--------------|
| 模型适配（云端/本地主脑） | `llm` |
| 记忆 | `storage` + skill 层 Hermes 协议 |
| 调度（资源错峰） | `schedule` + `sandbox` |
| 工具（ComfyUI/ffmpeg/GitHub/Ollama） | `mcp` + `fs`/`shell` |
| 沙箱（规划中） | `sandbox` + `guard` |
| 评测（规划中） | 官方无对应 → SuperMate 自研（闭源商业层） |

## 说明
- 以上均为官方 DSH 框架内置模块（Apache 风格开源，见官方 LICENSE）。
- SuperMate 在其上做组装与增强；核心增强（调度/记忆治理/沙箱/评测实现）属闭源商业层。
