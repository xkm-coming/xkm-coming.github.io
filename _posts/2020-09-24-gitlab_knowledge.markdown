### gitlab部署和使用文档



#### 一、在centos7上安装gitlab（在桥接模式下）

##### 1.下载基础依赖

```
#安装技术依赖

yum -y install curl policycoreutils-python openssh-server

#启动ssh服务&设置开机启动

systemctl enable sshd

systemctl start sshd
```

##### 2.安装postfix

```
yum -y install postfix

#启动postfix&设置开机启动

systemctl enable postfix

systemctl start postfix
```

##### 3.设置防火墙，开放ssh、http服务

```
firewall-cmd --add-service=ssh --permanet

firewall-cmd --add-service=http --permanet

#重载防火墙规则

firewall-cmd --reload
```

##### 4.添加Gitlab软件包存储库并下载Gitlab-ce软件包

```
#添加Gitlab软件包储存库

curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash

#安装gitlab-ce软件包

yum -y install gitlab-ce
```

##### 5.修改本地host，将域名指向服务器ip

```
vim /etc/hosts

```

往hosts文件添加内容如下：

```
192.168.29.138 gitlab.xkm.com
```

##### 6.修改gitlab配置文件中的站点Url

```
vim /etc/gitlab/gitlab.rb
```

将external_url后面的值改为：

```
http://gitlab.xkm.com
```

##### 7.重新配置并启动gitlab

```
gitlab-ctl reconfigure
```

##### 8.访问http://gitlab.xkm.com即可

#### 二、gitlab组件认识

- **repository**：代码库，可以是硬盘或分布式文件系统

- **nginx**：web的入口，将前端的http/https请求代理至gitlab-workhorse，nginx默认把http请求重定向至https请求，除了某些静态页面，nginx几乎将所有的hhtp/https请求传递给了gitlab-workhorse组件处理（利用unix socker通信）

- **gitlab-workhorse**：智能反向代理服务器

  （1）可以处理负载量大的HTTP请求（文件上传/下载，git push/pull以及归档下载等），能处理一些无需调用Rails组件的请求（一些静态资源），如果要处理到rails的话，会连接给反向代理给后端unicorn

  （2）workhorse 能修改 Rails 组件发来的响应

  （3）workhorse 能接管向 Rails 组件询问操作权限后的请求，例如处理 git clone 之前得确认当前客户的权限，在向 Rails 组件询问确认后 workhorse 将继续接管 git clone 的请求

  （4）workhorse 能修改发送给 Rails 组件之前的请求信息，例如：当处理 Git LFS 上传时，workhorse 首先向 Rails 组件询问当前用户是否有执行权限，然后它将请求体储存在一个临时文件里，接着它将修改过后的包含此临时文件路径的请求体发送给 Rails 组

  （5）workhorse 无法直接连接数据库，只能与 Rails 组件和 Redis 组件(可选)通信，所有到达 workhorse 的请求都是由上游代理服务器(nginx)转发

  （6）workhorse 不接受 https 连接

  （7）workhorse 不会清除空闲客户端连接（8）所有对 Rails 组件的请求都得经过 workhorse

```
+-----+  +------------------+  +---------+ 
|     |  |                  |  |         | 
NGINX +->| gitlab-workhorse +->| Unicorn |        
|     |  |                  |  |         | 
+-----+  +------------------+  +---------+
```

- **gitlab-shell**：用于处理 GitLab 所有的 git SSH 会话。gitlab-shell通过redis与sidekip进行通信，并直接或通过tcp间接访问unicorn。用于处理git命令和修改authorized keys列表

  当用户以 SSH 的方式访问 GitLab 时(例如 git pull/push over ssh)，GitLab Shell 组件会做下列事情：

  1、限制用户使用预定义的 git 命令(git push, git pull 等)（防止用户登陆终端进行其他操作）

  2、调用 GitLab Rails 的 API 接口以检查用户是否已授权，以及判断用户通过哪台 Gitaly 服务器访问代码仓库(Gitaly 组件的主要功能是进行与代码仓库相关的操作)

  3、在 SSH 客户端和 Gitaly 服务器之间来回拷贝数据

- **unicorn**：web服务器，提供动态网页和API接口，unicorn是多线程模型的Ruby web服务器，遵循Rack协议。gitlab的Rails应用程序是在unicorn服务器内运行的，而unicorn的多进程模型能提供更好的并发处理能力，并且提供更强的容错能力。多模型进程为：在生产环境需要在后台运行unicorn进程，首先会用fork方式创建一个进程，叫grandparent（启动unicorn的进程），再用fork方式创建parent进程（启动unicorn服务的进程），然后用fork方式创建第三个进程，叫master（unicorn服务中的主进程）：

  1、grandparent：表示用来启动unicorn进程的终端，监听master进程是否成功启动

  2、parent：是一个用来设置进程状态和掩码的中间过程，在启动master进程后会退出

  3、master：master进程不负责处理客户端的HTTP请求，但可以用fork方式创建一系列的worker进程，一个master进程管理多个worker进程，多个worker进程监听同一组套接字来处理客户端请求，同样，master进程可以创建新的worker进程替代原worker进程，整个过程不会舍弃用户的请求。由于多个worker进程，内存泄漏会显现在长时间运行的过程中，gitlab使用unicorn-worker-killer以让这些进程的内存泄露得以管控。

- **redis**：缓存每个客户端的sessions和后台队列，负责分发任务。

- **gitaly**：后台服务，专门负责访问磁盘以高效处理gitlab-shell和gitalb-workhorse的git操作，并缓存耗时操作，所有的git都通过它来完成

- **sidekiq**：后台核心服务，可以从redis队列中提取作业并对其进行处理，后台作业允许gitlab通过将工作移至后台来提供更快的请求/响应周期

- 数据库：包含repository中的数据，可以登陆web用户的权限

- **mail_room**：处理邮件请求。回复gitlab发出的邮件，gitlab会调用此服务来处理sidekiq、unicorn和gitlab-shell的任务

- **logrotate**：日志文件管理、切割

#### 三、gitlab-ctl命令认识

gitlab-ctl status：查看gitlab组件状态

gitlab-ctl start：启动全部服务

gitlab-ctl restart：重启全部服务

gitlab-ctl reconfigure：使配置文件生效

gitlab-ct show-config：验证配置文件

gitlab-ctl tail service_name：查看服务日志

#### 四、gitlab的web页面操作

一、项目管理

1.创建项目：项目用三种属性可以选择：

- 私有库：只有被赋予权限的用户可见
- 内部库：登陆用户可以下载
- 公开库：所有人可以下载

二、用户管理：在网页上点击Admin Area可创建用户

三、用户组管理：管理员添加组用户有五种属性可以选择：

- Guest(匿名用户)：可以创建issue，发表评论，不能读写版本库

- Reporter（报告人）：可以克隆代码，不能提交（例如产品经理）

- Developer（开发者）：可以克隆代码、开发、提交、push（例如开发人员）

- maintainer（管理者）：可以创建项目、添加tag、保护分支、添加项目成员、编辑项目（例如核心开发人员）

- Owner：可以设置项目访问权限、删除项目、迁移项目、管理组成员（例如开发组组长）

#### 五、克隆仓库的两种方法

##### 一、通过http/https去访问仓库

```
git clone http://gitlab.xkm.com/root/gitlab.git
```

##### 二、通过ssh去访问仓库

1.生成ssh的钥匙

```
ssh-keygen -t rsa 
```

2.将新生成的key添加到ssh-agent中

```
eval "$(ssh-agent -s)"

ssh-add ~/.ssh/id_rsa
```

3.将公钥提取出来，加到新建成的仓库中

复制粘贴到gitlab project中的ssh key中

4.创建个人信息

```
git config --global user.name "xkm"

git config --global user.email "xkm@192.168.29.138"
```

5.克隆仓库

```
git clone git@6b6cdb3c87df:yunwei/docs.git
```

#### 五、git的用法

- **clone**：git clone 仓库的Url       克隆仓库

```
git clone http://gitlab.xkm.com/root/gitlab.git

git clone git@6b6cdb3c87df:yunwei/docs.git
```

- **init**：git init         初始化一个仓库

```
cd /root/xkm

mkdir newproject

cd ./newproject

git init
```

- **add**：git add .（.表示all），git add 1.txt 2.txt 把工作区的文件缓存到暂存区，为commit做好准备

- **commit**：git commit -m “附加的信息”  把暂存区的文件提交到本地仓库

```
git commit -m “new profiles”
```

- **push**：git push origin master 本地仓库把更新后的仓库上传到远程仓库，请求合并

```
git add .

git commit -m "new profiles"

git push origin master
```

- **branch**：

```
git branch littlexkm  #（littlexkm是创建的分支名字）创建分支

git branch  #查看分支，带*的是当前所处的分支名字
```

- **diff**：

```
git diff #查看暂存器和工作区的差别

git diff --cached #已经暂存起来的文件(staged)和上次提交时的快照之间(HEAD)的差异
```

- **log**：git log 查看版本更新情况

- **git diff commitID commitID** 查看两版本之间的差别

- **git reset --harder commitID** 回滚到之前的版本

  ```
  git log    找到原本版本id
  
  git reset --hard 297ff2dcf20605297684f296a4b4ccaa1cf4dc48
  
  git push -f origin master  强制提交
  ```

