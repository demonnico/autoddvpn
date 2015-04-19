# 注意，以下教程需要刷ddwrt韌體，可能存在風險，請做好可能ddwrt會變磚塊的准備，如果沒把握的話，建議不要輕易嘗試，autoddvpn不對變磚負責。 #

# Introduction #

這份文件說明如何講autoddvpn寫入ddwrt韌體裡面讓每次重開機都有autoddvpn功能，這是目前autoddvpn最好的配置方式。



# Details #

首先你需要找出你的路由器使用哪一個ddwrt韌體，你可以上ddwrt的官方router database查詢下載，或者如果你的路由器剛剛刷好ddwrt的話，就可以直接拿這個韌體來使用。在接下來的過程當作，我們會:

  1. 解開ddwrt韌體
  1. 放入autoddvpn必要files
  1. 重新打包起來
  1. 用自制韌體升級ddwrt
  1. 重開機使其本身就具備autoddvpn功能


# 解開ddwrt韌體 #
准備好官方.bin韌體之後，你需要使用firmware\_mod\_kie來解開.bin, 這裡有svn下載方式
http://www.dd-wrt.com/wiki/index.php/Development
建議找一台Linux環境來操作，其他平台也許會碰到一些小麻煩。

svn取得firmware\_mod\_kie
```
mkdir firmware_mod_kit
  cd firmware_mod_kit
  svn checkout http://firmware-mod-kit.googlecode.com/svn/trunk/ firmware-mod-kit-read-only
  cd firmware-mod-kit-read-only/trunk/
```

將官方韌體解開到新的工作目錄 working\_directory/
```
$ ./extract_firmware.sh firmware.bin working_directory/
```
**如果extract\_firmware.sh解開有問題，請試試看新的extract-ng.sh and build.sh, 感謝網友提供的反饋(參考[#92](http://code.google.com/p/autoddvpn/issues/detail?id=92))**

這時 working\_directory/rootfs/裡面就是ddwrt filesystem的root(/)了, 因為ddwrt本身沒有/usr/loca/bin, 所以我們先
```
$ mkdir working_directory/rootfs/usr/local/bin
```

# 放入autoddvpn必要files #
將這幾個files放入 working\_directory/rootfs/usr/local/bin/ 下面

  1. http://autoddvpn.googlecode.com/svn/trunk/vpnup.sh
  1. http://autoddvpn.googlecode.com/svn/trunk/vpndown.sh
接著如果是搭配pptp使用請抓這個
  1. http://autoddvpn.googlecode.com/svn/trunk/bin/pptp/run.sh
或如果是搭配openvpn使用請抓這個
  1. http://autoddvpn.googlecode.com/svn/trunk/bin/openvpn/run.sh



放進去之後記得要全部chmod a+x 處理:
```
chmod a+x run.sh
chmod a+x vpnup.sh
chmod a+x vpndown.sh
```

# 重新打包起來 #
回到working\_directory之上一層, 然後下

```
$ ./build_firmware.sh output_directory/ working_directory/
```

這時就會在output\_directory/下面產生幾個韌體，其中 custom\_image-generic.bin 是 link到 custom\_image-asus.trx, 我們只要把 custom\_image-asus.trx給copy出來，改名成 .bin結尾，這時ddwrt的web界面就可以用這個.bin來升級韌體了。

# 用自制韌體升級ddwrt #
這個非常簡單，就到ddwrt的web管理界面直接指定這個新的.bin來升級即可，升級之前建議先用ddwrt web提供的backup功能將你的設置備份出來成 nvrambak.bin

注意，升級.bin韌體的時候，建議 "After flashing, reset to default settings" 見下面截屏。這樣可能會比較安全一點，但是如果同一個ddwrt版本升級的話，不reset也是可以的。

![http://autoddvpn.googlecode.com/files/FlashUpgrade.png](http://autoddvpn.googlecode.com/files/FlashUpgrade.png)

# 重開機 #
重開之後ddwrt的IP會變成192.168.1.1, 要重新設定admin密碼登入web，登入之後直接restore剛剛那個nvrambak.bin設置備份，這時所有之前的設置包括IP都會恢復，再重開一次，這時如果ssh進去看的話，就會在/usr/local/bin/下面看到autoddvpn的工具了

```
root@DD-WRT:~# ls -al /usr/local/bin/
drwxr-xr-x    2 root     root           55 Jul 18 19:40 .
drwxr-xr-x    3 root     root           30 Jul 18 19:39 ..
-rwxr-xr-x    1 root     root         1193 Jul 18 19:39 run.sh
-rwxr-xr-x    1 root     root        46059 Jul 18 19:17 vpndown.sh
-rwxr-xr-x    1 root     root        56351 Jul 18 19:17 vpnup.sh
```

# 設置rc\_startup #
這個版本開始，設置一律寫在rc\_startup, 原本如果已經寫在rc\_firewall的請清除掉。

rc\_startup只要一行即可

## /usr/local/bin/run.sh ##


或者也可以ssh進去用nvram指令來下
```
nvram set rc_firewall=''
nvram set rc_startup='/usr/local/bin/run.sh'
nvram commit
```

到這裡就搞定了，此時再最後確定一次ddwrt wan連線那邊的設置是否完成，然後就可以最後一次重開機。


# 開機之後檢查log #

重開機之後LOG如下

```
root@DD-WRT:/tmp# tail -f /tmp/autoddvpn.log 
[INFO#277] 01/Jan/1970:00:00:15 log starts
[INFO#277] 01/Jan/1970:00:00:15 modifying /tmp/pptpd_client/ip-up
[INFO#277] 01/Jan/1970:00:00:15 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#277] 01/Jan/1970:00:00:25 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#277] 01/Jan/1970:00:00:35 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#277] 01/Jan/1970:00:00:45 /tmp/pptpd_client/ip-up not exists, sleep 10sec.
[INFO#277] 01/Jan/1970:00:00:55 /tmp/pptpd_client/ip-up modified
[INFO#277] 01/Jan/1970:00:00:55 modifying /tmp/pptpd_client/ip-down
[INFO#277] 01/Jan/1970:00:00:55 /tmp/pptpd_client/ip-down modified
[INFO#277] 01/Jan/1970:00:00:55 ALL DONE. Let's wait for VPN being connected.
```

此時已經修改了 /tmp/pptpd\_client/ip-up and /tmp/pptpd\_client/ip-down
這兩個file正是chnroutes主要調整的地方。(再次感謝chnroutes的啟發）

如此一來PPTP連上或斷開就會觸發相應的vpnup.sh and vpndown.sh 於是整個autoddvpn的運作就幾乎跟chnroutes相同了，並且相當可靠，不會有之前回報的[bug#11](https://code.google.com/p/autoddvpn/issues/detail?id=1)  bug#12的問題了.

目前看起來相當不錯！