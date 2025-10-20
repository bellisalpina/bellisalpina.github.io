# 🚀 给 Java 程序挂代理

在开发或运行 Java 应用时，有时需要通过网络代理访问外部资源。本文介绍几种常见的 Java 程序代理设置方法。

## 1. 命令行设置 Socks5 代理

通过在启动命令中添加 JVM 参数，可以快速为 Java 应用设置 Socks5 代理。
```bash
java -DsocksProxyHost=127.0.0.1 -DsocksProxyPort=7890 -jar xxxxx.jar
```
**参数说明：**
*   `socksProxyHost`: 指定代理服务器的地址。
*   `socksProxyPort`: 指定代理服务器的端口。
> 💡 **提示**：请将 `127.0.0.1` 和 `7890` 替换为你的实际代理地址和端口。

## 2. 命令行设置 HTTP/HTTPS 代理

如果需要区分 HTTP 和 HTTPS 流量，可以分别设置对应的代理。
```bash
java -Dhttp.proxyHost=127.0.0.1 -Dhttp.proxyPort=7890 -Dhttps.proxyHost=127.0.0.1 -Dhttps.proxyPort=7890 -jar xxxxx.jar
```
**参数说明：**
*   `http.proxyHost` & `http.proxyPort`: 用于设置 HTTP 协议的代理。
*   `https.proxyHost` & `https.proxyPort`: 用于设置 HTTPS 协议的代理。
> ⚠️ **注意**：对于 HTTPS 连接，JVM 仍会先通过 HTTP 代理建立隧道，因此通常需要同时设置 HTTP 和 HTTPS 代理。

## 3. 使用系统默认代理

让 Java 程序自动读取并使用操作系统中已配置的代理设置，是最便捷的方式。
```bash
java -Djava.net.useSystemProxies=true -jar xxxxx.jar
```
设置此参数后，Java 应用会尝试获取系统（如 Windows、macOS 的网络设置）中的代理配置。

## 💎 总结

下表快速对比了三种设置方法：

| 场景 | 命令示例 | 说明 |
| :--- | :--- | :--- |
| **Socks5 代理** | `-DsocksProxyHost=... -DsocksProxyPort=...` | 通用性强，适用于大多数 TCP/UDP 流量。 |
| **HTTP/HTTPS 代理** | `-Dhttp.proxyHost=... -Dhttps.proxyHost=...` | 针对性设置，适用于基于 HTTP 协议的请求。 |
| **使用系统代理** | `-Djava.net.useSystemProxies=true` | 最便捷，让程序跟随系统设置，无需手动指定地址。 |

根据你的具体网络环境和需求，选择最合适的方法即可。