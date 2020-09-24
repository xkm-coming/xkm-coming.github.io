---
layout: post
title:  "install_apollo"
date:   2020-09-24 17:01:06 +0800
categories: jekyll update
---
#### 一、对apollo的基础认识

------



##### 1、传统配置文件和分布式配置中心的对比：

- 传统配置文件的缺点：

  如果修改了配置文件，要重新打包发布，重新发布任务，而且每个环境变更配置文件，很复杂

- 分布式配置中心的优点：

  将注册文件注册在配置中心上，可以使用分布式配置中心实时更新配置文件，统一管理，不需要重新打包发布

##### 2、apollo组件：

- `Config Service`：接口服务对象是Apollo客户端

- `Admin Server`：接口服务对象是Portal

- `Meta Server`：Portal通过域名访问Meta Server获取Admin Service服务列表（IP+Port），Client通过域名访问Meta Server获取Config Service服务列表（IP+Port）

- `Eureka`：提供服务注册和发现

- `Portal`：Web界面供用户管理配置

- `Client`：客户端程序

  在部署中，Config Service、Eureka和Meta Server三个逻辑角色会部署在同一个JVM进程中

##### 3、配置发布后的实时推送设计

- 用户在Portal操作配置发布
- Portal调用Admin Service的接口操作发布
- Admin Service发布配置后，发送Release
- Config Service收到ReleaseMessage后，通知对应的客户端（Config Service有一个线程会每秒扫描一次ReleaseMessage表，看看是否有新的消息记录）

##### 4、Config Service通知客户端的实现方式

- 客户端会发起一个Http请求到Config Service的notifications/v2接口

- NotificationControllerV2不会立即返回结果，而是把请求挂起

- 如果在60秒内没有该客户端关心的配置发布，那么会返回Http状态码304给客户端

- 如果有该客户端关心的配置发布，NotificationControllerV2会传入有配置变化的namespace信息，同时该请求会立即返回，客户端从返回的结果中获取到配置变化的namespace后，会立即请求Config Service获取该namespace的最新配置

  

#### 二、搭建apollo

------



##### 1、环境准备

**1.1 安装mysql8.0**

（1）访问https://dev.mysql.com/downloads/repo/yum/

> 在页面中找到"Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package"对应的版本进行下载，下载好后拷贝到虚拟机centos7中

```
wget https://dev.mysql.com/downloads/repo/yum/mysql80-community-release-el7-3.noarch.rpm

rpm -ivh mysql80-community-release-el7-3.noarch.rpm   #解压

yum repolist  #查看有没有mysql安装源

yum -y install mysql-community-server
```

（2）启动mysql

```
systemctl start mysqld
```

（3）登陆数据库

> mysql初始化的时候会生成一个自定义密码，第一次登陆mysql前要去/var/log/mysqld.log去查找密码用于登陆

```
[root@docker log]# cat mysqld.log |grep password

2020-08-12T00:30:08.429978Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: S9T?s*h%jvVo

mysql -u root -p

#修改密码

set global validate_password.policy=0;

set global validate_password.length=1;

ALTER USER "root"@"localhost" IDENTIFIED BY "gzjy5525"; 
```

**1.2 安装java1.8**

（1）用yum安装

```
yum search java  #查看java软件包的名字，找到1.8版本

yum -y install java-1.8.0-openjdk-devel-1.8.0.242.b08-1.el7.x86_64
```

（2）查看是否下载成功

```
[root@docker log]# java -version
openjdk version "1.8.0_262"
OpenJDK Runtime Environment (build 1.8.0_262-b10)
OpenJDK 64-Bit Server VM (build 25.262-b10, mixed mode)
```

##### 2、安装apollo

（1）下载apollo的压缩包，把他们都整合到一个文件夹里面去

```
mkdir -p /usr/local/apollo/apollo-configservice

mkdir /usr/local/apollo/apollo-portal

mkdir /usr/local/apollo/apollo-adminservice

unzip -o -d /usr/local/apollo/apollo-configservice apollo-configservice-1.6.2-github.zip

unzip -o -d /usr/local/apollo/apollo-portal apollo-portal-1.6.2-github.zip

unzip -o -d /usr/local/apollo/apollo-adminservice/ apollo-adminservice-1.6.2-github.zip
```

##### 3、导入ApolloPortalDB和ApolloConfigDB数据库

（1）创建专门存放数据库sql表的文件

```
mkdir /usr/local/apollo/sql
```

将在官网上找到的apolloconfigdb.sql和apolloportaldb.sql文件移到sql目录下，方便查找

（2）导入数据库

> 进入mysql的交互界面进行操作：

```
mysql -u root -p

source /usr/local/apollo/sql/apolloportaldb.sql;    #导入apolloportaldb数据库

select `Id`, `Key`, `Value`, `Comment` from `ApolloPortalDB`.`ServerConfig` limit 1;   #导入成功后测试是否成功

source /usr/local/apollo/sql/apolloconfigdb.sql；  #导入apolloconfigdb数据库

select `Id`, `Key`, `Value`, `Comment` from `ApolloConfigDB`.`ServerConfig` limit 1;  #导入成功后测试是否成功
```

##### 4、修改apollo-adminservice的相关配置文件

- **application-github.properties**：属性配置文件（application.properties）

（1）指定文件

> 进入 /usr/local/apollo/apollo-adminservice配置文件，进行修改

```
cd  /usr/local/apollo/apollo-adminservice/config

vim application-github.properties
```

（2）进行修改

> 修改内容如下：

```
#配置apollo-adminservice的数据库连接信息
spring.datasource.url = jdbc:mysql://localhost:3306/ApolloConfigDB?useSSL=false&characterEncoding=utf8
#MySQL在高版本需要指明是否进行SSL连接
spring.datasource.username = root
spring.datasource.password = gzjy5525
```

（3）更改文件名

> 启动脚本中需要的配置文件名为application.properties，所以要进行备份处理

```
cp application-github.properties application.properties
```

- **startup.sh脚本启动文件**

（1）找到文件

> 进入startup.sh，进行编辑

```
cd /usr/local/apollo/apollo-adminservice/scritps

vim startup.sh
```

（2）进行修改

> 照实际的环境设置一个JVM内存，以下是默认设置

```
export JAVA_OPTS="-server -Xms2560m -Xmx2560m -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=384m -XX:NewSize=1024m -XX:MaxNewSize=1024m -XX:SurvivorRatio=22"
```

> 在配置文件中：
>
> （1）JVM参数：scripts/startup.sh的JAVA_OPTS
>
> （2）日志输出路径：scripts/startup.sh和apollo-configservice.conf中的LOG_DIR
>
> （3）服务的监听端口：SERVER_PORT

##### 5、修改apollo-portal的相关配置文件

- **application-github.properties**

（1）找到文件

> 进入/usr/local/apollo/apollo-portal配置文件进行修改

```
cd /usr/local/apollo/apollo-portal/config

vim plication-github.properties
```

（2）进行修改

> 修改内容如下：

```
spring.datasource.url = jdbc:mysql://localhosth：3306/useSSL=false&ApolloPortalDB?characterEncoding=utf8
spring.datasource.username = root
spring.datasource.password = gzjy5525
```

（3）更改文件名

```
cp application-github.properties application.properties
```

- apollo-env.properties：

> Apollo Portal需要在不同的环境访问不同的meta service(apollo-configservice)地址，所以需要在打包时提供这些信息，在apollo-env.properties文件中会有相关的定义

（1）进入修改文件

```
vi apollo-env.properties
```

（2）修改内容

> 修改内容如下：

```
local.meta=http://localhost:8080
dev.meta=http://172,16.23.189:8080
#fat.meta=http://fill-in-fat-meta-server:8080
#uat.meta=http://fill-in-uat-meta-server:8080
#lpt.meta=${lpt_meta}
#pro.meta=http://fill-in-pro-meta-server:8080
```

- **startup.sh启动脚本文件**

（1）找到文件

```
cd /usr/local/apollo/apollo-portal/scripts

vi startup.sh
```

（2）进行修改

> 修改内容如下：

```
export JAVA_OPTS="-server -Xms4096m -Xmx4096m -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=384m -XX:NewSize=1536m -XX:MaxNewSize=1536m -XX:SurvivorRatio=22"
```

##### 6、修改apollo-configservice的相关配置文件

- **application-github.properties** 

（1）找到文件

> 进入/usr/local/apollo/apollo-configservice/config/配置文件，进行修改

```
cd /usr/local/apollo/apollo-configservice/config

vim application-github.properties
```

（2）进行修改

> 修改内容如下：

```
spring.datasource.url = jdbc:mysql://localhost:3306/ApolloConfigDB?useSSL=false&characterEncoding=utf8
spring.datasource.username = root
spring.datasource.password = gzjy5525
```

（3）更改文件名

```
cp application-github.properties application.properties
```

- **startup.sh启动脚本文件**

（1）找到文件

```
cd /usr/local/apollo/apollo-configservice/scripts

vim startup.sh
```

（2）修改文件

> 修改内容如下：

```
export JAVA_OPTS="-server -Xms6144m -Xmx6144m -Xss256k -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=384m -XX:NewSize=4096m -XX:MaxNewSize=4096m -XX:SurvivorRatio=18"
```

##### 7、关闭防火墙

```
systemctl stop firewalld
```



#### 三、可能出现的问题

------



##### 1、startup.sh启动失败

- 解决的办法：

- 内存不够

- 存在docker网卡

  > 分别编辑[apollo-configservice/src/main/resources/application.yml](https://github.com/ctripcorp/apollo/blob/master/apollo-configservice/src/main/resources/application.yml)和[apollo-adminservice/src/main/resources/application.yml](https://github.com/ctripcorp/apollo/blob/master/apollo-adminservice/src/main/resources/application.yml)，然后把需要忽略的网卡加进去

  ```
  spring:
        application:
            name: apollo-configservice
        profiles:
          active: ${apollo_profile}
        cloud:
          inetutils:
            ignoredInterfaces:
              - docker0
              - veth.*
  ```

- 连接数据库属性出错，高版本的数据库要求对SSL定义，在application.properties文件中，对url进行以下修改：

  ```
  spring.datasource.url = jdbc:mysql://localhost:3306/ApolloConfigDB?useSSL=false&characterEncoding=utf8
  #增加useSSL=false
  ```
