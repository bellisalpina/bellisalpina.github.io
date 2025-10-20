# 解决 53 端口被 systemd-resolved 占用的问题

当你需要自行安装 DNS 服务时，会发现 `systemd-resolved` 服务默认占用了 53 端口。以下是解决此问题的标准步骤。

## 1. 临时停止 systemd-resolved 服务

为了立即释放 53 端口，首先需要停止该服务。
```bash
sudo systemctl stop systemd-resolved
```

## 2. 修改配置文件

编辑 `systemd-resolved` 的主配置文件。
```bash
nano -w /etc/systemd/resolved.conf
```

## 3. 更改配置项

在文件中找到 `[Resolve]` 部分，修改或取消以下两行的注释，并保存文件。
```conf
[Resolve]
DNS=119.29.29.29  # 取消注释，并设置你希望的上游 DNS 服务器
...
DNSStubListener=no  # 关键步骤：取消注释，并将 yes 改为 no
...
```
> **说明**：
> - `DNS=`：设置系统的上游 DNS 服务器。
> - `DNSStubListener=no`：禁用监听在 `127.0.0.53:53` 的 DNS 存根，这是释放端口的核心。

## 4. 更新 resolv.conf 链接

将系统的 `/etc/resolv.conf` 文件链接到 `systemd-resolved` 生成的静态配置文件，以确保 DNS 解析正常工作。
```bash
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```
完成以上步骤后，你就可以启动自己的 DNS 服务了。