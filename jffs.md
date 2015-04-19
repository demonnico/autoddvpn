# Introduction #

這份說明描述怎麼讓autoddvpn運行在ddwrt的/jffs當中，完全不需要刷韌體或是依賴網路wget抓取腳本，這是目前最推薦的方法。

# Details #

這份文件會包含以下內容：
  * jffs的好處是什麼？
  * 如何打開jffs支持
  * 如何放置腳本到jffs
  * rc\_startup設置

# jffs的好處是什麼 #
可以讓autoddvpn腳本存放在路由器的可寫空間當中，而且重開機不會消失，因此非常可靠，不需要依賴wget取得網路腳本或者改寫韌體，同時具有方便性跟穩定性。


# 如何打開jffs支持 #
在ddwrt的Administration->Management下面找jffs支持，如果有則打開

![http://autoddvpn.googlecode.com/files/jffs-1.png](http://autoddvpn.googlecode.com/files/jffs-1.png)

如果沒有的話，可能是以下原因, 你需要升級你的ddwrt：
  1. 可用空間小於4MB, 而且不是使用mini版本
  1. 使用了vpn版本
參考官方說明：http://www.dd-wrt.com/wiki/index.php/Jffs

**注意，如果你DDWRT刷mega版本，svn 13xxx系列的可能會有JFFS問題，建議刷15xxx後期或17xxx(see [bug #81](http://code.google.com/p/autoddvpn/issues/detail?id=81))**

webgui啟用jffs的方式要注意順序：

  1. 進入Administration.
  1. 下拉，找到 JFFS2 設定
  1. Enable JFFS.
  1. Click Save.
  1. 等幾秒之後再按 Apply.
  1. 再等幾秒，然後選取Clean JFFS.
  1. 這時不要按"Save", 直接按"Apply"即可
  1. (這時候router會開始准備jffs空間)
  1. 然後把 "Clean JFFS" 設置關掉。
  1. 按 "Save".
  1. 重開router一次

或者也可以ssh進去用指令打開，最後重啟一次：
```
nvram set jffs_mounted=1
nvram set enable_jffs2=1
nvram set sys_enable_jffs2=1
nvram set clean_jffs2=1
nvram set sys_clean_jffs2=1
nvram commit
reboot
```

# 如何放置腳本到jffs #
重啟之後ssh進入ddwrt,切換到/jffs目錄，下載三個files：

```
$ mkdir /jffs/pptp
$ cd /jffs/pptp
$ wget http://autoddvpn.googlecode.com/svn/trunk/pptp/jffs/run.sh
$ for i in vpnup vpndown; do wget http://autoddvpn.googlecode.com/svn/trunk/$i.sh;done;
$ chmod a+x *.sh
```

這時記得ls -l /jffs/pptp/看一下是否檔案確實下載下來了，並且都是可執行的。


# rc\_startup設置 #
最後設置rc\_startup

```
$ nvram set rc_startup='/jffs/pptp/run.sh'
$ nvram commit
$ reboot
```

重開機之後檢查 autoddvpn.log

```
root@DD-WRT:~# tail -f /tmp/autoddvpn.log 
[INFO#297] 01/Jan/1970:00:00:13 log starts
[INFO#297] 01/Jan/1970:00:00:13 modifying /tmp/pptpd_client/ip-up
[INFO#297] 01/Jan/1970:00:00:13 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#297] 19/Jul/2010:10:50:43 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#297] 19/Jul/2010:10:50:53 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#297] 19/Jul/2010:10:51:03 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#297] 19/Jul/2010:10:51:13 /tmp/pptpd_client/ip-up modified
[INFO#297] 19/Jul/2010:10:51:13 modifying /tmp/pptpd_client/ip-down
[INFO#297] 19/Jul/2010:10:51:13 /tmp/pptpd_client/ip-down modified
[INFO#297] 19/Jul/2010:10:51:13 ALL DONE. Let's wait for VPN being connected.
```
之後等pptp撥上之後會載入routing tables, 可以用route指令來檢查。

```
root@DD-WRT:~# route | tail -n 10
117.128.0.0     192.168.172.254 255.192.0.0     UG    0      0        0 vlan1
59.192.0.0      192.168.172.254 255.192.0.0     UG    0      0        0 vlan1
183.192.0.0     192.168.172.254 255.192.0.0     UG    0      0        0 vlan1
183.0.0.0       192.168.172.254 255.192.0.0     UG    0      0        0 vlan1
113.64.0.0      192.168.172.254 255.192.0.0     UG    0      0        0 vlan1
116.128.0.0     192.168.172.254 255.192.0.0     UG    0      0        0 vlan1
120.192.0.0     192.168.172.254 255.192.0.0     UG    0      0        0 vlan1
112.0.0.0       192.168.172.254 255.192.0.0     UG    0      0        0 vlan1
127.0.0.0       *               255.0.0.0       U     0      0        0 lo
default         172.28.1.1      0.0.0.0         UG    0      0        0 ppp0
```

注意最後default gateway就是VPN gateway