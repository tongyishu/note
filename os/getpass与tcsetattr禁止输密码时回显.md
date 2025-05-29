设置密码时，为了防止密码泄露，一般都会要求程序不回显，或者回显为"*"字样。但这样存在一个问题：密码输入之后，用户不确定输入的密码是否是自己想要的。为了避免这种情况的发生，有了二次确认的机制：让用户输入两次，两次的结果相同，再进行下一步操作。

linux c中可使用以下两种方式让密码不回显。

# 使用unistd.h头文件中声明的getpass()函数

该方式无法被ctrl + c信号中断，输入回车才可结束输入。并且无法使用管道方式 `|`注入。

```c
#include <stdio.h>
#include <termio.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
	char *pass = getpass("Input password:");
	if (pass)
		printf("%s\n", pass);
	return 0;
}
```

# 使用termio.h头文件中声明的tcsetattr()函数

该方式可以被ctrl + c信号中断，也可以使用使用管道方式 `|`注入。

如果发现程序退出后，敲命令无回显，可以使用stty echo命令打开终端的回显（stty -echo命令可关闭终端的回显）。

```c
#include <stdio.h>
#include <string.h>
#include <termio.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
	struct termios old_settings;
	struct termios new_settings;

	tcgetattr(0, &old_settings);
	new_settings = old_settings;
	new_settings.c_lflag &= ~(ECHO | ICANON);
	tcsetattr(0, TCSANOW, &new_settings);

	char buf[512];
	if (fgets(buf, sizeof(buf), stdin)) {
		int len = strlen(buf);
		buf[len - 1] = 0;
		printf("%s\n", buf);
	}

	tcsetattr(0, TCSANOW, &old_settings);
	return 0;
}
```
