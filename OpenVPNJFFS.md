# how to setup autoddvpn with openvpn

# Introduction #

## 這份文件說明如何讓autoddvpn搭配OpenVPN環境使用。 ##

autoddvpn最早的設計是搭配PPTP VPN使用，然而在某些不允許PPTP穿透的網路環境可能會造成無法連上的問題，而OpenVPN的方式如果將OpenVPN server listen在TCP 443則可以滿足大部分的網路環境。

目前autoddvpn+openvpn的方式只提供JFFS方式來運行。

# Details #

  * ## 在ddwrt web界面設置openvpn client ##
  * ## 准備JFFS環境 ##
  * ## 如何放置腳本到JFFS ##
  * ## rc\_startup設置 ##
  * ## 測試 ##

# 在ddwrt web界面設置openvpn client #

基本上按照以下截屏來設置openvpn client即可，注意這個范例是將openvpn server listen在UDP 443 port, 請按照你的server具體來配置，但強烈建議你的server使用UDP or TCP 443。

**（說明：部分文件建議openvpn使用UDP會有最好的效能，但也有部分網友表示使用TCP反而更穩定，目前autoddvpn的開發環境是使用UDP 443）**

最後的 **CA Cert**, **Public Client Cert**, **Private Client Key** 這三個欄目請跟openvpn服務器管理員索取，你需要講內容當中的

-----BEGIN XXXXXX-----

-----END XXXXXX-----

連同當中的本文一起貼入，見截屏。

![http://autoddvpn.googlecode.com/files/openvpn-setup-2.png](http://autoddvpn.googlecode.com/files/openvpn-setup-2.png)

# 准備JFFS環境 #
您需要參考[這份文件](http://code.google.com/p/autoddvpn/wiki/jffs)裡面關於JFFS設置的說明來啟用JFFS支持，注意，只需要參考該文件來打開JFFS支持即可。


# 如何放置腳本到JFFS #
重啟之後ssh進入ddwrt,切換到/jffs目錄，創建/jffs/openvpn/子目錄之後下載三個files：

```
$ cd /jffs
$ mkdir /jffs/openvpn
$ cd /jffs/openvpn
$ wget http://autoddvpn.googlecode.com/svn/trunk/openvpn/jffs/run.sh
$ for i in vpnup vpndown; do wget http://autoddvpn.googlecode.com/svn/trunk/$i.sh;done;
$ chmod a+x *.sh
```

這時記得ls -l /jffs/openvpn/看一下是否檔案確實下載下來了，並且都是可執行的。


# rc\_startup設置 #
最後設置rc\_startup

```
$ nvram set rc_startup='/jffs/openvpn/run.sh'
$ nvram commit
$ reboot
```

重開機之後檢查 autoddvpn.log

```
root@DD-WRT:/tmp# tail -f /tmp/autoddvpn.log 
[INFO#357] 01/Jan/1970:00:00:17 log starts
[INFO#357] 01/Jan/1970:00:00:17 modifying /tmp/openvpncl/route-up.sh
[INFO#357] 01/Jan/1970:00:00:17 /tmp/openvpncl/route-up.sh not exists, sleep 10sec.
[INFO#357] 01/Jan/1970:00:00:28 /tmp/openvpncl/route-up.sh not exists, sleep 10sec.
[INFO#357] 28/Jul/2010:03:10:48 /tmp/openvpncl/route-up.sh modified
[INFO#357] 28/Jul/2010:03:10:48 modifying /tmp/openvpncl/route-down.sh
[INFO#357] 28/Jul/2010:03:10:48 /tmp/openvpncl/route-down.sh modified
[INFO#357] 28/Jul/2010:03:10:48 ALL DONE. Let's wait for VPN being connected.
[INFO#687] 28/Jul/2010:03:11:14 vpnup.sh started
[INFO#687] 28/Jul/2010:03:11:37 vpnup.sh ended
```

# 測試 #

這時如果你打開 http://whatismyip.org 應該會看到你的OpenVPN public IP, 表示你是透過OpenVPN訪問國外，同時打開 http://myip.cn 應該會看到國內的IP， 表示透過正常路由訪問國內，這樣就表示成功了。