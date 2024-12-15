# AList 静默运行脚本

使用以下 VBScript 代码可实现 AList 静默运行：

```vbscript
Dim ws
Set ws = WScript.CreateObject("WScript.Shell")
ws.run "alist.exe server", vbHide
Set ws = Nothing
WScript.Quit
```

### 使用方法

1. 将代码复制到文本文件中。
2. 保存为 `run_alist.vbs`。
3. 双击运行，AList 将在后台运行。