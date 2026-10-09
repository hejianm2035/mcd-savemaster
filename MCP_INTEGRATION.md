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
[2] query-nearby-stores / delivery-*                              // 锁定门店(拿 storeCode)
      ↓
[3] query-meals → query-meal-detail                               // 拉真实菜单 + 取标准售卖码
      ↓
[4] calculate-price × N 组合(含券, couponId+couponCode 成对)        // 多方案含券比价
      ↓
[5] 结构化表格对比 + 推荐                                          // 告诉用户省了多少
      ↓
[6] create-order (用户确认后) → 支付链接                           // 一键成交(真实付款,需确认)
```

## 4. 真实联调结论（⚠️ 关键，漏掉会跑出 0 元/报错）

> 以下为 2026-10-09 用真实 Token 联调 `mcd-mcp` v1.0.0 的实证结论。

1. **接入根路径是 `https://mcp.mcd.cn`（不要加 `/mcp` 后缀）**，否则 404。协议 Streamable HTTP，初始化后无需手动维护 `Mcp-Session-Id`（无状态也可）。
2. **`query-meals` / `calculate-price` 必填三件套**：`beType`（1=到店自提 / 2=麦乐送到家 / 5=得来速 / 6=团餐）+ `orderType`（1=到店 / 2=外送）+ `storeCode`。缺任一会报 `400 缺少参数`。
3. **菜单 code ≠ 可计价售卖码（最重要）**：`query-meals` 返回的是「人气热卖 / 精选单人餐」等**分组/展示码**，直接丢进 `calculate-price` 会**静默返回 `price:0 / productList:[]`**（不报错，极易误以为成功）。必须先 `query-meal-detail` 拿到标准售卖码（如 `1440`=麦辣鸡腿汉堡）再算价。
4. **优惠券必须 `couponId` + `couponCode` 成对传**：`calculate-price` 的 items 里只传 `couponId` 会报 `600010 使用优惠券需要couponId和couponCode`。`auto-bind-coupons` 返回两者齐全，直接透传。
5. **`query-nearby-stores` 到店场景不返回 `beCode`**，可省略；仅得来速/外送需 `beCode`（从门店或 `delivery-query-stores` 返回中取）。
6. **`auto-bind-coupons` 是真实领券**：一次调用可领到当前全部可领麦麦省券（实测 9 张全成功），属免费操作；`create-order` 是**真实下单付款**，Skill 设计为「用户确认后才调用」，演示不触发真实扣款。
7. 限流 600 次/分钟，正常比价调用远不会触顶；连续高频才需降速。

## 4. 业务价值

- **真实省钱**：所有比价均基于 `calculate-price` 的真实含券计算，而非估算，杜绝「假优惠」。
- **零门槛**：用户只需用自然语言说预算，无需在 App 里逐个翻券、比价。
- **决策透明**：多方案表格对比，用户看得见「为什么这套最划算」。
- **可延伸**：同一套「领券→查菜单→含券比价」范式可复用到营养搭配、活动雷达等场景。

## 5. 合规说明

- Token 为个人会员身份，作品定位为**个人省钱助手**，不假设服务多用户。
- 全程不收集、不存储用户凭证；配置仅使用占位符。
- 输出非官方建议，以麦当劳实时数据为准。
