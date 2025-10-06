# 使用OpenSSL为私网IP生成自签名证书

以下是使用OpenSSL为192.168.0.1和172.16.0.1生成自签名证书的步骤：

## 步骤1：创建配置文件

首先创建一个OpenSSL配置文件，命名为`san.cnf`，包含Subject Alternative Name (SAN)扩展：

```
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
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
IP.1 = 192.168.0.1
IP.2 = 172.16.0.1
DNS.1 = localhost
```

## 步骤2：生成私钥

```bash
openssl genrsa -out server.key 2048
```

## 步骤3：生成证书签名请求(CSR)

使用配置文件生成CSR：

```bash
openssl req -new -key server.key -out server.csr -config san.cnf
```

## 步骤4：生成自签名证书

使用私钥和CSR生成自签名证书：

```bash
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt -extfile san.cnf -extensions v3_req
```

## 步骤5：验证证书

验证证书是否包含了正确的IP地址：

```bash
openssl x509 -in server.crt -text -noout
```

在输出中查找**Subject Alternative Name**部分，确认其中包含了两个IP地址：192.168.0.1和172.16.0.1。

## 完成

现在有了三个文件：
- `server.key`：私钥
- `server.csr`：证书签名请求（可以删除）
- `server.crt`：包含两个IP地址的自签名证书

这些文件可以用于配置需要SSL/TLS的服务，如Web服务器、邮件服务器等。