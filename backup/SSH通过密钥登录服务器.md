# SSH通过密钥登录服务器  

要通过SSH密钥登录到服务器，需要生成一对密钥(公钥和私钥)，并将公钥放置在服务器上  

### 1. 生成密钥对
使用以下命令生成一对SSH密钥：
```
ssh-keygen -t rsa
```
- 按照提示，可以选择密钥保存的位置和设置一个密钥锁码，也可以留空以实现无密码登录

### 2. 添加公钥到服务器
1. 登录到服务器
2. 创建 `.ssh` 目录(如果尚不存在)
   ```
   mkdir -p ~/.ssh
   ```
3. 将公钥内容添加到 `authorized_keys` 文件中：
   ```
   cat id_rsa.pub >> ~/.ssh/authorized_keys
   ```
4. 设置正确的权限：
   ```
   chmod 600 ~/.ssh/authorized_keys
   chmod 700 ~/.ssh
   ```

### 3. 配置SSH服务
确保SSH服务允许使用密钥登录，编辑SSH配置文件 `/etc/ssh/sshd_config`，确保以下选项被设置为 `yes`：
```
PubkeyAuthentication yes
```
如果希望禁用密码登录，可以设置：
```
PasswordAuthentication no
```
完成后，重启SSH服务：
```
sudo service sshd restart
```