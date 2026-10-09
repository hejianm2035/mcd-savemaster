# MCP 接入说明 (MCP Integration)

## 1. 接入的 MCP Server

| 项 | 值 |
|---|---|
| 服务地址 | `https://mcp.mcd.cn` |
| 传输协议 | Streamable HTTP |
| 鉴权方式 | `Authorization: Bearer <YOUR_MCP_TOKEN>` |
| Token 申请 | https://open.mcd.cn/mcp （手机号登录 → 控制台 → 激活） |
| 限流 | 600 次 / 分钟（超出返回 429） |

**WorkBuddy 配置 JSON：**

```json
{
  "mcpServers": {
    "mcd-mcp": {
      "type": "streamablehttp",
      "url": "https://mcp.mcd.cn",
      "headers": {
        "Authorization": "Bearer YOUR_MCP_TOKEN"
      }
    }
  }
}
```

## 2. 本 Skill 使用的工具

| 工具名 | 用途 | 在工作流中的位置 |
|---|---|---|
| `auto-bind-coupons` | 一键领取所有当前可领的麦麦省优惠券 | 第 1 步 · 领券 |
| `query-my-coupons` | 查询账户下所有可用券 | 第 1 步 · 核对 |
| `query-store-coupons` | 查询当前门店可用券 | 第 1 步 · 核对 |
| `query-nearby-stores` | 查询附近可用门店（到店取餐） | 第 2 步 · 定门店 |
| `delivery-query-addresses` | 查询可配送地址 | 第 2 步 · 外送 |
| `delivery-query-stores` | 查询地址附近可配送门店 | 第 2 步 · 外送 |
| `query-meals` | 查询门店可售餐品（分类/编码/标签） | 第 3 步 · 拉菜单 |
| `query-meal-detail` | 查询套餐组成与可替换项 | 第 3 步 · 套餐展开 |
| `calculate-price` | 含券计算总价/优惠/配送费 | 第 4 步 · 比价 |
| `create-order` | 创建订单并返回支付链接 | 第 6 步 · 下单 |
| `cancel-order` | 取消订单 | 第 6 步 · 撤销 |
| `list-nutrition-foods` | 餐品营养数据（热量/蛋白等） | 健康场景补充 |
| `now-time-info` | 获取当前时间（判断活动期） | 时间相关判断 |

## 3. 调用流程

```
用户需求(预算/人数/偏好)
      ↓
[1] auto-bind-coupons → query-my-coupons / query-store-coupons   // 把券领满、摸清家底
      ↓
[2] query-nearby-stores / delivery-*                              // 锁定门店
      ↓
[3] query-meals → query-meal-detail                               // 拉真实菜单
      ↓
[4] calculate-price × N 组合(含券)                                 // 多方案含券比价
      ↓
[5] 结构化表格对比 + 推荐                                          // 告诉用户省了多少
      ↓
[6] create-order (用户确认后) → 支付链接                           // 一键成交
```

## 4. 业务价值

- **真实省钱**：所有比价均基于 `calculate-price` 的真实含券计算，而非估算，杜绝「假优惠」。
- **零门槛**：用户只需用自然语言说预算，无需在 App 里逐个翻券、比价。
- **决策透明**：多方案表格对比，用户看得见「为什么这套最划算」。
- **可延伸**：同一套「领券→查菜单→含券比价」范式可复用到营养搭配、活动雷达等场景。

## 5. 合规说明

- Token 为个人会员身份，作品定位为**个人省钱助手**，不假设服务多用户。
- 全程不收集、不存储用户凭证；配置仅使用占位符。
- 输出非官方建议，以麦当劳实时数据为准。
