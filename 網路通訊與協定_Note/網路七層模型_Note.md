## 1. Ethernet(2) → IP(3) → UDP(4) → DHCP(7)。

DHCP：是一個非常重要的網路協定，它直接關係到電腦能不能上網

DHCP (動態主機配置協定)：

(1) 核心功能：自動分配網路身分證

當一個新的設備（例如您的手機、筆電或智慧家電）連接到網路時，DHCP 伺服器會自動、快速且不重複地分配一個專屬的「網路身份證」，讓這個設備能夠立即加入並使用網路。

* IP 位址 (IP Address)： 設備在網路上的邏輯地址。

* 子網路遮罩 (Subnet Mask)： 識別網路範圍。

* 預設閘道 (Default Gateway)： 告訴設備如何找到路由器（Router），才能連接到網際網路。

* DNS 伺服器地址： 告訴設備要去哪裡查詢網址（例如 www.google.com）。

(2) 為什麼屬於應用層 (Layer 7)？

* L7 是提供服務的層次： 應用層（L7）負責提供使用者或應用程式所需的網路服務。

* DHCP 就是一個服務： 它提供了一個配置服務，讓設備應用程式（例如作業系統的網路堆疊）能夠獲得運行所需的基礎參數。

* 數據交換： DHCP 封包的格式和邏輯（如 Discover, Offer, Request, ACK 等步驟）是應用程式之間交換信息的標準。數據交換： DHCP 封包的格式和邏輯（如 Discover, Offer, Request, ACK 等步驟）是應用程式之間交換信息的標準。


## 2. \SERVER\SHARE

A. FTP：(File Transfer Protocol)

* 是一種專門用於在客戶端和伺服器之間進行檔案上傳和下載的標準協定。

B. SMB：Windows 檔案分享走 SMB (Server Message Block)，屬於應用層(V)

* 是一種網路檔案系統協定，它允許電腦上的應用程式或使用者讀取、寫入檔案，並請求網路上的服務（例如列印）或連線到伺服器。


C. POP3：(Post Office Protocol version 3)

* 是一種用於從電子郵件伺服器下載郵件到本地電腦的協定。

D. HTTP：(HyperText Transfer Protocol)_超文件傳輸 (網頁)

* 是網際網路上 最重要、最基礎 的協定，它是全球資訊網 (World Wide Web) 運作的基石。

* HTTPS： 這是 HTTP 加上 SSL/TLS 加密 的版本，使其成為安全的協定。加密/解密的動作則發生在 表示層 (L6)，但協定本身仍歸類於應用層。

## 3. ping 8.8.8.8

A. 應用層
B. 傳輸層

C. 網路層：(V)

* Ping 這個動作，是網路診斷中最常用且最基礎的工具，它之所以屬於網路層，是因為它使用的核心協定是 ICMP。

* Ping 是透過發送 ICMP (Internet Control Message Protocol, 網際網路控制訊息協定) 封包來實現的。

* Ping 發送的是 ICMP Echo Request（回應請求），並等待目標主機回覆 ICMP Echo Reply（回應回覆）。

* ICMP 的主要功能是報告錯誤、傳達控制資訊，以及檢測網路連線狀態。

* 它運作在 IP 協定(網路層的核心協定)之上，與 IP 密不可分，因此 ICMP 被歸類為網路層（Layer 3）的協定

D. 資料連結層

## 4. 瀏覽器訪問一個 HTTPS 網站時，TLS 加密金鑰交換過程發生在：

A. 應用層
B. 表達層 (V)
C. 傳輸層
D. 網路層

TLS 是 SSL（Secure Sockets Layer，安全通訊端層）的後繼技術和升級版本。
雖然大家習慣上仍稱之為 SSL/TLS 憑證，但現在實際運作和使用的都是 TLS。

* 加密 (Encryption)

將傳輸中的數據打亂，使其成為亂碼。即使數據被第三方攔截，也無法讀取其真實內容。

* 驗證 (Authentication)

檢查對方的身份證件和公章。

* 完整性 (Integrity)

數據箱上貼有防偽標籤，確保內容沒有被調包。

## 5.當瀏覽器向伺服器發出請求 GET /index.html HTTP/1.1 時，這個請求屬於哪一層的資料單位？

A. Segment（分段）
B. Packet（封包）
C. Frame（訊框）
D. Message（訊息）(V)

在 OSI 模型中，每一層都有其對應的資料單元名稱：

L7 應用層
* Message (訊息) / Data (數據)	
  * HTTP, FTP, DNS	
    * (HTTP 請求)

L4 傳輸層
* Segment (分段)	
  * TCP, UDP
    * (Segment 是 L4 對 L7 數據的切割)

L3 網路層
* Packet (封包)
  * IP, ICMP	
    * (Packet 是 L3 對 Segment 的封裝)

L2 資料連結層
* Frame (訊框)
  * MAC, ARP	
    * (Frame 是 L2 對 Packet 的封裝)

## 6.若在 Wireshark 中看到 TCP Header 內有「Sequence Number」與「ACK Number」，這些欄位的用途是：

A. 確保封包不重複與順序正確 (V)

B. 用來分配 IP 位址：這是 DHCP 的職責，DHCP 屬於應用層 (L7)。

C. 驗證 MAC 位址唯一性： MAC 地址在資料連結層 (L2)決定，且唯一性由設備製造商在燒錄時保證。

D. 設定資料加密金鑰：這是 TLS 協定（屬於 L6/L7 之間）在 握手（Handshake） 階段完成的職責。

## 7. 若 DNS 查詢發生問題，導致無法解析網域名稱，這時仍可用哪一種方式連線網站？

A. 使用 IP 位址直接訪問 (V)

B. 改用 HTTP/3 傳輸：HTTP/3 只是應用層（L7）的傳輸協定版本

C. 關閉防火牆：防火牆擋住的是傳輸層（L4）或網路層（L3）的特定端口或 IP，與 L7 的 DNS 解析失敗無關。

D. 開啟 Proxy 模式：Proxy（代理伺服器）通常需要您提供域名才能進行轉發請求，如果 DNS 失敗，Proxy 也無法解析域名。

DNS (Domain Name System, 域名系統)：

* 是一個應用層（L7）的服務，它唯一的職責是充當網際網路的「翻譯官」或「電話簿」。

* 將人類容易記憶的域名 (Domain Name)（例如 www.google.com）轉換成機器可識別的 IP 位址 (IP Address)（例如 142.250.66.196）。

## 8. 下列哪一個協定主要用在「郵件傳送」而不是「郵件接收」？

A. POP3：Post Office Protocol 3 (郵局協定 3)
* 郵件接收 (Retrieving)
    * 允許使用者下載郵件到本地設備，通常會將伺服器上的副本刪除（或設定保留）。

B. IMAP：Internet Message Access Protocol (網際網路訊息存取協定)
* 郵件接收與管理
    * 允許使用者在伺服器上管理郵件（例如同步狀態、保留副本、建立資料夾）。

C. SMTP：Simple Mail Transfer Protocol (簡單郵件傳輸協定) (V)
* 郵件傳送 (Sending)
    * 負責將郵件從寄件人的客戶端發送到郵件伺服器，以及在不同郵件伺服器之間傳遞郵件。

D. DNS：Domain Name System (域名系統)
* 名稱解析
    * 將域名（例如 gmail.com）轉換為 IP 位址，與郵件的傳送或接收無關。

## 9. 在 iPerf 測試中，使用參數 -u -b 1000m 代表什麼意思？

A. 使用 UDP 傳輸、速率限制在 1000 Mbps (V)

B. 使用 TCP 傳輸、速率限制在 1000 Mbps

C. 使用 UDP 傳輸、速率自動調整

D. 使用 TCP 傳輸、速率自動調整

## 10. 若在瀏覽器開發者工具中看到：Scheme: https Protocol: h3，請問此連線的實體傳輸層使用的是哪一種協定？

A. TCP：是 HTTP/1.1 和 HTTP/2 的基礎。

B. UDP (V)

* HTTP/3 是建立在一個名為 QUIC 的全新傳輸層協定之上
    * QUIC 協定的設計目的是為了取代 TCP，以解決 TCP 存在的幾個效率問題（特別是隊頭阻塞）。
        * QUIC 的基礎： QUIC 本身是運行在 UDP 協定之上的。
        * 無序、可靠（QUIC 提供了可靠性）、無隊頭阻塞 <=> 有序、可靠、有隊頭阻塞

C. ICMP： 網路層（L3）協定，用於診斷和錯誤報告（如 Ping）。

D. ARP： 資料連結層（L2）協定，用於將 IP 地址解析為 MAC 地址。

## 11. ARP（Address Resolution Protocol）屬於哪一層？

A. 實體層

B. 資料連結層：(V)

C. 網路層

D. 傳輸層

## 12. 以下哪一項最能代表第 6 層（表達層）的功能？

A. 管理 Port 編號

B. 定義加密與壓縮格式（例如 TLS、JPEG）：(V)

C. 控制封包路由

D. 負責 IP 位址解析

## 13. 當封包傳遞過程中路由器需要查詢「目的 IP」才能決定往哪送，請問這個行為屬於哪一層的工作？

A. 資料連結層

B. 網路層：(V)

C. 傳輸層

D. 實體層

## 14. 下列哪一項不是 TCP 具備的功能？

A. 封包重傳

B. 流量控制（Flow Control）

C. 擁塞控制（Congestion Control）

D. 低延遲即時性（Low Latency）：(V) UDP 才低延遲

## 15. 如果 Wi-Fi 訊號干擾造成 CRC 錯誤，請問封包會在哪一層被丟棄？

A. 傳輸層

B. 網路層

C. 資料連結層：(V)

D. 實體層

## 16. 當你執行 tracert 8.8.8.8，看到每一跳（hop）的 IP 位址，這個功能是基於哪個協定實現的？

A. ARP： 資料連結層（L2）協定，用於將 IP 地址解析為 MAC 地址。

B. ICMP：(V)，： 網路層（L3）協定，用於診斷和錯誤報告（如 Ping）。

C. TCP

D. UDP

## 17. 假設在 Wireshark 中看到：Protocol: DNS Query: www.example.com Response: 93.184.216.34

請問 DNS 屬於哪一層？

A. 應用層：(V) DNS 是應用層協定。

B. 傳輸層

C. 網路層

D. 表達層

## 18. 若在 iPerf UDP 測試中設定 -b 值過高導致丟包率增加，這種現象主要與哪一層的行為有關？

A. 應用層

B. 傳輸層：(V) 

C. 網路層

D. 資料連結層

## 19. HTTPS 網站的安全性主要透過哪種加密方式建立初始金鑰？

A. 對稱加密：（用於實際通訊），用於傳輸大量數據

B. 非對稱加密：(V)，非對稱加密（公鑰/私鑰）

C. CRC 校驗

D. 哈希運算：用於數據完整性檢查和密碼儲存，屬於單向加密，無法用於數據解密或金鑰交換。

## 20. 在實體層中，以下哪一項屬於它的傳輸媒介？

A. IP 封包

B. 光纖 / 電纜 / 無線電波：(V) 

C. TCP Segment

D. MAC 位址