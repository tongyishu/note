一般在python的工程中都会封装vlog模块来实现日志打印，如果在定位问题时，不清楚具体要调用哪个函数打印，或者打印的内容太多不好找，想将打印的内容单独重定向到一个文件方便查看，可以使用以下语句来实现：

```python
with open("./test.log", "a+") as f:
    print(f'[{datetime.now()}] {"hello world"}', file=f)
```

`test.log`为要输出的日志文件，`hello world`为要输出的内容，实际应用时只需要改动这两处即可。

运行结果：

```bash
# python test.py && cat test.log
[2025-03-24 11:09:01.322679] hello world
```