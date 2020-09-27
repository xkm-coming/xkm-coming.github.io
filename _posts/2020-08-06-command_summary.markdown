---
layout: post
title:  "command_summary"
date:   2020-08-06 09:44:18 +0800
typora-root-url: ..
category: jekyll
---

### 常用命令汇总

#### cd    进入目录

用法：cd + 目录名（绝对路径或./+相对路径）

```
cd  ~ 进入家目录

cd  .. 进入上一级目录

cd  -  进入上一次工作目录

cd !$ 进入上次命令的参数
```

#### pwd    查看当前工作目录的路径

```
-P:查看软连接的实际路径
```

#### rm 删除目录或文件

用法：rm [option] 路径

```
-i：删除前先询问

-f：强制删除

-r：递归删除
```

#### mv 移动文件

移动文件或修改文件名，根据第二参数类型（如目录，则移动文件，若为文件，则重命名）

当第二个参数为目录时，可刚多个文件以空格分隔作为第一参数，移动多个文件到参数2制定的目录中

#### mkdir 创建文件夹

用法：pwd [option] 路径

```
-m:对新建目录设置存取权限

-p：一次建立多个目录
```

#### rmdir 删除非空文件夹，要对父目录有写权限

用法：rmkdir [option] 路径

```
-p：父目录的子目录被删除后，若父目录成为空目录，则顺便一并删除
```

#### cp 复制文件，或将多个源文件复制到目标目录

```
-i：复制前询问

-r：复制目录以及目录内的内容

-a：复制的文件与原文件时间一样
```

#### cat 显示整个文件；从键盘创建一个文件（cat > filename）；将几个文件合并为一个文件（cat file1 file2 > file3）

```
-b：对非空输出行数

-n：输出所有行数
```

#### more 以一页页显示方便使用者逐页阅读

space：下一页显示

ctrl+b：往回一页

enter：向下n行，默认一行

q：退出

：f：显示当前行数和文件名

```
+n：从第几行开始显示

-n：定义屏幕大小为几行

+/pattern：在每个档案显示前搜索该字串，然后该字串前两行之后开始显示

-c：从顶部清屏，然后显示

-p：通过清除窗口而不是滚屏来对文件进行换页

-s：连续的多个空行显示为一行

-u：把文件内容的下划线去掉
```

#### less 与more相似，但可以随意浏览文件，且在less查看之前不会加载整个文件

pg up：向上翻页

pg down：向下翻页

q：退出

/字符串：向下搜索“字符串”的功能

？字符串：向上搜索字符串的功能“

&字符串：仅显示匹配到的行，为不是整个文件

```
-i：忽略搜索时的大小写

-N：显示每行的行数

-o：将less输出的内容在指定文件中保存起来

-s：显示连续空行为一行

n：重复前一个搜索

N：反向重复前一个搜索
```

#### head 显示文件开头至标准输出中

```
-n：现实的行数
```

#### tail 显示文件末尾内容

```
-f：循环读取（常用于查看递增的日志文件）

-n 行数：显示行数

-n +行数：从第几行开始显示末尾内容
```

#### which 查看可执行文件的位置（在PATH指定的路径中）

#### whereis 用于程序名的搜索，只搜索二进制文件（-b）、man说明文件（-m）和源代码文件（-s）

```
-u：搜索默认路径下除可执行代码、源代码文件以外的其他文件
```

#### locate：通过搜寻系统内建文档数据库达到快速找到档案

数据库由updatedb程序来更心，upadatedb是由cron daemon周期性调用的。默认情况下locate命令在搜寻数据库比由整个由硬盘资料搜寻来得块，但locate找到的数据是最近建立或刚更名的，可能会找不到最新数据

```
-l 数字：要显示的行数

-r：使用正则运算式来作为寻找条件
```

#### find：在文件树中查找文件

fine pathname [-option] [-print -exec -ok]

```
-print：将匹配的文件输出到标准输出

-exec：执行参数所给出的shell命令

-ok：跟-exec一样，但执行命令前会给出提示，让用户确认是否执行

-name：按照文件名查找文件

-perm：按文件权限查找文件

-usr：按文件拥有着查找文件

-group：按文件用户组查找文件

-type：查找某一类型的文件（b：块设备文件，d：目录，c：字符设备文件，p：管道文件，f：普通文件）

-size：按文件大小查找文件，- size 100M ：100M，-size +100M：100M及100M以上

-atime n：最后24*n内被访问的文件

-ctime n：最后24*n内被改变状态的文件

-mtime n：最后24*n内被改变文件数据的文件

   (用减号-来限定更改时间在距今n日以内的文件，而用加号+来限定更改时间在距今n日以前的文件。 )

 -newer 如果希望查找更改时间比某个文件新但比另一个文件旧的所有文件
```

#### tar 打包

```
-c：建立新的压缩文件

-v：显示操作过程

-f：指定压缩文件

-z：支持gzip压缩

-j：支持bzip2压缩

-t：显示压缩文件中的内容

-x：从压缩包中抽取文件
```

#### df 显示磁盘空间情况

```
-a：全部文件系统列表

-h：以方便阅读的方式显示内容

-i：显示inde号

-k：区块为1024字节

-l：只显示本地磁盘

-T：列出文件系统类型
```

#### du 查看文件和目录磁盘的空闲

```
-a：显示目录中所有文件大小

-k：以KB为单位显示文件大小

-m：以MB为单位显示文件大小

-g：以GB为单位显示文件大小

-s：仅显示总计

-c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和
```

#### top 查看当前系统正在执行进程的相关信息，包括进程ID、内存占用率、cpu占用率

```
-c：显示完整进程命令

-s：保密模式

-p 进程号：指定进程显示

-n 次数：循环显示次数
```

#### ps 查看当前运行的进程状态

进程有五种状态：运行R、中断S、不可中断D、僵死Z、停止T

```
-a：显示统一终端下所有进程

a：显示所有进程

c：显示进程真实名称

e：显示环境变量

f：显示进程间的关系

-aux：显示所有包含其他使用的进程
```

#### kill 发送指定信号到相应进程

```
-l：显示全部信号名称

-p：指定kill命令只打印相关进程的进程号，而不发送任何信号

-s：指定发送信号

-u：指定用户
```

#### free 显示系统内存使用情况，包括物理内存、交互区内存、和内核缓冲区内存

```
-b：以Byte显示

-k：以kb显示

-m：以mb显示

-g：以gb显示

-s 秒数：持续显示内存

-t：显示内存使用总合
```

#### wc 统计制定的文件中字节数、字数、行数，并以统计结果输出

```
-c：统计字节数

-l：统计行数

-m：统计字符数

-w：统计词数
```

#### date 显示或设定系统的日期与时间

```
-d 字符串：显示字符串所指的日期与时间

-s 字符串：根据字符串来设置日期与时间
```

#### useradd 添加用户

```
-d：指定用户家目录

-g：指定用户组

-G：指定多个用户组

-U：同时创建新组，组名与用户名一样

-s：用户可登陆
```

#### passwd 修改密码

#### usermod 修改用户

```
-d：修改家目录

-g：修改主要用户组名

-a：追加用户组

-l：修改登陆名

-s：修改login shell

-u：修改uid
```

#### userdel 删除用户

```
-f：强制删除

-r：移除家目录和用户邮箱

-Z：移除selinux
```

#### groupadd 添加用户组

```
-f：强制加入用户组

-g：指定gid
```

#### groupdel 删除用户组信息

#### groups 显示用户所属组

#### id 查看用户uid，gid和groups信息

chmod 修改权限

r（4）：读权限，可以读文件或内容

w（2）：写权限，可以创建、删除、重命名文件，可修改文件内容

x（1）：可执行权限，可以执行文件内容，可以访问目录中文件，切换到目录中

u：所属用户

g：所属用户组

o：其他人

a：用户、用户组和其他人

```
chmod u+r file：file的所属用户添加读权限

chmod g-r file：file的所属用户组删去读权限

chmod 775 file
```



#### rpm 软件包管理

```
-q：查询

-ql：劣处软件包文件列表

-qf：根据文件查找包名

-qi：查看包信息

-i：下载安装

-v：显示命令执行过程信息

-h：显示安装进度

--force：强制安装

--nodeps：忽略依赖

--nodigest：忽略数字签名

--nosignature：忽略签名

-Uvh：更新

-e：卸载

--import：导入数字签名文件
```

#### yum 软件管理工具

```
-y：自动应答

list：查看仓库的所有包

list installed：查看已安装的所有包

repolist：查看仓库列表

install：下载

remove：卸载

update：更新

clean up：清除仓库缓存文件
```

#### apt 

apt = apt-get、apt-cache 和 apt-config 中最常用命令选项的集合

```
apt-cache search/apt list：查看仓库的包

apt-cache show/apt show：查看装信息

apt-get install/apt get：安装

apt-get remove/apt remove：卸载

apt-get update/apt update：更新软件索引文件

apt-get purge/apt purge：删除（包含配置文件）

apt-get upgrade/apt upgrade：升级
```

#### grep

```
--color=auto：高亮显示匹配到的内容

-E：支持扩展正则表达式

-P：支持PERL正则表达式

-n：显示行号

-i：忽略大小写

-o：只匹配到内容

-v：反向匹配

-A N：匹配到行的后N行

-B N：匹配到行的前N行

-C N：匹配到行的前后N行
```

#### cut 从文件的每一行截取一部分内容

```
-d：分隔符

-f：字段数

-c：字符位置
```

#### sort 排序

```
-t：分隔符

-k：字段数

-r：逆序

-n：以数字大小排序

-u：去重
```

#### uniq 去重（去重前要先排序）

```
-c：统计重复次数
```

#### sed

sed  options '[Address]Command;[Address]Command;...' FILE

```
option：

-r：支持扩展正则

-n：一直默认内容输出

-i：直接修改源文件



Address：

空地址：匹配每一行

N  ：N到最后一行

N,M   ：N~M行



Command：

d：删除

p：显示

s///：替换
```

#### awk

```
-F：指定分隔符

$1：第一个字段

awk -F: '{print $1}' /etc/passwd
```

#### chkconfig

在centos6上控制服务，需要init.d目录下有LSB风格的服务控制脚本

```
chkconfig --list：查看服务在不同运行级别是否开机启动

chkconfig SERVICE_NAME on/off  ：开启/关闭服务

chkconfig -add SERVICE：将自定义的服务脚本加入到chkconfig控制中
```

管理当前服务

```
/etc/init.d/NAME start|stop|restart|reload|status

service NAME start|stop...
```

#### systemctl

start：开启

stop：关闭

status：查看状态

restart：重启 

```
systemctl start|stop|status|restart servicename

systemctl enable sshd.service：开机自启动

systemctl disable sshd.service：开机不启动
```

#### ps

```
-Z：查看SElinux属性
```

#### getenforce 查看SElinux状态

enforcing ：开启 

permissive：警告模式

disabled：关闭

```
getenforce
```

#### setenforce 设置SElinux状态

0：permissive模式

1：enforcing模式

#### docker

```
docker start/stop/restart 容器id     管理容器状态

docker rm 容器Id   删除容器

docker create 创建容器

docker tag 给容器打标签

docker ps查看所有容器的命令

docker images 查看已下载的所有镜像

docker pull 拉取镜像

docker push 上传镜像

docker search 搜寻镜像

docker run 运行镜像启动容器

docker rm 删除镜像

docker attach 进入容器

docker exec 进入容器（退出容器不会导致容器的停止）
```

#### mysql

```
mysql -u username -p password 进入数据库

mysqldump -u username -p database_name table_name > file_name 备份

在mysql交互界面：

create database database_name； 创建数据库

show databases；查看数据库

drop database database_name；删除数据库（drop删除过程，delete删除数据）

use database database_name；选库

creat table table_name；创表

insert into table_name 字段 values（记录）；往表中插入数据

select 字段 from table；查看行

delete from table_name where...；删除表内容

update table_name ...；更新

alter table table_name ...；修改
```



