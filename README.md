简体中文 | [繁体中文](README.zh-Hant.md) | [English](README.en.md)

# DMIT 香港 VPS 翻墙教程：用 sing-box 搭建 VLESS + REALITY 与 Hysteria2

<!-- section:intro -->
## 先说结论

本教程用一台 DMIT 香港 VPS 和官方 sing-box，搭建两条只给自己用的加密代理：一条是 VLESS + XTLS Vision + REALITY（TCP 443），另一条是 Hysteria2（UDP 443）。客户端每分钟测试一次两条线，自动使用能连通且较快的那一条。多数个人用户可以从 DMIT HKG.Pro TINY 开始：订购页标价 US$39.90/月，500 GB 双向流量，线路是 CN2 GIA 和 CMI（根据 DMIT 官方文档，查证日期 2026-11-17）。服务器用 Debian，sing-box 从官方 apt 源安装，所有命令都写在本文里，不需要运行任何第三方脚本；客户端用 iOS、Android、Windows、macOS 上的 sing-box 官方 App。没有任何协议能保证长期不被封锁，在中国大陆使用翻墙工具也有法律风险，开始前请先读「[风险提示](#风险提示)」。

文中事实最后查证于 2026-11-17（每条事实旁边注明了各自的查证日期）。文中只有 DMIT 下单链接是推广链接（[DMIT 下单链接](https://www.dmit.io/aff.php?aff=30385)），指向 DMIT 官网和文档的其他链接都是普通链接，详见「[推广链接与声明](#推广链接与声明)」。

目录：

1. [推广链接与声明](#推广链接与声明)
2. [DMIT 香港方案事实表](#dmit-香港方案事实表)
3. [怎么选方案](#怎么选方案)
4. [DMIT 允许和禁止的用法](#dmit-允许和禁止的用法)
5. [下单时选什么](#下单时选什么)
6. [需要你替换的三个值](#需要你替换的三个值)
7. [交给你的 AI agent](#交给你的-ai-agent)
8. [服务器设置](#服务器设置)
9. [客户端设置](#客户端设置)
10. [检查是否生效](#检查是否生效)
11. [日常维护](#日常维护)
12. [出问题时怎么办](#出问题时怎么办)
13. [常见问题](#常见问题)
14. [风险提示](#风险提示)
15. [更新记录](#更新记录)

<!-- section:disclosure -->
## 推广链接与声明

- 本文里的 DMIT 下单链接 https://www.dmit.io/aff.php?aff=30385 是作者的推广链接。通过它购买，作者会从 DMIT 获得佣金。按 [DMIT 推广 FAQ](https://docs.dmit.io/guide/faq/affiliate)，佣金一般是实付金额的 10%，部分高性价比产品可能降到 5%，部分促销期间可能升到 20%（根据 DMIT 官方文档，查证日期 2026-11-17）。
- 通过推广链接购买，价格是否与直接购买相同，DMIT 官方文档没有说明，本文无法确认（根据 DMIT 官方文档，查证日期 2026-11-17）。如果你不想使用推广链接，可以直接打开 [DMIT 官网](https://www.dmit.io/) 下单，本文的步骤完全一样。
- 本文是个人编写的非官方教程，与 DMIT 没有任何关联，不代表 DMIT，也不是 DMIT 的官方文档。DMIT 的规则以其官网和官方文档为准。
- 本文没有写给 AI 看的隐藏内容。「[交给你的 AI agent](#交给你的-ai-agent)」一节是公开的提示词，你可以自己读、自己改。

本文文字依 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 授权。转载或改写时，请署名并附上本文链接 https://github.com/weiexperimental/dmit-hk-vps-gfw-guide。完整授权文本见本仓库的 `LICENSE` 文件。

<!-- section:facts -->
## DMIT 香港方案事实表

下表只收录在 DMIT 官网、官方文档或官方公告里查到的内容。价格和规格随时可能变化（[TOS](https://www.dmit.io/pages/tos) 第 15 条：已签约期内价格不会上涨，但 DMIT 可以随时修改牌价和方案资源，不另行通知），下单前请以订购页为准。

| 项目 | 内容 | 来源 | 查证日期 |
|---|---|---|---|
| DMIT HKG.Pro TINY（订购页名称 HKG.AS3.Pro.TINY） | US$39.90/月，免开通费；1 vCore、1 GB 内存、20 GB SSD、1 个 IPv4、1 段 IPv6 /64；500 GB 双向流量；端口 1 Gbps | [订购页（Premium，AS3）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3) | 2026-11-17 |
| DMIT HKG.Pro STARTER（订购页名称 HKG.AS3.Pro.STARTER） | US$79.90/月，免开通费；1 vCore、2 GB 内存、40 GB SSD、1 个 IPv4、1 段 IPv6 /64；1000 GB 双向流量；端口 1 Gbps | [订购页（Premium，AS3）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3) | 2026-11-17 |
| DMIT HKG.EB（Eyeball，官方标注 Beta） | WEE：US$24.90/月，1 vCore、1 GB 内存、20 GB SSD，500 GB 双向流量，500 Mbps；TINY：US$39.90/月，1 vCore、1 GB 内存、20 GB SSD，800 GB 双向流量，1 Gbps | [订购页（Eyeball）](https://www.dmit.io/cart.php?region=hong-kong&network=eyeball) | 2026-11-17 |
| 线路（Premium，即 Pro） | 机房 Equinix HK2（葵涌）；经 CN2 GIA（AS23764）和 CMI（AS58453）；官方标示的去大陆平均延迟约 15 ms 是从香港到邻近的一个大陆城市的参考测量值，丢包低于 0.1%；官方注明只是参考值，实际取决于网络、路由和时段 | [DMIT 香港机房页](https://www.dmit.io/pages/location/hong-kong) | 2026-11-17 |
| 直连 | 官方云主机页：与中国电信（AS4809）、中国联通（AS9929）、中国移动国际（AS58807）直连 peering，Premium 另有 CN2 GIA | [DMIT 云主机页](https://www.dmit.io/pages/cloud-instance) | 2026-11-17 |
| 流量计算 | Pro 和 Eyeball 的订购页标注 Bidirectional（双向）：上传和下载都计入 | [订购页（Premium，AS3）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3)、[实例 FAQ](https://docs.dmit.io/guide/faq/instance) | 2026-11-17 |
| 超出流量 | 订购页标注「Paused / Quota Exceeded」，即暂停；流量每月自动重置，控制台显示下次重置时间；多数标准方案可以付费提前重置，费用等于该机月费；额外流量包只限 sPro 系列 | [订购页（Premium，AS3）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3)、[实例 FAQ](https://docs.dmit.io/guide/faq/instance) | 2026-11-17 |
| 退款 | 全额退款：新订单、购买不超过 3 日、流量不超过 30 GB（TOS 写明要扣支付手续费）；部分退款：新订单、购买不超过 30 日；续费单、用账户余额付款（只退回余额）、网络不满意、IP 地理位置等情况不退款 | [TOS](https://www.dmit.io/pages/tos) 第 19 条、[退款 FAQ](https://docs.dmit.io/guide/faq/refund) | 2026-11-17 |
| 更换 IP | Pro 和 EB 免费更换需同时满足：购买满 7 日、距上次更换满 15 日、整个 IP 的 ICMP 和所有 TCP 端口都被封、服务剩余至少 7 日；开通时已被封可开工单免费换。付费更换：随时可申请，每次 US$5，两次申请至少相隔 7 日，完成后不能撤销或退款 | [IP FAQ](https://docs.dmit.io/guide/faq/ip) | 2026-11-17 |
| 付款方式 | 账单 FAQ 提到 PayPal 和信用卡（需要完整的个人和地址信息）；完整的付款方式清单未能核实 | [账单 FAQ](https://docs.dmit.io/guide/faq/billing) | 2026-11-17 |
| 续费 | 月付机器在到期前 7 天出续费账单；到期后保留 3 天，3 天内付款可以恢复，之后删除且数据不能恢复 | [账单 FAQ](https://docs.dmit.io/guide/faq/billing) | 2026-11-17 |
| 下单要求 | DMIT 2026-11-14 公告：下单和重装系统（Reinstall OS）时要提供有效的 SSH 公钥；私钥改为在浏览器本地生成；客户区不再保存控制台（Console）的 root 密码 | [DMIT Telegram 公告](https://t.me/DMIT_INC/1292) | 2026-11-17 |
| IP 管理变化 | DMIT 公告：2026-12-01 起改为自购自管 IP；继续保留 154 开头 Cogent IP 的用户，从 2026-12-01 当日或之后出的续费单起每个 IPv4 加收 US$1/月；香港的 Cogent IP 会分批迁移到标准 IP，时间表另行公布；没有 154 开头 IP 的不受影响 | [DMIT Telegram 公告](https://t.me/DMIT_INC/1282) | 2026-11-17 |
| 工单回复 | TOS 只保证 72 小时内回复；工单 FAQ 写通常 24 小时内，节假日和周末可能延迟 | [TOS](https://www.dmit.io/pages/tos) 第 3.7 条、[工单 FAQ](https://docs.dmit.io/guide/faq/ticket) | 2026-11-17 |
| 规则版本 | AUP 最后更新 2025-11-12；TOS 最后更新 2026-01-22 | [AUP](https://www.dmit.io/pages/aup)、[TOS](https://www.dmit.io/pages/tos) | 2026-11-17 |

<!-- section:choose -->
## 怎么选方案

**三个系列对比**，数字来自上面的事实表（根据 DMIT 官方文档，查证日期 2026-11-17）：

| | DMIT HKG.Pro TINY | DMIT HKG.Pro STARTER | DMIT HKG.EB |
|---|---|---|---|
| 月费 | US$39.90 | US$79.90 | WEE US$24.90；TINY US$39.90 |
| 双向流量 | 500 GB | 1000 GB | WEE 500 GB；TINY 800 GB |
| 端口速度 | 1 Gbps | 1 Gbps | WEE 500 Mbps；TINY 1 Gbps |
| 内存 | 1 GB | 2 GB | 1 GB |
| 线路 | Premium：CN2 GIA + CMI | Premium：CN2 GIA + CMI | Eyeball：Tier 1 加「尽力而为」的中国路由（官方标注 Beta） |

怎么选：

- **先从 DMIT HKG.Pro TINY 开始。** 在本文列出的 Premium 线路方案里，它的月费最低（DMIT HKG.Pro STARTER 是 US$79.90，AN5 世代最便宜的 HKG.AN5.Pro.MINI 是 US$149.90）。DMIT 允许在同一系列内升级到更高配置，按新旧方案差价计费，但暂时不支持降级（根据 DMIT 官方文档，查证日期 2026-11-17），见 [实例 FAQ](https://docs.dmit.io/guide/faq/instance)。所以先买 TINY，流量不够再升级到 DMIT HKG.Pro STARTER，比一开始买大再想降级更稳妥。
- **流量用得多，或者需要 2 GB 内存，选 DMIT HKG.Pro STARTER。** 流量是 TINY 的两倍。
- **预算优先、能接受不稳定，才考虑 DMIT HKG.EB。** 官方说明香港 Eyeball 仍是 Beta，路由还在调整，暂不建议用于需要高稳定性的生产用途；它的中国路由是「尽力而为」（根据 DMIT 官方文档，查证日期 2026-11-17），见 [订购页（Eyeball）](https://www.dmit.io/cart.php?region=hong-kong&network=eyeball)、[香港机房页](https://www.dmit.io/pages/location/hong-kong)。

**双向流量怎么估：** 订购页写的是双向流量，上传和下载都算（根据 DMIT 官方文档，查证日期 2026-11-17）。代理服务器转发时，同一份数据会被计算两次：网站发给服务器一次（入），服务器再转给你一次（出）。所以粗略估算时，把你每月通过代理实际下载和上传的总量乘以 2，再和方案流量比较。换句话说，500 GB 双向流量，实际可用量大约是它的一半。你可以先看手机和电脑系统里的每月流量统计来估计自己的用量，上线后再用「[日常维护](#日常维护)」里的方法核对。

**为什么不选 Lite 和 Tier 1（T1）：**

- **Tier 1（T1）**：官方描述为「without China Optimization」，即没有针对中国大陆优化；DMIT 的 IP FAQ 还说，T1 系列新分配的 IP 不保证首次连接时在敏感地区可用，而 Premium 和 Eyeball 保证首次连接全球可用（通过工单处理）。它的价格低（例如 T1.TINY US$6.90/月、2000 GB），流量按「IN/OUT Max」计算（根据 DMIT 官方文档，查证日期 2026-11-17），见 [订购页（Tier 1）](https://www.dmit.io/cart.php?region=hong-kong&network=tier-1)、[云主机页](https://www.dmit.io/pages/cloud-instance)、[IP FAQ](https://docs.dmit.io/guide/faq/ip)。从大陆使用，主要看重的就是去大陆的线路，所以不推荐。
- **Lite**：在香港订购页的 Premium、Eyeball、Tier 1 三个筛选里，以及定价页上，都没有看到名为 Lite 的产品（根据 DMIT 官方文档，查证日期 2026-11-17），所以本文不讨论。
- **同一 Premium 组里的 AN5 世代**：最便宜的是 HKG.AN5.Pro.MINI，US$149.90/月，没有 TINY 和 STARTER（根据 DMIT 官方文档，查证日期 2026-11-17），见 [订购页（Premium，AN5）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=an5)。订购页要选 AS3 世代才能看到 TINY 和 STARTER。

### 公开测试汇总

只收录写明了测试方法和日期的第三方测试，下面都是**第三方测试，不是本文作者的测试**。截至 2026-11-15，没有找到专门测试当前 DMIT HKG.Pro TINY 或 STARTER（AS3 世代）、又写明方法和日期的第三方测试。

| 测试对象 | 结果摘要 | 方法 | 日期 | 来源 | 备注 |
|---|---|---|---|---|---|
| HKG.AN5.EB.TINY（Eyeball） | Geekbench 5 单核 2108、多核 2113；磁盘 fio 约 2.1 GB/s；到三网延迟作者评为「good」，但多数中国测速点用不了；电信回程评为优，联通和移动评为「可用」；作者建议停用 IPv6；系统信息显示 Debian 13 | Geekbench 5、fio、三网测速（截图）、到 9 个目标的 TCP traceroute、IP 解锁检测 | 发表 2026-10-07；截图日期 2026-09-19 | [hostalk.net](https://hostalk.net/posts/dmit_hkeb.html) | 文中购买链接带推广参数，文章没有明确披露；测的是 AN5 世代 Eyeball，不是 AS3 Pro TINY |
| HKG.Pro.MongKok（旧机型：1 vCPU、1 GB 内存、20 GB SSD、200 GB、150 Mbps） | 从大陆三个城市到三网的 traceroute，最后一跳延迟约 11 ms 至 49 ms；作者评电信 CN2 GIA 回程优秀；测速「基本跑满」 | 30 跳 traceroute（大陆三个城市到三网）、测速和 Geekbench 5（截图） | 发表 2024-08-07，更新 2025-11-24；截图日期 2024-08-05 | [hostalk.net](https://hostalk.net/posts/dmit_hkpro.html) | 文中购买链接带推广参数，文章没有明确披露；旧机型，只能作参考 |

<!-- section:rules -->
## DMIT 允许和禁止的用法

以下是 DMIT 可接受使用政策（[AUP](https://www.dmit.io/pages/aup)，最后更新 2025-11-12）和服务条款（[TOS](https://www.dmit.io/pages/tos)，最后更新 2026-01-22）里与本教程有关的部分摘要（根据 DMIT 官方文档，查证日期 2026-11-17）。摘要不能代替原文，请自己读一遍原文。

**允许：**

- 加密的私人代理。AUP 原文写明允许「Encrypted Private Proxy」。本教程搭的 REALITY 和 Hysteria2 就属于只给自己用的加密代理。

**禁止（与本教程有关的）：**

- **公共代理服务（Public Proxy Services）。** AUP 说这类服务可能导致 DMIT 的 IP 段或 ASN 被封。AUP 给出的判断依据是：收到其他网络关于该 IP 的垃圾邮件投诉，同时上行和下行流量相等。所以：不要把节点分享给大量的人，不要公开发布分享链接或配置文件。
- **去中国的 VPN 隧道。** AUP 禁止用服务建立去中国的 VPN 隧道，列举了 L2TP、OpenVPN、PPTP、IPSec、Cisco AnyConnect，并写明「包括但不限于」。WireGuard 也是 VPN 隧道，同样不要用 WireGuard、OpenVPN 这类 VPN 连回大陆。
- **转售。** AUP 禁止用 DMIT 的产品再分发和销售，也禁止以商业目的做二次虚拟化。不要卖节点、不要拿它做机场。
- **长时间大量占用带宽。**
- **非官方加速。** AUP 禁止使用不可接受设置的网络加速模块，举例有 FinalSpeed 和非官方 BBR（Unofficial BBR）。不要安装任何第三方加速脚本或模块。本教程的 Hysteria2 也不使用 Brutal 模式（见服务器设置）。
- **垃圾邮件。** AUP 有单独的 NO SPAM 一节。
- **其他：** 架设 TOR 出口节点、未经授权的端口扫描。
- **资源限制：** 所有产品默认保证 50% 的 CPU 使用率（另有注明除外），异常或过高的 CPU 用量可能被限制。

**违反之后**，以下内容来自 [AUP](https://www.dmit.io/pages/aup) 和 [TOS](https://www.dmit.io/pages/tos)（根据 DMIT 官方文档，查证日期 2026-11-17）：

- 视违规的严重程度，DMIT 可以暂停服务（suspension）或对 IP 做 null routing，为期 7 至 30 日，也可以立即终止服务；DMIT 还可以自行决定、有通知或没有通知地，因违反 AUP 终止你对服务的使用。
- DMIT 认为与垃圾邮件有关的账户会被立即终止。
- TOS 写明：因滥用产生的问题不退款（第 19.3 条）；账户共用、公共代理、转让、把付费资源开放给公众使用等情况，不属于可获补偿的情况（第 3.14、20.4 条）。滥用更换 IP 功能，可能被禁止使用该功能，见 [IP FAQ](https://docs.dmit.io/guide/faq/ip)。

<!-- section:order -->
## 下单时选什么

本节只说明下单时选什么，不逐项截图讲解。

- **先做一件事：** 2026-11-14 起，DMIT 下单时要提供 SSH 公钥（根据 DMIT 官方公告，查证日期 2026-11-17），见 [DMIT Telegram 公告](https://t.me/DMIT_INC/1292)。所以下单前先在自己电脑上完成「[S1 在自己电脑上生成 SSH 密钥](#s1-在自己电脑上生成-ssh-密钥)」，下单时把它打印出来的那一整行公钥（以 `ssh-ed25519` 开头）粘贴到订单要求填写公钥的地方。
- **方案：** DMIT HKG.Pro TINY，订购页依次选 Hong Kong、Premium、AS3 后显示为 HKG.AS3.Pro.TINY（根据 DMIT 官方文档，查证日期 2026-11-17）。流量需求大选 DMIT HKG.Pro STARTER；预算优先、能接受 Beta 选 DMIT HKG.EB。
- **系统：** 选订购页提供的最新 Debian 稳定版。本文的命令按 Debian 13 编写；写作时 Debian 13（trixie）是 Debian 的稳定版（[debian.org](https://www.debian.org/releases/)，查证日期 2026-11-15）。DMIT 订购页具体提供哪些 Debian 版本，本文未能核实。
- **付款周期：** 香港 Premium 和 Eyeball 订购页只显示 Monthly（月付）（根据 DMIT 官方文档，查证日期 2026-11-17）；是否有年付选项，本文未能核实。
- **付款方式：** 信用卡或 PayPal。账单 FAQ 提到：PayPal 和信用卡付款需要完整的个人和地址信息；PayPal 未验证、信用卡不支持 3D Secure、Stripe 风控拒绝，都可能导致付款失败（根据 DMIT 官方文档，查证日期 2026-11-17），见 [账单 FAQ](https://docs.dmit.io/guide/faq/billing)。
- **下单链接（推广链接）：** https://www.dmit.io/aff.php?aff=30385 。不想用推广链接：[DMIT 官网](https://www.dmit.io/)。

<!-- section:placeholders -->
## 需要你替换的三个值

本文代码里只有下面三个值需要你自己替换。其他所有值（UUID、密钥、密码、证书指纹等）都由服务器自动生成，保存在服务器的 `/etc/sing-box/guide.env`。

| 占位符 | 是什么 | 怎么填 | 例子 |
|---|---|---|---|
| `<SERVER_IP>` | 服务器的公网 IPv4 地址 | 在 DMIT 客户区或开通通知里找这台机器的 IPv4 地址（未能核实，以实际界面为准） | `203.0.113.10`（文档专用的示例地址，不要照抄） |
| `<SSH_USER>` | 登录服务器的用户名 | 看 DMIT 客户区或开通通知里是否写明（未能核实，以实际界面为准）；没有写明的话先试 `root`，不行再试 `debian` | `root` |
| `<REALITY_TARGET>` | REALITY 借用的网站域名 | 见下文 | `www.example.com`（只是格式示例，不要直接用） |

**替换规则：**

- 代码里的占位符如果放在单引号里，例如 `'<SERVER_IP>'`，替换时保留单引号，把尖括号连同里面的字一起换掉，例如 `'203.0.113.10'`。
- 忘了替换也不会悄悄写坏配置：生成密钥的那一步（S8）发现值里还有 `<` 或 `>` 就会报错停止，什么都不写；后面写配置的步骤（S9）也会因为缺少 `guide.env` 而停止。

**`<REALITY_TARGET>` 怎么选：**

REALITY 在握手时借用另一个网站的 TLS 特征，这个网站就是 `<REALITY_TARGET>`。选的时候：

1. 选一个中国大陆以外、在大陆能正常打开（没有被封锁）的网站，填它的域名，例如某个大型网站的 `www` 域名。
2. 它必须支持 TLS 1.3 和 HTTP/2。
3. 只填域名，不要加 `https://`，不要加路径或端口。

**例子：** `www.microsoft.com`、`www.apple.com`。这只是例子，不是推荐，也不是默认值，也不表示它们比别的网站更合适。2026-11-16 在本文的测试虚拟机上，S7 对这两个域名都打印了 `OK`；它们在大陆能不能正常打开（上面第 1 条），本文没有核实。不管选哪个，都要先在 S7 里检查。

这个域名还会用作 Hysteria2 自签证书里的名字，以及客户端配置里的 `server_name`。装好以后想换，按「[P4 更换借用网站](#p4-更换借用网站)」做。

**怎么在服务器上检查：** 登录服务器之后，在「[S7 检查借用网站](#s7-检查借用网站)」里运行检查命令。它在服务器上用 `openssl s_client` 连接这个网站的 443 端口，只允许 TLS 1.3，并请求 HTTP/2（ALPN `h2`），打印 `OK` 才算合格；打印 `NOT OK` 就换一个网站再试。你可以在 S7 里多试几个候选网站，挑一个合格的再继续。

<!-- section:agent -->
## 交给你的 AI agent

如果你用 Claude Code、Codex 这类 AI agent 帮你操作，可以把下面这段话连同本文一起交给它。这段话是公开写在这里的普通文字，你可以先读一遍、按需要修改；本文没有任何写给 AI 看的隐藏内容。

```text
请阅读这份 README（文件或链接由我提供），按「服务器设置」「客户端设置」「检查是否生效」三节，帮我配置一台 DMIT 香港 VPS。
服务器 IP：<SERVER_IP>
SSH 用户：<SSH_USER>
REALITY 借用网站：<REALITY_TARGET>（如果我没有填写，请按 README「需要你替换的三个值」一节的方法挑几个候选，在服务器上用 S7 检查；S7 合格之后、运行 S8 之前，先问我确认用哪一个）
要求：
1. 涉及付款或修改账户设置之前（包括 DMIT 客户区里的任何操作），先问我，得到我同意再做。
2. 只运行 README 里写明的命令，不要下载或运行任何第三方安装脚本。
3. 每一步运行后，对照这一步的「预期结果」；不一致时按「出错时」处理，处理不了就停下来，把输出告诉我。
4. 服务器上的代码块要以 root 身份运行。通过 SSH 非交互执行时，按 README「服务器设置」开头「怎么运行这些命令」的做法：不运行 S3；把之后每个在服务器上运行的代码块原样保存成一个文件（只替换占位符；UTF-8 编码、不带 BOM、LF 换行），在 macOS 或 Linux 终端里用 ssh <SSH_USER>@<SERVER_IP> sudo bash -s < 文件名 按顺序一个一个运行（用 root 登录时去掉 sudo；Windows 上用 WSL 或 Git Bash，本文未测试）。第一次 SSH 连接这台服务器之前先问我，由我先自己做一次 S2。
5. 不要把 /etc/sing-box/guide.env、/root/dmit-guide/client-profile.json、/root/dmit-guide/share-links.txt 的内容发到任何公开的地方。
6. 不要做 DMIT AUP 禁止的事，例如公共代理、转售、去中国的 VPN 隧道、非官方加速。
```

<!-- section:server -->
## 服务器设置

**要做什么：** 在服务器上装好 sing-box，开两条线：VLESS + XTLS Vision + REALITY 监听 TCP 443，Hysteria2 监听 UDP 443。最后生成一份客户端配置文件和两条分享链接。

**为什么是这两种协议：** 有博主根据个人经验把 VLESS + REALITY 排为最难检测、最稳定，Hysteria2 速度最快，但作者说明没有做正式测试（[lilting.ch](https://lilting.ch/en/articles/china-vpn-protocol-comparison)，2026-01-08，更新 2026-05-21）。2026-10-26 前后有用户报告大量节点同时失效；一位博主自己的 9 台 VPS 有 8 台下线，他观察到 Hysteria2 在这一轮表现不错，但只是目测，没有做多地区、多运营商测试（[elmagnifico.tech](https://elmagnifico.tech/2026/10/27/VPN-Compare/)，2026-10-27）。两条线一条走 TCP、一条走 UDP，一条不通时客户端会自动用另一条；但它们也可能同时失效，见「[风险提示](#风险提示)」。

**版本：** 写作时 sing-box 最新稳定版是 1.14.4（[GitHub release](https://github.com/SagerNet/sing-box/releases/tag/v1.14.4)，查证日期 2026-11-17）。本文的 Hysteria2 证书固定方式要求客户端的 sing-box 不低于 1.13.0（[sing-box TLS 文档](https://sing-box.sagernet.org/configuration/shared/tls/)）。本文的命令按 Debian 13 编写，不依赖 CPU 架构。2026-11-16 在两台 Debian 13（arm64）虚拟机之间测试过：服务器设置的全部命令（用 root 和用普通用户加 sudo 各一次，也测了重复粘贴、忘记替换占位符、机器上原来已有防火墙规则和重启），以及客户端配置文件里两条线各自能用、一条被挡住后自动换到另一条。测试时客户端用的是 Linux 上的 sing-box 命令行，并把 `tun` 换成了本机代理端口。同一天还让一个只拿到本文的 AI agent，在一台全新的 Debian 13（arm64）测试虚拟机上照着本文修改前的版本做完了服务器设置，本文已按它的反馈修改；修改后的命令（包括 S4 的内核检查和非交互的运行方法）是用脚本重新测试的，没有再让 agent 照着做一次。这些步骤没有在 DMIT 的机器上、amd64（x86-64）架构的机器上，或者 DMIT 自己提供的 Debian 系统映像上运行过；手机和电脑上的 App 没有测试，也没有在中国大陆测试。

**怎么运行这些命令：**

- 标着「在你自己的电脑上运行」的代码块在你自己的电脑上运行；其余代码块都在服务器上运行。
- 从 S3 开始，服务器上的代码块都要以 root 身份运行。手动操作时，按 S3 切换到 root，然后在同一个窗口里按顺序粘贴。
- 用脚本或 AI agent 通过 SSH 非交互执行时：
  - 不运行 S3（它只在交互的终端窗口里有用）。之后每个在服务器上运行的代码块，按顺序一个一个运行。
  - 把代码块的内容原样保存成一个文件，只替换占位符，其他一个字都不改，例如保存为 `s05.sh`。文件要用 UTF-8 编码（不带 BOM）和 LF 换行。用 CRLF 换行（Windows 上常见）时，服务器上的 bash 会把每行末尾的回车符当成命令的一部分，代码块不能正确运行。
  - 在你自己电脑上的 macOS 或 Linux 终端里运行 `ssh <SSH_USER>@<SERVER_IP> sudo bash -s < s05.sh`（用 root 登录时去掉 `sudo`）。这样服务器运行的，就是手动粘贴时的同一段命令。Windows 上可以在 WSL 或 Git Bash 里运行这条命令，本文未测试。
  - 第一次连接这台服务器时，SSH 要人确认服务器的指纹（见 S2）。没有终端窗口的 agent 没法确认，SSH 会报 `Host key verification failed.` 并退出。所以请你先自己做一次 S2，或者让 agent 在第一次连接之前先问你。
  - 退出码不是 0，表示这一步没有完成；退出码是 0，也要对照「预期结果」。
- 每个代码块都可以单独粘贴，也可以重复粘贴：已经生成的密钥不会重新生成，已经加过的防火墙规则不会重复添加。
- 很多代码块外面有一对圆括号 `( … )`。它的作用是：代码块中途出错时只停止这一段，不会关掉你的终端窗口。代码块打印 `ERROR:` 并停止，表示这一步没有完成，按「出错时」处理后再粘贴同一个代码块即可。

**会在服务器上生成的文件：**

| 文件 | 内容 |
|---|---|
| `/etc/sing-box/guide.env` | 所有自动生成的值（权限 600，等于你的钥匙，不要公开） |
| `/etc/sing-box/hy2.crt`、`/etc/sing-box/hy2.key` | Hysteria2 用的自签证书和私钥 |
| `/etc/sing-box/config.json` | 服务器配置 |
| `/root/dmit-guide/client-profile.json` | 客户端配置文件（导入 App 用；目录权限 700，文件权限 600） |
| `/root/dmit-guide/share-links.txt` | `vless://` 和 `hysteria2://` 分享链接（权限 600） |
| `/usr/local/sbin/sing-box-guide-firewall` | 防火墙脚本 |

<!-- step:server-01-ssh-key -->
### S1 在自己电脑上生成 SSH 密钥

**目的**：生成一对 SSH 密钥。下单时把公钥交给 DMIT，2026-11-14 起下单要提供公钥（根据 DMIT 官方公告，查证日期 2026-11-17），之后用私钥登录服务器，不用密码。**这一步在下单之前做。**

**命令**（在你自己的电脑上运行，macOS 和 Linux 用「终端」）：

```bash local
mkdir -p ~/.ssh && chmod 700 ~/.ssh
[ -f ~/.ssh/id_ed25519 ] || ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -C dmit-hk
cat ~/.ssh/id_ed25519.pub
```

`ssh-keygen` 会问你要不要设置密码短语（passphrase）：设置了更安全，以后每次登录要输入；直接按回车则留空。如果电脑上已经有 `~/.ssh/id_ed25519`，这段命令不会覆盖它，直接沿用。

Windows 10 和 11 自带 OpenSSH 客户端，但上面的写法是 bash。Windows 上请打开 PowerShell，运行 `ssh-keygen -t ed25519 -C dmit-hk`（按回车接受默认路径），再运行 `Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub` 查看公钥。

**预期结果**：最后打印一行以 `ssh-ed25519 ` 开头、以 ` dmit-hk` 结尾的文字（Windows 上同样以 ` dmit-hk` 结尾）。这一整行就是公钥，下单时粘贴进去。目录里没有 `.pub` 后缀的 `id_ed25519` 是私钥，永远不要发给任何人。

**出错时**：

- 提示 `ssh-keygen: command not found`：macOS 和主流 Linux 发行版都自带 OpenSSH；Windows 请在「设置 → 可选功能」里添加「OpenSSH 客户端」。
- 不记得已有密钥的密码短语：把 `id_ed25519` 换一个文件名（例如 `-f ~/.ssh/id_ed25519_dmit`）重新生成，之后登录时用 `ssh -i ~/.ssh/id_ed25519_dmit …` 指定它。

<!-- step:server-02-login -->
### S2 用 SSH 登录服务器

**目的**：DMIT 开通后，从自己电脑登录服务器。

**命令**（在你自己的电脑上运行，先替换 `<SSH_USER>` 和 `<SERVER_IP>`）：

```bash local
ssh <SSH_USER>@<SERVER_IP>
```

**预期结果**：第一次连接会问 `Are you sure you want to continue connecting (yes/no/[fingerprint])?`，输入 `yes` 回车。之后出现服务器的命令提示符：root 用户以 `#` 结尾，普通用户（例如 `debian`）以 `$` 结尾。

**出错时**：

- `Permission denied (publickey)`：用户名不对（换 `root` 或 `debian` 再试），或者下单时填的不是这台电脑上的公钥。重装系统时也要提供公钥（根据 DMIT 官方公告，查证日期 2026-11-17），可以在客户区用正确的公钥重装系统（未能核实，以实际界面为准）。
- `Connection timed out` 或一直没有反应：检查 IP 是否填对、机器是否已经开通并开机；如果之前能连、突然连不上，见「[P2 判断 IP 是否被封并更换 IP](#p2-判断-ip-是否被封并更换-ip)」。
- 私钥不在默认路径：加 `-i 私钥路径`，例如 `ssh -i ~/.ssh/id_ed25519_dmit <SSH_USER>@<SERVER_IP>`。
- 重装系统之后再连接，如果出现 `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`：重装可能会换掉服务器的主机密钥（指纹也随之改变），和你电脑上保存的不一致。确认这台机器是你自己刚重装的，在自己电脑上运行 `ssh-keygen -R <SERVER_IP>`，再重新做 S2。

<!-- step:server-03-root -->
### S3 切换到 root

**目的**：之后所有服务器命令都以 root 身份运行，这样不管你是用 `root` 还是 `debian` 这类普通用户登录，后面的代码块都一样，也不需要在每条命令前加 `sudo`。

**命令**（在服务器上运行）：

```bash server
[ "$(id -u)" -eq 0 ] || sudo -i
```

已经是 root 时这条命令什么都不做；普通用户会切换成 root（如果要求输入密码，输入这个普通用户的密码）。之后的代码块都在这个窗口里粘贴。断开重连后，要重新做 S2 和 S3。

**预期结果**：命令提示符以 `#` 结尾；输入 `whoami` 会显示 `root`。

**出错时**：

- `sudo: command not found`：说明你登录的是没有 sudo 的普通用户，请改用 `root` 登录。
- `is not in the sudoers file`：这个普通用户没有管理员权限，请改用 `root` 登录，或开工单问 DMIT 默认用户是哪一个。
- 用脚本或 AI agent 非交互执行：这一步无法切换后续命令的身份，不要运行它。按「服务器设置」开头「怎么运行这些命令」的做法，用 `sudo bash -s` 一个一个运行之后的代码块。如果提示 `sudo: a terminal is required to read the password`，说明这个用户使用 sudo 要输入密码，非交互执行时没法输入，请改用 root 登录。

<!-- step:server-04-update -->
### S4 更新系统并安装基础工具

**目的**：把系统更新到最新，并安装后面要用的工具：`curl`、`ca-certificates`、`openssl`、`nftables`（防火墙）、`qrencode`（在终端里显示二维码）和 `sudo`（C1 从电脑下载配置文件时要用，用 root 登录时也一样）。这些都来自 Debian 自己的软件源。

**命令**（在服务器上以 root 运行）：

```bash server
(
set -eu
export DEBIAN_FRONTEND=noninteractive
apt-get update
apt-get -y -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold full-upgrade
apt-get -y install ca-certificates curl openssl nftables qrencode sudo
running=$(uname -r)
newest=$(ls /boot/vmlinuz-* 2>/dev/null | sed 's|^/boot/vmlinuz-||' | sort -V | tail -n 1)
if [ -z "$newest" ]; then
  echo "Cannot tell whether a reboot is needed: no /boot/vmlinuz-* found."
elif [ "$newest" = "$running" ]; then
  echo "No reboot needed: the running kernel $running is the newest installed kernel."
else
  echo "Reboot recommended: the running kernel $running is not the newest installed kernel ($newest)."
fi
)
```

`--force-confold` 的意思是：如果某个软件包的配置文件被改过，保留现在的版本，不弹出询问。最后几行比较正在运行的内核（`uname -r`）和 `/boot` 里版本最新的内核，判断要不要重启。

**预期结果**：前面没有以 `E:` 开头的错误，最后一行是下面三种之一：

- `No reboot needed: …`：正在运行的就是最新的内核，直接做 S5。
- `Reboot recommended: …`：这次装了新内核，重启后才会用上。运行 `reboot` 重启（SSH 连接会断开），等一两分钟后重新登录服务器（手动操作时重新做 S2 和 S3；非交互执行时等到 SSH 能重新连上），再做 S5。也可以先不重启，做完后面的步骤再重启：本文设置的防火墙和 sing-box 都会开机自动启动。
- `Cannot tell whether a reboot is needed: …`：`/boot` 里找不到内核文件，这个检查没法判断。可以直接做 S5。

这里不用 `/var/run/reboot-required` 判断：在本文的测试虚拟机上，这个文件是 `unattended-upgrades` 软件包在内核更新后创建的，卸载这个软件包后，内核更新不再创建它；DMIT 的系统映像有没有装这个软件包，本文未能核实。

**出错时**：

- `Could not get lock /var/lib/dpkg/lock-frontend`：系统正在后台自动更新，等几分钟后再粘贴一次。
- `Temporary failure resolving`：服务器暂时解析不了域名，过一会儿再试；一直不行就开工单问 DMIT。
- `E: Sub-process /usr/bin/dpkg returned an error code (1)`：在服务器上以 root 运行 `dpkg --configure -a`，按屏幕上的提示操作，完成后再粘贴一次 S4。

<!-- step:server-05-firewall -->
### S5 设置防火墙

**目的**：只放行 22/tcp（SSH）、443/tcp（REALITY）和 443/udp（Hysteria2），并且不删除机器上原有的任何防火墙规则。

这一步做三件事：

1. 先确认 SSH 在 22 端口监听。如果不是，直接报错停止，免得把你自己挡在门外。
2. 写一个小脚本 `/usr/local/sbin/sing-box-guide-firewall`，并马上运行一次：
   - 机器上**没有**入站默认拒绝（policy drop）的防火墙时：新建一张只属于本教程的表 `inet sing_box_guide`。入站默认拒绝，只放行已建立的连接、本机回环、ICMP、DHCP 回包，以及 22/tcp、443/tcp、443/udp。
   - 机器上**已经有**入站链时：在每条原有入站链的最前面加三条放行规则（已建立的连接、22 和 443/tcp、443/udp），规则带注释 `sing-box-guide`。原有规则一条都不删、不改。如果原有防火墙是入站默认拒绝的，就不再新建表。之所以要加到原有的链里：nftables 里同一个入站钩子上的每一条基础链都放行，数据包才能通过；只在一张新表里放行，挡不住旧表的拒绝。
   - 原有的入站链如果是 iptables（Debian 默认的 nf_tables 后端）管理的 `filter` 表 `INPUT` 链，并且 `iptables -S INPUT` 读得出这条链，就改用 `iptables`／`ip6tables` 命令加四条放行规则（已建立的连接、22/tcp、443/tcp、443/udp，同样带注释 `sing-box-guide`）。直接用 `nft` 往这种链里加规则的话，`iptables-save` 会什么都不输出，`netfilter-persistent save` 会保存成空的规则文件，重启后原有规则就没了。`iptables -S INPUT` 读不出的链（例如你自己用 nft 写的、iptables 不认识的规则），仍然用 `nft` 加上面那三条规则。
   - 重复运行不会重复添加规则。
3. 写一个 systemd 服务 `sing-box-guide-firewall.service`，每次开机时在网络启动之前再运行这个脚本。nftables 规则默认不会自动保存，重启后就没了。系统自带的 `nftables.service` 被重启、重新加载，或者停止后再启动时，会清空所有规则，这个服务会跟着再运行一次脚本。只有一种情况不会自动恢复：`nftables.service` 原来没有运行，第一次被启动。这种情况，以及规则被别的程序清掉的情况，可以用「[P1 连不上时逐项检查](#p1-连不上时逐项检查)」发现并恢复。

可选：DMIT 客户区不再保存控制台（Console）的 root 密码（根据 DMIT 官方公告，查证日期 2026-11-17）。如果你想在 SSH 出问题时还能从控制台登录（未能核实，以实际界面为准），先在服务器上以 root 运行 `passwd` 给 root 设一个密码。请用一个长的随机密码（例如运行 `openssl rand -base64 18` 生成后自己保存好）。可以先在服务器上以 root 运行 `sshd -T | grep -E '^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication) '` 查看：如果显示 `permitrootlogin yes`，同时有 `passwordauthentication yes` 或 `kbdinteractiveauthentication yes`，别人也能用这个密码通过 SSH 尝试登录 root；如果显示 `permitrootlogin without-password`（也就是 `prohibit-password`），root 不能用密码通过 SSH 登录。

**命令**（在服务器上以 root 运行）：

```bash server
(
set -eu
if ! ss -Htln 'sport = :22' | grep -q .; then
  echo "ERROR: nothing listens on TCP port 22. This firewall only allows SSH on port 22, so it stops here." >&2
  exit 1
fi
cat > /usr/local/sbin/sing-box-guide-firewall <<'EOF'
#!/bin/sh
# Allow 22/tcp, 443/tcp and 443/udp. Never deletes or changes rules that are not its own.
set -eu
TABLE=sing_box_guide
TAG=sing-box-guide
chains=$(nft list chains | awk -v own="$TABLE" '
  $1 == "table" { fam = $2; tbl = $3 }
  $1 == "chain" { chn = $2 }
  $1 == "type" && $2 == "filter" && $4 == "input" && tbl != own && fam ~ /^(inet|ip|ip6)$/ {
    print fam, tbl, chn, ($0 ~ /policy drop/ ? "drop" : "accept")
  }')
printf '%s\n' "$chains" | while read -r fam tbl chn pol; do
  [ -n "$fam" ] || continue
  if [ "$fam" != inet ] && [ "$tbl" = filter ] && [ "$chn" = INPUT ] && iptables -V 2>/dev/null | grep -q nf_tables; then
    ipt=iptables
    [ "$fam" = ip ] || ipt=ip6tables
    if rules=$("$ipt" -S INPUT 2>/dev/null); then
      if printf '%s\n' "$rules" | grep -q -- "$TAG"; then continue; fi
      "$ipt" -I INPUT 1 -p udp --dport 443 -m comment --comment "$TAG" -j ACCEPT
      "$ipt" -I INPUT 1 -p tcp --dport 443 -m comment --comment "$TAG" -j ACCEPT
      "$ipt" -I INPUT 1 -p tcp --dport 22 -m comment --comment "$TAG" -j ACCEPT
      "$ipt" -I INPUT 1 -m conntrack --ctstate ESTABLISHED,RELATED -m comment --comment "$TAG" -j ACCEPT
      echo "Added accept rules with $ipt to existing chain: $fam $tbl $chn (policy $pol)"
      continue
    fi
  fi
  if nft list chain "$fam" "$tbl" "$chn" | grep -q "$TAG"; then continue; fi
  nft insert rule "$fam" "$tbl" "$chn" udp dport 443 accept comment "\"$TAG\""
  nft insert rule "$fam" "$tbl" "$chn" tcp dport '{ 22, 443 }' accept comment "\"$TAG\""
  nft insert rule "$fam" "$tbl" "$chn" ct state established,related accept comment "\"$TAG\""
  echo "Added accept rules to existing chain: $fam $tbl $chn (policy $pol)"
done
if printf '%s\n' "$chains" | grep -q ' drop$'; then
  echo "An existing firewall with policy drop was found. Rules were added to it; no new table was created."
  exit 0
fi
nft -f - <<'NFT'
table inet sing_box_guide
delete table inet sing_box_guide
table inet sing_box_guide {
  chain input {
    type filter hook input priority filter; policy drop;
    ct state established,related accept
    ct state invalid drop
    iifname "lo" accept
    meta l4proto { icmp, ipv6-icmp } accept
    udp sport 67 udp dport 68 accept
    udp sport 547 udp dport 546 accept
    tcp dport { 22, 443 } accept
    udp dport 443 accept
  }
}
NFT
echo "Firewall table inet sing_box_guide is active: inbound allows only 22/tcp, 443/tcp and 443/udp."
EOF
chmod 755 /usr/local/sbin/sing-box-guide-firewall
cat > /etc/systemd/system/sing-box-guide-firewall.service <<'EOF'
[Unit]
Description=Allow 22/tcp, 443/tcp and 443/udp for sing-box
DefaultDependencies=no
Wants=network-pre.target
After=local-fs.target nftables.service netfilter-persistent.service ufw.service
Before=network-pre.target sing-box.service shutdown.target
Conflicts=shutdown.target
PartOf=nftables.service
ReloadPropagatedFrom=nftables.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/sbin/sing-box-guide-firewall
ExecReload=/usr/local/sbin/sing-box-guide-firewall

[Install]
WantedBy=multi-user.target nftables.service
EOF
systemctl daemon-reload
systemctl enable sing-box-guide-firewall.service
systemctl restart sing-box-guide-firewall.service
journalctl -u sing-box-guide-firewall.service -n 5 --no-pager -o cat
systemctl is-active sing-box-guide-firewall.service
)
```

**预期结果**：最后一行是 `active`。前面几行会包含下面一种或两种：

- `Firewall table inet sing_box_guide is active: inbound allows only 22/tcp, 443/tcp and 443/udp.`（机器原来没有入站默认拒绝的防火墙）；
- `Added accept rules to existing chain: …`，或者 `Added accept rules with iptables to existing chain: …` 和 `Added accept rules with ip6tables to existing chain: …`，以及 `An existing firewall with policy drop was found. …`（机器原来就有防火墙）。

你当前的 SSH 连接不会断开。可以用 `nft list ruleset` 查看全部规则；机器用 iptables 管防火墙时，用 `iptables -S INPUT` 和 `ip6tables -S INPUT` 查看。

**出错时**：

- `ERROR: nothing listens on TCP port 22`：你的 SSH 不在 22 端口。本教程只放行 22，请先确认 SSH 端口（`ss -tlnp | grep ssh`），不要在这种情况下继续这一步。
- 粘贴后 SSH 断开、连不回去：从 DMIT 客户区的控制台登录（未能核实，以实际界面为准），这需要你之前设过 root 密码；登录后运行 `systemctl disable --now sing-box-guide-firewall.service` 和 `nft delete table inet sing_box_guide`，然后把情况和 `nft list ruleset` 的输出记下来再排查。
- 你用 ufw 或 firewalld 管理防火墙：它们重新加载时会覆盖本脚本加进去的规则。请改用它们自己的命令放行 22/tcp、443/tcp、443/udp（例如 `ufw allow 443/udp`）。

<!-- step:server-06-install -->
### S6 添加 sing-box 官方软件源并安装

**目的**：按 sing-box 官方文档，用官方 apt 源安装 sing-box（[sing-box 安装文档](https://sing-box.sagernet.org/installation/package-manager/)，查证日期 2026-11-15）。不使用任何安装脚本。

软件包会附带一个示例 `/etc/sing-box/config.json`，它不是本教程的配置。为免服务在 S9 之前误用它启动，这里把它改名为 `/etc/sing-box/config.json.example`（只在它确实是这个示例文件时才改名）。改名后文件不再以 `.json` 结尾，服务不会读取它。

**命令**（在服务器上以 root 运行）：

```bash server
(
set -eu
install -d -m 755 /etc/apt/keyrings
curl -fsSL https://sing-box.app/gpg.key -o /etc/apt/keyrings/sagernet.asc
chmod 644 /etc/apt/keyrings/sagernet.asc
cat > /etc/apt/sources.list.d/sagernet.sources <<'EOF'
Types: deb
URIs: https://deb.sagernet.org/
Suites: *
Components: *
Enabled: yes
Signed-By: /etc/apt/keyrings/sagernet.asc
EOF
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get -y install sing-box
if grep -qs '"shadowsocks"' /etc/sing-box/config.json; then
  mv /etc/sing-box/config.json /etc/sing-box/config.json.example
fi
sing-box version
)
```

**预期结果**：最后打印 `sing-box version 1.14.4`（或更新的版本号）和编译信息。服务这时还没有启动，这是正常的。

**出错时**：

- `curl: (6) Could not resolve host` 或下载超时：服务器暂时连不上 sing-box.app，过一会儿再粘贴一次。
- `NO_PUBKEY` 或 `The following signatures couldn't be verified`：密钥文件没下载完整，再粘贴一次这个代码块。
- `Unable to locate package sing-box`：`apt-get update` 没有读到 sagernet 源，检查 `/etc/apt/sources.list.d/sagernet.sources` 的内容是否与上面一致。

<!-- step:server-07-target -->
### S7 检查借用网站

**目的**：确认你选的 `<REALITY_TARGET>` 从服务器访问时支持 TLS 1.3 和 HTTP/2。选法见「[需要你替换的三个值](#需要你替换的三个值)」。

**命令**（在服务器上以 root 运行，先替换 `<REALITY_TARGET>`，保留两边的单引号）：

```bash server
(
T='<REALITY_TARGET>'
out=$(timeout 15 openssl s_client -connect "$T:443" -servername "$T" -tls1_3 -alpn h2 </dev/null 2>&1) || true
if printf '%s\n' "$out" | grep -q '^New, TLSv1\.3' && printf '%s\n' "$out" | grep -q '^ALPN protocol: h2'; then
  echo "OK: $T supports TLS 1.3 and HTTP/2"
else
  printf '%s\n' "$out" | tail -n 3 >&2
  echo "NOT OK: $T does not offer TLS 1.3 with HTTP/2, or this server cannot reach it. Choose another site." >&2
  exit 1
fi
)
```

**预期结果**：打印 `OK: 你的域名 supports TLS 1.3 and HTTP/2`。

**出错时**：

- 打印 `NOT OK`：这个网站不支持 TLS 1.3 或 HTTP/2，或者从服务器连不上。换一个候选网站，改掉单引号里的域名，再粘贴一次。
- `NOT OK` 前面出现 `nodename nor servname provided` 或 `Name or service not known`：域名写错了，或者你还没有把 `<REALITY_TARGET>` 换成真实域名。

<!-- step:server-08-env -->
### S8 生成 guide.env 和 Hysteria2 证书

**目的**：一次性生成本机所有密钥和密码，保存到 `/etc/sing-box/guide.env`（权限 600）。如果这个文件已经存在，这一步什么都不改，所以重复粘贴不会让已经导入的客户端失效。

`guide.env` 里的值：

| 名称 | 内容 |
|---|---|
| `SERVER_IP` | 你填的服务器 IPv4 |
| `REALITY_TARGET` | 你填的借用网站域名 |
| `UUID` | REALITY 用户 ID |
| `REALITY_PRIVATE_KEY`、`REALITY_PUBLIC_KEY` | REALITY 密钥对（私钥只留在服务器，公钥给客户端） |
| `SHORT_ID` | REALITY short ID |
| `HY2_PASSWORD` | Hysteria2 密码 |
| `HY2_CERT_SHA256` | 整张 Hysteria2 证书的 SHA-256 指纹（十六进制），给 `hysteria2://` 分享链接的 `pinSHA256` 用 |
| `HY2_PUBKEY_SHA256` | Hysteria2 证书公钥的 SHA-256（base64），给 sing-box 客户端的 `certificate_public_key_sha256` 用 |

**Hysteria2 证书怎么处理：** Hysteria2 必须有 TLS 证书。这里生成一张自签证书（有效期 10 年），不需要域名。客户端默认不信任自签证书，所以要「固定」（pin）它：sing-box 客户端配置用 `certificate_public_key_sha256`，填证书公钥的 SHA-256（base64），sing-box 1.13.0 起支持（[sing-box TLS 文档](https://sing-box.sagernet.org/configuration/shared/tls/)）；`hysteria2://` 分享链接用 `pinSHA256` 参数，按 Hysteria 官方的 URI 格式填服务器证书的 SHA-256 指纹（[Hysteria URI 格式](https://v2.hysteria.network/docs/developers/URI-Scheme/)）；分享链接为什么同时带 `insecure=1`，见 S11 末尾的说明。这两个值不一样，都在这一步算好。sing-box 客户端配置不用 `insecure: true`，因为它会接受任何证书；也不用 `certificate_sha256`，因为它要求 sing-box 1.15.0，比写作时的稳定版新。证书里的名字用你的借用网站域名，客户端的 `server_name` 也用它。

**命令**（在服务器上以 root 运行，先替换 `<SERVER_IP>` 和 `<REALITY_TARGET>`，保留单引号）：

```bash server
(
set -euo pipefail
SERVER_IP='<SERVER_IP>'
REALITY_TARGET='<REALITY_TARGET>'
ENV=/etc/sing-box/guide.env
if [ -e "$ENV" ]; then
  echo "$ENV already exists. Nothing was changed. To change one value, edit that line in $ENV."
  exit 0
fi
case "$SERVER_IP$REALITY_TARGET" in
  *'<'*|*'>'*)
    echo "ERROR: replace <SERVER_IP> and <REALITY_TARGET> in this block with your own values, then paste it again." >&2
    exit 1 ;;
esac
printf '%s\n' "$SERVER_IP" | grep -Eqx '[0-9]{1,3}(\.[0-9]{1,3}){3}' || { echo "ERROR: SERVER_IP must be an IPv4 address such as 203.0.113.10" >&2; exit 1; }
printf '%s\n' "$REALITY_TARGET" | grep -Eqx '[A-Za-z0-9-]+(\.[A-Za-z0-9-]+)+' || { echo "ERROR: REALITY_TARGET must be a host name only, without https:// or a path" >&2; exit 1; }
command -v sing-box >/dev/null || { echo "ERROR: sing-box is not installed. Do step S6 first." >&2; exit 1; }
out=$(timeout 15 openssl s_client -connect "$REALITY_TARGET:443" -servername "$REALITY_TARGET" -tls1_3 -alpn h2 </dev/null 2>&1) || true
if ! grep -q '^New, TLSv1\.3' <<<"$out" || ! grep -q '^ALPN protocol: h2' <<<"$out"; then
  echo "ERROR: $REALITY_TARGET does not offer TLS 1.3 with HTTP/2, or this server cannot reach it. Check it with S7 and choose another site." >&2
  exit 1
fi
umask 077
openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:prime256v1 -nodes -days 3650 \
  -subj "/CN=$REALITY_TARGET" -addext "subjectAltName=DNS:$REALITY_TARGET" \
  -keyout /etc/sing-box/hy2.key -out /etc/sing-box/hy2.crt
KEYPAIR=$(sing-box generate reality-keypair)
REALITY_PRIVATE_KEY=$(printf '%s\n' "$KEYPAIR" | awk '/PrivateKey/ {print $2}')
REALITY_PUBLIC_KEY=$(printf '%s\n' "$KEYPAIR" | awk '/PublicKey/ {print $2}')
UUID=$(sing-box generate uuid)
SHORT_ID=$(sing-box generate rand 8 --hex)
HY2_PASSWORD=$(sing-box generate rand 16 --hex)
HY2_CERT_SHA256=$(openssl x509 -in /etc/sing-box/hy2.crt -outform der | sha256sum | cut -d ' ' -f 1)
HY2_PUBKEY_SHA256=$(openssl x509 -in /etc/sing-box/hy2.crt -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64)
{
  echo "SERVER_IP=$SERVER_IP"
  echo "REALITY_TARGET=$REALITY_TARGET"
  echo "UUID=$UUID"
  echo "REALITY_PRIVATE_KEY=$REALITY_PRIVATE_KEY"
  echo "REALITY_PUBLIC_KEY=$REALITY_PUBLIC_KEY"
  echo "SHORT_ID=$SHORT_ID"
  echo "HY2_PASSWORD=$HY2_PASSWORD"
  echo "HY2_CERT_SHA256=$HY2_CERT_SHA256"
  echo "HY2_PUBKEY_SHA256=$HY2_PUBKEY_SHA256"
} > "$ENV.tmp"
if grep -q '^[A-Z0-9_]*=$' "$ENV.tmp"; then
  rm -f "$ENV.tmp"
  echo "ERROR: a generated value is empty, so $ENV was not written. Read the messages above, then paste this block again." >&2
  exit 1
fi
mv "$ENV.tmp" "$ENV"
chmod 600 "$ENV"
echo "Created $ENV:"
cat "$ENV"
)
```

**预期结果**：第一次运行时可能先打印一行 `-----`（openssl 的输出），然后是 `Created /etc/sing-box/guide.env:` 和 9 行 `名称=值`，每一行等号后面都有内容。再次粘贴时打印 `… already exists. Nothing was changed. …`。这些值等于你的钥匙：不要截图，也不要贴到论坛、群聊或 GitHub issue 里求助；求助时只贴这一步打印的报错（`ERROR` 开头的那一行，或者 openssl、sing-box 的报错），不要贴 `/etc/sing-box/guide.env` 的内容。

**出错时**：

- `ERROR: replace <SERVER_IP> and <REALITY_TARGET> …`：代码块里的占位符还没换。换好后再粘贴，这时什么文件都还没写。
- `ERROR: SERVER_IP must be an IPv4 address …` 或 `ERROR: REALITY_TARGET must be a host name only …`：值的格式不对，例如 IP 写错，或者域名前面多了 `https://`。
- `ERROR: … does not offer TLS 1.3 with HTTP/2 …`：这一步会再做一次 S7 的检查，结果不合格。按 S7 换一个借用网站，改掉单引号里的域名后再粘贴；这时什么文件都还没写。
- 还没打印 `Created …` 就停止了（前面是 openssl 或 sing-box 的报错，或者 `ERROR: a generated value is empty …`）：某个生成命令失败了（例如 sing-box 没有装好），`guide.env` 没有写入。先看前面的报错并修好，再粘贴这个代码块。
- 想修改某个值（例如填错了 IP）：不要删除 `guide.env`（删除后会生成新的密钥，所有客户端都要重新导入）。改法见「[P2 判断 IP 是否被封并更换 IP](#p2-判断-ip-是否被封并更换-ip)」和「[P4 更换借用网站](#p4-更换借用网站)」。

<!-- step:server-09-config -->
### S9 写入配置、检查并启动

**目的**：用 `guide.env` 里的值生成服务器配置。先写到 `/etc/sing-box/config.json.new`，用 `sing-box check` 检查；只有检查通过，才替换 `/etc/sing-box/config.json` 并重启服务。检查不通过时，正在运行的服务和旧配置都不受影响。

配置要点：

- REALITY：VLESS 入站，`flow` 为 `xtls-rprx-vision`，握手借用 `REALITY_TARGET` 的 443 端口（[sing-box VLESS 入站文档](https://sing-box.sagernet.org/configuration/inbound/vless/)）。
- Hysteria2：服务器不设置带宽（没有 `up_mbps` 和 `down_mbps`），并设置 `ignore_client_bandwidth: true`。按 sing-box 文档，这样 Hysteria2 不使用 Brutal 模式（[sing-box Hysteria2 入站文档](https://sing-box.sagernet.org/configuration/inbound/hysteria2/)）。不需要另外安装任何加速模块。
- 路由：拒绝以私有 IP 地址（服务器所在内网和本机）为目标的连接，以及去往 25 端口（发邮件）的连接。按 AUP，与垃圾邮件有关的账户会被立即终止，判断公共代理的依据之一也是其他网络的垃圾邮件投诉；拒绝 25 端口可以避免节点被人拿去发垃圾邮件。
- 官方的 systemd 服务用 `-C /etc/sing-box` 读取并合并这个目录下所有以 `.json` 结尾的文件（可以用 `systemctl cat sing-box` 查看）。所以这一步先确认目录里除了 `config.json` 没有别的 `.json` 文件，这样 `sing-box check` 检查的就是服务实际会读取的内容。本教程的客户端文件放在 `/root/dmit-guide/`，不在这个目录里。

**命令**（在服务器上以 root 运行）：

```bash server
(
set -euo pipefail
ENV=/etc/sing-box/guide.env
[ -f "$ENV" ] || { echo "ERROR: $ENV not found. Do step S8 first." >&2; exit 1; }
if grep -q '[<>]' "$ENV"; then echo "ERROR: $ENV still contains < or >. Fix that line first." >&2; exit 1; fi
others=$(find /etc/sing-box -maxdepth 1 -name '*.json' ! -name config.json)
if [ -n "$others" ]; then echo "ERROR: the sing-box service would also load: $others. Move these files out of /etc/sing-box first." >&2; exit 1; fi
set -a; . "$ENV"; set +a
umask 077
cat > /etc/sing-box/config.json.new <<EOF
{
  "log": {
    "level": "warn",
    "timestamp": true
  },
  "inbounds": [
    {
      "type": "vless",
      "tag": "reality-in",
      "listen": "::",
      "listen_port": 443,
      "users": [
        {
          "name": "me",
          "uuid": "$UUID",
          "flow": "xtls-rprx-vision"
        }
      ],
      "tls": {
        "enabled": true,
        "server_name": "$REALITY_TARGET",
        "reality": {
          "enabled": true,
          "handshake": {
            "server": "$REALITY_TARGET",
            "server_port": 443
          },
          "private_key": "$REALITY_PRIVATE_KEY",
          "short_id": [
            "$SHORT_ID"
          ]
        }
      }
    },
    {
      "type": "hysteria2",
      "tag": "hy2-in",
      "listen": "::",
      "listen_port": 443,
      "users": [
        {
          "name": "me",
          "password": "$HY2_PASSWORD"
        }
      ],
      "ignore_client_bandwidth": true,
      "tls": {
        "enabled": true,
        "certificate_path": "/etc/sing-box/hy2.crt",
        "key_path": "/etc/sing-box/hy2.key"
      }
    }
  ],
  "outbounds": [
    {
      "type": "direct",
      "tag": "direct"
    }
  ],
  "route": {
    "rules": [
      {
        "ip_is_private": true,
        "action": "reject"
      },
      {
        "port": 25,
        "action": "reject"
      }
    ],
    "final": "direct"
  }
}
EOF
sing-box check -c /etc/sing-box/config.json.new
mv /etc/sing-box/config.json.new /etc/sing-box/config.json
systemctl enable sing-box.service
systemctl restart sing-box.service
echo "OK: config checked, installed and sing-box restarted."
)
```

**预期结果**：最后一行是 `OK: config checked, installed and sing-box restarted.`。第一次运行时前面可能还有一行 `Created symlink …`。

**出错时**：

- `ERROR: /etc/sing-box/guide.env not found`：先做 S8。
- `ERROR: the sing-box service would also load: …`：`/etc/sing-box` 里有别的 `.json` 文件，服务会把它们和 `config.json` 合并。把列出的文件移出这个目录（例如移到 `/root/`），再粘贴一次。
- `sing-box check` 打印 `FATAL` 并停止：配置没有替换，服务没有重启。把 `FATAL` 那一行完整记下来，对照 S8 的输出检查 `guide.env` 是否完整，再粘贴一次。
- `Job for sing-box.service failed`：运行 `journalctl -u sing-box -n 30 --no-pager` 查看原因。

<!-- step:server-10-listen -->
### S10 确认服务在 443 端口监听

**目的**：确认 sing-box 正在运行，并且同时在 TCP 443 和 UDP 443 上监听。S9 重启服务后，sing-box 要一点时间才开始监听，所以这一步最多会等 10 秒再下结论。

**命令**（在服务器上以 root 运行）：

```bash server
(
set -eu
n=0
until systemctl is-active --quiet sing-box.service && ss -Htln 'sport = :443' | grep -q . && ss -Huln 'sport = :443' | grep -q .; do
  n=$((n + 1))
  [ "$n" -lt 10 ] || break
  sleep 1
done
if ! systemctl is-active --quiet sing-box.service; then
  echo "FAIL: sing-box is not running" >&2
  journalctl -u sing-box.service -n 20 --no-pager >&2
  exit 1
fi
ss -Htln 'sport = :443' | grep -q . || { echo "FAIL: nothing listens on TCP 443" >&2; exit 1; }
ss -Huln 'sport = :443' | grep -q . || { echo "FAIL: nothing listens on UDP 443" >&2; exit 1; }
ss -Htulpn 'sport = :443'
echo "OK: sing-box is running and listens on TCP 443 and UDP 443"
)
```

**预期结果**：打印两行端口信息，一行以 `udp` 开头、一行以 `tcp` 开头（两行的先后顺序不重要），都带 `sing-box`；最后一行是 `OK: sing-box is running and listens on TCP 443 and UDP 443`。

**出错时**：

- `FAIL: sing-box is not running`：看打印出来的日志。常见原因是 443 端口被别的程序占用，用 `ss -tulpn | grep ':443'` 查看是哪个程序。
- 只缺 TCP 或只缺 UDP：重新粘贴 S9，再运行这一步。

<!-- step:server-11-export -->
### S11 生成客户端配置和分享链接

**目的**：用 `guide.env` 里的值生成客户端配置文件 `/root/dmit-guide/client-profile.json` 和分享链接文件 `/root/dmit-guide/share-links.txt`（目录权限 700，文件权限 600），检查客户端配置，然后把两者打印出来，并为两条分享链接显示二维码。重复运行会得到同样的内容。这两个文件不放在 `/etc/sing-box`，因为服务会读取那里所有的 `.json` 文件。

客户端配置里有四个出站：`reality`（TCP 443 这条线）、`hy2`（UDP 443 这条线）、`auto`（每 1 分钟测试一次 `reality` 和 `hy2`，自动选用可用且较快的一条）、`direct`（直连，只用于私有地址）。其他所有流量都走 `auto`，也就是全局代理。DNS 通过代理用 DNS over HTTPS 查询，只返回 IPv4 地址。

**命令**（在服务器上以 root 运行）：

```bash server
(
set -euo pipefail
ENV=/etc/sing-box/guide.env
[ -f "$ENV" ] || { echo "ERROR: $ENV not found. Do step S8 first." >&2; exit 1; }
if grep -q '[<>]' "$ENV"; then echo "ERROR: $ENV still contains < or >. Fix that line first." >&2; exit 1; fi
set -a; . "$ENV"; set +a
umask 077
install -d -m 700 /root/dmit-guide
cat > /root/dmit-guide/client-profile.json <<EOF
{
  "log": {
    "level": "warn"
  },
  "dns": {
    "servers": [
      {
        "type": "https",
        "tag": "remote-dns",
        "server": "1.1.1.1",
        "detour": "auto"
      }
    ],
    "strategy": "ipv4_only"
  },
  "inbounds": [
    {
      "type": "tun",
      "tag": "tun-in",
      "address": [
        "172.19.0.1/30",
        "fdfe:dcba:9876::1/126"
      ],
      "auto_route": true,
      "strict_route": true
    }
  ],
  "outbounds": [
    {
      "type": "urltest",
      "tag": "auto",
      "outbounds": [
        "reality",
        "hy2"
      ],
      "interval": "1m"
    },
    {
      "type": "vless",
      "tag": "reality",
      "server": "$SERVER_IP",
      "server_port": 443,
      "uuid": "$UUID",
      "flow": "xtls-rprx-vision",
      "tls": {
        "enabled": true,
        "server_name": "$REALITY_TARGET",
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        },
        "reality": {
          "enabled": true,
          "public_key": "$REALITY_PUBLIC_KEY",
          "short_id": "$SHORT_ID"
        }
      }
    },
    {
      "type": "hysteria2",
      "tag": "hy2",
      "server": "$SERVER_IP",
      "server_port": 443,
      "password": "$HY2_PASSWORD",
      "tls": {
        "enabled": true,
        "server_name": "$REALITY_TARGET",
        "certificate_public_key_sha256": [
          "$HY2_PUBKEY_SHA256"
        ]
      }
    },
    {
      "type": "direct",
      "tag": "direct"
    }
  ],
  "route": {
    "rules": [
      {
        "action": "sniff"
      },
      {
        "protocol": "dns",
        "action": "hijack-dns"
      },
      {
        "ip_is_private": true,
        "outbound": "direct"
      }
    ],
    "final": "auto",
    "auto_detect_interface": true
  }
}
EOF
sing-box check -c /root/dmit-guide/client-profile.json
VLESS="vless://${UUID}@${SERVER_IP}:443?encryption=none&flow=xtls-rprx-vision&security=reality&sni=${REALITY_TARGET}&fp=chrome&pbk=${REALITY_PUBLIC_KEY}&sid=${SHORT_ID}&type=tcp#dmit-reality"
HY2="hysteria2://${HY2_PASSWORD}@${SERVER_IP}:443/?sni=${REALITY_TARGET}&insecure=1&pinSHA256=${HY2_CERT_SHA256}#dmit-hy2"
printf '%s\n' "$VLESS" "$HY2" > /root/dmit-guide/share-links.txt
chmod 700 /root/dmit-guide
chmod 600 /root/dmit-guide/client-profile.json /root/dmit-guide/share-links.txt
echo "===== /root/dmit-guide/client-profile.json ====="
cat /root/dmit-guide/client-profile.json
echo "===== /root/dmit-guide/share-links.txt ====="
cat /root/dmit-guide/share-links.txt
echo "===== QR code: dmit-reality ====="
qrencode -t ansiutf8 "$VLESS"
echo "===== QR code: dmit-hy2 ====="
qrencode -t ansiutf8 "$HY2"
)
```

**预期结果**：依次打印客户端配置（一段 JSON，里面已经没有 `$` 开头的变量名）、两条分享链接（一条以 `vless://` 开头，一条以 `hysteria2://` 开头），以及两个二维码。`sing-box check` 通过时不打印任何内容。这些内容（客户端配置、分享链接和二维码）等于你的钥匙：不要截图，也不要贴到论坛、群聊或 GitHub issue 里求助；求助时只贴 `ERROR` 或 `FATAL` 开头的那一行，如果那一行里有 UUID、密码或密钥，先换成 `xxx`。

**出错时**：

- `ERROR: /etc/sing-box/guide.env not found`：先做 S8。
- `sing-box check` 打印 `FATAL`：把那一行记下来；通常是 `guide.env` 某一行不完整，对照 S8 的预期结果检查。
- 二维码显示错乱：把终端窗口拉宽，或者改用分享链接的文字。

分享链接说明：

- `vless://` 链接用常见的 VLESS 分享格式。各个 App 能否导入，本文未能核实。
- `hysteria2://` 链接按 Hysteria 官方 URI 格式写成 `insecure=1&pinSHA256=…`（[Hysteria URI 格式](https://v2.hysteria.network/docs/developers/URI-Scheme/)）。原因是服务器用的是自签证书：官方 Hysteria 客户端用 `insecure` 决定是否跳过常规的 CA 证书链校验，再在证书校验回调里比对 `pinSHA256`（[Hysteria 客户端源代码](https://github.com/HyNetworks/hysteria/blob/master/app/cmd/client.go)）；而 Go 的 TLS 库规定，常规校验失败时根本不会调用这个回调（[Go crypto/tls 文档](https://pkg.go.dev/crypto/tls#Config)）。所以自签证书配 `insecure=0` 会连不上，`insecure=1&pinSHA256=…` 的意思是「不校验 CA 证书链，但仍然校验证书指纹」（查证日期 2026-11-15）。
- **这条 Hysteria2 连接的安全完全靠 `pinSHA256`。** 如果某个 App 忽略 `pinSHA256`，它会接受任何证书，有人在中途拦截时可能拿到你的 Hysteria2 密码。哪些 App 会校验 `pinSHA256`，本文未能核实（查证日期 2026-11-15）。所以请优先使用下面客户端设置里的 sing-box 配置文件，它用 `certificate_public_key_sha256` 固定了证书；用不了配置文件时，才导入 `hysteria2://` 链接。
- 某个 App 用 `hysteria2://` 链接连不上：确认导入后 `pinSHA256`（证书指纹）还在、「允许不安全证书」一类的选项是开启的（对应 `insecure=1`）。确认不了这个 App 会校验 `pinSHA256` 的话，不要用这个链接，改用配置文件。

<!-- section:client -->
## 客户端设置

**做法：** 四个平台都用 sing-box 官方 App，导入服务器 S11 生成的 `client-profile.json`。按 sing-box 官方文档，这些 App 都可以管理和运行本地或远程的 sing-box 配置文件（[sing-box 客户端文档](https://sing-box.sagernet.org/clients/)，查证日期 2026-11-15）；能否导入 `vless://`、`hysteria2://` 分享链接，官方文档没有提到。所以本文以配置文件为主。

**两条线怎么自动切换：** 配置文件里的 `auto` 组每 1 分钟测试一次 `reality` 和 `hy2`，自动使用能连通且较快的那一条；一条连不上时，下一次测试后会自动换到另一条。你不需要手动选节点。

**关于界面：** 下面 C2 到 C5 的 App 操作是根据 sing-box 官方文档整理的，没有在真机上逐一测试；不同版本的菜单名称可能略有不同。sing-box 官方文档提醒：有些第三方项目声称使用 sing-box，但未获认可，往往质量差、有广告（[sing-box 客户端文档](https://sing-box.sagernet.org/clients/)）。请只从下面列出的官方渠道下载。

**安全：** `client-profile.json` 和分享链接等于你的登录凭据。不要通过公开的地方、聊天群或网盘传输它们；用数据线、AirDrop 或你信任的私人方式传到自己的设备上。

<!-- step:client-01-profile -->
### C1 把客户端配置文件下载到电脑

**目的**：把服务器上的 `client-profile.json` 和 `share-links.txt` 复制到你自己的电脑，再传到手机或在电脑上导入。

**命令**（在你自己的电脑上运行，先替换 `<SSH_USER>` 和 `<SERVER_IP>`）：

```bash local
ssh <SSH_USER>@<SERVER_IP> sudo cat /root/dmit-guide/client-profile.json > client-profile.json
ssh <SSH_USER>@<SERVER_IP> sudo cat /root/dmit-guide/share-links.txt > share-links.txt
chmod 600 client-profile.json share-links.txt
head -c 200 client-profile.json
```

如果你不方便用上面的命令（例如在 Windows 上），也可以在服务器的 root 窗口里运行 `cat /root/dmit-guide/client-profile.json`，选中全部输出复制，在电脑上用文本编辑器新建文件粘贴进去，保存为 `client-profile.json`（编码选 UTF-8）。

这个文件的内容就是下面这个模板，`$` 开头的部分已经由服务器换成了 `guide.env` 里的值（S11 用的正是这个模板）：

```json client-profile
{
  "log": {
    "level": "warn"
  },
  "dns": {
    "servers": [
      {
        "type": "https",
        "tag": "remote-dns",
        "server": "1.1.1.1",
        "detour": "auto"
      }
    ],
    "strategy": "ipv4_only"
  },
  "inbounds": [
    {
      "type": "tun",
      "tag": "tun-in",
      "address": [
        "172.19.0.1/30",
        "fdfe:dcba:9876::1/126"
      ],
      "auto_route": true,
      "strict_route": true
    }
  ],
  "outbounds": [
    {
      "type": "urltest",
      "tag": "auto",
      "outbounds": [
        "reality",
        "hy2"
      ],
      "interval": "1m"
    },
    {
      "type": "vless",
      "tag": "reality",
      "server": "$SERVER_IP",
      "server_port": 443,
      "uuid": "$UUID",
      "flow": "xtls-rprx-vision",
      "tls": {
        "enabled": true,
        "server_name": "$REALITY_TARGET",
        "utls": {
          "enabled": true,
          "fingerprint": "chrome"
        },
        "reality": {
          "enabled": true,
          "public_key": "$REALITY_PUBLIC_KEY",
          "short_id": "$SHORT_ID"
        }
      }
    },
    {
      "type": "hysteria2",
      "tag": "hy2",
      "server": "$SERVER_IP",
      "server_port": 443,
      "password": "$HY2_PASSWORD",
      "tls": {
        "enabled": true,
        "server_name": "$REALITY_TARGET",
        "certificate_public_key_sha256": [
          "$HY2_PUBKEY_SHA256"
        ]
      }
    },
    {
      "type": "direct",
      "tag": "direct"
    }
  ],
  "route": {
    "rules": [
      {
        "action": "sniff"
      },
      {
        "protocol": "dns",
        "action": "hijack-dns"
      },
      {
        "ip_is_private": true,
        "outbound": "direct"
      }
    ],
    "final": "auto",
    "auto_detect_interface": true
  }
}
```

几点说明：`uTLS` 在 sing-box 文档里被标为「不推荐」，但 REALITY 客户端必须启用 uTLS（去掉 `utls` 后 `sing-box check` 会报错 `uTLS is required by reality client`），这里用 `chrome` 指纹。`inbounds` 里 `tun` 的两个 `address` 是 App 在你设备上建立的虚拟网卡地址，不是服务器地址。`dns` 里的 `server` 是 Cloudflare 的公共 DNS，查询经过代理（`auto`）发出。

**预期结果**：当前目录（你运行这些命令时所在的目录，可以用 `pwd` 查看）出现 `client-profile.json` 和 `share-links.txt`；`head` 打印的开头是 `{` 和 `"log"`。

**出错时**：

- `sudo: a terminal is required to read the password`：你的普通用户使用 sudo 要输入密码，这种情况下用上面「复制粘贴」的办法。
- `No such file or directory`：服务器上还没有做 S11。
- 文件是空的：看命令输出的错误信息；也可以改用复制粘贴的办法。
- 以后改过 `guide.env`（换 IP、换借用网站）：要重新运行 S11，再做一次 C1，并在 App 里重新导入。

<!-- step:client-02-ios -->
### C2 iOS：sing-box 官方 App

**目的**：在 iPhone 或 iPad 上导入配置并连接。

**操作**（根据 sing-box 官方文档，查证日期 2026-11-15）：

1. 用非中国大陆地区的 Apple 账户，在 App Store 安装 sing-box 官方 App（官方文档称为「sing-box for Apple platforms」，要求 iOS 15.0 或更新）。TestFlight 测试版名额只限赞助者。来源：[sing-box Apple 客户端文档](https://sing-box.sagernet.org/clients/apple/)。
2. 把 `client-profile.json` 传到设备上（例如用 AirDrop，或者用数据线放进「文件」App）。
3. 在 sing-box App 的配置（Profiles）页面新建一个本地配置，选择从文件导入，选中 `client-profile.json`。
4. 回到首页选中这个配置，点启动；系统第一次会请求添加 VPN 配置，选允许。

**预期结果**：App 显示已连接，系统状态栏出现 VPN 图标；按「[检查是否生效](#检查是否生效)」检查出口 IP。

**出错时**：

- App Store 搜不到：中国大陆区的 Apple 账户下载不了，官方文档要求非中国大陆账户。
- 导入时报错：确认文件是完整的 JSON（用文本编辑器打开，第一行是 `{`，最后一行是 `}`），然后重新导入。
- 备选（不是 sing-box 官方 App，本文不推荐）：Shadowrocket（App Store，US$2.99），它的版本记录提到 VLESS 和 Hysteria 2 的修正，以及通过服务器 URL、订阅和二维码导入（[App Store 页面](https://apps.apple.com/us/app/shadowrocket/id932747118)，查证日期 2026-11-15）。它能否正确导入本文的分享链接、是否校验 `pinSHA256`，本文未能核实；如果它不校验 `pinSHA256`，用分享链接导入的 Hysteria2 线路会接受任何证书。

<!-- step:client-03-android -->
### C3 Android：sing-box 官方 App

**目的**：在 Android 手机上导入配置并连接。

**操作**（根据 sing-box 官方文档，查证日期 2026-11-15）：

1. 安装 sing-box 官方 App（官方文档称为 SFA，要求 Android 5.0 或更新）。官方渠道：Google Play（含 Beta）、[GitHub Releases](https://github.com/SagerNet/sing-box/releases)、F-Droid（F-Droid 版使用统一签名，可重现构建）。来源：[sing-box Android 客户端文档](https://sing-box.sagernet.org/clients/android/)。
2. 把 `client-profile.json` 传到手机上。
3. 在 App 的配置（Profiles）页面新建一个本地配置，选择从文件导入，选中 `client-profile.json`。
4. 回到首页选中这个配置，点启动；系统第一次会请求建立 VPN 连接，选「确定」（OK）（未能核实，以实际界面为准）。

**预期结果**：App 显示已连接，通知栏出现 VPN 图标（钥匙图标）。

**出错时**：

- 打不开 Google Play：改用 GitHub Releases 或 F-Droid 下载。
- 装到了名字相近的第三方 App：卸载，只用上面列出的官方渠道。
- 导入时报错：同 C2。

<!-- step:client-04-windows -->
### C4 Windows：sing-box 官方桌面 App

**目的**：在 Windows 电脑上导入配置并连接。

**操作**（根据 sing-box 官方文档，查证日期 2026-11-15）：

1. 从 [GitHub Releases](https://github.com/SagerNet/sing-box/releases) 下载 sing-box 官方桌面 App（官方文档称为「sing-box for Desktop」，列出 Windows 10 或更新，支持 x64、x86、arm64）。来源：[sing-box 桌面客户端文档](https://sing-box.sagernet.org/clients/desktop/)。注意：它的代码仓库说明把 Windows 客户端标为「Experimental」（实验性）（[sing-box-for-desktop](https://github.com/SagerNet/sing-box-for-desktop)），文档和仓库说明的说法不一致，成熟度本文未能核实。
2. 打开 App，新建一个本地配置，从文件导入 `client-profile.json`。
3. 选中这个配置并启动。

**预期结果**：App 显示已连接，浏览器能打开境外网站。

**出错时**：

- 启动失败或提示权限不足：配置里使用了虚拟网卡（tun），可以试着用管理员身份运行 App。
- 桌面 App 无法使用：备选（不是 sing-box 官方 App）是 v2rayN（[GitHub](https://github.com/2dust/v2rayN)），它支持 Xray 和 sing-box 内核；它的说明没有提到分享链接和 JSON 配置导入，本文未能核实。

<!-- step:client-05-macos -->
### C5 macOS：sing-box 官方 App

**目的**：在 Mac 上导入配置并连接。

**操作**（根据 sing-box 官方文档，查证日期 2026-11-15）：

1. 安装 sing-box 官方 App（官方文档称为 SFM，与 iOS 版同属「sing-box for Apple platforms」，要求 macOS 13.0 或更新，以及非中国大陆地区的 Apple 账户）。官方文档列出的 macOS 独立版渠道：Homebrew Cask（如果你已经装了 Homebrew，可以在终端运行 `brew install sfm`），或者 [GitHub Releases](https://github.com/SagerNet/sing-box/releases)。是否在 Mac App Store 上架，本文未能核实。来源：[sing-box Apple 客户端文档](https://sing-box.sagernet.org/clients/apple/)。
2. 打开 App，新建一个本地配置，从文件导入 `client-profile.json`。
3. 选中这个配置并启动；系统第一次会请求添加 VPN 配置或系统扩展，选允许。

**预期结果**：App 显示已连接，菜单栏或系统设置里显示 VPN 已连接。

**出错时**：

- 系统提示扩展被阻止：在「系统设置 → 隐私与安全性」里允许，然后重新启动连接。
- 没有 Homebrew：不要为此去运行 Homebrew 的安装脚本，直接从 GitHub Releases 下载即可。

<!-- section:verify -->
## 检查是否生效

<!-- step:verify-01-exit-ip -->
### V1 检查出口 IP

**目的**：确认流量从你的 DMIT 服务器出去。

**命令**（在已经连上代理的电脑上运行；手机上直接用浏览器打开 https://ipinfo.io/json ）：

```bash local
curl -s https://ipinfo.io/json
```

Windows 上请在 PowerShell 里输入 `curl.exe -s https://ipinfo.io/json`。Windows PowerShell 5.1 里的 `curl` 是 `Invoke-WebRequest` 的别名，不是 curl 本身；Windows 10 和 11 自带 curl，写成 `curl.exe` 才会用到它（[Microsoft 文档](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-5.1)、[curl 官网](https://curl.se/windows/microsoft.html)，查证日期 2026-11-16）。

**预期结果**：`"ip"` 等于你的服务器 IP（`<SERVER_IP>`）。`"country"` 可能显示 `HK`，`"org"` 显示服务器所属的网络。注意：DMIT 的 IP 是随机分配的，DMIT 不保证 IP 的地理信息（根据 DMIT 官方文档，查证日期 2026-11-17），见 [IP FAQ](https://docs.dmit.io/guide/faq/ip)；所以只要 `"ip"` 对上，就说明代理在工作。

**出错时**：

- 显示的是你本地网络的 IP：代理没有生效。确认 App 显示已连接；电脑上确认没有其他代理软件同时运行。
- 打不开：按「[P1 连不上时逐项检查](#p1-连不上时逐项检查)」排查。

<!-- step:verify-02-dns-leak -->
### V2 基本的 DNS 泄漏检查

**目的**：确认域名查询也经过代理，而不是发给你本地的运营商。

**操作**：连上代理后，在浏览器打开 https://www.dnsleaktest.com/ ，点「Standard test」，等待结果。

**预期结果**：结果列表里的 DNS 服务器都属于 Cloudflare（本文配置的 DNS 是 Cloudflare 的公共 DNS，查询经过代理发出），而不是你所在地的电信、联通、移动等运营商。

**出错时**：

- 结果里出现你本地运营商的 DNS：确认导入的是 S11 生成的完整配置（里面有 `hijack-dns` 规则），断开后重新连接再测一次。
- 结果显示的是 Google 等其他 DNS 提供商：通常是浏览器自己开启了「安全 DNS」，它的查询同样经过代理，不算泄漏到本地运营商。
- 网站打不开：先完成 V1。

<!-- step:verify-03-both-lines -->
### V3 确认两条线都通

**目的**：确认 REALITY 和 Hysteria2 两条线都能用，这样一条出问题时另一条能顶上。

**操作**：在 sing-box App 里打开分组（Groups）页面，找到 `auto` 分组，查看或刷新 `reality` 和 `hy2` 的延迟测试结果（界面名称根据 sing-box 官方 App 整理，未在真机上测试，可能因版本不同而略有差异）。

**预期结果**：`reality` 和 `hy2` 都显示一个毫秒数，而不是超时或错误。

**出错时**：

- 只有 `hy2` 超时：你所在的网络可能限制了 UDP，或者服务器的 UDP 443 没有放行。在服务器上重新运行 S10；本地网络限制 UDP 时，只用 `reality` 也可以。
- 只有 `reality` 超时：在服务器上重新运行 S7，确认借用网站仍然合格；不合格就按「[P4 更换借用网站](#p4-更换借用网站)」处理。
- 两条都超时：按「[P1 连不上时逐项检查](#p1-连不上时逐项检查)」排查。

<!-- section:daily -->
## 日常维护

<!-- step:daily-01-panel -->
### D1 在 DMIT 客户区看流量

**目的**：了解本月已用流量和下次重置时间，避免流量用完被暂停。

**操作**：登录 DMIT 客户区，打开这台机器的管理页面查看流量（未能核实，以实际界面为准）。按 [实例 FAQ](https://docs.dmit.io/guide/faq/instance)，流量每月自动重置，控制台会显示下次重置时间（根据 DMIT 官方文档，查证日期 2026-11-17）。

**预期结果**：能看到下次重置时间（根据 DMIT 官方文档，查证日期 2026-11-17）；已用流量显示在哪里（未能核实，以实际界面为准）。DMIT HKG.Pro TINY 和 DMIT HKG.Pro STARTER 是双向计费，上传和下载都计入（根据 DMIT 官方文档，查证日期 2026-11-17）。

**出错时**：

- 找不到流量信息：开工单问 DMIT；TOS 只保证 72 小时内回复（根据 DMIT 官方文档，查证日期 2026-11-17）。
- 流量已经快用完：见「[P3 流量用完或服务被暂停](#p3-流量用完或服务被暂停)」。

<!-- step:daily-02-vnstat -->
### D2 在服务器上用 vnstat 统计流量

**目的**：在服务器上自己记录每月流量，和 DMIT 客户区的数字对照。`vnstat` 来自 Debian 自己的软件源。

**命令**（在服务器上以 root 运行，先做 S2、S3）：

```bash server
(
set -eu
DEBIAN_FRONTEND=noninteractive apt-get -y install vnstat
systemctl enable --now vnstat.service
vnstat -m
)
```

**预期结果**：刚安装时会显示还没有数据（例如 `eth0: No data.`，网卡名可能不同），过一段时间再运行 `vnstat -m`，会按月显示 `rx`（收）、`tx`（发）和 `total`（合计）。DMIT 是双向计费（根据 DMIT 官方文档，查证日期 2026-11-17），对照时看 `total`。vnstat 只从安装时开始统计，以 DMIT 客户区的数字为准。

**出错时**：

- `Unable to open database` 或一直没有数据：运行 `systemctl status vnstat` 看服务是否在运行，等几分钟后再看。

<!-- step:daily-03-upgrade -->
### D3 升级系统和 sing-box

**目的**：定期安装系统安全更新，并把 sing-box 升级到官方源的最新稳定版。

**命令**（在服务器上以 root 运行，先做 S2、S3）：

```bash server
(
set -eu
export DEBIAN_FRONTEND=noninteractive
apt-get update
apt-get -y -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold full-upgrade
sing-box version
sing-box check -C /etc/sing-box
systemctl restart sing-box.service
systemctl is-active sing-box.service
)
```

**预期结果**：打印新的 `sing-box version …`，最后一行是 `active`。这次升级如果装了新内核，要重启服务器才会用上；不确定的话，再粘贴一次 S4，看它的最后一行。

**出错时**：

- `sing-box check` 打印 `FATAL` 并停止：新版本不接受现在的配置，服务没有重启，正在运行的还是旧版本的进程（sing-box 软件包升级时不会自己重启服务）。但服务一旦重启（包括服务器重启），就会用新版本启动并失败，所以要尽快处理。先阅读 [sing-box 发布说明](https://github.com/SagerNet/sing-box/releases)；需要回到旧版本时，运行 `apt-cache policy sing-box` 查看可用版本，再运行 `apt-get install --allow-downgrades sing-box=版本号` 安装旧版本，然后运行 `systemctl restart sing-box.service`。
- 手机或电脑上的 sing-box App 也要保持更新；客户端版本不能低于 1.13.0。

<!-- section:trouble -->
## 出问题时怎么办

<!-- step:trouble-01-checklist -->
### P1 连不上时逐项检查

**目的**：按顺序排除常见原因。

**命令**（在服务器上以 root 运行，先做 S2、S3）：

```bash server
(
systemctl is-active sing-box.service sing-box-guide-firewall.service
ss -Htulpn 'sport = :443'
sing-box check -C /etc/sing-box && echo "config: OK"
if { nft list ruleset; iptables-save; ip6tables-save; } 2>/dev/null | grep -qE 'sing_box_guide|sing-box-guide'; then echo "firewall rules: OK"; else echo "firewall rules: MISSING"; fi
timedatectl | grep -E 'System clock synchronized|NTP service'
journalctl -u sing-box.service -n 30 --no-pager
)
```

然后逐项检查：

1. **服务器能不能登录？** 连 SSH 都连不上，先在 DMIT 客户区确认这台机器在运行（未能核实，以实际界面为准）；机器在运行但从大陆连不上，见 P2。
2. **DMIT 有没有暂停服务？** 订购页写明，超出流量后机器会被暂停，标注为 Paused / Quota Exceeded（根据 DMIT 官方文档，查证日期 2026-11-17），见 P3；月付机器到期后只保留 3 天，3 天内付款可以恢复（根据 DMIT 官方文档，查证日期 2026-11-17）。
3. **服务、端口和防火墙：** 上面的命令前两行应是两个 `active`，并且列出 TCP 和 UDP 443，还要打印 `firewall rules: OK`。服务或端口不对的话，重新运行 S5、S9、S10。打印 `firewall rules: MISSING` 时，说明本教程加的防火墙规则被清掉了（例如有人重新加载了别的防火墙配置），防火墙服务却可能仍显示 `active`：运行 `systemctl restart sing-box-guide-firewall.service`，再运行一次上面的命令。如果它提示 `start of the service was attempted too often`（短时间内重启太多次），先运行 `systemctl reset-failed sing-box-guide-firewall.service`，再重启一次。
4. **时间要准：** 服务器应显示 `System clock synchronized: yes`；手机和电脑要打开「自动设置时间」。时间相差太多时，TLS 证书校验可能失败。服务器显示 `no` 时，运行 `timedatectl set-ntp true`，一两分钟后再运行一次上面的命令；如果显示 `NTP service: n/a`（系统里没有可用的时间同步服务，这时 `timedatectl set-ntp true` 会报 `NTP not supported`），先运行 `apt-get -y install systemd-timesyncd`，再运行 `timedatectl set-ntp true`（[timedatectl 手册](https://manpages.debian.org/trixie/systemd/timedatectl.1.en.html)，查证日期 2026-11-16）。
5. **客户端配置是不是最新的：** 改过 `guide.env` 之后，要重新运行 S11 和 C1，并在 App 里重新导入。
6. **借用网站还合格吗：** 重新运行 S7。不合格见 P4。
7. **只有一条线不通：** 见 V3 的「出错时」。
8. **日志：** 最后一部分是 sing-box 最近的日志，留意 `ERROR` 或 `FATAL` 开头的行。

**预期结果**：找到并修好其中一项后，客户端重新连接能用。

**出错时**：以上都正常但仍然连不上，可能是 IP 被封，见 P2。

<!-- step:trouble-02-ip-blocked -->
### P2 判断 IP 是否被封并更换 IP

**目的**：判断服务器 IP 是否在大陆被封；被封时按 DMIT 规则更换 IP，并更新配置。

**命令**（先在你自己在大陆的电脑上、**关掉代理**后运行，替换 `<SERVER_IP>`）：

```bash local
ping -c 4 <SERVER_IP>
nc -vz -w 5 <SERVER_IP> 22
nc -vz -w 5 <SERVER_IP> 443
```

Windows 的 `ping` 没有 `-c`，用 `/n` 指定次数。Windows 上请在 PowerShell 里依次运行 `ping /n 4 <SERVER_IP>`、`Test-NetConnection <SERVER_IP> -Port 22` 和 `Test-NetConnection <SERVER_IP> -Port 443`；结果里 `TcpTestSucceeded : False` 表示这个端口连不上（[ping 文档](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ping)、[Test-NetConnection 文档](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection)，查证日期 2026-11-16）。

如果 ping 全部丢包、22 和 443 都连不上，而服务器没有因为流量用完或欠费被暂停，IP 很可能被封了。

**更换 IP 的规则**，见 [IP FAQ](https://docs.dmit.io/guide/faq/ip)（根据 DMIT 官方文档，查证日期 2026-11-17）：

- 免费更换，要同时满足：购买满 7 日；距上次更换满 15 日；整个 IP 的 ICMP 和所有 TCP 端口都被封；服务剩余有效期至少 7 日。开通时就已被封，可以开工单免费更换。
- 付费更换：随时可申请，每次 US$5，两次申请至少相隔 7 日；完成后不能撤销或退款；滥用可能被禁止使用这个功能。
- 通过工单申请。TOS 只保证 72 小时内回复，工单 FAQ 写通常 24 小时内。

**拿到新 IP 之后**，用新 IP 重新做 S2、S3，然后运行下面的代码块（把 `<SERVER_IP>` 换成**新的** IP，保留单引号）：

```bash server
(
set -eu
NEW_IP='<SERVER_IP>'
printf '%s\n' "$NEW_IP" | grep -Eqx '[0-9]{1,3}(\.[0-9]{1,3}){3}' || { echo "ERROR: replace <SERVER_IP> with the new IPv4 address first" >&2; exit 1; }
sed -i "s/^SERVER_IP=.*/SERVER_IP=$NEW_IP/" /etc/sing-box/guide.env
grep '^SERVER_IP=' /etc/sing-box/guide.env
)
```

然后重新运行 S11，再做 C1，并在所有 App 里重新导入。服务器配置里没有写 IP，不需要重做 S9。

**预期结果**：打印 `SERVER_IP=新的IP`；重新导入后，V1 显示的出口 IP 是新 IP。

**出错时**：

- `ERROR: replace <SERVER_IP> …`：代码块里的占位符还没换成新 IP。
- 不满足免费条件：可以选择付费更换，或等条件满足后再申请。

<!-- step:trouble-03-traffic -->
### P3 流量用完或服务被暂停

**目的**：了解流量用完后的处理方式。

**操作**：在 DMIT 客户区查看这台机器的状态（未能核实，以实际界面为准）和下次流量重置时间，然后从下面的做法里选一种（根据 DMIT 官方文档，查证日期 2026-11-17）：

- 等待每月自动重置，见 [实例 FAQ](https://docs.dmit.io/guide/faq/instance)。
- 付费提前重置：多数标准方案可以付费重置，费用等于这台机器的月费（续费价格除以计费周期）。额外流量包只限 sPro 系列。
- 升级到同一系列更大的方案：例如从 DMIT HKG.Pro TINY 升级到 DMIT HKG.Pro STARTER。升级按新旧方案差价计费，升级后按新方案价格计费，暂不支持降级。升级后已暂停的机器是否立即恢复，官方文档没有说明。

**预期结果**：付费重置完成后，这台机器的流量被重置（根据 DMIT 官方文档，查证日期 2026-11-17）。之后按 V1 检查能否连接。

**出错时**：

- 续费后机器仍然暂停：按 [账单 FAQ](https://docs.dmit.io/guide/faq/billing)，已经超出流量的机器，付款后不会恢复，要等下次流量重置；没有超出流量的，通常 10 分钟内自动恢复（根据 DMIT 官方文档，查证日期 2026-11-17）。
- 超过到期日 3 天仍未付款：机器会被删除且数据不能恢复（根据 DMIT 官方文档，查证日期 2026-11-17），要重新下单并从 S1 开始。

<!-- step:trouble-04-target -->
### P4 更换借用网站

**目的**：当前的 `<REALITY_TARGET>` 不再合格（S7 打印 `NOT OK`）或 REALITY 线路不通时，换一个借用网站。

**命令**：先用新的域名运行一次 S7，确认打印 `OK`。然后在服务器上以 root 运行下面的代码块（把 `<REALITY_TARGET>` 换成新的域名，保留单引号）：

```bash server
(
set -eu
NEW_TARGET='<REALITY_TARGET>'
printf '%s\n' "$NEW_TARGET" | grep -Eqx '[A-Za-z0-9-]+(\.[A-Za-z0-9-]+)+' || { echo "ERROR: replace <REALITY_TARGET> with the new host name first" >&2; exit 1; }
sed -i "s/^REALITY_TARGET=.*/REALITY_TARGET=$NEW_TARGET/" /etc/sing-box/guide.env
grep '^REALITY_TARGET=' /etc/sing-box/guide.env
)
```

然后依次重新运行 S9、S10、S11，再做 C1，并在所有 App 里重新导入。Hysteria2 的证书不需要重新生成：客户端靠固定的证书指纹认证服务器。

**预期结果**：打印 `REALITY_TARGET=新的域名`；S9 打印 `OK`，S10 打印 `OK`；重新导入后 V3 里 `reality` 显示延迟。

**出错时**：

- `ERROR: replace <REALITY_TARGET> …`：代码块里的占位符还没换。
- S9 的 `sing-box check` 报错：检查 `guide.env` 里 `REALITY_TARGET=` 这一行是否只有域名。

<!-- section:faq -->
## 常见问题

### DMIT HKG.Pro TINY 的流量够用吗？

取决于你每月的用量：500 GB 是双向流量，实际可用量大约是一半。先用手机和电脑的流量统计估算，再用 D1、D2 核对；不够时可以在同一系列内升级到 DMIT HKG.Pro STARTER（1000 GB 双向流量），暂不支持降级（根据 DMIT 官方文档，查证日期 2026-11-17）。

### 为什么同时用 REALITY 和 Hysteria2？

为了多一条线，两条互为备用。REALITY 走 TCP 443，Hysteria2 走 UDP 443，客户端的 `auto` 组每 1 分钟测试一次，平时用能连通且较快的那一条，一条不通时自动换另一条；但两条线也可能同时失效。

### 能分享给家人或朋友用吗？

只给自己和身边少数信任的人用，不要公开、不要收费。DMIT AUP 允许加密的私人代理，禁止公共代理服务和转售（根据 DMIT 官方文档，查证日期 2026-11-17）；AUP 没有给出多少人算「公共」的标准，所以人越少越稳妥。

### 通过推广链接购买会更贵吗？

DMIT 官方文档没有说明，本文无法确认（根据 DMIT 官方文档，查证日期 2026-11-17）。不想用推广链接，可以直接打开 [DMIT 官网](https://www.dmit.io/) 下单。

### 可以用 WireGuard 或 OpenVPN 吗？

不要用它们连回大陆。DMIT AUP 禁止建立去中国的 VPN 隧道，列举了 L2TP、OpenVPN、PPTP、IPSec、Cisco AnyConnect，并写明「包括但不限于」（根据 DMIT 官方文档，查证日期 2026-11-17）。

### 要不要装 BBR 加速或其他加速脚本？

不要。DMIT AUP 禁止使用不可接受设置的网络加速模块（举例有 FinalSpeed 和非官方 BBR）（根据 DMIT 官方文档，查证日期 2026-11-17）。本文的 Hysteria2 已经配置为不使用 Brutal 模式，不需要额外安装任何东西。

### 访问国内网站也会经过香港吗？

会。本文的客户端配置是全局代理，除了私有地址，所有流量都经过服务器，访问国内网站也会消耗流量、可能变慢。不需要翻墙时，可以在 App 里断开连接。

### 为什么有些网站或服务（例如流媒体、AI）打不开或提示地区不对？

DMIT 不保证任何网站或服务都能访问，IP 是随机分配的，地理信息也不保证（根据 DMIT 官方文档，查证日期 2026-11-17），见 [IP FAQ](https://docs.dmit.io/guide/faq/ip)。这是 IP 本身的问题，与本文的配置无关。

### 可以用 Shadowrocket 或 v2rayN 吗？

可以试，但它们不是 sing-box 官方 App，本文没有核实它们能否正确导入本文的分享链接、是否校验 Hysteria2 的 `pinSHA256`；不校验的话，用分享链接导入的 Hysteria2 线路会接受任何证书。本文以 sing-box 官方 App 加配置文件为主。

### IP 被封了怎么办？

先按 P2 确认，再按 DMIT 规则申请更换：满足条件可免费换，否则每次 US$5（根据 DMIT 官方文档，查证日期 2026-11-17）。换好后改 `guide.env` 里的 IP，重新生成并导入客户端配置。

### 流量用完会怎样？

机器会被暂停，订购页标注 `Paused / Quota Exceeded`；可以等每月自动重置，或付费提前重置，费用等于月费（根据 DMIT 官方文档，查证日期 2026-11-17）。见 P3。

### 买了不满意能退款吗？

新订单 3 日内且流量不超过 30 GB 可以全额退款，但 TOS 写明要扣支付手续费；30 日内可以申请部分退款。续费单、网络不满意、IP 地理位置等情况不退款（根据 DMIT 官方文档，查证日期 2026-11-17），见 [TOS](https://www.dmit.io/pages/tos) 第 19 条。

### 会自动续费吗？

会。TOS 写明服务会自动循环计费，直到你按取消程序取消（第 18.1 条）；要开启或保留自动付款、自动续费，账户必须启用并保持双重验证（2FA）（第 18.9 条）（根据 DMIT 官方文档，查证日期 2026-11-17）。

### 这个教程是 DMIT 官方的吗？

不是。这是个人编写的非官方教程，与 DMIT 没有关联；文中只有 DMIT 下单链接是作者的推广链接。

<!-- section:risks -->
## 风险提示

### 法律风险

在中国大陆使用未经批准的翻墙方法，有被处罚的风险。以下是公开报道里的例子，金额由报道作者引用，本文没有逐一核对其原始官方来源：

- 中国数字时代 2026-06-03 转载的一篇微信文章列出：南昌一名个人因使用 VPN 被警告并罚款人民币 15,000 元（文中未写日期）；一名公司员工用办公电脑观看境外流媒体，被警告并罚款人民币 15,000 元（文中未写日期）；福建宁德寿宁县一名个人 2020 年翻墙，2024 年 7 月被发现后被传唤处罚（文中未写金额）（[中国数字时代](https://chinadigitaltimes.net/chinese/727752.html)）。
- 人权观察（HRW）2026-03-17 的文章：中国公安部 2026-01-31 公布网络犯罪防治法草案，情节严重的最高可罚人民币 5,000,000 元、拘留最长 15 日，并可限制出境 6 个月至 3 年；该文没有提到 VPN 或个人翻墙（[HRW](https://www.hrw.org/zh-hant/news/2026/03/17/china-cybercrime-bill-entrenches-censorship-surveillance)）。

是否使用、怎样使用，请你自己判断并承担后果。

### 没有任何协议能永远不被封锁

下面都是第三方研究或用户自己的报告，不是 DMIT 或官方的确认：

- GFW Report 记录：北京时间 2025-08-20 约 00:34 至 01:48，约 74 分钟内，GFW 对 TCP 443 端口无条件注入伪造的 RST+ACK，影响中国与境外之间的 443 连接，其他端口没有受影响，原因不明（[GFW Report](https://gfw.report/blog/gfw_unconditional_rst_20250820/en/)）。这种情况下 REALITY 也会受影响。
- 2026-01-05 起，Xray-core 讨论区有多名用户报告 REALITY / VLESS 被封，有人整个 IP 连 22 端口都被封，也有人报告同一连接在家宽不通、在 4G 热点能通；这些是用户自述，没有官方确认或系统测量，讨论一直持续到 2026-07-02（[Xray-core 讨论](https://github.com/XTLS/Xray-core/discussions/5494)）。本文没有找到能证明这是一次单一大规模事件的、有方法和日期的报告。
- 2026-10-26 前后，有论坛用户把这一天称为「坠机」日，多个代理节点同时失效，VLESS 和 Hysteria2 都有人报告受影响（[VilaVPN 博客](https://blog.vilavpn.com/blocking-roundup-2026-10/)，2026-10-27；这是商业 VPN 供应商的博客，作者说明只是整理用户自述）。一位博主自己的 9 台 VPS 在 2026-10-26 改 UUID 后有 8 台下线（[elmagnifico.tech](https://elmagnifico.tech/2026/10/27/VPN-Compare/)）。
- 2026-08-20 有用户报告福建福州大部分翻墙服务失效，作者说明没有做正式技术检查（[net4people #651](https://github.com/net4people/bbs/issues/651)）。
- 有研究讨论 Hysteria v2（不使用 Brutal 时）可能存在跨层 RTT 指纹的问题，属于学术讨论，不是实测封锁报告（[net4people #528](https://github.com/net4people/bbs/issues/528)）。另有 USENIX Security 2026 论文分析了 2025-09 泄露的 Geedge Networks 防火墙源代码（[net4people #653](https://github.com/net4people/bbs/issues/653)）。

所以：本文的配置不能保证一直可用；被封时可能需要更换 IP（见 P2）或更换借用网站（见 P4）。本文没有在中国大陆做过任何测试，实际效果因地区和运营商而异。

### 只有一台 VPS 的风险

本文只用一台服务器。这台服务器的 IP 被封、流量用完被暂停、欠费、或者 DMIT 出现故障时，两条线会同时不可用。IP 被封时更换 IP 有条件和等待时间（见 P2），工单回复最长可能要 72 小时（根据 DMIT 官方文档，查证日期 2026-11-17）。如果你的工作或生活离不开它，请提前想好备用的上网方式。本文不讲多台服务器的架构。

<!-- section:changelog -->
## 更新记录

- 2026-11-15：初稿（草稿）。事实最后查证日期 2026-11-15；sing-box 版本按 1.14.4 编写。
- 2026-11-16：在两台 Debian 13（arm64）虚拟机之间测试了服务器设置和客户端连接（范围见「[服务器设置](#服务器设置)」），并按测试结果修改：S8 会再检查一次借用网站，并先生成全部值再写入，任何一个生成命令失败或生成的值为空都会停止、不写入；S10 会等服务开始监听；机器原来用 iptables 管防火墙、并且 `iptables -S INPUT` 读得出原有的链时，S5 改用 iptables 命令加规则；防火墙服务在网络启动前运行，`nftables.service` 重启、重新加载，或者停止后再启动时，都会重新加规则；P1 会检查防火墙规则，并说明规则缺失和时间不同步时怎么修；补充 Windows 上的 V1、P2 命令；D3 的回退命令加上 `--allow-downgrades`。
- 2026-11-16：让一个只拿到本文的 AI agent 在全新的测试虚拟机上照着做完服务器设置，按它要猜的地方修改：写明 AI agent 怎样通过 SSH 非交互运行服务器上的代码块（在 macOS 或 Linux 终端里运行，文件用 LF 换行，第一次连接先由读者确认服务器指纹）；`<REALITY_TARGET>` 加了例子，并说明这个域名还用在哪里；S4 最后会打印要不要重启，D3 也提醒装了新内核要重启；S10 的预期结果不再暗示两行的先后顺序；C1 说明文件保存在哪个目录。
- 2026-11-17：再次核对 DMIT 官网页面、官方文档和公告，以及 sing-box 最新稳定版：价格、方案、规则和 sing-box 版本（1.14.4）都没有变化，这些事实的查证日期改为 2026-11-17。事实表的线路一行改为写明：官方标示的约 15 ms 只是从香港到邻近的一个大陆城市的参考测量值。更正 Tier 1 的说明：来源是 DMIT 的 IP FAQ，它说 T1 系列新分配的 IP 不保证首次连接时在敏感地区可用，而 Premium 和 Eyeball 保证首次连接全球可用（通过工单处理）；原来「IP 也不保证在所有国家和地区都能用」的写法，在所引用的页面上找不到。IP 管理变化一行改为：加收从 2026-12-01 当日或之后出的续费单起（原来写「之后」）。S2 改为说重装可能会换掉服务器的主机密钥，而不是一定会换。S8 改为求助时只贴这一步打印的报错（`ERROR` 开头的那一行，或者 openssl、sing-box 的报错），不要贴 `guide.env` 的内容。
