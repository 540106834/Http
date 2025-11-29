
#  域名系统中的两大角色：**Registrar（注册商）** 和 **Registry（注册局）**

## 1️ Registry —— “域名的最高权威数据库”

Registry（注册局）是 **顶级域名（TLD） 的官方管理者**。
例如：

| 顶级域名 | Registry（注册局） |
| ---- | ------------- |
| .com | Verisign      |
| .net | Verisign      |
| .org | PIR           |
| .cn  | CNNIC         |
| .jp  | JPRS          |

### Registry 的核心作用（官方数据库层）：

#### ✔ 1. 保存域名的根记录（主要是 NS）

* **Registry 只保存域名最核心的东西：NS（权威 DNS 地址）和 domain 状态。**
* 不保存你的 A 记录、CNAME、MX 等，这些存储在 **权威 DNS 服务器** 上。

例：

```
example.com 的 registry 中只保存：
NS = ns1.cloudflare.com
NS = ns2.cloudflare.com
```

#### ✔ 2. 提供 TLD 区域文件（zone）给根 DNS

根 DNS 查询 `.com` 时，会读取 Verisign（.com 注册局）提供的 zone 文件。

#### ✔ 3. 执行域名生命周期（注册、删除、过期）

* Pending Delete
* Redemption
* Grace Period
  这些逻辑都由 Registry 执行。

#### ✔ 4. 不面向普通用户

你无法直接在 Registry 买域名（除了少数如 `.co.uk`）。

---

## 2️ Registrar —— “向用户卖域名的商店”

Registrar 是你购买域名的地方，例如：

* Namecheap
* GoDaddy
* Google Domains（已被 Squarespace 收购）
* 阿里云
* 腾讯云
* Cloudflare Registrar

### Registrar 的核心作用（商业层）：

#### ✔ 1. 帮你向 Registry 下单购买域名

比如你在 Namecheap 下单 `abc.com`：

* Namecheap 通过 EPP 协议将订单发给 `.com` Registry（Verisign）
* Registry 负责把 `abc.com` 写入官方数据库

#### ✔ 2. 帮用户设置 NS（权威 DNS 地址）

你在 Registrar 页面改域名的 `NS 记录` 时，是 **Registrar 把你的 NS 写入 Registry**。

#### ✔ 3. 提供域名管理界面

比如：

* DNS 管理（如果你使用 Registrar 的 DNS）
* 修改 NS
* 续费、转移
* Whois 信息维护
* 隐私保护（Privacy protection）

#### ✔ 4. 用户支付的入口

你付钱给 Registrar，而 Registrar 再付部分费用给 Registry。

---

#  一个完整例子：example.com 是怎么存在的？

假设用户在 **Namecheap** 购买 `example.com`：

### ① 用户 → Registrar（Namecheap）

用户输入：

```
我想买 example.com
```

### ② Registrar → Registry（Verisign）

Registrar 用 EPP 协议告诉 Registry：

```
创建 domain example.com
NS = ns1.cloudflare.com
NS = ns2.cloudflare.com
注册人：xxxx
过期时间：2026-01-01
```

### ③ Registry 写入官方数据库

Registry 的 `.com` 区域文件 now 包含：

```
example.com   NS   ns1.cloudflare.com
example.com   NS   ns2.cloudflare.com
```

### ④ 根 DNS / TLD DNS 读取这个 Registry 数据

当有人访问 `example.com`：

* 根 DNS 找 `.com` TLD
* `.com` TLD（来自 Registry）告诉解析器：

```
example.com 的权威 DNS 在 Cloudflare
```

### ⑤ Cloudflare（权威 DNS）存储域名真实记录

Cloudflare 的系统里才有：

```
A = 1.2.3.4
AAAA = ...
CNAME = ...
MX = ...
TXT = ...
```

---

#  总结：Responsibilities 对比

| 功能            | Registrar（注册商） | Registry（注册局） |
| ------------- | -------------- | ------------- |
| 卖域名           | ✔              | ✖             |
| 域名官方数据库       | ✖              | ✔（只存 NS + 状态） |
| 用户界面          | ✔              | ✖             |
| 写入 NS         | ✔（通过 EPP）      | ✔（执行并保存）      |
| 存储 A/MX/CNAME | ✖（除非提供 DNS 服务） | ✖             |
| 管理 TLD        | ✖              | ✔             |
| 域名生命周期管理      | ✖              | ✔             |

---

#  一个最重要的记忆点

**Registry 才是官方数据库，但它只存最关键的 NS，不存 A/CNAME/MX 等。
Registrar 是卖域名的商店，负责帮你把 NS 写入 Registry。**

