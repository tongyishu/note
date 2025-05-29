# RF测试框架

RF（Robot Framework）是一个通用的开源自动化测试框架，通常使用python/java进行集成。简单的讲，RF可以把python中的函数名当成关键字，直接在RF框架中调用。

RF框架 --------> Python脚本 --------> 被测程序

# 网站链接

RF官网地址：
https://robotframework.org/

RF使用指导：
https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html

RF下载地址：
https://pypi.org/project/robotframework/

# 下载安装

```bash
python -m pip install robotframework
```

# 使用方法

```bash
robot [options] 
```

示例：

```bash
robot mytest.robot
robot -l NONE -r NONE -o NONE mytest.robot
robot --loglevel=DEBUG -s ovs myconf.robot
```

options是选项，其中-l NONE是禁用html的log输出，-r NONE是禁用html的报告输出，-o NONE是禁用XML的日志输出，--loglevel指定日志的级别，-s指定测试套（测试用例所在目录）。paths是一般是项目的配置文件，也可以是测试用例文件，均为.robot后缀。使用robot --help可以查看更多的帮助信息。

# 工程目录

一般的工程目录结构如下（只包含robot配置文件和robot测试用例，不包含python脚本）：

```bash
my_project/
    test_configs/
        config1.robot
        config2.robot
        ...
    test_scripts/
        script1.py
        script2.py
        ...
    test_suite1/
        __init__.robot
        my_test1.robot
        my_test2.robot
        ...
    test_suite2
        __init__.robot
        my_test3.robot
        ...
```

# 变量类型

${}表示标准变量

&{}表示字典变量

@{}表示链表变量

# 脚本调用

RF的一个非常有用的特性是，在.robot文件中可以直接调用.py文件的函数。比如在 `mylib.py` 中有如下函数：

```python
def add_numbers(a, b):
    return a + b
```

只需要在.robot文件中导入mylib.py：

```bash
*** Settings ***
Library    mylib.py
```

就可以通过如下方式调用（忽略大小写、忽略空格、忽略下划线）：

```bash
*** Test Cases ***
mytest
    ${sum}=    add_numbers    3    4
    ${sum}=    Add_Numbers    3    4
    ${sum}=    addnumbers_    3    4
    ${sum}=    _addnumbers    3    4
    ${sum}=    aDd_nUMbers    3    4
    ${sum}=    add numbers    3    4
    ${sum}=    ADD NUMBERS    3    4
```

注意：RF的log关键字和python中的print具有同样的效果，都将内容打印到output.xml中。

# 使用举例

以下代码使用Python的`signal`和`scapy`库来实现UDP数据包的捕获和发送，并利用多线程来实现并发执行：

`captured`用于记录已捕获的数据包数量，

`stop`用于通知捕获数据包的线程停止执行，

`stop_event()`函数用于判断停止事件是否触发。

`handle_packets()`函数用于处理捕获到的数据包，此处我们只关心UDP协议，并且只对目标端口为12580的数据包进行计数。

`send_packets()`函数用于发送UDP数据包，这里我们使用`sendp()`函数来发送500个UDP数据包。

`sniff_packets()`函数用于捕获数据包，内部调用`sniff()`函数，并通过`filter`参数指定过滤条件，`prn`参数指定回调函数，`stop_filter`参数指定是否停止抓包。

`test_udp`函数是测试主函数，它创建了两个线程，分别用于捕获数据包和发送数据包。然后通过调用`start()`方法启动线程，实现并发执行。最后，调用`join()`方法等待发送数据包的线程结束后再执行后续代码，并将`stop`变量设置为1，通知捕获数据包的线程停止执行。

```python
from signal import *
from scapy.all import *
import threading
captured = 0
stop = 0

def stop_event(event):
    global stop
    return stop == 1

def handle_packets(packet):
    global captured
    if packet.haslayer(UDP):
        if (packet[UDP].dport == 12580):
            captured += 1

def send_packets():
    sendp(Ether()/IP(dst='127.0.0.1')/UDP(dport=12580), inter=0.01, count=500, verbose=False)

def sniff_packets():
    sniff(filter='udp', prn=handle_packets, stop_filter=stop_event)

def test_udp():
    global stop
    sniffer = threading.Thread(target=sniff_packets)
    sniffer.start()
    sender = threading.Thread(target=send_packets)
    sender.start()
    sender.join()
    stop = 1
    sniffer.join()
    return captured
```

robot自动化

```bash
*** Settings ***
Library    testlib.py
Library    String

*** Test Cases ***
Example Test
    ${result}=    test_udp
    Should Be Equal As Strings    ${result}    500
```

# 内置关键字

参考RF使用指导：
https://robotframework.org/robotframework/latest/RobotFrameworkUserGuide.html

```bash
Call Method
Catenate
Comment
Continue For Loop
Continue For Loop If
Convert To Binary
Convert To Boolean
Convert To Bytes
Convert To Hex
Convert To Integer
Convert To Number
Convert To Octal
Convert To String
Create Dictionary
Create List
Evaluate
Exit For Loop
Exit For Loop If
Fail
Fatal Error
Get Count
Get Length
Get Library Instance
Get Time
Get Variable Value
Get Variables
Import Library
Import Resource
Import Variables
Keyword Should Exist
Length Should Be
Log
Log Many
Log To Console
Log Variables
No Operation
Pass Execution
Pass Execution If
Regexp Escape
Reload Library
Remove Tags
Repeat Keyword
Replace Variables
Return From Keyword
Return From Keyword If
Run Keyword
Run Keyword And Continue On Failure
Run Keyword And Expect Error
Run Keyword And Ignore Error
Run Keyword And Return
Run Keyword And Return If
Run Keyword And Return Status
Run Keyword And Warn On Failure
Run Keyword If
Run Keyword If All Tests Passed
Run Keyword If Any Tests Failed
Run Keyword If Test Failed
Run Keyword If Test Passed
Run Keyword If Timeout Occurred
Run Keyword Unless
Run Keywords
Set Global Variable
Set Library Search Order
Set Local Variable
Set Log Level
Set Suite Documentation
Set Suite Metadata
Set Suite Variable
Set Tags
Set Task Variable
Set Test Documentation
Set Test Message
Set Test Variable
Set Variable
Set Variable If
Should Be Empty
Should Be Equal
Should Be Equal As Integers
Should Be Equal As Numbers
Should Be Equal As Strings
Should Be True
Should Contain
Should Contain Any
Should Contain X Times
Should End With
Should Match
Should Match Regexp
Should Not Be Empty
Should Not Be Equal
Should Not Be Equal As Integers
Should Not Be Equal As Numbers
Should Not Be Equal As Strings
Should Not Be True
Should Not Contain
Should Not Contain Any
Should Not End With
Should Not Match
Should Not Match Regexp
Should Not Start With
Should Start With
Skip
Skip If
Sleep
Variable Should Exist
Variable Should Not Exist
Wait Until Keyword Succeeds
```
