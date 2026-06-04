# Agent 安全与对齐

本章聚焦 Agent 系统的安全性设计，涵盖威胁建模、权限控制、对齐策略、注入防御等关键议题。随着 Agent 自主性的提升，安全问题变得愈发重要，需要在系统设计阶段就纳入全面的安全考量。

## 本章内容

- [威胁模型](./threat-model.md) — Agent 系统面临的主要安全威胁分析
- [权限控制与沙箱](./permission-control.md) — 限制 Agent 行为边界的机制设计
- [对齐策略](./alignment-strategies.md) — 确保 Agent 行为符合人类意图的方法
- [Prompt 注入防御](./prompt-injection.md) — 防御恶意 Prompt 注入攻击
- [输出验证与护栏](./output-validation.md) — Agent 输出的安全校验机制
- [审计与日志](./audit-and-logging.md) — Agent 行为的可追溯性设计
