使用 `ssh -J` 进行跳转连接（SSH Jump Host）是通过一个或多个中间服务器（跳板机）连接到目标主机的便捷方式。以下是详细用法和配置指南：

# **基本语法**

```bash
ssh -J [跳板机用户@跳板机地址] [目标用户@目标主机地址]
```

* **单跳板示例** ：

  ```bash
  ssh -J user1@jump.example.com user2@target.example.com
  ```

  等效于：先通过 `user1@jump.example.com` 登录跳板机，再从跳板机连接到 `user2@target.example.com`。
* **多跳板示例** （逗号分隔）：

  ```bash
  ssh -J user1@jump1.example.com,user2@jump2.example.org user3@final.target
  ```

  连接路径：本地 → `jump1` → `jump2` → `final.target`。

# **配置文件简化**

在 `~/.ssh/config` 中预定义跳转规则，避免重复输入参数。

**示例配置** ：

```bash
# 跳板机配置
Host jump
    HostName jump.example.com
    User user1
    IdentityFile ~/.ssh/jump_key  # 跳板机私钥

# 目标主机配置
Host target
    HostName target.example.com
    User user2
    ProxyJump jump                 # 指定跳板机
    IdentityFile ~/.ssh/target_key # 目标主机私钥
```

使用简化命令连接：

```bash
ssh target
```



# **多级跳转**

通过逗号分隔多个跳板机：

```bash
ssh -J user1@jump1,user2@jump2 user3@target
```

或配置文件中：

```bash
Host target
    ProxyJump jump1,jump2  # 依次通过 jump1 和 jump2
```

# **端口与身份验证**

* 跳板机和目标主机的 SSH 端口若非默认（22），需显式指定：

  ```bash
  ssh -J user1@jump1:2222 user2@target:2222
  ```
* 确保本地到跳板机、跳板机到目标主机的 SSH 密钥或密码已正确配置。

# **调试连接**

* 添加 `-v` 参数查看详细连接过程：

  ```bash
  ssh -v -J jump@example.com user@target
  ```

# **替代方案（旧版本兼容）**

如果 OpenSSH 版本低于 7.3（不支持 `-J`），使用 `ProxyCommand`：

```bash
ssh -o "ProxyCommand=ssh -W %h:%p user@jump" user@target
```

或在配置文件中：

```bash
Host target
    ProxyCommand ssh -W %h:%p jump
```

# **通过跳板机传输文件**

使用 `scp` 或 `rsync` 时，同样支持 `-J` 参数：

```bash
scp -J user@jump example.txt user@target:/path/
```
