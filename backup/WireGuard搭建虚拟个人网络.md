# WireGuard搭建虚拟个人网络

### 安装WireGuard
```
# 安装wireguard
apt install wireguard-tools -y

# 开启流量转发
echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | tee -a /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
```

### 进入配置存储路径，调整目录权限
```
chmod 0777 /etc/wireguard
cd /etc/wireguard
```

### 生成公钥私钥
```
wg genkey | tee server.key | wg pubkey > server.key.pub
```

### 生成client1公钥私钥
```
wg genkey | tee client1.key | wg pubkey > client1.key.pub
```

### 创建服务器配置文件
```
echo "
[Interface]
PrivateKey = $(cat server.key)
Address = 10.0.0.1/24
ListenPort = 50820
DNS = 119.29.29.29

PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
#注意eth0需要为本机网卡名称

[Peer]
PublicKey = $(cat client1.key.pub)
AllowedIPs = 10.0.0.2/32" > wg0.conf
```

### 增加client2
```
# 生成公钥私钥
wg genkey | tee client2.key | wg pubkey > client2.key.pub

# 追加到wg0.conf配置
echo "
[Peer]
PublicKey =  $(cat client2.key.pub)
AllowedIPs = 10.0.0.3/32" >> wg0.conf
```

### 设置WireGuard服务自启
```
systemctl enable wg-quick@wg0
```

### 启动WireGuard
```
# 启动wg0
wg-quick up wg0
# 关闭wg0
wg-quick down wg0
```

### 创建client1配置
```
echo "
[Interface]
PrivateKey = $(cat client1.key)
Address = 10.0.0.2/24
DNS = 119.29.29.29

[Peer]
PublicKey = $(cat server.key.pub)
AllowedIPs = 10.0.0.0/24
Endpoint = 公网IP:50820
PersistentKeepalive = 30" > client1.conf
```