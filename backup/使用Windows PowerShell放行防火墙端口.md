# 使用Windows PowerShell放行防火墙端口

### ⚠️ 重要前提：以管理员身份运行 PowerShell
所有修改防火墙规则的命令都需要管理员权限。
1.  在开始菜单或搜索栏中输入 `PowerShell`。
2.  右键点击 "Windows PowerShell"，选择 **"以管理员身份运行"**。

### 使用 `New-NetFirewallRule` 命令
在管理员权限的 PowerShell 窗口中，复制并粘贴以下命令，然后按回车键执行。
```powershell
New-NetFirewallRule -DisplayName "Allow TCP Inbound 13477" -Direction Inbound -Protocol TCP -LocalPort 13477 -Action Allow -Profile Any
```
#### 命令详解：
*   `New-NetFirewallRule`: 这是创建新防火墙规则的 PowerShell 命令。
*   `-DisplayName "Allow TCP Inbound 13477"`: 为规则设置一个易于识别的名称。这个名称会出现在防火墙的图形界面中，方便你以后管理。
*   `-Direction Inbound`: 指定规则的方向为“入站”，即允许外部访问本机的端口。
*   `-Protocol TCP`: 指定协议为 TCP。
*   `-LocalPort 13477`: 指定要放行的本地端口号是 13477。
*   `-Action Allow`: 指定动作为“允许”。
*   `-Profile Any`: 指定此规则在任何网络配置文件下都生效。

### 如何删除（撤销）规则？
如果你不再需要这个规则，可以使用以下命令将其删除。同样需要管理员权限。
```powershell
Remove-NetFirewallRule -DisplayName "Allow TCP Inbound 13477"
```
执行后，该规则就会被永久删除。