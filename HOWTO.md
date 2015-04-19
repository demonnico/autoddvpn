# Introduction #

這份文件說明如何設置 autoddvpn 環境

# Details #

autoddvpn目前有三種運行模式，各有不同的優缺點，大家可以選擇適合的模式來運行：

## wget遠程腳本模式 ##
## jffs模式 ##
## custom firmware自制韌體模式 ##

# wget遠程腳本模式 #
autoddvpn開機之後，藉由rc\_firewall腳本來執行wget取得遠程腳本來運行，這是最早期autoddvpn開發的模式

**優點：只需要在ddwrt的web界面設定即可，技術門檻最低。**

**缺點：存在一些已知的問題，可能會有不穩定的情況。**

# jffs模式 #
將所需要執行的腳本放入/jffs/可寫入的filesystem裡面，並且定義rc\_startup使其一開機就能自動執行

**優點：同時具備升級的彈性以及執行的獨立性，完全不需要依賴網路就能執行腳本。**這是目前最推薦的方式

**缺點：用戶需要具備ssh以及基本linux操作經驗，每次升級都需要ssh進入ddwrt裡面操作。**

# custom firmware自制韌體模式 #
自己制作具備autoddvpn功能的DDWRT韌體，經由韌體升級之後使其韌體本身就具備autoddvpn功能

  * 優點：非常適合大批同一款路由器的升級，開發人員只要准備好.bin韌體，所有用戶只要透過web來升級韌體即可，對用戶來說是最簡單的方式。

  * 缺點：對開發者來說比較麻煩，每一次腳本升級都需要重新包裝韌體。




## 在選擇使用哪一種模式之前，請先進行下面的基本設置： ##


# 設置PPTP client #
如下圖

![http://autoddvpn.googlecode.com/files/pptp-vpn-client-config.png](http://autoddvpn.googlecode.com/files/pptp-vpn-client-config.png)

說明：

  * **PPTP主機請用IP設置， 不要設置FQDN，否則[之後會斷線](http://code.google.com/p/autoddvpn/issues/detail?id=31)**
  * MPPE Encryption裡面輸入 **mppe required,no40,no56,stateless**
  * Remote Subnet 與Remote Subnet Mask是你PPTP撥上之後的VPN子網路與遮罩，請依據自己的環境設置，必要的話可以先用電腦嘗試連上PPTP, 觀察取得的VPN IP/Netmask是多少，假如是取得192.168.199.3, 則通常設置成Remote Subnet 192.168.199.0  Remote Subnet Mask 255.255.255.0即可，以此類推。**(注意：這兩個參數不用設定也是可以連上PPTP, 但是無法調整正確的路由表，造成無法順利運作，請務必弄清楚這兩個數值，如果設定錯的話會運行失敗的)**
## 如果實在不知道怎麼找出這兩個數值，請參考[這裡的教學](http://code.google.com/p/autoddvpn/issues/detail?id=17) ##
  * Username Password是你的PPTP撥號的帳戶密碼


# 設置DNS #
DD-WRT使用dnsmasq來做簡易的name cache服務，因為dnsmasq只會forward到上游DNS以及cache查詢結果，並不會跟bind9一樣從Root DNS一路查詢下來，因此如果上游DNS資料被污染的話，dnsmasq的資料也會被污染。

這裡我們關閉了dnsmasq服務，強迫使用Google DNS與OpenDNS, 因為建立VPN之後DD-WRT跟境外DNS之間就走加密VPN了，因此不用擔心被污染的問題，請注意，DDWRT提供的三台靜態DNS全部都要設置上去，分別設置為：
```
8.8.8.8
8.8.4.4
208.67.222.222
```
注意：
# 必須三台都設置，如果設置少於三台，DDWRT會使用ISP動態發放的國內DNS來補上，這會造成風險

設定方式如下：

![http://autoddvpn.googlecode.com/files/dns-config-2.png](http://autoddvpn.googlecode.com/files/dns-config-2.png)

這樣的設置的優缺點是：

**優點**
  1. 建立VPN之後，將完全不會有DNS劫持問題

**缺點**
  1. DD-WRT下面的所有電腦的DNS都會被設置為Google DNS
  1. DD-WRT本身不做name cache, 因此所有查詢都會經由VPN到Google去，相對的會比較慢
  1. 國內一些CDN加速的網站，例如www.qq.com, 很可能Google查詢的結果並不會是最優化的結果，可能訪問會比較慢一點，實際要看每個人的感受。

## 然而可以透過[DNSMasq自定域名國內查詢](http://code.google.com/p/autoddvpn/wiki/DNSMasq)的方式來解決以上缺點，但建議第一次使用autoddvpn的用戶先不要理會這部分，等autoddvpn正常運行之後再來看這份文件。 ##


## 接著請選擇三種模式其中一種來操作即可 ##

### [wget模式](http://code.google.com/p/autoddvpn/wiki/WgetMode) ###
### [jffs模式](http://code.google.com/p/autoddvpn/wiki/jffs) ###
### [custom firmware自制韌體模式](http://code.google.com/p/autoddvpn/wiki/CustomFirmware) ###