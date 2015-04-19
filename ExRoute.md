# Introduction #

這份文件說明如何在autoddvpn環境裡面設定自定義直連網段。

# Details #

  * 為什麼要自定義直連
  * 如何實現

# 為什麼要自定義直連 #

autoddvpn繼承chnroutes的設計，實現國內國外路由的分離，凡是連往國內一律使用wan gateway，而連往所有國外IP則一律使用VPN gateway，這樣的設計可以免除很多麻煩，也不需要維護復雜的翻牆名單，然而有些國外網站是你經常連上，而你很肯定這網站不需要翻牆，並且直連效果會更好的。像這種情況就需要自定義直連。

因為每一個網站在每個環境可能連線速度感受不同，同樣是Flickr也許網通用戶直連反而慢，但電信用戶直連就會更好，所以autoddvpn設計了這樣的架構，讓大家可以自定義需要直連的網段，而不是強迫大家一起直連。

# 如何實現 #
  1. 使用exroute\_list 來指定預先定義好的exroute profile
  1. 使用exroute\_custom 來指定自定義網段或單個IP

## 使用exroute\_list 來指定預先定義好的exroute profile ##
目前autoddvpn有提供的exroute profile有：

  1. flickr
  1. vimeo
  1. dropbox
  1. nexttv
  1. gmail
  1. gtalk

詳見[這裡的列表](http://code.google.com/p/autoddvpn/source/browse/#svn/trunk/exroute.d)

這些網站我們都找出了他們的IP網段，定義在autoddvpn的svn裡面， 這樣大家可以共享方便的設定，不需要每一個人都去查找到底什麼網站使用什麼網段。然而每個人的需求不同，有些人可能完全不看vimeo or flickr, 有些人也甚至完全不用dropbox的，因此這些預先定義的alias可以依據實際的需要來設定使用，一旦設定了之後，exroute profile背後定義的網段就會直連，速度往往比透過VPN還快，甚至可以節省VPN流量。

倘若需要這些網站直連，只需要
```
nvram set exroute_enable=1
nvram set exroute_list='dropbox flickr gmail gtalk msn nexttv vimeo'
nvram commit
```

這樣即可。

倘若您有新的需求，例如需要定義所有apple.com使用的IP直連，這部分就必須自己想辦法來找出apple使用的網段了。這部分一般存在比較不可掌控的因素，因為很多網站往往使用了CDN架構，IP會依據DNS查照的來源不同解析不同的IP網段，因此我們不把這樣網站的alias定義在SVN當中，我們建議使用接下來介紹的exroute\_custom方式來設置。

## 使用exroute\_custom 來指定自定義網段或單個IP ##

此外也可以自己定義IP網段，定義exroute\_custom， 這裡通常是autoddvpn沒有預先定義的網段，依據個人需要可以自己來指定。autoddvpn會在ddwrt開機之後透過nvram get來獲得exroute\_custom定義的網段，並且動態地把這些網段設定成直連。

設定方式如下：

```
nvram set exroute_enable=1
nvram set exroute_custom="68.142.192.0/18 69.147.64.0/18 67.195.0.0/16 174.36.30.70"
nvram commit
```

**注意：exroute\_custom這裡設置的內容可以是一個網段或是單個IP， 注意使用單個IP的時候不需要加上netmask, 例如只需要174.36.30.70即可，不需要174.36.30.70/32， 而整個class C則是 174.36.30.0/24**

詳見[這裡](http://code.google.com/p/autoddvpn/issues/detail?id=7)的說明：

## 已經定義exroute\_custom之後如果還要新增IP ##
如果之前已經定義過 exroute\_custom了，發現還需要新增的話，例如需要加60.60.60.60這單個IP

```
nvram set exroute_custom="$(nvram get exroute_custom) 60.60.60.60" 
nvram commit
```
這樣即可，不放心的話可以檢查一下：
```
nvram get exroute_custom
```
這樣會列出所有自定義直連的網段

# JFFS模式如何使用自定義直連 #
必須把exroute profile從svn wget抓下 /jffs/exroute.d/目錄底下

```
$ mkdir /jffs/exroute.d/
$ cd /jffs/exroute.d/
$ for i in dropbox flickr gmail gtalk msn nexttv vimeo; do wget http://autoddvpn.googlecode.com/svn/trunk/exroute.d/$i;done
```

這樣JFFS模式每次開機的時候，vpnup.sh就會根據exroute\_list裡面定義的內容來查找 /jffs/exroute.d/裡面相應的exroute profile，啟動之後log范例如下：

```
root@DD-WRT:/tmp# tail -f autoddvpn.log 
[INFO#304] 01/Jan/1970:00:00:12 log starts
[INFO#304] 01/Jan/1970:00:00:12 modifying /tmp/openvpncl/route-up.sh
[INFO#304] 01/Jan/1970:00:00:13 /tmp/openvpncl/route-up.sh not exists, sleep 10sec.
[INFO#304] 28/Jul/2010:07:20:44 /tmp/openvpncl/route-up.sh not exists, sleep 10sec.
[INFO#304] 28/Jul/2010:07:20:54 /tmp/openvpncl/route-up.sh modified
[INFO#304] 28/Jul/2010:07:20:54 modifying /tmp/openvpncl/route-down.sh
[INFO#304] 28/Jul/2010:07:20:54 /tmp/openvpncl/route-down.sh modified
[INFO#304] 28/Jul/2010:07:20:54 ALL DONE. Let's wait for VPN being connected.
[INFO#696] 28/Jul/2010:07:21:04 vpnup.sh started
[INFO#696] 28/Jul/2010:07:21:25 preparing the exceptional routes
[INFO#696] 28/Jul/2010:07:21:25 modifying the exceptional routes
[INFO#696] 28/Jul/2010:07:21:25 fetching exceptional routes for flickr
[INFO#696] 28/Jul/2010:07:21:25 adding 68.142.214.43 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:25 adding 69.147.90.159 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:25 adding 69.147.90.215 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:25 adding 67.195.19.66 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:25 adding 67.195.19.74 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:25 fetching exceptional routes for dropbox
[INFO#696] 28/Jul/2010:07:21:26 adding 174.129.27.0/24 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:26 adding 184.73.211.0/24 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:26 adding 204.236.220.0/24 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:26 fetching exceptional routes for vimeo
[INFO#696] 28/Jul/2010:07:21:26 adding 66.235.126.128 via wan_gateway
[INFO#696] 28/Jul/2010:07:21:27 modifying custom exceptional routes if available
[INFO#696] 28/Jul/2010:07:21:29 vpnup.sh ended
```


# JFFS模式如何更新定義好的exroute profile #
例如nexttv這個profile已經有設定了，但是profile在SVN上面做了修改，需要更新：

```
$ cd /jffs/exroute.d/
$ rm nexttv  注意一定要先rm掉
$ wget http://autoddvpn.googlecode.com/svn/trunk/exroute.d/nexttv
```

然後再重開機ddwrt即可，這樣新的nexttv定義就會生效

## JFFS模式目前不管pptp or openvpn都支持自定義直連 ##


# wget模式如何使用自定義直連 #
wget模式因為不提供/jffs/寫入功能，因此所有內容都是開機之後動態抓取處理的。目前wget模式同樣也支持exroute功能，同樣也只需要

```
nvram set exroute_enable=1
nvram set exroute_list='dropbox flickr gmail gtalk msn nexttv vimeo'
nvram commit
reboot
```

這樣重開機之後，可以觀察/tmp/autoddvpn.log就會看到exroute功能生效，所有名單動態從svn抓取存入/tmp/exroute.d/下，並且進行路由直連。

log如下
```

[INFO#4561] 08/Aug/2010:11:53:51 vpnup.sh started
[INFO#4561] 08/Aug/2010:11:53:56 preparing the exceptional routes
[INFO#4561] 08/Aug/2010:11:53:56 modifying the exceptional routes
[INFO#4561] 08/Aug/2010:11:53:56 fetching exceptional routes for flickr
[INFO#4561] 08/Aug/2010:11:53:56 adding 68.142.214.43 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:56 adding 69.147.90.159 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:56 adding 69.147.90.215 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:56 adding 67.195.19.66 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:56 adding 67.195.19.74 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:56 adding 68.142.214.24 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:56 fetching exceptional routes for vimeo
[INFO#4561] 08/Aug/2010:11:53:56 adding 66.235.126.128 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:56 fetching exceptional routes for nexttv
[INFO#4561] 08/Aug/2010:11:53:57 adding 210.242.234.154 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:57 adding 96.17.8.110 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:57 adding 203.69.113.0/24 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:57 fetching exceptional routes for dropbox
[INFO#4561] 08/Aug/2010:11:53:57 adding 174.129.27.0/24 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:57 adding 184.73.211.0/24 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:57 adding 204.236.220.0/24 via wan_gateway
[INFO#4561] 08/Aug/2010:11:53:57 modifying custom exceptional routes if available
[INFO#4561] 08/Aug/2010:11:53:57 vpnup.sh ended
```