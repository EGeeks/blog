# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [IPv6 (getaddrinfo & inet_ntop)](http://mingw-users.1079350.n2.nabble.com/IPv6-getaddrinfo-amp-inet-ntop-td5891996.html)
> [mingw中的socket基础](http://blog.chinaunix.net/uid-23065002-id-4557735.html)
> [windows下linux下socket编程区别](https://www.cnblogs.com/songhexiang/p/6682247.html)
> [【坑】winsock2.h和windows.h的include顺序](https://www.jianshu.com/p/2e2acfd9e51f)

# Please include winsock2.h before windows.h
自己的源文件使用了`winsock2.h`，而Qt的头文件里包含了`windows.h`，所以把自己的头文件放到最前面即可。

# 使用
服务端，
```c
#if defined (WIN32)
    SOCKET sockfd;
#elif defined(__linux__)
    int sockfd;
#endif
	struct addrinfo hints, *servinfo, *p;
	struct sockaddr_in sockaddr;
	unsigned int len;
	int yes = 1;
	int rv;
	char PORT[6];
	sprintf(PORT, "%d", server->port_number);
	memset(&hints, 0, sizeof hints);
	hints.ai_family = AF_UNSPEC;
	hints.ai_socktype = SOCK_STREAM;
	hints.ai_flags = AI_PASSIVE; // use my IP

	if ((rv = getaddrinfo(NULL, PORT, &hints, &servinfo)) != 0) {
#if defined (WIN32)
        fprintf(stderr, "getaddrinfo: WSAGetLastError=%d\n",  WSAGetLastError());
#elif defined(__linux__)
        fprintf(stderr, "getaddrinfo: gai_strerror=%s\n", gai_strerror(rv));
#endif
		return 1;
	}

// loop through all the results and bind to the first we can
	for (p = servinfo; p != NULL; p = p->ai_next) {
		if ((sockfd = socket(p->ai_family, p->ai_socktype, p->ai_protocol))
				== -1) {
			perror("server: socket");
			continue;
		}

		if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(int))
				== -1) {
			perror("setsockopt");
			exit(1);
		}

		if (bind(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
#if defined (WIN32)
            closesocket(sockfd);
#elif defined(__linux__)
            close(sockfd);
#endif
			perror("server: bind");
			continue;
		}

		len = sizeof(sockaddr);
        if (getsockname(sockfd, (struct sockaddr *) &sockaddr, &len) != 0) {
#if defined (WIN32)
            closesocket(sockfd);
#elif defined(__linux__)
            close(sockfd);
#endif
			perror("server: getsockname");
			continue;
		}
		server->port_number = ntohs( sockaddr.sin_port );

		break;
	}

	if (p == NULL) {
		fprintf(stderr, "server: failed to bind\n");
		return 2;
	}

	freeaddrinfo(servinfo); // all done with this structure

	if (listen(sockfd, 5) == -1) {
		perror("listen");
		exit(1);
	}
	if (server->debug_level)
		printf("server: waiting for connections...\n");

#if defined (NET_USE_PTHERAD)
    server->fd = sockfd;
    server->run = 1;
    int ret;
    pthread_t tid;
    ret = pthread_create(&tid, NULL, accept_cb, server);
    if(ret != 0)
    {
        printf("%s failed\n", __FUNCTION__);
        return ret;
    }
#elif defined (NET_USE_LIBEV)
    ev_io_init(&server->listen_watcher, accept_cb, sockfd, EV_READ);

    server->listen_watcher.data = server;
    ev_io_start(server->loop, &server->listen_watcher);
#endif

    return 0;
```

