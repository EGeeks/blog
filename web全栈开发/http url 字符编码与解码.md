# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [URL参数中有中文的处理](https://blog.csdn.net/qq_25615395/article/details/80067118)
> [js对url进行编码和解码（三种方式区别）](https://www.cnblogs.com/z-one/p/6542955.html)
> [URL地址中的中文乱码问题的解决](https://blog.csdn.net/blueheart20/article/details/43766713)
> [Url中Json数据处理](https://www.jianshu.com/p/6d85fbf112fb)
> [C语言实现UrlEncode编码/UrlDecode解码](https://www.cnblogs.com/quliuliu2013/p/9915288.html)
> [C语言 HTTP中的chunked解码实现](https://www.cnblogs.com/dsblab/archive/2012/01/23/2328905.html)
> [Base64 编解码C语言实现](http://www.cnblogs.com/syxchina/archive/2010/07/25/2197388.html)
> [c语言的url转码解码](https://blog.csdn.net/gwq5210/article/details/42027491)
> [什么是Base64算法](https://blog.csdn.net/wufaliang003/article/details/79573512)
> [为什么请求时,需要使用URLEncode做encode转码操作](https://blog.csdn.net/u013833031/article/details/78828539)
> [axios默认对get请求进行了一次编码 导致url 两次编码](https://blog.csdn.net/dongheng123/article/details/118031949)

# 为什么需要url encode
url在参数值中出现&字符会截断参数，如果参数中就包含=或&这种特殊字符的时候，或者参数中有中文，就会导致参数值错误。如果前端使用axios，则默认对get请求进行编码，编码方法类似于escape方法。

# url javascript
## 方法
1. escape不编码` 0-9[a-Z] $ - _ . + ! * ' ( ) `，以及某些保留字，对url整体进行编码
2. encodeURI不编码`! @ # $ & * ( ) = : / ; ? + '`
3. encodeURIComponent同encodeURI，这种方法只对url的一部分进行编码

## 测试
在表单中传递json，采用escape和unescape，
```javascript
  methods: {
    onSubmit () {
      console.log('log: ' + JSON.stringify(this.form))
      this.$http.get('api/cgi-bin/helloworld.cgi?json=' + escape(JSON.stringify(this.form))
      ).then((response) => {
        console.info(unescape(response.body))
      }, (response) => {
        console.error(response)
      })
    }
  }
```
采用encodeURIComponent和decodeURIComponent
```javascript
  methods: {
    onSubmit () {
      console.log('log: ' + JSON.stringify(this.form))
      this.$http.get('api/cgi-bin/helloworld.cgi?json=' + encodeURIComponent(JSON.stringify(this.form))
      ).then((response) => {
        console.info(decodeURIComponent(response.body))
      }, (response) => {
        console.error(response)
      })
    }
  }
```
效果，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417220404839.png)
# url c语言
## 方法
参考一下代码，
```c
#include <stdio.h>
#include <string.h>
 
#define BUFSIZE 2048
 
int hex2dec(char c)
{
    if ('0' <= c && c <= '9')
    {
        return c - '0';
    }
    else if ('a' <= c && c <= 'f')
    {
        return c - 'a' + 10;
    }
    else if ('A' <= c && c <= 'F')
    {
        return c - 'A' + 10;
    }
    else
    {
        return -1;
    }
}
 
char dec2hex(short int c)
{
    if (0 <= c && c <= 9)
    {
        return c + '0';
    }
    else if (10 <= c && c <= 15)
    {
        return c + 'A' - 10;
    }
    else
    {
        return -1;
    }
}

void urlencode(char url[])
{
    int i = 0;
    int len = strlen(url);
    int res_len = 0;
    char res[BUFSIZE];
    for (i = 0; i < len; ++i)
    {
        char c = url[i];
        if (    ('0' <= c && c <= '9') ||
                ('a' <= c && c <= 'z') ||
                ('A' <= c && c <= 'Z') ||
                c == '/' || c == '.')
        {
            res[res_len++] = c;
        }
        else
        {
            int j = (short int)c;
            if (j < 0)
                j += 256;
            int i1, i0;
            i1 = j / 16;
            i0 = j - i1 * 16;
            res[res_len++] = '%';
            res[res_len++] = dec2hex(i1);
            res[res_len++] = dec2hex(i0);
        }
    }
    res[res_len] = '\0';
    strcpy(url, res);
}

void urldecode(char url[])
{
    int i = 0;
    int len = strlen(url);
    int res_len = 0;
    char res[BUFSIZE];
    for (i = 0; i < len; ++i)
    {
        char c = url[i];
        if (c != '%')
        {
            res[res_len++] = c;
        }
        else
        {
            char c1 = url[++i];
            char c0 = url[++i];
            int num = 0;
            num = hex2dec(c1) * 16 + hex2dec(c0);
            res[res_len++] = num;
        }
    }
    res[res_len] = '\0';
    strcpy(url, res);
}
```
## 测试
编写cgi，
```c
  fprintf(cgiOut, "Content-type:text/html\n\n");

  szGet = getenv("QUERY_STRING");
  if (szGet != NULL && strlen(szGet) > 0) {
    fprintf(cgiOut, "%s\n", szGet);
    fp = fopen("api.json", "w+");
    if (!fp) {
      fprintf(cgiOut, "fopen failed\n");
    }
    cgiFormString("json", json, 256);
    urldecode(json);
    if (fp)
      fprintf(fp, "%s\n", json);
    urlencode(json);
    fprintf(cgiOut, "cgi-json: ");
    cgiHtmlEscape(json);
    if (fp)
      fclose(fp);
  }
```
浏览器，
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019042122025970.PNG?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1podV9aaHVfMjAwOQ==,size_16,color_FFFFFF,t_70)
打开`api.json`，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190421220012184.PNG)
# base64
