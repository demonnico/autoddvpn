# Introduction #

這裡說明如何在autoddvpn設定nexttv壹電視直連，並且指定最快的nexttv CDN節點

# Details #

  * 更新你的jffs內容
  * 更新/jffs/exroute.d/
  * 設定exroute\_list
  * 修改hosts

# 更新你的jffs內容 #
> 如果你是**jffs+pptp**, 你需要從這裡更新 run-dev.sh vpnup.sh vpndown.sh到ddwrt的**/jffs/**下
```
http://autoddvpn.googlecode.com/svn/trunk/jffs/run-dev.sh
http://autoddvpn.googlecode.com/svn/trunk/jffs/vpnup.sh
http://autoddvpn.googlecode.com/svn/trunk/jffs/vpndown.sh
```
> 如果你是**jffs+openvpn**, 你需要從這裡更新 run-dev.sh vpnup.sh vpndown.sh到ddwrt的**/jffs/openvpn/**下
```
http://autoddvpn.googlecode.com/svn/trunk/openvpn/jffs/run-dev.sh
http://autoddvpn.googlecode.com/svn/trunk/openvpn/jffs/vpnup.sh
http://autoddvpn.googlecode.com/svn/trunk/openvpn/jffs/vpndown.sh
```
**注意，如果直接wget到ddwrt當中，需要先rm掉local file再wget, 然後記得chmod +x處理**


  * 更新/jffs/exroute.d/
```
$ mkdir /jffs/exroute.d/
$ cd /jffs/exroute.d/
$ wget http://autoddvpn.googlecode.com/svn/trunk/exroute.d/nexttv
或者同一個網址底下也有flickr vimeo dropbox的file可以wget, 用這樣來全抓
$ for i in flickr vimeo dropbox nexttv; do wget http://autoddvpn.googlecode.com/svn/trunk/exroute.d/$i;done

```
  * 設定exroute\_list
```
$ nvram set exroute_enable=1
$ nvram set exroute_list='nexttv'
或
$ nvram set exroute_list='flickr vimeo dropbox nexttv'
$ nvram commit
```
最後重開機即可
```
$ reboot
```

重開機之後可以在/tmp/autoddvpn.log看到如下內容

```

[INFO#5211] 04/Aug/2010:23:50:30 preparing the exceptional routes
[INFO#5211] 04/Aug/2010:23:50:30 modifying the exceptional routes
[INFO#5211] 04/Aug/2010:23:50:30 fetching exceptional routes for flickr
[INFO#5211] 04/Aug/2010:23:50:31 adding 68.142.214.43 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:31 adding 69.147.90.159 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:31 adding 69.147.90.215 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:31 adding 67.195.19.66 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:31 adding 67.195.19.74 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:31 adding 68.142.214.24 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:31 fetching exceptional routes for dropbox
[INFO#5211] 04/Aug/2010:23:50:31 adding 174.129.27.0/24 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:32 adding 184.73.211.0/24 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:32 adding 204.236.220.0/24 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:32 fetching exceptional routes for vimeo
[INFO#5211] 04/Aug/2010:23:50:32 adding 66.235.126.128 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:32 fetching exceptional routes for nexttv
[INFO#5211] 04/Aug/2010:23:50:32 adding 210.242.234.154 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:32 adding 96.17.8.110 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:32 adding 203.69.113.0/24 via wan_gateway
[INFO#5211] 04/Aug/2010:23:50:32 modifying custom exceptional routes if available
[INFO] 04/Aug/2010:23:50:34 vpnup.sh ended
```

這樣就表示nexttv在autoddvpn裡面直連了

# 修改hosts #
最後你需要修改mac or windows的hosts

```

203.69.113.92   akamai.twtv.nextmedia.com
203.69.113.99   cp93793.live.edgefcs.net
```

# 測試 #
```
$ traceroute cp93793.live.edgefcs.net
or
$ traceroute akamai.twtv.nextmedia.com
```

注意看第二個hop是你的wan gw而不是VPN gw這樣就對了。

上海電信ADSL直連的話大約不到50ms
```
macbook:~ macbook$ ping cp93793.live.edgefcs.net
PING cp93793.live.edgefcs.net (203.69.113.99): 56 data bytes
64 bytes from 203.69.113.99: icmp_seq=0 ttl=116 time=43.021 ms
64 bytes from 203.69.113.99: icmp_seq=1 ttl=116 time=43.164 ms
64 bytes from 203.69.113.99: icmp_seq=2 ttl=116 time=43.203 ms
```

直連之後觀看nexttv就非常流程，看看截屏吧
![http://autoddvpn.googlecode.com/files/nexttv-screenshot.png](http://autoddvpn.googlecode.com/files/nexttv-screenshot.png)

## 更多關於autoddvpn自定義直連技術的細節請參考[這裡](http://code.google.com/p/autoddvpn/wiki/ExRoute) ##