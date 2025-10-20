# 将 FRP Server 配置为 systemd 服务

使用 `systemd` 可以将 `frps` 进程作为系统服务管理，实现开机自动启动、崩溃后自动重启，并通过 `systemctl` 命令方便地进行启停和状态查看，是推荐的服务部署方式。

### 📋 前提条件

> **请注意**：本教程假设 `frps` 可执行文件及其配置文件 `frps.toml` 均位于 `/root/frp/` 目录下。如果你的路径不同，请相应地修改后续配置文件中的路径。

### 📝 第一步：创建 `systemd` 服务文件

1.  **创建并编辑服务文件**。

    使用你喜欢的文本编辑器（如 `nano` 或 `vim`）创建 `/etc/systemd/system/frps.service` 文件：

    ```bash
    nano /etc/systemd/system/frps.service
    ```

2.  **填入服务配置内容**。

    将以下配置粘贴到文件中：

    ```ini
    [Unit]
    Description=Frp Server Service
    After=network.target
    
    [Service]
    Type=simple
    User=root
    Group=root
    WorkingDirectory=/root/frp
    ExecStart=/root/frp/frps -c /root/frp/frps.toml
    Restart=on-failure
    RestartSec=5s
    
    [Install]
    WantedBy=multi-user.target
    ```

#### 🔧 配置详解

*   `[Unit]` 部分：定义服务的元数据。
    *   `Description`: 服务的描述信息。
    *   `After=network.target`: 确保在网络服务启动后再启动 `frps`。
*   `[Service]` 部分：定义服务如何运行。
    *   `Type=simple`: 表示服务是一个前台进程，`systemd` 会认为主进程就是 `ExecStart` 启动的进程。
    *   `User` / `Group`: 以 `root` 用户和用户组身份运行服务。
    *   `WorkingDirectory`: 设置服务的工作目录，这对于解析相对路径很重要。
    *   `ExecStart`: 启动服务的具体命令。
    *   `Restart=on-failure`: 当服务异常退出时，自动重启。
    *   `RestartSec=5s`: 重启前等待 5 秒，防止频繁重启导致资源浪费。
*   `[Install]` 部分：定义如何安装服务。
    *   `WantedBy=multi-user.target`: 表示在多用户模式下（即正常的命令行界面）启用该服务。

### 🚀 第二步：启用并管理服务

依次执行以下命令，让 `systemd` 识别新服务、设置开机自启并立即启动它。

```bash
# 1. 重新加载 systemd 配置，使其识别新的服务文件
systemctl daemon-reload
# 2. 设置 frps 服务为开机自启
systemctl enable frps.service
# 3. 立即启动 frps 服务
systemctl start frps.service
```

> **💡 小贴士**：
> *   停止服务：`systemctl stop frps.service`
> *   重启服务：`systemctl restart frps.service`

### ✅ 第三步：验证服务状态与查看日志

1.  **检查服务状态**，确认其是否正在运行：

    ```bash
    systemctl status frps.service
    ```

    如果看到 `active (running)` 的绿色字样，说明服务已成功启动。
    
2.  **查看服务的实时日志**，用于排查问题：

    ```bash
    # -u 指定服务单元, -f 实时跟踪日志输出
    journalctl -u frps.service -f
    ```

    按 `Ctrl + C` 即可退出日志跟踪。