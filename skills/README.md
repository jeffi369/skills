# Skill 层（Skills）— SuperMate Harness System

> Skill = 教"怎么做"的任务级技能（SKILL.md 结构化指令/流程清单）。
> 与插件层（提供"能做什么"的执行能力）不同，Skill 层解决"怎么做"。

## 结构

```
SuperMate Harness System
 ├── skills/   → Skill 层（任务级：技能/指令/流程清单）
 └── plugins/  → 插件层（系统级：模型/记忆/调度/工具/沙箱/评测/UI）
```

## 官方基础技能索引
[官方基础技能（deepseek-ai/deepseek-harness，只选官方）](official-dsh-skills.md)

## 技能清单（现状与规划）

| 技能 | 现状 | 说明 |
|------|------|------|
| **deepseek-eyes** | ✅ 已发布 | 视觉 skill（Ollama）：看图/资产入库/提示词管理 |
| **h3-video-producer** | ✅ 已发布 | 本地版完整视频生产（H3/Z-Image + 生成门禁） |
| **supermate** | ✅ 已发布 | 智能体身份（能力矩阵/插件调用协议/复盘模板） |
| **官方技能引用** | 📋 索引 | 见 official-dsh-skills.md，按需拷贝/复用 |

## 说明
- Skill 资产 = SKILL.md + 参考文档/脚本/模板（任务级，可独立安装/替换）。
- 官方技能 Apache 风格开源，可直接复用；SuperMate 核心增强属闭源商业层，见根目录《双许可说明.md》。
