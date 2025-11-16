# ** `curl` 详解**
覆盖：基本用法、HTTP 请求构造、常用参数、实际案例、抓包排查、调试技巧等。


# 1️ `curl` 是什么？

`curl`（Client URL）是一个命令行工具，用于发送 **HTTP/HTTPS** 及其他协议请求，并显示服务器响应。
核心特点：

* 支持 HTTP、HTTPS、FTP、SFTP、SMTP 等多种协议
* 可以模拟浏览器或 API 客户端请求
* 支持 GET、POST、PUT、DELETE 等 HTTP 方法
* 支持请求头、自定义 body、cookies、代理等

---

# 2️ 基本语法

```bash
curl [options] <URL>
```

示例：

```bash
curl https://example.com
```

* 默认发送 **GET 请求**
* 输出响应内容到终端

---

# 3️ 常用参数分类详解

### 🔹 请求方法

| 参数          | 作用                    |
| ----------- | --------------------- |
| `-X GET`    | 指定 GET 请求（默认不写也是 GET） |
| `-X POST`   | 指定 POST 请求            |
| `-X PUT`    | PUT 请求                |
| `-X DELETE` | DELETE 请求             |
| `-X PATCH`  | PATCH 请求              |

示例：

```bash
curl -X POST https://api.example.com/users
```

---

### 🔹 请求体（发送数据）

| 参数                                    | 作用                                       | 示例                                                                                |
| ------------------------------------- | ---------------------------------------- | --------------------------------------------------------------------------------- |
| `-d "key=value"`                      | 表单数据 `application/x-www-form-urlencoded` | `curl -d "name=Tom&age=20" http://example.com`                                    |
| `--data-binary @file`                 | 上传文件内容                                   | `curl --data-binary @test.json http://example.com`                                |
| `-H "Content-Type: application/json"` | 指定请求体类型                                  | `curl -H "Content-Type: application/json" -d '{"name":"Tom"}' http://example.com` |

---

### 🔹 请求头（Headers）

| 参数                | 作用              | 示例                                      |
| ----------------- | --------------- | --------------------------------------- |
| `-H "Key: Value"` | 添加自定义请求头        | `curl -H "Authorization: Bearer xxx"`   |
| `-u user:pass`    | HTTP Basic Auth | `curl -u admin:123 https://example.com` |

---

### 🔹 输出与调试

| 参数            | 作用                    | 示例                                      |
| ------------- | --------------------- | --------------------------------------- |
| `-v`          | 显示完整请求/响应（请求行、头、body） | `curl -v https://example.com`           |
| `-i`          | 显示响应头 + body          | `curl -i https://example.com`           |
| `-I`          | 仅显示响应头                | `curl -I https://example.com`           |
| `-o filename` | 将响应保存到文件              | `curl -o test.html https://example.com` |
| `-s`          | 静默模式（不显示进度条）          | `curl -s https://example.com`           |
| `-L`          | 自动跟随重定向               | `curl -L http://example.com`            |

---

### 🔹 Cookie 与会话

| 参数               | 作用            | 示例                                       |
| ---------------- | ------------- | ---------------------------------------- |
| `-b "key=value"` | 发送 cookie     | `curl -b "sessionid=abc123"`             |
| `-c cookies.txt` | 保存 cookie     | `curl -c cookies.txt http://example.com` |
| `-b cookies.txt` | 使用已保存的 cookie | `curl -b cookies.txt http://example.com` |

---

### 🔹 HTTPS 与证书

| 参数                  | 作用              | 示例                                           |
| ------------------- | --------------- | -------------------------------------------- |
| `-k` / `--insecure` | 忽略证书验证（不安全，仅调试） | `curl -k https://selfsigned.com`             |
| `--cert client.pem` | 指定客户端证书         | `curl --cert client.pem https://example.com` |
| `--key client.key`  | 指定私钥            | 同上                                           |

---

### 🔹 限制与超时

| 参数                      | 作用    | 示例                                             |
| ----------------------- | ----- | ---------------------------------------------- |
| `--connect-timeout <秒>` | 连接超时  | `curl --connect-timeout 5 https://example.com` |
| `--max-time <秒>`        | 总请求超时 | `curl --max-time 10 https://example.com`       |
| `--retry <次数>`          | 失败重试  | `curl --retry 3 https://example.com`           |

---

# 4️ 常见使用场景

### 🔹 GET 请求

```bash
curl https://api.example.com/users
```

### 🔹 POST 请求 JSON

```bash
curl -X POST https://api.example.com/users \
     -H "Content-Type: application/json" \
     -d '{"name":"Tom","age":20}'
```

### 🔹 PUT 更新

```bash
curl -X PUT https://api.example.com/users/123 \
     -H "Content-Type: application/json" \
     -d '{"name":"Tom Lee","age":21}'
```

### 🔹 下载文件

```bash
curl -o file.zip https://example.com/file.zip
```

### 🔹 跟踪重定向

```bash
curl -L http://example.com
```

### 🔹 调试请求（显示完整请求/响应）

```bash
curl -v -X POST https://example.com/api \
     -H "Content-Type: application/json" \
     -d '{"name":"Tom"}'
```

---

# 5️ curl 与抓包/排错

* `curl -v` 可显示请求行、请求头、响应头、body
* 与 **tcpdump / Wireshark** 配合，可以抓到底层 TCP/HTTP 流量
* 可模拟浏览器请求，便于排查 API 问题
* 可用于脚本自动化测试接口

---

# 6️ Tips & 高级用法

1. **带用户名密码**

```bash
curl -u user:pass http://example.com
```

2. **PUT 上传文件**

```bash
curl -X PUT --data-binary @file.txt https://example.com/upload
```

3. **下载大文件显示进度条**

```bash
curl -O https://example.com/bigfile.zip
```

4. **发送自定义 HTTP 方法**

```bash
curl -X PATCH https://example.com/api/users/123 \
     -H "Content-Type: application/json" \
     -d '{"age":22}'
```

5. **调试跨域**

```bash
curl -v -H "Origin: https://example.com" https://api.example.com
```

---

# 7️ 总结

* `curl` 是万能命令行 HTTP 客户端
* 支持 GET/POST/PUT/DELETE/PATCH
* 可以自定义请求头、请求体、Cookie、证书、超时
* 可结合 `-v` 和抓包工具排查网络/API 问题
* 用它可以快速模拟浏览器行为、调试 RESTful API


