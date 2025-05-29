patchelf用于修改ELF文件的内容。常用于修改rpath和依赖的三方库名称。

`patchelf OPTION FILE`

# 参数解析

| **参数**                    | **功能**                                        |
| --------------------------------- | ----------------------------------------------------- |
| --page-size                       | 指定page size的大小为SIZE，而不是默认的               |
| --set-interpreter INTERPRETER     | 设置可执行文件的动态加载器（ELF解释器）               |
| --print-interpreter               | 打印ELF文件的动态加载器（ELF解释器）                  |
| --set-soname SONAME               | 设置so的名称为SONAME                                  |
| --print-soname                    | 打印so的名称（.dynamic section DT_SONAME）            |
| --set-rpath RPATH                 | 设置ELF文件的rpath                                    |
| --remove-rpath                    | 删除ELF文件的rpath                                    |
| --print-rpath                     | 打印ELF文件的rpath                                    |
| --replace-needed LIB_ORIG LIB_NEW | 修改依赖库，用新的依赖库LIB_NEW替换原本依赖的LIB_ORIG |
| --no-default-lib                  | 搜索依赖的库时忽略默认搜索路径                        |

# 使用示例

`patchelf --print-interpreter /usr/bin/w` 查看w命令ELF解释器（输出为 /lib64/ld-linux-x86-64.so.2）

`patchelf --set-rpath /usr/share/ libtest.so` 设置libtest.so的rpath为 /usr/share/

`patchelf --replace-needed libdpdk_v1.so libdpdk_v2.so libtest.so` 将libtest.so依赖的老版本libdpdk_v1.so替换成新版本libdpdk_v2.so
