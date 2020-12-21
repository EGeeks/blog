# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)


# 下载
[官网](https://www.python.org/)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190223225256913.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 安装
默认安装路径，
```shell
C:\Users\qinge\AppData\Local\Programs\Python\Python37
```
安装结束有一个取消最大文件路径长度MAX_PATH为260字符的限制，我这里没有使能。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190223225352560.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)配置环境变量，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190223231200632.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
安装时如果选择了Add Python to PATH，就不需要手动配置环境变量了，
![34](https://img-blog.csdnimg.cn/20201220234400712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)

# 插件
更新pip，安装flake8、yapf，pylint

```shell
C:\Users\qinge>pip install flake8
Collecting flake8
  Downloading https://files.pythonhosted.org/packages/54/a7/adf0c095af5b6c33d560780404504e9d58d9a1999253834f2b2d141098d8/flake8-3.7.6-py2.py3-none-any.whl (68kB)
    100% |████████████████████████████████| 71kB 24kB/s
Collecting pycodestyle<2.6.0,>=2.5.0 (from flake8)
  Downloading https://files.pythonhosted.org/packages/0e/0c/04a353e104d2f324f8ee5f4b32012618c1c86dd79e52a433b64fceed511b/pycodestyle-2.5.0-py2.py3-none-any.whl (51kB)
    100% |████████████████████████████████| 51kB 25kB/s
Collecting mccabe<0.7.0,>=0.6.0 (from flake8)
  Downloading https://files.pythonhosted.org/packages/87/89/479dc97e18549e21354893e4ee4ef36db1d237534982482c3681ee6e7b57/mccabe-0.6.1-py2.py3-none-any.whl
Collecting pyflakes<2.2.0,>=2.1.0 (from flake8)
  Downloading https://files.pythonhosted.org/packages/16/3b/b6a508ad148ce1ef50bd7a9a783afbb8d775616fc4ae5e3007c8815a3c85/pyflakes-2.1.0-py2.py3-none-any.whl (62kB)
    100% |████████████████████████████████| 71kB 20kB/s
Collecting entrypoints<0.4.0,>=0.3.0 (from flake8)
  Downloading https://files.pythonhosted.org/packages/ac/c6/44694103f8c221443ee6b0041f69e2740d89a25641e62fb4f2ee568f2f9c/entrypoints-0.3-py2.py3-none-any.whl
Installing collected packages: pycodestyle, mccabe, pyflakes, entrypoints, flake8
Successfully installed entrypoints-0.3 flake8-3.7.6 mccabe-0.6.1 pycodestyle-2.5.0 pyflakes-2.1.0
You are using pip version 18.1, however version 19.0.3 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.
C:\Users\qinge>pip install yapf
Collecting yapf
  Downloading https://files.pythonhosted.org/packages/f7/b6/266774a11dc81539dcf3b5117cd7a3c1c2b11a47853940b7f0803cf389b2/yapf-0.26.0-py2.py3-none-any.whl (176kB)
    100% |████████████████████████████████| 184kB 63kB/s
Installing collected packages: yapf
Successfully installed yapf-0.26.0
You are using pip version 18.1, however version 19.0.3 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.

C:\Users\qinge>python -m pip install --upgrade pip
Collecting pip
  Downloading https://files.pythonhosted.org/packages/d8/f3/413bab4ff08e1fc4828dfc59996d721917df8e8583ea85385d51125dceff/pip-19.0.3-py2.py3-none-any.whl (1.4MB)
    100% |████████████████████████████████| 1.4MB 197kB/s
Installing collected packages: pip
  Found existing installation: pip 18.1
    Uninstalling pip-18.1:
      Successfully uninstalled pip-18.1
Successfully installed pip-19.0.3
C:\Users\qinge>python -m pip install -U pylint --user
Collecting pylint
  Downloading https://files.pythonhosted.org/packages/60/c2/b3f73f4ac008bef6e75bca4992f3963b3f85942e0277237721ef1c151f0d/pylint-2.3.1-py3-none-any.whl (765kB)
     |████████████████████████████████| 768kB 125kB/s
Collecting colorama; sys_platform == "win32" (from pylint)
  Downloading https://files.pythonhosted.org/packages/4f/a6/728666f39bfff1719fc94c481890b2106837da9318031f71a8424b662e12/colorama-0.4.1-py2.py3-none-any.whl
Requirement already satisfied, skipping upgrade: mccabe<0.7,>=0.6 in c:\users\qinge\appdata\local\programs\python\python37\lib\site-packages (from pylint) (0.6.1)
Collecting astroid<3,>=2.2.0 (from pylint)
  Downloading https://files.pythonhosted.org/packages/d5/ad/7221a62a2dbce5c3b8c57fd18e1052c7331adc19b3f27f1561aa6e620db2/astroid-2.2.5-py3-none-any.whl (193kB)
     |████████████████████████████████| 194kB 125kB/s
Collecting isort<5,>=4.2.5 (from pylint)
  Downloading https://files.pythonhosted.org/packages/e5/b0/c121fd1fa3419ea9bfd55c7f9c4fedfec5143208d8c7ad3ce3db6c623c21/isort-4.3.21-py2.py3-none-any.whl (42kB)
     |████████████████████████████████| 51kB 71kB/s
Collecting six (from astroid<3,>=2.2.0->pylint)
  Downloading https://files.pythonhosted.org/packages/73/fb/00a976f728d0d1fecfe898238ce23f502a721c0ac0ecfedb80e0d88c64e9/six-1.12.0-py2.py3-none-any.whl
Collecting lazy-object-proxy (from astroid<3,>=2.2.0->pylint)
  Downloading https://files.pythonhosted.org/packages/bb/86/10c9846c822ee923c157d0abe24d322472315ea47a216e6bf99e3b5b771f/lazy_object_proxy-1.4.1-cp37-cp37m-win_amd64.whl
Collecting wrapt (from astroid<3,>=2.2.0->pylint)
  Downloading https://files.pythonhosted.org/packages/23/84/323c2415280bc4fc880ac5050dddfb3c8062c2552b34c2e512eb4aa68f79/wrapt-1.11.2.tar.gz
Collecting typed-ast>=1.3.0; implementation_name == "cpython" (from astroid<3,>=2.2.0->pylint)
  Downloading https://files.pythonhosted.org/packages/47/a1/7a24868c15d84ed7446106d6c3d73807f58232a695452c0a29679e5a1523/typed_ast-1.4.0-cp37-cp37m-win_amd64.whl (155kB)
     |████████████████████████████████| 163kB 119kB/s
Installing collected packages: colorama, six, lazy-object-proxy, wrapt, typed-ast, astroid, isort, pylint
  Running setup.py install for wrapt ... done
  WARNING: The script isort.exe is installed in 'C:\Users\qinge\AppData\Roaming\Python\Python37\Scripts' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
  WARNING: The scripts epylint.exe, pylint.exe, pyreverse.exe and symilar.exe are installed in 'C:\Users\qinge\AppData\Roaming\Python\Python37\Scripts' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed astroid-2.2.5 colorama-0.4.1 isort-4.3.21 lazy-object-proxy-1.4.1 pylint-2.3.1 six-1.12.0 typed-ast-1.4.0 wrapt-1.11.2
```

# VSCode开发Python
> [用VSCode写python的正确姿势](https://www.cnblogs.com/0to9/p/6361474.html)
> [如何用VSCode愉快的写Python](https://www.cnblogs.com/pleiades/p/8146658.html)
> [VS Code 尝鲜之 配置Python开发环境](https://blog.csdn.net/u013205877/article/details/78883405)
> [Vs code + python3.7环境搭建](https://blog.csdn.net/qq_19342635/article/details/82147972)

打开插件窗口，看到Recommend和Popular就直接有Python，安装一个Python 2019.1.0用于python开发。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190223223045314.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
## reStructuredText
> [Sphinx 使用手册](https://zh-sphinx-doc.readthedocs.io/en/latest/rest.html)

安装Python软件，VSCode安装插件`restructuredText`，
```bash
$ pip install sphinx
$ pip install restructuredtext-lint
$ pip install recommonmark
$ pip3 install sphinx sphinx_rtd_theme recommonmark sphinx-markdown-tables sphinxemoji rst2pdf
```
焦点落入`.rst`文件的时候，vscode上方弹出对话框让你选择`conf.py`，不选择预览出现`TypeError: Cannot read property 'confPyDirectory' of undefined`错误，接着出现`ModuleNotFoundError: No module named 'recommonmark'`问题，都是因为电脑上安装了多个Python的原因。建立工程，生成的`index.html`在`_build`目录中，
```bash
$ sphinx-quickstart 
欢迎使用 Sphinx 3.2.1 快速配置工具。

Please enter values for the following settings (just press Enter to
accept a default value, if one is given in brackets).

Selected root path: .

You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> 独立的源文件和构建目录（y/n） [n]: 

The project name will occur in several places in the built documentation.
> 项目名称: Vitis-Tutorials
> 作者名称: Xilinx
> 项目发行版本 []: 1.0.0

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
https://www.sphinx-doc.org/en/master/usage/configuration.html#confval-language.
> 项目语种 [en]: 

创建文件 /home/qe/project/vitis/Vitis-Tutorials/docs/conf.py。
创建文件 /home/qe/project/vitis/Vitis-Tutorials/docs/index.rst。
创建文件 /home/qe/project/vitis/Vitis-Tutorials/docs/Makefile。
创建文件 /home/qe/project/vitis/Vitis-Tutorials/docs/make.bat。

完成：已创建初始目录结构。

You should now populate your master file /home/qe/project/vitis/Vitis-Tutorials/docs/index.rst and create other documentation
source files. Use the Makefile to build the docs, like so:
   make builder
where "builder" is one of the supported builders, e.g. html, latex or linkcheck.

qe@qe-pc:~/project/vitis/Vitis-Tutorials/docs$ cp conf.py conf.py.sphinx-quickstart
qe@qe-pc:~/project/vitis/Vitis-Tutorials/docs$ cp conf.py.bak conf.py
qe@qe-pc:~/project/vitis/Vitis-Tutorials/docs$ 
qe@qe-pc:~/project/vitis/Vitis-Tutorials/docs$ 
qe@qe-pc:~/project/vitis/Vitis-Tutorials/docs$ make html
```

