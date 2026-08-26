# code-design —— 职责纯粹的 Java 后端类设计 Skill

一个 [Claude Code](https://claude.com/claude-code) skill：在 AI 写或改 Java/Spring 业务代码**之前**介入，用一套「判断框架」约束类职责划分，防止两类高频问题——

1. **被存量差代码带偏**：AI 天然模仿仓库里眼前的代码。对照实验实测：无本 skill 时，面对过程式 setter 灌值的旧代码，AI 100% 顺着写；加载本 skill 后，AI 能主动甄别出差先例并改走职责内聚的写法
2. **类职责混杂**：配置判断漏进业务 Service、适配器里写业务逻辑、组装逻辑摊成过程式脚本、同源参数拆散成长参数列表

## 核心思想（一句话）

> **一个类只回答一个问题；依赖方向单向；每层只见能力、不见细节。**

这是单一职责原则（SRP）的可操作表述——不数「修改原因」，数「回答的问题」。

## 内容概览

- **总纲**：SRP 判断框架
- **第 0 条**：先找活例子再动手（含差先例甄别——这是实测最关键的一条）
- **分层职责表**：DO 贫血 / DTO 轻量充血（`Resp.of()` 静态工厂）/ Service 取数+编排 / Adapter 纯翻译 / 外部 Client 能力封装（`isEnabled()`）/ Registry 发现分发
- **一个自包含完整例子**：订单列表关联组装（充血 + Optional 链 + 防 N+1/空 IN）
- **模式选型表**：策略+模板+工厂 / 适配器+注册器 / Spring 事件 / 条件装配等常见场景
- **Red Flags 清单**：6 个出现即回头改的信号

## 安装

```bash
# 全局（所有项目生效）
git clone https://github.com/MikerPa/claude-skills.git && cp -r claude-skills/code-design ~/.claude/skills/

# 或单项目生效
cp -r claude-skills/code-design <你的项目>/.claude/skills/
```

安装后无需任何配置——当你在 Claude Code 里写/改 Service、DTO、适配器、外部客户端，或做组装转换、接流程引擎/第三方系统时，skill 会按描述自动触发加载。

也适用于其他支持 [AgentSkills 规范](https://agentskills.io/specification) 的 AI 编码工具。

## 适用范围

- Java 17+ / Spring Boot 3.x，Controller-Service-Mapper（或类似）分层架构的中后台项目
- 不限于特定脚手架；例子使用 Hutool + Lombok，可按项目技术栈等价替换

## 验证方式

本 skill 按 TDD 流程开发：先用 subagent 跑无 skill 基线（合成场景、隔离场景、差先例诱导场景），记录真实失败模式，再针对失败写规则，最后对照实验验证规则生效（A/B 组差异见上）。规则集刻意保持最小——不教「怎么写好代码」（模型本身会），只锚定三类真实失败：差先例诱导、跳过找先例直接造轮子、项目专属职责边界。

## License

MIT
