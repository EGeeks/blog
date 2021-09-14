# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [fastcgi官网](https://fastcgi-archives.github.io/)
> [使用VS2010的nmake命令编译MakeFile流程](https://blog.csdn.net/xiaoluer/article/details/56008921)
> [visual studio 2017 Community nmake](https://blog.csdn.net/yw_chiina/article/details/81328781)
> [Fastcgi](https://blog.csdn.net/sdoyuxuan/article/details/81175165)
> [fastcgi c/c++ API 说明](https://www.cnblogs.com/dj0325/p/9185614.html)
> [c 语言写的fastcgi 程序](https://blog.csdn.net/qzier_go/article/details/7340868)
> [用C语言开发FastCGI应用程序——fcgi_stdio包API](https://blog.csdn.net/wangqing_12345/article/details/52171444)
> [fcgi程序两种编写风格](https://blog.csdn.net/nyist327/article/details/43792753)
> [FastCGI+lighttpd开发之介绍和环境搭建](https://segmentfault.com/a/1190000004006596)
> [nginx+spawn-fcgi+demo+fcgi库函数](https://blog.csdn.net/yockie/article/details/52065888)
> [FCGI个人学习记录](https://blog.csdn.net/seucbh84/article/details/10067631)
> [HttpFcgi模块](https://www.nginx.cn/doc/standard/httpfcgi.html)

# 使用
fastcgi官网迁移到了github了，下载[FastCGI Developer's Kit](https://github.com/FastCGI-Archives/fcgi2)，编译，
```shell
$ ./autogen.sh
$ ./configure --prefix=/cygdrive/c/dog/program/cgi/fcgi2-2.4.2/install LDFLAGS=-L/lib/w32api
$ make
$ make install
$ ls install
bin  include  lib
$ ls install/bin/
cgi-fcgi.exe  cygfcgi++-0.dll  cygfcgi-0.dll
$ ls install/include/
fastcgi.h  fcgi_config.h  fcgi_stdio.h  fcgiapp.h  fcgimisc.h  fcgio.h  fcgios.h
$ ls install/lib
libfcgi.a  libfcgi.dll.a  libfcgi.la  libfcgi++.a  libfcgi++.dll.a  libfcgi++.la  pkgconfig
$ ls examples/ | grep exe
authorizer.exe
echo.exe
echo-cpp.exe
echo-x.exe
log-dump.exe
size.exe
threaded.exe
```
配置nginx.conf，
```shell
location = /cmd {
	fastcgi_pass 127.0.0.1:8088;
	fastcgi_index index.cgi;
	include fastcgi.conf;
}
```
执行，
```shell
$ ../spawn-fcgi-1.6.4/src/spawn-fcgi.exe -a 127.0.0.1 -p 8088 -f examples/echo.exe -n
$ spawn-fcgi: child spawned successfully: PID: 282
$ kill 282
```
浏览器输入`http://localhost/cmd`，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190312225305961.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
关注一下html输出的这些环境变量，开发一些特殊功能时可能会用到，
```html
<title>FastCGI echo</title><h1>FastCGI echo</h1>
Request number 2,  Process ID: 293<p>
No data from standard input.<p>
Request environment:<br>
<pre>
FCGI_ROLE=RESPONDER
SCRIPT_FILENAME=C:\dog\software\nginx-1.14.2/html/cmd
QUERY_STRING=
REQUEST_METHOD=GET
CONTENT_TYPE=
CONTENT_LENGTH=
SCRIPT_NAME=/cmd
REQUEST_URI=/cmd
DOCUMENT_URI=/cmd
DOCUMENT_ROOT=C:\dog\software\nginx-1.14.2/html
SERVER_PROTOCOL=HTTP/1.1
REQUEST_SCHEME=http
GATEWAY_INTERFACE=CGI/1.1
SERVER_SOFTWARE=nginx/1.14.2
REMOTE_ADDR=127.0.0.1
REMOTE_PORT=51161
SERVER_ADDR=127.0.0.1
SERVER_PORT=80
SERVER_NAME=localhost
REDIRECT_STATUS=200
HTTP_HOST=localhost
HTTP_CONNECTION=keep-alive
HTTP_CACHE_CONTROL=max-age=0
HTTP_UPGRADE_INSECURE_REQUESTS=1
HTTP_USER_AGENT=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.103 Safari/537.36
HTTP_ACCEPT=text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
HTTP_ACCEPT_ENCODING=gzip, deflate, br
HTTP_ACCEPT_LANGUAGE=zh-CN,zh;q=0.9
</pre><p>
Initial environment:<br>
<pre>
USERDOMAIN=DESKTOP-4KV9JL3
OS=Windows_NT
COMMONPROGRAMFILES=C:\Program Files\Common Files
PROCESSOR_LEVEL=6
PSModulePath=C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
CommonProgramW6432=C:\Program Files\Common Files
CommonProgramFiles(x86)=C:\Program Files (x86)\Common Files
LANG=zh_CN.UTF-8
TZ=Asia/Shanghai
HOSTNAME=DESKTOP-4KV9JL3
PUBLIC=C:\Users\Public
OLDPWD=/cygdrive/c/dog/program/cgi/fcgi2-2.4.2/bin
USERNAME=qinge
LOGONSERVER=\\DESKTOP-4KV9JL3
PROCESSOR_ARCHITECTURE=AMD64
LOCALAPPDATA=C:\Users\qinge\AppData\Local
COMPUTERNAME=DESKTOP-4KV9JL3
FPS_BROWSER_APP_PROFILE_STRING=Internet Explorer
USER=qinge
!::=::\
SYSTEMDRIVE=C:
USERPROFILE=C:\Users\qinge
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
SYSTEMROOT=C:\Windows
USERDOMAIN_ROAMINGPROFILE=DESKTOP-4KV9JL3
PROCESSOR_IDENTIFIER=Intel64 Family 6 Model 61 Stepping 4, GenuineIntel
NVM_SYMLINK=C:\Program Files\nodejs
PWD=/cygdrive/c/dog/program/cgi/fcgi2-2.4.2/bin/bin
HOME=/home/qinge
TMP=/tmp
OneDrive=C:\Users\qinge\OneDrive
PROCESSOR_REVISION=3d04
FPS_BROWSER_USER_PROFILE_STRING=Default
PROFILEREAD=true
NUMBER_OF_PROCESSORS=4
ProgramW6432=C:\Program Files
COMSPEC=C:\Windows\system32\cmd.exe
APPDATA=C:\Users\qinge\AppData\Roaming
SHELL=/bin/bash
TERM=xterm
WINDIR=C:\Windows
NVM_HOME=C:\Users\qinge\AppData\Roaming\nvm
ProgramData=C:\ProgramData
SHLVL=1
MINTTY_SHORTCUT=/cygdrive/c/Users/Public/Desktop/Cygwin64 Terminal.lnk
PRINTER=OneNote
PROGRAMFILES=C:\Program Files
ALLUSERSPROFILE=C:\ProgramData
TEMP=/tmp
NO_XILINX_DATA_LICENSE=HIDDEN
DriverData=C:\Windows\System32\Drivers\DriverData
SESSIONNAME=Console
ProgramFiles(x86)=C:\Program Files (x86)
PATH=/cygdrive/c/dog/program/cgi/fcgi2-2.4.2/libfcgi/.libs:/cygdrive/c/dog/program/cgi/fcgi2-2.4.2/libfcgi/.libs:/cygdrive/c/dog/program/cgi/fcgi2-2.4.2/bin/lib:/cygdrive/c/dog/program/cgi/fcgi2-2.4.2/bin/bin:/usr/local/bin:/usr/bin:/cygdrive/c/Windows/system32:/cygdrive/c/Windows:/cygdrive/c/Windows/System32/Wbem:/cygdrive/c/Windows/System32/WindowsPowerShell/v1.0:/cygdrive/c/Windows/System32/OpenSSH:/cygdrive/c/Program Files/Git/cmd:/cygdrive/c/Program Files (x86)/STMicroelectronics/STM32 ST-LINK Utility/ST-LINK Utility:/cygdrive/c/Users/qinge/AppData/Roaming/nvm:/cygdrive/c/Program Files/nodejs:/cygdrive/c/Users/qinge/AppData/Local/Microsoft/WindowsApps:/cygdrive/c/Users/qinge/AppData/Roaming/nvm:/cygdrive/c/Program Files/nodejs:/cygdrive/c/Users/qinge/AppData/Local/Programs/Microsoft VS Code/bin:/cygdrive/c/Users/qinge/AppData/Local/Programs/Python/Python37:/cygdrive/c/Users/qinge/AppData/Local/Programs/Python/Python37/Scripts
PS1=\[\e]0;\w\a\]\n\[\e[32m\]\u@\h \[\e[33m\]\w\[\e[0m\]\n\$ 
HOMEDRIVE=C:
INFOPATH=/usr/local/info:/usr/share/info:/usr/info
HOMEPATH=\Users\qinge
ORIGINAL_PATH=/cygdrive/c/Windows/system32:/cygdrive/c/Windows:/cygdrive/c/Windows/System32/Wbem:/cygdrive/c/Windows/System32/WindowsPowerShell/v1.0:/cygdrive/c/Windows/System32/OpenSSH:/cygdrive/c/Program Files/Git/cmd:/cygdrive/c/Program Files (x86)/STMicroelectronics/STM32 ST-LINK Utility/ST-LINK Utility:/cygdrive/c/Users/qinge/AppData/Roaming/nvm:/cygdrive/c/Program Files/nodejs:/cygdrive/c/Users/qinge/AppData/Local/Microsoft/WindowsApps:/cygdrive/c/Users/qinge/AppData/Roaming/nvm:/cygdrive/c/Program Files/nodejs:/cygdrive/c/Users/qinge/AppData/Local/Programs/Microsoft VS Code/bin:/cygdrive/c/Users/qinge/AppData/Local/Programs/Python/Python37:/cygdrive/c/Users/qinge/AppData/Local/Programs/Python/Python37/Scripts
EXECIGNORE=*.dll
_=./cgi-fcgi.exe
BIN_SH=xpg4
DUALCASE=1
</pre><p>
```
# 表单
axios发送表单，

```js
    onSubmit () {
      console.log('log: ' + JSON.stringify(this.form))
      this.$http.get('api/cmd?json=' + encodeURIComponent(JSON.stringify(this.form))
      ).then((response) => {
        console.info(decodeURIComponent(response.body))
      }, (response) => {
        console.error(response)
      })
    }
```
表单在环境变量的QUERY_STRING中，测试，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190427230456269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
# 读写文件
读写文件请按如下方式打开，使用输入输出流不行，待定位。
```c
open(fileNamePtr, O_RDONLY, (S_IRGRP | S_IROTH | S_IRUSR));
open(fileNamePtr, O_WRONLY | O_CREAT, (S_IWGRP | S_IWOTH | S_IWUSR));
```

