# **HTTP GET 请求详解**
覆盖协议细节、缓存机制、参数解析、抓包分析、常见误区、真实业务问题排查示例——让你彻底理解 GET。

#  1. GET 请求的核心语义

**GET = 获取资源的表示（Retrieve Representation）
目的：读取数据，不应产生副作用。**

✔ 理论上 **安全（Safe）**
✔ 幂等（Idempotent）
✔ 可缓存（Cacheable）
✔ 不改变服务器状态（至少不应该）

**GET 是“读取资源”的唯一正确选择。**

---

#  2. GET 请求的结构（协议格式）

典型 GET 请求报文：

```
GET /users?page=2&limit=20 HTTP/1.1
Host: api.example.com
User-Agent: curl/8.0
Accept: application/json
Accept-Encoding: gzip, deflate
Connection: keep-alive
```

特点：

### **2.1 请求行**

```
GET <path>?<query_string> HTTP/1.1
```

### **2.2 只有 Header（一般无 body）**

HTTP 标准**允许** GET 带 body，但：

* 大多数 Web 服务器不支持（Nginx、Apache）
* 大多数框架忽略它（Spring、Express、Gin）
* 中间代理可能丢弃或导致缓存失败

**因此：GET 实际上不应携带请求体**。
所有参数都应放在：

* Query String
* Header
* Path Variable

---

#  3. Query String 详解（URL 参数）

GET 参数通过 URL 查询字符串传递：

```
/users?name=tom&age=20&sort=desc
```

特点：

* Key/Value 形式
* URL 编码（UTF-8 + %xx）
* 允许重复 key：

  ```
  tags=a&tags=b&tags=c
  ```
* 属于 URL 而不是 HTTP body

一般用于：

* 分页
* 搜索
* 筛选
* 排序
* 多条件查询

---

#  4. 响应结构（典型示例）

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 128
Cache-Control: max-age=600
ETag: "abc123"

{
  "items": [...],
  "page": 1,
  "size": 20
}
```

关键字段：

### **4.1 Cache-Control**

控制浏览器是否可缓存
如：

```
Cache-Control: max-age=600, public
```

### **4.2 ETag / If-None-Match（强缓存 + 协商缓存）**

用于判断资源是否改变

流程：

1. 第一次返回

   ```
   ETag: "123abc"
   ```
2. 第二次请求

   ```
   If-None-Match: "123abc"
   ```
3. 如果资源未变，返回：

   ```
   304 Not Modified
   ```

 **GET 是最重要的 HTTP 缓存对象。**

---

#  5. 304（Not Modified）详解

请求头：

```
If-None-Match: "abc123"
```

响应：

```
304 Not Modified
```

含义：

* 不需要返回 Body
* 浏览器直接使用本地缓存

**极大减少带宽**

---

#  6. GET 是否有请求体？

按照标准 **允许**，但实际几乎不使用。

原因：

1. 很多服务器忽略 body
2. 代理缓存不支持
3. 会误导其他开发者
4. 并不符合 GET 语义

**结论：不要在 GET 放 Body。**

---

#  7. GET 是幂等的 → 重复请求不会造成副作用

定义：

```
GET /product/1
```

无论调用 1 次还是 100 次，都不会改变任何资源。

但注意：

###  并不是“对服务器完全没影响”

* 可能增加访问计数
* 可能触发日志
* 可能导致缓存生成

但这些不属于 HTTP 语义中的“副作用”。

---

#  8. GET vs POST 的真正区别（非常关键）

| 项目   | GET           | POST    |
| ---- | ------------- | ------- |
| 语义   | 读取            | 创建/处理   |
| 参数   | URL           | Body    |
| 缓存   | ✔             | ❌       |
| 幂等   | ✔             | ❌       |
| 可记录性 | 可能暴露在日志/浏览器历史 | 相对安全    |
| 长度   | URL 有长度限制     | Body 无限 |

### 实战中错误示例（常见问题）：

```
GET /create_order?user=1
```

⚠️ 这违反了 GET 的无副作用原则，应使用 POST。

---

#  9. GET 抓包分析（Wireshark 实例）

抓包看到：

```
GET /v1/users?page=1 HTTP/1.1
Host: api.example.com
Accept: application/json
```

响应：

```
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 120
ETag: "v123"
Cache-Control: max-age=60
```

### 分析过程：

1. TCP 三次握手
2. TLS 握手（HTTPS）
3. 发送 GET 请求
4. 服务器返回 200
5. 浏览器缓存 ETag
6. 再次请求时，如果使用缓存，返回 304

---

#  10. GET 常见面试题（附答案）

### **Q1：GET 能不能带 body？**

能，但不推荐。大多数服务器/代理不支持。

### **Q2：GET 和 POST 的区别？**

语义 + 副作用 + 缓存 + 传输方式，不是“长度限制”。

### **Q3：GET 是幂等的吗？为什么？**

是。重复执行不会改变资源状态。

### **Q4：GET 为什么能被缓存？**

因为 GET 没有副作用，缓存安全。

### **Q5：什么时候使用 GET？**

所有读取类接口。

---

#  11. GET 最佳实践

### ✔ 1. 使用分页参数

```
?limit=20&page=1
```

### ✔ 2. 使用过滤参数

```
?status=active&sort=created_at_desc
```

### ✔ 3. 强制缓存

```
Cache-Control: max-age=60
ETag: "xxxx"
```

### ✔ 4. 永远不要用 GET 做写操作

---

# 需要我继续扩展吗？

我还能提供：

### 🔥《HTTP GET 请求实战指南》（含 curl、Postman、抓包截图）

### 🔥《HTTP GET 配合 Nginx 缓存架构图》

### 🔥《结合前端+后端的 GET 缓存策略（ETag、Last-Modified）》

你想继续哪一部分？
