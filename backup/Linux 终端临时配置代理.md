# Linux 终端临时配置代理

在 Linux 终端中，可以通过设置环境变量来快速配置临时的网络代理。这种方式仅对当前终端会话有效，关闭终端后即失效。

## 设置方法

### HTTP/HTTPS 代理

适用于标准的 HTTP 代理服务器。

```bash
# 设置 HTTP 和 HTTPS 代理
export http_proxy="http://127.0.0.1:1087"
export https_proxy="http://127.0.0.1:1087" # 注意：https_proxy 通常也使用 http:// 协议头
```

### SOCKS5 代理

如果使用的是 SOCKS5 代理（如 V2Ray、Shadowsocks 的本地监听端口），则设置如下：

```bash
# 设置 SOCKS5 代理
export http_proxy="socks5://127.0.0.1:1080"
export https_proxy="socks5://127.0.0.1:1080"
```

### 代理所有协议

若希望所有协议的流量（如 FTP、Git 等）都默认通过代理，可以设置 `all_proxy`：

```bash
# 设置全局代理
export all_proxy="socks5://127.0.0.1:1080"
```

## 取消代理

要取消当前终端的代理设置，只需删除对应的环境变量即可：

```bash
unset http_proxy
unset https_proxy
unset all_proxy
```

## 验证与测试

配置完成后，可以使用 `curl` 命令来验证代理是否生效。`-vv` 参数会显示详细的连接过程，便于排查问题。

```bash
curl -vv https://www.github.com
```

> **注意**：
> `ping` 命令使用 ICMP 协议，通常不经过 HTTP/SOCKS 代理。因此，即使代理设置成功，`ping` 外网域名也可能会失败。请使用 `curl` 或 `wget` 等基于 TCP 的工具进行测试。