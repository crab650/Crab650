---
source: personal-learning-workspace
content_type: note
source_id: 18
published_date: 2026-08-15
---

# 從 Philips Camera 實驗理解網路

> 分類：網路  
> Tags：無  
> 建立日期：2026-08-15

# 網路基礎學習筆記：從 Philips Camera 實驗理解網路

日期：2026-08-15

## 一、這次為什麼開始學網路？

原本的目標很簡單：

> 家裡有一台 Philips 網路攝影機，我能不能用 Python 對它做一些開發？

但是開始研究後發現，要控制或分析一台網路設備，首先必須理解：

- IP Address
- LAN
- Subnet
- Gateway
- MAC Address
- ARP
- Ping / ICMP
- TCP / UDP
- Port
- Router
- NAT
- Packet Capture
- PCAP
- ONVIF
- WS-Discovery
- IoT Cloud / P2P

因此這台 Camera 變成了一個實際的「家庭網路實驗設備」。

---

# 二、我的家庭網路實驗環境

目前確認的設備：

Windows PC：

192.168.110.169

Philips Camera：

192.168.110.34

Reyee Router：

192.168.110.1

Router WAN：

192.168.1.2

大致網路架構：

Windows PC
192.168.110.169
        │
        │
        ▼
Reyee Router
LAN：192.168.110.1
        │
        ├──────── Philips Camera
        │         192.168.110.34
        │
        ▼
WAN：192.168.1.2
        │
        ▼
上一層 Router
192.168.1.1
        │
        ▼
Internet

---

# 三、IP Address 是什麼？

IP Address 可以先理解成：

> 一台設備在 IP 網路中的地址。

例如：

192.168.110.1   → Router
192.168.110.34  → Camera
192.168.110.169 → PC

這三台設備都位於：

192.168.110.0/24

可以暫時把：

192.168.110

理解成同一個「網路社區」。

最後面的：

.1
.34
.169

則用來區分這個網路中的不同設備。

---

# 四、LAN 是什麼？

LAN：

Local Area Network

中文：

區域網路

例如家裡的：

PC
Camera
手機
Router
智慧家電

可以組成一個 LAN。

我的 LAN 目前是：

192.168.110.0/24

Camera 與 PC 都在這個 LAN 中。

---

# 五、Subnet Mask 是什麼？

我的 Windows PC 可以看到：

Subnet Mask：

255.255.255.0

它與：

192.168.110.0/24

是相關的表示方式。

目前先理解：

> Subnet Mask 幫助設備判斷「目標 IP 是不是跟我在同一個網路」。

例如：

PC：

192.168.110.169

Camera：

192.168.110.34

兩者屬於同一個：

192.168.110.0/24

所以屬於同一個 LAN。

---

# 六、Default Gateway 是什麼？

我的 Default Gateway 是：

192.168.110.1

也就是 Reyee Router。

Gateway 可以理解成：

> 當我要前往其他網路時，封包預設先交給哪一台 Router。

例如：

Camera：

192.168.110.34

要連：

43.x.x.x

Camera 發現對方不在：

192.168.110.0/24

所以會把封包交給 Gateway：

192.168.110.1

概念：

Camera
192.168.110.34
        │
        ▼
Gateway
192.168.110.1
        │
        ▼
Internet
        │
        ▼
Remote Server

---

# 七、Ping 是什麼？

Windows 可以執行：

ping 192.168.110.34

測試 PC 是否可以到達 Camera。

也可以：

ping 192.168.110.1

測試 PC 到 Router。

也可以：

ping 8.8.8.8

測試到 Internet 上某個 IP 的可達性。

重要觀念：

Ping 成功

不代表：

Camera 的網站可以使用
Camera 的 RTSP 可以使用
Camera 的 API 可以使用

它主要代表 IP 層級的可達性，且對方有回應 ICMP Echo。

---

# 八、MAC Address 是什麼？

IP 是網路層使用的地址。

但是在本地 Ethernet / Wi-Fi 網路中，還會涉及：

MAC Address

我們找到 Philips Camera：

IP：

192.168.110.34

MAC：

cc:b8:5e:d6:1d:52

所以可以建立：

192.168.110.34
        ↓
cc:b8:5e:d6:1d:52

IP 與 MAC 是不同概念。

之後需要進一步學習：

ARP

了解電腦如何從 IP 找到 LAN 中對應的 MAC Address。

---

# 九、Port 是什麼？

一台設備只有 IP 還不夠。

同一台設備可以同時提供很多不同服務。

例如：

192.168.110.34:80

代表：

設備：

192.168.110.34

服務 Port：

80

可以把 IP 想成：

一棟大樓

Port 想成：

大樓裡不同的門或服務窗口。

常見例子：

22  → SSH
80  → HTTP
443 → HTTPS
554 → RTSP

所以我們之前測試：

Test-NetConnection 192.168.110.34 -Port 554

其實是在問：

> 192.168.110.34 的 TCP 554 Port 是否接受連線？

如果：

TcpTestSucceeded : False

不代表 Camera 壞掉。

只代表：

> 從目前測試位置無法成功建立到該 TCP Port 的連線。

---

# 十、TCP 與 UDP

目前先建立基本概念。

TCP：

比較重視可靠傳輸、連線狀態、順序等。

常見：

HTTP
HTTPS
SSH

UDP：

沒有 TCP 那樣的連線建立與可靠傳輸機制，較輕量。

常出現在：

DNS
影音
Discovery
IoT
即時通訊

我們分析 Camera 時同時發現 TCP 與 UDP 流量。

---

# 十一、ONVIF / WS-Discovery 實驗

我們曾經測試：

WS-Discovery / ONVIF Discovery

Python 程式送出：

239.255.255.250:3702

的 WS-Discovery Probe。

Debug 顯示：

Local interface:
192.168.110.169

Multicast:
239.255.255.250:3702

Probe 成功送出。

但是：

WS-Discovery: NOT DETECTED
ONVIF: NOT DETECTED

重要觀念：

NOT DETECTED

不等於：

一定不支援 ONVIF

只能說：

> 在目前測試條件下，沒有收到設備的 WS-Discovery 回覆。

---

# 十二、為什麼 PC 抓不到 Camera 封包？

我們曾經在 Windows PC 使用 Raw Socket 做 Passive Capture。

結果：

Idle：
0 packets

Live View：
0 packets

一開始容易誤以為：

Camera 沒有流量。

但是這是不正確的推論。

一般 Switched LAN / Wi-Fi 環境：

Camera → Router

的 Unicast 流量不一定會複製給旁邊的 PC。

所以：

PC 看不到封包

不代表：

Camera 沒有傳送封包。

這讓我理解了一個重要概念：

> Packet Capture 能看到什麼，與 Capture Point 在網路拓撲中的位置有關。

---

# 十三、為什麼 Router 比 PC 更適合抓 Camera？

Camera 要連 Internet：

Camera
   ↓
Router
   ↓
Internet

Router 本身就在 Camera 流量的路徑上。

因此我們登入：

Reyee EW1200G-PRO

使用 Router 的：

抓包診斷

產生：

.pcap

檔案。

這次真的看到了 Camera：

192.168.110.34

的網路流量。

---

# 十四、PCAP 是什麼？

PCAP 可以理解成：

> 封包擷取紀錄檔。

Router 把觀察到的 Network Packets 記錄下來。

之後可以使用：

Wireshark
Python
tshark
其他分析工具

進行離線分析。

可以研究：

Source IP
Destination IP
TCP / UDP
Source Port
Destination Port
Packet Length
Timestamp
DNS
TLS
Traffic Pattern

---

# 十五、Camera 實際觀察到的對外通訊

從 PCAP 中觀察到 Philips Camera：

192.168.110.34

會主動與 Internet 上的 Server 通訊。

曾觀察到：

TCP 80
TCP 443
UDP 32100

以及多個 Remote IP。

因此開始建立一個假設：

Philips Camera
        │
        │ Outbound Connection
        ▼
Internet / Cloud / P2P Infrastructure
        ▲
        │
    Mobile App

但：

看到某個 Remote IP / Port

不代表已經證明它一定是影片串流或 Philips Cloud。

仍需要更多實驗與證據。

---

# 十六、NAT

這次實驗第一次實際看到 NAT 的概念。

LAN 內 Camera：

192.168.110.34

經過 Reyee Router 往 WAN 送出後，在 WAN 側看到：

192.168.1.2

因此可以理解：

LAN：

Camera
192.168.110.34
        │
        ▼
Router
        │
        │ NAT
        ▼
WAN
192.168.1.2
        │
        ▼
Internet

所以：

192.168.110.34

是 LAN 裡的 Private IP。

封包經 Router 往外傳送時，Router 會進行 NAT。

---

# 十七、為什麼 Camera 沒開 443，卻看到它使用 443？

這是這次非常重要的觀念。

之前測：

PC → Camera:443

沒有成功。

這是在測：

> Camera 自己是否在 TCP 443 等待別人連入。

但是 PCAP 看到：

Camera → Internet Server:443

這是在說：

> Camera 主動連到別人的 TCP 443。

完全是不同方向。

例如：

情況 A：

PC
 ↓
Camera:443

Camera 是 Server。

情況 B：

Camera
 ↓
Cloud Server:443

Camera 是 Client。

所以：

Camera 本身沒有開 443

與：

Camera 主動使用別人的 443

完全可以同時成立。

---

# 十八、開始理解資安人員在做什麼

透過這個 Camera 實驗，我開始接觸：

Asset Discovery
Host Discovery
Service Discovery
Protocol Analysis
Traffic Analysis

例如：

先找設備：

192.168.110.34

再研究：

它有哪些 Port？

再研究：

它使用哪些 Protocol？

再研究：

它跟哪些 Server 通訊？

再研究：

Idle 與 Live View 有什麼差異？

這就是 Network / IoT Security Analysis 的基礎思考方式之一。

---

# 十九、Attack Surface

一台設備可能暴露：

SSH
HTTP
HTTPS
RTSP
ONVIF
API
其他服務

每增加一個可存取的服務，都可能增加需要管理的安全面。

因此資安分析不是單純：

「找到 Port 就攻擊。」

而是：

設備有哪些服務？
↓
這些服務是否必要？
↓
是否需要 Authentication？
↓
版本是否安全？
↓
設定是否正確？
↓
誰可以存取？
↓
是否存在異常流量？

這就是 Attack Surface 的基本概念。

---

# 二十、目前已經實際操作過的東西

我已經實際碰過：

- ipconfig
- ping
- arp
- IP Address
- MAC Address
- LAN
- Gateway
- Router
- TCP Port
- TCP Service Detection
- UDP
- Multicast
- WS-Discovery
- ONVIF Discovery
- Passive Packet Capture
- Router Packet Capture
- PCAP
- LAN / WAN
- NAT
- IoT Cloud 通訊分析

雖然很多概念還沒有完全學懂，但已經有真實操作經驗。

---

# 二十一、接下來的學習順序

下一階段不要急著研究複雜的 Camera 私有協定。

按照：

IP
↓
Subnet
↓
Gateway
↓
MAC
↓
ARP
↓
ICMP / Ping
↓
TCP / UDP
↓
Port
↓
DHCP
↓
DNS
↓
Routing
↓
NAT
↓
HTTP / HTTPS
↓
TLS
↓
Packet / PCAP
↓
Wireshark
↓
IoT Networking
↓
Network Security

逐步學習。

---

# 二十二、學會網路對程式設計有什麼幫助？

未來遇到：

API Timeout
Server 無法連線
MES 連不到 ERP
Camera 無法取得影像
Database 遠端連不上
Docker Service 無法存取
Cloud API 異常

就不會只檢查：

「是不是程式寫錯？」

而會開始分層檢查：

IP 是否正確？
↓
Network 是否可達？
↓
Gateway / Routing 是否正確？
↓
Port 是否開啟？
↓
Firewall 是否允許？
↓
DNS 是否正常？
↓
TCP 是否建立？
↓
TLS 是否成功？
↓
HTTP / API 是否正常？
↓
最後才檢查應用程式邏輯

這會大幅提高系統整合與 Troubleshooting 能力。

---

# 二十三、我的目標

我不是要單純成為 Network Engineer。

我的目標是把：

程式設計
+
Network
+
IoT
+
MES / 工廠設備
+
Data Analysis
+
AI

整合起來。

未來可以理解：

PLC
Camera
Sensor
Robot
AGV
MES
ERP
Server
Cloud

彼此到底如何透過 Network 通訊。

最終希望做到：

IoT Device
      ↓
Network
      ↓
Data Collection
      ↓
Database
      ↓
Data Analysis
      ↓
AI
      ↓
MES / Automation

---

# 今日最重要的觀念

1. IP 是設備在 IP 網路中的地址。
2. LAN 是區域網路。
3. Subnet 用來判斷哪些 IP 屬於同一網路。
4. Gateway 是前往其他網路時的預設下一站。
5. IP 與 MAC 是不同層次的地址。
6. Port 用來區分一台設備上的不同網路服務。
7. Ping 成功不代表某個應用服務一定可以使用。
8. Camera 沒開 TCP 443，不代表 Camera 不會主動連別人的 TCP 443。
9. PC 抓不到封包，不代表 Camera 沒有流量。
10. 抓包位置會影響可以觀察到哪些封包。
11. Router 位於 Camera 與 Internet 之間，因此是很好的觀察點。
12. PCAP 可以用來分析設備真正的網路行為。
13. NAT 會讓 LAN 與 WAN 看到的來源地址不同。
14. 網路資安的基礎，是先理解正常的網路如何運作。
15. 工具不是重點，理解工具背後正在做什麼才是重點。
