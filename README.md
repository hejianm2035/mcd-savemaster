# 🍔 麦门省钱大师 · McDonald's Savings Master

> 基于 **麦当劳中国 MCP** 能力开发的 WorkBuddy 省钱点餐 Skill —— 你只管说一句预算，它把券用满、把最划算的一单端到你面前。

<p align="center">
  <b>「30 块预算，一个人，想吃点有肉的，怎么点最划算？」</b><br/>
  → 麦门省钱大师自动领券、拉真实菜单、含券比价，3 秒给你最优解。
</p>

---

## ✨ 为什么需要它

打开麦当劳 App，券一大堆，但：
- 哪张券能叠加？不知道。
- 这个套餐换掉一个单品会不会更便宜？懒得算。
- 30 块到底能凑出什么组合？眼睛看花了。

**麦门省钱大师**把这些交给 AI：一键领走所有可领券 → 拉取你门店的真实菜单 → 用 `calculate-price` 做**含券实算**比价 → 表格告诉你哪套最划算、省了多少。

## 🚀 功能特性

- 🎟️ **一键领全券**：`auto-bind-coupons` 把麦麦省可领券一次领完
- 🍽️ **真实菜单比价**：基于 `query-meals` 真实餐品与编码，不是拍脑袋
- 💰 **含券实算**：`calculate-price` 自动计入优惠与配送费，价格真实可信
- 📊 **多方案对比**：至少 2~3 套组合，表格呈现原价 / 券后价 / 省了多少
- 🛒 **一键下单**：确认后 `create-order` 直接给支付链接
- 🥗 **营养模式**（可选）：关心热量时用 `list-nutrition-foods` 标注每份能量

## 📦 安装（WorkBuddy）

1. **申请麦当劳 MCP Token**
   打开 https://open.mcd.cn/mcp → 手机号登录 → 控制台 → 激活 → 复制 Token。
2. **在 WorkBuddy 配置连接器**
   左侧【专家·技能·连接器】→【连接器】→【自定义连接器】→【配置 MCP】，粘贴：
   ```json
   {
     "mcpServers": {
       "mcd-mcp": {
         "type": "streamablehttp",
         "url": "https://mcp.mcd.cn",
         "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
       }
     }
   }
   ```
   把 `YOUR_MCP_TOKEN` 换成你的真实 Token，保存并启用。
3. **加载本 Skill**
   将本仓库的 `SKILL.md` 放入 WorkBuddy 的 Skills 目录（或按 WorkBuddy 技能规范加载），即可在对话框中直接说需求。

## 💬 使用示例

```
用户：预算 30 块，一个人，想吃点有肉的，怎么点最划算？
麦门省钱大师：
  已为你领取 5 张可用券 ✅
  锁定门店：XX 路餐厅（到店取餐）

  | 方案 | 餐品 | 原价 | 券后价 | 省了 | 点评 |
  |------|------|------|--------|------|------|
  | A | 巨无霸套餐 + 派 | ¥41 | ¥30 | ¥11 | 用了满减券 |
  | B | 麦辣鸡腿堡 + 可乐 + 薯条 | ¥38 | ¥28 | ¥10 | 单品组合更灵活 |
  | C | 双层吉士汉堡套餐 | ¥36 | ¥31 | ¥5  | 分量最足 |

  ✅ 推荐 A：刚好卡在 30 预算内，比原价省 11 块，最划算。要我直接下单吗？
```

> 更多示例见 [`examples/sample-dialog.md`](examples/sample-dialog.md)。演示 GIF 稍后补充至 `images/`。

## 🏗️ 技术架构

- **底座**：麦当劳中国官方 MCP Server（`https://mcp.mcd.cn`，Streamable HTTP + Bearer Token）
- **形态**：WorkBuddy Skill（纯 AI 编排，无需后端 / 前端 / 服务器）
- **核心工具链**：`auto-bind-coupons` → `query-meals` → `calculate-price` → `create-order`
- 详见 [`MCP_INTEGRATION.md`](MCP_INTEGRATION.md)。

## 📁 目录结构

```
mcd-savemaster/
├── SKILL.md                  # Skill 本体（核心源代码）
├── README.md                 # 本文件
├── CONTEST_DECLARATION.md    # 参赛声明（官方原样，不可改）
├── MCP_INTEGRATION.md        # MCP 接入与工具调用说明
├── workbuddy.md              # WorkBuddy 开发对话上下文（专项奖核验）
├── examples/
│   └── sample-dialog.md      # 示例对话
└── images/                   # 演示截图 / GIF
```

## ⚠️ 合规声明

- 本项目为**麦当劳程序员节创意开发大赛**参赛作品，由参赛者独立开发，**非麦当劳官方产品**。
- 输出仅供参考，价格 / 供应状态以麦当劳官方实时结果为准。
- 不收集、不存储任何用户 Token 或账号凭证；配置仅使用占位符。
- 本助手定位为**个人省钱助手**，Token 为用户本人会员身份。
- 完整声明见 [`CONTEST_DECLARATION.md`](CONTEST_DECLARATION.md)。

## ☕ 喜欢就点个 Star

如果它帮你省下了那 11 块钱，欢迎在 GitHub 上点个 ⭐ —— 你的 Star 既是对作者的鼓励，也是这场创意大赛排名的依据。感谢支持！

---

<p align="center">Made with 🍔 by 麦门省钱大师 · 麦当劳程序员节创意开发大赛参赛作品</p>
