# workbuddy.md

> 本文件为「麦门省钱大师」使用 **WorkBuddy** 开发时的对话上下文导出文件，用于核验是否符合 WorkBuddy 联动活动奖励条件。
> **提交参赛前，请用 WorkBuddy 的导出功能，将本项目的实际开发对话覆盖本文件。**

## 开发过程摘要（初版提交时记录）

1. **规则调研**：用户了解「麦当劳程序员创意开发大赛」，确认核心机制——排名 = GitHub 公开仓库 Star 数；前 100 名获奖（实物周边 + 提交 `workbuddy.md` 得 3000 积分；前三名额外巨无霸券 + 10240 积分）。
2. **能力盘点**：拉取麦当劳官方 MCP Server（`mcp.mcd.cn`）全部 33 个工具，按 6 大域归类，识别「省钱/比价」为高价值组合方向（`auto-bind-coupons` + `query-meals` + `calculate-price`）。
3. **方向确认**：与用户沟通后选定「麦门省钱大师」——自然语言说预算，AI 自动领券、查真实菜单、含券比价、生成最优订单。
4. **Skill 设计**（在 WorkBuddy 内完成）：定义角色、严格 6 步工作流、结构化表格输出规范、合规约束（非官方产品、不收集 Token、个人助手定位）。
5. **仓库骨架产出**：`SKILL.md`（核心）、`README.md`、`CONTEST_DECLARATION.md`（官方原样）、`MCP_INTEGRATION.md`、`examples/sample-dialog.md`、`LICENSE`、`.gitignore`。

## 后续待补（由 WorkBuddy 实际对话导出替换本段）

- 真实联调记录：`auto-bind-coupons` 等工具在 WorkBuddy 中的实际调用与返回。
- 演示 GIF / 截图（放入 `images/`）。
- 基于真实返回的提示词调优记录。
