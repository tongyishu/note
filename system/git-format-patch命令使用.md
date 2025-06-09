# git format-patch -1

将最新一次的commit制作成patch，该命令等同于git format-patch HEAD^，-2就是将最近2次的commit制作成patch，依次类推。

# git format-patch commit1..commit2

将commit1和commit2之间的所有commit提交制作成patch，需要注意的是两个commit之间有".."连接

# git format-patch -1 --stdout > xxx.patch

将patch输出到指定文件，这里为xxx.patch

# git apply --stat xxx.patch

查看patch的统计情况

# git apply --check xxx.patch

检查patch是否能打上，没有任何输出表明OK

# git apply xxx.patch

将patch打到代码中，与git am的区别是，git apply并不会将commit message等打上去，打完patch后需要重新git add和git commit，而git am会直接将patch的所有信息打上去，而且不用重新git add和git commit。

# git am xxx.patch

将patch到代码中，如果有冲突，有以下两种处理方式：

1. 使用git am --abort放弃当前的patch
2. 解决冲突，然后git add，再使用git am --resolved继续应用该patch

处理完后，使用git log会发现该patch已经变成了一个commit追加到代码中。
