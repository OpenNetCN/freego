# Clash机场专用名词解释字典（终极指南）

## 1. 引言

### Clash机场与科学上网概述

在数字化时代，全球信息高度互联，但由于网络审查或内容限制，许多互联网用户无法访问某些网站或服务。为了解决这一问题，Clash机场应运而生。Clash是一款高性能代理客户端，可通过配置远程代理节点，实现**科学上网**，绕过防火墙和地域限制。

所谓“机场”，是用户常见的对**付费代理服务提供商**的俗称，用户可以通过订阅这些机场提供的节点，实现高速、稳定、安全的翻墙体验。

### 为什么了解术语很重要

Clash及其相关配置较为专业，涉及大量技术术语，如“Vmess”、“Rule”、“Fake-IP”等。理解这些术语不仅有助于正确配置软件，也能帮助用户选择最合适的机场服务，提高上网体验并避免安全隐患。

---

## 2. Clash软件简介

### Clash是什么？

Clash 是一个基于 Go 语言编写的规则驱动代理客户端，支持多种协议（如 Vmess、Vless、Trojan、Shadowsocks）。它允许用户通过配置文件，自定义规则进行流量分流、广告过滤、协议转换等操作，是目前科学上网领域最强大的工具之一。

### Clash与Clash for Windows、Clash Verge、Clash Meta的区别

| 软件名称 | 平台 | 特点 |
|----------|------|------|
| **Clash core** | 多平台 | 基础核心，命令行运行 |
| **Clash for Windows (CFW)** | Windows | 图形化界面，适合新手 |
| **Clash Verge** | Windows/macOS/Linux | 界面美观、支持Meta内核 |
| **Clash Meta** | 多平台 | 新一代核心，支持更多协议如Reality、Hysteria2 |

用户可根据自身需求选择合适的前端界面与核心版本。

---

## 3. 节点（Node）

### 节点的定义

节点是指一条代理连接的配置数据，通过该连接，用户可将流量转发到境外服务器，实现科学上网。一个节点包含了连接的服务器地址、端口、协议、UUID等参数信息。

### 常见节点类型

- **Vmess**：由V2Ray开发，常见于许多机场，兼容性好。
- **Vless**：Vmess的轻量升级版本，去除了加密，传输更灵活。
- **Trojan**：基于HTTPS协议，伪装性强，抗封锁能力好。
- **Shadowsocks (SS)**：较早期的加密代理协议，速度快但安全性稍弱。

---

## 4. 节点参数详解

### 常见字段说明

| 字段名 | 含义 |
|--------|------|
| `server` | 节点服务器地址 |
| `port` | 服务器端口号 |
| `uuid` | 用户唯一识别码（用于Vmess/Vless） |
| `alterId` | 额外ID（旧版Vmess专属） |
| `cipher` | 加密方式（如chacha20-ietf-poly1305） |
| `tls` | 是否启用TLS加密（true/false） |

### 节点订阅链接（Subscription URL）

机场通常提供一个订阅链接，用户将该链接粘贴至Clash配置界面即可获取全部节点信息，并可自动更新。链接格式通常以 `.yaml` 或 `.txt` 结尾。

---

## 5. 代理模式（Mode）

### Global、Rule、Direct 模式解释

Clash支持三种核心代理模式：

- **Global（全局模式）**：所有流量都通过代理节点转发。适合完全绕过防火墙，但访问国内网站时可能变慢。
- **Rule（规则模式）**：依据规则集自动判断哪些流量走代理，哪些不走。最推荐的模式。
- **Direct（直连模式）**：所有流量不经过代理，常用于测试连接或访问本地网络。

### 何时使用哪种模式？

| 使用场景 | 推荐模式 |
|----------|----------|
| 看Netflix、YouTube | Global |
| 中英文网站同时访问 | Rule |
| 使用银行/本地服务 | Direct |

---

## 6. 规则集（Rule Set）

### Rule的概念

规则（Rule）是Clash的核心之一，用于判断特定流量应通过哪个节点或是否走直连。规则可以是域名、IP范围、进程名称等形式。借助规则集，Clash可以做到“智能分流”，即国内流量直连，国外流量自动代理。

### 常见规则类型

| 类型 | 描述 | 示例 |
|------|------|------|
| DOMAIN | 匹配域名 | `DOMAIN,google.com` |
| DOMAIN-SUFFIX | 匹配域名后缀 | `DOMAIN-SUFFIX,youtube.com` |
| DOMAIN-KEYWORD | 包含关键字 | `DOMAIN-KEYWORD,google` |
| IP-CIDR | 匹配IP段 | `IP-CIDR,8.8.8.8/32` |
| GEOIP | 匹配国家或地区 | `GEOIP,CN` |
| PROCESS-NAME | 匹配进程 | `PROCESS-NAME,chrome.exe` |
| MATCH | 匹配所有流量（默认规则） | `MATCH` |

建议用户使用带有规则集订阅功能的机场，以确保规则长期维护和更新。

---

## 7. DNS配置

### DNS在Clash中的作用

Clash中的DNS模块决定域名如何被解析，并直接影响“域名规则”能否正确应用。错误的DNS解析可能导致无法命中规则或连接失败。

### Fake-IP与真实DNS的区别

- **Fake-IP 模式**：Clash将未匹配到真实DNS记录的域名生成一个伪造IP，从而确保规则命中率更高。推荐用于国内外流量混合场景。
- **真实DNS模式**：通过真实DNS服务器解析域名，但对流量分析要求较高，命中率可能不如Fake-IP。

```yaml
dns:
  enable: true
  listen: 0.0.0.0:53
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - 114.114.114.114
    - tls://dns.rubyfish.cn:853
  enhanced-mode: fake-ip
```

---

## 8. 负载均衡与自动切换

### 负载均衡类型

Clash支持三种策略实现节点自动化管理：

- **URL-Test**：定期测试指定网址（如https://www.gstatic.com/generate_204）的响应速度，选择最快节点。
- **Fallback**：主节点失效后自动切换至备用节点。
- **Load-Balance**：将流量平均分配至多个节点（适合高频率分流需求）。

```yaml
proxies:
  - name: node1
    type: vmess
    server: 1.1.1.1
  - name: node2
    type: vmess
    server: 2.2.2.2

proxy-groups:
  - name: AutoGroup
    type: url-test
    proxies:
      - node1
      - node2
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
```

### 节点测速与健康检查

测速逻辑基于响应时间、丢包率和可用性判断。Clash本身无法显示详细测速图表，但借助控制面板如 Yacd 可以实时查看延迟。

---

## 9. Tun模式与系统代理

### Tun模式的用途与配置

Tun模式可将系统层级所有应用的流量“透明接管”至Clash，实现比传统HTTP/HTTPS代理更强的全局穿透能力。尤其适用于不支持手动设置代理的游戏、软件或系统级服务。

#### Clash Meta启用Tun示例：

```yaml
tun:
  enable: true
  stack: system
  dns-hijack:
    - 198.18.0.2:53
```

### 系统代理设置方式

- **Windows**：Clash for Windows 启动后自动设置系统代理。
- **macOS/Linux**：需手动设置系统网络偏好中的HTTP和Socks代理。
- **Android**：Magisk模块（如Clash for Android内核）或VPN模式实现流量接管。

---

## 10. 外部控制面板

### Clash Dashboard控制面板

Clash运行时会暴露本地控制接口（REST API + Web GUI）。常见的Dashboard有两种：

1. **CFW自带面板**（默认本地127.0.0.1:9090）
2. **Yacd 面板**：简洁易用的第三方前端，可直接接入Clash配置。

### 使用Yacd、Meta-Yacd、ClashN等工具

- **Yacd**：最轻量的Web控制面板，支持实时切换节点、查看规则。
- **Meta-Yacd**：适配Clash.Meta的Yacd改版，支持Reality协议。
- **ClashN**：Windows下的Clash客户端集成界面，适合进阶用户。

---

---

## 11. 网络协议说明

### TCP、UDP的区别

- **TCP（Transmission Control Protocol）**：面向连接、传输稳定。适合网页浏览、视频播放等对传输完整性要求高的服务。
- **UDP（User Datagram Protocol）**：无连接、传输速度快但稳定性较差。适合游戏、实时语音等低延迟需求场景。

### TLS、XTLS、gRPC、H2等传输协议解析

| 协议 | 特点 | 常用于 |
|------|------|--------|
| **TLS** | 标准加密传输层协议，支持HTTPS伪装 | Trojan, Vmess |
| **XTLS** | 特殊加密技术，提升传输性能 | Vless (Reality协议) |
| **gRPC** | 谷歌提出的高性能协议，基于HTTP/2 | Trojan/Vless中继传输 |
| **H2 (HTTP/2)** | 低延迟、高并发连接 | Trojan, Vless, Reality |
| **Reality** | 新型XTLS替代方案，具备防探测、伪装能力 | Clash Meta支持 |

---

## 12. 加密方式（Cipher）

### 常见加密算法介绍

| 加密方式 | 描述 |
|----------|------|
| `aes-128-gcm` | 高安全性，对硬件要求较低，广泛兼容 |
| `chacha20-ietf-poly1305` | 适合移动设备的高性能加密方式 |
| `none` | 无加密，适用于Vless裸协议（通过TLS保障） |

### 如何选择合适的加密方式

- 对于低功耗设备：推荐 `chacha20-ietf-poly1305`
- 对于通用设备：`aes-128-gcm` 或 `aes-256-gcm` 是稳定之选
- 使用Vless/Reality等协议时，可选择 `none`，由TLS层加密保障

---

## 13. IP类型详解

### 住宅IP（家宽IP）与数据中心IP的区别

- **住宅IP（家宽）**：来自普通家庭网络，通常不易被识别为代理，适合访问敏感服务。
- **数据中心IP**：来自IDC机房，速度稳定但容易被服务商识别、限制。

### 原生IP、非原生IP的概念

- **原生IP**：由本国ISP分配的IP，具备完整网络权限（如原生美区IP可访问美国专属服务）。
- **非原生IP**：转接或托管方式分配，部分服务可能限制访问或出现内容缺失。

### 静态IP与动态IP区别

- **静态IP**：固定不变的IP地址，适合做端口转发、自建服务。
- **动态IP**：每次重连可能改变地址，适合普通用户但不利于持久映射。

---

## 14.  线路类型与传输路径说明

不同线路决定了机场速度、稳定性、抗封锁能力。常见线路说明如下：

###  常见线路类型对比

| 类型         | 路径结构                                  | 特点                           |
|--------------|-------------------------------------------|--------------------------------|
| **直连**     | 用户 → 境外节点                           | 延迟低，易被墙封，稳定差       |
| **中转**     | 用户 → 中转节点 → 境外节点                | 性价比高，抗封锁中等           |
| **BGP中转**  | 用户 → BGP中转 → 境外节点                 | 晚高峰稳定，适配三网           |
| **IEPL专线** | 用户 → 专线内网 → 境外节点                | 延迟低，稳定高，流媒体友好     |
| **IPLC专线** | 用户 → 物理专线 → 境外节点                | 企业级稳定，不易被墙           |
| **多跳隧道** | 用户 → Socks → gRPC/WS → 境外节点         | 抗封锁强，速度一般             |
| **自建VPS**  | 用户 → 自建代理 → 网站                    | 灵活可控，需运维               |

### 🧠 常用术语简析

- **三网优化**：自动选择电信/联通/移动最优路径
- **回程优化**：提升海外访问国内资源速度
- **原生IP**：用于解锁 Netflix/ChatGPT/TikTok 等服务
- **抗封锁性**：专线 > BGP中转 > 普通中转 > 直连

### ✅ 推荐选型

- 新手：BGP中转 / 普通中转
- 重度使用：IEPL / IPLC
- 自由可控：自建VPS
- 解锁流媒体/AI工具：优先原生IP + 专线



---

## 15. 倍率（Ratio）说明

### 什么是倍率？

倍率指的是机场服务中，**一个节点所消耗流量与实际使用流量的比值**。例如1:5倍率表示用户使用1GB，计费为5GB。

### 倍率对流量计算的影响

- **倍率高**：流量消耗快但价格相对便宜
- **倍率低或1:1**：计费公平但服务更昂贵

### 常见倍率设置示例

| 区域/节点 | 倍率 | 特点 |
|-----------|------|------|
| 日本/韩国IEPL | 1:1 | 速度快，适合游戏/视频 |
| 美国/香港中转 | 1:3 ~ 1:5 | 解锁强，但流量消耗快 |
| ChatGPT/Netflix专线 | 1:10 或更高 | 高级内容节点，精准优化 |

用户应根据使用习惯和流量预算合理选择使用不同倍率的节点。

---
---

## 16. 解锁能力解析

### 什么是解锁？

“解锁”指的是使用代理节点访问**本地受限或仅限某地区可用的服务内容**。例如：Netflix美区、TikTok国际版、ChatGPT、YouTube Music、Midjourney 等。

### 支持解锁的服务类型

- **流媒体平台**：Netflix、Disney+、YouTube、HBO Max、BBC iPlayer
- **社交与视频平台**：TikTok、Instagram、Facebook
- **AI与开发平台**：OpenAI（ChatGPT）、Midjourney、Claude、GitHub Copilot、Bing AI
- **金融与电商服务**：PayPal、Amazon、Stripe、eBay（取决于节点IP归属地）

### 如何判断节点是否支持解锁？

- 使用 Clash 控制面板的“测速检测”功能，或外部工具如：
    - [Netflix Checker](https://github.com/sjlleo/netflix-verify)
    - [TikTok Region Test](https://test.ipw.cn)
    - Speedtest + IP查询看是否为原生IP/住宅IP
- 节点备注中常有标注，如 `HK(解锁TikTok)`, `US(Netflix 全解)`

### 解锁失败的常见原因与对策

| 原因 | 解决办法 |
|------|----------|
| IP非原生/被污染 | 更换节点或联系机场反馈 |
| DNS泄漏导致识别为本地 | 使用Fake-IP或DoH DNS |
| 缓存地区信息 | 清除浏览器缓存/更改语言偏好 |
| 服务已封禁该段IP | 尽量使用住宅或IEPL/IPLC线路 |

---

## 17. 常见错误与诊断

### 节点无法连接的常见原因

1. 节点配置错误（端口/UUID/cipher等参数）
2. 节点失效或被墙（国内网络封锁）
3. Clash配置文件加载异常
4. 代理规则冲突或DNS解析失败

### 日志查看与调试技巧

- Clash GUI面板通常提供日志窗口，查看节点连接状态。
- `Clash.Meta` 和 `CFW` 都支持 `verbose` 日志级别，用于排查连接细节。
- 使用 `ping`、`tracert`、`curl` 等命令测试网络连通性。
- 可查看 `/log` 或终端返回的握手信息定位问题源头。

---

## 18. 高级功能

### GeoIP与GeoSite

- **GeoIP**：基于IP地址归属国家的流量识别，例如 `GEOIP,CN` 用于识别中国大陆流量。
- **GeoSite**：基于域名规则的预设站点分类集，如 `geosite:netflix` 用于一键识别所有Netflix相关流量。

```yaml
rules:
  - GEOIP,CN,DIRECT
  - GEOSITE,netflix,US-NODE
```

### 自定义配置文件（config.yaml）技巧

- 将常用规则集和分组预设进配置文件中，无需每次手动切换
- 可定义多个 proxy-group 实现一键切换场景（如ChatGPT专用组、解锁组、游戏组）
- 结合 `rule-providers` 动态拉取远程规则集，便于维护更新

---

## 19. 安全性与隐私保护

### 如何防止DNS泄漏

DNS泄漏会暴露你的真实访问目的地，即便你使用代理，也可能让ISP知道你访问了某些网站。避免DNS泄漏的方法包括：

- 启用 Fake-IP 模式
- 配置可信的 DNS-over-HTTPS（DoH）服务器
- 设置系统 DNS 与 Clash DNS 一致，避免“操作系统直通”

### 隐私增强配置建议

- 不使用无加密或明文协议（如原始SOCKS/HTTP代理）
- 使用支持TLS/XTLS的协议（Trojan/Vless+Reality）
- 禁用日志记录功能（如配置中 `log-level: silent`）
- 选择支持“混淆”或“伪装”功能的节点（如gRPC伪装）

---

## 20. Clash机场相关术语汇总表

### 简明术语表（按字母或功能分类）

| 术语 | 含义 | 分类 |
|------|------|------|
| 节点 | 一个可用的代理连接配置 | 核心组件 |
| 订阅链接 | 获取节点信息的远程地址 | 管理机制 |
| Rule | 流量分流规则 | 配置功能 |
| TLS | 安全加密协议 | 传输层 |
| Fake-IP | Clash伪造DNS解析的机制 | 网络控制 |
| 原生IP | 被目标平台识别为真实用户IP | 解锁机制 |
| 倍率 | 节点流量计费比率 | 商业计费 |
| 中转 | 多跳传输优化机制 | 线路优化 |

### 推荐使用场景与提示

- 需要看视频推荐使用：香港、台湾、美国节点，倍率不高优先
- AI工具解锁推荐：美国原生IP+Reality协议节点
- 追求稳定可用推荐：IEPL、CN2 GIA、IPLC等中高端线路

---
---

## 21. 结语

### 持续学习Clash进阶知识

Clash不仅仅是一个翻墙工具，它是一整套高度灵活、功能强大的网络流量控制平台。随着协议发展与网络监管变化，Clash社区不断更新核心和配置方式。建议用户持续关注以下内容：

- Clash 核心更新（如 Meta、Premium 支持 Reality、Hysteria2）
- 新兴协议与加密算法（如 uTLS、gQUIC）
- 社区维护规则集和配置模板（如lhie1、ACL4SSR、DivineEngine）

### 推荐资源与社区

| 名称 | 链接 | 说明 |
|------|------|------|
| Clash GitHub | https://github.com/Dreamacro/clash | 官方核心项目 |
| Clash.Meta | https://github.com/MetaCubeX | Meta 版本维护 |
| Yacd 控制面板 | https://github.com/haishanh/yacd | Web控制前端 |
| ACL4SSR 规则集 | https://github.com/ACL4SSR/ACL4SSR | 分流规则集合 |
| Telegram 频道 | 搜索：Clash 中文频道 | 社区交流与机场评测 |
| YouTube 频道 | 搜索：科学上网 教程 Clash | 视频教学资源丰富 |

借助社区力量，保持配置更新与技术前沿，是长期稳定使用Clash机场的关键。

---

## FAQs

### 1. Clash机场和VPN有什么区别？
Clash机场基于代理协议分流流量，功能更强大、可定制，而VPN通常是全局加密隧道，适合一键使用但可控性较弱。

### 2. 为什么我的节点看不了Netflix？
可能使用的是非原生IP或被识别为代理IP，建议更换带有“解锁Netflix”标识的节点。

### 3. Clash中Fake-IP有什么风险？
Fake-IP是伪造DNS机制，不会带来安全风险，反而可避免DNS泄漏，提高分流命中率。

### 4. 什么是Reality协议？为什么它越来越流行？
Reality是基于VLESS协议的新型加密与伪装机制，具备更强抗封锁性和隐蔽性，已被大量机场采用。

### 5. 倍率越高是不是越好？
不一定。高倍率节点价格低但流量消耗快，适合轻度使用；低倍率节点适合追求体验和稳定性的用户。


## 推荐阅读

- [机场推荐榜单 | 2025科学上网指南 ](https://gptvpnhelper.com/airport-access/)

