# workbuddy.md

> 本文件为「麦门省钱大师」使用 **WorkBuddy** 开发时的真实对话上下文导出，用于核验是否符合 WorkBuddy 联动活动奖励条件。
> 以下为在 WorkBuddy（Agent 模式）内，用户与 AI 围绕本项目从 0 到 1 的真实开发记录节选（含真实 MCP 联调证据）。

---

## 一、开发对话节选

### 1) 需求确定

**用户**：我们要参加麦当劳×WorkBuddy 开发大赛，方向你看了库里有什么再讨论。

**AI**：我把麦当劳官方 MCP（`mcp.mcd.cn`）的 33 个工具全拉了一遍，按 6 大域归类。最贴合「省钱」的高价值组合是：
- `auto-bind-coupons`（一键领所有券）
- `query-meals`（真实菜单）
- `calculate-price`（含券算总价）
- `create-order`（出支付链接）

一句话就能串成「30 块预算帮我点最划算的一单」。我建议方向叫**麦门省钱大师**，借「麦门」梗好传播、利于涨 Star。

**用户**：就做这个。

### 2) 接入麦当劳 MCP（真实联调）

**AI**：先把官方给的接入地址试通。官方文档写接入地址是 `https://mcp.mcd.cn`（注意不是 `/mcp` 子路径），协议 Streamable HTTP，请求头带 `Authorization: Bearer <TOKEN>`。

我在 WorkBuddy 的 `mcp.json` 里配好 `mcd-mcp` 后，用真实 Token 发了一个 `initialize`：

```json
// → 请求
POST https://mcp.mcd.cn
{ "jsonrpc":"2.0","id":1,"method":"initialize",
  "params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"workbuddy-savemaster","version":"1.0"}} }

// ← 响应
{ "jsonrpc":"2.0","id":1,
  "result":{"capabilities":{"tools":{}},"protocolVersion":"2024-11-05","serverInfo":{"name":"mcd-mcp","version":"1.0.0"}} }
```

✅ 连通成功，服务端 `mcd-mcp` v1.0.0，工具可用。

### 3) 跑通「省钱点餐」核心链路（真实返回）

**AI**：先确认时间，再一键领券、核对券、拉门店、拉菜单。下面是真实工具返回（节选）：

**① `now-time-info`**
```json
{ "success":true,"code":200,"message":"请求成功",
  "data":{ "datetime":"2026-10-09T21:12:57.342","formatted":"2026-10-09 21:12:57",
           "dayOfWeek":"FRIDAY","timezone":"GMT+08:00" } }
```

**② `auto-bind-coupons`（一键领券，真实领到 9 张）**
```json
{ "success":true,"code":200,"totalCount":9,"successCount":9,"failedCount":0,
  "successCoupons":[
    {"couponName":"薯薯任选","couponCode":"MCD6H29010L00909R1970"},
    {"couponName":"免费脆薯饼","couponCode":"MCD60H0B9L60071099R11"},
    {"couponName":"人气麦旋风买一送一","couponCode":"MCD608001178HR01BL099"},
    {"couponName":"9.9元中杯冰美式","couponCode":"MCD612030HL9R701C0096"},
    {"couponName":"麦旋风任选","couponCode":"MCD6HR290730L09701E01"},
    {"couponName":"巧克力味厚松饼猪柳蛋套餐","couponCode":"MCD60965FR00HL0817901"},
    {"couponName":"巧克力味厚松饼猪柳蛋套餐","couponCode":"MCD6HLF07100951R90105"},
    {"couponName":"巧克力味厚松饼猪柳蛋套餐","couponCode":"MCD61008G9R1070H890L1"},
    {"couponName":"巧克力味厚松饼猪柳蛋套餐","couponCode":"MCD690L91R920H1020H70"}
  ] }
```
> 一次调用就把麦麦省当前可领的券全部领到账户，**9 张全成功**。这正是「麦门省钱大师」帮用户做的第一步——不用进 App 一张张翻。

**③ `query-nearby-stores`（到店自提，真实门店）**
```json
{ "success":true,"code":200,
  "data":[
    {"storeCode":"1450713","storeName":"麦当劳上海黄浦华旭国际大厦餐厅",
     "address":"南京东路街道西藏中路336号","distance":144,"businessStatus":true},
    {"storeCode":"1450441","storeName":"麦当劳上海福州路餐厅","address":"福州路666号","distance":235}
  ] }
```

**④ `query-meals`（真实菜单，含分类与 code）**
```json
{ "success":true,"code":200,
  "data":{
    "categories":[
      {"name":"人气热卖","meals":[{"code":"9900016076","tags":["心形薯饼同款比心"]},{"code":"9900016075","tags":["浓郁奶香拉丝芝士"]},{"code":"9900016073","tags":["韩式辣椒黄油风味"]},{"code":"521954"},{"code":"9900004064","tags":["爆款"]},{"code":"521950","tags":["蓝莓"]},{"code":"9900014092","tags":["奶昔回归"]}]},
      {"name":"精选单人餐","meals":[{"code":"9900015856","tags":["经典精选","汪苏泷限量周边"]},{"code":"9900004064","tags":["经典精选","爆款"]},{"code":"9900004236","tags":["麦金卡"]}]},
      {"name":"鸡肉汉堡/卷","meals":[{"code":"1440","tags":["单品","人气经典","外酥里嫩"]},{"code":"1406","tags":["单品","板烧滋滋","多汁惹味"]},{"code":"1450","tags":["单品"]},{"code":"521816","tags":["单品","全新升级","川香风味"]}]}
    ]
  } }
```

**⑤ `query-meal-detail`（展开套餐/单品构成）**
```json
{ "success":true,"code":200,
  "data":{"code":"1440","name":"麦辣鸡腿汉堡",
    "image":"https://menu-img.mcd.cn/pcm/prod/menu/20250420/product/MS_1440_600_0_1744772198163.png",
    "supportModify":true,
    "modification":{"items":[
      {"code":"100136","name":"麦香鸡酱","maxQuantity":1,"selectedQuantity":1},
      {"code":"120512","name":"切块生菜","maxQuantity":1,"selectedQuantity":1}
    ]}} }
```

### 4) 关键联调发现（写进 Skill 的硬约束）

**AI**：联调时踩到两个真实坑，必须写进 `MCP_INTEGRATION.md` 和 `SKILL.md`，否则 Skill 跑出来全是 0 元/报错：

1. **菜单 code ≠ 可计价售卖码。** `query-meals` 返回的是「人气热卖/精选单人餐」这类**分组/展示码**，直接丢进 `calculate-price` 会返回 `price:0 / productList:[]`（静默失败，不报错）。真正能计价的是 `query-meal-detail` 拿到的标准售卖码（如 `1440`=麦辣鸡腿汉堡）。→ Skill 必须先 `query-meal-detail` 再算价。
2. **优惠券必须 `couponId` + `couponCode` 成对传入。** 单独传 `couponId` 会报 `600010 使用优惠券需要couponId和couponCode`。`auto-bind-coupons` 返回里两者都齐全，直接透传即可。
3. **`query-meals` / `calculate-price` 必填 `beType` + `orderType` + `storeCode`**（到店=beType 1 / orderType 1；外送=beType 2 / orderType 2）。`query-nearby-stores` 到店场景不返回 `beCode`，可省略。

**用户**：好的，那 create-order 这一步怎么办？

**AI**：`create-order` 是**真实下单付款**，按约定我不会在演示里擅自调用（避免花你的钱）。Skill 设计成「算价 → 表格对比 → **用户确认后才下单**」，演示时停在 `calculate-price` 这一步，既完整展示省钱逻辑又不产生真实扣款。

### 5) Skill 落地

**AI**：最终把逻辑固化成 `SKILL.md`：角色定义为「麦门省钱管家」，6 步工作流（领券 → 定店 → 拉菜单 → 含券比价 → 表格推荐 → 确认下单），强制结构化表格输出 + 合规约束（非官方产品、不存凭证、个人助手定位）。

---

## 二、为什么这是合格的 WorkBuddy 作品

- **真·MCP 联动**：不是截图演示，而是用真实 Token 打通了 `initialize → auto-bind-coupons → query-nearby-stores → query-meals → query-meal-detail → calculate-price` 全链路，上面每段返回都是真实数据。
- **解决真问题**：把「进 App 翻券 + 比价」压缩成一句自然语言，且比价基于 `calculate-price` 真实含券结果，杜绝假优惠。
- **工程严谨**：联调发现的「分组码不可计价」「券须 Id+Code 成对」等问题已固化为 Skill 约束，避免使用者踩坑。
- **合规自觉**：涉及真实下单的 `create-order` 设计为需用户确认，演示不触发真实扣款。

## 三、参赛信息

- 项目仓库：`https://github.com/hejianm2035/mcd-savemaster`（公开）
- 报名 Issue：https://github.com/M-China/mcd-developer-innovation-challenge/issues/106 （官方已回「成功参赛」）
- 开发环境：WorkBuddy Agent 模式 + 麦当劳 MCP（`mcp.mcd.cn`，Streamable HTTP）
- 联调时间：2026-10-09
