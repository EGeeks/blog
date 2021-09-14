# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [搭建Nginx+nginx-rtmp-module的hls流媒体服务器并用OBS进行推流](https://blog.csdn.net/Ricardo18/article/details/89359623)
> [Nginx-RTMP服务搭建](https://www.jianshu.com/p/f4508c4e6a90?utm_campaign=haruki)
> [基于nginx的rtmp直播服务器（nginx-rtmp-module实现）](https://www.cnblogs.com/zhangmingda/p/12638985.html)
> [流媒体协议介绍（rtp/rtcp/rtsp/rtmp/mms/hls）](https://mp.weixin.qq.com/s/3Xi1vntJkEodKnmSFv9ncw)

# rtmp
下载[rtmp](https://github.com/arut/nginx-rtmp-module)，为hi3531d交叉编译uclib版本的nginx，更新配置文件，
```bash
rtmp {  
  
    server {  
  
        listen 1935;  #监听的端口
  
        chunk_size 4000;  
        
        application hls {
            live on;
            hls on;
            hls_path /usr/local/html/hls;#视频流存放地址
            hls_fragment 5s;
            hls_playlist_length 15s;
            hls_continuous on; #连续模式。
            hls_cleanup on;    #对多余的切片进行删除。
            hls_nested on;     #嵌套模式。
        }
    }  
}
```

