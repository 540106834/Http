# **curl 和 nc（netcat）用法总结**

#  curl vs nc：核心区别

| 工具             | 用途                                    | 特点             |
| -------------- | ------------------------------------- | -------------- |
| **curl**       | 模拟 HTTP/HTTPS 请求（也可支持 FTP、gRPC-Web 等） | 应用于上层协议测试      |
| **nc（netcat）** | 直接测试 TCP / UDP 端口连通性，收发原始数据           | 最底层的网络工具，不解析协议 |

---

# 1️ curl：测试 HTTP 服务

### **1. 基础测试**

```bash
curl http://localhost:8080
```

### **2. 查看返回头部**

```bash
curl -I http://localhost:8080
```

### **3. 带超时、重试的健康检查**

```bash
curl --connect-timeout 3 --max-time 5 http://localhost:8080/health
```

### **4. 显示详细调试信息**

```bash
curl -v http://localhost:8080
```

### **5. POST JSON**

```bash
curl -X POST http://localhost:8080/api \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}'
```

---

# 2️ nc（netcat）：测试 TCP/UDP 端口是否能连通

### **1. 测试端口是否能连上（TCP）**

```bash
nc -zv 192.168.1.10 3306
```

解释：

* `-z`：仅扫描，不建立数据会话
* `-v`：显示详细信息

成功示例：

```
Connection to 192.168.1.10 3306 port [tcp/mysql] succeeded!
```

失败示例：

```
nc: connect to 192.168.1.10 port 3306 (tcp) failed: Connection refused
```

---

# 3️ nc 也能模拟服务端 / 客户端

### **启动一个临时 TCP 服务**

```bash
nc -l 8080
```

然后在另一个终端连接：

```bash
nc 127.0.0.1 8080
```

你输入什么对方都能看到 → 好用来调试数据发送。

---

# 4️ nc 测试 UDP

```bash
nc -u -l 9999
```

客户端：

```bash
nc -u 127.0.0.1 9999
```

---

# 5️ curl 和 nc 什么时候用？

### ✔ 用 curl：

* HTTP 服务是否正常？
* 返回状态码是否正确？
* Web 接口 API 调试
* Ingress / Service / 网关排查

### ✔ 用 nc：

* 端口能否连接？
* TCP 链路中哪里被阻断？
* 调试自定义协议（Redis、MQTT 等）
* 快速模拟一个 TCP 服务端或客户端

---

# 6️ 典型 Kubernetes 场景

###  测 Pod -> Service 连通

```bash
nc -zv myservice.default.svc 8080
```

###  测 Pod 内部 HTTP 健康

```bash
curl -v http://localhost:8080/health
```

###  测 NodePort 是否开放

```bash
nc -zv <node-ip> <nodeport>
```

---

需要我给你做个 **curl 与 nc 的常见问题速查表** 或 **实际生产排查案例** 吗？
