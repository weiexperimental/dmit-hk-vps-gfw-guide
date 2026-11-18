[簡體中文](README.md) | 繁體中文 | [English](README.en.md)

# DMIT 香港 VPS 翻牆教學：用 sing-box 架設 VLESS + REALITY 與 Hysteria2

<!-- section:intro -->
## 先說結論

本教學用一台 DMIT 香港 VPS 和官方 sing-box，架設兩條只供自己使用的加密代理：一條是 VLESS + XTLS Vision + REALITY（TCP 443），另一條是 Hysteria2（UDP 443）。客戶端每分鐘測試一次兩條線路，自動使用能連通而且較快的那一條。大多數個人用戶可以從 DMIT HKG.Pro TINY 開始：訂購頁標價 US$39.90/月，500 GB 雙向流量，線路是 CN2 GIA 和 CMI（根據 DMIT 官方文件，查證日期 2026-11-17）。伺服器用 Debian，sing-box 從官方 apt 源安裝，所有指令都寫在本文裏，不需要執行任何第三方腳本；客戶端用 iOS、Android、Windows、macOS 上的 sing-box 官方 App。沒有任何協議能保證長期不被封鎖，在中國大陸使用翻牆工具亦有法律風險，開始前請先閱讀「[風險提示](#風險提示)」。

文中事實最後查證於 2026-11-17（每項事實旁邊都註明了各自的查證日期）。文中只有 DMIT 下單連結是推廣連結（[DMIT 下單連結](https://www.dmit.io/aff.php?aff=30385)），指向 DMIT 官網和文件的其他連結都是普通連結，詳見「[推廣連結與聲明](#推廣連結與聲明)」。

目錄：

1. [推廣連結與聲明](#推廣連結與聲明)
2. [DMIT 香港方案事實表](#dmit-香港方案事實表)
3. [如何選擇方案](#如何選擇方案)
4. [DMIT 允許和禁止的用法](#dmit-允許和禁止的用法)
5. [下單時選甚麼](#下單時選甚麼)
6. [需要你替換的三個值](#需要你替換的三個值)
7. [交給你的 AI agent](#交給你的-ai-agent)
8. [伺服器設定](#伺服器設定)
9. [客戶端設定](#客戶端設定)
10. [檢查是否生效](#檢查是否生效)
11. [日常維護](#日常維護)
12. [出問題時怎麼辦](#出問題時怎麼辦)
13. [常見問題](#常見問題)
14. [風險提示](#風險提示)
15. [更新記錄](#更新記錄)

<!-- section:disclosure -->
## 推廣連結與聲明

- 本文中的 DMIT 下單連結 https://www.dmit.io/aff.php?aff=30385 是作者的推廣連結。經這條連結購買，作者會從 DMIT 獲得佣金。按照 [DMIT 推廣 FAQ](https://docs.dmit.io/guide/faq/affiliate)，佣金一般是實付金額的 10%，部分高性價比產品可能降至 5%，部分促銷期間可能升至 20%（根據 DMIT 官方文件，查證日期 2026-11-17）。
- 經推廣連結購買的價格是否與直接購買相同，DMIT 官方文件沒有說明，本文無法確認（根據 DMIT 官方文件，查證日期 2026-11-17）。如果你不想使用推廣連結，可以直接開啟 [DMIT 官網](https://www.dmit.io/) 下單，本文的步驟完全一樣。
- 本文是個人編寫的非官方教學，與 DMIT 沒有任何關聯，不代表 DMIT，亦不是 DMIT 的官方文件。DMIT 的規則以其官網和官方文件為準。
- 本文沒有寫給 AI 看的隱藏內容。「[交給你的 AI agent](#交給你的-ai-agent)」一節是公開的提示詞，你可以自己閱讀、自己修改。

本文文字依 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 授權。轉載或改寫時，請署名並附上本文連結 [https://github.com/weiexperimental/dmit-hk-vps-gfw-guide](https://github.com/weiexperimental/dmit-hk-vps-gfw-guide)。完整授權文本見本倉庫的 `LICENSE` 檔案。

<!-- section:facts -->
## DMIT 香港方案事實表

下表只收錄在 DMIT 官網、官方文件或官方公告中查到的內容。價格和規格隨時可能變動（[TOS](https://www.dmit.io/pages/tos) 第 15 條：已簽約期內價格不會上調，但 DMIT 可以隨時修改牌價和方案資源，不另行通知），下單前請以訂購頁為準。

| 項目 | 內容 | 來源 | 查證日期 |
|---|---|---|---|
| DMIT HKG.Pro TINY（訂購頁名稱 HKG.AS3.Pro.TINY） | US$39.90/月，免開通費；1 vCore、1 GB 記憶體、20 GB SSD、1 個 IPv4、1 段 IPv6 /64；500 GB 雙向流量；端口 1 Gbps | [訂購頁（Premium，AS3）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3) | 2026-11-17 |
| DMIT HKG.Pro STARTER（訂購頁名稱 HKG.AS3.Pro.STARTER） | US$79.90/月，免開通費；1 vCore、2 GB 記憶體、40 GB SSD、1 個 IPv4、1 段 IPv6 /64；1000 GB 雙向流量；端口 1 Gbps | [訂購頁（Premium，AS3）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3) | 2026-11-17 |
| DMIT HKG.EB（Eyeball，官方標註 Beta） | WEE：US$24.90/月，1 vCore、1 GB 記憶體、20 GB SSD，500 GB 雙向流量，500 Mbps；TINY：US$39.90/月，1 vCore、1 GB 記憶體、20 GB SSD，800 GB 雙向流量，1 Gbps | [訂購頁（Eyeball）](https://www.dmit.io/cart.php?region=hong-kong&network=eyeball) | 2026-11-17 |
| 線路（Premium，即 Pro） | 機房 Equinix HK2（葵涌）；經 CN2 GIA（AS23764）和 CMI（AS58453）；官方標示的去大陸平均延遲約 15 ms 是從香港到鄰近的一個大陸城市的參考測量值，丟包低於 0.1%；官方註明只是參考值，實際取決於網絡、路由和時段 | [DMIT 香港機房頁](https://www.dmit.io/pages/location/hong-kong) | 2026-11-17 |
| 直連 | 官方雲端主機頁：與中國電信（AS4809）、中國聯通（AS9929）、中國移動國際（AS58807）直連 peering，Premium 另有 CN2 GIA | [DMIT 雲端主機頁](https://www.dmit.io/pages/cloud-instance) | 2026-11-17 |
| 流量計算 | Pro 和 Eyeball 的訂購頁標註 Bidirectional（雙向）：上載和下載都計算在內 | [訂購頁（Premium，AS3）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3)、[實例 FAQ](https://docs.dmit.io/guide/faq/instance) | 2026-11-17 |
| 超出流量 | 訂購頁標註「Paused / Quota Exceeded」，即暫停；流量每月自動重置，控制台顯示下次重置時間；大多數標準方案可以付費提前重置，費用等於該機月費；額外流量包只限 sPro 系列 | [訂購頁（Premium，AS3）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3)、[實例 FAQ](https://docs.dmit.io/guide/faq/instance) | 2026-11-17 |
| 退款 | 全額退款：新訂單、購買不超過 3 日、流量不超過 30 GB（TOS 寫明要扣除支付手續費）；部分退款：新訂單、購買不超過 30 日；續費單、用帳戶餘額付款（只退回餘額）、對網絡不滿意、IP 地理位置等情況不退款 | [TOS](https://www.dmit.io/pages/tos) 第 19 條、[退款 FAQ](https://docs.dmit.io/guide/faq/refund) | 2026-11-17 |
| 更換 IP | Pro 和 EB 免費更換須同時符合：購買滿 7 日、距上次更換滿 15 日、整個 IP 的 ICMP 和所有 TCP 端口都被封、服務剩餘至少 7 日；開通時已被封可開工單免費更換。付費更換：隨時可申請，每次 US$5，兩次申請至少相隔 7 日，完成後不能撤銷或退款 | [IP FAQ](https://docs.dmit.io/guide/faq/ip) | 2026-11-17 |
| 付款方式 | 帳單 FAQ 提到 PayPal 和信用卡（需要完整的個人和地址資料）；完整的付款方式清單未能核實 | [帳單 FAQ](https://docs.dmit.io/guide/faq/billing) | 2026-11-17 |
| 續費 | 月付機器在到期前 7 天發出續費帳單；到期後保留 3 天，3 天內付款可以恢復，之後刪除且資料不能恢復 | [帳單 FAQ](https://docs.dmit.io/guide/faq/billing) | 2026-11-17 |
| 下單要求 | DMIT 2026-11-14 公告：下單和重裝系統（Reinstall OS）時要提供有效的 SSH 公鑰；私鑰改為在瀏覽器本機產生；客戶區不再保存控制台（Console）的 root 密碼 | [DMIT Telegram 公告](https://t.me/DMIT_INC/1292) | 2026-11-17 |
| IP 管理變動 | DMIT 公告：2026-12-01 起改為自購自管 IP；繼續保留 154 開頭 Cogent IP 的用戶，由 2026-12-01 當日或之後發出的續費單起，每個 IPv4 加收 US$1/月；香港的 Cogent IP 會分批遷移到標準 IP，時間表另行公佈；沒有 154 開頭 IP 的不受影響 | [DMIT Telegram 公告](https://t.me/DMIT_INC/1282) | 2026-11-17 |
| 工單回覆 | TOS 只保證 72 小時內回覆；工單 FAQ 寫通常 24 小時內，假期和週末可能延遲 | [TOS](https://www.dmit.io/pages/tos) 第 3.7 條、[工單 FAQ](https://docs.dmit.io/guide/faq/ticket) | 2026-11-17 |
| 規則版本 | AUP 最後更新 2025-11-12；TOS 最後更新 2026-01-22 | [AUP](https://www.dmit.io/pages/aup)、[TOS](https://www.dmit.io/pages/tos) | 2026-11-17 |

<!-- section:choose -->
## 如何選擇方案

**三個系列比較**，數字來自上面的事實表（根據 DMIT 官方文件，查證日期 2026-11-17）：

| | DMIT HKG.Pro TINY | DMIT HKG.Pro STARTER | DMIT HKG.EB |
|---|---|---|---|
| 月費 | US$39.90 | US$79.90 | WEE US$24.90；TINY US$39.90 |
| 雙向流量 | 500 GB | 1000 GB | WEE 500 GB；TINY 800 GB |
| 端口速度 | 1 Gbps | 1 Gbps | WEE 500 Mbps；TINY 1 Gbps |
| 記憶體 | 1 GB | 2 GB | 1 GB |
| 線路 | Premium：CN2 GIA + CMI | Premium：CN2 GIA + CMI | Eyeball：Tier 1 加「盡力而為」的中國路由（官方標註 Beta） |

怎樣選：

- **先從 DMIT HKG.Pro TINY 開始。** 在本文列出的 Premium 線路方案中，它的月費最低（DMIT HKG.Pro STARTER 是 US$79.90，AN5 世代最便宜的 HKG.AN5.Pro.MINI 是 US$149.90）。DMIT 允許在同一系列內升級到更高配置，按新舊方案差價計費，但暫時不支援降級（根據 DMIT 官方文件，查證日期 2026-11-17），見 [實例 FAQ](https://docs.dmit.io/guide/faq/instance)。所以先買 TINY，流量不夠再升級到 DMIT HKG.Pro STARTER，比一開始買大的再想降級更穩妥。
- **流量用得多，或者需要 2 GB 記憶體，選 DMIT HKG.Pro STARTER。** 流量是 TINY 的兩倍。
- **預算優先、能接受不穩定，才考慮 DMIT HKG.EB。** 官方說明香港 Eyeball 仍是 Beta，路由仍在調整，暫不建議用於需要高穩定性的生產用途；它的中國路由是「盡力而為」（根據 DMIT 官方文件，查證日期 2026-11-17），見 [訂購頁（Eyeball）](https://www.dmit.io/cart.php?region=hong-kong&network=eyeball)、[香港機房頁](https://www.dmit.io/pages/location/hong-kong)。

**雙向流量怎樣估算：** 訂購頁寫的是雙向流量，上載和下載都計算在內（根據 DMIT 官方文件，查證日期 2026-11-17）。代理伺服器轉發時，同一份資料會被計算兩次：網站傳給伺服器一次（入），伺服器再轉給你一次（出）。所以粗略估算時，把你每月經代理實際下載和上載的總量乘以 2，再和方案流量比較。換句話說，500 GB 雙向流量，實際可用量大約是它的一半。你可以先看手機和電腦系統中的每月流量統計來估計自己的用量，上線後再用「[日常維護](#日常維護)」中的方法核對。

**為甚麼不選 Lite 和 Tier 1（T1）：**

- **Tier 1（T1）**：官方描述為「without China Optimization」，即沒有針對中國大陸優化；DMIT 的 IP FAQ 亦指出，T1 系列新分配的 IP 不保證首次連接時在敏感地區可用，而 Premium 和 Eyeball 則保證首次連接全球可用（經工單處理）。它的價格低（例如 T1.TINY US$6.90/月、2000 GB），流量按「IN/OUT Max」計算（根據 DMIT 官方文件，查證日期 2026-11-17），見 [訂購頁（Tier 1）](https://www.dmit.io/cart.php?region=hong-kong&network=tier-1)、[雲端主機頁](https://www.dmit.io/pages/cloud-instance)、[IP FAQ](https://docs.dmit.io/guide/faq/ip)。從大陸使用，主要看重的就是去大陸的線路，所以不推薦。
- **Lite**：在香港訂購頁的 Premium、Eyeball、Tier 1 三個篩選中，以及定價頁上，都沒有看到名為 Lite 的產品（根據 DMIT 官方文件，查證日期 2026-11-17），所以本文不作討論。
- **同一 Premium 組別中的 AN5 世代**：最便宜的是 HKG.AN5.Pro.MINI，US$149.90/月，沒有 TINY 和 STARTER（根據 DMIT 官方文件，查證日期 2026-11-17），見 [訂購頁（Premium，AN5）](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=an5)。訂購頁要選 AS3 世代才能看到 TINY 和 STARTER。

### 公開測試匯總

只收錄寫明了測試方法和日期的第三方測試，以下全部是**第三方測試，不是本文作者的測試**。截至 2026-11-15，沒有找到專門測試現時 DMIT HKG.Pro TINY 或 STARTER（AS3 世代）、又寫明方法和日期的第三方測試。

| 測試對象 | 結果摘要 | 方法 | 日期 | 來源 | 備註 |
|---|---|---|---|---|---|
| HKG.AN5.EB.TINY（Eyeball） | Geekbench 5 單核 2108、多核 2113；磁碟 fio 約 2.1 GB/s；到三網延遲作者評為「good」，但大部分中國測速點用不了；電信回程評為優，聯通和移動評為「可用」；作者建議停用 IPv6；系統資訊顯示 Debian 13 | Geekbench 5、fio、三網測速（截圖）、到 9 個目標的 TCP traceroute、IP 解鎖檢測 | 發表 2026-10-07；截圖日期 2026-09-19 | [hostalk.net](https://hostalk.net/posts/dmit_hkeb.html) | 文中購買連結帶推廣參數，文章沒有明確披露；測的是 AN5 世代 Eyeball，不是 AS3 Pro TINY |
| HKG.Pro.MongKok（舊機型：1 vCPU、1 GB 記憶體、20 GB SSD、200 GB、150 Mbps） | 從大陸三個城市到三網的 traceroute，最後一跳延遲約 11 ms 至 49 ms；作者評電信 CN2 GIA 回程優秀；測速「基本跑滿」 | 30 跳 traceroute（大陸三個城市到三網）、測速和 Geekbench 5（截圖） | 發表 2024-08-07，更新 2025-11-24；截圖日期 2024-08-05 | [hostalk.net](https://hostalk.net/posts/dmit_hkpro.html) | 文中購買連結帶推廣參數，文章沒有明確披露；舊機型，只能作參考 |

<!-- section:rules -->
## DMIT 允許和禁止的用法

以下是 DMIT 可接受使用政策（[AUP](https://www.dmit.io/pages/aup)，最後更新 2025-11-12）和服務條款（[TOS](https://www.dmit.io/pages/tos)，最後更新 2026-01-22）中與本教學有關部分的摘要（根據 DMIT 官方文件，查證日期 2026-11-17）。摘要不能代替原文，請自己閱讀一遍原文。

**允許：**

- 加密的私人代理。AUP 原文寫明允許「Encrypted Private Proxy」。本教學架設的 REALITY 和 Hysteria2 就屬於只供自己使用的加密代理。

**禁止（與本教學有關的）：**

- **公共代理服務（Public Proxy Services）。** AUP 說這類服務可能導致 DMIT 的 IP 段或 ASN 被封。AUP 給出的判斷依據是：收到其他網絡關於該 IP 的垃圾郵件投訴，同時上行和下行流量相等。所以：不要把節點分享給大量的人，不要公開發佈分享連結或設定檔。
- **去中國的 VPN 隧道。** AUP 禁止用服務建立去中國的 VPN 隧道，列舉了 L2TP、OpenVPN、PPTP、IPSec、Cisco AnyConnect，並寫明「包括但不限於」。WireGuard 也是 VPN 隧道，同樣不要用 WireGuard、OpenVPN 這類 VPN 連回大陸。
- **轉售。** AUP 禁止用 DMIT 的產品再分發和銷售，亦禁止以商業目的做二次虛擬化。不要賣節點，不要拿它做機場。
- **長時間大量佔用頻寬。**
- **非官方加速。** AUP 禁止使用不可接受設定的網絡加速模組，例子有 FinalSpeed 和非官方 BBR（Unofficial BBR）。不要安裝任何第三方加速腳本或模組。本教學的 Hysteria2 亦不使用 Brutal 模式（見伺服器設定）。
- **垃圾郵件。** AUP 有單獨的 NO SPAM 一節。
- **其他：** 架設 TOR 出口節點、未經授權的端口掃描。
- **資源限制：** 所有產品預設保證 50% 的 CPU 使用率（另有註明除外），異常或過高的 CPU 用量可能被限制。

**違反之後**，以下內容來自 [AUP](https://www.dmit.io/pages/aup) 和 [TOS](https://www.dmit.io/pages/tos)（根據 DMIT 官方文件，查證日期 2026-11-17）：

- 視乎違規的嚴重程度，DMIT 可以暫停服務（suspension）或對 IP 做 null routing，為期 7 至 30 日，亦可以立即終止服務；DMIT 還可以自行決定、在有通知或沒有通知的情況下，因違反 AUP 終止你對服務的使用。
- DMIT 認為與垃圾郵件有關的帳戶會被立即終止。
- TOS 寫明：因濫用產生的問題不退款（第 19.3 條）；帳戶共用、公共代理、轉讓、把付費資源開放給公眾使用等情況，不屬於可獲補償的情況（第 3.14、20.4 條）。濫用更換 IP 功能，可能被禁止使用該功能，見 [IP FAQ](https://docs.dmit.io/guide/faq/ip)。

<!-- section:order -->
## 下單時選甚麼

本節只說明下單時選甚麼，不逐項截圖講解。

- **先做一件事：** 2026-11-14 起，DMIT 下單時要提供 SSH 公鑰（根據 DMIT 官方公告，查證日期 2026-11-17），見 [DMIT Telegram 公告](https://t.me/DMIT_INC/1292)。所以下單前先在自己電腦上完成「[S1 在自己電腦上產生 SSH 密鑰](#s1-在自己電腦上產生-ssh-密鑰)」，下單時把它印出的那一整行公鑰（以 `ssh-ed25519` 開頭）貼到訂單要求填寫公鑰的地方。
- **方案：** DMIT HKG.Pro TINY，訂購頁依次選 Hong Kong、Premium、AS3 後顯示為 HKG.AS3.Pro.TINY（根據 DMIT 官方文件，查證日期 2026-11-17）。流量需求大就選 DMIT HKG.Pro STARTER；預算優先、能接受 Beta 就選 DMIT HKG.EB。
- **系統：** 選訂購頁提供的最新 Debian 穩定版。本文的指令按 Debian 13 編寫；寫作時 Debian 13（trixie）是 Debian 的穩定版（[debian.org](https://www.debian.org/releases/)，查證日期 2026-11-15）。DMIT 訂購頁具體提供哪些 Debian 版本，本文未能核實。
- **付款週期：** 香港 Premium 和 Eyeball 訂購頁只顯示 Monthly（月付）（根據 DMIT 官方文件，查證日期 2026-11-17）；是否有年付選項，本文未能核實。
- **付款方式：** 信用卡或 PayPal。帳單 FAQ 提到：PayPal 和信用卡付款需要完整的個人和地址資料；PayPal 未驗證、信用卡不支援 3D Secure、Stripe 風控拒絕，都可能導致付款失敗（根據 DMIT 官方文件，查證日期 2026-11-17），見 [帳單 FAQ](https://docs.dmit.io/guide/faq/billing)。
- **下單連結（推廣連結）：** https://www.dmit.io/aff.php?aff=30385 。不想用推廣連結：[DMIT 官網](https://www.dmit.io/)。

<!-- section:placeholders -->
## 需要你替換的三個值

本文程式碼中只有下面三個值需要你自己替換。其他所有值（UUID、密鑰、密碼、證書指紋等）都由伺服器自動產生，保存在伺服器的 `/etc/sing-box/guide.env`。

| 佔位符 | 是甚麼 | 怎樣填 | 例子 |
|---|---|---|---|
| `<SERVER_IP>` | 伺服器的公共 IPv4 地址 | 在 DMIT 客戶區或開通通知中找這台機器的 IPv4 地址（未能核實，以實際介面為準） | `203.0.113.10`（文件專用的示例地址，不要照抄） |
| `<SSH_USER>` | 登入伺服器的用戶名稱 | 看 DMIT 客戶區或開通通知中有沒有寫明（未能核實，以實際介面為準）；沒有寫明的話先試 `root`，不行再試 `debian` | `root` |
| `<REALITY_TARGET>` | REALITY 借用的網站域名 | 見下文 | `www.example.com`（只是格式示例，不要直接使用） |

**替換規則：**

- 程式碼中的佔位符如果放在單引號內，例如 `'<SERVER_IP>'`，替換時保留單引號，把尖括號連同裏面的字一起換掉，例如 `'203.0.113.10'`。
- 忘了替換也不會悄悄寫壞設定：產生密鑰的那一步（S8）發現值中還有 `<` 或 `>` 就會報錯停止，甚麼都不寫；後面寫設定的步驟（S9）亦會因為缺少 `guide.env` 而停止。

**`<REALITY_TARGET>` 怎樣選：**

REALITY 在握手時借用另一個網站的 TLS 特徵，這個網站就是 `<REALITY_TARGET>`。選擇時：

1. 選一個中國大陸以外、在大陸能正常開啟（沒有被封鎖）的網站，填它的域名，例如某個大型網站的 `www` 域名。
2. 它必須支援 TLS 1.3 和 HTTP/2。
3. 只填域名，不要加 `https://`，不要加路徑或端口。

**例子：** `www.microsoft.com`、`www.apple.com`。這只是例子，不是推薦，亦不是預設值，也不表示它們比其他網站更合適。2026-11-16 在本文的測試虛擬機上，S7 對這兩個域名都印出了 `OK`；它們在大陸能否正常開啟（上面第 1 點），本文沒有核實。無論選哪個，都要先在 S7 中檢查。

這個域名還會用作 Hysteria2 自簽證書中的名稱，以及客戶端設定中的 `server_name`。裝好以後想更換，按「[P4 更換借用網站](#p4-更換借用網站)」做。

**怎樣在伺服器上檢查：** 登入伺服器之後，在「[S7 檢查借用網站](#s7-檢查借用網站)」中執行檢查指令。它在伺服器上用 `openssl s_client` 連接這個網站的 443 端口，只允許 TLS 1.3，並請求 HTTP/2（ALPN `h2`），印出 `OK` 才算合格；印出 `NOT OK` 就換一個網站再試。你可以在 S7 中多試幾個候選網站，挑一個合格的再繼續。

<!-- section:agent -->
## 交給你的 AI agent

如果你用 Claude Code、Codex 這類 AI agent 幫你操作，可以把下面這段文字連同本文一起交給它。這段文字是公開寫在這裏的普通文字，你可以先讀一遍、按需要修改；本文沒有任何寫給 AI 看的隱藏內容。

```text
請閱讀這份 README（檔案或連結由我提供），按「伺服器設定」「客戶端設定」「檢查是否生效」三節，幫我設定一台 DMIT 香港 VPS。
伺服器 IP：<SERVER_IP>
SSH 用戶：<SSH_USER>
REALITY 借用網站：<REALITY_TARGET>（如果我沒有填寫，請按 README「需要你替換的三個值」一節的方法挑選幾個候選，在伺服器上用 S7 檢查；S7 合格之後、執行 S8 之前，先問我確認用哪一個）
要求：
1. 涉及付款或修改帳戶設定之前（包括 DMIT 客戶區中的任何操作），先問我，得到我同意才做。
2. 只執行 README 中寫明的指令，不要下載或執行任何第三方安裝腳本。
3. 每一步執行後，對照這一步的「預期結果」；不一致時按「出錯時」處理，處理不了就停下來，把輸出告訴我。
4. 伺服器上的程式碼區塊要以 root 身份執行。經 SSH 非互動執行時，按 README「伺服器設定」開頭「怎樣執行這些指令」的做法：不執行 S3；把之後每個在伺服器上執行的程式碼區塊原樣保存成一個檔案（只替換佔位符；UTF-8 編碼、不帶 BOM、LF 換行），在 macOS 或 Linux 終端機中用 ssh <SSH_USER>@<SERVER_IP> sudo bash -s < 檔案名稱 按順序逐個執行（用 root 登入時去掉 sudo；Windows 上用 WSL 或 Git Bash，本文未測試）。第一次 SSH 連接這台伺服器之前先問我，由我先自己做一次 S2。
5. 不要把 /etc/sing-box/guide.env、/root/dmit-guide/client-profile.json、/root/dmit-guide/share-links.txt 的內容發到任何公開的地方。
6. 不要做 DMIT AUP 禁止的事，例如公共代理、轉售、去中國的 VPN 隧道、非官方加速。
```

<!-- section:server -->
## 伺服器設定

**要做甚麼：** 在伺服器上裝好 sing-box，開兩條線路：VLESS + XTLS Vision + REALITY 監聽 TCP 443，Hysteria2 監聽 UDP 443。最後產生一份客戶端設定檔和兩條分享連結。

**為甚麼是這兩種協議：** 有博客作者根據個人經驗把 VLESS + REALITY 排為最難檢測、最穩定，Hysteria2 速度最快，但作者說明沒有做正式測試（[lilting.ch](https://lilting.ch/en/articles/china-vpn-protocol-comparison)，2026-01-08，更新 2026-05-21）。2026-10-26 前後有用戶報告大量節點同時失效；一位博客作者自己的 9 台 VPS 有 8 台下線，他觀察到 Hysteria2 在這一輪表現不錯，但只是目測，沒有做多地區、多電訊商測試（[elmagnifico.tech](https://elmagnifico.tech/2026/10/27/VPN-Compare/)，2026-10-27）。兩條線路一條走 TCP、一條走 UDP，一條不通時客戶端會自動用另一條；但它們亦可能同時失效，見「[風險提示](#風險提示)」。

**版本：** 寫作時 sing-box 最新穩定版是 1.14.4（[GitHub release](https://github.com/SagerNet/sing-box/releases/tag/v1.14.4)，查證日期 2026-11-17）。本文的 Hysteria2 證書固定方式要求客戶端的 sing-box 不低於 1.13.0（[sing-box TLS 文件](https://sing-box.sagernet.org/configuration/shared/tls/)）。本文的指令按 Debian 13 編寫，不依賴 CPU 架構。2026-11-16 在兩台 Debian 13（arm64）虛擬機之間測試過：伺服器設定的全部指令（用 root 和用普通用戶加 sudo 各一次，亦測試了重複貼上、忘記替換佔位符、機器上原本已有防火牆規則和重新開機），以及客戶端設定檔中兩條線路各自能用、一條被擋住後自動換到另一條。測試時客戶端用的是 Linux 上的 sing-box 命令行版本，並把 `tun` 換成了本機代理端口。同一天亦讓一個只拿到本文的 AI agent，在一台全新的 Debian 13（arm64）測試虛擬機上按照本文修改前的版本做完了伺服器設定，本文已按它的反饋修改；修改後的指令（包括 S4 的內核檢查和非互動的執行方法）是用腳本重新測試的，沒有再讓 agent 按照本文做一次。這些步驟沒有在 DMIT 的機器上、amd64（x86-64）架構的機器上，或者 DMIT 自己提供的 Debian 系統映像上執行過；手機和電腦上的 App 沒有測試，亦沒有在中國大陸測試。

**怎樣執行這些指令：**

- 標明「在你自己的電腦上執行」的程式碼區塊在你自己的電腦上執行；其餘程式碼區塊都在伺服器上執行。
- 從 S3 開始，伺服器上的程式碼區塊都要以 root 身份執行。手動操作時，按 S3 切換到 root，然後在同一個視窗中按順序貼上。
- 用腳本或 AI agent 經 SSH 非互動執行時：
  - 不執行 S3（它只在互動的終端機視窗中有用）。之後每個在伺服器上執行的程式碼區塊，按順序逐個執行。
  - 把程式碼區塊的內容原樣保存成一個檔案，只替換佔位符，其他一個字都不改，例如保存為 `s05.sh`。檔案要用 UTF-8 編碼（不帶 BOM）和 LF 換行。用 CRLF 換行（Windows 上常見）時，伺服器上的 bash 會把每行末尾的回車字元當成指令的一部分，程式碼區塊不能正確執行。
  - 在你自己電腦上的 macOS 或 Linux 終端機中執行 `ssh <SSH_USER>@<SERVER_IP> sudo bash -s < s05.sh`（用 root 登入時去掉 `sudo`）。這樣伺服器執行的，就是手動貼上時的同一段指令。Windows 上可以在 WSL 或 Git Bash 中執行這條指令，本文未測試。
  - 第一次連接這台伺服器時，SSH 要人確認伺服器的指紋（見 S2）。沒有終端機視窗的 agent 無法確認，SSH 會報 `Host key verification failed.` 並退出。所以請你先自己做一次 S2，或者讓 agent 在第一次連接之前先問你。
  - 結束碼不是 0，表示這一步沒有完成；結束碼是 0，亦要對照「預期結果」。
- 每個程式碼區塊都可以單獨貼上，也可以重複貼上：已經產生的密鑰不會重新產生，已經加入的防火牆規則不會重複加入。
- 很多程式碼區塊外面有一對圓括號 `( … )`。它的作用是：程式碼區塊中途出錯時只停止這一段，不會關掉你的終端機視窗。程式碼區塊印出 `ERROR:` 並停止，表示這一步沒有完成，按「出錯時」處理後再貼上同一個程式碼區塊即可。

**會在伺服器上產生的檔案：**

| 檔案 | 內容 |
|---|---|
| `/etc/sing-box/guide.env` | 所有自動產生的值（權限 600，等同你的鑰匙，不要公開） |
| `/etc/sing-box/hy2.crt`、`/etc/sing-box/hy2.key` | Hysteria2 用的自簽證書和私鑰 |
| `/etc/sing-box/config.json` | 伺服器設定 |
| `/root/dmit-guide/client-profile.json` | 客戶端設定檔（匯入 App 用；目錄權限 700，檔案權限 600） |
| `/root/dmit-guide/share-links.txt` | `vless://` 和 `hysteria2://` 分享連結（權限 600） |
| `/usr/local/sbin/sing-box-guide-firewall` | 防火牆腳本 |

<!-- step:server-01-ssh-key -->
### S1 在自己電腦上產生 SSH 密鑰

**目的**：產生一對 SSH 密鑰。下單時把公鑰交給 DMIT，2026-11-14 起下單要提供公鑰（根據 DMIT 官方公告，查證日期 2026-11-17），之後用私鑰登入伺服器，不用密碼。**這一步在下單之前做。**

**指令**（在你自己的電腦上執行，macOS 和 Linux 用「終端機（Terminal）」）：

```bash local
mkdir -p ~/.ssh && chmod 700 ~/.ssh
[ -f ~/.ssh/id_ed25519 ] || ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -C dmit-hk
cat ~/.ssh/id_ed25519.pub
```

`ssh-keygen` 會問你要不要設定密碼短語（passphrase）：設定了更安全，以後每次登入都要輸入；直接按 Enter 則留空。如果電腦上已經有 `~/.ssh/id_ed25519`，這段指令不會覆蓋它，會直接沿用。

Windows 10 和 11 內置 OpenSSH 用戶端（OpenSSH Client），但上面的寫法是 bash。Windows 上請開啟 PowerShell，執行 `ssh-keygen -t ed25519 -C dmit-hk`（按 Enter 接受預設路徑），再執行 `Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub` 查看公鑰。

**預期結果**：最後印出一行以 `ssh-ed25519 ` 開頭、以 ` dmit-hk` 結尾的文字（Windows 上同樣以 ` dmit-hk` 結尾）。這一整行就是公鑰，下單時貼上。目錄中沒有 `.pub` 副檔名的 `id_ed25519` 是私鑰，永遠不要發給任何人。

**出錯時**：

- 提示 `ssh-keygen: command not found`：macOS 和主流 Linux 發行版都內置 OpenSSH；Windows 請在「設定 → 選用功能（Optional features）」中加入「OpenSSH 用戶端（OpenSSH Client）」。
- 不記得已有密鑰的密碼短語：把 `id_ed25519` 換一個檔案名稱（例如 `-f ~/.ssh/id_ed25519_dmit`）重新產生，之後登入時用 `ssh -i ~/.ssh/id_ed25519_dmit …` 指定它。

<!-- step:server-02-login -->
### S2 用 SSH 登入伺服器

**目的**：DMIT 開通後，從自己電腦登入伺服器。

**指令**（在你自己的電腦上執行，先替換 `<SSH_USER>` 和 `<SERVER_IP>`）：

```bash local
ssh <SSH_USER>@<SERVER_IP>
```

**預期結果**：第一次連接會問 `Are you sure you want to continue connecting (yes/no/[fingerprint])?`，輸入 `yes` 再按 Enter。之後出現伺服器的提示符號：root 用戶以 `#` 結尾，普通用戶（例如 `debian`）以 `$` 結尾。

**出錯時**：

- `Permission denied (publickey)`：用戶名稱不對（換 `root` 或 `debian` 再試），或者下單時填的不是這台電腦上的公鑰。重裝系統時也要提供公鑰（根據 DMIT 官方公告，查證日期 2026-11-17），可以在客戶區用正確的公鑰重裝系統（未能核實，以實際介面為準）。
- `Connection timed out` 或一直沒有反應：檢查 IP 是否填對、機器是否已經開通並開機；如果之前能連上、突然連不上，見「[P2 判斷 IP 是否被封並更換 IP](#p2-判斷-ip-是否被封並更換-ip)」。
- 私鑰不在預設路徑：加 `-i 私鑰路徑`，例如 `ssh -i ~/.ssh/id_ed25519_dmit <SSH_USER>@<SERVER_IP>`。
- 重裝系統之後再連接，如果出現 `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`：重裝可能會更換伺服器的主機密鑰（指紋亦會隨之改變），與你電腦中保存的不同。確認這台機器是你自己剛重裝的，在自己電腦上執行 `ssh-keygen -R <SERVER_IP>`，再重新做 S2。

<!-- step:server-03-root -->
### S3 切換到 root

**目的**：之後所有伺服器指令都以 root 身份執行，這樣無論你是用 `root` 還是 `debian` 這類普通用戶登入，後面的程式碼區塊都一樣，也不需要在每條指令前加 `sudo`。

**指令**（在伺服器上執行）：

```bash server
[ "$(id -u)" -eq 0 ] || sudo -i
```

已經是 root 時這條指令甚麼都不做；普通用戶會切換成 root（如果要求輸入密碼，輸入這個普通用戶的密碼）。之後的程式碼區塊都在這個視窗中貼上。斷線重連後，要重新做 S2 和 S3。

**預期結果**：提示符號以 `#` 結尾；輸入 `whoami` 會顯示 `root`。

**出錯時**：

- `sudo: command not found`：表示你登入的是沒有 sudo 的普通用戶，請改用 `root` 登入。
- `is not in the sudoers file`：這個普通用戶沒有管理員權限，請改用 `root` 登入，或開工單問 DMIT 預設用戶是哪一個。
- 用腳本或 AI agent 非互動執行：這一步無法切換後續指令的身份，不要執行它。按「伺服器設定」開頭「怎樣執行這些指令」的做法，用 `sudo bash -s` 逐個執行之後的程式碼區塊。如果提示 `sudo: a terminal is required to read the password`，表示這個用戶使用 sudo 要輸入密碼，非互動執行時無法輸入，請改用 root 登入。

<!-- step:server-04-update -->
### S4 更新系統並安裝基本工具

**目的**：把系統更新到最新，並安裝後面要用的工具：`curl`、`ca-certificates`、`openssl`、`nftables`（防火牆）、`qrencode`（在終端機中顯示二維碼）和 `sudo`（C1 從電腦下載設定檔時要用，用 root 登入時也一樣）。這些都來自 Debian 自己的軟件源。

**指令**（在伺服器上以 root 執行）：

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

`--force-confold` 的意思是：如果某個軟件包的設定檔被改過，保留現有的版本，不彈出詢問。最後幾行比較正在運行的內核（`uname -r`）和 `/boot` 中版本最新的內核，判斷要不要重新開機。

**預期結果**：前面沒有以 `E:` 開頭的錯誤，最後一行是以下三種之一：

- `No reboot needed: …`：正在運行的就是最新的內核，直接做 S5。
- `Reboot recommended: …`：這次裝了新內核，重新開機後才會用上。執行 `reboot` 重新開機（SSH 連線會中斷），等一兩分鐘後重新登入伺服器（手動操作時重新做 S2 和 S3；非互動執行時等到 SSH 能重新連上），再做 S5。也可以先不重新開機，做完後面的步驟再重新開機：本文設定的防火牆和 sing-box 都會開機自動啟動。
- `Cannot tell whether a reboot is needed: …`：`/boot` 中找不到內核檔案，這個檢查無法判斷。可以直接做 S5。

這裏不用 `/var/run/reboot-required` 判斷：在本文的測試虛擬機上，這個檔案是 `unattended-upgrades` 軟件包在內核更新後建立的，移除這個軟件包後，內核更新不再建立它；DMIT 的系統映像有沒有裝這個軟件包，本文未能核實。

**出錯時**：

- `Could not get lock /var/lib/dpkg/lock-frontend`：系統正在後台自動更新，等幾分鐘後再貼上一次。
- `Temporary failure resolving`：伺服器暫時解析不了域名，稍後再試；一直不行就開工單問 DMIT。
- `E: Sub-process /usr/bin/dpkg returned an error code (1)`：在伺服器上以 root 執行 `dpkg --configure -a`，按畫面上的提示操作，完成後再貼上一次 S4。

<!-- step:server-05-firewall -->
### S5 設定防火牆

**目的**：只放行 22/tcp（SSH）、443/tcp（REALITY）和 443/udp（Hysteria2），並且不刪除機器上原有的任何防火牆規則。

這一步做三件事：

1. 先確認 SSH 在 22 端口監聽。如果不是，直接報錯停止，以免把你自己擋在門外。
2. 寫一個小腳本 `/usr/local/sbin/sing-box-guide-firewall`，並馬上執行一次：
   - 機器上**沒有**入站預設拒絕（policy drop）的防火牆時：新建一張只屬於本教學的表 `inet sing_box_guide`。入站預設拒絕，只放行已建立的連線、本機回環、ICMP、DHCP 回應封包，以及 22/tcp、443/tcp、443/udp。
   - 機器上**已經有**入站鏈時：在每條原有入站鏈的最前面加三條放行規則（已建立的連線、22 和 443/tcp、443/udp），規則帶註解 `sing-box-guide`。原有規則一條都不刪、不改。如果原有防火牆是入站預設拒絕的，就不再新建表。之所以要加到原有的鏈中：nftables 中同一個入站鉤子上的每一條基礎鏈都放行，封包才能通過；只在一張新表中放行，擋不住舊表的拒絕。
   - 原有的入站鏈如果是 iptables（Debian 預設的 nf_tables 後端）管理的 `filter` 表 `INPUT` 鏈，並且 `iptables -S INPUT` 讀得出這條鏈，就改用 `iptables`／`ip6tables` 指令加四條放行規則（已建立的連線、22/tcp、443/tcp、443/udp，同樣帶註解 `sing-box-guide`）。直接用 `nft` 往這種鏈中加規則的話，`iptables-save` 會甚麼都不輸出，`netfilter-persistent save` 會保存成空的規則檔案，重新開機後原有規則就沒有了。`iptables -S INPUT` 讀不出的鏈（例如你自己用 nft 寫的、iptables 不認識的規則），仍然用 `nft` 加上面那三條規則。
   - 重複執行不會重複加入規則。
3. 寫一個 systemd 服務 `sing-box-guide-firewall.service`，每次開機時在網絡啟動之前再執行這個腳本。nftables 規則預設不會自動保存，重新開機後就沒有了。系統內置的 `nftables.service` 被重新啟動、重新載入，或者停止後再啟動時，會清空所有規則，這個服務會隨之再執行一次腳本。只有一種情況不會自動恢復：`nftables.service` 原本沒有執行，第一次被啟動。這種情況，以及規則被其他程式清掉的情況，可以用「[P1 連不上時逐項檢查](#p1-連不上時逐項檢查)」發現並恢復。

可選：DMIT 客戶區不再保存控制台（Console）的 root 密碼（根據 DMIT 官方公告，查證日期 2026-11-17）。如果你想在 SSH 出問題時仍能從控制台登入（未能核實，以實際介面為準），先在伺服器上以 root 執行 `passwd` 給 root 設定一個密碼。請使用一個長的隨機密碼（例如執行 `openssl rand -base64 18` 產生後自行妥善保存）。可以先在伺服器上以 root 執行 `sshd -T | grep -E '^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication) '` 查看：如果顯示 `permitrootlogin yes`，同時有 `passwordauthentication yes` 或 `kbdinteractiveauthentication yes`，其他人也能用這個密碼透過 SSH 嘗試登入 root；如果顯示 `permitrootlogin without-password`（即是 `prohibit-password`），root 不能用密碼透過 SSH 登入。

**指令**（在伺服器上以 root 執行）：

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

**預期結果**：最後一行是 `active`。前面幾行會包含下面一種或兩種：

- `Firewall table inet sing_box_guide is active: inbound allows only 22/tcp, 443/tcp and 443/udp.`（機器原本沒有入站預設拒絕的防火牆）；
- `Added accept rules to existing chain: …`，或者 `Added accept rules with iptables to existing chain: …` 和 `Added accept rules with ip6tables to existing chain: …`，以及 `An existing firewall with policy drop was found. …`（機器原本就有防火牆）。

你目前的 SSH 連線不會中斷。可以用 `nft list ruleset` 查看全部規則；機器用 iptables 管理防火牆時，用 `iptables -S INPUT` 和 `ip6tables -S INPUT` 查看。

**出錯時**：

- `ERROR: nothing listens on TCP port 22`：你的 SSH 不在 22 端口。本教學只放行 22，請先確認 SSH 端口（`ss -tlnp | grep ssh`），不要在這種情況下繼續這一步。
- 貼上後 SSH 中斷、連不回去：從 DMIT 客戶區的控制台登入（未能核實，以實際介面為準），這需要你之前設定過 root 密碼；登入後執行 `systemctl disable --now sing-box-guide-firewall.service` 和 `nft delete table inet sing_box_guide`，然後把情況和 `nft list ruleset` 的輸出記下來再排查。
- 你用 ufw 或 firewalld 管理防火牆：它們重新載入時會覆蓋本腳本加入的規則。請改用它們自己的指令放行 22/tcp、443/tcp、443/udp（例如 `ufw allow 443/udp`）。

<!-- step:server-06-install -->
### S6 加入 sing-box 官方軟件源並安裝

**目的**：按 sing-box 官方文件，用官方 apt 源安裝 sing-box（[sing-box 安裝文件](https://sing-box.sagernet.org/installation/package-manager/)，查證日期 2026-11-15）。不使用任何安裝腳本。

軟件包會附帶一個範例 `/etc/sing-box/config.json`，它不是本教學的設定。為免服務在 S9 之前誤用它啟動，這裏把它改名為 `/etc/sing-box/config.json.example`（只在它確實是這個範例檔案時才改名）。改名後檔案不再以 `.json` 結尾，服務不會讀取它。

**指令**（在伺服器上以 root 執行）：

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

**預期結果**：最後印出 `sing-box version 1.14.4`（或更新的版本號）和編譯資訊。服務這時還沒有啟動，這是正常的。

**出錯時**：

- `curl: (6) Could not resolve host` 或下載逾時：伺服器暫時連不上 sing-box.app，稍後再貼上一次。
- `NO_PUBKEY` 或 `The following signatures couldn't be verified`：密鑰檔案沒有下載完整，再貼上一次這個程式碼區塊。
- `Unable to locate package sing-box`：`apt-get update` 沒有讀到 sagernet 源，檢查 `/etc/apt/sources.list.d/sagernet.sources` 的內容是否與上面一致。

<!-- step:server-07-target -->
### S7 檢查借用網站

**目的**：確認你選的 `<REALITY_TARGET>` 從伺服器連接時支援 TLS 1.3 和 HTTP/2。選法見「[需要你替換的三個值](#需要你替換的三個值)」。

**指令**（在伺服器上以 root 執行，先替換 `<REALITY_TARGET>`，保留兩邊的單引號）：

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

**預期結果**：印出 `OK: 你的域名 supports TLS 1.3 and HTTP/2`。

**出錯時**：

- 印出 `NOT OK`：這個網站不支援 TLS 1.3 或 HTTP/2，或者從伺服器連不上。換一個候選網站，改掉單引號內的域名，再貼上一次。
- `NOT OK` 前面出現 `nodename nor servname provided` 或 `Name or service not known`：域名寫錯了，或者你還沒有把 `<REALITY_TARGET>` 換成真實域名。

<!-- step:server-08-env -->
### S8 產生 guide.env 和 Hysteria2 證書

**目的**：一次性產生本機所有密鑰和密碼，保存到 `/etc/sing-box/guide.env`（權限 600）。如果這個檔案已經存在，這一步甚麼都不改，所以重複貼上不會令已經匯入的客戶端失效。

`guide.env` 中的值：

| 名稱 | 內容 |
|---|---|
| `SERVER_IP` | 你填的伺服器 IPv4 |
| `REALITY_TARGET` | 你填的借用網站域名 |
| `UUID` | REALITY 用戶 ID |
| `REALITY_PRIVATE_KEY`、`REALITY_PUBLIC_KEY` | REALITY 密鑰對（私鑰只留在伺服器，公鑰給客戶端） |
| `SHORT_ID` | REALITY short ID |
| `HY2_PASSWORD` | Hysteria2 密碼 |
| `HY2_CERT_SHA256` | 整張 Hysteria2 證書的 SHA-256 指紋（十六進位），給 `hysteria2://` 分享連結的 `pinSHA256` 用 |
| `HY2_PUBKEY_SHA256` | Hysteria2 證書公鑰的 SHA-256（base64），給 sing-box 客戶端的 `certificate_public_key_sha256` 用 |

**Hysteria2 證書怎樣處理：** Hysteria2 必須有 TLS 證書。這裏產生一張自簽證書（有效期 10 年），不需要域名。客戶端預設不信任自簽證書，所以要「固定」（pin）它：sing-box 客戶端設定用 `certificate_public_key_sha256`，填證書公鑰的 SHA-256（base64），sing-box 1.13.0 起支援（[sing-box TLS 文件](https://sing-box.sagernet.org/configuration/shared/tls/)）；`hysteria2://` 分享連結用 `pinSHA256` 參數，按 Hysteria 官方的 URI 格式填伺服器證書的 SHA-256 指紋（[Hysteria URI 格式](https://v2.hysteria.network/docs/developers/URI-Scheme/)）；分享連結為甚麼同時帶 `insecure=1`，見 S11 末尾的說明。這兩個值不一樣，都在這一步算好。sing-box 客戶端設定不用 `insecure: true`，因為它會接受任何證書；也不用 `certificate_sha256`，因為它要求 sing-box 1.15.0，比寫作時的穩定版新。證書中的名稱用你的借用網站域名，客戶端的 `server_name` 也用它。

**指令**（在伺服器上以 root 執行，先替換 `<SERVER_IP>` 和 `<REALITY_TARGET>`，保留單引號）：

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

**預期結果**：第一次執行時可能先印出一行 `-----`（openssl 的輸出），然後是 `Created /etc/sing-box/guide.env:` 和 9 行 `名稱=值`，每一行等號後面都有內容。再次貼上時印出 `… already exists. Nothing was changed. …`。這些值等同你的鑰匙：不要截圖，亦不要貼到論壇、群組或 GitHub issue 中求助；求助時只貼這一步印出的報錯（`ERROR` 開頭的那一行，或者 openssl、sing-box 的報錯），不要貼 `/etc/sing-box/guide.env` 的內容。

**出錯時**：

- `ERROR: replace <SERVER_IP> and <REALITY_TARGET> …`：程式碼區塊中的佔位符還沒換。換好後再貼上，這時甚麼檔案都還沒寫。
- `ERROR: SERVER_IP must be an IPv4 address …` 或 `ERROR: REALITY_TARGET must be a host name only …`：值的格式不對，例如 IP 寫錯，或者域名前面多了 `https://`。
- `ERROR: … does not offer TLS 1.3 with HTTP/2 …`：這一步會再做一次 S7 的檢查，結果不合格。按 S7 換一個借用網站，改掉單引號內的域名後再貼上；這時甚麼檔案都還沒寫。
- 還沒印出 `Created …` 就停止了（前面是 openssl 或 sing-box 的報錯，或者 `ERROR: a generated value is empty …`）：某個產生指令失敗了（例如 sing-box 沒有裝好），`guide.env` 沒有寫入。先看前面的報錯並修正，再貼上這個程式碼區塊。
- 想修改某個值（例如填錯了 IP）：不要刪除 `guide.env`（刪除後會產生新的密鑰，所有客戶端都要重新匯入）。改法見「[P2 判斷 IP 是否被封並更換 IP](#p2-判斷-ip-是否被封並更換-ip)」和「[P4 更換借用網站](#p4-更換借用網站)」。

<!-- step:server-09-config -->
### S9 寫入設定、檢查並啟動

**目的**：用 `guide.env` 中的值產生伺服器設定。先寫到 `/etc/sing-box/config.json.new`，用 `sing-box check` 檢查；只有檢查通過，才替換 `/etc/sing-box/config.json` 並重新啟動服務。檢查不通過時，正在執行的服務和舊設定都不受影響。

設定要點：

- REALITY：VLESS 入站，`flow` 為 `xtls-rprx-vision`，握手借用 `REALITY_TARGET` 的 443 端口（[sing-box VLESS 入站文件](https://sing-box.sagernet.org/configuration/inbound/vless/)）。
- Hysteria2：伺服器不設定頻寬（沒有 `up_mbps` 和 `down_mbps`），並設定 `ignore_client_bandwidth: true`。按 sing-box 文件，這樣 Hysteria2 不使用 Brutal 模式（[sing-box Hysteria2 入站文件](https://sing-box.sagernet.org/configuration/inbound/hysteria2/)）。不需要另外安裝任何加速模組。
- 路由：拒絕以私有 IP 地址（伺服器所在的內部網絡和本機）為目標的連線，以及去往 25 端口（發送電郵）的連線。按 AUP，與垃圾郵件有關的帳戶會被立即終止，判斷公共代理的依據之一亦是其他網絡的垃圾郵件投訴；拒絕 25 端口可以避免節點被人拿去發垃圾郵件。
- 官方的 systemd 服務用 `-C /etc/sing-box` 讀取並合併這個目錄下所有以 `.json` 結尾的檔案（可以用 `systemctl cat sing-box` 查看）。所以這一步先確認目錄中除了 `config.json` 沒有其他 `.json` 檔案，這樣 `sing-box check` 檢查的就是服務實際會讀取的內容。本教學的客戶端檔案放在 `/root/dmit-guide/`，不在這個目錄中。

**指令**（在伺服器上以 root 執行）：

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

**預期結果**：最後一行是 `OK: config checked, installed and sing-box restarted.`。第一次執行時前面可能還有一行 `Created symlink …`。

**出錯時**：

- `ERROR: /etc/sing-box/guide.env not found`：先做 S8。
- `ERROR: the sing-box service would also load: …`：`/etc/sing-box` 中有其他 `.json` 檔案，服務會把它們和 `config.json` 合併。把列出的檔案移出這個目錄（例如移到 `/root/`），再貼上一次。
- `sing-box check` 印出 `FATAL` 並停止：設定沒有替換，服務沒有重新啟動。把 `FATAL` 那一行完整記下來，對照 S8 的輸出檢查 `guide.env` 是否完整，再貼上一次。
- `Job for sing-box.service failed`：執行 `journalctl -u sing-box -n 30 --no-pager` 查看原因。

<!-- step:server-10-listen -->
### S10 確認服務在 443 端口監聽

**目的**：確認 sing-box 正在執行，並且同時在 TCP 443 和 UDP 443 上監聽。S9 重新啟動服務後，sing-box 要一點時間才開始監聽，所以這一步最多會等 10 秒再下結論。

**指令**（在伺服器上以 root 執行）：

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

**預期結果**：印出兩行端口資訊，一行以 `udp` 開頭、一行以 `tcp` 開頭（兩行的先後次序不重要），都帶 `sing-box`；最後一行是 `OK: sing-box is running and listens on TCP 443 and UDP 443`。

**出錯時**：

- `FAIL: sing-box is not running`：看印出來的日誌。常見原因是 443 端口被其他程式佔用，用 `ss -tulpn | grep ':443'` 查看是哪個程式。
- 只缺 TCP 或只缺 UDP：重新貼上 S9，再執行這一步。

<!-- step:server-11-export -->
### S11 產生客戶端設定和分享連結

**目的**：用 `guide.env` 中的值產生客戶端設定檔 `/root/dmit-guide/client-profile.json` 和分享連結檔案 `/root/dmit-guide/share-links.txt`（目錄權限 700，檔案權限 600），檢查客戶端設定，然後把兩者印出來，並為兩條分享連結顯示二維碼。重複執行會得到同樣的內容。這兩個檔案不放在 `/etc/sing-box`，因為服務會讀取那裏所有的 `.json` 檔案。

客戶端設定中有四個出站：`reality`（TCP 443 這條線路）、`hy2`（UDP 443 這條線路）、`auto`（每 1 分鐘測試一次 `reality` 和 `hy2`，自動選用可用而且較快的一條）、`direct`（直連，只用於私有地址）。其他所有流量都走 `auto`，即是全局代理。DNS 經代理用 DNS over HTTPS 查詢，只返回 IPv4 地址。

**指令**（在伺服器上以 root 執行）：

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

**預期結果**：依次印出客戶端設定（一段 JSON，裏面已經沒有 `$` 開頭的變數名稱）、兩條分享連結（一條以 `vless://` 開頭，一條以 `hysteria2://` 開頭），以及兩個二維碼。`sing-box check` 通過時不印出任何內容。這些內容（客戶端設定、分享連結和二維碼）等同你的鑰匙：不要截圖，亦不要貼到論壇、群組或 GitHub issue 中求助；求助時只貼 `ERROR` 或 `FATAL` 開頭的那一行，如果那一行中有 UUID、密碼或密鑰，先換成 `xxx`。

**出錯時**：

- `ERROR: /etc/sing-box/guide.env not found`：先做 S8。
- `sing-box check` 印出 `FATAL`：把那一行記下來；通常是 `guide.env` 某一行不完整，對照 S8 的預期結果檢查。
- 二維碼顯示錯亂：把終端機視窗拉寬，或者改用分享連結的文字。

分享連結說明：

- `vless://` 連結用常見的 VLESS 分享格式。各個 App 能否匯入，本文未能核實。
- `hysteria2://` 連結按 Hysteria 官方 URI 格式寫成 `insecure=1&pinSHA256=…`（[Hysteria URI 格式](https://v2.hysteria.network/docs/developers/URI-Scheme/)）。原因是伺服器用的是自簽證書：官方 Hysteria 客戶端用 `insecure` 決定是否跳過常規的 CA 證書鏈驗證，再在證書驗證回調中比對 `pinSHA256`（[Hysteria 客戶端原始碼](https://github.com/HyNetworks/hysteria/blob/master/app/cmd/client.go)）；而 Go 的 TLS 程式庫規定，常規驗證失敗時根本不會呼叫這個回調（[Go crypto/tls 文件](https://pkg.go.dev/crypto/tls#Config)）。所以自簽證書配 `insecure=0` 會連不上，`insecure=1&pinSHA256=…` 的意思是「不驗證 CA 證書鏈，但仍然驗證證書指紋」（查證日期 2026-11-15）。
- **這條 Hysteria2 連線的安全完全靠 `pinSHA256`。** 如果某個 App 忽略 `pinSHA256`，它會接受任何證書，有人在中途攔截時可能取得你的 Hysteria2 密碼。哪些 App 會驗證 `pinSHA256`，本文未能核實（查證日期 2026-11-15）。所以請優先使用下面客戶端設定中的 sing-box 設定檔，它用 `certificate_public_key_sha256` 固定了證書；用不了設定檔時，才匯入 `hysteria2://` 連結。
- 某個 App 用 `hysteria2://` 連結連不上：確認匯入後 `pinSHA256`（證書指紋）還在、「允許不安全證書」一類的選項已經開啟（對應 `insecure=1`）。確認不了這個 App 會驗證 `pinSHA256` 的話，不要用這個連結，改用設定檔。

<!-- section:client -->
## 客戶端設定

**做法：** 四個平台都用 sing-box 官方 App，匯入伺服器 S11 產生的 `client-profile.json`。按 sing-box 官方文件，這些 App 都可以管理和執行本機或遠端的 sing-box 設定檔（[sing-box 客戶端文件](https://sing-box.sagernet.org/clients/)，查證日期 2026-11-15）；能否匯入 `vless://`、`hysteria2://` 分享連結，官方文件沒有提及。所以本文以設定檔為主。

**兩條線路怎樣自動切換：** 設定檔中的 `auto` 群組每 1 分鐘測試一次 `reality` 和 `hy2`，自動使用能連通而且較快的那一條；一條連不上時，下一次測試後會自動換到另一條。你不需要手動選節點。

**關於介面：** 下面 C2 至 C5 的 App 操作是根據 sing-box 官方文件整理的，沒有在實機上逐一測試；不同版本的選單名稱可能略有不同。sing-box 官方文件提醒：有些第三方項目聲稱使用 sing-box，但未獲認可，往往質素差、有廣告（[sing-box 客戶端文件](https://sing-box.sagernet.org/clients/)）。請只從下面列出的官方渠道下載。

**安全：** `client-profile.json` 和分享連結等同你的登入憑證。不要經公開的地方、聊天群組或雲端儲存傳送它們；用數據線、AirDrop 或你信任的私人方式傳到自己的裝置上。

<!-- step:client-01-profile -->
### C1 把客戶端設定檔下載到電腦

**目的**：把伺服器上的 `client-profile.json` 和 `share-links.txt` 複製到你自己的電腦，再傳到手機或在電腦上匯入。

**指令**（在你自己的電腦上執行，先替換 `<SSH_USER>` 和 `<SERVER_IP>`）：

```bash local
ssh <SSH_USER>@<SERVER_IP> sudo cat /root/dmit-guide/client-profile.json > client-profile.json
ssh <SSH_USER>@<SERVER_IP> sudo cat /root/dmit-guide/share-links.txt > share-links.txt
chmod 600 client-profile.json share-links.txt
head -c 200 client-profile.json
```

如果你不方便用上面的指令（例如在 Windows 上），也可以在伺服器的 root 視窗中執行 `cat /root/dmit-guide/client-profile.json`，選取全部輸出並複製，在電腦上用文字編輯器新建檔案貼上，保存為 `client-profile.json`（編碼選 UTF-8）。

這個檔案的內容就是下面這個範本，`$` 開頭的部分已經由伺服器換成了 `guide.env` 中的值（S11 用的正是這個範本）：

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

幾點說明：`uTLS` 在 sing-box 文件中被標為「不推薦」，但 REALITY 客戶端必須啟用 uTLS（去掉 `utls` 後 `sing-box check` 會報錯 `uTLS is required by reality client`），這裏用 `chrome` 指紋。`inbounds` 中 `tun` 的兩個 `address` 是 App 在你裝置上建立的虛擬網卡地址，不是伺服器地址。`dns` 中的 `server` 是 Cloudflare 的公共 DNS，查詢經代理（`auto`）發出。

**預期結果**：目前目錄（你執行這些指令時所在的目錄，可以用 `pwd` 查看）出現 `client-profile.json` 和 `share-links.txt`；`head` 印出的開頭是 `{` 和 `"log"`。

**出錯時**：

- `sudo: a terminal is required to read the password`：你的普通用戶使用 sudo 要輸入密碼，這種情況下用上面「複製貼上」的辦法。
- `No such file or directory`：伺服器上還沒有做 S11。
- 檔案是空的：看指令輸出的錯誤訊息；也可以改用複製貼上的辦法。
- 以後改過 `guide.env`（換 IP、換借用網站）：要重新執行 S11，再做一次 C1，並在 App 中重新匯入。

<!-- step:client-02-ios -->
### C2 iOS：sing-box 官方 App

**目的**：在 iPhone 或 iPad 上匯入設定並連接。

**操作**（根據 sing-box 官方文件，查證日期 2026-11-15）：

1. 用非中國大陸地區的 Apple 帳戶，在 App Store 安裝 sing-box 官方 App（官方文件稱為「sing-box for Apple platforms」，要求 iOS 15.0 或更新版本）。TestFlight 測試版名額只限贊助者。來源：[sing-box Apple 客戶端文件](https://sing-box.sagernet.org/clients/apple/)。
2. 把 `client-profile.json` 傳到裝置上（例如用 AirDrop，或者用數據線放進「檔案（Files）」App）。
3. 在 sing-box App 的設定檔（Profiles）頁面新建一個本機設定檔，選擇從檔案匯入，選取 `client-profile.json`。
4. 回到首頁選取這個設定檔，點啟動；系統第一次會要求加入 VPN 設定，選允許。

**預期結果**：App 顯示已連接，系統狀態列出現 VPN 圖示；按「[檢查是否生效](#檢查是否生效)」檢查出口 IP。

**出錯時**：

- App Store 搜尋不到：中國大陸區的 Apple 帳戶下載不了，官方文件要求非中國大陸帳戶。
- 匯入時報錯：確認檔案是完整的 JSON（用文字編輯器開啟，第一行是 `{`，最後一行是 `}`），然後重新匯入。
- 備選（不是 sing-box 官方 App，本文不推薦）：Shadowrocket（App Store，US$2.99），它的版本記錄提到 VLESS 和 Hysteria 2 的修正，以及經伺服器 URL、訂閱和二維碼匯入（[App Store 頁面](https://apps.apple.com/us/app/shadowrocket/id932747118)，查證日期 2026-11-15）。它能否正確匯入本文的分享連結、是否驗證 `pinSHA256`，本文未能核實；如果它不驗證 `pinSHA256`，用分享連結匯入的 Hysteria2 線路會接受任何證書。

<!-- step:client-03-android -->
### C3 Android：sing-box 官方 App

**目的**：在 Android 手機上匯入設定並連接。

**操作**（根據 sing-box 官方文件，查證日期 2026-11-15）：

1. 安裝 sing-box 官方 App（官方文件稱為 SFA，要求 Android 5.0 或更新版本）。官方渠道：Google Play（含 Beta）、[GitHub Releases](https://github.com/SagerNet/sing-box/releases)、F-Droid（F-Droid 版使用統一簽名，可重現構建）。來源：[sing-box Android 客戶端文件](https://sing-box.sagernet.org/clients/android/)。
2. 把 `client-profile.json` 傳到手機上。
3. 在 App 的設定檔（Profiles）頁面新建一個本機設定檔，選擇從檔案匯入，選取 `client-profile.json`。
4. 回到首頁選取這個設定檔，點啟動；系統第一次會要求建立 VPN 連線，選「確定」（OK）（未能核實，以實際介面為準）。

**預期結果**：App 顯示已連接，通知欄出現 VPN 圖示（鑰匙圖示）。

**出錯時**：

- 打不開 Google Play：改用 GitHub Releases 或 F-Droid 下載。
- 裝了名字相近的第三方 App：解除安裝，只用上面列出的官方渠道。
- 匯入時報錯：同 C2。

<!-- step:client-04-windows -->
### C4 Windows：sing-box 官方桌面 App

**目的**：在 Windows 電腦上匯入設定並連接。

**操作**（根據 sing-box 官方文件，查證日期 2026-11-15）：

1. 從 [GitHub Releases](https://github.com/SagerNet/sing-box/releases) 下載 sing-box 官方桌面 App（官方文件稱為「sing-box for Desktop」，列出 Windows 10 或更新版本，支援 x64、x86、arm64）。來源：[sing-box 桌面客戶端文件](https://sing-box.sagernet.org/clients/desktop/)。注意：它的程式碼儲存庫說明把 Windows 客戶端標為「Experimental」（實驗性）（[sing-box-for-desktop](https://github.com/SagerNet/sing-box-for-desktop)），文件和儲存庫說明的說法不一致，成熟度本文未能核實。
2. 開啟 App，新建一個本機設定檔，從檔案匯入 `client-profile.json`。
3. 選取這個設定檔並啟動。

**預期結果**：App 顯示已連接，瀏覽器能開啟境外網站。

**出錯時**：

- 啟動失敗或提示權限不足：設定中使用了虛擬網卡（tun），可以試試以管理員身份執行 App（Run as administrator）。
- 桌面 App 無法使用：備選（不是 sing-box 官方 App）是 v2rayN（[GitHub](https://github.com/2dust/v2rayN)），它支援 Xray 和 sing-box 內核；它的說明沒有提及分享連結和 JSON 設定匯入，本文未能核實。

<!-- step:client-05-macos -->
### C5 macOS：sing-box 官方 App

**目的**：在 Mac 上匯入設定並連接。

**操作**（根據 sing-box 官方文件，查證日期 2026-11-15）：

1. 安裝 sing-box 官方 App（官方文件稱為 SFM，與 iOS 版同屬「sing-box for Apple platforms」，要求 macOS 13.0 或更新版本，以及非中國大陸地區的 Apple 帳戶）。官方文件列出的 macOS 獨立版渠道：Homebrew Cask（如果你已經裝了 Homebrew，可以在終端機執行 `brew install sfm`），或者 [GitHub Releases](https://github.com/SagerNet/sing-box/releases)。是否在 Mac App Store 上架，本文未能核實。來源：[sing-box Apple 客戶端文件](https://sing-box.sagernet.org/clients/apple/)。
2. 開啟 App，新建一個本機設定檔，從檔案匯入 `client-profile.json`。
3. 選取這個設定檔並啟動；系統第一次會要求加入 VPN 設定或系統延伸功能，選允許。

**預期結果**：App 顯示已連接，選單列（menu bar）或系統設定中顯示 VPN 已連接。

**出錯時**：

- 系統提示延伸功能被阻擋：在「系統設定 → 私隱與安全性（Privacy & Security）」中允許，然後重新啟動連線。
- 沒有 Homebrew：不要為此去執行 Homebrew 的安裝腳本，直接從 GitHub Releases 下載即可。

<!-- section:verify -->
## 檢查是否生效

<!-- step:verify-01-exit-ip -->
### V1 檢查出口 IP

**目的**：確認流量從你的 DMIT 伺服器出去。

**指令**（在已經連上代理的電腦上執行；手機上直接用瀏覽器開啟 https://ipinfo.io/json ）：

```bash local
curl -s https://ipinfo.io/json
```

Windows 上請在 PowerShell 中輸入 `curl.exe -s https://ipinfo.io/json`。Windows PowerShell 5.1 中的 `curl` 是 `Invoke-WebRequest` 的別名，不是 curl 本身；Windows 10 和 11 內置 curl，寫成 `curl.exe` 才會用到它（[Microsoft 文件](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-5.1)、[curl 官網](https://curl.se/windows/microsoft.html)，查證日期 2026-11-16）。

**預期結果**：`"ip"` 等於你的伺服器 IP（`<SERVER_IP>`）。`"country"` 可能顯示 `HK`，`"org"` 顯示伺服器所屬的網絡。注意：DMIT 的 IP 是隨機分配的，DMIT 不保證 IP 的地理資訊（根據 DMIT 官方文件，查證日期 2026-11-17），見 [IP FAQ](https://docs.dmit.io/guide/faq/ip)；所以只要 `"ip"` 對得上，就表示代理正在運作。

**出錯時**：

- 顯示的是你本地網絡的 IP：代理沒有生效。確認 App 顯示已連接；電腦上確認沒有其他代理軟件同時執行。
- 打不開：按「[P1 連不上時逐項檢查](#p1-連不上時逐項檢查)」排查。

<!-- step:verify-02-dns-leak -->
### V2 基本的 DNS 洩漏檢查

**目的**：確認域名查詢也經過代理，而不是發給你本地的電訊商。

**操作**：連上代理後，在瀏覽器開啟 https://www.dnsleaktest.com/ ，點「Standard test」，等待結果。

**預期結果**：結果列表中的 DNS 伺服器都屬於 Cloudflare（本文設定的 DNS 是 Cloudflare 的公共 DNS，查詢經代理發出），而不是你所在地的電信、聯通、移動等電訊商。

**出錯時**：

- 結果中出現你本地電訊商的 DNS：確認匯入的是 S11 產生的完整設定（裏面有 `hijack-dns` 規則），中斷後重新連接再測試一次。
- 結果顯示的是 Google 等其他 DNS 供應商：通常是瀏覽器自己開啟了「安全 DNS」，它的查詢同樣經過代理，不算洩漏到本地電訊商。
- 網站打不開：先完成 V1。

<!-- step:verify-03-both-lines -->
### V3 確認兩條線路都通

**目的**：確認 REALITY 和 Hysteria2 兩條線路都能用，這樣一條出問題時另一條能頂上。

**操作**：在 sing-box App 中開啟群組（Groups）頁面，找到 `auto` 群組，查看或重新整理 `reality` 和 `hy2` 的延遲測試結果（介面名稱根據 sing-box 官方 App 整理，未在實機上測試，可能因版本不同而略有差異）。

**預期結果**：`reality` 和 `hy2` 都顯示一個毫秒數值，而不是逾時或錯誤。

**出錯時**：

- 只有 `hy2` 逾時：你所在的網絡可能限制了 UDP，或者伺服器的 UDP 443 沒有放行。在伺服器上重新執行 S10；本地網絡限制 UDP 時，只用 `reality` 也可以。
- 只有 `reality` 逾時：在伺服器上重新執行 S7，確認借用網站仍然合格；不合格就按「[P4 更換借用網站](#p4-更換借用網站)」處理。
- 兩條都逾時：按「[P1 連不上時逐項檢查](#p1-連不上時逐項檢查)」排查。

<!-- section:daily -->
## 日常維護

<!-- step:daily-01-panel -->
### D1 在 DMIT 客戶區查看流量

**目的**：了解本月已用流量和下次重置時間，避免流量用完被暫停。

**操作**：登入 DMIT 客戶區，開啟這台機器的管理頁面查看流量（未能核實，以實際介面為準）。按 [實例 FAQ](https://docs.dmit.io/guide/faq/instance)，流量每月自動重置，控制台會顯示下次重置時間（根據 DMIT 官方文件，查證日期 2026-11-17）。

**預期結果**：能看到下次重置時間（根據 DMIT 官方文件，查證日期 2026-11-17）；已用流量顯示在哪裏（未能核實，以實際介面為準）。DMIT HKG.Pro TINY 和 DMIT HKG.Pro STARTER 是雙向計費，上載和下載都計算在內（根據 DMIT 官方文件，查證日期 2026-11-17）。

**出錯時**：

- 找不到流量資訊：開工單問 DMIT；TOS 只保證 72 小時內回覆（根據 DMIT 官方文件，查證日期 2026-11-17）。
- 流量已經快用完：見「[P3 流量用完或服務被暫停](#p3-流量用完或服務被暫停)」。

<!-- step:daily-02-vnstat -->
### D2 在伺服器上用 vnstat 統計流量

**目的**：在伺服器上自己記錄每月流量，和 DMIT 客戶區的數字對照。`vnstat` 來自 Debian 自己的軟件源。

**指令**（在伺服器上以 root 執行，先做 S2、S3）：

```bash server
(
set -eu
DEBIAN_FRONTEND=noninteractive apt-get -y install vnstat
systemctl enable --now vnstat.service
vnstat -m
)
```

**預期結果**：剛安裝時會顯示還沒有數據（例如 `eth0: No data.`，網卡名稱可能不同），過一段時間再執行 `vnstat -m`，會按月顯示 `rx`（收）、`tx`（發）和 `total`（合計）。DMIT 是雙向計費（根據 DMIT 官方文件，查證日期 2026-11-17），對照時看 `total`。vnstat 只從安裝時開始統計，以 DMIT 客戶區的數字為準。

**出錯時**：

- `Unable to open database` 或一直沒有數據：執行 `systemctl status vnstat` 看服務是否正在執行，等幾分鐘後再看。

<!-- step:daily-03-upgrade -->
### D3 升級系統和 sing-box

**目的**：定期安裝系統安全更新，並把 sing-box 升級到官方源的最新穩定版。

**指令**（在伺服器上以 root 執行，先做 S2、S3）：

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

**預期結果**：印出新的 `sing-box version …`，最後一行是 `active`。這次升級如果裝了新內核，要重新開機才會用上；不確定的話，再貼上一次 S4，看它的最後一行。

**出錯時**：

- `sing-box check` 印出 `FATAL` 並停止：新版本不接受現在的設定，服務沒有重新啟動，正在執行的仍是舊版本的進程（sing-box 軟件包升級時不會自行重新啟動服務）。但服務一旦重新啟動（包括伺服器重新開機），就會用新版本啟動並失敗，所以要盡快處理。先閱讀 [sing-box 發佈說明](https://github.com/SagerNet/sing-box/releases)；需要退回舊版本時，執行 `apt-cache policy sing-box` 查看可用版本，再執行 `apt-get install --allow-downgrades sing-box=版本號` 安裝舊版本，然後執行 `systemctl restart sing-box.service`。
- 手機或電腦上的 sing-box App 也要保持更新；客戶端版本不能低於 1.13.0。

<!-- section:trouble -->
## 出問題時怎麼辦

<!-- step:trouble-01-checklist -->
### P1 連不上時逐項檢查

**目的**：按順序排除常見原因。

**指令**（在伺服器上以 root 執行，先做 S2、S3）：

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

然後逐項檢查：

1. **伺服器能不能登入？** 連 SSH 都連不上，先在 DMIT 客戶區確認這台機器正在運行（未能核實，以實際介面為準）；機器在運行但從大陸連不上，見 P2。
2. **DMIT 有沒有暫停服務？** 訂購頁寫明，超出流量後機器會被暫停，標註為 Paused / Quota Exceeded（根據 DMIT 官方文件，查證日期 2026-11-17），見 P3；月付機器到期後只保留 3 天，3 天內付款可以恢復（根據 DMIT 官方文件，查證日期 2026-11-17）。
3. **服務、端口和防火牆：** 上面的指令前兩行應是兩個 `active`，並且列出 TCP 和 UDP 443，還要印出 `firewall rules: OK`。服務或端口不對的話，重新執行 S5、S9、S10。印出 `firewall rules: MISSING` 時，表示本教學加入的防火牆規則被清掉了（例如有人重新載入了其他防火牆設定），防火牆服務卻可能仍顯示 `active`：執行 `systemctl restart sing-box-guide-firewall.service`，再執行一次上面的指令。如果它提示 `start of the service was attempted too often`（短時間內重新啟動太多次），先執行 `systemctl reset-failed sing-box-guide-firewall.service`，再重新啟動一次。
4. **時間要準確：** 伺服器應顯示 `System clock synchronized: yes`；手機和電腦要開啟「自動設定時間」。時間相差太多時，TLS 證書驗證可能失敗。伺服器顯示 `no` 時，執行 `timedatectl set-ntp true`，一兩分鐘後再執行一次上面的指令；如果顯示 `NTP service: n/a`（系統中沒有可用的時間同步服務，這時 `timedatectl set-ntp true` 會報 `NTP not supported`），先執行 `apt-get -y install systemd-timesyncd`，再執行 `timedatectl set-ntp true`（[timedatectl 手冊](https://manpages.debian.org/trixie/systemd/timedatectl.1.en.html)，查證日期 2026-11-16）。
5. **客戶端設定是不是最新的：** 改過 `guide.env` 之後，要重新執行 S11 和 C1，並在 App 中重新匯入。
6. **借用網站還合格嗎：** 重新執行 S7。不合格見 P4。
7. **只有一條線路不通：** 見 V3 的「出錯時」。
8. **日誌：** 最後一部分是 sing-box 最近的日誌，留意以 `ERROR` 或 `FATAL` 開頭的行。

**預期結果**：找到並修正其中一項後，客戶端重新連接就能使用。

**出錯時**：以上都正常但仍然連不上，可能是 IP 被封，見 P2。

<!-- step:trouble-02-ip-blocked -->
### P2 判斷 IP 是否被封並更換 IP

**目的**：判斷伺服器 IP 是否在大陸被封；被封時按 DMIT 規則更換 IP，並更新設定。

**指令**（先在你自己在大陸的電腦上、**關閉代理**後執行，替換 `<SERVER_IP>`）：

```bash local
ping -c 4 <SERVER_IP>
nc -vz -w 5 <SERVER_IP> 22
nc -vz -w 5 <SERVER_IP> 443
```

Windows 的 `ping` 沒有 `-c`，用 `/n` 指定次數。Windows 上請在 PowerShell 中依次執行 `ping /n 4 <SERVER_IP>`、`Test-NetConnection <SERVER_IP> -Port 22` 和 `Test-NetConnection <SERVER_IP> -Port 443`；結果中 `TcpTestSucceeded : False` 表示這個端口連不上（[ping 文件](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ping)、[Test-NetConnection 文件](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection)，查證日期 2026-11-16）。

如果 ping 全部丟包、22 和 443 都連不上，而伺服器沒有因為流量用完或欠費被暫停，IP 很可能被封了。

**更換 IP 的規則**，見 [IP FAQ](https://docs.dmit.io/guide/faq/ip)（根據 DMIT 官方文件，查證日期 2026-11-17）：

- 免費更換，要同時符合：購買滿 7 日；距上次更換滿 15 日；整個 IP 的 ICMP 和所有 TCP 端口都被封；服務剩餘有效期至少 7 日。開通時已經被封，可以開工單免費更換。
- 付費更換：隨時可申請，每次 US$5，兩次申請至少相隔 7 日；完成後不能撤銷或退款；濫用可能被禁止使用這個功能。
- 經工單申請。TOS 只保證 72 小時內回覆，工單 FAQ 寫通常 24 小時內。

**拿到新 IP 之後**，用新 IP 重新做 S2、S3，然後執行下面的程式碼區塊（把 `<SERVER_IP>` 換成**新的** IP，保留單引號）：

```bash server
(
set -eu
NEW_IP='<SERVER_IP>'
printf '%s\n' "$NEW_IP" | grep -Eqx '[0-9]{1,3}(\.[0-9]{1,3}){3}' || { echo "ERROR: replace <SERVER_IP> with the new IPv4 address first" >&2; exit 1; }
sed -i "s/^SERVER_IP=.*/SERVER_IP=$NEW_IP/" /etc/sing-box/guide.env
grep '^SERVER_IP=' /etc/sing-box/guide.env
)
```

然後重新執行 S11，再做 C1，並在所有 App 中重新匯入。伺服器設定中沒有寫 IP，不需要重做 S9。

**預期結果**：印出 `SERVER_IP=新的IP`；重新匯入後，V1 顯示的出口 IP 是新 IP。

**出錯時**：

- `ERROR: replace <SERVER_IP> …`：程式碼區塊中的佔位符還沒換成新 IP。
- 不符合免費條件：可以選擇付費更換，或等條件符合後再申請。

<!-- step:trouble-03-traffic -->
### P3 流量用完或服務被暫停

**目的**：了解流量用完後的處理方式。

**操作**：在 DMIT 客戶區查看這台機器的狀態（未能核實，以實際介面為準）和下次流量重置時間，然後從下面的做法中選一種（根據 DMIT 官方文件，查證日期 2026-11-17）：

- 等待每月自動重置，見 [實例 FAQ](https://docs.dmit.io/guide/faq/instance)。
- 付費提前重置：大多數標準方案可以付費重置，費用等於這台機器的月費（續費價格除以計費週期）。額外流量包只限 sPro 系列。
- 升級到同一系列更大的方案：例如從 DMIT HKG.Pro TINY 升級到 DMIT HKG.Pro STARTER。升級按新舊方案差價計費，升級後按新方案價格計費，暫不支援降級。升級後已暫停的機器是否立即恢復，官方文件沒有說明。

**預期結果**：付費重置完成後，這台機器的流量會被重置（根據 DMIT 官方文件，查證日期 2026-11-17）。之後按 V1 檢查能否連接。

**出錯時**：

- 續費後機器仍然暫停：按 [帳單 FAQ](https://docs.dmit.io/guide/faq/billing)，已經超出流量的機器，付款後不會恢復，要等下次流量重置；沒有超出流量的，通常 10 分鐘內自動恢復（根據 DMIT 官方文件，查證日期 2026-11-17）。
- 超過到期日 3 天仍未付款：機器會被刪除且資料不能恢復（根據 DMIT 官方文件，查證日期 2026-11-17），要重新下單並從 S1 開始。

<!-- step:trouble-04-target -->
### P4 更換借用網站

**目的**：當目前的 `<REALITY_TARGET>` 不再合格（S7 印出 `NOT OK`）或 REALITY 線路不通時，換一個借用網站。

**指令**：先用新的域名執行一次 S7，確認印出 `OK`。然後在伺服器上以 root 執行下面的程式碼區塊（把 `<REALITY_TARGET>` 換成新的域名，保留單引號）：

```bash server
(
set -eu
NEW_TARGET='<REALITY_TARGET>'
printf '%s\n' "$NEW_TARGET" | grep -Eqx '[A-Za-z0-9-]+(\.[A-Za-z0-9-]+)+' || { echo "ERROR: replace <REALITY_TARGET> with the new host name first" >&2; exit 1; }
sed -i "s/^REALITY_TARGET=.*/REALITY_TARGET=$NEW_TARGET/" /etc/sing-box/guide.env
grep '^REALITY_TARGET=' /etc/sing-box/guide.env
)
```

然後依次重新執行 S9、S10、S11，再做 C1，並在所有 App 中重新匯入。Hysteria2 的證書不需要重新產生：客戶端靠固定的證書指紋認證伺服器。

**預期結果**：印出 `REALITY_TARGET=新的域名`；S9 印出 `OK`，S10 印出 `OK`；重新匯入後 V3 中 `reality` 顯示延遲。

**出錯時**：

- `ERROR: replace <REALITY_TARGET> …`：程式碼區塊中的佔位符還沒換。
- S9 的 `sing-box check` 報錯：檢查 `guide.env` 中 `REALITY_TARGET=` 這一行是否只有域名。

<!-- section:faq -->
## 常見問題

### DMIT HKG.Pro TINY 的流量夠用嗎？

視乎你每月的用量：500 GB 是雙向流量，實際可用量大約是一半。先用手機和電腦的流量統計估算，再用 D1、D2 核對；不夠時可以在同一系列內升級到 DMIT HKG.Pro STARTER（1000 GB 雙向流量），暫不支援降級（根據 DMIT 官方文件，查證日期 2026-11-17）。

### 為甚麼同時用 REALITY 和 Hysteria2？

為了多一條線路，兩條互為備用。REALITY 走 TCP 443，Hysteria2 走 UDP 443，客戶端的 `auto` 群組每 1 分鐘測試一次，平時用能連通而且較快的那一條，一條不通時自動換另一條；但兩條線路也可能同時失效。

### 能分享給家人或朋友用嗎？

只給自己和身邊少數信任的人用，不要公開、不要收費。DMIT AUP 允許加密的私人代理，禁止公共代理服務和轉售（根據 DMIT 官方文件，查證日期 2026-11-17）；AUP 沒有給出多少人算「公共」的標準，所以人越少越穩妥。

### 經推廣連結購買會更貴嗎？

DMIT 官方文件沒有說明，本文無法確認（根據 DMIT 官方文件，查證日期 2026-11-17）。不想用推廣連結，可以直接開啟 [DMIT 官網](https://www.dmit.io/) 下單。

### 可以用 WireGuard 或 OpenVPN 嗎？

不要用它們連回大陸。DMIT AUP 禁止建立去中國的 VPN 隧道，列舉了 L2TP、OpenVPN、PPTP、IPSec、Cisco AnyConnect，並寫明「包括但不限於」（根據 DMIT 官方文件，查證日期 2026-11-17）。

### 要不要裝 BBR 加速或其他加速腳本？

不要。DMIT AUP 禁止使用不可接受設定的網絡加速模組（例子有 FinalSpeed 和非官方 BBR）（根據 DMIT 官方文件，查證日期 2026-11-17）。本文的 Hysteria2 已經設定為不使用 Brutal 模式，不需要額外安裝任何東西。

### 瀏覽國內網站也會經過香港嗎？

會。本文的客戶端設定是全局代理，除了私有地址，所有流量都經過伺服器，瀏覽國內網站也會消耗流量、可能變慢。不需要翻牆時，可以在 App 裏中斷連線。

### 為甚麼有些網站或服務（例如串流媒體、AI）打不開或提示地區不對？

DMIT 不保證任何網站或服務都能連上，IP 是隨機分配的，地理資訊也不保證（根據 DMIT 官方文件，查證日期 2026-11-17），見 [IP FAQ](https://docs.dmit.io/guide/faq/ip)。這是 IP 本身的問題，與本文的設定無關。

### 可以用 Shadowrocket 或 v2rayN 嗎？

可以試，但它們不是 sing-box 官方 App，本文沒有核實它們能否正確匯入本文的分享連結、是否驗證 Hysteria2 的 `pinSHA256`；不驗證的話，用分享連結匯入的 Hysteria2 線路會接受任何證書。本文以 sing-box 官方 App 加設定檔為主。

### IP 被封了怎麼辦？

先按 P2 確認，再按 DMIT 規則申請更換：符合條件可免費更換，否則每次 US$5（根據 DMIT 官方文件，查證日期 2026-11-17）。換好後改 `guide.env` 中的 IP，重新產生並匯入客戶端設定。

### 流量用完會怎樣？

機器會被暫停，訂購頁標註 `Paused / Quota Exceeded`；可以等每月自動重置，或付費提前重置，費用等於月費（根據 DMIT 官方文件，查證日期 2026-11-17）。見 P3。

### 買了不滿意能退款嗎？

新訂單 3 日內而且流量不超過 30 GB 可以全額退款，但 TOS 寫明要扣除支付手續費；30 日內可以申請部分退款。續費單、對網絡不滿意、IP 地理位置等情況不退款（根據 DMIT 官方文件，查證日期 2026-11-17），見 [TOS](https://www.dmit.io/pages/tos) 第 19 條。

### 會自動續費嗎？

會。TOS 寫明服務會自動循環計費，直到你按取消流程取消（第 18.1 條）；要開啟或保留自動付款、自動續費，帳戶必須啟用並保持雙重驗證（2FA）（第 18.9 條）（根據 DMIT 官方文件，查證日期 2026-11-17）。

### 這份教學是 DMIT 官方的嗎？

不是。這是個人編寫的非官方教學，與 DMIT 沒有關聯；文中只有 DMIT 下單連結是作者的推廣連結。

<!-- section:risks -->
## 風險提示

### 法律風險

在中國大陸使用未經批准的翻牆方法，有被處罰的風險。以下是公開報道中的例子，金額由報道作者引用，本文沒有逐一核對其原始官方來源：

- 中國數字時代 2026-06-03 轉載的一篇微信文章列出：南昌一名個人因使用 VPN 被警告並罰款人民幣 15,000 元（文中未寫日期）；一名公司員工用辦公電腦觀看境外串流媒體，被警告並罰款人民幣 15,000 元（文中未寫日期）；福建寧德壽寧縣一名個人 2020 年翻牆，2024 年 7 月被發現後被傳喚處罰（文中未寫金額）（[中國數字時代](https://chinadigitaltimes.net/chinese/727752.html)）。
- 人權觀察（HRW）2026-03-17 的文章：中國公安部 2026-01-31 公佈網絡犯罪防治法草案，情節嚴重的最高可罰人民幣 5,000,000 元、拘留最長 15 日，並可限制出境 6 個月至 3 年；該文沒有提到 VPN 或個人翻牆（[HRW](https://www.hrw.org/zh-hant/news/2026/03/17/china-cybercrime-bill-entrenches-censorship-surveillance)）。

是否使用、怎樣使用，請你自己判斷並承擔後果。

### 沒有任何協議能永遠不被封鎖

以下全部是第三方研究或用戶自己的報告，不是 DMIT 或官方的確認：

- GFW Report 記錄：北京時間 2025-08-20 約 00:34 至 01:48，約 74 分鐘內，GFW 對 TCP 443 端口無條件注入偽造的 RST+ACK，影響中國與境外之間的 443 連線，其他端口沒有受影響，原因不明（[GFW Report](https://gfw.report/blog/gfw_unconditional_rst_20250820/en/)）。這種情況下 REALITY 也會受影響。
- 2026-01-05 起，Xray-core 討論區有多名用戶報告 REALITY / VLESS 被封，有人整個 IP 連 22 端口都被封，也有人報告同一連線在家居寬頻不通、在 4G 熱點能通；這些是用戶自述，沒有官方確認或系統性的量度，討論一直持續到 2026-07-02（[Xray-core 討論](https://github.com/XTLS/Xray-core/discussions/5494)）。本文沒有找到能證明這是一次單一大規模事件、有方法和日期的報告。
- 2026-10-26 前後，有論壇用戶把這一天稱為「墜機」日，多個代理節點同時失效，VLESS 和 Hysteria2 都有人報告受影響（[VilaVPN 博客](https://blog.vilavpn.com/blocking-roundup-2026-10/)，2026-10-27；這是商業 VPN 供應商的博客，作者說明只是整理用戶自述）。一位博客作者自己的 9 台 VPS 在 2026-10-26 改 UUID 後有 8 台下線（[elmagnifico.tech](https://elmagnifico.tech/2026/10/27/VPN-Compare/)）。
- 2026-08-20 有用戶報告福建福州大部分翻牆服務失效，作者說明沒有做正式技術檢查（[net4people #651](https://github.com/net4people/bbs/issues/651)）。
- 有研究討論 Hysteria v2（不使用 Brutal 時）可能存在跨層 RTT 指紋的問題，屬於學術討論，不是實測封鎖報告（[net4people #528](https://github.com/net4people/bbs/issues/528)）。另有 USENIX Security 2026 論文分析了 2025-09 外洩的 Geedge Networks 防火牆原始碼（[net4people #653](https://github.com/net4people/bbs/issues/653)）。

所以：本文的設定不能保證一直可用；被封時可能需要更換 IP（見 P2）或更換借用網站（見 P4）。本文沒有在中國大陸做過任何測試，實際效果因地區和電訊商而異。

### 只有一台 VPS 的風險

本文只用一台伺服器。這台伺服器的 IP 被封、流量用完被暫停、欠費、或者 DMIT 出現故障時，兩條線路會同時不可用。IP 被封時更換 IP 有條件和等候時間（見 P2），工單回覆最長可能要 72 小時（根據 DMIT 官方文件，查證日期 2026-11-17）。如果你的工作或生活離不開它，請預先想好後備的上網方式。本文不講多台伺服器的架構。

<!-- section:changelog -->
## 更新記錄

- 2026-11-15：初稿（草稿）。事實最後查證日期 2026-11-15；sing-box 版本按 1.14.4 編寫。
- 2026-11-16：在兩台 Debian 13（arm64）虛擬機之間測試了伺服器設定和客戶端連接（範圍見「[伺服器設定](#伺服器設定)」），並按測試結果修改：S8 會再檢查一次借用網站，並先產生全部值再寫入，任何一個產生指令失敗或產生的值為空都會停止、不寫入；S10 會等服務開始監聽；機器原本用 iptables 管理防火牆、並且 `iptables -S INPUT` 讀得出原有的鏈時，S5 改用 iptables 指令加規則；防火牆服務在網絡啟動前執行，`nftables.service` 重新啟動、重新載入，或者停止後再啟動時，都會重新加入規則；P1 會檢查防火牆規則，並說明規則缺失和時間不同步時怎樣修正；補充 Windows 上的 V1、P2 指令；D3 的回退指令加上 `--allow-downgrades`。
- 2026-11-16：讓一個只拿到本文的 AI agent 在全新的測試虛擬機上按照本文做完伺服器設定，按它要猜測的地方修改：寫明 AI agent 怎樣經 SSH 非互動執行伺服器上的程式碼區塊（在 macOS 或 Linux 終端機中執行，檔案用 LF 換行，第一次連接先由讀者確認伺服器指紋）；`<REALITY_TARGET>` 加了例子，並說明這個域名還用在哪裏；S4 最後會印出要不要重新開機，D3 也提醒裝了新內核要重新開機；S10 的預期結果不再暗示兩行的先後次序；C1 說明檔案保存在哪個目錄。
- 2026-11-17：再次核對 DMIT 官網頁面、官方文件和公告，以及 sing-box 最新穩定版：價格、方案、規則和 sing-box 版本（1.14.4）都沒有變化，這些事實的查證日期改為 2026-11-17。事實表的線路一行改為寫明：官方標示的約 15 ms 只是從香港到鄰近的一個大陸城市的參考測量值。更正 Tier 1 的說明：來源是 DMIT 的 IP FAQ，它指出 T1 系列新分配的 IP 不保證首次連接時在敏感地區可用，而 Premium 和 Eyeball 則保證首次連接全球可用（經工單處理）；原來「IP 亦不保證在所有國家和地區都能使用」的寫法，在所引用的頁面上找不到。IP 管理變動一行改為：加收由 2026-12-01 當日或之後發出的續費單起（原本寫「之後」）。S2 改為說明重裝可能會更換伺服器的主機密鑰，而不是一定會更換。S8 改為求助時只貼這一步印出的報錯（`ERROR` 開頭的那一行，或者 openssl、sing-box 的報錯），不要貼 `guide.env` 的內容。
