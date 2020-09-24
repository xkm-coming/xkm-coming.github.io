---
layout: post
title:  "openresty"
date:   2020-09-24 17:01:06 +0800
categories: jekyll update
---
### 一、基础知识准备

#### 1、进程：

一个程序在一个数据集中的一次动态执行过程，它是系统进行资源分配和调度的基本单位

#### 2、线程：

是独立运行和独立调度的基本单位，一个进程中可以包含若干个线程（一个线程挂掉整个进程挂掉），它们可以利用进程所拥有的资源，让一个进程执行多个任务。当在程序中创建多线程的时候，同一时刻多个线程是同时执行的，不过实质上多个线程是并发的，因为只有一个CPU，所以同一个时刻只会有一个线程在执行。在一个时间片内哪个线程执行时不确定的，可通过控制线程的优先级，不过真正的线程调度是由CPU来决定的。

#### 3、协程：

协程是指协作程序，协程之间通过协作（函数调用）来完成一个既定的任务，协程不是被操作系统内核所管理，而完全是由程序所控制（也就是在用户态执行），协程在子程序内部是可中断的，然后转而执行别的子程序，在适当的时候再返回来接着执行，比多线程模式更节约开销

#### 4、Nginx：

（1)nginx工作模式：一般来说，nginx是以多进程方式工作的，nginx在启动后，会有一个master进程和多个worker进程（由master进程fork而来），master进程主要用来管理worker进程，worker进程处理客户端请求，一个worker进程处理一个进程请求

（2)异步非阻塞

（3）nginx在负载均衡、反向代理、代理缓存、限流等场景中发挥着不可小觑的作用，但nginx作为一个web容器使用的还不是那么广泛，其原因如下：

- 需要用c/c++开发，学习和开发成本高

- 需要熟悉nginx的源代码，需要遵循一系列复杂的规则

#### 5、nginx的http request的处理过程：

（1）`NGX_HTTP_POST_READ_PAHASE`：读取请求内容阶段，在 Nginx 读取并解析完请求头之后就立即开始运行， ngx_realip标准模块的指令处理在这个阶段 

（2）`NGX_HTTP_SERVER_REWRITE_PHASE`：请求地址重写阶段，，当 ngx_rewrite 模块的配置指令在 server 配置块中时是运行在 server-rewrite 阶段

（3）`NGX_HTTP_FIND_CONFIG_PHASE`：配置查找阶段，Nginx 核心来完成当前请求与 location 配置块之间的配对工作

（4）`NGX_HTTP_REWRITE_PHASE`：请求地址重写阶段，ngx_rewrite 模块的指令用于 location 块中时是运行在这个 rewrite 阶段

（5）`NGX_HTTP_POST_REWRITE_PHASE`：请求地址重写提交阶段

（6）`NGX_HTTP_PREACCESS_PHASE`：访问权限检查准备阶段

（7）`NGX_HTTP_ACCESS_PHASE`：访问权限检查阶段，ngx_auth_request 模块的指令在这个阶段处理：

```
location /hello {

	allow 127.0.0.1;
	deny all;

    echo "hello world";
}
```

输出结果：

```
$ curl 'http://localhost:8080/hello'
	hello world
```

原因分析： ngx_access 模块自己的多条配置指令之间是按顺序执行的，直到遇到第一条满足条件的指令就不再执行后续的 allow 和 deny 指令。如果首先匹配的指令是 allow，则会继续执行后续其他模块的指令或者跳到后续的处理阶段；而如果首先满足的是 deny 则会立即中止当前整个请求的处理，并立即返回 403 错误

（8）`NGX_HTTP_POST_ACCESS_PHASE`：访问权限检查提交阶段

（9）`NGX_HTTP_TRY_FILES_PHASE`：配置项try_files处理阶段

（10）`NGX_HTTP_CONTENT_PHASE`：内容产生阶段，运行在这个阶段的配置指令一般都肩负着生成“内容”并输出 HTTP 响应的使命，echo指令、echo_exec 指令、proxy_pass指令等都是在这个阶段处理。如果一个 location 中未使用任何 content 阶段的指令，即没有模块注册“内容处理程序”时，那么会把当前请求的 URI 映射到文件系统的静态资源服务模块，按照它们在 content 阶段的运行顺序，依次是 ngx_index 模块， ngx_autoindex 模块，以及 ngx_static 模块：

- **ngx_index 模块**：主要用于在文件系统目录中自动查找指定的首页文件，如 index.html 和 index.htm 

```
location / {
	root /var/www/;
	index index.htm index.html;
	//先获取/var/www/index.htm资源，如果没有再去获取/var/www/index.html资源
	//ngx_index 模块一旦找到了 index 指令中列举的文件之后，就会发起这样的“内部跳转”，仿佛用户是直接请求的这个文件所对应的 URI 一样
}

location /index.html {
	set $a 32;
	echo "a = $a";
}
```

输出结果：

```
$ curl 'http://localhost:8080/'
a = 32
//跳转到/index.htm后重新执行11个阶段
```

- **ngx_autoindex 模块**：可以用于自动生成这样的“目录索引”网页
- **ngx_static 模块**：主要实现服务静态文件的功能， ngx_index 模块发起“内部跳转”，ngx_static把相应的首页文件服务出去（即把该文件的内容作为响应体数据输出，并设置相应的响应头）

（11）`NGX_HTTP_LOG_PHASE`：日志模块处理阶段



### 二、openresty

章亦春将nginx核心、LuaJIT、ngx_lua模块（由Lua和Nginx粘合而成）、许多有用的Lua库和常用的第三方Nginx模块组合在一起成为OpenResty。开发人员安装OpenRestory，使用Lua编写脚本，部署到Nginx Web容器中运行，可以很轻松开发出Web服务

#### 1、ngx_lua的原理：

- 基于Nginx核心为基础，完全运行于Nginx服务内部中。运行稳定，轻量级，内存占用少。
-  Lua 代码既可以直接内联在 Nginx 配置文件中，也可以单独放置在外部 ”*.lua“源码文件（或者 Lua 字节码文件）里，然后在 Nginx 配置文件中引用这些文件的路径。
- 利用lua内建协程，调用异步API。每个worker进程都有一个Lua解释器或者LuaJIT，当在设备操作时若不能获取资源时，协程会被挂起，在异步callback事件到来时，唤醒该协程，继续执行，从而实现全异步的nginx机制，I/O操作都委托给nginx事件模型，实现完全的非阻塞调用。对于每个用户请求，ngx_lua会唤醒一个协程用于执行用户代码处理请求，当请求处理完成这个协程会被销毁

#### 2、openresty组件：

`nginx`：一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务

`luaJIT`：是Lua编译语言的即时编译器

`lua-resty-dns`：为ngx-lua nginx模块提供了DNS解析器

`lua-resty-mysql`：为ngx_lua nginx模块的MySQL客户端驱动程序

`lua-resty-mencached`：

`lua-resty-redis`：是ngx_lua nginx模块的Redis客户端驱动程序

`lua-resty-limit-traffic`：用于限制和控制Openresty/ngx_lua中流量的lua库

`lua nginx module`：将LuaJIT嵌入到nginx内核中，并通过nginx子请求将功能强大的Lua线程集成到nginx事件模型中

#### 3、Lua Nginx Module指令的执行顺序：

（1）loading-config：用于初始化全局配置/预加载lua模块

- int_by_lua：在Master 进程创建Worker进程时，此指令中加载的全局变量会进行Copy-OnWrite，即会复制到所有全局变量到 Worker进程（加在http下）
- init_by_lua_block
- init_by_lua_file

（2）starting-worker：用于定时拉取配置/数据，或者上游服务的健康检查

- init_worker_by_lua：用于启动一些定时任务，比如心跳检查，定时拉取服务器配置等等；此处的任务是跟Worker进程数量有关系 的，比如有2个Worker进程那么就会启动两个完全一样的定时任务，可搭配以下命令一起使用：

  ```
  lua_max_pending_timers 1024; #最大等待任务数
  lua_max_running_timers 256; #最大同时运行任务数
  ```

- init_worker_by_lua_block

- init_worker_by_lua_file

（3）right-before-ssl：对ssl做特殊处理

- ssl_certificate_by_lua_block
- ssl_certificate_by_lua_file

（4）rewrite：设置nginx变量，可以实现复杂的赋值逻辑

- set_by_lua
- set_by_lua_block
- set_by_lua_file

（5）rewirte tail：实现复杂的转发、重定向、缓存等功能

- rewrite_by_lua
- rewrite_by_lua_block
- rewrite_by_lua_file

（6）access tail：IP准入、接口权限等情况集中处理（配合iptable完成简单防火墙）

- access_by_lua
- access_by_block
- access_by_lua_file

（7）content：动态负载均衡

- balancer_by_lua_block
- valancer_by_lua_file

（8）content：内容处理器，接受请求处理并输出响应

- content_by_lua
- content_by_lua_block
- content_by_lua_file

（9）output-header-filter：设置返回的header和cookie

- header_filter_by_lua
- header_filter_by_lua_block
- header_filter_by_lua_file

（10）output-body-filter：对响应数据进行过滤

- body_filter_by_lua
- body_filter_by_lua_block
- body_filter_by_lua_file

（11）log：机录访问量，统计平均响应时间

- log_by_lua
- log_by_lua_block
- log_by_lua_file

#### 4、openresty命令

跟nginx的命令的使用方式一样

/usr/local/openresty/nginx/sbin/nginx -t：检查配置文件

/usr/local/openresty/nginx/sbin/nginx -s stop|reload|restart：暂停|重载|重启

#### 5、openresty的重载和重启

openresty的工作方式是多进程，

- openresty -s reload：重新加载配置并启动openresty，生成新的nginx进程，给master进程发送信息，让master杀死worker进程，并重新启动新一轮工作

- openresty -s restart：给master进程发送信号，把原本暂停状态的worker进程重新启动



#### 三、nginx配置文件：

##### 1、变量

- 用户自定义变量
- Nginx内建变量

（1）Nginx变量只能存放在一种类型的值，那就是字符串

- 变量表示方式：$value_name

- 输出变量的方式：echo "$a"、echo "foo=[$foo]"、echo "${a}hello"

      server {
          listen 8080;
          
          location /test {
              set $foo hello;
              echo "foo: $foo"; 
              //使用第三方 ngx_echo 模块的 echo 配置指令将 $foo 变量的值作为当前请求的响应体输出
          }
      }
  如果想输出 `$` 这样的特殊字符，可以通过不支持“变量插值”的模块配置指令专门构造出取值为 `$` 的 Nginx 变量：

      geo $dollar {
              default "$";
      }
      //通过标准模块 ngx_geo 提供的配置指令 geo 来为变量 $dollar 赋予字符串 "$"
      
      server {
          listen 8080;
      
          location /test {
              echo "This is a dollar sign: $dollar";
          }
      }

- 变量创建：在Nginx配置加载时创建变量，变量一旦被创建，其变量名的课件范围就是整个Nginx配置，但每个请求都有所有变量的独立副本，或者说都有各变量用来存放值的容器的独立副本，彼此互不干扰，例如：如果我给/foo接口下给$foo赋值，不影响/bar接口下$foo的值，/bar接口下未给$foo赋值前，$foo为空字符

- 变量赋值：在请求实际处理时进行变量赋值

（2）内部跳转

- 使用echo_exec 配置指令进行内部跳转

  ```
  server {
  	listen 8080;
          
  	location /foo {
  		set $a hello;
  		echo_exec /bar;
  		//使用第三方模块 ngx_echo 提供的 echo_exec 配置指令，发起到 location /bar 的“内部跳转”，当前正在处理的请求不变，还是原来的那一套 Nginx 变量的容器副本，只是当前的location发生了变化
      }
  
      location /bar {
          echo "a = [$a]";
      }
  }
  ```

处理流程：先在 location /foo 中通过 set 指令将 $a 变量的值赋为字符串 hello，然后通过 echo_exec 指令发起内部跳转，又进入到 location /bar 中，再输出 $a 变量的值（输出$a 为"hello"）

- 使用rewrite 配置指令进行内部跳转

      server {
      	listen 8080;
              
      	location /foo {
              set $a hello;
              rewrite ^ /bar;
          }
      
          location /bar {
              echo "a = [$a]";
          }
      }

（3）常用的Nginx内建变量

Nginx内置变量存放在ngx_http_core_module模块中，对内建变量进行改写会影响到其他模块

- 从请求行解析到的变量

  （1）$arg_name：请求行中name参数的值

  在配置文件中location里可以使用$arg_name获取URL请求中的name参数值

  ```
  location /test {
          echo "name: $arg_name";
          echo "class: $arg_class";
      }
  ```

  在请求过程中，可以用"?"赋值

  ```
  $ curl 'http://localhost:8080/test?name=Tom&class=3'
  ```

  输出结果：

  ```
  name: Tom
  class: 3
  ```

  （2）$args：请求行中的参数

  （3）$request：整个请求行

  ```
   location /test {
       echo "request = $request_uri";
   }
  ```

  发起请求：

  ```
  $ curl 'http://localhost:8080/test?a=3&b=4'
  ```

  输出结果：

  ```
  request_uri =GET /test?a=3&b=4 HTTP/1.1
  ```

  （4）$request_method：请求方法

  （5）$request_uri：完整的请求URI

  ```
  location /test {
       echo "request_uri = $request_uri";
   }
  ```

  发起请求：

  ```
  $ curl 'http://localhost:8080/test?a=3&b=4'
  ```

  输出结果：

  ```
  request_uri = /test?a=3&b=4
  ```

  （6）$uri：URI，除去查询字符串

  （7）$document_uri：同$uri

  （8）$query_string：同$args

  （9）$server_protocol：请求协议（如HTTP/1.0 HTTP/1.1)

- 从请求头中解析到的变量

  （1）$host：该变量按如下优先级获得：请求行中解析到的host、请求头“Host”中的host、配置文件中匹配到的server_name

  （2）$remote_addr：客户端ip地址

  （3）$remote_port：客户端端口

  （4）$http_user_agent：用户代理

  （5）$http_cookie：“Cookie”请求头的值

  （6）$cookie_name：Cookie中名为name的值

  （7）$http_referer：Http-Referer”请求头的值 

- 其他

  （1）$status：http响应状态码

  （2）$request_time：请求处理时间

  （3）$upstream_response_time：从与upstream建立连接到收到最后一个字节所经历的时间（nginx做反向代理服务器时可用）

  （4）$upstream_connect_time：与upstream建立连接所消耗的时间（nginx做反向代理服务器时可用）

（4）变量的两种存放方式

- 储存在一个全局的 hash 表里，使用该变量时需要进行查询

- 储存在一个全局动态数组里，每个变量存在一个唯一的索引

像"$args"这样具有无数变种的变量群，是“未索引的”。当读取这样的变量时，其实是它的“取处理程序”在起作用，即实时扫描当前请求的 URL 参数串，提取出变量名所指定的 URL 参数的值

（5）变量可缓存

Nginx 变量也可以选择将其值容器用作缓存，这样在多次读取变量的时候，就只需要调用“取处理程序”计算一次，像ngx_map模块，标准的ngx_geo等模块也一样使用了变量值的缓存机制

（6）父子间请求变量共享

##### 2、location的正则匹配

修饰符：

- =：精确匹配，请求url和定义的字符串完全相等
- ~：正则匹配
- ~*：正则匹配，不区分大小写
- ^~：最佳匹配

匹配规则：

（1）找到使用前缀字符定义的location，选择最长匹配

（2）如果找到精确匹配的location就结束查找，使用精确匹配

（3）如果没有精确匹配，就按顺序查找正则匹配，找到则结束查找，使用正则匹配

（4）如果没有正则匹配则使用步骤（1）中的最长匹配

##### 3、nginx配置文件

```
#nginx进程数，建议设置为等于CPU总核心数。
worker_processes 8;

#全局错误日志定义类型，（从低到高）[ debug | info | notice | warn | error | crit ]
error_log /var/log/nginx/error.log info;

#进程文件
pid /var/run/nginx.pid;

#一个nginx进程打开的最多文件描述符数目，理论值应该是最多打开文件数（系统的值ulimit -n）与nginx进程数相除，但是nginx分配请求并不均匀，所以建议与ulimit -n的值保持一致。
worker_rlimit_nofile 65535;

#工作模式与连接数上限
events
{
    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll模型是Linux 2.6以上版本内核中的高性能网络I/O模型，如果跑在FreeBSD上面，就用kqueue模型。
    use epoll;
    
    #单个进程最大连接数（最大连接数=连接数*进程数）
    worker_connections 65535;
}

#设定http服务器
http
{
    include mime.types; #文件扩展名与文件类型映射表
    default_type application/octet-stream; #默认文件类型
    #charset utf-8; #默认编码
    server_names_hash_bucket_size 128; #服务器名字的hash表大小
    client_header_buffer_size 32k; #上传文件大小限制
    large_client_header_buffers 4 64k; #设定请求缓
    client_max_body_size 8m; #设定请求缓
    sendfile on; #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成off。
    autoindex on; #开启目录列表访问，合适下载服务器，默认关闭。
    tcp_nopush on; #防止网络阻塞
    tcp_nodelay on; #防止网络阻塞
    keepalive_timeout 120; #长连接超时时间，单位是秒

    #FastCGI相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    #gzip模块设置
    gzip on; #开启gzip压缩输出
    gzip_min_length 1k; #最小压缩文件大小
    gzip_buffers 4 16k; #压缩缓冲区
    gzip_http_version 1.0; #压缩版本（默认1.1，前端如果是squid2.5请使用1.0）
    gzip_comp_level 2; #压缩等级
    gzip_types text/plain application/x-javascript text/css application/xml;
    #压缩类型，默认就已经包含text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个warn。
    gzip_vary on;
    #limit_zone crawler $binary_remote_addr 10m; #开启限制IP连接数的时候需要使用

    upstream blog.ha97.com {
    #upstream的负载均衡，weight是权重，可以根据机器配置定义权重。weigth参数表示权值，权值越高被分配到的几率越大。
    server 192.168.80.121:80 weight=3;
    server 192.168.80.122:80 weight=2;
    server 192.168.80.123:80 weight=3;
	}

    #虚拟主机的配置
    server
    {
        #监听端口
        listen 80;
        #域名可以有多个，用空格隔开
        server_name www.ha97.com ha97.com;
        index index.html index.htm index.php;
        root /data/www/ha97;#指定根目录的位置
        
        location ~ .*\.(php|php5)?$
        {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }
        #图片缓存时间设置
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires 10d;
        }
        
        #JS和CSS缓存时间设置
        location ~ .*\.(js|css)?$
        {
            expires 1h;
        }
        
        #日志格式设定
        log_format access ‘$remote_addr – $remote_user [$time_local] “$request” ‘
        ‘$status $body_bytes_sent “$http_referer” ‘
        ‘”$http_user_agent” $http_x_forwarded_for’;
        #定义本虚拟主机的访问日志
        access_log /var/log/nginx/ha97access.log access;

        #对 “/” 启用反向代理
        location / {
            proxy_pass http://127.0.0.1:88;
            proxy_redirect off;
            #后端的web服务器可以通过X-Real-IP获取用户真实IP
            proxy_set_header X-Real-IP $remote_addr;
            #后端的Web服务器可以通过X-Forwarded-For获取反向代理服务器的IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #后端的web服务器可以通过Host获取用户真实主机名
            proxy_set_header Host $host;
            #以下是一些反向代理的配置
            client_max_body_size 10m; #允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数，
            proxy_connect_timeout 90; #nginx跟后端服务器连接超时时间(代理连接超时)
            proxy_send_timeout 90; #后端服务器数据回传时间(代理发送超时)
            proxy_read_timeout 90; #连接成功后，后端服务器响应时间(代理接收超时)
            proxy_buffer_size 4k; #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k; #proxy_buffers缓冲区，网页平均在32k以下的设置
            proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
            proxy_temp_file_write_size 64k;
            #设定缓存文件夹大小，大于这个值，将从upstream服务器传
        }

        #设定查看Nginx状态的地址
        location /NginxStatus {
            stub_status on;#可以访问URL http://localhost/nginx_status 来查看状态信息
            access_log on;
            auth_basic “NginxStatus”;
            auth_basic_user_file conf/htpasswd;
            #htpasswd文件的内容可以用apache提供的htpasswd工具来产生
		}
}
```

##### 5、nginx反向代理

nginx中有两个模块都有proxy_pass的指令

- ngx_http_proxy_module的proxy_pass：在server段使用使用, 只需要提供域名或ip地址和端口

- ngx_stream_proxy_module的proxy_pass：在location段，location中的if段，limit_except段中使用，处理需要提供域名或ip地址和端口外，还需要提供协议，如"http"或"https"，还有一个可选的uri可以配置

  ```
   location /testb {
          proxy_pass http://www.other.com:8801/;
      }
  ```

**http_proxy_module中的proxy_pass：对上游服务器使用http/https进行反向代理**

（1）模块指令：

proxy_pass URL：

- URL必须以http://或https://开头，接下来是域名，IP，unix socket地址，upstream，且可加端口（当location为正则表达式时，proxy_pass不能包含URI部分）
- 当URL参数中携带URI与否，会导致向上游请求的URL不同，不携带URI，则直接转发；不携带URL，则根据location替换
- URL参数可以携带变量
- 更复杂的URL替换可以在location中添加rewrite break语句

proxy_method：修改请求方法

proxy_http_version 1.0|1.1：修改http版本，默认1.0



#### 四、php-fmp

##### 1、CGI、fastcgi、php-fpm、php-cgi

`CGI`：客户端给nginx服务器发送获取.php文件时，nginx服务器会把请求转发给php解析器帮忙，CGI是规定传回结构和格式的协议

`fastcgi`：CGI有个致命的缺点，就是每一次web请求都会有启动和退出过程，也就是最为人诟病的fork-and-execute模式，在高并发下，系统资源大量消耗在进程的创建和销毁上。为了解决这个问题便出现了FastCGI。FastCGI使用常驻进程处理一系列请求。FastCGI接口采用C/S架构，这些常驻进程由FastCGI Server管理，当一个请求到来的时候，Web Server将环境变量和页面信息通过进程间通信发送给FastCGI进程，FastCGI进程处理完后将结果返回给Web Server，Web Server再返回给客户端，一个请求结束后，Web Server和FastCGI进程都没有退出

`PHP-FPM` ：是一种 FastCGI 协议的实现，管理多个php-cgi进程

`php-cgi`：cgi协议的实现

##### 2、php-fpm 的配置文件

/usr/local/php/sbin/php-fpm
/usr/local/php/etc/php-fpm.conf
/usr/local/php/etc/php.ini

##### 3、php-fpm命令

/usr/local/php/sbin/php-fpm -t：测试

/usr/local/php/sbin/php-fpm：启动

kill pid：杀死进程

##### 4、/usr/local/php/etc/php-fpm.d/www.conf.default  配置文件

```
pid = run/php-fpm.pid 
#pid设置，默认在安装目录中的var/run/php-fpm.pid，建议开启

error_log = log/php-fpm.log 
#错误日志，默认在安装目录中的var/log/php-fpm.log  
log_level = notice 
#错误级别. 可用级别为: alert（必须立即处理）, error（错误情况）, warning（警告情况）, notice（一般重要信息）, debug（调试信息）. 默认: notice.

emergency_restart_threshold = 60 
emergency_restart_interval = 60s 
#表示在emergency_restart_interval所设值内出现SIGSEGV或者SIGBUS错误的php-cgi进程数如果超过 emergency_restart_threshold个，php-fpm就会优雅重启。这两个选项一般保持默认值。

process_control_timeout = 0 
#设置子进程接受主进程复用信号的超时时间. 可用单位: s(秒), m(分), h(小时), 或者 d(天) 默认单位: s(秒). 默认值: 0.  
daemonize = yes 
#后台执行fpm,默认值为yes，如果为了调试可以改为no。在FPM中，可以使用不同的设置来运行多个进程池。 这些设置可以针对每个进程池单独设置。

listen = 127.0.0.1:9000 
#fpm监听端口，即nginx中php处理的地址，一般默认值即可。可用格式为: 'ip:port', 'port','/path/to/unix/socket'. 每个进程池都需要设置

listen.backlog = -1 
#backlog数，-1表示无限制，由操作系统决定，此行注释掉就行。
listen.allowed_clients = 127.0.0.1

#允许访问FastCGI进程的IP，设置any为不限制IP，如果要设置其他主机的nginx也能访问这台FPM进程，listen处要设置成本地可被访问的IP。默认值是any。每个地址是用逗号分隔. 如果没有设置或者为空，则允许任何服务器请求连接  
listen.owner = www 
listen.group = www 
listen.mode = 0666 
#unix socket设置选项，如果使用tcp方式访问，这里注释即可。

user = www group = www 
#启动进程的帐户和组

pm = dynamic #对于专用服务器，pm可以设置为static。
#如何控制子进程，选项有static和dynamic。如果选择static，则由pm.max_children指定固定的子进程数。如果选择dynamic，则由下开参数决定： 
pm.max_children #，子进程最大数 
pm.start_servers #，启动时的进程数 
pm.min_spare_servers #，保证空闲进程数最小值，如果空闲进程小于此值，则创建新的子进程 pm.max_spare_servers #，保证空闲进程数最大值，如果空闲进程大于此值，此进行清理  
pm.max_requests = 1000 #设置每个子进程重生之前服务的请求数.  
pm.status_path = /status #FPM状态页面的网址. 如果没有设置, 则无法访问状态页面. 默认值: none. munin监控会使用到 

ping.path = /ping 
#FPM监控页面的ping网址. 如果没有设置, 则无法访问ping页面. 该页面用于外部检测FPM是否存活并且可以响应请求. 请注意必须以斜线开头 (/)。  
ping.response = pong
#用于定义ping请求的返回相应. 返回为 HTTP 200 的 text/plain 格式文本. 默认值: pong.

request_terminate_timeout = 0 
#设置单个请求的超时中止时间. 该选项可能会对php.ini设置的'max_execution_time'因为某些特殊原因没有中止运行的脚本有用. 设置为 '0' 表示 'Off'.当经常出现502错误时可以尝试更改此选项。  
request_slowlog_timeout = 10s 
#当一个请求该设置的超时时间后，就会将对应的PHP调用堆栈信息完整写入到慢日志中. 设置为 '0' 表示 'Off'

slowlog = log/$pool.log.slow 
#慢请求的记录日志,配合request_slowlog_timeout使用  

rlimit_files = 1024 
#设置文件打开描述符的rlimit限制. 默认值: 系统定义值默认可打开句柄是1024，可使用 ulimit -n查看，ulimit -n 2048修改。

rlimit_core = 0 
#设置核心rlimit最大限制值. 可用值: 'unlimited' 、0或者正整数. 默认值: 系统定义值. 

chroot = 
#启动时的Chroot目录. 所定义的目录需要是绝对路径. 如果没有设置, 则chroot不被使用.  

chdir =
#设置启动目录，启动时会自动Chdir到该目录. 所定义的目录需要是绝对路径. 默认值: 当前目录，或者/目录（chroot时）

catch_workers_output = yes 
#重定向运行过程中的stdout和stderr到主要的错误日志文件中. 如果没有设置, stdout 和 stderr 将会根据FastCGI的规则被重定向到 /dev/null . 默认值: 空.
```



#### 五、跨域

##### 1、跨域的几种情况

 `aaa.com => bbb.com`
 `aaa.com:80 =>aaa.com:8080`
 `http://aaa.com => https://bb.com`
 `aaa.com => ccc.aaa.com`

##### 2、解决方法

（1）jsonp
（2）服务器设置跨域CORS
（3）nginx反向代理

##### 3、Nginx通过添加 Access-Control-Allow-Origin、Access-Control-Allow-Methods、Access-Control-Allow-Headers 等HTTP头信息的方式控制浏览器缓存。

```
"Access-Control-Allow-Origin" 设置允许发起跨域请求的网站
"Access-Control-Allow-Methods" 设置允许发起跨域请求请求的HTTP方法
"Access-Control-Allow-Headers" 设置允许跨域请求包含 Content-Type头
```



#### 六、基于openresty+php的环境下搭建wordpress

##### 1.安装openresty：

安装依赖

```
yum install pcre-devel openssl-devel gcc curl
```

安装好openresty-1.17.8.2.tar软件包，拷贝到虚拟机里

```
scp D:\软件安全下载目录\openresty-1.15.8.2.tar root@192.168.29.139：~/
```

解压文件并进入文件中

```
tar -xzvf openresty-1.15.8.2.tar.gz

cd openresty-1.15.8.2.tar.gz
```

进行编译安装

```
./configure --prefix=/usr/local/openresty --with-http_ssl_module

make && make install
```

进入nginx配置文件

```
vim /usr/local/openresty/nginx/conf/nginx.conf
```

在server下面增加以下内容：

```
server {
        listen       80;
        server_name  192.168.29.139;
        //修改访问主机名

        location / {
            root   /data/xkm;
            //当URL匹配'/'时，会获取/data/xkm下面的文件
            index  index.php;
        }

     
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
        
		//当URL请求获取动态资源时：
        location ~ \.php$ {
            root           /data/xkm;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            include        fastcgi_params;
        }

```

测试配置文件是否出错：

```
/usr/local/opresty/nginx/sbin/nginx -t
```

重新加载配置文件

```
/usr/local/opresty/nginx/sbin/nginx -s reload
```

##### 2、安装php：

安装依赖

```
yum -y install libxml2-dev
yum -y install libcurl3-openssl-dev
yum -y install libcurl4-gnutls-dev
yum -y install libbz2-dev
yum -y install libjpeg-dev
yum -y install libpng-dev
yum -y install libxpm-dev
yum -y install libfreetype6-dev
yum -y install libt1-dev
yum -y install libmcrypt-dev
yum -y install libmysql++-dev
yum -y install libxslt1-dev
```

安装php-7.2.8.tar.gz软件包后拷贝到虚拟机里，解压文件并进入文件中

```
tar -xzvf php-7.2.8.tar.gz

cd php-7.2.8
```

进行编译安装：

```
./configure --prefix=/usr/local/php7 --exec-prefix=/usr/local/php7 --with-config-file-path=/usr/local/php7/etc  --with-curl  --with-freetype-dir  --with-gd  --with-gettext  --with-iconv-dir  --with-kerberos  --with-libdir=lib64  --with-libxml-dir  --with-mysqli  --with-openssl  --with-pcre-regex  --with-pdo-mysql  --with-pdo-sqlite  --with-pear  --with-png-dir  --with-xmlrpc  --with-xsl  --with-zlib --with-zlib-dir --with-mhash  --with-openssl-dir --with-jpeg-dir --enable-gd-jis-conv  --enable-fpm  --enable-bcmath  --enable-libxml  --enable-inline-optimization  --enable-mbregex  --enable-mbstring  --enable-opcache  --enable-pcntl  --enable-shmop  --enable-soap  --enable-sockets  --enable-sysvsem  --enable-xml  --enable-maintainer-zts  --enable-zip

make && make install
```

安装完毕后

```
cd /usr/local/php7/sbin

./php-fpm    #启动服务
```

但有个报错，说找不到配置文件，原因是，编译安装得到的配置文件名字为：php-fpm.conf.default，只要把名字改为服务启动需要的配置名字即可

```
cd /usr/local/php7/etc/php-fpm.conf.default  /usr/local/php7/etc/php-fpm.conf

vi /usr/local/php7/etc/php-fpm.conf
```

进入配置文件会发现最后一行需要修改：

```
include=/usr/local/php7/etc/php-fpm.d/www.conf.default
```

##### 3、安装wordpress

下载好wordpress-4.8.6.tar.gz软件包，拷贝到虚拟机中，并对压缩包进行解压操作

```
tar -xzvf wordpress-4.8.6.tar.gz
```

拷贝目录中的所有文件到/data/xkm目录下

```
cp -p ~/wordpress/ /data/xkm
```

修改权限

```
chown -R nobody：nobody /data/xkm
```

##### 4、我之前已经下载好了mysql，登陆mysql即可

```
mysql -u root -p
```

执行以下指令，创建数据库

```
create database worddb；

create user 'rebecca'@'192.168.29.139' identified by '123';

grant all on worddb.* to 'rebecca'@'192.168.29.139';

flush privileges；
```

##### 5、访问192.168.29.139/index.php即可访问wordpress
