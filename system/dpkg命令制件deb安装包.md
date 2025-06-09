# 目录结构

在myapp的同级目录下执行`dpkg -b myapp myapp.deb`即可制作myapp.deb软件包

```bash
myapp/

├── boot

│ └── readme.txt

└── DEBIAN

├── control

├── postinst

├── postrm

├── preinst

└── prerm
```

**boot/readme.txt** 要安装的软件内容

**DEBIAN/control** 打包的控制文件

**DEBIAN/postinst** 安装后执行的脚本

**DEBIAN/postrm** 卸载后执行的脚本

**DEBIAN/preinst** 安装前执行的脚本

**DEBIAN/prerm** 卸载前执行的脚本

# control文件内容（必须项，其它为可选项）

```bash
Package: myapp

Version: 1.0

Section: utils

Priority: optional

Architecture: amd64

Maintainer: tongyishu

Description: my application
```

# dpkg常用命令

**dpkg -b myapp myapp.deb** 从myapp目录制作myapp.deb包

**dpkg -e myapp.deb tmp** 解压myapp.deb包中的DEBIAN内容至tmp

**dpkg -x myapp.deb tmp** 解压myapp.deb包中的软件内容至tmp

**dpkg -I myapp.deb** 查看myapp.deb包中的DEBIAN内容

**dpkg -c myapp.deb** 查看myapp.deb包中的软件内容

**dpkg -i myapp.deb** 安装myapp.deb包

**dpkg -r myapp** 卸载已安装的myapp软件

**dpkg -s myapp** 查看已安装的myapp状态

**dpkg -L myapp** 查看已安装的myapp内容

**dpkg -l** 查看已安装的所有软件
