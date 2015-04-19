# Introduction #

這份文件說明如何快速更新graceMode的vpnup.sh腳本

# 限制 #
目前只適合graceMode+OpenVPN的運行環境

# 准備 #

  * 在DDWRT裡面建立/jffs/bin/目錄
  * 下載更新腳本到 /jffs/bin/

```
mkdir /jffs/bin/
cd /jffs/bin/
wget http://autoddvpn.googlecode.com/svn/trunk/grace.d/autoddvpn_update.sh
chmod a+x autoddvpn_update.sh
```


# CLI更新方式 #
  * 直接在DDWRT裡面執行
```
/jffs/bin/autoddvpn_update.sh
```

# WEB UI 更新方式 #
  * 在Web UI的Administration->Commands裡面輸入一行 **/jffs/bin/autoddvpn\_update.sh**
  * 點擊左下方 **Run Commands** 按鈕即可

![http://autoddvpn.googlecode.com/files/autoddvpn_update.png](http://autoddvpn.googlecode.com/files/autoddvpn_update.png)

# 返回 #
**如不需更新時返回
```
[INFO] already up-to-date
```** 更新完成時返回
```
[INFO] need update(client version: 507 is lower than server verson: 508)
[INFO] start updating
[OK] update completed
[OK] OpenVPN restarting and refreshing the routing table now.
[INFO] this may take a while to apply new rules
```
此時autoddvpn會重新套用新的vpnup.sh腳本，需要一些時間才會生效