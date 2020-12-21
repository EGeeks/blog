
# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [python 文件读写操作](https://www.cnblogs.com/zyber/p/9578240.html)
> [Python读写文件](https://www.cnblogs.com/qq931399960/p/11118659.html)
> [Python If ... Else](https://www.w3school.com.cn/python/python_conditions.asp)

# 文件操作
python文件对象提供了三个“读”方法： read()、readline() 和 readlines()。每种方法可以接受一个变量以限制每次读取的数据量。
- read() 每次读取整个文件，它通常用于将文件内容放到一个字符串变量中。如果文件大于可用内存，为了保险起见，可以反复调用read(size)方法，每次最多读取size个字节的内容。
- readlines() 之间的差异是后者一次读取整个文件，象 .read() 一样。.readlines() 自动将文件内容分析成一个行的列表，该列表可以由 Python 的 for ... in ... 结构进行处理。
- readline() 每次只读取一行，通常比readlines() 慢得多。仅当没有足够内存可以一次读取整个文件时，才应该使用 readline()。

```python
with open('hw.txt', 'r') as f:
    hw_list = f.readlines()
print(hw_list)
print(len(hw_list))
for item in hw_list:
    print(item.rstrip('\n'))
   
with open('test1.txt', 'r') as f1:
    list1 = f1.readlines()
for i in range(0, len(list1)):
    list1[i] = list1[i].rstrip('\n')
```
由于文件读写产生IOError后面的f.close()就不会调用。可以使用try finally来实现调用close，或使用with自动帮我们调用close()方法，
```bash
try:
    f = open('/path/to/file', 'r')
    print(f.read())
finally:
    if f:
        f.close()
```
但是异常还是会弹出，
```shell
PS C:\dog\program\python\test> python .\test.py
  File "C:\dog\program\python\test\test.py", line 1, in <module>
    with open('hw.txt', 'r') as f:
FileNotFoundError: [Errno 2] No such file or directory: 'hw.txt'
```

# for循环
记得加冒号，否则语法错误，
```python
for item in hw_list:
    print(item.rstrip('\n'))
```

# 数组
如果使用`[]`初始化数组，需要调用`append`来添加元素，直接引用`car[i]`会报错`IndexError: list assignment index out of range`，
```python
cars = ["Porsche", "Volvo", "BMW"]
print(cars)
cars = []
cars.append("Porsche")
cars.append("Volvo")
print(cars)
print(len(cars))
i = 0
if i >= len(cars):
    print("error index")
else:
    print(cars[i])
i = 4
if i >= len(cars):
    print("error index")
else:
    print(cars[i])
```

