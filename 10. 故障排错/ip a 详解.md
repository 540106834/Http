下面给你 **最清晰、开发运维都能立刻用的 `ip a`（= `ip addr`）详解**，包含字段意义、常见场景、排错必看点。

---

#  **`ip a` 是什么？**

`ip a` 是 `ip addr show` 的简写，用来查看 **网卡、IP 地址、MAC、状态** 等信息。

它的作用范围主要在 **OSI 第 2 层 + 第 3 层**（链路层 + 网络层）。

---

#  **示例输出（标准 Linux 格式）**

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:3e:ab:cd brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.10/24 brd 192.168.1.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe3e:abcd/64 scope link 
       valid_lft forever preferred_lft forever
```

下面我帮你逐行讲解。

---

#  **第一行（接口基本信息）**

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
```

### **字段解释**

| 字段              | 含义              |
| --------------- | --------------- |
| `2:`            | 网卡编号（本机内部标号）    |
| `eth0:`         | 网卡名称            |
| `<...>`         | 网卡 flags（非常重要）  |
| `mtu 1500`      | 最大传输单元（默认 1500） |
| `qdisc`         | 队列机制（一般不用管）     |
| `state UP`      | 网卡是否开启（管理员层面）   |
| `mode DEFAULT`  | 接口模式            |
| `group default` | 网络接口分组          |
| `qlen 1000`     | 发送队列长度          |

---

# ☑ **重点字段：`<BROADCAST,MULTICAST,UP,LOWER_UP>`**

这是排故中最重要的一串。

### 常见 flags 说明

| 标记          | 含义                       |
| ----------- | ------------------------ |
| `UP`        | 网卡已在系统中启用（ifconfig up）   |
| `LOWER_UP`  | **物理链路已连接**（网线 / 光模块 OK） |
| `DOWN`      | 网卡关闭                     |
| `BROADCAST` | 支持广播                     |
| `MULTICAST` | 支持多播                     |
| `RUNNING`   | 网卡驱动已加载并正常工作             |

### 排错判断逻辑

| 状态                     | 说明                             |
| ---------------------- | ------------------------------ |
| `UP` + `LOWER_UP`      | **网卡完全正常**                     |
| `UP` 但 *没有* `LOWER_UP` | **物理层断了**（网线拔了 / Switch 口未 UP） |
| `DOWN`                 | 网卡没启用                          |

---

#  **第二行（MAC 地址）**

```
link/ether 00:16:3e:3e:ab:cd brd ff:ff:ff:ff:ff:ff
```

| 字段                      | 含义        |
| ----------------------- | --------- |
| `link/ether`            | 以太网设备     |
| `00:16:3e:3e:ab:cd`     | 本机 MAC 地址 |
| `brd ff:ff:ff:ff:ff:ff` | 广播地址      |

---

#  **第三行（IPv4 地址）**

```
inet 192.168.1.10/24 brd 192.168.1.255 scope global eth0
```

| 字段                  | 含义         |
| ------------------- | ---------- |
| `inet`              | IPv4 地址    |
| `192.168.1.10/24`   | IP + 子网掩码  |
| `brd 192.168.1.255` | 广播地址       |
| `scope global`      | 全局作用域，可被访问 |
| `eth0`              | 所属网卡       |

### **IPv4 有效期（DHCP 常见）**

```
valid_lft forever       # 有效期
preferred_lft forever   # 优选期
```

---

#  **第四行（IPv6 地址）**

```
inet6 fe80::216:3eff:fe3e:abcd/64 scope link
```

| 字段           | 说明                      |
| ------------ | ----------------------- |
| `inet6`      | IPv6 地址                 |
| `fe80::/64`  | 链路本地地址（每个网卡都有）          |
| `scope link` | 仅同一链路可达（不像 IPv4 可以全局访问） |

---

#  **`ip a` 实战排错要点（重点）**

### **1. 网卡状态是否正常？**

```
state UP
<...UP,LOWER_UP>
```

* 若 `LOWER_UP` 不存在 → 物理层问题

---

### **2. 是否正确拿到 IP？**

看：

```
inet 192.168.x.x/24
```

如果没有：

* DHCP 出问题
* 或配置文件没加载

---

### **3. IP 是否配置到正确网卡？**

例如你查的是 `eth1`，但 IP 在 `eth0`。

---

### **4. 是否有多个 IP？（常见 Bug 原因）**

多个 IPv4 会这样：

```
inet 10.0.0.1/24
inet 10.0.0.2/24
```

这会导致：

* 出口路由不稳定
* SNAT 选择错误

---

### **5. Docker / K8s 环境：是否是容器网卡？**

你会看到：

```
vethXXXXX
docker0
cni0
flannel.1
```

这些必须与容器网络插件一致。

---

#  **`ip a` 的常见用法**

### 查看某个网卡

```
ip a show eth0
```

### 启用网卡

```
ip link set eth0 up
```

### 禁用网卡

```
ip link set eth0 down
```

### 添加 IP

```
ip a add 10.0.0.1/24 dev eth0
```

---

#  **如果你想，我还可以给你：**

✅ 带图版：`ip a` 字段标注图
✅ `ip a` + `ip route` + `ip neigh` 超强联动排错教程
✅ K8s Pod 内看 `ip a` 怎么判断 CNI 问题
只要告诉我你要哪个版本即可！
