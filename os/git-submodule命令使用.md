git submodule建立了主模块和子模块的依赖关系：子模块路径、子模块的远程仓库、子模块的版本号。

# git submodule add [URL]

该命令会将URL中的仓库作为项目的一个子模块，在.git/config和.gitmodules文件中添加URL中的仓库信息。

# git submodule init [PATH]

该命令会将.gitmodule文件中路径为PATH的子模块初始化，写入.git/config文件。如果不加PATH，则初始化所有子模块。

# git submodule deinit [PATH] [--all] [--force]

该命令将.gitmodules中路径为PATH的子模块卸载，即将子模块从.git/config中删除。PATH和--all二选其一，--all是将.gitmodule中的所有的子模块从.git/config中删除。--force为强制删除，即使暂存区还有保存的内容。

# git submodule status [PATH]

查看所有PATH的子模块的状态。如果不加PATH，则查看所有的子模块的状态。

# git submodule update [PATH]

签出路径为PATH的子模块的内容。如果不加PATH，则签出所有的子模块。

# git submodule foreach "git pull"

对每个子模块执行git pull命令。
