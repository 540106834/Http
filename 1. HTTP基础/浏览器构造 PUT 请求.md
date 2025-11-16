下面我给你**最完整、开发者视角的「浏览器如何构造 PUT 请求」详解**。
包括：浏览器能否自动发 PUT、触发方式、数据包构造过程、HTTP 头、预检、实际发包流程等。

你看完这份就能 **完全理解浏览器是怎么发出 PUT 的**。

---

# ✅ **一、浏览器会自动发 PUT 吗？**

**不会。**

浏览器不会因为“访问网站”“跳转页面”“输入网址”“点击链接”而自动触发 PUT。

**PUT 只能由 JS 主动构造发起**：

* `fetch()`
* `XMLHttpRequest`
* 某些前端库（Axios／jQuery ajax）
* Form 表单（但表单 *不支持* PUT）

也就是说：

> **PUT 请求 = 必须由脚本构造。浏览器本身不会自动创建 PUT。**

---

# ✅ **二、浏览器构造 PUT 的本质：Fetch / XHR 创建一个 HTTP 报文**

浏览器不会手写报文，它按照 HTTP 规范自动构造：

### **典型 PUT 请求结构**

```
PUT /api/user/123 HTTP/1.1
Host: example.com
Content-Type: application/json
Content-Length: 58
Origin: https://your-frontend.com
Referer: https://your-frontend.com/page
User-Agent: Mozilla/5.0 (...)
Accept: */*
Connection: keep-alive

{"name":"Tom","email":"tom@example.com"}
```

浏览器自动生成这些头：

* Host
* User-Agent
* Accept
* Referer
* Origin
* Connection
* Content-Length（自动计算）

你不需要手写。

---

# ✅ **三、浏览器发 PUT 的三种方式**

---

## **🔹方式 1：fetch（现代浏览器主流）**

```js
fetch("/api/user/123", {
  method: "PUT",
  headers: {
    "Content-Type": "application/json"
  },
  body: JSON.stringify({
    name: "Tom",
    email: "tom@example.com"
  })
})
```

---

## **🔹方式 2：XMLHttpRequest（旧风格）**

```js
var xhr = new XMLHttpRequest();
xhr.open("PUT", "/api/user/123");
xhr.setRequestHeader("Content-Type", "application/json");
xhr.send(JSON.stringify({
  name: "Tom",
  email: "tom@example.com"
}));
```

---

## **🔹方式 3：form 表单（不支持 PUT）**

HTML form 支持的 method 只有：

* GET
* POST

如果你写：

```html
<form method="PUT">
```

浏览器会自动变成：

```
method="GET"
```

或者直接忽略。

> **所以任何 PUT 都必须通过 JS 构造。**

---

# ✅ **四、跨域情况下，浏览器会自动添加「预检请求」**

PUT 属于 **非简单请求**
触发跨域预检（CORS preflight）。

## 浏览器会先发送：

### **OPTIONS 预检请求：**

```
OPTIONS /api/user/123 HTTP/1.1
Origin: https://frontend.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: content-type
```

服务器必须返回：

```
Access-Control-Allow-Origin: https://frontend.com
Access-Control-Allow-Methods: PUT
Access-Control-Allow-Headers: content-type
```

预检通过后，浏览器才会发真正 PUT 请求。

---

# ✅ **五、浏览器构造 PUT 的完整网络流程（HTTP/HTTPS + TCP/TLS）**

下面是最接近真实报文的步骤：

```
① JS 调用 fetch(method=PUT)
       ↓
② 浏览器准备 HTTP 报文
       ↓
③ DNS 查询（如果没有缓存）
       ↓
④ 建立 TCP（三次握手）
       ↓
⑤ 如果是 HTTPS → 建立 TLS 握手
       ↓
⑥ 如果跨域则先发 OPTIONS 预检
       ↓
⑦ 发送 PUT 报文（HTTP Request）
       ↓
⑧ 接收 Response
       ↓
⑨ JS Promise resolve()
```

这一套流程任何抓包工具（F12 / Wireshark / tcpdump）都能观察到。

---

# ✅ **六、浏览器 PUT 请求的抓包示例（真实示例）**

完整 PUT 报文（抓包截取）：

```
PUT /api/user/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Origin: https://app.front.com
Referer: https://app.front.com/user/123
User-Agent: Mozilla/5.0
Content-Length: 73

{"name":"Tom","email":"tom@example.com","age":20}
```

浏览器自动加的：

* User-Agent
* Origin
* Referer
* Content-Length
* Accept
* Connection

---

# ✅ **七、为什么 PUT 一定要用 JS？**

浏览器为了安全性，不允许用户在无意间发出改变服务器数据的请求。

PUT 是“变更资源”的操作，因此必须：

* **由脚本主动调用**
* **明确指定 method**
* **明确设置 Content-Type**
* **必须带 Origin/Referer（安全检查）**

---

# 🧠 **总结（牢记）**

| 问题                | 答案                    |
| ----------------- | --------------------- |
| 浏览器会自动发 PUT 吗？    | ❌ 永远不会                |
| PUT 如何产生？         | ✔ 必须 JS 发（fetch/XHR）  |
| 表单能不能发 PUT？       | ❌ 不可以                 |
| PUT 是否触发 CORS 预检？ | ✔ 会                   |
| 浏览器 PUT 请求结构？     | 浏览器根据规范自动构造完整 HTTP 报文 |

---

如果你愿意，我还能给你：

📌 **PUT 的真实 TCP/TLS 抓包图（分析每一帧）**
📌 **PUT / POST / PATCH 区别表（最佳实践）**
📌 **浏览器构造 PUT 请求—内部源码级流程图（Chrome 网络栈）**

你要哪个？
