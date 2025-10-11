# Ubuntu启用BBR

### 1. 检查内核版本
确保系统内核版本为4.9或更高，通过以下命令检查内核版本：
```
uname -r
```
### 2. 修改系统配置
修改`sysctl.conf`以启用BBR，执行以下命令将配置添加到`/etc/sysctl.conf`文件中：
```
echo 'net.core.default_qdisc=fq' | tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | tee -a /etc/sysctl.conf
```
- 第一条命令设置默认队列规则为`fq`(Fair Queueing)
- 第二条命令将TCP拥塞控制算法设置为BBR

### 3. 应用更改
修改完成后，重新加载sysctl配置以使更改生效，运行以下命令：
```
sysctl -p
```

### 5. 检查BBR模块
虽然BBR是内核自带的，不需要作为独立模块加载，但可以使用以下命令确认BBR模块是否已加载：
```
lsmod | grep bbr
```