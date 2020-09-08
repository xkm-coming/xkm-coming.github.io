### 一、saltstack的基础知识

------

#### 1、saltstack的概念

- SaltStack 具备配置管理、远程执行、监控等功能，一般可以理解为是简化版的 Puppet 和加强版的 Func

- SaltStack 本身是基于 Python 语言开发实现，结合了轻量级的消息队列软件 ZeroMQ 与 Python 第三方模块（Pyzmq、PyCrypto、Pyjinjia2、python-msgpack 和 PyYAML 等）构建。

#### 2、saltstack的开发语言

- python

#### 3、saltstack的工作方式

- **Master/Minion（ZeroMQ）**：master和所有minion直连，minion接收来自于master的指令，完成命令的执行或者配置管理

  ![image](Picture/saltstack_pictures)

- **Masterless：minion**：本地运行，可以直接管理本地，不用调用salt-master而影响其他satl-minion

- **Salt-SSH（0.17+）**：利用salt-ssh管理远程主机，可以独立运行的，不需要minion端，功能类似expect的密码推送脚本

- **用syndic方式扩展salt的管理架构**：通过syndic对minion进行管理和设置，syndic接受来自master的任务，然后将任务下发给所有由syndic管理的minion机器，最后将所有minion执行的结果返回给syndic，syndic再将结果发回给master

  ![image](Picture/saltstack_pictures)

#### 4、Master/Minion架构的具体说明

##### 1）组件介绍

- Master ：服务端的控制中心，负责 Salt 命令运行和资源状态的管理
- Minion ：客户端的安装组件，会主动去连接 Master 端，并从 Master 端得到资源状态信息，同步资源管理信息
- ZeroMQ：一款开源的消息队列软件，master用ZeroMQ向minion发布消息，客户端和服务端安装时会接入ZeroMQ内库

##### 2）master和minion建立通信的流程：

- minion在启动时，会自动生成私钥和公钥，主动将公钥发送给服务器端
- 服务端验证并接受公钥，认证完后会将自己的公钥发给客户端，之后两者建立可靠且加密的通信连接

##### 3）master发布信息的流程：

> master通过4405端口发送消息，通过4406端口接受处理结果，salt客户端程序不监听端口，客户端启动后，会主动连接master端注册，然后一直保持该TCP连接

- 在服务端上敲入命令，获取一个 jid
- master接受到服务端的命令后，将要执行的命令发送给客户端 minion
- minion接受并处理命令，通过4506端口返回处理结果
- Master 接收到客户端返回的结果，并将结果写的文件中
- 服务器通过轮询获取 Job 执行结果，将结果输出到终端

#### 5、SaltStack 的优势

- 部署简单、管理方便；

- 支持大部分的操作系统，如 Unix／Linux／Windows 环境；

- 架构上使用C/S管理模式，易于扩展；

- 配置简单、功能覆盖广；

- 主控端（Master）与被控端（Minion）基于证书认证，确保安全可靠的通信；

- 支持 API 及自定义 Python 模块，轻松实现功能扩展；

#### 6、saltstack的相关网址

- 官方网站：http://www.saltstack.com

- 项目地址：https://github.com/saltstack/salt

  

### 二、部署和管理saltstack

------

#### 1、安装

##### 1）在master主机上部署salt-master

```
[root@master ~]#yum -y install https://repo.saltstack.com/yum/redhat/salt-repo-latest-3.el7.noarch.rpm

[root@master ~]#yum -y install salt-master
```

##### 2）在minion主机上部署salt-minion

```
[root@minion ~]#yum -y install https://repo.saltstack.com/yum/redhat/salt-repo-latest-3.el7.noarch.rpm

[root@minion ~]#yum -y install salt-minion
```

##### 3）修改minion主机的配置文件，minion主动连接master

```undefined
[root@minion ~]#vi /etc/salt/minion

##找到master，修改成master主机的IP地址号
#注意‘：’后面有空格
master: 172.16.22.174

##找到id
#id可以不修改，默认使用主机名作标识
#id:
```

##### 4）关闭两台主机上的防火墙，确保两台主机可以进行通信

```
[root@master ~]#systemctl stop firewalld
```

##### 5）启动master主机的salt-master服务

```
[root@master ~]#systemctl start salt-master
```

##### 6）启动minion主机的salt-minion服务

```
[root@minion ~]#systemctl start salt-minion
```

##### 7）在master主机上手动进行认证

```
#查看salt-key

[root@master ~]# salt-key -L
Accepted Keys:
Denied Keys:
Unaccepted Keys:
minion
Rejected Keys:

#手动添加认证

[root@master ~]salt-key -a minion
```

##### 8）当minion-key的状态由unaccepted改为accepted时，说明认证成功，可以尝试远程执行测试

```
#检测minion是否在干活

[root@master ~]#salt "*" test.ping
minion:
    True
    
#检查是否可以远程执行命令

[root@master ~]#salt "*" cmd.run 'echo helloword!'
minion:
    helloword!
```

#### 2、master配置文件

> /etc/salt/master

```
#指定bind的地址，默认值：0.0.0.0(所有的网络地址接口)； 绑定到本地的某个网络地址接口

interface： 0.0.0.0 

#指定发布端口，master认证完minion后会下发此端口号给minion

publish_port： 4505

#指定master进程的运行用户，如果调整，则需要调整部分目录的权限

user： root

#指定结果返回端口，与minion配置文件中的master_port对应

ret_port： 4506

#指定master的pid文件位置

pidfile: /var/run/salt-master.pid

#该目录为salt运行的根目录，改变它可以使salt从另外一个目录开始运行，好比chroot 

root_dir: /

#这个目录是用来存放pki认证秘钥

pki_dir: /etc/salt/pki/master

#用来存放缓存salt工作执行的命令信息

cachedir: /var/cache/salt

#在启动时验证和设置权限配置目录

verify_env: True

#设置保持老的工作信息的过期时间，minion会执行结果返回给master，master会缓存到本地的cachedir，该参数指定缓存多长时间，以供查看之前的执行结果，会占用磁盘空间（默认为24h）

keep_jobs: 24

#master是否缓存执行结果，如果规模庞大（超过5000台），要使用其他方式来存储jobs，关闭本选项

job_cache：

#指定timeout时间，如果minion规模庞大或网络状况不好，应增大该值

timeout: 5 

#salt输出命令的格式

output: nested

#结果输出有颜色

color: True

#套接口目录

sock_dir: /var/run/salt/master

#缓存minion的grains and pillar数据在缓存目录中

minion_data_cache: True

#return存储

event_return: mysql

#rerurn队列

event_return_queue: 0

#event的大小

max_event_size: 1048576

#最大打开文件数量，不能比umlist中的文件数量要大

max_open_files: 100000

#启动用来接收或应答minion的线程数

worker_threads: 5

#启用“开放模式” ，这种模式仍然保持加密，但是关闭身份验证

open_mode: False

#自动认证，接受所有minion的公钥

auto_accept: False

#自动认证超时时间

autosign_timeout: 120

#定义自动签证规则文件

autosign_file：/etc/salt/autosign.conf

#定义自动拒绝签证规则文件

autoreject_file：/etc/salt/autoreject.conf

#pki文件访问权限

permissive_pki_access: False

#用户模块执行限制

pulisher_acl：

#黑名单

publisher_acl_blacklist：

#是否允许minion传送文件到master

file_recv：False

#执行日志级别，支持的日志级别有‘garbage’，‘trace’，‘debug’，‘info’，‘warning’，‘error’，‘critical’

log_level：warning


```

#### 3、master配置文件中的timeout概念

> timeout定义的参数在网络状况差时派上用场
>
> 下面是发送信息的两种状况：1）正常情况，2）网络状况差的时候

1）发送命令给master，master通过pub传送命令到minion的sub，命令下发完后就结束进程

2）master等待timeout中定义的时间，在限定时间内返回结果，命令执行完毕；在限定时间内没有返回结果，master会执行findjob指令去查询minion的jid，如果找到jid，master会再次等待一轮限定时间，如果proc下没有创建jid的运行，master会直接退出

#### 4、minion配置文件

> /etc/salt/minion

```
#salt-master的地址

master：salt


#master的结果返回端口，要与master配置文件中的ret_port端口对应

maser_port：4506

#指定运行用户

user：root

#salt用来识别minion主机的标识，没有指定时，使用当前主机名

id：

#是否缓存结果

cache_jobs：Flase

#在文件操作时，如果文件发送变更，指定备份目标，当前有效的值为minion，备份在cachedir/file_backups目录下，以原始文件名称加时间戳来命名

backup_mode：minion

#指定模块对应的providers，当master远程执行pkg模块时，会调用yum命令

providers：
  pkg: yumpkg5

#指定file client默认去哪里寻找文件，使用local参数时意味着为masterless模式


file_client：remote

#指定日志级别

loglevel：Warning

#minion是否与master保持keppalive检查，zeromq3以下的版本存在keepalive bug，会导致某些情况下连接异常后minion无法重连master，建议有条件的话升级到zeromq3以上的版本

tcp_keepalive：True
```

#### 5、saltstack管理

##### 对master的管理：

`systemctl start salt-master`：启动服务

`systemctl enable salt-master`：开机自启动

`systemctl stop salt-master`：停止服务

`systemctl restart salt-master`：重启服务

##### 对minion的管理：

`systemctl start salt-minion`：启动服务

`systemctl enable salt-minion`：开机自启动

`systemctl stop salt-minion`：停止服务

`systemctl restart salt-minion`：重启服务

### 三、saltstack命令

------

#### 1、salt命令

- ##### 用法：

```
salt '<target>' <function> [arguments]
```

- ##### target:

1）默认使用glob匹配minion id

```
salt '*' test.ping
```

2）使用Grains系统来通过minion的系统信息进行过滤

```
salt -G 'os:Ubuntu' test.ping
```

3）使用正则表达式匹配monion id

```
salt -E 'virtmach[0-9]' test.ping
```

4）使用IP/子网段匹配

```
salt -S ‘’172.16.23.197‘ test.ping
```

5）使用列表

6）使用混合模式

- ##### function：

> funcation是module提供的功能，module选择：https://docs.saltstack.cn/ref/modules/all/index.html

| function           | 作用                                                         | 例子                                                         |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Archive.*          | 实现系统层面的压缩包调用，支持gzip、gunzip、rar、tar、unrar、unzip等 | alt '*' archive.gunzip /tmp/sourcefile.txt.gz                |
| file.manage_file   | 修改文件                                                     |                                                              |
| file.access        | 查看是否有文件权限                                           | salt '*' file.access /etc/salt/minion f                      |
| file.append        | 追加文件内容                                                 | salt '*' file.append /etc/motd ‘helloword’                   |
| file.chgrp         | 更改文件所属组                                               | salt '*' file.chgrp /etc/passwd root                         |
| file.chown         | 更改文件用户和文件所属组                                     | salt '*' file.chown /etc/passwd root root                    |
| file.copy          | 复制文件                                                     | salt '*' file.copy /path/to/src_dir /path/to/dst_dir recurse=True remove_existing=True |
| cmd.run            | 执行传递的命令并以字符串形式返回输出                         | salt '*' cmd.run 'echo helloword!'                           |
| cmd.script         | 远程下载脚本在客户端执行，脚本文件可以放在master(放在/srv/salt下)或http/FTP服务器上 | salt ‘*’ cmd.script salt://scripts/test.sh'                  |
| cp.get_dir         | 用于递归地从master 复制目录                                  | salt '*' cp.get_dir salt://path/to/dir/ /minion/dest         |
| cp.get_file        | 从master复制文件                                             | salt '*' cp.get_file salt://path/to/file /minion/dest        |
| cp.get_url         | 下载URL内容到minion指定位置                                  | salt '*' cp.get_url http://ftp.linux.ncsu.edu/pub/epel/6/i386/epel-release-6-8.noarch.rpm  /tmp/epel-releas |
| cp.push            | 从minion发送文件到master                                     | salt '*' cp.push /etc/httpd/conf/httpd.conf                  |
| network.active_tcp | 返回活跃的tcp连接信息                                        | salt '*' network.active_tcp                                  |
| network.connect    | 测试端口是否连接上                                           | salt '*' network.connect archlinux.org 80                    |
| network.ip_addrs   | 获取IP地址                                                   | salt '*' network.ip_addrs                                    |
| service.available  | 检查服务是否正常                                             | salt '*' service.available sshd                              |
| service.start      | 启动服务                                                     | salt '*' service.start  sshd                                 |
| service.status     | 查看服务状态                                                 | salt '*' service.status sshd                                 |
| service.stop       | 停止服务                                                     | salt '*' service.stop sshd                                   |
| pkg.installed      | 安装软件包 (pkgs 同时安装多个包)                             | salt '*' pkg.install httpd                                   |
| pkg.latest_version | 最新版本                                                     | salt '*' pkg.latest_version httpd                            |
| pkg.removed        | 删除软件包                                                   | salt '*' pkg.remove httpd                                    |
| pkg.purge          | 卸载并删除配置文件（会把软件包和配置文件都删除掉）           | salt '*' pkg.purge httpd                                     |

- ##### 其他常用命令:

`salt '*' grains.items`：查看minion全部信息

`salt '*' grains.item os`：查看minion的os信息

`salt '*' grains.get os`：与salt '' grains.item os用法相同

`salt '*' saltuil.refresh_pillar`：刷新pillar值

#### 2、salt-cp的用法：

- ##### 常用命令：

`salt-cp '*' [ options ] SOURCE DEST`：使用glob匹配minion id，从master复制文件到minion
`salt-cp -E '.' [ options ] SOURCE DEST`：使用正则匹配minion id，从master复制文件到minion
`salt-cp -G 'os:CentOS*' [ options ] SOURCE DEST`：使用Grains系统进行过滤，从master复制文件到minion

#### 3、salt-key常用命令

`satl-key -h`：查看帮助

`salt-key -L`：列出当前所有的key

`salt-key -a salt-minion`：接受id为salt-minion的key

`salt-key -A`：接受所有的key

`salt-key -r salt-minion`：解决id为salt-minion的key

`salt-key -R`：解决所有的key

`salt-key -d salt-minion`：删除id为salt-minion的key

`salt-key -D`：删除所有的key

### 四、key认证

------

#### 1、key认证概念

- 认证采用RSA key方式确认身份，传输采用AES加密算法，master&minion端采用key管理

#### 2、key保存位置

> key默认存储在/etc/salt/pki目录下

```
[root@minion salt]# tree pki
pki
├── master
└── minion
    ├── minion_master.pub     #存放master公钥
    ├── minion.pem            #minion私钥
    └── minion.pub            #minion公钥

2 directories, 3 files
```

```
[root@master salt]# tree ./pki
./pki
├── master
│   ├── master.pem            #master私钥
│   ├── master.pub            #master公钥
│   ├── minions               #存放被接收的minion公钥
│   │   └── minion
│   ├── minions_autosign      #
│   ├── minions_denied        #存放被拒绝的minion公钥
│   ├── minions_pre           #minion发过来的公钥放在pre目录下，目前还没有被master管理
│   └── minions_rejected
└── minion

7 directories, 3 files
```

#### 3、两种认证方式：

1）手动认证：使用salt-key

2）自动认证：可以修改master配置文件

- 修改open_mode：该选项开启后, Master将关闭Auth功能, 并告诉master接受所有认证

```
[root@master ~]# vi /etc/salt/master

#找到open_mode: False改为

open_mode: True
```

- 修改auth_accept：Master将自动接受所有minion发过来的Pub Key

```
[root@master ~]# vi /etc/salt/master

#找到auto_accept: False改为

auto_accept: True
```

### 五、saltsack的state.sls的使用

------

#### 1、sls的概念

- SLS（代表SaLt State文件）是Salt State系统的核心。SLS描述了系统的目标状态，由格式简单的数据构成。这经常被称作配置管理，可以用yaml语言编写

#### 2、yaml语法

1）缩进：Salt需要每个缩进级别由两个空格组成。不要使用tabs

2）冒号：字典的keys的表现形式是一个以冒号结尾的字符串，Values的表现形式冒号下面的每一行，用一个空格隔开，value也可以通过缩进与key联接，例如

```
my_key:
  my_value
```

3） 短横杠：一个短横杠加一个空格表示列表项，多个项使用同样的缩进级别作为同一列表的一部分

```
- list_value_one
- list_value_two
- list_value_three
```

#### 3、三种常见模块的用法

##### 1）软件包管理状态模块：pkg

- 基本用法：

- `pkg.installed`：确保软件包已安装，如果没有安装进行安装

- `pkg.lastest`：确保软件包是最新版本，如果不是，进行升级

- `pkg.remove`：确保软件包已卸载，如果之前已安装，进行卸载

- `pkg.purge`：卸载并删除配置文件（会把软件包和配置文件都删除掉）

##### 2）文件状态模块：file

- 基本用法：

- file.managed：保证文件存在并且为对应的状态

- file.rescurse：保证目录存在并且为对应状态

- file.absent：确保文件不存在，如果存在则进行删除操作

##### 3）服务状态模块：service

- 基本用法：

- service.running：确保服务处于运行状态，如果没有运行则进行启动

- service.enabled：确保服务会开机自动启动

- service.disabled：确保服务不会开机自动启动

- service.dead：确保服务当前没有运行

#### 4、top.sls

##### 1）top.sls的重要性

- top.sls 是配置管理的入口文件，一切都是从这里开始，在master 主机上，默认存放在/srv/salt/目录. top.sls 默认从 base 标签开始解析执行,下一级是操作的目标，可以通过正则，grain模块,或分组名,来进行匹配,再下一级是要执行的state文件

##### 2）top.sls的编写规则

> /srv/salt/top.sls

- 按主机

```
base:  
  '*':    
    - webserver
```

- 通过分组名进行匹配的示例，必须要有 - match: nodegroup

```
base:
  group1:
    - match: nodegroup    
    - webserver
```

- 通过grain模块匹配的示例，必须要有- match: grain

```
base:
  'os:Fedora':
    - match: grain
    - webserver
```

#### 5、sls文件命名规则

1）sls文件的扩展名（.sls）忽略

2）目录的层级结构用'.'来进行划分

```
base:  
  '*':    
    - nginx.nginx     #nginx目录下的nginx.sls文件
```

3）目录下有init.sls文件时，可直接指定目录

#### 6、state的层级关系

- include：包含某些文件，容易修改维护

```
include：
  - nginx
```

- extend：修改，或扩展引用的state文件的某个字段

```
include:
  - apache  
extend：  
  apache      
    pkg:      
      - name: vim      
      - installed     
```

#### 7、利用state.sls部署nginx

1）决定文件结构

```
[root@master ~]# trees /srv/salt    
-bash: trees: command not found
[root@master ~]# tree /srv/salt 
/srv/salt
├── nginx
│   ├── nginx.conf
│   └── nginx.sls
├── nginx.sls
└── top.sls
```

2）编辑top.sls

```
vi /etc/salt/top.sls



base:
  '*':
    - nginx
```

3）编辑nginx.sls文件

```
nginx:
  pkg.installed:
    - name: nginx
  file.managed:
    - name: /etc/nginx/nginx.conf
    - source: salt://nginx/nginx.conf
  service.running:
    - user: nginx
    - enable: True
```

4）在master主机下载nginx，修改好nginx.conf配置，放在指定目录下

5）执行远程操作命令

```
salt ’minion‘ state.sls nginx
```