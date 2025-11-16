# **`tcpdump` 详解 + 真实案例大全**。
内容清晰、实用、偏 DevOps/SRE/后端常用排错场景。


#  一、`tcpdump` 是什么？

`tcpdump` 是 Linux 最强悍的网络抓包工具，用于在指定网卡上捕获网络层、传输层、应用层数据包。

它可以：

* 抓 TCP/UDP/ICMP 包
* 抓指定端口流量（如 80/443/3306）
* 抓某个 Pod/主机的进出流量
* 分析三次握手、延迟、超时、丢包
* 保存为 `.pcap` 用 Wireshark 深度解析

---

#  二、命令核心结构（必须理解）

```
tcpdump [参数] [过滤器]
```

---

#  三、最常用参数详解（必须掌握）

| 参数             | 作用                   |
| -------------- | -------------------- |
| `-i eth0`      | 指定网卡                 |
| `-n`           | 不解析域名（避免速度慢）         |
| `-nn`          | 不解析域名 + 不解析端口名       |
| `-v` / `-vv`   | 显示更详细的信息             |
| `-c 10`        | 抓 10 个包后停止           |
| `-w file.pcap` | 将抓包写入文件              |
| `-r file.pcap` | 读取抓包文件               |
| `-X`           | 以 16 进制+ASCII 输出     |
| `-A`           | 以 ASCII 输出（适合看 HTTP） |
| `-s 0`         | 抓全包（默认只抓前 96 bytes）  |

---

#  四、最常用过滤器（必须掌握）

| 过滤器                  | 说明         |     |      |
| -------------------- | ---------- | --- | ---- |
| `host 1.2.3.4`       | 只抓某个 IP    |     |      |
| `src host 1.2.3.4`   | 指定源 IP     |     |      |
| `dst host 1.2.3.4`   | 指定目的 IP    |     |      |
| `port 80`            | 指定端口       |     |      |
| `src port 80`        | 来自端口 80 的包 |     |      |
| `tcp` 、 `udp`、`icmp` | 指定协议       |     |      |
| `net 10.0.0.0/8`     | 指定网段       |     |      |
| `&&、                 |            | 、!` | 逻辑组合 |

示例：

```
tcpdump -i eth0 tcp port 8080 and host 10.0.0.5
```

---

#  五、最常用实战命令（最实用收藏）

### ① 抓某端口流量（HTTP、API、服务排错）

```
tcpdump -i eth0 -nn port 8080
```

### ② 抓某个 IP 的所有流量

```
tcpdump -i any host 192.168.1.10
```

### ③ 抓 TCP 三次握手过程

```
tcpdump -i eth0 -nn tcp[tcpflags] & tcp-syn != 0
```

### ④ 抓 HTTP 明文数据（非 HTTPS）

```
tcpdump -i eth0 -A -s 0 port 80
```

### ⑤ 抓 DNS 请求

```
tcpdump -i eth0 -nn port 53
```

### ⑥ 保存到文件供 Wireshark 分析

```
tcpdump -i eth0 -s 0 -w out.pcap
```

### ⑦ 只抓进来的包（ingress）

```
tcpdump -i eth0 -nn inbound and port 443
```

### ⑧ 抓丢包、重传（排查网络不稳定）

```
tcpdump -i eth0 -nn tcp port 8080 and 'tcp[tcpflags] == tcp-retransmit'
```

---

#  六、实战案例（重点）

下面的案例都是一线运维/后端常用的故障定位方式。

---

# 案例 1：接口请求超时，怀疑没到服务

问题：前端调用 API 超时。
排查：查看服务器是否收到请求。

```
tcpdump -i eth0 -nn port 8080
```

 **结果**
如果 **完全没有数据包** → 请求根本没有到服务，有可能：

* DNS 错误
* 防火墙阻断
* Pod/service/ingress 转发有问题
* 外层网关无流量

如果 **看到 SYN 包但没有后续 ACK** → 目标端口未监听 / 被防火墙丢弃。

---

# 案例 2：端口明明监听，客户端连不上

客户端提示：`connection refused`

抓包：

```
tcpdump -i eth0 -nn 'tcp[tcpflags] == tcp-syn' and port 3306
```

 **看返回是否有 RST 包**：

* 如果收到 **RST** → 服务端端口未真正监听
* 没收到包 → 路由/防火墙/安全组丢包

---

# 案例 3：DNS 查询慢，网站打开卡顿

```
tcpdump -i eth0 -nn port 53
```

如果 DNS 请求要等待几秒才返回 → DNS 服务器问题。

---

# 案例 4：检查是否有丢包/重传（网络慢）

```
tcpdump -i eth0 -nn tcp port 443
```

分析：

* 有很多 **Retransmission 重传包** → 网络存在丢包
* RTT 跳变很大 → 网络拥塞或链路不稳

---

# 案例 5：分析三次握手是否正常

```
tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-syn != 0'
```

看是否有：

* SYN
* SYN/ACK
* ACK

缺失某一步就能定位链路问题。

---

# 案例 6：排查内网调用丢包（服务间调用失败）

抓双向流量：

```
tcpdump -i eth0 -nn host 10.20.1.5 and port 9090
```

结合：

* `curl -v`
* `ss -lntup`
* `traceroute`

即可锁定问题点。

---

# 案例 7：查看 HTTP POST 请求内容（明文）

```
tcpdump -i eth0 -A -s 0 port 8080
```

可以直接看到：

* Header
* Body
* JSON 内容

---

# 案例 8：抓包证明是对方服务器的问题

你抓包看到：

* SYN 已发出
* 但一直没有 SYN/ACK 返回
  → 说明对方服务器没回应。

把 pcap 给对方即可。

---

# 案例 9：Kubernetes Pod 内抓包

Pod 里常只有 `busybox` 无 tcpdump，你可以：

```
kubectl debug pod-name --image=ubuntu --copy-to=debug
```

然后：

```
tcpdump -i eth0 port 8080
```

可以排查 Service / Ingress / Pod 链路。

---

# 案例 10：抓 HTTPS 握手，排查证书问题

```
tcpdump -i eth0 -nn port 443 -s 0 -w tls.pcap
```

然后用 Wireshark 查看：

* TLS version
* Cipher suite
* 是否握手失败
* 是否 SNI 错误

---

#  总结：tcpdump 使用标准流程（SRE 推荐）

1. **抓包前** 确认网卡：

   ```
   ip a
   ```

2. **抓指定流量**

   ```
   tcpdump -i eth0 -nn host x.x.x.x and port 8080
   ```

3. **必要时抓全包并保存**

   ```
   tcpdump -i eth0 -s 0 -w file.pcap
   ```

4. **用 Wireshark 深度分析**

5. 搭配工具：

   * `curl -v`
   * `ss -lntup`
   * `ping`
   * `traceroute`

---

如果你想，我可以再给你：

✅ **tcpdump 最常用 30 条命令合集**
或
✅ **网络排错全流程：从 curl 到 tcpdump 的“SOP 图”**


