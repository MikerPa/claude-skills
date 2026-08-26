# claude-skills

个人的 [Claude Code](https://claude.com/claude-code) skill 集合。按 [AgentSkills 规范](https://agentskills.io/specification) 编写，也可用于其他支持该规范的 AI 编码工具。

## Skills

| Skill | 简介 |
|-------|------|
| [code-design](./code-design/) | 职责纯粹的 Java 后端类设计：单一职责（SRP）判断框架、贫血/充血分层、设计模式选型、存量代码对齐纪律。实测防止 AI 被仓库里的差先例带偏 |

## 使用方式

按单个 skill 的 README 安装到 `~/.claude/skills/`（全局生效）或项目的 `.claude/skills/`（单项目生效），无需其他配置，Claude Code 会在匹配的场景自动触发加载。

## 开发方式

每个 skill 按 TDD 流程开发：先用 subagent 跑无 skill 基线记录真实失败模式，再针对失败写最小规则集，最后对照实验验证生效。规则刻意保持最小——只锚定实测会失败的点，不重复模型已具备的能力。

## License

MIT
