# 2026 Clash 机场各种协议差别介绍与选择指南（VMess / VLESS / Trojan / Reality / Hysteria2）

最近更新：2026-06-07

> 说明：原版 Clash（`Dreamacro/clash`）已不再适合作为下载入口；本文里的 “Clash” 泛指规则分流生态的客户端/内核（常见为 `mihomo` / `sing-box`）。不同内核对协议（Reality/TUIC/Hysteria2 等）的支持程度不完全一致。
>
> 核对时间：2026-06-07。本次核到的 latest：`mihomo v1.19.27`、`sing-box v1.13.13`、`Xray-core v26.3.27`、`Hysteria app/v2.9.2`、`tuic-server-1.0.0`。协议字段和客户端支持变化很快，实际可用性以机场面板参数、内核版本和官方文档为准。

## Clash 介绍

Clash 是一款跨平台的规则化网络代理客户端，主要用于科学上网、网络调试、流量转发等目的。它采用 YAML 格式进行配置，支持多种代理协议，并通过灵活的规则系统实现自动选择最佳节点。Clash 的设计理念强调高可扩展性和稳定性，深受高级用户与开发者欢迎。

### Clash 的应用场景

- 科学上网突破网络封锁
- 网站加速与内容解锁
- 国际数据传输优化
- 多节点选择与负载均衡
- 网络请求调试

## Clash 支持的协议概览（总览图先看这里）

规则分流生态可覆盖多种代理协议与传输方式，但“是否能用”最终取决于你使用的内核版本、客户端实现、以及机场的落地方式。常见协议包括：

- VMess（V2Ray 核心协议）
- VLESS（更轻量的无状态版本）
- Shadowsocks（SS）
- ShadowsocksR（SSR，存量/逐步淘汰）
- Trojan
- SOCKS5
- HTTP(S) 代理
- Hysteria2 / TUIC（基于 UDP 的高性能协议）
- Reality（常见为 VLESS + Reality 组合，用于更强伪装/抗探测）
- xHTTP / gRPC / HTTP/2 等传输方式（不是独立代理协议，但会影响兼容性）

### 协议对比简表（基础）

| 协议       | 加密机制   | 伪装能力 | UDP支持 | 兼容性 | 备注 |
|------------|------------|----------|---------|--------|--------|
| VMess      | 有         | 中       | 支持    | 高     | 存量较多，旧配置常见 |
| VLESS      | 无（需 TLS）| 强       | 支持    | 高     | 常与 TLS/Reality/Vision 搭配 |
| Shadowsocks| 有         | 弱       | 支持    | 很高   | 简洁通用，但高级伪装弱 |
| Trojan     | TLS        | 强       | 支持    | 高     | HTTPS 伪装思路，配置相对通用 |
| Hysteria2  | QUIC+UDP   | 中       | 强      | 中     | 依赖 UDP/QUIC 网络环境 |
| TUIC       | TLS+UDP    | 中       | 强      | 中     | 依赖 UDP/QUIC 网络环境 |
| Reality    | TLS+X25519 | 强       | 支持    | 中-高  | 参数敏感，需客户端/内核支持 |

## VMess

VMess 是 V2Ray 项目的核心协议之一，具备加密传输、身份验证等功能，仍能在不少存量机场和旧订阅中看到。

### 工作原理

- 客户端与服务端建立连接后，通过动态生成的加密信息进行身份验证。
- 支持多种传输方式：TCP、mKCP、WebSocket、HTTP/2、QUIC 等。
- 默认启用 AES-128-GCM 或 ChaCha20 加密算法。

### 安全性分析

VMess 具备身份验证和传输加密能力，避免明文传输，并通过时间戳机制降低重放风险。但相较于后起的 VLESS + TLS/Reality 组合，在伪装和生态更新上不再是优先选择。

### 优缺点

**优点：**
- 加密算法内建，配置简单。
- 与 V2Ray 完美兼容，部署广泛。
- 支持多种传输方式与混淆手段。

**缺点：**
- 易被主动探测识别（在无 TLS 伪装时）。
- 配置更新缓慢，依赖 V2Ray 核心升级。

## VLESS

VLESS 是 V2Ray / Xray 生态中常见的轻量协议，采用无状态设计，本身不包含加密层，通常需要结合 TLS、XTLS Vision 或 Reality 等方式实现安全传输。

### 特性与改进

- 移除了加密逻辑，完全依赖外部加密（如 TLS）。
- 减少客户端和服务器之间的数据交互负担。
- 支持 XTLS，为绕过防火墙提供更高隐蔽性。

### VLESS 与 VMess 的差异

| 特性             | VMess      | VLESS      |
|------------------|------------|------------|
| 是否加密         | 是         | 否（需TLS）|
| 状态机制         | 有状态     | 无状态     |
| 伪装能力         | 中         | 强         |
| TLS/XTLS支持     | 支持       | 强制       |
| 性能优化         | 一般       | 更优       |

### TLS 与 XTLS 的优势

- **TLS**：通用加密传输标准，可结合 CDN 进行伪装，适合 Web 环境中传输。
- **XTLS**：专为代理协议设计的扩展传输层，具备低延迟、高隐蔽、低功耗等优势，适合高性能节点使用。

---

## Shadowsocks (SS)

Shadowsocks 是一款轻量级、高性能的加密代理协议，最早由中国开发者 clowwindy 创建，旨在突破网络审查和屏蔽。

### 基本原理

- 使用 SOCKS5 代理协议接口。
- 客户端将请求通过加密算法发送至服务端，服务端解密并转发请求。
- 双向通信均加密，避免明文泄露。

### 加密算法选择

Shadowsocks 支持多种对称加密算法，包括：

- AES-256-GCM
- ChaCha20-IETF-Poly1305
- AES-128-GCM

推荐使用 GCM 或 Poly1305 类别算法，以获得更高安全性和性能。

### SS 与 VMess/VLESS 对比

| 特性         | Shadowsocks   | VMess       | VLESS       |
|--------------|---------------|-------------|-------------|
| 加密机制     | 客户端自选    | 内建        | 外部依赖    |
| 隐蔽性       | 较弱          | 中          | 强          |
| 协议扩展性   | 差            | 强          | 强          |
| 性能         | 高            | 中          | 高          |

**优点：**
- 简洁、轻量、易于部署。
- 广泛兼容客户端、平台和路由器。

**缺点：**
- 缺乏高级伪装能力，容易被审查识别。
- 不支持多路复用和动态节点切换。

## ShadowsocksR (SSR)

ShadowsocksR 是 Shadowsocks 的非官方增强版本，由 BreakWa11 开发，加入了多项混淆机制和协议扩展，目的是增强抗封锁能力。

### SSR 的特点

- 支持多种协议混淆（如 `auth_sha1_v4`, `http_simple`）。
- 支持混淆插件与流量分流。
- 对不同流量特征进行模拟，降低被探测风险。

### 应用场景

- 高封锁环境（如中国大陆）中绕过 GFW。
- 与 CDN、TLS 伪装结合效果更佳。
- 多为私有协议，不建议用于公共环境。

**注意**：由于 SSR 属于非主流协议，已基本停止更新，新机场和客户端逐步弃用。

## Trojan

Trojan 是一种基于 TLS 加密层的代理协议，设计思路是让代理流量尽量接近普通 HTTPS 流量，从而降低被 DPI（深度包检测）识别的概率。

### 工作机制

- Trojan 基于 TLS 1.3 建立连接。
- 使用域名证书伪装为普通 HTTPS 网站（如 www.google.com）。
- 流量特征接近 HTTPS，适合部分中转与前置伪装场景。

### 安全性分析

- 完整依赖 TLS 加密，安全性主要取决于证书、服务端和客户端配置。
- 可集成 Let's Encrypt 免费证书，易于部署。
- 配置合理时特征更接近常规 HTTPS，但并不代表永远无法被识别。

### Trojan 与 VLESS 的差异

| 特性       | Trojan      | VLESS       |
|------------|-------------|-------------|
| TLS伪装     | 必须        | 可选        |
| 支持CDN     | 是          | 是          |
| 加密层      | 原生TLS     | 依赖外部    |
| 性能        | 高          | 更高（XTLS）|
| 易部署性    | 中          | 一般        |

**优点：**
- 伪装思路清晰，天然贴近 HTTPS。
- 可与主流 Web 服务器共存（如 Nginx）。

**缺点：**
- 对服务器 TLS 配置要求较高。
- 初学者配置门槛较大。

## HTTP/HTTPS Proxy

传统的 HTTP/HTTPS 代理是一种较为古老的代理协议形式，主要用于 Web 请求代理传输。

### HTTP 代理介绍

- 仅支持 TCP 连接（主要为 Web 请求）。
- 不支持 UDP、流媒体、游戏等需求。
- 安全性较低，容易被监控与识别。

### HTTPS 代理介绍

- 增加 TLS 加密层，数据安全性稍强。
- 部分客户端支持 CONNECT 方法穿透 TLS。

### 与现代代理协议的对比

| 特性         | HTTP/HTTPS Proxy | VMess/VLESS/Trojan |
|--------------|------------------|--------------------|
| 加密支持     | 仅 HTTPS         | 原生或外接支持     |
| UDP支持      | 不支持           | 支持               |
| 伪装能力     | 弱               | 强                 |
| 易识别性     | 高               | 低                 |

**优点：**
- 配置简易，客户端原生支持。
- 在企业、校园网中可短期应急使用。

**缺点：**
- 缺乏安全与隐蔽性。
- 不适合长期科学上网使用。

---

## SOCKS5

SOCKS5 是一种通用型代理协议，最初由 NEC 开发，用于客户端与服务器之间转发任意协议的网络请求。

### 工作机制

- 支持 TCP 和 UDP 流量。
- 不对流量进行加密，仅做数据转发。
- 可实现基于用户名与密码的简单认证。

### 应用场景与限制

- **优点：**
    - 协议简单、兼容性高。
    - 原生支持浏览器与多种客户端。
    - 支持 UDP（如 DNS、游戏）转发。

- **缺点：**
    - 无加密与伪装机制，易被识别。
    - 不适合公共网络环境中使用。
    - 安全性完全取决于传输链路。

适用于本地转发、内网代理、中继链构建等需求，但不建议单独用于科学上网主通道。

## Reality（常见为 VLESS + Reality）

Reality 是 Xray 生态中常见的传输/伪装方案，通常与 VLESS 搭配使用。它通过 TLS 相关握手特征和 X25519 等机制降低流量异常特征，但实际效果取决于服务端配置、目标 SNI、客户端实现和网络环境。

### 协议简介

- 常见组合是 VLESS + Reality + Vision。
- 依赖 `public-key`、`short-id`、`servername`、`flow` 等参数。
- 参数填错时通常不是“慢”，而是直接无法连接。

### 对抗审查能力

- Reality 的优势是降低可识别特征，但不是“永远不可识别”。
- 已成为 2026 年许多机场和自建方案中的常见选项。
- 是否稳定仍受线路质量、IP 状态和服务端配置影响。

### 优点：

- 伪装能力较强；
- 与 VLESS/Vision 生态结合紧密；
- 在强干扰网络中常被优先考虑。

**缺点：**
- 配置复杂，对参数一致性要求高；
- 客户端需使用兼容 Reality 的内核；
- 订阅转换工具如果不支持相关字段，容易转坏。

## Hysteria / Hysteria2

Hysteria 系列是一类基于 UDP/QUIC、专为复杂网络优化的协议。当前更常见的是 **Hysteria2**（新版本），部分机场仍可能保留旧版 Hysteria 作为兼容选项。

### 协议特性

- 基于 QUIC 协议，内建 UDP 加密支持。
- 自动丢包重传与拥塞控制机制。
- 默认使用 TLS 进行传输加密。

### 性能优势

- 极低延迟、抗丢包能力强。
- 可自适应流量质量，保持稳定连接。
- 在网络波动较大的环境（如移动网络）中表现优越。

### 使用场景

- 游戏加速（如 Steam、战网等）。
- 跨境视频通话、直播推流。
- 对 UDP 需求高的 App（如 WhatsApp、Telegram）。

**注意**：需支持 UDP 的服务器端口，否则无法正常运行。

2026 年额外注意：Hysteria2 本次核对 latest 为 `app/v2.9.2`。机场端如果升级了服务端，客户端也应尽量保持较新版本，避免出现 TCP 正常但 UDP 转发异常、端口跳跃参数不兼容等问题。

## TUIC

TUIC 是近年来新兴的代理协议，主打基于 UDP 的高性能加密传输，常被视为 Hysteria2 的“竞品”。

本次核对时，TUIC 独立项目 latest 指向 `tuic-server-1.0.0`，不是高频更新型项目。普通用户更应该关注自己使用的客户端/内核是否正确实现 TUIC，而不是单独追逐协议仓库版本号。

### 协议概述

- 采用 QUIC 协议栈，内建 TLS 加密。
- 具备多路复用、0-RTT 建连等先进特性。
- 更注重稳定性与安全性。

### Hysteria2 vs TUIC

| 特性             | Hysteria2      | TUIC           |
|------------------|----------------|----------------|
| 协议基础         | UDP + TLS      | QUIC + TLS     |
| 多路复用         | 支持           | 更强           |
| 抗丢包机制       | 具备拥塞控制   | 具备拥塞控制   |
| 性能稳定性       | 取决于网络与配置 | 取决于网络与配置 |
| 隐蔽性           | 中             | 中             |

### 使用建议

- 如果你的网络 UDP/QUIC 环境好，Hysteria2/TUIC 都可能带来更好的实时体验。
- 如果校园网、公司网、路由器或运营商限制 UDP，二者都可能不如常规 Trojan/VLESS 稳。
- 不要只看协议名判断优劣，同地区、同线路、晚高峰实测更重要。

## xHTTP / gRPC / HTTP/2 等传输

这类通常不是单独的代理协议，而是 VLESS/Trojan/VMess 等协议外层使用的传输方式。

### 2026 年为什么要关注？

- 新版 Xray/mihomo 生态里，xHTTP 相关配置更常见；
- gRPC、HTTP/2、WebSocket 仍常用于 CDN、中转或伪装场景；
- 订阅转换工具和老客户端可能不认识新字段，导致导入成功但节点不可用。

### 使用建议

- 机场后台如果提供 `Mihomo / Clash.Meta` 专用订阅，优先用专用订阅；
- 旧客户端不识别 `xhttp`、`reality-opts`、`alpn`、`flow` 等字段时，先更新客户端；
- 不要手动改 YAML 里的传输字段，除非你明确知道服务端配置。

---

## Clash 配置各类协议

Clash 使用 YAML 配置文件支持多种协议组合。以下为各类型协议的配置示例与注意事项：

### 配置示例：VMess 节点

```yaml
proxies:
  - name: "VMess 节点"
    type: vmess
    server: example.com
    port: 443
    uuid: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    alterId: 0
    cipher: auto
    tls: true
    network: ws
    ws-opts:
      path: "/ws"
      headers:
        Host: example.com
```

### 配置示例：VLESS + XTLS

```yaml
  - name: "VLESS XTLS"
    type: vless
    server: example.com
    port: 443
    uuid: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    flow: xtls-rprx-vision
    tls: true
    network: tcp
    servername: example.com
```

### 配置示例：Trojan

```yaml
  - name: "Trojan TLS"
    type: trojan
    server: example.com
    port: 443
    password: "your_password"
    sni: example.com
    skip-cert-verify: false
    network: tcp
```

### 常见问题

- Reality/TUIC/Hysteria2/xHTTP 等新协议或新传输依赖“内核支持 + 客户端实现”，优先使用较新的 `mihomo` / `sing-box` 内核与对应前端。
- 同一协议在不同订阅/客户端里字段可能不一致（如 `sni/servername`、`alpn`、`flow`、`udp`、`reality-opts`），导入后建议对照机场面板参数核对。
- 注意填写正确的 `servername(SNI)`、路径（如 WS/gRPC）、以及是否需要跳过证书校验，避免 TLS handshake 失败。

## 对比总结

对各协议的综合特性进行整理，便于用户依据自身需求做出选择。

| 协议      | 加密 | UDP | 多路复用 | 伪装能力 | 兼容性 | 适用场景             |
|-----------|------|-----|----------|--------|--------|----------------------|
| VMess     | 有   | 是  | 支持     | 中     | 高     | 存量订阅、通用代理   |
| VLESS     | 无   | 是  | 支持     | 高     | 高     | TLS/Reality 场景     |
| Trojan    | TLS  | 是  | 支持     | 高     | 高     | HTTPS 伪装需求       |
| SS        | 有   | 是  | 无       | 低     | 很高   | 简洁代理、移动设备   |
| SSR       | 有   | 是  | 无       | 中     | 低     | 存量旧配置           |
| Hysteria2 | TLS  | 是  | 支持     | 中     | 中     | 游戏、实时通信       |
| TUIC      | TLS  | 是  | 强       | 中     | 中     | 高性能 UDP 传输      |
| Reality   | TLS  | 是  | 支持     | 高     | 中-高  | 抗干扰、强伪装       |
| SOCKS5    | 无   | 是  | 无       | 低     | 高     | 内网、本地中转       |
| HTTP(S)   | TLS  | 否  | 无       | 低     | 高     | 临时 Web 代理        |

## 结论

### 协议选择建议

- **新手用户**：推荐使用机场默认提供、能被主流客户端稳定导入的 Trojan / VLESS / Shadowsocks / Reality 节点，不要一开始就手改参数。
- **老用户/折腾党**：优先尝试 VLESS + Reality、TUIC、Hysteria2（前提是机场与客户端/内核支持）。
- **游戏加速**：优先选择 Hysteria2 或 TUIC，改善 UDP 丢包与抖动（效果也强依赖线路质量）。
- **轻量配置**：Shadowsocks 仍具价值，适用于移动设备或路由器。

### 后续发展趋势

- Reality 仍在普及，是 2026 年常见选项之一。
- QUIC/UDP 类协议（如 TUIC、Hysteria2）会在游戏、语音和高性能场景中继续出现，但“是否稳定”仍取决于 UDP 网络环境、机场入口/落地与运维。
- xHTTP、gRPC、Reality 相关字段更新较快，旧客户端和第三方订阅转换会更容易出兼容问题。
- 客户端生态会继续向 `mihomo` / `sing-box` 收敛：前者偏 “Clash 配置 + 规则分流”，后者偏 “多协议/多形态”。

---

## FAQs

**1. 为什么有的客户端能用 Reality/TUIC/Hysteria2，有的不能？**

本质是“内核能力”不同：有的客户端内置 `mihomo`，有的用 `sing-box`，也有的仍是老内核；同名客户端不同版本也可能差异很大。

**2. 为什么我的 VLESS 节点连接失败？**

检查 servername、UUID、端口是否正确，同时确认 TLS 是否启用，并确保服务器端配置无误。

**3. 哪些协议适合稳定科学上网抗审查？**

Reality、Trojan、VLESS + TLS 都具备较强伪装能力，是目前科学上网主流选择。

**4. Shadowsocks 还能用吗？**

可以，但在高级审查环境中容易被识别，适合轻度使用或配合插件增强。

**5. Clash 是否支持移动端？**

是。移动端常见方案包括 Android 的 `mihomo` 系客户端、以及 iOS 的 Shadowrocket/Stash/Loon/Surge 等（以客户端实际支持的协议为准）。

---

## 推荐阅读

- [机场推荐榜单 | 2026科学上网指南 ](https://petnooko.com/)
- [Clash机场常用名称解释](./mingci.md)
