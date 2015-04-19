# Introduction #

autoddvpn是從[chnroutes](http://code.google.com/p/chnroutes/)啟發而來的解決方案。它讓DD-WRT路由器可以達成類似chnroutes的效果，讓路由器下面的所有上網設備包括Windows, Mac, iPhone, iPad都可以透明翻牆，不需要任何安裝或設置。換句話說，您所需要做的，就是裝上路由器，打開電源，然後所有電腦就仿佛在牆外一般，完全感覺不到牆的存在。

# Details #

這份文件包括：

  * 如何設置autoddvpn
  * autoddvpn運作原理
  * 已知問題

# 如何設置 #

autoddvpn目前有傳統模式(classicMode)與優雅模式(graceMode)，傳統模式裡面實現了最常見的國內國外路由分離，一勞永逸。優雅模式只有需要翻牆的IP才走VPN，對於DNS劫持或P2P應用的支持最好，優雅模式是目前autoddvpn最推薦的設置方式。

## 傳統模式(classicMode) ##
如果您只有PPTP帳號，請參考[這份HOWTO文件](http://code.google.com/p/autoddvpn/wiki/HOWTO)

如果您想搭配OpenVPN帳號，請參考[這份HOWTO文件](http://code.google.com/p/autoddvpn/wiki/OpenVPNJFFS)

## 優雅模式(graceMode) ##

參考[graceMode文件](http://code.google.com/p/autoddvpn/wiki/graceMode)，這是autoddvpn最推薦的配置方式。


# 運作原理 #

_待完成_

# 已知問題 #

參考[這份文件](http://code.google.com/p/autoddvpn/wiki/KnownIssues)