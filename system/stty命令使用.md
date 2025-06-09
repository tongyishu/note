# stty修改和打印终端的字符设置

**-a, --all** 以人可读的形式打印当前所有的配置

**-g, --save** 以机器可读的形式打印当前所有的配置

**-F, --file=DEVICE** 打开并使用指定的设备，而不是标准输入stdin

# 特殊字符

**eof CHAR** 文件结束符

**eol CHAR** 行结束符

**erase CHAR** 清除最后一个输入的字符

**intr CHAR** 发送intr信号

**kill CHAR** 清除当前行

**quit CHAR** 发送quit信号

**start CHAR** 重启输出功能

**stop CHAR** 关闭输出

**susp CHAR** 向终端发送stop信号

# 特殊设置

**N** 设置输入/输出的速率为N bauds

**ispeed N** 设置输入速率为N bauds

**min N** 和-icanon一起使用, 一次至少读取N个字符

**ospeed N** 设置输出速率为N bauds

**speed** 打印终端的速率

**time N** 和-icanon一起使用, 将读取超时的时间设置为十分之N秒

**[-]cread** 允许/不允许接收输入

**csN** 设置字符大小为N bits, N in [5..8]

**[-]echo** 启用/禁用回显

**[-]echok** 在kill字符后，回显/不回显换行符

**[-]echonl** 即使没有其它字符，也要回显/不回显换行符

**[-]icanon** 启用/禁用erase, kill, werase, rprnt特殊字符

**[-]isig** 启用/禁用interrupt, quit, suspend特殊字符

**[-]noflsh** 启用/禁用收到intrt和quit字符后的flush功能
