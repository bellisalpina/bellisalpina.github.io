# AList 静默运行脚本

本文档介绍了两种实现 AList 程序在 Windows 系统下静默运行（即不显示控制台窗口）的方法。

## 方法一：使用 VBScript 脚本

这是一种经典且简单的方法，通过 VBScript 的 `WScript.Shell` 对象来隐藏程序窗口。

**操作步骤：**

1.  创建一个新文件，将其命名为 `run_hidden.vbs`（或任何你喜欢的 `.vbs` 名称）。
2.  将以下代码粘贴到文件中并保存。

```vbscript
Dim ws
Set ws = WScript.CreateObject("WScript.Shell")
ws.run "alist.exe server", vbHide
Set ws = Nothing
WScript.Quit
```

**使用说明：**

将 `run_hidden.vbs` 文件与 `alist.exe` 放在同一目录下，双击该 `.vbs` 文件即可在后台启动 AList。

## 方法二：使用批处理脚本 (嵌套调用)

这种方法通过一个批处理文件调用另一个，并利用 PowerShell 来隐藏窗口，灵活性更高（例如，可以方便地以管理员身份运行）。

### 步骤一：创建主启动脚本

首先，创建一个名为 `start.bat` 的文件，用于执行启动 AList 的核心命令。

```batch
@echo off
cd /d "%~dp0"
.\alist.exe server
```

**代码说明：**

*   `@echo off`: 关闭命令回显，使窗口更干净。
*   `cd /d "%~dp0"`: 切换到批处理文件所在的目录，确保能正确找到 `alist.exe`。

### 步骤二：创建静默启动脚本

接着，创建第二个名为 `start_no_window.bat` 的文件，它将调用 `start.bat` 并隐藏其窗口。

```batch
@echo off
cd /d "%~dp0"
powershell.exe -ExecutionPolicy Bypass -Command "Start-Process '.\start.bat' -WindowStyle Hidden"
```

**代码说明：**

*   `powershell.exe ...`: 调用 PowerShell。
*   `-ExecutionPolicy Bypass`: 临时绕过执行策略，允许运行脚本。
*   `Start-Process`: 启动一个新进程。
*   `'.\start.bat'`: 要启动的进程是 `start.bat`。
*   `-WindowStyle Hidden`: 关键参数，用于隐藏新进程的窗口。

**使用说明：**

将这两个 `.bat` 文件与 `alist.exe` 放在同一目录下，双击 `start_no_window.bat` 即可实现静默启动。

### (可选) 以管理员身份运行

如果 AList 需要管理员权限来访问某些目录，只需在 `start_no_window.bat` 的 PowerShell 命令中添加 `-Verb RunAs` 参数即可。

修改后的 `start_no_window.bat` 内容如下：

```batch
@echo off
cd /d "%~dp0"
powershell.exe -ExecutionPolicy Bypass -Command "Start-Process '.\start.bat' -WindowStyle Hidden -Verb RunAs"
```

**注意：** 使用此参数运行时，系统会弹出 UAC（用户账户控制）窗口请求授权，点击“是”后，AList 将以管理员权限在后台运行。