# Nginx

Nginx （engine x 的缩写）是一个开源的 Web 和反向代理服务器。支持 HTTP，HTTPS 和电子邮件代理协议。

OpenRestry （Nginx 的一个发行版）是基于 Nginx 和 Lua 实现的 Web 应用网关，集成可大量的第三方模块。

## 安装 OpenRestry

- 添加 repo `yum-config-manager --add-repo https://openresty.org/package/centos/openresty.repo`
- 安装 `yum install openresty`

## 配置文件

- `/usr/local/openresty/nginx/conf/nginx.conf`，如果单独编译安装 nginx，应该是 `/usr/local/nginx/conf/nginx.conf`

```nginx
# 守护进程模式
daemon on;
# 配置启动用户
user admin admin;

# 主进程带子进程的模式
master_process on;
# 进程数，建议和 CPU 核数一致
# PS: tengine可以把设置成 auto, http://segmentfault.com/q/1010000000132550
worker_processes auto;
worker_cpu_affinity auto;

# 单个 worker 进程可以同时处理多少个文件,可以需要和 ulimit -a 对应
worker_rlimit_nofile 10240;

# worker进程的优先级，不能小于等于-5
worker_priority 0;

# 输出日志地址
error_log logs/error.log warn;
pid logs/nginx.pid;
lock_file logs/nginx.lock;

events {
    # 一个 worker 进程能够同时连接的数量
    worker_connections 10240;
    # event的模型类型;
    use epoll;
    # 启用一个接受互斥锁来打开套接字监听
    accept_mutex on;
    # 定义一个worker进程在尝试再次获取资源之前等待多久
    accept_mutex_delay 500ms;
    # 是否立刻接受从所有监听队列进入的连接
    multi_accept off;
}

http {
    include       mime.types;
    default_type  text/plain;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # 当资源没有找到时，是否log
    log_not_found off;

    # nginx将启用 sendfile内核来调用处理文件传递
    sendfile        on;
    # 配合sendfile，开启TCP_CORK的socket选项
    # nginx将尝试在单个tcp数据包中发送整个http响应头
    #tcp_nopush     on;

    # 需要完全禁用keepalive的话，将这个值设置成0
    keepalive_timeout  75; # 对应到apache的KeepAliveTimeout
    keepalive_requests 1000;  # 对应到apache的MaxKeepAliveRequests

    # 客户端两次读取操作的最大延迟, 对应apache的Timeout
    send_timeout 10;

    # hide nginx version,对等apache的ServerTokens
    server_tokens off;

    # 反向代理相关配置:
    # 包括传递给后端服务器的请求头信息，关闭对相应中的Location头信息和Refresh头信息做文本替换，以及设置缓冲区大小
    # 转发到后端服务器的请求中的Host HTTP头默认为代理的主机名,这样的设置令nginx可以换而使用客户端请求中的原始主机名
    proxy_set_header        Host $host;
    # 让后端机器得到真实的IP
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        Web-Server-Type nginx;
    proxy_set_header        WL-Proxy-Client-IP $remote_addr;
    # 确保用于套接字通讯的IP地址
    proxy_set_header        X-Forwarded-For    $proxy_add_x_forwarded_for;
    # 让nginx以“ as it is”方式重定向到客户端，对响应本身不做处理
    proxy_redirect          off;
    # 是否缓冲后端服务器的响应
    proxy_buffering on;
    # 设置缓冲数量大小，用于存放从后端服务器读取的响应数据
    # 128 8k的意思是128个缓冲，每个缓冲8k
    # TODO 这个需要测试
    proxy_buffers           256 32k;
    # 缓冲区的大小，得和上面的proxy_buffers对应
    proxy_buffer_size 32k;
    # 超过多少后缓冲去刷新，通常配置成proxy_buffer_size * 2
    proxy_busy_buffers_size 64k;
    # 代理的超时
    # 连接超时
    proxy_connect_timeout 60;
    # 读取超时
    proxy_read_timeout 60;
    # 发送超时
    proxy_send_timeout 60;
    # 如果客户端放弃请求，那么nginx也放弃
    proxy_ignore_client_abort off;
    # 当后端错误时，根据后端的返回码，来匹配自身的error_page的值
    proxy_intercept_errors  on;

    # URL 重定向为： server_name 中的第一个域名 + 目录名 + /
    # 如果是off的话： 原 URL 中的域名 + 目录名 + /
    server_name_in_redirect on;

    # 限制客户端请求体的最大值
    # TODO 这个需要再确认一下
    client_max_body_size 20M;
    # 定义用于持有request body内存缓冲的大小。超过该大小，内容将被保存到临时文件中
    # TODO 这个也需要再确认
    client_body_buffer_size 128k;

    # gzip
    gzip on;
    gzip_min_length 1k;  # 大于多少才压缩
    gzip_http_version       1.0;   # 用了反向代理的话，末端通信是HTTP/1.0
    gzip_buffers 4 16k;
    gzip_comp_level 2;  # 压缩级别，9是最大
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/rss+xml application/json;
    gzip_vary off;  # on的话会在Header里增加"Vary: Accept-Encoding"
    gzip_disable     "MSIE [1-6]\.";
    #添加 weight 字段可以表示权重，值越高权重越大，默认值是 1
    upstream neo_proxy {
        server 127.0.0.1:3005 weight=1;
        server 127.0.0.1:3004 weight=1;
    }
    server {
        listen       3003;
        server_name  neo.benditoutiao.com;
        access_log  logs/neo_benditoutiao_com_access.log  main;

       # 跳转到实际的服务
       location / {
           proxy_pass http://neo_proxy/;
       }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

}
```
