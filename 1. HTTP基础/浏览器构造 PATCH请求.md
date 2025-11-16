下面给你整理一份 **浏览器是如何构造 PATCH 请求的完整流程**。
它和 POST/PUT 类似，但有几个关键差异我会特别指出。

---

# 🚀 一、PATCH 是什么？

**PATCH 用于“部分更新资源”**。
比如只改用户昵称，不改其他字段。

---

# 🚀 二、浏览器构造 PATCH 请求的完整流程

以下步骤从“你的 JS 代码触发 PATCH 请求”开始讲。

---

# 1️⃣ 触发请求

浏览器只有在 **JavaScript** 中才会主动构造 PATCH 请求，比如：

```js
fetch("/api/user/123", {
  method: "PATCH",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ nickname: "NewName" })
});
```

HTML 的 `<form>` **不支持 PATCH**，所以 PATCH 基本都是通过 JS 发送的。

---

# 2️⃣ 解析 URL

浏览器将 URL 拆分：

```
协议：https
域名：example.com
端口：443
路径：/api/user/123
```

---

# 3️⃣ DNS 查询

获取服务器 IP：

```
example.com → 93.184.216.34
```

---

# 4️⃣ 建立 TCP 连接（3 次握手）

同 GET/POST：

```
SYN →  
    ← SYN/ACK  
ACK →
```

---

# 5️⃣ HTTPS 场景下建立 TLS 加密通道

（验证证书，协商加密算法）

只有握手完成后才发送 PATCH 内容。

---

# 6️⃣ 浏览器开始构造 PATCH 请求报文

PATCH 的 HTTP 报文结构与 POST 基本一致：

```
请求行
请求头
空行
请求体（partial update 数据）
```

我们展开看看👇

---

## 📌 ① 请求行（Request Line）

```
PATCH /api/user/123 HTTP/1.1
```

---

## 📌 ② 请求头（Headers）

浏览器自动加上的头：

```
Host: example.com
User-Agent: Mozilla/5.0
Accept: */*
Connection: keep-alive
```

还有你 JS 指定的：

```
Content-Type: application/json
Content-Length: 26
```

常见 Content-Type 有三种：

### A. application/json（最常用）

```
{"nickname":"NewName"}
```

### B. application/merge-patch+json（RFC 推荐）

表示“合并更新”

```
{"nickname": "NewName"}
```

### C. application/json-patch+json

用于操作描述（更复杂）：

```
[
  { "op": "replace", "path": "/nickname", "value": "NewName" }
]
```

浏览器不会自动构造这种，你必须手写。

---

## 📌 ③ 请求体（Body）

PATCH 的核心：
**只包含你要修改的字段。**

例子：

```
{"nickname":"NewName"}
```

---

# 7️⃣ 浏览器发送 PATCH 请求

最终报文看起来像这样：

```
PATCH /api/user/123 HTTP/1.1
Host: example.com
Content-Type: application/json
Content-Length: 26

{"nickname":"NewName"}
```

---

# 8️⃣ 服务器处理并返回响应

常见响应：

```
HTTP/1.1 200 OK
Content-Type: application/json

{"status":"updated"}
```

浏览器收到数据后交给 fetch 的 `then()` 或 async/await 处理。

---

# 🎯 完整流程总结图

```
JS 调用 fetch()
↓
浏览器解析 URL
↓
DNS 查询
↓
TCP 三次握手
↓
（HTTPS）TLS 握手
↓
浏览器构造 PATCH 请求（请求行 + 请求头 + 请求体）
↓
发送请求
↓
服务器返回响应
↓
浏览器交给 JS 回调处理
```

---

# 如果你需要，我可以继续为你整理：

✔ PATCH vs PUT vs POST 的区别（最容易搞混）
✔ PATCH 请求在 Wireshark 抓包中的实际样子
✔ 浏览器对不同 PATCH Content-Type 的自动构造方式（非常实用）

你想看哪一个？
