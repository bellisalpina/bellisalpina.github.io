# Git 代理设置指南

> 本文以 **Clash** 为例，介绍如何为 Git 配置网络代理。

## 一、HTTP/HTTPS 协议代理

通过 HTTP/HTTPS 协议访问的 Git 仓库 URL 示例如下：

```bash
http://github.com/example/example.git
https://github.com/example/example.git
```

### 1. 设置代理

#### 全局代理

为所有 `http` 和 `https` 的 Git 操作设置代理。

```bash
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

#### 仅代理 GitHub

只为访问 `github.com` 设置代理，不影响其他 Git 服务器。

```bash
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```

### 2. 取消代理

#### 取消全局代理

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

#### 取消 GitHub 代理

```bash
git config --global --unset http.https://github.com.proxy
```

## 二、SSH 协议代理

通过 SSH 协议访问的 Git 仓库 URL 示例如下：

```bash
git@github.com:example/example.git
ssh://git@github.com/example/example.git
```

> ⚠️ **注意**：以下配置仅适用于 **Windows** 系统的 Git 客户端。

### 1. 配置代理

1. 找到并编辑 SSH 配置文件。文件路径通常为：`C:\Users\你的用户名\.ssh\config`（如果文件或目录不存在，请手动创建）
2. 在文件末尾添加以下内容：
    ```bash
    Host github.com
        User git
        ProxyCommand connect -H 127.0.0.1:7890 %h %p
    ```

### 2. 取消代理

打开上述 `config` 文件，**删除或注释掉**（在行首添加 `#`）为 `github.com` 添加的配置即可。

## 💡 小贴士

*   **端口号**：`7890` 是 Clash 的默认 HTTP 代理端口。如果你的 Clash 设置了其他端口，请相应修改。
*   **验证代理**：设置完成后，可以通过 `git config --global --get http.proxy` 命令查看当前的全局代理设置。
*   **`connect` 命令**：Windows 版的 Git 通常自带 `connect.exe`，无需额外安装。