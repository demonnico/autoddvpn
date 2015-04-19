# Introduction #

這份文件說明如何不遠程執行run.sh而是運行local的腳本達成autoddvpn效果, 您需要操作基本的SSH指令來完成部分工作，這份文件適合能夠SSH登入DD-WRT的進階使用者。

# 背景 #

autoddvpn預設是運行遠程的run.sh腳本達成翻牆的效果，如果你有以下這些考量：

  1. 擔心遠程腳本可能有安全之虞
  1. 擔心googlecode有一天可能會連不上
  1. 希望更可靠地運行翻牆腳本

那麼你可以使用本地運行的方式。但對一般情況來說，遠程運行大多可以滿足使用需求。

# 條件 #

  1. 你的DD-WRT需要支持USB設備, 並且mount到/jffs成可讀寫

# 操作方式 #

  1. 請先詳讀遠程運行的[HOWTO](http://code.google.com/p/autoddvpn/wiki/HOWTO)，了解所有的設定方式，基本上所有設定都跟遠程腳本的方式相同。
  1. 確定您已經開啟USB功能，並且掛載到 /jffs, 參考下面圖示：

![http://autoddvpn.googlecode.com/files/usb-1.png](http://autoddvpn.googlecode.com/files/usb-1.png)
![http://autoddvpn.googlecode.com/files/usb-2.png](http://autoddvpn.googlecode.com/files/usb-2.png)
  1. 接著SSH進入系統，將以下兩個文檔wget放置在/jffs/下
```
$ cd /jffs/
$ wget http://autoddvpn.googlecode.com/svn/trunk/runlocal.sh
$ wget http://autoddvpn.googlecode.com/svn/trunk/vpnup.sh
$ chmod a+x runlocal.sh
$ chmod a+x vpnup.sh
```
  1. 接著編輯rc\_firewall指令，輸入一行
```
/jffs/runlocal.sh --local
```
  1. 重開機之後應該就可以了