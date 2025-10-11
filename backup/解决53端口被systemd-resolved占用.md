# 解决53端口被systemd-resolved占用

### 1. 先临时停用systemd-resolved服务
```bash
systemctl stop systemd-resolved
```

### 2. 编辑`/etc/systemd/resolved.conf`
```bash
nano -w /etc/systemd/resolved.conf
```

### 3. 修改并保存
```conf
[Resolve]
DNS=119.29.29.29  #取消注释
...
DNSStubListener=no  #取消注释，把yes改为no
```

### 4. 链接
```bash
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```