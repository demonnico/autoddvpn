# Introduction #

這份文件說明如何設置 autoddvpn wget模式

# 設置rc\_firewall腳本 #

## 在DDWRT web界面進入Administration->Commands ##

在輸入框裡面輸入
```
cd /tmp;wget http://autoddvpn.googlecode.com/svn/trunk/pptp/wget/run.sh  && /bin/sh run.sh || touch failed
```
然後按下**Save Firewall**, 這個腳本會在網路連上之後進行，實際是存在DD-WRT nvram的rc\_firewall變數裡面。


# 測試 #
以上設置好之後，機器重開就搞定了！

但請注意，DD-WRT重開機之後，到PPTP VPN自動撥上，通常需要一兩分鍾甚至更久的時間，建議可以在電腦下面試著ping 8.8.8.8看看網路通不通，或是用traceroute指令對8.8.8.8查詢，通常路由表的第一個hop就是你的DD-WRT private IP, 而第二跳就是你的VPN gateway 私有IP了，如果你的第二跳不是VPN IP, 那也別擔心, autoddvpn正在修改你的路由表，可能需要30秒的等待時間。如果等了很久還是不行，請查看這個[DEBUG](http://code.google.com/p/autoddvpn/wiki/DEBUG)說明。