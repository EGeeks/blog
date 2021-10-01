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
> [详解nginx websocket配置](https://www.cnblogs.com/gao88/p/11846928.html)
> [Nginx支持WebSocket反向代理-学习小结](https://www.cnblogs.com/kevingrace/p/9512287.html)

# rtmp流媒体服务器
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

# websocket
```bash
http{
 map $http_upgrade $connection_upgrade { #根据客户端请求中$http_upgrade 的值，来构造改变$connection_upgrade的值
  default upgrade;
  ''   close;
 }
 upstream websocket {
  #ip_hash;
  server localhost:8010; 
  server localhost:8011;
 }
# 以下配置是在server上下文中添加，location指用于websocket连接的path。
 server {
  listen    80;
  server_name localhost;
  access_log /var/log/nginx/yourdomain.log;
  location / {
   proxy_pass http://websocket;
   proxy_read_timeout 300s;
   proxy_set_header Host $host;
   proxy_set_header X-Real-IP $remote_addr;
   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   proxy_http_version 1.1;
   proxy_set_header Upgrade $http_upgrade;
   proxy_set_header Connection $connection_upgrade;
  }
 }
}
```

