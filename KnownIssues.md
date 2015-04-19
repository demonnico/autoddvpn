# Introduction #

這個頁面說明autoddvpn目前的已知問題還有缺點

# 國內某些網站可能無法獲得最好的CDN加速 #
因為autoddvpn強迫使用VPN來查詢google DNS, 因此一些國內網站例如SINA, QQ, Douban等透過google DNS解析出來的結果往往不是最好的節點。例如您在上海的話很可能解析出來的節點是在北京，這是使用google DNS或OpenDNS都會碰到的CDN問題，但倘若您使用國內服務不多的話也許這個問題相對來說就不重要了。

如果要解決這個問題，可以使用ddwrt上的DNSMasq搭配自定義域名本地查詢的方式來實現。
## 詳見[這份文件](http://code.google.com/p/autoddvpn/wiki/DNSMasq)。 ##

# 搭配P2P大量使用的時候會造成VPN斷線 #

其實任何沒有適當分配帶寬的P2P應用都有可能會造成網路的不穩定。實際測試發現，如果跑P2P但是路由器沒做好QoS或是適當的限速，這樣會造成網路掉包，最後PPTP反復斷線又重連，這會造成很不穩定的情況。所以目前建議如果要使用P2P應用的話不要火力全開，以免造成VPN不穩定。

您可以在使用P2P的同時一直ping 8.8.8.8, 如果出現經常性的掉包就表示品質已經很不穩定了。

好的VPN品質大約持續維持在200-300ms左右，並且不掉包

```
macbook:~ macbook$ ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: icmp_seq=0 ttl=55 time=240.326 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=55 time=198.588 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=55 time=197.643 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=55 time=231.719 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=55 time=203.183 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=55 time=237.717 ms
64 bytes from 8.8.8.8: icmp_seq=6 ttl=55 time=211.087 ms
64 bytes from 8.8.8.8: icmp_seq=7 ttl=55 time=205.846 ms
64 bytes from 8.8.8.8: icmp_seq=8 ttl=55 time=204.162 ms
64 bytes from 8.8.8.8: icmp_seq=9 ttl=55 time=200.667 ms
64 bytes from 8.8.8.8: icmp_seq=10 ttl=55 time=206.977 ms
64 bytes from 8.8.8.8: icmp_seq=11 ttl=55 time=199.268 ms
64 bytes from 8.8.8.8: icmp_seq=12 ttl=55 time=322.939 ms
64 bytes from 8.8.8.8: icmp_seq=13 ttl=55 time=221.255 ms
64 bytes from 8.8.8.8: icmp_seq=14 ttl=55 time=199.690 ms
64 bytes from 8.8.8.8: icmp_seq=15 ttl=55 time=213.775 ms
64 bytes from 8.8.8.8: icmp_seq=16 ttl=55 time=203.666 ms
```

## 更多autoddvpn與p2p所有相關的議題，請看[這裡](http://code.google.com/p/autoddvpn/wiki/AboutP2P)。 ##

# 使用Mac OS X iChat可能無法遠程桌面 #
估計是連往apple服務器是走VPN，而國內用戶則直連，在IP信息交換的時候會出現不一致，這應該是一個bug，建議可以用TeamViewer軟件取代iChat遠程桌面。

# 使用wget模式的時候，有可能wget會卡住 #
ddwrt裡面的wget是使用busybox的精簡版wget，經過測試在某些情況wget會卡住，因為wget無法設置--timeout因此只能重開機。這部分目前的追蹤情況詳見[這裡](http://code.google.com/p/autoddvpn/issues/detail?id=16), 如果您有jffs的話建議使用jffs+pptp or jffs+openvpn, 會比wget模式穩定。

# pptp in pptp 的問題 #
如果您已經部署了pptp模式的autoddvpn, 然而ddwrt下面的電腦還需要pptp連線到國外網站，這樣可能會有pptp in pptp的問題，有可能會連不上的。解決方式是將此VPN IP設置在自定義直連的exroute\_custom參數裡面即可解決。關於自定義直連詳見[這裡](http://code.google.com/p/autoddvpn/wiki/ExRoute)