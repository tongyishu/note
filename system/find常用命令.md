# 按名称查找

* **`-name "模式"`**
  按文件名匹配（区分大小写）

  ```bash
  find /home -name "*.txt"  # 查找 /home 下所有 .txt 文件
  ```
* **`-iname "模式"`**
  按文件名匹配（不区分大小写）

  ```bash
  find . -iname "readme.md"  # 查找当前目录下的 README.md、Readme.MD 等
  ```

---

# 按类型查找

* **`-type [类型]`**
  指定文件类型：

  * `f`：普通文件
  * `d`：目录
  * `l`：符号链接

  ```bash
  find /var/log -type f  # 查找 /var/log 下的所有文件
  find . -type d         # 查找当前目录下的所有子目录
  ```

---

# 按时间查找

* **`-mtime [天数]`**
  按文件修改时间：

  * `+n`：n 天前修改
  * `-n`：n 天内修改

  ```bash
  find /backup -mtime -7  # 查找 7 天内修改过的文件
  ```
* **`-mmin [分钟数]`**
  按分钟为单位的时间（修改时间）

  ```bash
  find /tmp -mmin -30  # 查找最近 30 分钟内修改过的文件
  ```

---

# 按大小查找

* **`-size [大小]`**
  按文件大小：

  * `+n`：大于 n
  * `-n`：小于 n
  * 单位：`k`（KB）、`M`（MB）、`G`（GB）

  ```bash
  find /var -size +10M    # 查找大于 10MB 的文件
  find . -size -100k     # 查找小于 100KB 的文件
  ```

---

# 按权限查找

* **`-perm [权限]`**
  按文件权限匹配：

  ```bash
  find /etc -perm 644          # 查找权限为 644 的文件
  find . -perm /u=x            # 查找用户有执行权限的文件
  find / -perm -4000           # 查找设置了 SUID 权限的文件
  ```

# 逻辑运算符

* `-a`（与，默认）
* `-o`（或）
* `!`（非）

```bash
find . -name "*.log" -size +1M  # 查找所有 .log 文件且大于 1MB
find /var \( -name "*.tmp" -o -name "*.bak" \)  # 查找 .tmp 或 .bak 文件
find . ! -name "*.txt"          # 排除所有 .txt 文件
```

# 删除文件

* **`-delete`**

  ```bash
  find /tmp -name "*.tmp" -delete  # 删除所有 .tmp 文件
  ```

# 执行命令

* **`-exec [命令] {} \;`**
  `{}` 表示匹配的文件，`\;` 表示命令结束

  ```bash
  find . -name "*.jpg" -exec cp {} /backup \;  # 复制所有 .jpg 到 /backup
  ```
* **`-ok [命令] {} \;`**
  交互式确认后再执行命令

  ```bash
  find . -name "*.log" -ok rm {} \;  # 删除前询问
  ```

# 输出结果

* **`-print`** （默认输出，可省略）

  ```bash
  find /home -name "*.pdf" -print
  ```
