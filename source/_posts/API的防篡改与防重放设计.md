---
title: API的防篡改与防重放设计
date: 2025-05-24 10:01:14
tags:
---

# 背景

随着互联网技术的发展，越来越多的系统采用开放式架构，通过 Web 或移动端暴露 API 接口进行服务交互。API 不仅承担着前后端通信的桥梁角色，也往往是业务核心的入口，涉及用户登录、支付、数据操作等关键流程。因此，API 的安全性直接关系到系统整体的稳定性与数据安全。然而，在开放网络环境下，API 面临着诸多安全威胁，尤其以**数据篡改**与**重放攻击（Replay Attack）**最为常见且隐蔽。

# 危害

**数据篡改**：

- **非法交易：** 篡改支付金额或订单信息，达到少付钱甚至零付钱的目的。
- **越权访问：** 修改用户ID访问他人账户或敏感数据。
- **作弊行为：** 在游戏或服务系统中伪造请求数据进行刷分、刷资源。
- **业务逻辑破坏：** 通过构造非法参数，破坏业务流程或导致系统异常。

**重放攻击**：

- **重复支付/下单：** 攻击者重放支付请求造成资金损失。
- **恶意刷接口：** 重复请求接口造成资源消耗或额度盗用。
- **认证绕过：** 利用历史登录Token重放认证请求，冒充合法用户登录。
- **数据污染：** 通过重放操作不断添加或修改数据，扰乱数据一致性。

# 解决

**思想：**使用签名机制保证请求数据未被修改，使用随机数+redisTTL检查重放攻击



**原始参数**

```go
type Api struct {
	AppID string
	Info  string
}
```

**改造后**

```go

type Api struct {
	AppID string
	Info  string
	Nonce string // 随机数，确保唯一
	Sign  string // 签名，防止参数被篡改
}

```



**签名生成逻辑(客户端生成，服务端验签)**

```go
func generateSignature(secret string, params map[string]string) string {
	// 将参数按 key 排序（去除 sign 本身）
	var keys []string
	for k := range params {
		if k != "sign" {
			keys = append(keys, k)
		}
	}
	sort.Strings(keys)

	// 拼接参数为字符串格式：key1=val1&key2=val2...
	var builder strings.Builder
	for i, k := range keys {
		builder.WriteString(k + "=" + params[k])
		if i < len(keys)-1 {
			builder.WriteString("&")
		}
	}

	// HMAC-SHA256 签名
	mac := hmac.New(sha256.New, []byte(secret))
	mac.Write([]byte(builder.String()))
	return hex.EncodeToString(mac.Sum(nil))
}

```



**服务器验证流程**

1. 验证Nonce：查询redis是否有该Nonce，有就abort，没有就记录该Nonce并添加适合业务的过期时间
2. 验证Sign：使用相同签名生成逻辑验证签名是否相同
3. 验证其他：auth等

# 其他

- 如果是http请求，Nonce和Sign可以放到header或者URL中

