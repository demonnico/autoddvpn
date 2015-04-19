autoddvpn是從[chnroutes](http://code.google.com/p/chnroutes/)啟發而來的解決方案。它讓DD-WRT路由器可以達成類似chnroutes的效果，讓路由器下面的所有上網設備包括Windows, Mac, iPhone, iPad都可以透明翻牆，不需要任何安裝或設置。換句話說，您所需要做的，就是裝上路由器，打開電源，然後所有電腦就仿佛在牆外一般，完全感覺不到牆的存在。

### 為什麼要用這個解決方案呢？ ###
  1. 你想直接在無線路由器翻牆，只要一打開就全家翻牆，什麼都不用安裝設定，一勞永逸
  1. 你不是技術geek, 你沒時間去研究翻牆技術，你只想按下電源開關，然後統統搞定，就像喝水一樣簡單

### 我已經有SSH翻牆方案了，為什麼我還要autoddvpn? ###
這裡有一份推薦使用chnroutes取代SSH翻牆的介紹文章[請點這裡](http://docs.google.com/Doc?docid=0AXq-kIgdHknTZGhtazJzdHJfNDE2Zm5rdDRmZGQ&hl=en), autoddvpn繼承了chnroutes的設計概念，達到chnroutes同樣的效果。

## 需要環境： ##
  1. 支持PPTP或OpenVPN client的DD-WRT路由器
  1. 一個PPTP或OpenVPN帳號
  1. ADSL或DHCP上網環境
  1. 不需要安裝任何軟件

## 注意： ##
  1. 目前只支持DD-WRT
  1. 目前autoddvpn最好的配置模式是graceMode, 可以直接參考[這份文件](http://code.google.com/p/autoddvpn/wiki/graceMode)。


# [了解更多](http://code.google.com/p/autoddvpn/wiki/README) #