# 场景与前提条件

1、假设有两个机器Windows和Linux，两个机器上的代码git HEAD指向的commit一致

2、在Windows上对代码做了修改，需要将修改移动到Linux上做编译

3、已经使用ssh-copy-id做了免密登陆

# 一键同步修改

```bash
git diff --cached > my.patch && \
scp my.patch root@10.20.69.18:/home/my_project/my.patch && \
ssh root@10.20.69.18 \
' \
cd /home/my_project/ && \
git reset --hard HEAD && \
git apply my.patch \
'
```

# 对该命令的解释

`''`单引号外的是本地执行的命令

`''`单引号内的是远程执行的命令

1、`git diff --cached > my.patch` 将Windows上的本地暂存区的修改打到my.patch，不加`--cached`将工作区的修改打到 my.patch

2、`scp my.patch root@10.20.69.18:/home/project/my.patch` 将my.patch从Windows免密拷贝到Linux上

3、`ssh root@10.20.69.18 'XXX'` 通过ssh执行远程`XXX`命令

4、`cd /home/project/` 进入远程Linux主机的代码目录

5、`git reset --hard HEAD` 将远程Linux主机的代码恢复到初始的HEAD commit

6、`git apply my.patch` 将远程Linux主机上的my.patch以补丁的形式打入代码中
