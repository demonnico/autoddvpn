# Introduction #

這份文件說明autoddvpn使用時，如何獲得國內CDN加速的效果。

# Details #

  * 問題背景
  * 可能的解決方式
  * 設定方法

# 問題背景 #
為了避免DNS污染，autoddvpn在ddwrt的設置上使用了google DNS, 經過VPN建立之後所有DNS查詢都會透過VPN連往google DNS, 確保DNS解析不被污染，然而這卻會造成國內一些門戶網站的CDN加速機制失效。

例如上海電信透過google DNS解析sina可能會出現北京CDN節點，於是上海電信就必須連往北京電信甚至北京網通才能打開網頁，這是一個很大的困擾。

然而這個問題不是autoddvpn的問題，只要使用google DNS或是OpenDNS的用戶全部會面臨這個問題。


# 可能的解決方式 #
  1. ddwrt 裡面如果能運行bind9將會是完美的解決方式，然而ddwrt目前裡面只運行DNSmasq這個僅能做DNS query forward給上游ISP的軟件
  1. 如果您是使用Mac，您可以將內建的bind9運行起來，並且將部分域名強迫forward給上游ISP。