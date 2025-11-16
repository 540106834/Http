 **`ss` 命令实用详解及案例**
 从基础到进阶，涵盖网络排错、服务监控和排查端口占用，非常实用。


#  **`ss` 命令简介**

* `ss` = **Socket Statistics**
* 替代 `netstat` 的现代工具
* 功能：

  1. 查看 **TCP/UDP 连接状态**
  2. 查看 **监听端口**
  3. 查看 **socket 信息**（包括进程 PID/程序名）
* 优点：速度快，信息更全

---

#  **`ss` 核心参数**

| 参数                            | 含义                    | 示例                        |
| ----------------------------- | --------------------- | ------------------------- |
| `-t`                          | 显示 TCP 连接             | `ss -t`                   |
| `-u`                          | 显示 UDP 连接             | `ss -u`                   |
| `-l`                          | 显示监听端口                | `ss -l`                   |
| `-n`                          | 不解析域名或端口              | `ss -tn`                  |
| `-p`                          | 显示进程 PID / 名称         | `ss -tp`                  |
| `-a`                          | 显示所有 socket（包括监听和非监听） | `ss -a`                   |
| `-r`                          | 显示路由信息                | `ss -r`                   |
| `state <状态>`                  | 筛选特定连接状态              | `ss -t state ESTABLISHED` |
| `sport = :80` / `dport = :80` | 按源/目的端口过滤             | `ss -t sport = :443`      |
| `-4` / `-6`                   | IPv4 / IPv6           | `ss -4t`                  |

---

#  **常用场景与示例**

### 1️ 查看所有 TCP 连接

```bash
ss -t
```

输出示例：

```
State      Recv-Q Send-Q Local Address:Port   Peer Address:Port
ESTAB      0      0      192.168.1.10:22    192.168.1.100:54321
```

* **State** → 连接状态（ESTAB=已建立）
* **Local Address:Port** → 本地端口
* **Peer Address:Port** → 远端地址

---

### 2️ 查看所有监听端口

```bash
ss -ltn
```

* `-l` → 监听
* `-t` → TCP
* `-n` → 不解析域名
  输出示例：

```
State  Recv-Q Send-Q  Local Address:Port
LISTEN 0      128     0.0.0.0:80
LISTEN 0      128     127.0.0.1:3306
```

* 可以快速判断本机哪些端口正在监听

---

### 3️ 查看 UDP 监听端口

```bash
ss -lun
```

输出示例：

```
State  Recv-Q Send-Q Local Address:Port
UNCONN 0      0      0.0.0.0:123
UNCONN 0      0      0.0.0.0:161
```

* UDP socket 状态通常是 `UNCONN`（未连接）

---

### 4️ 查看指定状态的连接

```bash
ss -t state ESTABLISHED
```

* 查看已建立的 TCP 连接
* 排查活跃会话、排查僵尸连接

---

### 5️ 查看端口对应进程

```bash
ss -tpn
```

输出示例：

```
ESTAB 0 0 192.168.1.10:22 192.168.1.100:54321 users:(("sshd",pid=1234,fd=3))
LISTEN 0 128 0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=2345,fd=6))
```

* **users** → 进程名 + PID + 文件描述符
* 快速定位哪个进程占用端口

---

### 6️ 按端口过滤

```bash
ss -tn sport = :443
```

* 查看源端口为 443 的 TCP 连接

---

### 7️ 网络排错组合案例

#### 场景：某服务端口无法访问

```bash
ss -ltnp | grep 8080
```

1. 确认端口是否在监听
2. 查看对应进程 PID / 名称
3. 检查防火墙策略

#### 场景：排查占用过多连接的服务

```bash
ss -s
```

* 显示 socket 总览：

```
TCP:   10 (estab 5, closed 2, orphaned 0, synrecv 0, timewait 3/0), ports 3
```

* 快速了解连接状态分布

---

#  **总结**

* `ss` 是 **现代网络排错必备工具**
* 用法灵活：监听端口查看、连接状态排查、进程匹配、端口占用分析
* 常用组合：

```
ss -ltnp        # 查看 TCP 监听及对应进程
ss -tunp        # 查看所有 TCP/UDP 活跃连接及进程
ss -t state ESTABLISHED  # 查看已建立 TCP 连接
ss -s           # socket 总览
```

