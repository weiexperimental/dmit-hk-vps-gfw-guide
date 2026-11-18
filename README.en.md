[简体中文](README.md) | [繁體中文](README.zh-Hant.md) | English

# DMIT Hong Kong VPS Guide: Get Past the GFW in China with sing-box (VLESS + REALITY and Hysteria2)

<!-- section:intro -->
## Summary

This guide uses one DMIT Hong Kong VPS and the official sing-box to build two encrypted proxies for your own use only: one is VLESS + XTLS Vision + REALITY (TCP 443), the other is Hysteria2 (UDP 443). The client tests both connections every minute and automatically uses the one that works and is faster. Most individual users can start with DMIT HKG.Pro TINY: the order page lists it at US$39.90/month with 500 GB of bidirectional traffic, and its network routes are CN2 GIA and CMI (per DMIT official docs, verified 2026-11-17). The server runs Debian, sing-box is installed from the official apt repository, and every command is written out in this guide, so you never need to run a third-party script; on the client side you use the official sing-box app on iOS, Android, Windows and macOS. No protocol can guarantee that it will stay unblocked for long, and using tools to get past the GFW in mainland China also carries legal risk, so read "[Risks](#risks)" before you start.

The facts in this guide were last verified on 2026-11-17 (each fact shows its own verification date). Only the DMIT order link in this guide is an affiliate link ([DMIT order link](https://www.dmit.io/aff.php?aff=30385)); the other links to DMIT's website and docs are plain links; see "[Affiliate link and disclaimer](#affiliate-link-and-disclaimer)" for details.

Contents:

1. [Affiliate link and disclaimer](#affiliate-link-and-disclaimer)
2. [DMIT Hong Kong plan facts](#dmit-hong-kong-plan-facts)
3. [How to choose a plan](#how-to-choose-a-plan)
4. [What DMIT allows and forbids](#what-dmit-allows-and-forbids)
5. [What to choose when you order](#what-to-choose-when-you-order)
6. [The three values you replace](#the-three-values-you-replace)
7. [Hand it to your AI agent](#hand-it-to-your-ai-agent)
8. [Server setup](#server-setup)
9. [Client setup](#client-setup)
10. [Check that it works](#check-that-it-works)
11. [Routine maintenance](#routine-maintenance)
12. [Troubleshooting](#troubleshooting)
13. [FAQ](#faq)
14. [Risks](#risks)
15. [Changelog](#changelog)

<!-- section:disclosure -->
## Affiliate link and disclaimer

- The DMIT order link in this guide, https://www.dmit.io/aff.php?aff=30385, is the author's affiliate link. If you buy through it, the author earns a commission from DMIT. According to the [DMIT affiliate FAQ](https://docs.dmit.io/guide/faq/affiliate), the commission is usually 10% of the amount paid; it may drop to 5% for some high value-for-money products and rise to 20% during some promotions (per DMIT official docs, verified 2026-11-17).
- DMIT's official docs do not say whether buying through the affiliate link costs the same as buying directly, and this guide cannot confirm it (per DMIT official docs, verified 2026-11-17). If you do not want to use the affiliate link, open the [DMIT website](https://www.dmit.io/) directly and order there; the steps in this guide are exactly the same.
- This is an unofficial guide written by an individual. It has no connection with DMIT, does not speak for DMIT, and is not DMIT's official documentation. DMIT's website and official docs are the authority on DMIT's rules.
- This guide contains no hidden content written for AI. The "[Hand it to your AI agent](#hand-it-to-your-ai-agent)" section is a public prompt that you can read and edit yourself.

The text of this guide is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). To reuse it, credit it with a link to https://github.com/weiexperimental/dmit-hk-vps-gfw-guide. The full license text is in the `LICENSE` file of this repository.

<!-- section:facts -->
## DMIT Hong Kong plan facts

The table below only includes what was found on DMIT's website, in its official docs or in its official announcements. Prices and specs can change at any time ([TOS](https://www.dmit.io/pages/tos) section 15: prices do not rise during a term you have already signed up for, but DMIT can change list prices and plan resources at any time without notice), so check the order page before you order.

| Item | Details | Source | Verified |
|---|---|---|---|
| DMIT HKG.Pro TINY (order page name HKG.AS3.Pro.TINY) | US$39.90/month, no setup fee; 1 vCore, 1 GB RAM, 20 GB SSD, 1 IPv4, 1 IPv6 /64; 500 GB bidirectional traffic; 1 Gbps port | [Order page (Premium, AS3)](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3) | 2026-11-17 |
| DMIT HKG.Pro STARTER (order page name HKG.AS3.Pro.STARTER) | US$79.90/month, no setup fee; 1 vCore, 2 GB RAM, 40 GB SSD, 1 IPv4, 1 IPv6 /64; 1000 GB bidirectional traffic; 1 Gbps port | [Order page (Premium, AS3)](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3) | 2026-11-17 |
| DMIT HKG.EB (Eyeball, officially labeled Beta) | WEE: US$24.90/month, 1 vCore, 1 GB RAM, 20 GB SSD, 500 GB bidirectional traffic, 500 Mbps; TINY: US$39.90/month, 1 vCore, 1 GB RAM, 20 GB SSD, 800 GB bidirectional traffic, 1 Gbps | [Order page (Eyeball)](https://www.dmit.io/cart.php?region=hong-kong&network=eyeball) | 2026-11-17 |
| Network routes (Premium, i.e. Pro) | Data center Equinix HK2 (Kwai Chung); via CN2 GIA (AS23764) and CMI (AS58453); the average latency to mainland China of about 15 ms that DMIT shows is its reference measurement from Hong Kong to one nearby mainland city, with packet loss below 0.1%; DMIT notes these are reference values only, and real results depend on the network, routing and time of day | [DMIT Hong Kong location page](https://www.dmit.io/pages/location/hong-kong) | 2026-11-17 |
| Direct peering | Official cloud instance page: direct peering with China Telecom (AS4809), China Unicom (AS9929) and China Mobile International (AS58807); Premium also has CN2 GIA | [DMIT cloud instance page](https://www.dmit.io/pages/cloud-instance) | 2026-11-17 |
| Traffic counting | The Pro and Eyeball order pages say Bidirectional: both upload and download count | [Order page (Premium, AS3)](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3), [Instance FAQ](https://docs.dmit.io/guide/faq/instance) | 2026-11-17 |
| Going over the traffic limit | The order page says "Paused / Quota Exceeded", which means the instance is paused; traffic resets automatically every month, and the control panel shows the next reset time; most standard plans can pay for an early reset, which costs the same as that instance's monthly fee; extra traffic packs are only for the sPro series | [Order page (Premium, AS3)](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=as3), [Instance FAQ](https://docs.dmit.io/guide/faq/instance) | 2026-11-17 |
| Refunds | Full refund: a new order, no more than 3 days after purchase, and no more than 30 GB of traffic used (the TOS says payment fees are deducted); partial refund: a new order, no more than 30 days after purchase; no refund for renewal orders, payments made from the account balance (refunded to the balance only), dissatisfaction with the network, IP geolocation and similar cases | [TOS](https://www.dmit.io/pages/tos) section 19, [Refund FAQ](https://docs.dmit.io/guide/faq/refund) | 2026-11-17 |
| IP change | A free change for Pro and EB requires all of these: at least 7 days since purchase, at least 15 days since the last change, ICMP and all TCP ports blocked for the whole IP, and at least 7 days of service left; if the IP is already blocked when the instance is delivered, you can open a ticket for a free change. Paid change: you can request one at any time, US$5 each time, with at least 7 days between requests; a completed change cannot be undone or refunded | [IP FAQ](https://docs.dmit.io/guide/faq/ip) | 2026-11-17 |
| Payment methods | The Billing FAQ mentions PayPal and credit cards (full personal and address details are required); the complete list of payment methods could not be verified | [Billing FAQ](https://docs.dmit.io/guide/faq/billing) | 2026-11-17 |
| Renewal | Monthly instances get a renewal invoice 7 days before they expire; after expiry the instance is kept for 3 days and can be restored if you pay within those 3 days; after that it is deleted, and the data cannot be recovered | [Billing FAQ](https://docs.dmit.io/guide/faq/billing) | 2026-11-17 |
| Ordering requirement | DMIT announcement of 2026-11-14: you must provide a valid SSH public key when you order and when you use Reinstall OS; private keys are now generated locally in the browser; the client area no longer stores the root password for the Console | [DMIT Telegram announcement](https://t.me/DMIT_INC/1292) | 2026-11-17 |
| Change in IP management | DMIT announcement: from 2026-12-01, DMIT switches to IP addresses that it buys and manages itself; users who keep Cogent IPs that start with 154 pay an extra US$1/month per IPv4, starting with the renewal invoices issued on or after 2026-12-01; Cogent IPs in Hong Kong will be moved to standard IPs in batches, on a schedule to be announced; users without an IP that starts with 154 are not affected | [DMIT Telegram announcement](https://t.me/DMIT_INC/1282) | 2026-11-17 |
| Ticket replies | The TOS only guarantees a reply within 72 hours; the Ticket FAQ says usually within 24 hours, with possible delays on holidays and weekends | [TOS](https://www.dmit.io/pages/tos) section 3.7, [Ticket FAQ](https://docs.dmit.io/guide/faq/ticket) | 2026-11-17 |
| Rule versions | AUP last updated 2025-11-12; TOS last updated 2026-01-22 | [AUP](https://www.dmit.io/pages/aup), [TOS](https://www.dmit.io/pages/tos) | 2026-11-17 |

<!-- section:choose -->
## How to choose a plan

**The three series compared**, with numbers from the fact table above (per DMIT official docs, verified 2026-11-17):

| | DMIT HKG.Pro TINY | DMIT HKG.Pro STARTER | DMIT HKG.EB |
|---|---|---|---|
| Monthly fee | US$39.90 | US$79.90 | WEE US$24.90; TINY US$39.90 |
| Bidirectional traffic | 500 GB | 1000 GB | WEE 500 GB; TINY 800 GB |
| Port speed | 1 Gbps | 1 Gbps | WEE 500 Mbps; TINY 1 Gbps |
| Memory | 1 GB | 2 GB | 1 GB |
| Network routes | Premium: CN2 GIA + CMI | Premium: CN2 GIA + CMI | Eyeball: Tier 1 plus "reasonable effort" China routing (officially labeled Beta) |

How to choose:

- **Start with DMIT HKG.Pro TINY.** Among the Premium-network plans listed in this guide, it has the lowest monthly fee (DMIT HKG.Pro STARTER is US$79.90, and the cheapest plan of the AN5 generation, HKG.AN5.Pro.MINI, is US$149.90). DMIT lets you upgrade to a higher plan within the same series and charges the price difference between the new and old plans, but downgrades are not supported for now (per DMIT official docs, verified 2026-11-17); see the [Instance FAQ](https://docs.dmit.io/guide/faq/instance). So buying TINY first and upgrading to DMIT HKG.Pro STARTER if you run short of traffic is safer than buying a big plan and hoping to downgrade later.
- **If you use a lot of traffic or need 2 GB of memory, choose DMIT HKG.Pro STARTER.** It has twice the traffic of TINY.
- **Consider DMIT HKG.EB only if budget comes first and you can live with instability.** DMIT says that Hong Kong Eyeball is still Beta and its routing is still being tuned, so it is not yet recommended for production use that needs high stability; its China routing is "reasonable effort" (per DMIT official docs, verified 2026-11-17); see the [Order page (Eyeball)](https://www.dmit.io/cart.php?region=hong-kong&network=eyeball) and the [Hong Kong location page](https://www.dmit.io/pages/location/hong-kong).

**How to estimate bidirectional traffic:** The order page lists bidirectional traffic, so both upload and download count (per DMIT official docs, verified 2026-11-17). When a proxy server relays data, the same data is counted twice: once when the website sends it to the server (in), and again when the server forwards it to you (out). So for a rough estimate, take the total you actually download and upload through the proxy each month, multiply it by 2, and compare the result with the plan's traffic. In other words, 500 GB of bidirectional traffic gives you about half of that in real use. You can first estimate your usage from the monthly data statistics on your phone and computer, and check it once you are up and running with the methods in "[Routine maintenance](#routine-maintenance)".

**Why not Lite or Tier 1 (T1):**

- **Tier 1 (T1)**: DMIT describes it as "without China Optimization", meaning it is not optimized for mainland China; DMIT's IP FAQ also says that a newly assigned T1 IP is not guaranteed to be reachable from sensitive regions on the first connection, while Premium and Eyeball are guaranteed to be reachable worldwide on the first connection (handled through a ticket). It is cheap (for example T1.TINY at US$6.90/month with 2000 GB), and its traffic is counted as "IN/OUT Max" (per DMIT official docs, verified 2026-11-17); see the [Order page (Tier 1)](https://www.dmit.io/cart.php?region=hong-kong&network=tier-1), the [cloud instance page](https://www.dmit.io/pages/cloud-instance) and the [IP FAQ](https://docs.dmit.io/guide/faq/ip). For use from mainland China, the route into the mainland is what matters most, so it is not recommended.
- **Lite**: No product named Lite appears under any of the three filters on the Hong Kong order page (Premium, Eyeball, Tier 1) or on the pricing page (per DMIT official docs, verified 2026-11-17), so this guide does not cover it.
- **The AN5 generation in the same Premium group**: The cheapest plan is HKG.AN5.Pro.MINI at US$149.90/month, and there is no TINY or STARTER (per DMIT official docs, verified 2026-11-17); see the [Order page (Premium, AN5)](https://www.dmit.io/cart.php?region=hong-kong&network=premium&generation=an5). To see TINY and STARTER, choose the AS3 generation on the order page.

### Public test summary

Only third-party tests that state their method and date are included, and everything below is a **third-party test, not a test by the author of this guide**. As of 2026-11-15, no third-party test was found that specifically tests the current DMIT HKG.Pro TINY or STARTER (AS3 generation) and also states its method and date.

| What was tested | Summary of results | Method | Date | Source | Notes |
|---|---|---|---|---|---|
| HKG.AN5.EB.TINY (Eyeball) | Geekbench 5 single-core 2108, multi-core 2113; disk fio about 2.1 GB/s; the author rated latency to the three carriers "good", but most China speed-test points could not be used; the China Telecom return route was rated excellent, and China Unicom and China Mobile "usable" (可用); the author recommends disabling IPv6; the system information shows Debian 13 | Geekbench 5, fio, speed tests to the three carriers (screenshots), TCP traceroute to 9 targets, IP unlock checks | Published 2026-10-07; screenshots dated 2026-09-19 | [hostalk.net](https://hostalk.net/posts/dmit_hkeb.html) | The purchase links in the article carry an affiliate parameter, which the article does not clearly disclose; it tests the AN5-generation Eyeball, not the AS3 Pro TINY |
| HKG.Pro.MongKok (old model: 1 vCPU, 1 GB RAM, 20 GB SSD, 200 GB, 150 Mbps) | Traceroutes from three mainland cities to the three carriers, with last-hop latency of about 11 ms to 49 ms; the author rated the China Telecom CN2 GIA return route excellent; speed tests "basically maxed out" (基本跑满) | 30-hop traceroute (from three mainland cities to the three carriers), speed tests and Geekbench 5 (screenshots) | Published 2024-08-07, updated 2025-11-24; screenshots dated 2024-08-05 | [hostalk.net](https://hostalk.net/posts/dmit_hkpro.html) | The purchase links in the article carry an affiliate parameter, which the article does not clearly disclose; an old model, for reference only |

<!-- section:rules -->
## What DMIT allows and forbids

Below is a summary of the parts of DMIT's Acceptable Use Policy ([AUP](https://www.dmit.io/pages/aup), last updated 2025-11-12) and Terms of Service ([TOS](https://www.dmit.io/pages/tos), last updated 2026-01-22) that matter for this guide (per DMIT official docs, verified 2026-11-17). A summary cannot replace the original, so read the original yourself.

**Allowed:**

- Encrypted private proxies. The AUP text explicitly allows "Encrypted Private Proxy". The REALITY and Hysteria2 setup in this guide is an encrypted proxy for your own use only.

**Forbidden (the parts that matter for this guide):**

- **Public Proxy Services.** The AUP says such services can get DMIT's IP ranges or ASN blocked. The test the AUP gives is: spam complaints about the IP from other networks, together with equal upstream and downstream traffic. So do not share your node with many people, and do not publish share links or config files.
- **VPN tunnels to China.** The AUP forbids using the service to build VPN tunnels to China, lists L2TP, OpenVPN, PPTP, IPSec and Cisco AnyConnect, and says "including but not limited to". WireGuard is also a VPN tunnel, so do not use WireGuard, OpenVPN or similar VPNs to connect back to the mainland either.
- **Resale.** The AUP forbids redistributing and selling DMIT's products, and also forbids secondary virtualization for commercial purposes. Do not sell nodes, and do not use it to run a commercial proxy service (an "airport", 机场).
- **Using large amounts of bandwidth for long periods.**
- **Unofficial acceleration.** The AUP forbids network acceleration modules with unacceptable settings, and gives FinalSpeed and unofficial BBR as examples. Do not install any third-party acceleration script or module. The Hysteria2 setup in this guide also does not use Brutal mode (see Server setup).
- **Spam.** The AUP has a separate NO SPAM section.
- **Other:** Running TOR exit nodes and unauthorized port scanning.
- **Resource limits:** All products guarantee 50% CPU usage by default (unless stated otherwise), and abnormal or excessive CPU usage may be limited.

**If you break the rules**, this is what the [AUP](https://www.dmit.io/pages/aup) and the [TOS](https://www.dmit.io/pages/tos) say (per DMIT official docs, verified 2026-11-17):

- Depending on how serious the violation is, DMIT can suspend the service or null-route the IP for 7 to 30 days, or terminate the service immediately; DMIT can also, at its own discretion and with or without notice, end your use of the service for violating the AUP.
- Accounts that DMIT considers to be related to spam are terminated immediately.
- The TOS says that problems caused by abuse are not refunded (section 19.3), and that account sharing, public proxies, transfers and opening paid resources to the public are not cases eligible for compensation (sections 3.14 and 20.4). If you abuse the IP change feature, you may be banned from using it; see the [IP FAQ](https://docs.dmit.io/guide/faq/ip).

<!-- section:order -->
## What to choose when you order

This section only says what to choose when you order; it does not walk through each screen with screenshots.

- **Do this first:** Since 2026-11-14, DMIT requires an SSH public key when you order (per DMIT official announcement, verified 2026-11-17); see the [DMIT Telegram announcement](https://t.me/DMIT_INC/1292). So before you order, complete "[S1 Generate an SSH key on your own computer](#s1-generate-an-ssh-key-on-your-own-computer)" on your own computer, and when you order, paste the whole public-key line it prints (starting with `ssh-ed25519`) into the field where the order asks for a public key.
- **Plan:** DMIT HKG.Pro TINY. On the order page, choose Hong Kong, Premium and AS3 in turn, and it appears as HKG.AS3.Pro.TINY (per DMIT official docs, verified 2026-11-17). If you need a lot of traffic, choose DMIT HKG.Pro STARTER; if budget comes first and you can accept Beta, choose DMIT HKG.EB.
- **OS:** Choose the latest stable Debian release that the order page offers. The commands in this guide are written for Debian 13; at the time of writing, Debian 13 (trixie) is Debian's stable release ([debian.org](https://www.debian.org/releases/), verified 2026-11-15). This guide could not verify which Debian versions the DMIT order page offers.
- **Billing cycle:** The Hong Kong Premium and Eyeball order pages only show Monthly (per DMIT official docs, verified 2026-11-17); this guide could not verify whether a yearly option exists.
- **Payment method:** Credit card or PayPal. The Billing FAQ says that PayPal and credit card payments need full personal and address details, and that an unverified PayPal account, a credit card without 3D Secure support, or a rejection by Stripe's risk checks can all make the payment fail (per DMIT official docs, verified 2026-11-17); see the [Billing FAQ](https://docs.dmit.io/guide/faq/billing).
- **Order link (affiliate link):** https://www.dmit.io/aff.php?aff=30385. If you do not want to use the affiliate link: [DMIT website](https://www.dmit.io/).

<!-- section:placeholders -->
## The three values you replace

In this guide's code, you only replace the three values below. All other values (UUID, keys, passwords, certificate fingerprints and so on) are generated automatically on the server and saved in `/etc/sing-box/guide.env` on the server.

| Placeholder | What it is | How to fill it in | Example |
|---|---|---|---|
| `<SERVER_IP>` | The server's public IPv4 address | Find this machine's IPv4 address in the DMIT client area or in the activation notice (unverified; check the actual interface) | `203.0.113.10` (an address reserved for documentation; do not copy it) |
| `<SSH_USER>` | The user name for logging in to the server | Check whether the DMIT client area or the activation notice states it (unverified; check the actual interface); if it does not, try `root` first, then `debian` | `root` |
| `<REALITY_TARGET>` | The domain of the website that REALITY borrows | See below | `www.example.com` (only an example of the format; do not use it as is) |

**Replacement rules:**

- If a placeholder in the code is inside single quotes, such as `'<SERVER_IP>'`, keep the single quotes and replace the angle brackets together with the text inside them, for example `'203.0.113.10'`.
- If you forget to replace one, the config is not quietly broken: the step that generates the keys (S8) stops with an error and writes nothing if a value still contains `<` or `>`; the later step that writes the config (S9) also stops, because `guide.env` is missing.

**How to choose `<REALITY_TARGET>`:**

During the handshake, REALITY borrows the TLS characteristics of another website, and that website is `<REALITY_TARGET>`. When you choose it:

1. Choose a website outside mainland China that opens normally from the mainland (that is, not blocked), and enter its domain, for example the `www` domain of a large website.
2. It must support TLS 1.3 and HTTP/2.
3. Enter only the domain: no `https://`, no path and no port.

**Examples:** `www.microsoft.com`, `www.apple.com`. These are only examples: they are not recommendations or defaults, and they do not mean these sites are better suited than others. On 2026-11-16, on this guide's test virtual machine, S7 printed `OK` for both domains; this guide has not verified whether they open normally from the mainland (point 1 above). Whichever site you choose, check it in S7 first.

This domain is also used as the name in the Hysteria2 self-signed certificate and as `server_name` in the client profile. To change it after setup, follow "[P4 Change the target site](#p4-change-the-target-site)".

**How to check it on the server:** After you log in to the server, run the check command in "[S7 Check the target site](#s7-check-the-target-site)". It runs `openssl s_client` on the server to connect to port 443 of the site, allows only TLS 1.3, and requests HTTP/2 (ALPN `h2`). The site passes only if the command prints `OK`; if it prints `NOT OK`, try another site. You can try several candidate sites in S7 and pick one that passes before you continue.

<!-- section:agent -->
## Hand it to your AI agent

If an AI agent such as Claude Code or Codex helps you with the setup, you can give it the text below together with this guide. This text is ordinary text, published openly here; you can read it first and change it as you need. This guide has no hidden content written for AI.

```text
Please read this README (I will give you the file or the link) and follow its three sections "Server setup", "Client setup" and "Check that it works" to set up a DMIT Hong Kong VPS for me.
Server IP: <SERVER_IP>
SSH user: <SSH_USER>
REALITY target site: <REALITY_TARGET> (if I have not filled this in, pick a few candidates with the method in the README section "The three values you replace" and check them on the server with S7; after S7 passes and before you run S8, ask me which one to use)
Requirements:
1. Before anything that involves payment or changes account settings (including anything in the DMIT client area), ask me first, and go ahead only after I agree.
2. Run only the commands written in the README. Do not download or run any third-party install script.
3. After each step, compare the result with that step's "Expected". If it does not match, follow "If it fails"; if you cannot fix it, stop and show me the output.
4. Run the server code blocks as root. When you run them non-interactively over SSH, follow "How to run these commands" at the start of the README's "Server setup": do not run S3; save each later code block that runs on the server as a file exactly as written (replace only the placeholders; UTF-8 without BOM, LF line endings), and in a macOS or Linux terminal run them one at a time, in order, with ssh <SSH_USER>@<SERVER_IP> sudo bash -s < filename (drop sudo if you log in as root; on Windows use WSL or Git Bash, which this guide has not tested). Before the first SSH connection to this server, ask me, so that I can do S2 myself once first.
5. Do not post the contents of /etc/sing-box/guide.env, /root/dmit-guide/client-profile.json or /root/dmit-guide/share-links.txt anywhere public.
6. Do not do anything the DMIT AUP forbids, such as public proxies, resale, VPN tunnels to China or unofficial acceleration.
```

<!-- section:server -->
## Server setup

**What you do:** Install sing-box on the server and open two connections: VLESS + XTLS Vision + REALITY listening on TCP 443, and Hysteria2 listening on UDP 443. At the end, generate a client profile and two share links.

**Why these two protocols:** One blogger, based on personal experience, ranks VLESS + REALITY as the hardest to detect and the most stable, and Hysteria2 as the fastest, but the author says no formal testing was done ([lilting.ch](https://lilting.ch/en/articles/china-vpn-protocol-comparison), 2026-01-08, updated 2026-05-21). Around 2026-10-26, users reported that many nodes failed at the same time; one blogger saw 8 of his own 9 VPSes go offline, and he observed that Hysteria2 held up well in this round, but only by watching his own servers, without tests across several regions or carriers ([elmagnifico.tech](https://elmagnifico.tech/2026/10/27/VPN-Compare/), 2026-10-27). One connection uses TCP and the other uses UDP, so when one stops working, the client switches to the other automatically; but both can also fail at the same time; see "[Risks](#risks)".

**Versions:** At the time of writing, the latest stable sing-box release is 1.14.4 ([GitHub release](https://github.com/SagerNet/sing-box/releases/tag/v1.14.4), verified 2026-11-17). The way this guide pins the Hysteria2 certificate needs sing-box 1.13.0 or later on the client ([sing-box TLS docs](https://sing-box.sagernet.org/configuration/shared/tls/)). The commands in this guide are written for Debian 13 and do not depend on the CPU architecture. On 2026-11-16 they were tested between two Debian 13 (arm64) virtual machines: all the server setup commands (once as root and once as a normal user with sudo, plus pasting a block twice, forgetting to replace a placeholder, a machine that already had firewall rules, and a reboot), and, for the client profile, that each connection works on its own and that the client switches to the other one when one is blocked. In these tests the client was the sing-box command line on Linux, with `tun` replaced by a local proxy port. On the same day, an AI agent that was given only this guide completed the server setup on a fresh Debian 13 (arm64) test virtual machine, following the version of this guide from before the revision, and this guide was revised based on its feedback; the revised commands (including the kernel check in S4 and the non-interactive method) were retested with scripts, and the agent was not asked to follow the guide a second time. These steps have not been run on a DMIT machine, on an amd64 (x86-64) machine, or on the Debian system image that DMIT provides; the phone and computer apps were not tested, and nothing was tested in mainland China.

**How to run these commands:**

- Code blocks marked "run on your own computer" run on your own computer; all other code blocks run on the server.
- From S3 on, the server code blocks must run as root. When you work by hand, switch to root with S3, then paste the blocks in order in the same window.
- When a script or an AI agent runs them non-interactively over SSH:
  - Do not run S3 (it only helps in an interactive terminal window). Run each later code block that runs on the server, one at a time, in order.
  - Save the content of the code block as a file exactly as written, replacing only the placeholders and changing nothing else, for example as `s05.sh`. The file must use UTF-8 encoding (without BOM) and LF line endings. With CRLF line endings (common on Windows), bash on the server treats the carriage return at the end of each line as part of the command, and the code block does not run correctly.
  - In a macOS or Linux terminal on your own computer, run `ssh <SSH_USER>@<SERVER_IP> sudo bash -s < s05.sh` (drop `sudo` if you log in as root). This way the server runs exactly the same commands as when you paste them by hand. On Windows you can run this command in WSL or Git Bash; this guide has not tested that.
  - The first time you connect to this server, SSH needs a person to confirm the server's fingerprint (see S2). An agent without a terminal window cannot confirm it, so SSH prints `Host key verification failed.` and exits. So do S2 yourself once first, or have the agent ask you before its first connection.
  - An exit code other than 0 means the step did not complete; even when the exit code is 0, compare the output with "Expected".
- You can paste each code block on its own, and you can paste it again: keys that were already generated are not regenerated, and firewall rules that were already added are not added twice.
- Many code blocks are wrapped in a pair of parentheses `( … )`. They make sure that if a code block fails partway, only that block stops, and your terminal window does not close. If a code block prints `ERROR:` and stops, the step did not complete; deal with it as described in "If it fails", then paste the same code block again.

**Files created on the server:**

| File | Contents |
|---|---|
| `/etc/sing-box/guide.env` | All automatically generated values (permissions 600; it is as good as your keys, so never make it public) |
| `/etc/sing-box/hy2.crt`, `/etc/sing-box/hy2.key` | The self-signed certificate and private key for Hysteria2 |
| `/etc/sing-box/config.json` | Server config |
| `/root/dmit-guide/client-profile.json` | Client profile (for importing into the app; directory permissions 700, file permissions 600) |
| `/root/dmit-guide/share-links.txt` | `vless://` and `hysteria2://` share links (permissions 600) |
| `/usr/local/sbin/sing-box-guide-firewall` | Firewall script |

<!-- step:server-01-ssh-key -->
### S1 Generate an SSH key on your own computer

**Goal**: Generate an SSH key pair. You give the public key to DMIT when you order, because orders have required a public key since 2026-11-14 (per DMIT official announcement, verified 2026-11-17); after that you log in to the server with the private key instead of a password. **Do this step before you order.**

**Command** (run on your own computer; on macOS and Linux, use the terminal):

```bash local
mkdir -p ~/.ssh && chmod 700 ~/.ssh
[ -f ~/.ssh/id_ed25519 ] || ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -C dmit-hk
cat ~/.ssh/id_ed25519.pub
```

`ssh-keygen` asks whether you want to set a passphrase: setting one is more secure, but you then enter it every time you log in; press Enter to leave it empty. If your computer already has `~/.ssh/id_ed25519`, these commands do not overwrite it and simply reuse it.

Windows 10 and 11 include an OpenSSH client, but the commands above are written for bash. On Windows, open PowerShell, run `ssh-keygen -t ed25519 -C dmit-hk` (press Enter to accept the default path), then run `Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub` to see the public key.

**Expected**: The last line printed starts with `ssh-ed25519 ` and ends with ` dmit-hk` (on Windows it also ends with ` dmit-hk`). That whole line is the public key; paste it in when you order. The file in that directory without the `.pub` suffix, `id_ed25519`, is the private key; never send it to anyone.

**If it fails**:

- You see `ssh-keygen: command not found`: macOS and mainstream Linux distributions include OpenSSH; on Windows, add "OpenSSH Client" under "Settings → Optional features".
- You do not remember the passphrase of an existing key: replace `id_ed25519` with another file name (for example `-f ~/.ssh/id_ed25519_dmit`) and generate a new key, then specify it when you log in with `ssh -i ~/.ssh/id_ed25519_dmit …`.

<!-- step:server-02-login -->
### S2 Log in to the server with SSH

**Goal**: After DMIT delivers the server, log in to it from your own computer.

**Command** (run on your own computer; replace `<SSH_USER>` and `<SERVER_IP>` first):

```bash local
ssh <SSH_USER>@<SERVER_IP>
```

**Expected**: The first connection asks `Are you sure you want to continue connecting (yes/no/[fingerprint])?`; type `yes` and press Enter. Then the server's command prompt appears: for root it ends with `#`, and for a normal user (such as `debian`) it ends with `$`.

**If it fails**:

- `Permission denied (publickey)`: the user name is wrong (try `root` or `debian`), or the public key you gave when you ordered is not the one on this computer. Reinstalling the OS also requires a public key (per DMIT official announcement, verified 2026-11-17); you can reinstall the OS from the client area with the correct public key (unverified; check the actual interface).
- `Connection timed out`, or no response at all: check that the IP is right and that the machine has been delivered and is running; if you could connect before and suddenly cannot, see "[P2 Check whether the IP is blocked and change it](#p2-check-whether-the-ip-is-blocked-and-change-it)".
- The private key is not at the default path: add `-i <private-key-path>`, for example `ssh -i ~/.ssh/id_ed25519_dmit <SSH_USER>@<SERVER_IP>`.
- If SSH prints `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` after a reinstall: a reinstall can give the server a new host key (and so a new fingerprint), which no longer matches the key your computer saved. If you are sure this is the machine you just reinstalled, run `ssh-keygen -R <SERVER_IP>` on your own computer, then do S2 again.

<!-- step:server-03-root -->
### S3 Switch to root

**Goal**: Run all later server commands as root. This way the later code blocks are the same whether you log in as `root` or as a normal user such as `debian`, and you do not need to put `sudo` in front of every command.

**Command** (run on the server):

```bash server
[ "$(id -u)" -eq 0 ] || sudo -i
```

If you are already root, this command does nothing; a normal user is switched to root (if you are asked for a password, enter this normal user's password). Paste all later code blocks in this window. After you disconnect and reconnect, do S2 and S3 again.

**Expected**: The command prompt ends with `#`; typing `whoami` shows `root`.

**If it fails**:

- `sudo: command not found`: you logged in as a normal user without sudo; log in as `root` instead.
- `is not in the sudoers file`: this normal user has no admin rights; log in as `root` instead, or open a ticket to ask DMIT which user is the default.
- A script or AI agent runs the blocks non-interactively: this step cannot change the user for the later commands, so do not run it. Follow "How to run these commands" at the start of "Server setup" and run each later code block one at a time with `sudo bash -s`. If you see `sudo: a terminal is required to read the password`, this user needs a password for sudo, which cannot be typed in non-interactive mode; log in as root instead.

<!-- step:server-04-update -->
### S4 Update the system and install basic tools

**Goal**: Bring the system fully up to date and install the tools used later: `curl`, `ca-certificates`, `openssl`, `nftables` (the firewall), `qrencode` (shows QR codes in the terminal) and `sudo` (needed when C1 downloads the client profile to your computer, even if you log in as root). All of these come from Debian's own repositories.

**Command** (run on the server as root):

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

`--force-confold` means: if a package's config file has been changed, keep the current version and do not ask. The last few lines compare the running kernel (`uname -r`) with the newest kernel in `/boot` to decide whether a reboot is needed.

**Expected**: No error line starting with `E:` appears, and the last line is one of these three:

- `No reboot needed: …`: the running kernel is the newest one; go straight to S5.
- `Reboot recommended: …`: this update installed a new kernel, which is used only after a reboot. Run `reboot` to restart (the SSH connection drops), wait a minute or two, then log in to the server again (when working by hand, do S2 and S3 again; when running non-interactively, wait until SSH can connect again), then do S5. You can also skip the reboot for now and reboot after the later steps: the firewall and sing-box that this guide sets up both start automatically at boot.
- `Cannot tell whether a reboot is needed: …`: no kernel file was found in `/boot`, so this check cannot tell. You can go straight to S5.

This check does not use `/var/run/reboot-required`: on this guide's test virtual machine, that file was created by the `unattended-upgrades` package after a kernel update, and once that package was removed, kernel updates no longer created it; this guide could not verify whether DMIT's system image has that package installed.

**If it fails**:

- `Could not get lock /var/lib/dpkg/lock-frontend`: the system is updating automatically in the background; wait a few minutes and paste again.
- `Temporary failure resolving`: the server temporarily cannot resolve domain names; try again later, and if it keeps failing, open a ticket with DMIT.
- `E: Sub-process /usr/bin/dpkg returned an error code (1)`: run `dpkg --configure -a` on the server as root, follow the prompts on screen, and when it finishes, paste S4 again.

<!-- step:server-05-firewall -->
### S5 Set up the firewall

**Goal**: Allow only 22/tcp (SSH), 443/tcp (REALITY) and 443/udp (Hysteria2), without deleting any firewall rule already on the machine.

This step does three things:

1. It first checks that SSH listens on port 22. If not, it stops with an error, so that you do not lock yourself out.
2. It writes a small script, `/usr/local/sbin/sing-box-guide-firewall`, and runs it once right away:
   - When the machine has **no** firewall that drops inbound traffic by default (policy drop): it creates a new table that belongs only to this guide, `inet sing_box_guide`. Inbound traffic is dropped by default, and only established connections, local loopback, ICMP, DHCP replies, and 22/tcp, 443/tcp and 443/udp are allowed.
   - When the machine **already has** inbound chains: it adds three accept rules at the top of each existing inbound chain (established connections, 22 and 443/tcp, 443/udp), with the comment `sing-box-guide`. No existing rule is deleted or changed. If the existing firewall drops inbound traffic by default, no new table is created. Why the rules go into the existing chains: in nftables, a packet only gets through if every base chain on the same inbound hook accepts it; allowing it in a new table alone cannot get past a drop in the old table.
   - If an existing inbound chain is the `filter` table's `INPUT` chain managed by iptables (with Debian's default nf_tables backend), and `iptables -S INPUT` can read that chain, the script uses the `iptables`/`ip6tables` commands instead to add four accept rules (established connections, 22/tcp, 443/tcp, 443/udp, also with the comment `sing-box-guide`). If `nft` were used to add rules directly to such a chain, `iptables-save` would print nothing, `netfilter-persistent save` would save an empty rules file, and the existing rules would be gone after a reboot. For chains that `iptables -S INPUT` cannot read (for example rules you wrote yourself with nft that iptables does not understand), it still uses `nft` to add the three rules above.
   - Running it again does not add the rules twice.
3. It writes a systemd service, `sing-box-guide-firewall.service`, that runs this script again at every boot, before the network starts. nftables rules are not saved automatically by default and are lost at reboot. When the system's own `nftables.service` is restarted, reloaded, or stopped and then started again, it clears all rules, and this service then runs the script again. Only one case is not restored automatically: `nftables.service` was not running before and is started for the first time. That case, and rules cleared by some other program, can be found and fixed with "[P1 Checklist when you cannot connect](#p1-checklist-when-you-cannot-connect)".

Optional: the DMIT client area no longer stores the root password for the Console (per DMIT official announcement, verified 2026-11-17). If you want to be able to log in from the Console when SSH has problems (unverified; check the actual interface), first run `passwd` on the server as root to set a root password. Use a long random password (for example, generate one with `openssl rand -base64 18` and keep it somewhere safe). You can first run `sshd -T | grep -E '^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication) '` on the server as root to check: if it shows `permitrootlogin yes` together with `passwordauthentication yes` or `kbdinteractiveauthentication yes`, other people can also try this password to log in as root over SSH; if it shows `permitrootlogin without-password` (the same as `prohibit-password`), root cannot log in over SSH with a password.

**Command** (run on the server as root):

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

**Expected**: The last line is `active`. The lines before it include one or both of the following:

- `Firewall table inet sing_box_guide is active: inbound allows only 22/tcp, 443/tcp and 443/udp.` (the machine had no firewall that drops inbound traffic by default);
- `Added accept rules to existing chain: …`, or `Added accept rules with iptables to existing chain: …` and `Added accept rules with ip6tables to existing chain: …`, together with `An existing firewall with policy drop was found. …` (the machine already had a firewall).

Your current SSH connection does not drop. You can view all rules with `nft list ruleset`; if the machine manages its firewall with iptables, view them with `iptables -S INPUT` and `ip6tables -S INPUT`.

**If it fails**:

- `ERROR: nothing listens on TCP port 22`: your SSH is not on port 22. This guide only allows 22, so first check your SSH port (`ss -tlnp | grep ssh`), and do not go on with this step in this situation.
- SSH drops after you paste, and you cannot reconnect: log in from the Console in the DMIT client area (unverified; check the actual interface), which only works if you set a root password earlier; after logging in, run `systemctl disable --now sing-box-guide-firewall.service` and `nft delete table inet sing_box_guide`, then write down what happened and the output of `nft list ruleset` before you look into it.
- You manage the firewall with ufw or firewalld: when they reload, they overwrite the rules this script added. Use their own commands to allow 22/tcp, 443/tcp and 443/udp instead (for example `ufw allow 443/udp`).

<!-- step:server-06-install -->
### S6 Add the official sing-box repository and install

**Goal**: Install sing-box from the official apt repository, as the official sing-box docs describe ([sing-box installation docs](https://sing-box.sagernet.org/installation/package-manager/), verified 2026-11-15). No install script is used.

The package comes with an example `/etc/sing-box/config.json`, which is not this guide's config. So that the service does not start with it by mistake before S9, this step renames it to `/etc/sing-box/config.json.example` (only if it really is that example file). After the rename the file name no longer ends in `.json`, so the service does not read it.

**Command** (run on the server as root):

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

**Expected**: At the end it prints `sing-box version 1.14.4` (or a newer version number) and build information. The service has not started yet; this is normal.

**If it fails**:

- `curl: (6) Could not resolve host`, or the download times out: the server temporarily cannot reach sing-box.app; paste again a little later.
- `NO_PUBKEY` or `The following signatures couldn't be verified`: the key file did not download completely; paste this code block again.
- `Unable to locate package sing-box`: `apt-get update` did not read the sagernet repository; check that the content of `/etc/apt/sources.list.d/sagernet.sources` matches the block above.

<!-- step:server-07-target -->
### S7 Check the target site

**Goal**: Confirm that the `<REALITY_TARGET>` you chose supports TLS 1.3 and HTTP/2 when the server connects to it. For how to choose it, see "[The three values you replace](#the-three-values-you-replace)".

**Command** (run on the server as root; replace `<REALITY_TARGET>` first and keep the single quotes around it):

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

**Expected**: It prints `OK: <your-domain> supports TLS 1.3 and HTTP/2`.

**If it fails**:

- It prints `NOT OK`: the site does not support TLS 1.3 or HTTP/2, or the server cannot reach it. Choose another candidate site, change the domain inside the single quotes, and paste again.
- Before `NOT OK` you see `nodename nor servname provided` or `Name or service not known`: the domain is misspelled, or you have not yet replaced `<REALITY_TARGET>` with a real domain.

<!-- step:server-08-env -->
### S8 Generate guide.env and the Hysteria2 certificate

**Goal**: Generate all keys and passwords for this machine in one go and save them in `/etc/sing-box/guide.env` (permissions 600). If this file already exists, this step changes nothing, so pasting it again does not break clients that have already imported the profile.

Values in `guide.env`:

| Name | Contents |
|---|---|
| `SERVER_IP` | The server IPv4 address you entered |
| `REALITY_TARGET` | The target site domain you entered |
| `UUID` | REALITY user ID |
| `REALITY_PRIVATE_KEY`, `REALITY_PUBLIC_KEY` | The REALITY key pair (the private key stays on the server only; the public key goes to the client) |
| `SHORT_ID` | REALITY short ID |
| `HY2_PASSWORD` | Hysteria2 password |
| `HY2_CERT_SHA256` | The SHA-256 fingerprint (hex) of the whole Hysteria2 certificate, used by the `hysteria2://` share link's `pinSHA256` |
| `HY2_PUBKEY_SHA256` | The SHA-256 (base64) of the Hysteria2 certificate's public key, used by the sing-box client's `certificate_public_key_sha256` |

**How the Hysteria2 certificate is handled:** Hysteria2 must have a TLS certificate. This step generates a self-signed certificate (valid for 10 years), so you do not need a domain. Clients do not trust self-signed certificates by default, so the certificate has to be pinned: the sing-box client profile uses `certificate_public_key_sha256` with the SHA-256 (base64) of the certificate's public key, supported since sing-box 1.13.0 ([sing-box TLS docs](https://sing-box.sagernet.org/configuration/shared/tls/)); the `hysteria2://` share link uses the `pinSHA256` parameter with the SHA-256 fingerprint of the server certificate, following the official Hysteria URI format ([Hysteria URI format](https://v2.hysteria.network/docs/developers/URI-Scheme/)); for why the share link also carries `insecure=1`, see the note at the end of S11. These two values are different, and this step calculates both. The sing-box client profile does not use `insecure: true`, because that accepts any certificate; it also does not use `certificate_sha256`, because that needs sing-box 1.15.0, which is newer than the stable release at the time of writing. The name in the certificate is your target site domain, and the client's `server_name` uses it too.

**Command** (run on the server as root; replace `<SERVER_IP>` and `<REALITY_TARGET>` first and keep the single quotes):

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

**Expected**: On the first run it may first print a line of `-----` (openssl output), then `Created /etc/sing-box/guide.env:` and 9 lines of `NAME=value`, each with something after the equals sign. When you paste it again, it prints `… already exists. Nothing was changed. …`. These values are as good as your keys: do not screenshot them or paste them into forums, chat groups or GitHub issues when you ask for help; paste only the error output this step prints (a line that starts with `ERROR`, or an openssl or sing-box error message), never the contents of `/etc/sing-box/guide.env`.

**If it fails**:

- `ERROR: replace <SERVER_IP> and <REALITY_TARGET> …`: the placeholders in the code block have not been replaced. Replace them and paste again; nothing has been written yet at this point.
- `ERROR: SERVER_IP must be an IPv4 address …` or `ERROR: REALITY_TARGET must be a host name only …`: the value has the wrong format, for example a mistyped IP, or `https://` in front of the domain.
- `ERROR: … does not offer TLS 1.3 with HTTP/2 …`: this step runs the S7 check again, and the site did not pass. Choose another target site as in S7, change the domain inside the single quotes, and paste again; nothing has been written yet at this point.
- It stops before printing `Created …` (with an openssl or sing-box error before it, or `ERROR: a generated value is empty …`): one of the commands that generate the values failed (for example sing-box is not installed properly), and `guide.env` was not written. Read the error above and fix it first, then paste this code block again.
- You want to change a value (for example you entered the wrong IP): do not delete `guide.env` (deleting it generates new keys, and every client has to import again). For how to change it, see "[P2 Check whether the IP is blocked and change it](#p2-check-whether-the-ip-is-blocked-and-change-it)" and "[P4 Change the target site](#p4-change-the-target-site)".

<!-- step:server-09-config -->
### S9 Write the config, check it and start sing-box

**Goal**: Generate the server config from the values in `guide.env`. It is first written to `/etc/sing-box/config.json.new` and checked with `sing-box check`; only if the check passes does it replace `/etc/sing-box/config.json` and restart the service. If the check fails, the running service and the old config are not affected.

Key points of the config:

- REALITY: a VLESS inbound with `flow` set to `xtls-rprx-vision`, borrowing port 443 of `REALITY_TARGET` for the handshake ([sing-box VLESS inbound docs](https://sing-box.sagernet.org/configuration/inbound/vless/)).
- Hysteria2: the server sets no bandwidth (no `up_mbps` or `down_mbps`), and `ignore_client_bandwidth: true` is set. According to the sing-box docs, this means Hysteria2 does not use Brutal mode ([sing-box Hysteria2 inbound docs](https://sing-box.sagernet.org/configuration/inbound/hysteria2/)). You do not need to install any acceleration module.
- Routing: connections to private IP addresses (the server's internal network and the machine itself) are rejected, and so are connections to port 25 (sending mail). Under the AUP, accounts related to spam are terminated immediately, and one of the signs the AUP uses to spot a public proxy is spam complaints from other networks; rejecting port 25 keeps your node from being used by someone to send spam.
- The official systemd service uses `-C /etc/sing-box` to read and merge every file ending in `.json` in this directory (you can check this with `systemctl cat sing-box`). So this step first makes sure that the directory has no `.json` file other than `config.json`, so that `sing-box check` checks exactly what the service will read. This guide's client files are in `/root/dmit-guide/`, not in this directory.

**Command** (run on the server as root):

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

**Expected**: The last line is `OK: config checked, installed and sing-box restarted.` On the first run there may also be a `Created symlink …` line before it.

**If it fails**:

- `ERROR: /etc/sing-box/guide.env not found`: do S8 first.
- `ERROR: the sing-box service would also load: …`: `/etc/sing-box` contains other `.json` files, and the service would merge them with `config.json`. Move the listed files out of this directory (for example to `/root/`), then paste again.
- `sing-box check` prints `FATAL` and stops: the config was not replaced, and the service was not restarted. Write down the whole `FATAL` line, check `guide.env` for completeness against the output of S8, then paste again.
- `Job for sing-box.service failed`: run `journalctl -u sing-box -n 30 --no-pager` to see why.

<!-- step:server-10-listen -->
### S10 Confirm the service listens on port 443

**Goal**: Confirm that sing-box is running and listens on both TCP 443 and UDP 443. After S9 restarts the service, sing-box needs a moment before it starts listening, so this step waits up to 10 seconds before it decides.

**Command** (run on the server as root):

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

**Expected**: It prints two lines of port information, one starting with `udp` and one starting with `tcp` (their order does not matter), both showing `sing-box`; the last line is `OK: sing-box is running and listens on TCP 443 and UDP 443`.

**If it fails**:

- `FAIL: sing-box is not running`: read the log it prints. A common cause is another program using port 443; run `ss -tulpn | grep ':443'` to see which one.
- Only TCP or only UDP is missing: paste S9 again, then run this step again.

<!-- step:server-11-export -->
### S11 Generate the client profile and share links

**Goal**: From the values in `guide.env`, generate the client profile `/root/dmit-guide/client-profile.json` and the share-link file `/root/dmit-guide/share-links.txt` (directory permissions 700, file permissions 600), check the client profile, then print both and show QR codes for the two share links. Running it again gives the same content. These two files are not kept in `/etc/sing-box`, because the service reads every `.json` file there.

The client profile has four outbounds: `reality` (the TCP 443 connection), `hy2` (the UDP 443 connection), `auto` (tests `reality` and `hy2` every 1 minute and automatically uses the one that works and is faster) and `direct` (a direct connection, used only for private addresses). All other traffic goes through `auto`, which makes this a global proxy. DNS queries go through the proxy using DNS over HTTPS, and only IPv4 addresses are returned.

**Command** (run on the server as root):

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

**Expected**: In order, it prints the client profile (a block of JSON with no variable names starting with `$` left in it), the two share links (one starting with `vless://`, one starting with `hysteria2://`), and two QR codes. When `sing-box check` passes, it prints nothing. This output (the client profile, the share links and the QR codes) is as good as your keys: do not screenshot it or paste it into forums, chat groups or GitHub issues when you ask for help; paste only the line that starts with `ERROR` or `FATAL`, and if that line contains a UUID, a password or a key, replace it with `xxx` first.

**If it fails**:

- `ERROR: /etc/sing-box/guide.env not found`: do S8 first.
- `sing-box check` prints `FATAL`: write down that line; usually a line in `guide.env` is incomplete, so check it against the Expected result of S8.
- The QR codes look garbled: make the terminal window wider, or use the text of the share links instead.

Notes on the share links:

- The `vless://` link uses the common VLESS share-link format. This guide could not verify which apps can import it.
- The `hysteria2://` link is written as `insecure=1&pinSHA256=…`, following the official Hysteria URI format ([Hysteria URI format](https://v2.hysteria.network/docs/developers/URI-Scheme/)). The reason is that the server uses a self-signed certificate: the official Hysteria client uses `insecure` to decide whether to skip the normal CA certificate chain check, and compares `pinSHA256` in a certificate verification callback ([Hysteria client source code](https://github.com/HyNetworks/hysteria/blob/master/app/cmd/client.go)); but Go's TLS library says that when normal verification fails, this callback is never called ([Go crypto/tls docs](https://pkg.go.dev/crypto/tls#Config)). So a self-signed certificate with `insecure=0` cannot connect, and `insecure=1&pinSHA256=…` means "do not check the CA certificate chain, but still check the certificate fingerprint" (verified 2026-11-15).
- **The security of this Hysteria2 connection rests entirely on `pinSHA256`.** If an app ignores `pinSHA256`, it accepts any certificate, and someone who intercepts the connection could get your Hysteria2 password. This guide could not verify which apps check `pinSHA256` (verified 2026-11-15). So prefer the sing-box profile from Client setup below, which pins the certificate with `certificate_public_key_sha256`; import the `hysteria2://` link only if you cannot use the profile.
- An app cannot connect with the `hysteria2://` link: check that `pinSHA256` (the certificate fingerprint) is still there after the import, and that an option such as "allow insecure certificates" is turned on (it matches `insecure=1`). If you cannot confirm that the app checks `pinSHA256`, do not use this link; use the profile instead.

<!-- section:client -->
## Client setup

**Approach:** On all four platforms, use the official sing-box app and import the `client-profile.json` that S11 generated on the server. According to the official sing-box docs, these apps can all manage and run local or remote sing-box configuration files ([sing-box client docs](https://sing-box.sagernet.org/clients/), verified 2026-11-15); the official docs do not say whether they can import `vless://` or `hysteria2://` share links. So this guide relies mainly on the profile.

**How the two connections switch automatically:** The `auto` group in the profile tests `reality` and `hy2` every 1 minute and automatically uses the one that works and is faster; when one cannot connect, the client switches to the other after the next test. You do not need to pick a node by hand.

**About the app screens:** The app steps in C2 to C5 below are based on the official sing-box docs and were not tested one by one on real devices; menu names may differ slightly between versions. The official sing-box docs warn that some third-party projects claim to use sing-box but are not endorsed, and are often of poor quality and contain ads ([sing-box client docs](https://sing-box.sagernet.org/clients/)). Download only from the official channels listed below.

**Security:** `client-profile.json` and the share links are as good as your login credentials. Do not send them through public places, chat groups or cloud drives; move them to your own devices with a cable, AirDrop or another private method you trust.

<!-- step:client-01-profile -->
### C1 Download the client profile to your computer

**Goal**: Copy `client-profile.json` and `share-links.txt` from the server to your own computer, then send them on to your phone or import them on the computer.

**Command** (run on your own computer; replace `<SSH_USER>` and `<SERVER_IP>` first):

```bash local
ssh <SSH_USER>@<SERVER_IP> sudo cat /root/dmit-guide/client-profile.json > client-profile.json
ssh <SSH_USER>@<SERVER_IP> sudo cat /root/dmit-guide/share-links.txt > share-links.txt
chmod 600 client-profile.json share-links.txt
head -c 200 client-profile.json
```

If you cannot easily use the commands above (for example on Windows), you can also run `cat /root/dmit-guide/client-profile.json` in the root window on the server, select and copy all of the output, paste it into a new file in a text editor on your computer, and save it as `client-profile.json` (choose UTF-8 encoding).

The content of this file is the template below, with the parts that start with `$` already replaced by the server with the values from `guide.env` (S11 uses exactly this template):

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

A few notes: `uTLS` is marked "Not Recommended" in the sing-box docs, but a REALITY client must enable uTLS (if you remove `utls`, `sing-box check` reports `uTLS is required by reality client`), so this profile uses the `chrome` fingerprint. In `inbounds`, the two `address` entries of `tun` are addresses of the virtual network adapter that the app creates on your device, not server addresses. In `dns`, the `server` is Cloudflare's public DNS, and queries are sent through the proxy (`auto`).

**Expected**: `client-profile.json` and `share-links.txt` appear in the current directory (the directory you were in when you ran these commands; you can see it with `pwd`); the output of `head` begins with `{` and `"log"`.

**If it fails**:

- `sudo: a terminal is required to read the password`: your normal user needs a password for sudo; in that case, use the copy-and-paste method above.
- `No such file or directory`: S11 has not been run on the server yet.
- The file is empty: read the error message that the command printed; you can also use the copy-and-paste method instead.
- If you change `guide.env` later (a new IP or a new target site): run S11 again, do C1 again, and import the profile again in the app.

<!-- step:client-02-ios -->
### C2 iOS: the official sing-box app

**Goal**: Import the profile and connect on an iPhone or iPad.

**Do** (per the official sing-box docs, verified 2026-11-15):

1. With an Apple account from a region outside mainland China, install the official sing-box app from the App Store (the official docs call it "sing-box for Apple platforms"; it needs iOS 15.0 or later). TestFlight beta places are for sponsors only. Source: [sing-box Apple client docs](https://sing-box.sagernet.org/clients/apple/).
2. Move `client-profile.json` to the device (for example with AirDrop, or into the Files app with a cable).
3. On the Profiles page of the sing-box app, create a new local profile, choose to import it from a file, and select `client-profile.json`.
4. Go back to the home page, select this profile and tap start; the first time, the system asks to add a VPN configuration; choose Allow.

**Expected**: The app shows that it is connected, and a VPN icon appears in the status bar; check the exit IP as described in "[Check that it works](#check-that-it-works)".

**If it fails**:

- You cannot find it in the App Store: Apple accounts from the mainland China region cannot download it; the official docs require an account from outside mainland China.
- An error appears during import: make sure the file is complete JSON (open it in a text editor; the first line is `{` and the last line is `}`), then import it again.
- Alternative (not the official sing-box app, and not recommended by this guide): Shadowrocket (App Store, US$2.99). Its version history mentions fixes for VLESS and Hysteria 2, and importing through server URLs, subscriptions and QR codes ([App Store page](https://apps.apple.com/us/app/shadowrocket/id932747118), verified 2026-11-15). This guide could not verify whether it imports this guide's share links correctly or whether it checks `pinSHA256`; if it does not check `pinSHA256`, a Hysteria2 connection imported from the share link accepts any certificate.

<!-- step:client-03-android -->
### C3 Android: the official sing-box app

**Goal**: Import the profile and connect on an Android phone.

**Do** (per the official sing-box docs, verified 2026-11-15):

1. Install the official sing-box app (the official docs call it SFA; it needs Android 5.0 or later). Official channels: Google Play (including Beta), [GitHub Releases](https://github.com/SagerNet/sing-box/releases) and F-Droid (the F-Droid build uses a unified signature and is a reproducible build). Source: [sing-box Android client docs](https://sing-box.sagernet.org/clients/android/).
2. Move `client-profile.json` to the phone.
3. On the Profiles page of the app, create a new local profile, choose to import it from a file, and select `client-profile.json`.
4. Go back to the home page, select this profile and tap start; the first time, the system asks to set up a VPN connection; tap OK (unverified; check the actual interface).

**Expected**: The app shows that it is connected, and a VPN icon (a key icon) appears in the notification bar.

**If it fails**:

- Google Play does not open: download from GitHub Releases or F-Droid instead.
- You installed a third-party app with a similar name: uninstall it and use only the official channels listed above.
- An error appears during import: same as in C2.

<!-- step:client-04-windows -->
### C4 Windows: the official sing-box desktop app

**Goal**: Import the profile and connect on a Windows computer.

**Do** (per the official sing-box docs, verified 2026-11-15):

1. Download the official sing-box desktop app from [GitHub Releases](https://github.com/SagerNet/sing-box/releases) (the official docs call it "sing-box for Desktop" and list Windows 10 or later, with x64, x86 and arm64 support). Source: [sing-box desktop client docs](https://sing-box.sagernet.org/clients/desktop/). Note: its code repository description labels the Windows client "Experimental" ([sing-box-for-desktop](https://github.com/SagerNet/sing-box-for-desktop)); the docs and the repository description do not agree, and this guide could not verify how mature it is.
2. Open the app, create a new local profile, and import `client-profile.json` from a file.
3. Select this profile and start it.

**Expected**: The app shows that it is connected, and the browser can open websites outside China.

**If it fails**:

- It fails to start or says it lacks permission: the profile uses a virtual network adapter (tun), so try running the app as administrator.
- The desktop app does not work for you: an alternative (not the official sing-box app) is v2rayN ([GitHub](https://github.com/2dust/v2rayN)), which supports the Xray and sing-box cores; its description does not mention importing share links or JSON profiles, and this guide could not verify that.

<!-- step:client-05-macos -->
### C5 macOS: the official sing-box app

**Goal**: Import the profile and connect on a Mac.

**Do** (per the official sing-box docs, verified 2026-11-15):

1. Install the official sing-box app (the official docs call it SFM; like the iOS app, it is part of "sing-box for Apple platforms", and it needs macOS 13.0 or later and an Apple account from a region outside mainland China). The channels that the official docs list for the standalone macOS version are Homebrew Cask (if you already have Homebrew, you can run `brew install sfm` in the terminal) and [GitHub Releases](https://github.com/SagerNet/sing-box/releases). This guide could not verify whether it is available on the Mac App Store. Source: [sing-box Apple client docs](https://sing-box.sagernet.org/clients/apple/).
2. Open the app, create a new local profile, and import `client-profile.json` from a file.
3. Select this profile and start it; the first time, the system asks to add a VPN configuration or a system extension; choose Allow.

**Expected**: The app shows that it is connected, and the menu bar or System Settings shows that the VPN is connected.

**If it fails**:

- The system says that an extension was blocked: allow it in "System Settings → Privacy & Security", then start the connection again.
- You do not have Homebrew: do not run Homebrew's install script just for this; download from GitHub Releases instead.

<!-- section:verify -->
## Check that it works

<!-- step:verify-01-exit-ip -->
### V1 Check the exit IP

**Goal**: Confirm that your traffic leaves through your DMIT server.

**Command** (run on a computer that is already connected through the proxy; on a phone, just open https://ipinfo.io/json in the browser):

```bash local
curl -s https://ipinfo.io/json
```

On Windows, type `curl.exe -s https://ipinfo.io/json` in PowerShell. In Windows PowerShell 5.1, `curl` is an alias for `Invoke-WebRequest`, not curl itself; Windows 10 and 11 include curl, and you only get it by typing `curl.exe` ([Microsoft docs](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-5.1), [curl website](https://curl.se/windows/microsoft.html), verified 2026-11-16).

**Expected**: `"ip"` equals your server IP (`<SERVER_IP>`). `"country"` may show `HK`, and `"org"` shows the network that the server belongs to. Note: DMIT assigns IPs randomly and does not guarantee IP geolocation (per DMIT official docs, verified 2026-11-17); see the [IP FAQ](https://docs.dmit.io/guide/faq/ip). So as long as `"ip"` matches, the proxy is working.

**If it fails**:

- It shows the IP of your local network: the proxy is not working. Check that the app shows it is connected; on a computer, make sure no other proxy software is running at the same time.
- The page does not open: troubleshoot with "[P1 Checklist when you cannot connect](#p1-checklist-when-you-cannot-connect)".

<!-- step:verify-02-dns-leak -->
### V2 Basic DNS leak check

**Goal**: Confirm that domain name lookups also go through the proxy, instead of going to your local ISP.

**Do**: After you connect to the proxy, open https://www.dnsleaktest.com/ in the browser, click "Standard test" and wait for the result.

**Expected**: The DNS servers in the result list all belong to Cloudflare (the DNS in this guide's profile is Cloudflare's public DNS, and queries are sent through the proxy), not to China Telecom, China Unicom, China Mobile or another ISP where you are.

**If it fails**:

- Your local ISP's DNS appears in the result: make sure you imported the complete profile generated by S11 (it contains the `hijack-dns` rule), then disconnect, reconnect and test again.
- The result shows Google or another DNS provider: usually the browser itself has turned on "Secure DNS"; those queries also go through the proxy, so they do not leak to your local ISP.
- The site does not open: complete V1 first.

<!-- step:verify-03-both-lines -->
### V3 Confirm both connections work

**Goal**: Confirm that both REALITY and Hysteria2 work, so that when one has problems, the other can take over.

**Do**: In the sing-box app, open the Groups page, find the `auto` group, and view or refresh the latency test results for `reality` and `hy2` (the screen names are based on the official sing-box app, were not tested on real devices, and may differ slightly between versions).

**Expected**: Both `reality` and `hy2` show a number of milliseconds, not a timeout or an error.

**If it fails**:

- Only `hy2` times out: the network you are on may restrict UDP, or UDP 443 is not allowed on the server. Run S10 on the server again; if your local network restricts UDP, using only `reality` is also fine.
- Only `reality` times out: run S7 on the server again to confirm that the target site still passes; if it does not, follow "[P4 Change the target site](#p4-change-the-target-site)".
- Both time out: troubleshoot with "[P1 Checklist when you cannot connect](#p1-checklist-when-you-cannot-connect)".

<!-- section:daily -->
## Routine maintenance

<!-- step:daily-01-panel -->
### D1 Check traffic in the DMIT client area

**Goal**: See how much traffic you have used this month and when it resets next, so that the instance is not paused because the traffic ran out.

**Do**: Log in to the DMIT client area and open the management page of this machine to see its traffic (unverified; check the actual interface). According to the [Instance FAQ](https://docs.dmit.io/guide/faq/instance), traffic resets automatically every month, and the control panel shows the next reset time (per DMIT official docs, verified 2026-11-17).

**Expected**: You can see the next reset time (per DMIT official docs, verified 2026-11-17) and the used traffic, wherever the page shows it (unverified; check the actual interface). DMIT HKG.Pro TINY and DMIT HKG.Pro STARTER count traffic in both directions: both upload and download count (per DMIT official docs, verified 2026-11-17).

**If it fails**:

- You cannot find the traffic information: open a ticket with DMIT; the TOS only guarantees a reply within 72 hours (per DMIT official docs, verified 2026-11-17).
- The traffic is almost used up: see "[P3 Traffic used up or service paused](#p3-traffic-used-up-or-service-paused)".

<!-- step:daily-02-vnstat -->
### D2 Track traffic on the server with vnstat

**Goal**: Keep your own monthly traffic record on the server, to compare with the figures in the DMIT client area. `vnstat` comes from Debian's own repositories.

**Command** (run on the server as root; do S2 and S3 first):

```bash server
(
set -eu
DEBIAN_FRONTEND=noninteractive apt-get -y install vnstat
systemctl enable --now vnstat.service
vnstat -m
)
```

**Expected**: Right after installation it shows that there is no data yet (for example `eth0: No data.`; the network interface name may differ). Run `vnstat -m` again after a while, and it shows `rx` (received), `tx` (sent) and `total` for each month. DMIT counts traffic in both directions (per DMIT official docs, verified 2026-11-17), so compare with `total`. vnstat only counts from the time you installed it; the figures in the DMIT client area are authoritative.

**If it fails**:

- `Unable to open database`, or still no data: run `systemctl status vnstat` to see whether the service is running, and look again after a few minutes.

<!-- step:daily-03-upgrade -->
### D3 Upgrade the system and sing-box

**Goal**: Install system security updates regularly, and upgrade sing-box to the latest stable release from the official repository.

**Command** (run on the server as root; do S2 and S3 first):

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

**Expected**: It prints the new `sing-box version …`, and the last line is `active`. If this upgrade installed a new kernel, the server must reboot before it uses that kernel; if you are not sure, paste S4 again and look at its last line.

**If it fails**:

- `sing-box check` prints `FATAL` and stops: the new version does not accept the current config. The service was not restarted, and the running process is still the old version (the sing-box package does not restart the service by itself when it is upgraded). But as soon as the service restarts (including when the server reboots), it starts with the new version and fails, so deal with this soon. First read the [sing-box release notes](https://github.com/SagerNet/sing-box/releases); if you need to go back to the old version, run `apt-cache policy sing-box` to see the available versions, run `apt-get install --allow-downgrades sing-box=<version>` to install the old one, and then run `systemctl restart sing-box.service`.
- Keep the sing-box app on your phone and computer up to date as well; the client version must not be lower than 1.13.0.

<!-- section:trouble -->
## Troubleshooting

<!-- step:trouble-01-checklist -->
### P1 Checklist when you cannot connect

**Goal**: Rule out the common causes one by one, in order.

**Command** (run on the server as root; do S2 and S3 first):

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

Then check each item:

1. **Can you log in to the server?** If even SSH cannot connect, first check in the DMIT client area that the machine is running (unverified; check the actual interface); if the machine is running but cannot be reached from the mainland, see P2.
2. **Has DMIT paused the service?** The order page says that a machine that goes over its traffic is paused, shown as Paused / Quota Exceeded (per DMIT official docs, verified 2026-11-17); see P3. A monthly machine is kept for only 3 days after it expires, and paying within those 3 days restores it (per DMIT official docs, verified 2026-11-17).
3. **Services, ports and firewall:** The first two lines from the command above should be `active` twice, TCP and UDP 443 should be listed, and it should print `firewall rules: OK`. If the services or ports are wrong, run S5, S9 and S10 again. If it prints `firewall rules: MISSING`, the firewall rules this guide added have been cleared (for example someone reloaded a different firewall config), even though the firewall service may still show `active`: run `systemctl restart sing-box-guide-firewall.service`, then run the command above again. If that says `start of the service was attempted too often` (too many restarts in a short time), first run `systemctl reset-failed sing-box-guide-firewall.service`, then restart it once more.
4. **The time must be right:** The server should show `System clock synchronized: yes`; on phones and computers, turn on "Set time automatically". If the clocks are too far off, TLS certificate checks may fail. If the server shows `no`, run `timedatectl set-ntp true`, then run the command above again after a minute or two; if it shows `NTP service: n/a` (the system has no usable time sync service, and in that case `timedatectl set-ntp true` reports `NTP not supported`), first run `apt-get -y install systemd-timesyncd`, then run `timedatectl set-ntp true` ([timedatectl manual](https://manpages.debian.org/trixie/systemd/timedatectl.1.en.html), verified 2026-11-16).
5. **Is the client profile up to date?** After you change `guide.env`, run S11 and C1 again, and import the profile again in the app.
6. **Does the target site still pass?** Run S7 again. If it does not pass, see P4.
7. **Only one connection fails:** See "If it fails" in V3.
8. **Log:** The last part of the output is sing-box's recent log; look for lines that start with `ERROR` or `FATAL`.

**Expected**: Once you find and fix one of these items, the client works again after it reconnects.

**If it fails**: If everything above looks normal and you still cannot connect, the IP may be blocked; see P2.

<!-- step:trouble-02-ip-blocked -->
### P2 Check whether the IP is blocked and change it

**Goal**: Find out whether the server IP is blocked from the mainland; if it is, change the IP under DMIT's rules and update the config.

**Command** (first run these on your own computer in the mainland, **with the proxy turned off**; replace `<SERVER_IP>`):

```bash local
ping -c 4 <SERVER_IP>
nc -vz -w 5 <SERVER_IP> 22
nc -vz -w 5 <SERVER_IP> 443
```

Windows `ping` has no `-c`; use `/n` to set the count. On Windows, run `ping /n 4 <SERVER_IP>`, `Test-NetConnection <SERVER_IP> -Port 22` and `Test-NetConnection <SERVER_IP> -Port 443` in PowerShell, one after another; `TcpTestSucceeded : False` in the result means that port cannot be reached ([ping docs](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/ping), [Test-NetConnection docs](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection), verified 2026-11-16).

If every ping is lost and neither 22 nor 443 can be reached, and the server has not been paused for running out of traffic or for an unpaid bill, the IP is very likely blocked.

**Rules for changing the IP**, from the [IP FAQ](https://docs.dmit.io/guide/faq/ip) (per DMIT official docs, verified 2026-11-17):

- A free change needs all of these: at least 7 days since purchase; at least 15 days since the last change; ICMP and all TCP ports blocked for the whole IP; at least 7 days of service left. If the IP was already blocked when the instance was delivered, you can open a ticket for a free change.
- Paid change: you can request one at any time, US$5 each time, with at least 7 days between requests; a completed change cannot be undone or refunded; abuse may get you banned from this feature.
- Request it through a ticket. The TOS only guarantees a reply within 72 hours; the Ticket FAQ says usually within 24 hours.

**After you get the new IP**, do S2 and S3 again with the new IP, then run the code block below (replace `<SERVER_IP>` with the **new** IP and keep the single quotes):

```bash server
(
set -eu
NEW_IP='<SERVER_IP>'
printf '%s\n' "$NEW_IP" | grep -Eqx '[0-9]{1,3}(\.[0-9]{1,3}){3}' || { echo "ERROR: replace <SERVER_IP> with the new IPv4 address first" >&2; exit 1; }
sed -i "s/^SERVER_IP=.*/SERVER_IP=$NEW_IP/" /etc/sing-box/guide.env
grep '^SERVER_IP=' /etc/sing-box/guide.env
)
```

Then run S11 again, do C1 again, and import the profile again in every app. The server config does not contain the IP, so you do not need to redo S9.

**Expected**: It prints `SERVER_IP=<new-IP>`; after you import again, V1 shows the new IP as the exit IP.

**If it fails**:

- `ERROR: replace <SERVER_IP> …`: the placeholder in the code block has not been replaced with the new IP yet.
- You do not meet the conditions for a free change: you can choose a paid change, or wait until you meet the conditions and then apply.

<!-- step:trouble-03-traffic -->
### P3 Traffic used up or service paused

**Goal**: Know what to do when the traffic runs out.

**Do**: In the DMIT client area, check this machine's status (unverified; check the actual interface) and the next traffic reset time, then pick one of the options below (per DMIT official docs, verified 2026-11-17):

- Wait for the automatic monthly reset; see the [Instance FAQ](https://docs.dmit.io/guide/faq/instance).
- Pay for an early reset: most standard plans can pay for a reset, which costs the same as this machine's monthly fee (the renewal price divided by the billing cycle). Extra traffic packs are only for the sPro series.
- Upgrade to a bigger plan in the same series, for example from DMIT HKG.Pro TINY to DMIT HKG.Pro STARTER. The upgrade is charged at the price difference between the new and old plans, after the upgrade you are billed at the new plan's price, and downgrades are not supported for now. The official docs do not say whether a paused machine is restored right away after an upgrade.

**Expected**: After the paid reset is done, this machine's traffic is reset (per DMIT official docs, verified 2026-11-17). Then check with V1 that you can connect.

**If it fails**:

- The machine is still paused after you renew: according to the [Billing FAQ](https://docs.dmit.io/guide/faq/billing), a machine that has gone over its traffic is not restored after payment and has to wait for the next traffic reset; one that has not gone over is usually restored automatically within 10 minutes (per DMIT official docs, verified 2026-11-17).
- Still unpaid 3 days after the due date: the machine is deleted and the data cannot be recovered (per DMIT official docs, verified 2026-11-17); you have to order again and start from S1.

<!-- step:trouble-04-target -->
### P4 Change the target site

**Goal**: Switch to another target site when the current `<REALITY_TARGET>` no longer passes (S7 prints `NOT OK`) or the REALITY connection does not work.

**Command**: First run S7 once with the new domain and make sure it prints `OK`. Then run the code block below on the server as root (replace `<REALITY_TARGET>` with the new domain and keep the single quotes):

```bash server
(
set -eu
NEW_TARGET='<REALITY_TARGET>'
printf '%s\n' "$NEW_TARGET" | grep -Eqx '[A-Za-z0-9-]+(\.[A-Za-z0-9-]+)+' || { echo "ERROR: replace <REALITY_TARGET> with the new host name first" >&2; exit 1; }
sed -i "s/^REALITY_TARGET=.*/REALITY_TARGET=$NEW_TARGET/" /etc/sing-box/guide.env
grep '^REALITY_TARGET=' /etc/sing-box/guide.env
)
```

Then run S9, S10 and S11 again in that order, do C1 again, and import the profile again in every app. The Hysteria2 certificate does not need to be regenerated: the client authenticates the server by the pinned certificate fingerprint.

**Expected**: It prints `REALITY_TARGET=<new-domain>`; S9 prints `OK` and S10 prints `OK`; after you import again, `reality` shows a latency in V3.

**If it fails**:

- `ERROR: replace <REALITY_TARGET> …`: the placeholder in the code block has not been replaced yet.
- `sing-box check` in S9 reports an error: check that the `REALITY_TARGET=` line in `guide.env` contains only the domain.

<!-- section:faq -->
## FAQ

### Is the DMIT HKG.Pro TINY traffic allowance enough?

It depends on how much you use each month: the 500 GB is bidirectional traffic, so the amount you can really use is about half. Estimate first with the data statistics on your phone and computer, then check with D1 and D2; if it is not enough, you can upgrade within the same series to DMIT HKG.Pro STARTER (1000 GB of bidirectional traffic), but downgrades are not supported for now (per DMIT official docs, verified 2026-11-17).

### Why use both REALITY and Hysteria2?

To have a second connection, so that the two back each other up. REALITY uses TCP 443 and Hysteria2 uses UDP 443; the client's `auto` group tests both every 1 minute, normally uses the one that works and is faster, and switches to the other automatically when one fails; but both connections can also fail at the same time.

### Can I share it with family or friends?

Only for yourself and a few trusted people close to you; do not make it public and do not charge for it. The DMIT AUP allows encrypted private proxies and forbids public proxy services and resale (per DMIT official docs, verified 2026-11-17); the AUP gives no number for how many people count as "public", so the fewer people, the safer.

### Does buying through the affiliate link cost more?

DMIT's official docs do not say, and this guide cannot confirm it (per DMIT official docs, verified 2026-11-17). If you do not want to use the affiliate link, open the [DMIT website](https://www.dmit.io/) directly and order there.

### Can I use WireGuard or OpenVPN?

Do not use them to connect back to the mainland. The DMIT AUP forbids building VPN tunnels to China, lists L2TP, OpenVPN, PPTP, IPSec and Cisco AnyConnect, and says "including but not limited to" (per DMIT official docs, verified 2026-11-17).

### Should I install BBR acceleration or another speed-up script?

No. The DMIT AUP forbids network acceleration modules with unacceptable settings (examples include FinalSpeed and unofficial BBR) (per DMIT official docs, verified 2026-11-17). The Hysteria2 setup in this guide is already configured not to use Brutal mode, so you do not need to install anything extra.

### Does traffic to websites inside China also go through Hong Kong?

Yes. The client profile in this guide is a global proxy: apart from private addresses, all traffic goes through the server, so visiting websites inside China also uses your traffic and may be slower. When you do not need to get past the GFW, you can disconnect in the app.

### Why do some websites or services (such as streaming or AI) not open, or say I am in the wrong region?

DMIT does not guarantee that any website or service can be reached; IPs are assigned randomly, and their geolocation is not guaranteed either (per DMIT official docs, verified 2026-11-17); see the [IP FAQ](https://docs.dmit.io/guide/faq/ip). This comes from the IP itself and has nothing to do with this guide's config.

### Can I use Shadowrocket or v2rayN?

You can try, but they are not the official sing-box app, and this guide has not verified whether they import this guide's share links correctly or whether they check Hysteria2's `pinSHA256`; if they do not check it, a Hysteria2 connection imported from a share link accepts any certificate. This guide relies mainly on the official sing-box app with the profile.

### What if my IP is blocked?

First confirm it with P2, then apply for a change under DMIT's rules: if you meet the conditions, the change is free; otherwise it costs US$5 each time (per DMIT official docs, verified 2026-11-17). After the change, update the IP in `guide.env`, then generate the client profile again and import it.

### What happens when the traffic runs out?

The machine is paused, and the order page shows `Paused / Quota Exceeded`; you can wait for the automatic monthly reset, or pay for an early reset, which costs the same as the monthly fee (per DMIT official docs, verified 2026-11-17). See P3.

### Can I get a refund if I am not satisfied?

A new order within 3 days and with no more than 30 GB of traffic used can get a full refund, but the TOS says payment fees are deducted; within 30 days you can request a partial refund. Renewal orders, dissatisfaction with the network, IP geolocation and similar cases are not refunded (per DMIT official docs, verified 2026-11-17); see [TOS](https://www.dmit.io/pages/tos) section 19.

### Does it renew automatically?

Yes. The TOS says the service is billed automatically in recurring cycles until you cancel it through the cancellation procedure (section 18.1); to turn on or keep automatic payment and automatic renewal, two-factor authentication (2FA) must be turned on for the account and kept on (section 18.9) (per DMIT official docs, verified 2026-11-17).

### Is this an official DMIT guide?

No. This is an unofficial guide written by an individual and has no connection with DMIT; only the DMIT order link in this guide is the author's affiliate link.

<!-- section:risks -->
## Risks

### Legal risk

Using unapproved methods to get past the GFW in mainland China carries a risk of penalties. Below are examples from public reports; the amounts are quoted by the authors of those reports, and this guide has not checked each one against its original official source:

- A WeChat article republished by China Digital Times on 2026-06-03 lists these cases: an individual in Nanchang was warned and fined RMB 15,000 for using a VPN (the article gives no date); a company employee who watched foreign streaming media on an office computer was warned and fined RMB 15,000 (the article gives no date); an individual in Shouning County, Ningde, Fujian, who got past the GFW in 2020 was discovered in July 2024 and summoned and penalized (the article gives no amount) ([China Digital Times](https://chinadigitaltimes.net/chinese/727752.html)).
- A Human Rights Watch (HRW) article of 2026-03-17: China's Ministry of Public Security published a draft cybercrime prevention law on 2026-01-31, under which serious cases can bring a fine of up to RMB 5,000,000, detention of up to 15 days, and a ban on leaving the country for 6 months to 3 years; the article does not mention VPNs or individuals getting past the GFW ([HRW](https://www.hrw.org/zh-hant/news/2026/03/17/china-cybercrime-bill-entrenches-censorship-surveillance)).

Whether and how you use it is your own decision, and you bear the consequences.

### No protocol stays unblocked forever

All of the following are third-party research or users' own reports, not confirmations from DMIT or any official body:

- GFW Report recorded that on 2025-08-20, from about 00:34 to 01:48 Beijing time (about 74 minutes), the GFW unconditionally injected forged RST+ACK packets on TCP port 443, affecting port 443 connections between China and the rest of the world; other ports were not affected, and the cause is unknown ([GFW Report](https://gfw.report/blog/gfw_unconditional_rst_20250820/en/)). In a case like this, REALITY is affected too.
- From 2026-01-05, several users in the Xray-core discussions reported REALITY / VLESS being blocked; some had the whole IP blocked, even port 22, and some reported that the same connection failed on home broadband but worked over a 4G hotspot; these are users' own accounts, without official confirmation or systematic measurement, and the discussion went on until 2026-07-02 ([Xray-core discussion](https://github.com/XTLS/Xray-core/discussions/5494)). This guide found no report with a method and date that shows this was a single large-scale event.
- Around 2026-10-26, forum users called that day the "crash" day (坠机), when many proxy nodes failed at the same time; there were reports of VLESS being affected and reports of Hysteria2 being affected ([VilaVPN blog](https://blog.vilavpn.com/blocking-roundup-2026-10/), 2026-10-27; this is the blog of a commercial VPN provider, and the author says it only collects users' own accounts). One blogger saw 8 of his own 9 VPSes go offline after he changed their UUIDs on 2026-10-26 ([elmagnifico.tech](https://elmagnifico.tech/2026/10/27/VPN-Compare/)).
- On 2026-08-20, a user reported that most services for getting past the GFW had stopped working in Fuzhou, Fujian; the author says no formal technical check was done ([net4people #651](https://github.com/net4people/bbs/issues/651)).
- Some research discusses a possible cross-layer RTT fingerprint problem in Hysteria v2 (when Brutal is not used); this is an academic discussion, not a report of measured blocking ([net4people #528](https://github.com/net4people/bbs/issues/528)). Separately, a USENIX Security 2026 paper analyzed the source code of the Geedge Networks firewall that leaked in 2025-09 ([net4people #653](https://github.com/net4people/bbs/issues/653)).

So the setup in this guide cannot be guaranteed to keep working; when it is blocked, you may need to change the IP (see P2) or the target site (see P4). This guide has not done any testing in mainland China, and real results vary by region and carrier.

### The risk of having only one VPS

This guide uses only one server. If its IP is blocked, it is paused because the traffic ran out, the bill is unpaid, or DMIT has an outage, both connections stop working at the same time. Changing a blocked IP comes with conditions and waiting time (see P2), and a ticket reply can take up to 72 hours (per DMIT official docs, verified 2026-11-17). If your work or life depends on it, plan another way to get online in advance. This guide does not cover setups with several servers.

<!-- section:changelog -->
## Changelog

- 2026-11-15: First version (draft). Facts last verified 2026-11-15; written for sing-box 1.14.4.
- 2026-11-16: Tested the server setup and the client connections between two Debian 13 (arm64) virtual machines (for the scope, see "[Server setup](#server-setup)"), and made changes based on the results: S8 checks the target site again, and generates all values before it writes anything, so if any generating command fails or any generated value is empty, it stops and writes nothing; S10 waits for the service to start listening; when the machine already manages its firewall with iptables and `iptables -S INPUT` can read the existing chain, S5 adds the rules with iptables commands instead; the firewall service runs before the network starts, and adds its rules again whenever `nftables.service` is restarted, reloaded, or stopped and started again; P1 checks the firewall rules and explains how to fix missing rules and an unsynchronized clock; added Windows commands for V1 and P2; added `--allow-downgrades` to the rollback command in D3.
- 2026-11-16: Had an AI agent that was given only this guide complete the server setup on a fresh test virtual machine, and revised the places where it had to guess: explained how an AI agent runs the server code blocks non-interactively over SSH (in a macOS or Linux terminal, with LF line endings in the file, and with the reader confirming the server fingerprint on the first connection); added examples for `<REALITY_TARGET>` and explained where else this domain is used; S4 now ends by printing whether a reboot is needed, and D3 also reminds you to reboot after a new kernel is installed; the Expected result of S10 no longer implies an order for its two lines; C1 says which directory the files are saved in.
- 2026-11-17: Checked DMIT's official pages, docs and announcements, and the latest stable sing-box release, again on 2026-11-17: prices, plans, rules and the sing-box version (1.14.4) have not changed, and the verification dates of these facts now read 2026-11-17. The route row of the fact table now says that DMIT's figure of about 15 ms is only a reference measurement from Hong Kong to one nearby mainland city. Corrected the Tier 1 note: its source is DMIT's IP FAQ, which says that a newly assigned T1 IP is not guaranteed to be reachable from sensitive regions on the first connection, while Premium and Eyeball are guaranteed to be reachable worldwide on the first connection (handled through a ticket); the earlier wording, that T1 IPs are not guaranteed to work in every country and region, was not on the pages it cited. The IP management row now says that the US$1 fee starts with the renewal invoices issued on or after 2026-12-01 (it said "after"). S2 now says that a reinstall can change the server's host key, instead of saying that it always does. S8 now tells you to paste the error output it prints when you ask for help (a line that starts with `ERROR`, or an openssl or sing-box error message), never the contents of `guide.env`.
