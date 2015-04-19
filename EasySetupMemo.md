# 簡易setup #

  1. 接上路由器， WAN port接上 ADSL貓, 開機
  1. 電腦用有線網路，連上 http://10.0.1.1 管理界面，admin/admin登入
  1. 左上角setup的部分，聯網方式選PPPoE, 設定好ADSL帳戶密碼，SAVE
  1. 必要的話可以修改無線網路的ESSID以及密碼，SAVE
  1. 最後按Apply, 然後五秒鍾之後重起路由器或是電源拔掉五秒鍾再接上

兩分鍾之後，電腦不要開啟任何Facebook or Youtube網頁，請先打開

http://myip.cn 看看是否能連上國內網路，這網站會告訴你的國內公網IP以及ISP

如果可以的話再打開

http://whatismyip.org/ 如果看到IP是 **64.71.141.150** 就表示成功了，如果不是這個IP，仍是上面那個國內公網IP的話，表示VPN還在撥號，再稍後一下，可以繼續刷當前這個網頁，直到看到 **64.71.141.150** 即可。

## 注意，在還沒看到 **64.71.141.150** 之前如果嘗試打開牆外的網站例如youtube or facebook等會造成DNS被污染，這樣即使翻牆也無法連上網站。 ##

## Windows下清除DNS污染方式： ##
  1. 關閉所有瀏覽器
  1. 開始->執行->cmd 然後下 **ipconfig /flushdns** 再enter送出即可清除DNS cache


## Mac下清除DNS污染方式： ##
  1. 關閉所有瀏覽器
  1. 打開終端機視窗，輸入 **dscacheutil -flushcache** 再enter送出即可