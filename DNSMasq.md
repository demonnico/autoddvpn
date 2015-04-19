# Introduction #

這份文件說明autoddvpn使用時，如何藉由DNSmasq的自定義域名轉向，實現DNS查詢分流，進而獲得國內CDN加速的效果。

# Details #

  * 問題背景
  * 可能的解決方式
  * 設定方法
  * 可能存在的問題


# 問題背景 #
為了避免DNS污染，autoddvpn在ddwrt的設置上使用了google DNS, 經過VPN建立之後所有DNS查詢都會透過VPN連往google DNS, 確保DNS解析不被污染，然而這卻會造成國內一些門戶網站的CDN加速機制失效。

例如上海電信透過google DNS解析sina可能會出現北京CDN節點，於是上海電信就必須連往北京電信甚至北京網通才能打開網頁，這是一個很大的困擾。同樣的問題也會發生在taobao, douban甚至alipay，或許你相對來說較少使用國內網站，但是你肯定不希望打開國內網站的等待時間比別人還要長。

這問題不是autoddvpn才有的問題，只要使用google DNS或是OpenDNS的用戶全部會面臨這個問題。

# 可能的解決方式 #
## 開啟ddwrt的DNSmasq for DNS功能，讓ddwrt裡面的DNSmasq兼做DNS forwarder and name cache server ##

這樣做的好處是：
  1. 所有域名解析透過ddwrt dnsmasq來進行，dnsmasq預設一律將dns解析forward到上游google DNS and OpenDNS, 而這裡的forward是透過VPN安全傳送的。
  1. 透過定義額外的dnsmasq參數，指定部分國內域名forward給本地ISP DNS來解析。例如上海電信用戶將sina, QQ, douban的域名forward給上海電信DNS，如此一來sina, QQ, douban等國內服務就可以享受CDN加速的效果，而不影響國外域名的安全查詢，換句話說這也是DNS查詢的國內國外分流。

# 設定方法 #

在「設置」->「基本設置」當中：
  1. 設置靜態DNS為兩台google DNS與一台OpenDNS, 注意需要三台都設置，否則DDWRT將會使用PPPoE獲得的本地ISP DNS, 這會造成DNS劫持的風險，請直接按照以下截屏來設置靜態DNS即可
  1. 「为DNS使用DNSMasq」打勾
![http://autoddvpn.googlecode.com/files/dnsmasq-1.png](http://autoddvpn.googlecode.com/files/dnsmasq-1.png)

在「服務」設置當中做如下的設置：

![http://autoddvpn.googlecode.com/files/dnsmasq-2.png](http://autoddvpn.googlecode.com/files/dnsmasq-2.png)

  1. 把「DNSMasq」與「本地DNS」都勾起來
  1. 「DNSMasq附加選項」這裡設置為以下格式：

```
server=/sina.com.cn/202.96.209.5
server=/qq.com/202.96.209.5
server=/baidu.com/202.96.209.5
server=/douban.com/202.96.209.5
```
  * 注意：/xxxx/ 當中是你希望forward給本地ISP DNS查詢的域名，後面的IP是你的ISP DNS IP， 這裡是以上海電信為例，如果你是北京網通或其他地方的電信甚至其他ISP， 請換成您ISP的DNS IP。


最後「保存設置」->「應用」即可。

如果您可以ssh/telnet進去ddwrt, 您可以在透過這個命令查看您的設置：

```
root@DD-WRT:~# nvram  get dnsmasq_options
server=/sina.com.cn/202.96.209.5
server=/qq.com/202.96.209.5
server=/baidu.com/202.96.209.5
server=/douban.com/202.96.209.5
```



# 測試 #

把ddwrt開起來，確定autoddvpn已經運行，然後查看你的電腦透過dhcp派發的DNS是否為ddwrt的IP, 如果是的話表示DNSMasq for DNS已經運行了。

從上海電信測試douban CDN解析
```
macbook:~ macbook$ nslookup img2.douban.com
Server:		192.168.1.254 <--- 這是你的ddwrt IP, 表示DNS query是透過ddwrt查詢
Address:	192.168.1.254#53

Non-authoritative answer:
img2.douban.com	canonical name = img2.douban.com.fastcdn.com.
img2.douban.com.fastcdn.com	canonical name = xnop001.fastcdn.com.
xnop001.fastcdn.com	canonical name = xnop001.globalcdn.cn.
Name:	xnop001.globalcdn.cn
Address: 222.73.115.168 <--- 得到douban在上海電信的CDN節點
```

繼續測試如果透過google DNS查詢的結果
```
macbook:~ macbook$ nslookup img2.douban.com 8.8.8.8
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
img2.douban.com	canonical name = img2.douban.com.fastcdn.com.
img2.douban.com.fastcdn.com	canonical name = xnop001.fastcdn.com.
xnop001.fastcdn.com	canonical name = xnop001.globalcdn.cn.
Name:	xnop001.globalcdn.cn
Address: 218.6.12.180 <-- 得到上海以外的節點
```

這樣就表示成功了！其他定義的域名可以比照這個方式來測試即可。


# 可能存在的問題 #
這個方式唯一存在的問題是，當ddwrt開機，DNSmasq運行起來，然而VPN尚未建立起來的時候，如果此時有twitter client或試圖連往Facebook, Youtube的DNS查詢請求對DNSMasq查詢，這時DNSMasq連往Google DNS與OpenDNS的forward request是沒有加密的，因此非常容易被DNS污染，一旦DNSMasq被污染之後所有後續的查詢都會返回污染的結果。

解決方式：

  1. 確定VPN確實建立之後再運行twitter client或瀏覽Facebook, Youtube網頁
  1. 萬一真的被污染了，請telnet/ssh進入ddwrt之後執行這個命令重啟DNSMasq服務：

```
# stopservice dnsmasq
# startservive dnsmasq
```

# for the geeks :-) #
如果你高興的話，你可以用下面這個腳本抓出全中國alexa 100強的網站，讓這些網站進行DNS本地查詢。

```
#!/bin/sh

ispdns='202.96.209.5'
for i in 0 1 2 3 4
do
   curl -s "http://www.alexa.com/topsites/countries;$i/CN" | grep "small topsites-label"  | \
   sed -e "s#.*>\([^ ]*\)<.*#server=/\1/$ispdns#g"
done
```

輸出結果
```
server=/baidu.com/202.96.209.5
server=/qq.com/202.96.209.5
server=/taobao.com/202.96.209.5
server=/sina.com.cn/202.96.209.5
server=/google.com.hk/202.96.209.5
server=/163.com/202.96.209.5
server=/google.com/202.96.209.5 <--但記得刪除google.com, 否則會被污染
server=/soso.com/202.96.209.5
server=/sohu.com/202.96.209.5
server=/youku.com/202.96.209.5
server=/sogou.com/202.96.209.5
server=/tudou.com/202.96.209.5
server=/hao123.com/202.96.209.5
server=/google.cn/202.96.209.5
server=/yahoo.com/202.96.209.5
server=/ifeng.com/202.96.209.5
server=/tianya.cn/202.96.209.5
server=/renren.com/202.96.209.5
server=/kaixin001.com/202.96.209.5
server=/cnzz.com/202.96.209.5
....省略...
```

或者你可以運行上面腳本，但最後輸出過濾掉 ".cn"字符，也就是top 100但是沒有.cn字符的(大多是.com), 最前面或最後面再加一行
```
server=/cn/202.96.209.5
```

如此一來**(1)top 100中國.com網站**加上**(2)所有.cn網站**，一律本地ISP DNS查詢，也許這樣更好。

除此之外有些網站不在top 100，但是你偶爾會用到，也建議你加入的包括
```
ctrip.com
homeinns.com
```

我目前的設置：
```
root@DD-WRT:~# nvram get dnsmasq_options
address=/www.facebook.com/66.220.149.25 
address=/www.youtube.com/72.14.213.190
server=/cn/202.96.209.5
server=/baidu.com/202.96.209.5
server=/qq.com/202.96.209.5
server=/taobao.com/202.96.209.5
server=/google.com.hk/202.96.209.5
server=/163.com/202.96.209.5
server=/soso.com/202.96.209.5
server=/sohu.com/202.96.209.5
server=/youku.com/202.96.209.5
server=/sogou.com/202.96.209.5
server=/tudou.com/202.96.209.5
server=/hao123.com/202.96.209.5
server=/yahoo.com/202.96.209.5
server=/ifeng.com/202.96.209.5
server=/renren.com/202.96.209.5
server=/kaixin001.com/202.96.209.5
server=/ku6.com/202.96.209.5
server=/xunlei.com/202.96.209.5
server=/chinaz.com/202.96.209.5
server=/alibaba.com/202.96.209.5
server=/alipay.com/202.96.209.5
server=/live.com/202.96.209.5
server=/douban.com/202.96.209.5
server=/56.com/202.96.209.5
server=/xinhuanet.com/202.96.209.5
server=/gougou.com/202.96.209.5
server=/soufun.com/202.96.209.5
server=/mop.com/202.96.209.5
server=/4399.com/202.96.209.5
server=/youdao.com/202.96.209.5
server=/51.la/202.96.209.5
server=/126.com/202.96.209.5
server=/58.com/202.96.209.5
server=/360buy.com/202.96.209.5
server=/soku.com/202.96.209.5
server=/csdn.net/202.96.209.5
server=/2345.com/202.96.209.5
server=/tom.com/202.96.209.5
server=/51.com/202.96.209.5
server=/yesky.com/202.96.209.5
server=/ganji.com/202.96.209.5
server=/it168.com/202.96.209.5
server=/ynet.com/202.96.209.5
server=/39.net/202.96.209.5
server=/51job.com/202.96.209.5
server=/pchome.net/202.96.209.5
server=/onetad.com/202.96.209.5
server=/microsoft.com/202.96.209.5
server=/eastmoney.com/202.96.209.5
server=/hudong.com/202.96.209.5
server=/doubleclick.com/202.96.209.5
server=/1133.cc/202.96.209.5
server=/alimama.com/202.96.209.5
server=/paipai.com/202.96.209.5
server=/china.com/202.96.209.5
server=/msn.com/202.96.209.5
server=/admin5.com/202.96.209.5
server=/7k7k.com/202.96.209.5
server=/verycd.com/202.96.209.5
server=/qidian.com/202.96.209.5
server=/pcpop.com/202.96.209.5
server=/bing.com/202.96.209.5
server=/dangdang.com/202.96.209.5
server=/zhaopin.com/202.96.209.5
server=/huanqiu.com/202.96.209.5
server=/duowan.com/202.96.209.5
server=/ppstream.com/202.96.209.5
server=/sougames.com/202.96.209.5
server=/xici.net/202.96.209.5
server=/onlinedown.net/202.96.209.5
server=/88db.com/202.96.209.5
server=/pptv.com/202.96.209.5
server=/naqigs.com/202.96.209.5
server=/mozilla.com/202.96.209.5
server=/dianping.com/202.96.209.5
server=/hc360.com/202.96.209.5
server=/360doc.com/202.96.209.5
server=/ctrip.com/202.96.209.5
server=/homeinns.com/202.96.209.5
```

Reference: [Man Page of DNSMASQ](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html)