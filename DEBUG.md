# Introduction #

如何debug系統autoddvpn的運作狀態？

# 注意 #

  * DD-WRT重開機之後，通常需要1-2分鍾連上PPTP VPN, 實際連上去的速度視情況而定，所以別急著重開機就馬上上網。
  * 重開機完成1-2分鍾後，您可以在電腦測試打開 http://whatismyip.org 這時您看到的IP應該是您VPN gateway的public IP, 而打開 http://myip.cn 則是看到國內IP， 這兩個IP如果不一樣就成功了
  * 如果兩個IP始終一樣，你可以查看autoddvpn.log, 方法如下：


# 如何查看log #
您需要ssh進入DD-WRT， 並且查看/tmp/autoddvpn.log， pptp+wget為例子內容如下：

```
[INFO#445] 01/Jan/1970:00:00:26 log starts
[INFO#445] 01/Jan/1970:00:00:26 getting vpnup.sh <--網路取得vpnup.sh
[INFO#445] 01/Jan/1970:00:00:41 getting vpndown.sh <--網路取得vpndown.sh
[INFO#445] 01/Jan/1970:00:00:41 modifying /tmp/pptpd_client/ip-up
[INFO#445] 01/Jan/1970:00:00:42 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#445] 01/Jan/1970:00:00:52 /tmp/pptpd_client/ip-up modified
[INFO#445] 01/Jan/1970:00:00:52 modifying /tmp/pptpd_client/ip-down
[INFO#445] 01/Jan/1970:00:00:52 /tmp/pptpd_client/ip-down modified
[INFO#445] 01/Jan/1970:00:00:52 ALL DONE. Let's wait for VPN being connected.  <-- 前置准備動作完成，等待pptp撥號成功
[INFO#722] 06/Aug/2010:10:21:04 vpnup.sh started <-- 撥號成功了，開始進行vpnup.sh動作
[INFO#722] 06/Aug/2010:10:21:25 preparing the exceptional routes
[INFO#722] 06/Aug/2010:10:21:25 modifying the exceptional routes
[INFO#722] 06/Aug/2010:10:21:25 fetching exceptional routes for flickr
[INFO#722] 06/Aug/2010:10:21:25 adding 68.142.214.43 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 adding 69.147.90.159 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 adding 69.147.90.215 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 adding 67.195.19.66 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 adding 67.195.19.74 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 adding 68.142.214.24 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 fetching exceptional routes for dropbox
[INFO#722] 06/Aug/2010:10:21:26 adding 174.129.27.0/24 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 adding 184.73.211.0/24 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 adding 204.236.220.0/24 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 fetching exceptional routes for vimeo
[INFO#722] 06/Aug/2010:10:21:26 adding 66.235.126.128 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:26 fetching exceptional routes for nexttv
[INFO#722] 06/Aug/2010:10:21:26 adding 210.242.234.154 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:27 adding 96.17.8.110 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:27 adding 203.69.113.0/24 via wan_gateway
[INFO#722] 06/Aug/2010:10:21:27 modifying custom exceptional routes if available
[INFO] 06/Aug/2010:10:21:28 vpnup.sh ended  <--- vpnup.sh完成，到這裡應該基本OK了

```

或者也可以在DD-WRT的web管理界面的命令輸入框打 **cat /tmp/autoddvpn.log** 然後按下Run Command, 也會傳回LOG記錄。

# log看起來正常但是還是無法翻牆？ #
進入DDWRT下這個指令

```
root@DD-WRT:~# route -n | tail -n 20
112.224.0.0     192.168.172.254 255.224.0.0     UG    0      0        0 vlan2
122.64.0.0      192.168.172.254 255.224.0.0     UG    0      0        0 vlan2
110.192.0.0     192.168.172.254 255.224.0.0     UG    0      0        0 vlan2
60.0.0.0        192.168.172.254 255.224.0.0     UG    0      0        0 vlan2
58.32.0.0       192.168.172.254 255.224.0.0     UG    0      0        0 vlan2
182.96.0.0      192.168.172.254 255.224.0.0     UG    0      0        0 vlan2
180.96.0.0      192.168.172.254 255.224.0.0     UG    0      0        0 vlan2
61.128.0.0      192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
111.0.0.0       192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
223.64.0.0      192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
59.192.0.0      192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
117.128.0.0     192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
183.192.0.0     192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
183.0.0.0       192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
113.64.0.0      192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
120.192.0.0     192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
116.128.0.0     192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
112.0.0.0       192.168.172.254 255.192.0.0     UG    0      0        0 vlan2
127.0.0.0       0.0.0.0         255.0.0.0       U     0      0        0 lo
0.0.0.0         192.168.199.1   0.0.0.0         UG    0      0        0 ppp0
```

注意最後一行，這是你的default GW也就是VPN GW， 上面這個例子VPN GW是192.168.199.1
你可以直接ping這個IP， 正常情況下應該要有穩定持續不掉包的返回，如果沒有返回或是有掉包情況，這樣會導致你的VPN連線很不穩定，這通常可能是你的ISP到境外VPN主機網路不穩定造成的。

# ping VPN GW返回很正常但是還是無法翻牆 #
  1. 請確定 http://whatismyip.org 看到的的確是國外IP
  1. 可以ping到VPN GW
  1. 在Windows or Mac下使用traceroute or tracert指令對8.8.8.8查詢，例如

```
macbook-2:~ macbook$ traceroute -n 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 64 hops max, 52 byte packets
 1  10.0.1.1  5.454 ms  0.737 ms  3.562 ms
 2  192.168.199.1  401.621 ms  395.949 ms  405.031 ms
 3  74.82.169.65  412.484 ms  395.876 ms  402.661 ms
 4  172.18.0.65  394.927 ms  411.505 ms  391.229 ms
 5  172.18.201.1  388.315 ms  400.306 ms  395.004 ms
...
```
注意第二跳應該是你的VPN GW IP。

4. 如果1-3都沒問題了，但是某些網站例如Facebook Youtube Twitter還是打不開，表示你很可能被DNS污染了，
你需要先關閉所有瀏覽器，然後：

在Mac下用
```
$ dscacheutil -flushcache
```
來清除cache
或Windows下用
```
$ ipconfig /flushdns
```
來清除被污染的DNS cache即可。


# 以上都無法解決 #

如果以上trouble-shooting都無法幫助你，請到[發Issues給我](http://code.google.com/p/autoddvpn/issues/list)， 方便我追蹤。

如果你最後搞定了，請在twitter上告訴我 @autoddvpn 。