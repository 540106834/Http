# **Traceroute 排错详解**
非常适合网络运维、开发和运维 SOP 使用。内容覆盖原理、命令、输出字段含义、失败原因和排查方法。

#  **Traceroute 是什么？**

* `traceroute`（Linux）/`tracert`（Windows）用于 **定位 IP 层（L3）路径中每一跳** 的网络连通性。
* 作用：

  1. 查看数据包从本机到目标 IP 经过的路由器（hop）
  2. 定位网络链路的中断点或延迟高的节点

原理：

* 利用 **TTL（Time To Live）逐跳递增**，触发路由器返回 ICMP 超时报文（Time Exceeded）
* 根据返回的每一跳 IP 和延迟，绘制路径

---

#  **Traceroute 基本命令**

### Linux

```bash
traceroute <目标IP或域名>
```

常用参数：

```
-m 30       # 最大跳数，默认 30
-n          # 不解析域名，只显示 IP
-p 33434    # UDP 端口（可选）
-I          # 使用 ICMP（类似 Windows ping）
```

### Windows

```cmd
tracert <目标IP或域名>
```

---

#  **Traceroute 输出示例（Linux）**

```
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  192.168.1.1  0.412 ms  0.386 ms  0.359 ms
 2  10.0.0.1     1.123 ms  1.098 ms  1.056 ms
 3  100.64.0.1   10.234 ms 10.567 ms 10.789 ms
 4  * * *
 5  8.8.8.8      30.456 ms 30.123 ms 30.678 ms
```

### 字段说明

| 字段   | 说明                         |
| ---- | -------------------------- |
| 第一列  | 跳数（hop）                    |
| 第二列  | 路由器 IP（或域名，如果未使用 `-n`）     |
| 后面三列 | 每一跳的延迟（ms），默认发 3 个 probe 包 |
| `*`  | 无响应（可能 ICMP 被阻止，丢包，或超时）    |

---

#  **Traceroute 排错逻辑**

### 1️ 每一跳都通，延迟正常

* 路由链路正常
* 出网正常
* 如果应用慢 → 可能是 TCP/应用层问题

### 2️ 某跳 `* * *`

* 该跳路由器不返回 ICMP Time Exceeded（防火墙屏蔽）
* 下一跳仍可达 → **不是问题**
* 下一跳也 `*` → 可能链路阻塞或丢包

### 3️ TTL 超时 / 无下一跳

* 数据包无法到达目标 → 路由缺失或断链
* 排查：

  * 本地默认路由 (`ip route`)
  * 上级路由器配置
  * 防火墙阻止

### 4️ 延迟异常高

* 某跳延迟明显高 → 可能链路拥塞、跨运营商或防火墙限速
* 注意：部分公网路由器 ICMP 优先级低 → ping / traceroute 延迟高不一定影响实际 TCP

---

#  **常用 traceroute 排错技巧**

1. **只显示 IP，不解析域名**（速度快）

```bash
traceroute -n 8.8.8.8
```

2. **使用 ICMP（类似 ping）**

```bash
traceroute -I 8.8.8.8
```

3. **增加最大跳数**

```bash
traceroute -m 50 <目标>
```

4. **抓包对照**

```bash
tcpdump -i eth0 icmp
```

* 对应 TTL probe 包是否到达
* 是否有 ICMP Time Exceeded 返回

---

#  **Traceroute 排错 SOP 示例**

```
1. ping 内网/网关 → 确认 LAN 正常
2. ping 外网 IP → 确认出网
3. traceroute <目标IP> → 定位链路断点
4. 检查 traceroute 输出：
   - * 下一跳丢失 → 可能防火墙 / ICMP 被阻
   - 延迟异常高 → 可能链路拥塞 / ISP 问题
5. traceroute -I / -n → 进一步验证
6. tcpdump → 确认 probe 包是否出入
7. 根据 traceroute 定位问题路由器 / 防火墙
```

---

#  **Traceroute 常见场景与判断**

| 场景                   | 说明            | 排查               |
| -------------------- | ------------- | ---------------- |
| traceroute 所有跳都通     | 网络正常          | 继续 ping 域名 / 测端口 |
| 某跳 * * *，下一跳通        | ICMP 被屏蔽      | 可忽略              |
| 某跳延迟高，后续正常           | 路由器 ICMP 优先级低 | 可忽略，不影响 TCP      |
| 某跳及之后全部 *            | 目标不可达         | 检查路由、防火墙、NAT     |
| traceroute 到外网 IP 不通 | 出网问题          | 检查默认路由、网关、防火墙    |

---

#  **总结**

* `ping` 只能判断连通性
* `traceroute` 可以定位链路中 **哪一跳出问题**
* 排错顺序建议：

```
ping 内网 → ping 网关 → ping 外网 IP → traceroute → ping 域名 → TCP/应用测试
```

* traceroute + tcpdump 可以精确判断 **丢包/ICMP屏蔽/链路断点**

