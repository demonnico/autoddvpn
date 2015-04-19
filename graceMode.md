# Introduction #

這份文件說明autoddvpn的graceMode運作原理。

# Details #

  * 緣起
  * 面臨的問題
  * 解決之道
  * 設置方式
  * 測試
  * 其他議題


# 緣起 #

autoddvpn最早是從[chnroutes](http://code.google.com/p/chnroutes/)的設計所啟蒙而來的，這個國內國外分流的概念後來在國內有很多人開始實做，包括tomato路由器、openwrt路由器甚至商業的威眾路由器，然而國內國外分流這個概念雖然方便，但也面臨了很多問題，autoddvpn不得不思考因應新的網路環境該如何找出更好更舒服的解決方式，graceMode就是在這樣的背景下誕生的想法。


# 面臨的問題 #

國內國外分流面臨的問題很多，例如：

  1. 不需要翻牆的國外網站也被迫走VPN了，這樣會嚴重依賴VPN的穩定性，同時浪費了VPN的流量。
  1. 所有DNS查詢為了避免被劫持，必須使用8.8.8.8 Google DNS或OpenDNS, 這樣一來跟Google DNS之間必須依賴可靠穩定的VPN連線，所有DNS UDP query封包走了很遠的路到DNS查詢使得網頁開啟變慢，萬一VPN斷線則所有查詢都沒有回應連國內網站也上不去，再來就是國內網站的CDN效果完全失效，上海電信的用戶可能會連到北京聯通的服務器，結果一切都亂了
  1. p2p無法使用，或者必須封鎖國外的p2p traffic或者server節點
  1. 國內到美西的國際帶寬越來越不穩定，夜間尖峰時段掉包率可以到20-30%, 使得VPN tunnel非常不穩定，一旦VPN不穩定，所有依賴VPN的解析或傳輸全部受到影響。

# 解決之道 #

  1. 找出真正需要翻牆的網站所使用的IP網段，只有這些IP網段才走VPN
  1. 使用本地DNS(ISP提供的DNS或local DNS), 讓DNS查詢可以非常快速穩定，同時具有CDN加速的優勢，然而對於有劫持風險的域名例如twitter, youtube, facebook等則強迫使用Google DNS, 這樣一來如果VPN不穩導致DNS解析不穩，頂多也只是影響twitter, youtube, facebook等有劫持風險的網站，其他網站完全不用擔心解析問題。(注：事實上本文最後的dnsmasq\_options設置方式完全可以避免twitter facebook等劫持風險)
  1. p2p可以正常使用，因為你的p2p節點不會在facebook, twitter, youtube這些公司的IDC機房裡面，因此所有p2p流量全部都是直連，包括國外節點，你只需要擔心你的DDWRT路由器CPU內存等硬件配備是否足以支撐即可。
  1. 盡量不依賴VPN穩定度，萬一VPN不穩頂多需要翻牆的網站連不上，其余國內網站例如douban或國外網站例如gmail必須訪問正常，不能有任何影響。

# 設置方式(以OpenVPN為例) #
  1. DDWRT必須啟動JFFS，[參考這裡](http://code.google.com/p/autoddvpn/wiki/jffs)
  1. 設置DDWRT使其可以正常連上WAN
  1. DDWRT setup裡面的 Static DNS 1~3 如果您是DHCP環境或PPPoE上網，請保持0.0.0.0，如果是static IP環境請使用本地網路環境的DNS,  不要設定Google DNS或其他境外或其他ISP的DNS。
  1. Services->DNSMasq裡面**DNSMasq Local DNS No DNS Rebind** 全部都 **Enable**
  1. Additional DNSMasq Options 請參考本文最後的備注填入。

說明：DNSMasq是DDWRT裡面的一個name cache server, 它可以定義static A RR或者定義某些域名強迫從某個DNS來做解析，上面這個設置范例是先定義好www.facebook.com www.youtube.com 等域名解析結果，對於經常上facebook, youtube and twitter的人會有很大的幫助，同時指定某些容易被DNS劫持的域名例如facebook.com fbcdn.net twitter.com youtube.com等泛域名一律由Google DNS 8.8.8.8來做解析，因為8.8.8.8之後會強迫走VPN，因此不用擔心DNS劫持，除非VPN斷線路由表被清空，這時DNSMasq才有短暫機會可能被污染。

# 接著參考[這份文件](http://code.google.com/p/autoddvpn/wiki/OpenVPNManualStartUP)，准備好你的OpenVPN設置，使其能手動啟動，設置文件放在 /jffs/openvpn/openvpn.conf ，必須注意的是，設置文件裡面不要設置 **redirect-gateway def1** , 否則會失效。

# 接著ssh進入DDWRT：
```
# mkdir /jffs/openvpn
# cd /jffs/openvpn
# wget http://autoddvpn.googlecode.com/svn/trunk/grace.d/vpnup.sh
# wget http://autoddvpn.googlecode.com/svn/trunk/grace.d/vpndown.sh
# chmod a+x *.sh
# nvram set rc_startup='date -s "2010-07-29 12:00:00"; openvpn --config /jffs/openvpn/openvpn.conf --daemon'
# nvram commit
```

**注意：如果你的OpenVPN不是手動啟動，而是透過DDWRT web UI來設置，則你需要再下載run.sh，並且使用不同的rc\_startup：**

```
# cd /jffs/openvpn/
# wget http://autoddvpn.googlecode.com/svn/trunk/openvpn/jffs/run.sh
# nvram set rc_startup='/jffs/openvpn/run.sh'
```
**建議你使用手動方式啟動OpenVPN比較好。**

最後重新啟動DDWRT即可，啟動之後會產生兩個log
  1. /tmp/autoddvpn.log 這是autoddvpn的log
  1. /tmp/openvpn.log 這是openvpn的log


# 設置方式(以PPTP為例) #

# DDWRT必須啟動JFFS，[參考這裡](http://code.google.com/p/autoddvpn/wiki/jffs)
  1. 設置DDWRT使其可以正常連上WAN
  1. DDWRT setup裡面的 Static DNS 1~3 請保持0.0.0.0 不要設定Google DNS或任何DNS，讓DDWRT預設使用ISP或DHCP提供的DNS
  1. Services->DNSMasq裡面**DNSMasq Local DNS No DNS Rebind** 全部都 **Enable**
  1. Additional DNSMasq Options 請參考本文最後的備注填入。

說明：DNSMasq是DDWRT裡面的一個name cache server, 它可以定義static A RR或者定義某些域名強迫從某個DNS來做解析，上面這個設置范例是先定義好www.facebook.com www.youtube.com 等域名解析結果，對於經常上facebook, youtube and twitter的人會有很大的幫助，同時指定某些容易被DNS劫持的域名例如facebook.com fbcdn.net twitter.com youtube.com等泛域名一律由Google DNS 8.8.8.8來做解析，因為8.8.8.8之後會強迫走VPN，因此不用擔心DNS劫持，除非VPN斷線路由表被清空，這時DNSMasq才有短暫機會可能被污染。
  1. 接著ssh進入DDWRT：
```
# mkdir /jffs/pptp
# cd /jffs/pptp
# wget http://autoddvpn.googlecode.com/svn/trunk/grace.d/vpnup.sh
# wget http://autoddvpn.googlecode.com/svn/trunk/grace.d/vpndown.sh
# wget http://autoddvpn.googlecode.com/svn/trunk/pptp/jffs/run.sh
# chmod a+x *.sh
# nvram set rc_startup='/jffs/pptp/run.sh'
# nvram commit
```

# 接著參考[這份文件](http://code.google.com/p/autoddvpn/wiki/HOWTO) **設置PPTP client** 的部分，准備好你的PPTP client設置

最後重新啟動DDWRT即可，啟動之後會產生一個log
  1. /tmp/autoddvpn.log 這是autoddvpn的log

# 測試 #

在DDWRT shell裡面 **traceroute www.facebook.com** 看看第一個節點是不是你的VPN private IP節點，也就是你的VPN gateway。同時再測試 **traceroute www.apple.com** 看看第一個節點是不是你的WAN gateway，如果都沒錯那就是成功了！


# 其他議題 #

## 如果有些網站確實被牆了，但是graceMode連不上，我該怎麼辦？ ##
graceMode裡面所有需要翻牆的IP網段定義在vpnup.sh裡面，這個腳本是我透過gfwList的內容分析自動產生而來的，但仍然有可能會有不盡全面之處，目前唯一能做的是自行定義vpnup\_custom內容，先將被牆的網站域名透過nslookup找出實際的IP，然後再用traceroute測試是否這個IP沒有走VPN，最後在DDWRT裡面手動執行
```
route add -host xxx.xxx.xxx.xxx gw <your_vpn_gateway>
```
然後再測試是否可以訪問網站，如果成功了就可以編輯vpnup\_custom文件，依據自己到情況放到/jffs/openvpn/或/jffs/pptp/下，這樣下次重開機之後就依然可以訪問這個網站了。vpnup\_custom設置范例如下：
```
# this script will be executed after loading the routing rules in vpnup.sh
# Example:
# route add -host 208.67.222.222 gw $VPNGW
# route add -net 74.125.0.0/16 gw $VPNGW

route add -host 96.16.157.15 gw $VPNGW
```

## DDWRT能如同gfwList自動更新IP網段嗎？ ##
我正在研究這個部分，不過因為DDWRT裡面沒有svn 沒有curl ，只有自帶一個非常精簡的wget command, 因此要比對SVN上面的版本號再決定是否更新vpnup.sh內容並不容易，如果大家有好的方法歡迎告訴我，目前恐怕只能手動更新了。

## 我可以不直接修改vpnup.sh而是修改自定義的文件嗎？ ##
可以的，如同前面所說的，編輯vpnup\_custom文件，依據自己到情況放到/jffs/openvpn/或/jffs/pptp/下, 裡面定義自己的route add/del內容, vpnup.sh會先載入本身的規則再載入vpnup\_custom，這樣就如同gfwList一樣可以優先以自己定義的為主了(後面定義的會override前面定義的規則)。

## 有些網站雖然不需要翻牆，但是走VPN比較快，可以強迫走VPN嗎？ ##
可以的，如同前面所說的，定義vpnup\_custom文件即可。

## 我可以放心使用P2P嗎？ ##
可以的，graceMode完全可以放心使用p2p,  所有p2p流量都不會走VPN，除非你要翻牆的網站本身使用的IP網段也跑p2p了，不過這個機會非常小。

## dnsmasq options無法涵蓋所有被污染的域名？ ##
是的，沒有人知道GFW到底污染了多少域名，如果要維護一個很大的
dnsmasq options清單是很困難的，目前也無法提供很好的自動更新機制。不過有些域名雖然被污染了，但是可能你一年卻碰不到一次。graceMode的缺點是沒辦法全面，只能透過客制化修改dnsmasq options以及vpnup\_custom來盡量趨於滿足自己的需求，但優點卻是非常明顯的。如果你非常需要解決DNS污染問題，只能在DDWRT裡面的static DNS設置好google DNS與OpenDNS, 然後如同[這份文件](http://code.google.com/p/autoddvpn/wiki/DNSMasq)最後所描述的，將國內大網站的域名指定本地ISP DNS來解析。也就是整個方式反過來思考，這樣好處是解析內容肯定不會被污染，不管GFW污染了多少域名，然而缺點就是DNS解析會非常依賴VPN穩定，如果VPN暫時斷了或不穩，可能很多不需要翻牆的網站也訪問不了了。

graceMode的設計在於彈性、客制化以及連線速度，缺點是沒辦法全面，同時沒辦法提供很好的自動更新機制，因此自己可以決定怎樣的DNS配置方式最適合自己。


## 我如何知道googlecode上面的vpnup.sh有新版本了？ ##
在自動更新機制出來之前，其實你不需要知道，只要你每日上的網站都能正常訪問，這樣你不更新vpnup.sh也沒問題，萬一有網站無法訪問而你確定是被牆了，這時候你才需要更新vpnup.sh來試試看。

## 我有其他graceMode設計上的想法或疑問，我該如何回報？ ##
請到[這裡](http://code.google.com/p/autoddvpn/issues/detail?id=23)來回報

## 我發現有某個網站的IP沒有被vpnup.sh收錄，我該如何回報？ ##
如果網址是www開頭打不開，建議您[提交到gfwList](https://autoproxy.org/~jimmy/ap/)，gfwList一旦收入之後，autoddvpn這邊會在短時間內做相應的更新。對於其他不是www開頭的域名，例如news.domain.com 這樣的域名，請直接在vpnup\_custom裡面進行設置，請勿提交上來。

# 備注 #
dnsmasq\_options最新內容請參考[這裡](http://code.google.com/p/autoddvpn/source/browse/trunk/grace.d/dnsmasq_options)