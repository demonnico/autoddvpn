# Introduction #

這份文件說明如何更進階地在autoddvpn裡面啟動OpenVPN，以獲得更多的功能支持。

# Details #

  * 手動啟動OpenVPN的目的
  * 設置方式


# 手動啟動OpenVPN的目的 #
  1. 如果你的openvpn server在啟動的時候需要指定user/pass, 也就是 auth-user-pass這樣的參數
  1. 如果需要openvpn client可以ping server，必要的時候進行自動重啟
  1. 你有多個openvpn server，需要進行remote-random的隨機連線配置
  1. 你可以設定更多openvpn的進階選項，而不必受限於DDWRT web UI所能提供的

# 設置方式 #

在/jffs/openvpn/下編輯一個 openvpn.conf ，內容范例如下

```
up 'iptables -A POSTROUTING -t nat -o tun0 -j MASQUERADE; /jffs/openvpn/vpnup.sh openvpn'
down 'iptables -D POSTROUTING -t nat -o tun0 -j MASQUERADE; /jffs/openvpn/vpndown.sh openvpn'

client                 
dev tun                      
                            
ca /jffs/openvpn/ca.crt
cert /jffs/openvpn/client.crt
key /jffs/openvpn/client.key
             
<connection>
remote <server1_ip_address> 443 udp
</connection>             
             
<connection>
remote <server2_ip_address> 53 tcp
</connection>
                     
remote-random
     
resolv-retry infinite
nobind     
float      

persist-key
persist-tun
                      
comp-lzo      
verb 3 
remote-cert-tls server

ping 10
ping-restart 60       
redirect-gateway def1
auth-user-pass /jffs/openvpn/password.txt  
log /tmp/openvpn.log
```

注意:

  * 如果你只有一個openvpn server可以用，則只需要寫一個< connection >即可，同時不需要 remote-random
  * 如果你的openvpn server不需要user authentication, 則auth-user-pass不用寫
  * ping 10 ping-restart 60表示每10秒client ping一次server, 如果60秒都沒有回應就重啟client，如果你對連線比較敏感的話，可以設為ping 5  ping-restart 20
  * 如果openvpn server沒有啟動lzo壓縮傳輸，則comp-lzo不用寫
  * 如果你是是使用[graceMode](http://code.google.com/p/autoddvpn/wiki/graceMode), 請務必把 redirect-gateway def1 這行移除，這很重要。此外建議加上 route-nopull,  這樣可以拒絕服務端push過來的路由信息，自己維護default gw。(see [#14](http://code.google.com/p/autoddvpn/issues/detail?id=14#c14))
  * 最後一行log寫入/tmp/下，每次重開機會被清空，如果你完全不需要看log，這行可以不用。
**如果你的DDWRT是svn 17xxx版本的，openvpn.conf 加上 script-security 3 system 否則會無法連上**

修改rc\_startup
```
nvram set rc_startup='date -s "2010-07-29 12:00:00"; openvpn --config /jffs/openvpn/openvpn.conf --daemon'
```
**注意：rc\_startup修改了時間是為了修正[這個issue](http://code.google.com/p/autoddvpn/issues/detail?id=14), 否則TLS handshaking會fail。**

開機時不啟動ddwrt自帶的openvpn client, 而是由rc\_startup來啟動。
```
nvram set openvpncl_enable=0
nvram commit
```

重開機之前你可以先手動測試：
```
killall openvpn
openvpn --config /jffs/openvpn/openvpn.conf --verb 5
```
這是打開verbose模式，你可以看到很多執行的output, 來判斷是否運作正常，如果沒問題就可以重開讓它自動執行了。

最後感謝 @jkgtw @yegle 的測試以及回報。