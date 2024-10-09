# 为Git设置代理

> 以**Clash**为例  

## HTTP/HTTPS协议  
使用 HTTP / HTTPS 传输协议连接到 Git 仓库的 URL 实例如下：  
```
http://github.com/example/example.git
https://github.com/example/example.git
```
 
+ 全局代理  
```
git config --global http.proxy http://127.0.0.1:7890
```
+ 只代理Github  
```
git config --global http.https://github.com.proxy http://127.0.0.1:7890
```
+ 撤销全局代理  
```
git config --global --unset http.proxy
```
+ 撤销代理GitHub
```
git config --global --unset http.https://github.com.proxy
```


## SSH协议  
使用 SSH 传输协议连接到 Git 仓库的 URL 实例如下：
```
git@github.com:example/example.git
ssh://git@github.com/example/example.git
```

以下内容仅限**Windows**端Git  
+ 编辑`C:\Users\example\.ssh\config`，输入以下内容：  
```
Host github.com
    User git
    ProxyCommand connect -H 127.0.0.1:7890 %h %p
```
+ 撤销代理：删除添加的内容