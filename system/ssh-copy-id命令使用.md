# 什么是ssh-copy-id

`ssh-copy-id`命令用于将本地主机的公钥拷贝到远程主机的 `authorized_keys`文件中。该操作允许用户每次登录远程主机时，无需输入密码。`ssh-copy-id`是ssh自带的工具，该命令本质上是一个脚本（`/usr/bin/ssh-copy-id`），使用 `vim /usr/bin/ssh-copy-id`可以查看该脚本的内容。

# 如何用ssh-copy-id

以下是 `ssh-copy-id`的使用步骤：

1. 生成ssh密钥对（如果已有密钥对，忽略此步）：

* 在本地机器上打开终端并运行：`ssh-keygen`
* 按提示将密钥保存在`~/.ssh/id_rsa`文件中，公钥保存在`~/.ssh/id_rsa.pub`文件中。

2. 将公钥拷贝到远程主机：

* 使用`ssh-copy-id`命令，指定远程机器的用户和IP地址：`ssh-copy-id username@remotehost`
* 此命令会提示输入指定用户在远程主机上的密码。
* 密码验证成功后，会将本地主机的公钥`~/.ssh/id_rsa.pub`添加到远程主机的`~/.ssh/authorized_keys`文件中。

3. 使用ssh登录至远程主机：

* 复制公钥后，无需密码即可登录远程机器：`ssh username@remotehost`

4. 使用scp拷贝文件到远程主机：

* 复制公钥后，无需密码即可拷贝文件到远程主机：`scp filename.txt username@remotehost:~/filename.txt`
