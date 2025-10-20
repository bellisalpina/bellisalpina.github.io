# Ubuntu 启用 BBR

BBR（Bottleneck Bandwidth and RTT）是 Google 开发的一种 TCP 拥塞控制算法，可以有效提升网络吞吐量和降低延迟。以下是在 Ubuntu 系统上启用 BBR 的详细步骤。

## 1. 检查内核版本

BBR 要求 Linux 内核版本为 4.9 或更高。首先，使用以下命令检查当前内核版本：

```bash
uname -r
```

如果输出的版本号低于 `4.9.0`，请先升级内核。

## 2. 修改系统配置

接下来，修改系统配置文件 `/etc/sysctl.conf` 来启用 BBR。执行以下命令，将所需配置追加到文件末尾：

```bash
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
```

这两行配置的作用分别是：

*   `net.core.default_qdisc=fq`：将默认的队列调度算法设置为 `fq`（Fair Queueing）。
*   `net.ipv4.tcp_congestion_control=bbr`：将 TCP 的拥塞控制算法设置为 `bbr`。

## 3. 应用配置

配置添加完毕后，执行以下命令以重新加载 `sysctl` 配置，使更改立即生效：

```bash
sysctl -p
```

如果命令执行后没有报错，并显示了你刚刚添加的两行配置，说明应用成功。

## 4. 验证 BBR 状态

最后，验证 BBR 是否已成功启用。可以通过以下两种方式检查：

**方法一：检查当前使用的拥塞控制算法**

执行以下命令，查看当前系统正在使用的 TCP 拥塞控制算法：

```bash
sysctl net.ipv4.tcp_congestion_control
```

如果输出结果为 `net.ipv4.tcp_congestion_control = bbr`，则表示设置成功。

**方法二：检查内核可用的拥塞控制算法**

查看内核当前支持的拥塞控制算法列表：

```bash
cat /proc/sys/net/ipv4/tcp_available_congestion_control
```

如果输出中包含 `bbr`，则表示内核已支持 BBR。