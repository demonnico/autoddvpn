# 目前對p2p的支持情形 #

目前看起來ddwrt似乎無法根據ip/port來進行獨立設置，除非切VLAN，但是這樣可能會搞得更復雜，而且穩定
性堪慮，所以我比較傾向不再去針對p2p做調整，autoddvpn的目標不是發揮ddwrt的功能極致，而是讓家家
戶戶都可以直接裝上就可以穩定而且長時間翻牆，就像裝衛星電視一樣容易，而且源代碼完全公開，不會有在路由器被植入木馬的問題。

# 如果跑p2p會對VPN server壓力很大嗎？ #

這部分我持保留態度，基本上一個PPTP通道如果有太多session，可能PPTP連線就會斷了，或者PPTP tunnel本身就沒辦法承受太多的session, 所以在VPN server負擔不了之前，可能PPTP連線就先斷了，我相信即使OpenVPN也會面臨同樣的問題。

然而，一旦PPTP連線斷了，所有連往國外的連線會沒有可用的gateway, 這時如果使用原本的wan gateway的話會立刻有DNS劫持與污染的危險，因此搭配使用p2p只會讓你的ddwrt變得更不穩定，翻牆的效能也變得更差。


# 如果真的想跑p2p怎麼辦？ #

如果目前真的有很強烈的p2p需求，目前建議用兩台路由器堆疊的方式，ADSL接過來的第一台switch接上p2p主
機，而這個switch下面再放ddwrt+autoddvpn+wlan, 讓所有電腦透過這裡翻出去。不過需要再次提醒的，p2p容易造成網路loading很大，如果ddwrt到VPN server之間的packet loss或latency太高，很可能PPTP連線就會中斷，並且不斷reconnect, 這時候你的電腦就會完全癱瘓無法連往國外IP。