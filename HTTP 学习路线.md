
#  **HTTP 学习路线图**

## **阶段 1：HTTP 基础（协议理解）**

**目标**：理解 HTTP 请求/响应结构及常用方法

1. **HTTP 概念**

   * 什么是 HTTP，为什么它是无状态协议
   * HTTP 与 TCP/IP 的关系
   * HTTP 与 HTTPS 的区别

2. **请求与响应**

   * 请求行、请求头、请求体
   * 响应状态码（1xx~5xx）
   * 常用 HTTP 方法：GET / POST / PUT / DELETE / PATCH / OPTIONS / HEAD

3. **头部知识**

   * Content-Type、Accept、Authorization、User-Agent、Set-Cookie
   * Cookie / Session / Token 基本原理

4. **实践**

   * 使用 `curl` 发送 GET/POST 请求
   * 使用浏览器 DevTools 查看请求/响应

---

## **阶段 2：HTTP 高级特性**

**目标**：掌握性能优化和高级功能

1. **持久连接**

   * Connection: keep-alive
   * HTTP/1.1 vs HTTP/2 vs HTTP/3
   * 多路复用、流量优化、QUIC 协议

2. **缓存**

   * Cache-Control, ETag, Last-Modified
   * 浏览器缓存 vs CDN 缓存

3. **重定向与代理**

   * 3xx 重定向原理
   * 正向代理 / 反向代理

4. **安全**

   * HTTPS / TLS
   * HSTS、证书验证
   * 常见攻击防护：XSS / CSRF

5. **实践**

   * 配置 Nginx 或 Traefik 代理并观察请求
   * 测试 HTTP/2 支持的网页

---

## **阶段 3：API 与微服务**

**目标**：能设计、调用、调试 API 并理解微服务通信

1. **REST API**

   * 资源、URL 设计
   * 状态码规范
   * JSON / XML / Protobuf

2. **GraphQL / gRPC**

   * GraphQL 查询、变更
   * gRPC 原理 + HTTP/2 传输

3. **跨域与安全**

   * CORS 原理及处理
   * JWT / OAuth2 / API Key

4. **实践**

   * 使用 Postman 调试 REST API
   * 编写一个简单的 REST API（Node.js / Flask / Spring Boot）
   * 使用浏览器或 curl 调用并解析返回

---

## **阶段 4：运维、调试与性能优化**

**目标**：能分析微服务、云原生架构中的 HTTP 调用链

1. **链路分析**

   * Prometheus / Grafana 查看 HTTP 请求指标
   * Thanos / ServiceMesh (Istio/Linkerd) 观察请求链路

2. **性能优化**

   * 压测工具：ab, wrk, hey
   * HTTP Keep-Alive、连接池、请求合并
   * TLS 性能调优

3. **故障排查**

   * HTTP 状态码和错误排查
   * 日志分析、抓包工具（Wireshark / tcpdump）
   * 超时、重试、熔断机制

---

## **阶段 5：实战项目**

* 构建微服务架构（K8s + Prometheus + Grafana + ServiceMesh）
* 自己写一个应用 + Node Exporter + kube-state-metrics + 自定义 ServiceMonitor
* 用 `curl`/Postman 测试全链路数据采集
* 调优 HTTP 请求性能，分析 Prometheus / Grafana 中的延迟指标

---

 **学习技巧**

* 理论结合实践：每学一个概念都动手测试
* 学会抓包与分析请求：Chrome DevTools + curl + tcpdump
* 跟微服务架构结合：观察实际 HTTP 请求在集群里的流动


