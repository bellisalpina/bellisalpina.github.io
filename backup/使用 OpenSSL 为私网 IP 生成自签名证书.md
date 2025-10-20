# 使用 OpenSSL 为私网 IP 生成自签名证书

以下是为内网 IP `192.168.0.1` 和 `172.16.0.1` 生成自签名证书的完整步骤。

## 步骤 1：创建配置文件

首先，创建一个名为 `san.cnf` 的 OpenSSL 配置文件。该文件的核心是定义 Subject Alternative Name (SAN) 扩展，用于将证书与指定的 IP 地址关联。

```ini
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = CN
ST = Beijing
L = Beijing
O = Company
OU = Department
CN = Internal

[v3_req]
keyUsage = keyEncipherment, digitalSignature
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
IP.1 = 192.168.0.1
IP.2 = 172.16.0.1
DNS.1 = localhost
```

## 步骤 2：生成私钥

使用 `openssl` 生成一个 2048 位的 RSA 私钥。

```bash
openssl genrsa -out server.key 2048
```

## 步骤 3：生成证书签名请求 (CSR)

基于上一步的配置文件，生成证书签名请求 (CSR)。

```bash
openssl req -new -key server.key -out server.csr -config san.cnf
```

## 步骤 4：生成自签名证书

使用私钥和 CSR，签发有效期为 365 天的自签名证书。

```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt -extfile san.cnf -extensions v3_req
```

## 步骤 5：验证证书

验证生成的证书，确认 SAN 扩展已正确包含指定的 IP 地址。

```bash
openssl x509 -in server.crt -text -noout
```

在命令输出的 **Subject Alternative Name** 部分，应该能看到如下信息：

```
X509v3 extensions:
    ...
    X509v3 Subject Alternative Name:
        IP Address:192.168.0.1, IP Address:172.16.0.1, DNS:localhost
    ...
```

## 完成

操作完成后，会生成以下三个文件：

- `server.key`：服务器的私钥文件，请妥善保管。
- `server.csr`：证书签名请求，已完成其使命，可以删除。
- `server.crt`：服务器的公钥证书文件，已包含指定的 IP 地址。

这些文件可以用于配置需要 SSL/TLS 加密的内部服务，例如 Web 服务器、API 网关等。

### 提示

由于这是自签名证书，客户端（如浏览器、操作系统）默认不信任它。在测试环境中，需要手动将 `server.crt` 安装到客户端的信任存储区，才能避免安全警告。