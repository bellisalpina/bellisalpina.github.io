# Linux终端临时配置代理

在Linux中，给当前终端临时配置代理可以通过设置环境变量来实现。以下是具体的步骤：

### 临时配置代理

1. **HTTP代理**：
   如果想通过HTTP代理访问网络，可以在终端中输入以下命令：
   ```
   export http_proxy="http://127.0.0.1:1087"
   export https_proxy="https://127.0.0.1:1087"
   ```

2. **SOCKS代理**：
   如果使用的是SOCKS代理，可以使用以下命令：
   ```
   export http_proxy="socks5://127.0.0.1:1080"
   export https_proxy="socks5://127.0.0.1:1080"
   ```

3. **全部流量走代理**：
   如果希望所有流量都通过代理，可以设置`all_proxy`：
   ```
   export all_proxy="socks5://127.0.0.1:1080/"
   ```

### 测试代理是否生效

设置完成后，可以使用`curl`命令测试代理是否生效，例如：
```bash
curl -vv https://www.github.com
```
注意，`ping`命令通常无法通过代理访问外网，因为它使用的是ICMP协议，而代理主要用于HTTP和SOCKS协议的流量
