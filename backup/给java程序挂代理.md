# 给java程序挂代理

+ **命令行挂Socks5代理**

启动Socks5代理命令如下
```
java -DsocksProxyHost=127.0.0.1 -DsocksProxyPort=7890 -jar xxxxx.jar
```
`socksProxyHost`是主机，`socksProxyPort`是端口号

* * * * *
+ **命令行挂HTTP代理**

如果想对一个Java程序设置HTTP代理，按如下所示设置
```
java -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=7890 -Dhttps.proxyHost=127.0.0.1 -Dhttps.proxyPort=7890 -jar xxxxx.jar
```
`http.proxyHost` 和`http.proxyPort`用于设置HTTP代理

`https.proxyHost`和 `https.proxyPort`用于设置HTTPS代理

* * * * *
+ **使用系统代理**
```
java -Djava.net.useSystemProxies=true -jar xxxxx.jar
```
那么 xxxxx.jar 程序将会走系统代理
