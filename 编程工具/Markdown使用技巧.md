# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Markdown Preview Enhanced官方使用文档](https://shd101wyy.github.io/markdown-preview-enhanced/#/)
> [Pandoc官网](https://www.pandoc.org/index.html)

# 转PDF
VS Code安装插件`Markdown Preview Enhanced`，打开预览记得选择`Markdown Preview Enhanced`的按钮，默认的的预览没有右键菜单，无法支持导出PDF，右键选择`Chrome > PDF`。
![127](https://img-blog.csdnimg.cn/20191215180142905.png)
20200507补充：一个免费的在线文档互转网站[Speedpdf](https://speedpdf.com/)。
20201205补充：点击没反应
![30](https://img-blog.csdnimg.cn/20201205030217117.png)
# 转word
右键选择`Pandoc`总是弹出`error: Output format needs to be specified`，下载安装Pandoc，命令行，
```shell
C:\Users\qinge\Downloads>pandoc.exe -s Finace-FPGA方案设计文档.md -o Finace-FPGA方案设计文档.docx
C:\Users\qinge\Downloads>pandoc.exe -s Finace-FPGA软件使用文档.md -o Finace-FPGA软件使用文档.docx
C:\Users\qinge\Downloads>pandoc.exe -s Finace-FPGA硬件使用文档.md -o Finace-FPGA硬件使用文档.docx
```
# 表格内换行
在两行内容中间加上br，类似下面这种，<>需要加。
```html
zhu<br>ce
```
# 流程图内换行
添加两个空格之后，按回车，br是不起作用的
# 流程图方向
`->`是默认的从上向下的方向，控制其他方向用下面的方法：
![这里写图片描述](https://img-blog.csdn.net/20180627000754391?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
效果如下，CSDN的效果很垃圾没法看，试试连续向右连接10个框，直接跑到笔记本屏幕外面了。
```mermaid
flowchat
op0=>operation: Camera
op1=>operation: Video Data
op2=>operation: SDI
op0(right)->op1->op2
```
