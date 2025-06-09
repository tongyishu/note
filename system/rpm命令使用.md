`rpm -qa` 查询所有已安装的rpm包

`rpm -qf /usr/bin/python` 查询某一文件所属的rpm包

`rpm -ql python-2.7.5-76.el7.x86_64` 查看已安装的python-2.7.5-76.el7.x86_64的文件列表

`rpm -qi python-2.7.5-76.el7.x86_64` 查看已安装的python-2.7.5-76.el7.x86_64的信息

`rpm -qR python-2.7.5-76.el7.x86_64` 查看已安装的python-2.7.5-76.el7.x86_64的依赖

`rpm -qpl python-2.7.5-76.el7.x86_64.rpm` 查询python-2.7.5-76.el7.x86_64.rpm包的文件列表

`rpm -qpi python-2.7.5-76.el7.x86_64.rpm` 查询python-2.7.5-76.el7.x86_64.rpm包的信息

`rpm -qpR python-2.7.5-76.el7.x86_64.rpm` 查询python-2.7.5-76.el7.x86_64.rpm包的依赖

`rpm -ivh python-2.7.5-76.el7.x86_64.rpm` 安装python-2.7.5-76.el7.x86_64.rpm包

`rpm -Uvh python-2.7.5-76.el7.x86_64.rpm` 升级python-2.7.5-76.el7.x86_64.rpm包

`rpm -e python-2.7.5-76.el7.x86_64` 卸载已安装的python-2.7.5-76.el7.x86_64

`rpm --force` 强制安装

`rpm --nodeps` 忽略依赖

`rpm --noscripts` 等价于--nopre --nopost --nopreun --nopostun，不执行安装前后的脚本动作
