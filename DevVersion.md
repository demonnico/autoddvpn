# Introduction #

這份文件說明開發版本的autoddvpn狀況

# About #

相對於穩定版本，autoddvpn同時也提供開發版本，用來解決一些大家回報的問題，通常已經修正但尚未測試完成的功能都會放在dev版本。



> ## 使用方式 ##
rc\_firewall遠程腳本改成如下即可：

```
cd /tmp;wget http://autoddvpn.googlecode.com/svn/trunk/run-nokill-wait.sh  && /bin/sh run-nokill-wait.sh || touch failed
```

關於這個測試版本的討論請參考這個Issue
http://code.google.com/p/autoddvpn/issues/detail?id=11

## 有任何問題請[發issue給我](http://code.google.com/p/autoddvpn/issues/list) ##