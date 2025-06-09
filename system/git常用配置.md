# git使用vim作为默认编辑器

```bash
git config --global core.editor "vim"
```

不设置的话，在linux环境下可能会使用nano作为默认编辑器。

# git指定ssh密钥拉取代码

```bash
git config --global core.sshcommand 'ssh -i ~/.ssh/id_ed25519 -F /dev/null'
```

这里使用ed25519算法生成的密钥文件，而不是默认的rsa算法。

# git忽略文件模式

文件从linux环境拷贝到windows环境，文件模式（filemode，755 -> 644）会发生变化。可以通过以下配置忽略这种变化：

```bash
git config core.filemode=false
```

该配置的本质是不将filemode加入git的追踪范围之内，修改之后会在.git/config文件中生成相应的配置。

该命令只针对单个git仓库，全局配置要使用--global选项。
