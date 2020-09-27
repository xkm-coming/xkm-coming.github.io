---
layout: post
title:  "jenkins"
date:   2020-09-24 18:52:18 +0800
typora-root-url: ..
categories: jekyll update
---


### 一、基础知识


#### 1、持续集成

- 持续集成（俗称CI），持续集成强调开发人员提交新代码之后，立刻进行构建、（单元）测试。持续集成注重将各个开发者的工作集合到一个代码仓库中，通常每天会进行几次， 主要目的是尽早发现集成错误，使团队更加紧密结合，更好地协作。

#### 2、持续交付

- 持续交付（CD）实际上是 CI 的扩展，其中软件交付流程进一步自动化，以便随时轻松地部署到生成环境中，比如，我们完成单元测试后，可以把代码部署到连接数据库的Stagin环境中更多的测试，如果代码没有问题，可以继续手动部署到生产环境。

- CD 集中依赖于部署流水线，团队通过流水线自动化测试和部署过程。此流水线是一个自动化系统， 可以针对构建执行一组渐进的测试套件。CD 具有高度的自动化，并且在一些云计算环境中也易于配置。而且，在测试环节出现会问题会向团队发出警报。

#### 3、持续部署

- 持续部署扩展了持续交付，以便软件构建在通过所有测试时自动部署。在这样的流程中， 不需要人为决定何时及如何投入生产环境。

#### 4、持续集成的流程

- `提交`：开发者向代码仓库提交代码

- `测试`：代码仓库对提交操作配置hook，只要提交代码或者合并进主干，就会跑自动化测试

- `构建`：通过第一轮测试，代码可以合并进主干，就算可以交付，交付后，进行构建，把源码转换为可以允许的实际代码，比如安装依赖、配置资源

- `测试`：构建后要进行第二轮测试（不是必要的）

- `部署`：过了第二轮测试，当前的代码是可以直接部署的版本，将这个版本的所有文件打包存档，发到生产服务器

- `回滚`：一旦当前版本发生问题，就可以回滚到上一个版本的构建结果



### 二、Jenkins：一款持续集成(CI)工具


#### 1、Jenkins

- 主要用于持续、自动的构建/测试软件项目、监控外部任务的运行，Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行

#### 2、Jenkins的优势

（1）开源的java语言开发持续集成工具，支持持续集成，持续部署

（2）易于安装部署配置：可通过yum安装，或下载war包以及通过docker容器等快速实现安装部署，拥有方便的web界面配置管理

（3）消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果完成时，通过E-mail通知，生成JUnit/TestNG测试报告

（3）分布式构建：多台计算机一起构建/测试

（4）文件识别：Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar

（5）丰富的插件支持

#### 3、Gitlab+Jenkins实现持续集成流程

- 开发人员每天进行代码提交，提交到Git仓库

- Jenkins作为持续集成工具，使用Git工具到Git仓库库拉取代码到集成服务器，再配合JDK、Maven等软件完成代码编译，代码测试与审查，测试，打包等工作，在整个过程中只要有一步出错，都重新再执行一次整个流程

- Jenkins把生成的jar或war包分发到测试服务器或者生产服务器，测试人员或用户就可以执行一次整个流程



### 三、Jenkins的安装


1、准备镜像源安装

- 访问https://pkg.jenkins.io/redhat-stable/，页面有centos7安装Jenkins的教程

```
#在本地安装yum源

wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo

#导入用于签名的gpg软件包密钥

rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key  
```

2、安装Jenkins的环境依赖以及Jenkins服务

```
yum install -y java-1.8.0

yum install -y jenkins
```

3、修改Jenkins端口配置

- 修改端口，避免端口冲突

```
#/etc/sysconfig/jenkins配置文件中有定义端口的变量"JENKINS_PORT"

vi /etc/sysconfig/jenkins

#修改内容如下：

JENKINS_PORT="8000"
```

4、启动Jenkins

- 用systemctl可以管理Jenkins服务

```
systemctl start jenkins
```

5、查看Jenkins是否启动成功

```
systemctl status jenkins

#出现以下信息即为启动成功

[root@docker ~]# systemctl status jenkins
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: active (running) since Thu 2020-08-13 09:15:58 EDT; 8s ago
```

6、关闭防火墙

```
systemctl stop firewalld
```

7、获取密码

- 第一次登陆页面需要输入密码解锁Jenkins，在指定文件中可以获得密码

```
cat /var/lib/jenkins/secrets/initialAdminPassword
```

8、浏览器访问http://172.16.23.194:8000

- 输入密码解锁------选择自定义安装插件------创建管理员，设置用户名和密码------用管理员身份登陆Jenkins



### 四、Jenkins的管理


#### 1、jenkins的相关文件

- 用rpm -ql jenkins可以查到jenkins的相关文件

- `/var/log/jenkins`：jenkins的日志文件

- `/etc/rc.d/init.d/jenkins`：jenkins的启动脚本

- `/usr/lib/jenkins/jenkins.war`：jenkins的war包

- `/var/lib/jenkins`：工作目录（创建的用户和任务都保存在此目录下）

- `/etc/sysconfig/jenkins`：jenkins的配置文件（可以定义Jenkins启动参数和端口）

#### 2、jenkins的相关命令

- `systemctl stop jenkins`：停止jenkins服务

- `systemctl start jenkins`：开启jenkins服务

- `systemctl status jenkins`：查看jenkins状态

- `less  /var/log/jenkins/jenkins.log`：查看日志信息



### 五、Jenkins的插件应用

------



#### 1、用户权限管理

- Role-based Authorization Strategy插件可以用来管理用户权限

1-1 角色的分类：

- 全局角色：可以对jenkins系统进行设置与项目的操作，如果全局角色只有Overall的Read而没有Job的Read，是看不到任务的

  | Overall(全局) |      |            |               |                       | Credentials(凭证) |        |      |        |               | Slave(节点) |        |        |            |         |       | Job(任务) |        |           |      |          |       |            |          | View(视图) |        |           |      |
  | ------------- | ---- | ---------- | ------------- | --------------------- | ----------------- | ------ | ---- | ------ | ------------- | ----------- | ------ | ------ | ---------- | ------- | ----- | --------- | ------ | --------- | ---- | -------- | ----- | ---------- | -------- | ---------- | ------ | --------- | ---- |
  | Administer    | Read | RunScripts | UploadPlugins | ConfigureUpdateCenter | Create            | Update | View | Delete | ManageDomains | Configure   | Delete | Create | Disconnect | Connect | Build | Create    | Delete | Configure | Read | Discover | Build | Workspace  | Cancel   | Create     | Delete | Configure | Read |
  | 管理员(最大)  | 阅读 | 运行脚本   | 升级插件      | 配置升级中心          | 创建              | 更新   | 查看 | 删除   | 管理域        | 配置        | 删除   | 创建   | 断开连接   | 连接    | 构建  | 创建      | 删除   | 配置      | 阅读 | 重定向   | 构建  | 查看工作区 | 取消构建 | 创建       | 删除   | 配置      | 阅读 |

- 项目角色：只能对项目进行操作

  | Role     | Pattern        | 凭据     |        |               |        |                | 代理             |           |                      |        |        |                      |              |            | 任务  |        |           |        |        |                                                              |                                        |                  |          |              | 运行                         |                          |                                      |        |
  | -------- | -------------- | -------- | ------ | ------------- | ------ | -------------- | ---------------- | --------- | -------------------- | ------ | ------ | -------------------- | ------------ | ---------- | ----- | ------ | --------- | ------ | ------ | ------------------------------------------------------------ | -------------------------------------- | ---------------- | -------- | ------------ | ---------------------------- | ------------------------ | ------------------------------------ | ------ |
  |          |                | Create   | Delete | ManageDomains | Update | View           | Build            | Configure | Connect              | Create | Delete | Disconnect           | ExtendedRead | Provision  | Build | Cancel | Configure | Create | Delete | Discover                                                     | ExtendedRead                           | Move             | Read     | Workspace    | Delete                       | Replay                   | Update                               | Tag    |
  | 角色名称 | 可见项目的匹配 | 增加凭据 | 删除   | 管理域        | 更新   | 得到凭据的视图 | 在代理上允许任务 | 配置代理  | 连接代理或让代理上线 | 创建   | 删除   | 断开连接或让代理下线 | 读取代理配置 | 提供新节点 | 构建  | 取消   | 配置      | 创建   | 删除   | 该权限允许用户查找任务，比read权限低，如果没有read则会返回404错误 | 授予只读访问，构建中的敏感信息会被暴露 | 从文件中移动任务 | 查看任务 | 查看空间内容 | 运行从构建历史中删除指定记录 | 执行编辑过的流水线的能力 | 允许用户修改一次构建的描述或其他属性 | 建标签 |

1-2 用户授权实操

- 要求：创建一个job，授权该job给一个用户(dev_user)，这个用户可以自由构建该job并能查看构建日志，但不能修改该job的配置

（1）下载Role-based Authorization Strategy插件

（2）打开功能

- 系统管理------安全:全局安全配置------授权策略:Role-Based Strategy 

![image](/Picture/jenkins_pictures/1.png)

![image](/Picture/jenkins_pictures/1-2.png)

（3）创建角色,赋予权限

- 系统管理------未分类:Manage and Assignn Roles------Manage Roles：

  - Role：role00
  - Pattern：job
  - job：build、Discover、ExtendedRead 、Read、Workspace

![image](/Picture/jenkins_pictures/2.png)

![image](/Picture/jenkins_pictures/2-2.png)

（4）新建用户：dev_user

- 系统管理-------安全:管理用户------新建用户

![image](/Picture/jenkins_pictures/3.png)

![image](/Picture/jenkins_pictures/3-2.png)

（5）分配角色

- 系统管理-------未分类:Manage and Assign Roles-------Assign Roles：

- Item roles：ADD dev_user------选择role00角色

![image](/Picture/jenkins_pictures/4.png)

![image](/Picture/jenkins_pictures/4-2.png)

#### 2、Job的多参数选择设计

- Extended Choice Parameter plugin插件可以实现多参数选择

2-1 参数设计

- name：变量名
- Parameter Type：参数类型
- Number of Visible Items：选项个数
- Choose Source for Value：选择值的来源，这里选择Value，手动加入变量，变量与变量之间用”，“隔开，例如：http://1.1.1.1/root/test.git,http://2.2.2.2/root/test/git.http

2-2 创建job实操

- 要求：支持参数构建、构建可以选择仓库分支、提供一个变量(version)输入框可以填写软件安装的版本、提供一个下拉菜单变量(action)可以选择(install或remove)，构建执行命令可以类似(yum $action openresty-$version)

（1）安装Extended Choice Parameter plugin插件

（2）Jenkins的项目配置：选择General:参数化构建过程

- 构建变量url
- name：url
- Parameter Type：Radio Buttons
- Number of Visible Items：2
- Choose Source for Value------Value：http://1.1.1.1/root/test.git,http://2.2.2.2/root/test/git.http

![image](/Picture/jenkins_pictures/5.png)

- 构建变量version
- name：version
- Parameter Type：Radio Buttons
- Number of Visible Items：3
- Choose Source for Value------Value：1.1,2.2,3.3

- 构建变量action
- name：action
- Parameter Type：Radio Buttons
- Number of Visible Items：2
- Choose Source for Value------Value：remove,stall

（3）源码管理配置

- 选择Git------- Repository URL:${url}

![image](/Picture/jenkins_pictures/6.png)

（4）构建配置

- 执行shell:yum -y ${action} openresty-${version}

![image](/Picture/jenkins_pictures/7.png)



#### 3、jenkins凭证管理

- Credentials Binding插件可以用来管理凭证功能

3-1 凭证的概念

凭证可以用来存储需要密文保护的数据库密码，Gitlab密码信息，Docker私有仓库信息等，以便jenkins可以和这些第三方的应用进行交互

3-2 Jenknins中存储凭证的类型：

（1）秘密文本 ：令牌，例如API令牌（例如GitHub个人访问令牌）

（2）用户名和密码 ：在对应字段指定credential 的 Username 和 Password

（3）机密文件 ：本质上是文件中的机密内容

（4）SSH用户名与私有密钥 ： SSH公钥/私钥对

（5）证书 ： PKCS#12 证书文件和可选密码

（6）Docker主机证书身份验证凭据

3-3 两种创建凭证的方式：

（1）在创建项目中配置凭证

- 源码管理:Git------Credentials:ADD

![image](/Picture/jenkins_pictures/8.png)

![image](/Picture/jenkins_pictures/9.png)

（2）在Jenkins的页面进行添加

- 系统管理------安全:Manage Credentials------域:添加凭证

![image](/Picture/jenkins_pictures/10.png)

![image](/Picture/jenkins_pictures/10-2.png)



### 六、webhook

1、webhook的概念

- webhooks是一个api概念，准确的说webhoo是一种web回调或者http的push API，是向APP或者其他应用提供实时信息的一种方式。Webhook在数据产生时立即发送数据，也就是说你能实时收到数据。

- Webhook有时也被称为反向API，因为它提供了API规则，你需要设计要使用的API。Webhook将向你的应用发起http请求，典型的是post请求，应用程序由请求驱动。

#### 2、webhook的一般实现方式：

- 使用webhook就需要为对应的服务端设计一个hook url,用于接收服务端的请求，通常webhook请求过来的数据格式为xml和json

#### 3、基于gitlab及jenkins运用webhook

（1）下载Gitlab Hook Plugin和Gitlab Plugin插件

（2）Jenkins的项目配置：

- 构建:Build when a change is pushed to GitLab. GitLab webhook URL: http://172.16.23.194:8000/project/webhook-test

![image](/Picture/jenkins_pictures/11.png)
![image](/Picture/jenkins_pictures/18.png)

（3）gitlab的系统配置：

- 行政区(界面上方工具图形)------setting:network------Outbound requests:Expand:勾选Allow requests to the local network from web hooks and services

![image](/Picture/jenkins_pictures/13.png)

![image](/Picture/jenkins_pictures/14.png)

（4）gitlab的项目配置：

- setting:webhooks------URL:http://172.16.23.194:8000/project/webhook-test ------ADD webhook------test:push events

![image](/Picture/jenkins_pictures/15.png)

![image](/Picture/jenkins_pictures/16.png)

![image](/Picture/jenkins_pictures/17.png)

![image](/Picture/jenkins_pictures/17-2.png)
