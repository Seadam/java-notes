# Linux

#### 内容摘要

用户管理和组管理，linux 开发环境搭建，Nginx，配置 Nginx 集群，负载均衡策略，集群 Session 共享解决方案

------

### 用户管理和组管理

需要 **sudo** 权限

#### 用户管理

* `useradd username`  创建用户，不会创建 `/home/username` 目录
* `useradd username -m` 创建用户，并创建目录
* `passwd username`  为 username 用户设置密码
* `su username` 切换用户
* `userdel username` 删除用户，不会删除 `/home/username` 目录
* `userdel -r username` 删除用户，并删除目录

####  组管理

* `id username` 查看用户所属组
* `groupadd groupname` 创建组
* `useradd username -g groupname` 创建用户并设置组
* `usermod -g gid username` 修改 username 用户主组
* `usermod -G groupname username` 修改 username 用户所在附属组 groupname
* `usermod -aG groupname username` 为 username 用户增加附属组 groupname
* `groupdel groupname` 删除组，若组内还有用户无法删除

### 文件权限

需要 **sudo** 权限

文件分类

* 普通文件：文本文件、数据文件、可执行二进制文件等 `-`
* 目录文件：目录文件是一种特殊的文件 `d`
* 设备文件：每一个外部设备是一个文件夹 `l`

文件权限，9 个字母，每 3 个字符一组

* `d rwx rwx rwx`


* 第一组：文件所有者对该文件所拥有的权限
* 第二组：文件所有组所在组的其他成员，对该文件所拥有的权限
* 第三组：与文件所有组不同组的其他成员，对该文件所拥有的权限

文件权限的表示

* `r : 100` 读
* `w : 010` 写
* `x : 001` 执行
* 每一组对应 3 个二进制位，也对应八进制，如
  * `rwx : 111 : 7`
  * `r_x : 101 : 5`

改变文件权限

* `chmod mod 文件名`  
  * mod 可以是 r / w / x
  * 也可以是 777
  * 如 `chmod 754 /home/test/test`
* `chmod [who] + / - mode 文件名`
  * who
    * `u` ：针对所有用户 user
    * `g` ：针对文件拥有组 group
    * `o` ：针对不同组 other
  * mode ： r / w / x
  * 如 `chmod u-r+x, g+r, o+w test`
* 对于文件夹，需要递归修改的话，加 `-R`

### linux 开发环境搭建

#### Java

```bash
$ wget http://download.oracle.com/otn-pub/java/jdk/9.0.4+11/c2514751926b4512b076cc82f959763f/jdk-9.0.4_linux-x64_bin.tar.gz
/usr/local/java$ tar -zxvf apache-tomcat-9.0.6.tar.gz

# vim /etc/profile
# java
JAVA_HOME=/usr/local/java/jdk1.8.0_141
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
export JAVAHOME PATH CLASSPATH
```

#### Tomcat

下载

```bash
~$ sudo wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.6/bin/apache-tomcat-9.0.6.tar.gz
```

创建目录

```bash
~$ mkdir /usr/local/tomcat
```

将压缩包复制到该目录下，并解压缩

```bash
~$ cp apache-tomcat-9.0.6.tar.gz /usr/local/tomcat
~$ cd /usr/local/tomcat
/usr/local/tomcat$ tar -zxvf apache-tomcat-9.0.6.tar.gz
```

进入解压后的 bin 目录，启动 tomcat

```bash
/usr/local/tomcat$ cd apache-tomcat-9.0.6/bin
/usr/local/tomcat/apache-tomcat-9.0.6/bin$ ./startup.sh
```

8080 端口问题

```bash
$ iptables  -I  INPUT  -p  tcp  --dport  8080  -j  ACCEPT  
$ /etc/init.d/iptables  save
$ /etc/init.d/iptables  restart
```

#### mysql

安装

```bash
$ sudo apt-get install mysql-server
```

配置远程访问

1. 开启远程 ip 访问

```bash
# 修改 mysql 配置文件 /etc/mysql/mysql.conf.d/mysqld.conf
vim /etc/mysql/mysql.conf.d/mysqld.conf

# 找到
bind-address=127.0.0.1

# 注释掉
# bind-address=127.0.0.1

# 或者
bind-address=0.0.0.0
```

2. 开启服务端远程访问

```mysql
# 进入数据库，授权远程 root 登录权限
mysql>grant all privileges on *.* to 'root'@'%' identified by 'root123456' with grant option;
flush privileges;
```

3. 重启 mysql

```bash
$ service mysql restart
```

3. 远程访问

```bash
# -h 远程连接 ip
# -P 远程连接 port
# -u 远程连接 root 用户
$ mysql -h 192.168.3.85 -P 3306 -u root -p      
```



### Nginx

#### 简介

可支持 3~5 万高并发，HTTP ／**反向代理**服务器，也是 IMAP／POP3／SMTP 代理服务器，适合配置服务器集群

#### 正向代理

客户端代理技术，客户端通过正向代理服务器，访问不同服务器上的资源

#### 反向代理

服务端代理技术，客户端通过反向代理服务器，访问不同后端服务器上的资源

#### 负载均衡 *load balancing*

由一个服务器（反向代理服务器）负责处理请求，分发给多个服务器，减少并发压力，提高吞吐量

#### 动静分离

Nginx 服务器直接将静态资源返回给客户端，而不转发给后端服务器处理，提高系统吞吐量

#### 安装 Nginx

下载安装包

移动并解压

```bash
~$ mv nginx-1.8.0.tar.gz /usr/local/nginx
/usr/local/nginx$ sudo tar -xzvf nginx-1.8.0.tar.gz 
```

启动配置文件，检查依赖完整性

```bash
/usr/local/nginx/nginx-1.8.0$ sudo ./configure
# the HTTP rewrite module requires the PCRE library.
sudo apt-get install libpcre3 libpcre3-dev
#  the HTTP gzip module requires the zlib library.
sudo apt-get install zlib1g-dev # ZLIB1G 

# make
sudo make && sudo make install
```

启动

```bash
/usr/local/nginx/sbin$ sudo ./nginx
```

### 配置 Nginx 集群

#### 开启多个 **Tomcat** 服务器

1. 相同 Web App
2. 复制多份
3. 修改 `server.xml` 端口
4. 两个分别启动 `./startup.sh`

```xml
<Server port="8006" shutdown="SHUTDOWN">
   <Connector port="8090" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
</Server>
```

#### 配置集群

编辑配置文件

```bash
/usr/local/nginx/conf$ vim nginx.conf
```

```nginx
// 添加
upstream com.sample.www {
server 127.0.0.1:8080;
server 127.0.0.1:8090;
}

location / {
root   html;
index  index.html index.htm;
proxy_pass http://com.sample.www;
}
```

让配置文件生效

```bash
/usr/local/nginx/sbin$ ./nginx -s reload
```

#### 基本操作

```bash
# 启动 nginx
/usr/local/nginx/sbin$ sudo ./nginx
# 让配置文件生效 nginx
/usr/local/nginx/sbin$ ./nginx -s reload
# 重新打开日志文件
/usr/local/nginx/sbin$ ./nginx -s reopen
# 测试配置文件是否正确
/usr/local/nginx/sbin$ ./nginx -t -c ../conf/nginx.conf
# 关闭 nginx，快速停止
/usr/local/nginx/sbin$ ./nginx -s stop
# 关闭 nginx，完整有序的停止
/usr/local/nginx/sbin$ ./nginx quit

# 其他停止方式
ps -ef | grep nginx
kill -QUIT 进程号 # 从容停止
kill -TERM 进程号 # 快速停止
pkill -9 nginx # 强制停止
```

#### 查看 Web 请求落地到哪台服务器

```nginx
location / {
root   html;
index  index.html index.htm;
proxy_pass http://com.sample.www;
# 增加下面两行
add_header backendIP $upstream_addr;
add_header backendCode $upstream_status;
}
```

#### 负载均衡策略 （upstream 用法）

##### 轮询 （weight = 1）

默认，选项，当 **weight** 不指定时，默认为 1

每个请求按时间顺讯逐一分配到不同的后端服务器，如果某个服务器 **Down** 掉，能自动剔除

```nginx
upstream test {
server 127.0.0.1:8080;
server 127.0.0.1:8090;
}
```

##### weight

指定轮训权重，与访问比率成正比，用于服务器性能不均的情况，某个服务器 **Down** 掉，能自动剔除

```nginx
upstream test {
server 127.0.0.1:8080 weight=1;
server 127.0.0.1:8090 weight=2;
}
```

##### ip_hash

每个请求访问按 **ip** 的 **hash** 结果分配，这样每个访客固定访问一个后端服务器

可以解决 **session** 跨服务器问题，如果某个服务器 **Down** 掉，要手工剔除

```nginx
upstream test {
server 127.0.0.1:8080;
server 127.0.0.1:8090;
ip_hash;
}
```

##### fair （第三方插件）

按后端服务器的响应时间分配请求，响应时间短的优先分配

```nginx
upstream test {
server 127.0.0.1:8080;
server 127.0.0.1:8090;
fair;
}
```

##### url_hash（第三方插件）

按访问 url 的 hash 结果分配，每个 url 固定访问一个都断服务器，适合缓存服务器

```nginx
upstream test {
server 127.0.0.1:8080;
server 127.0.0.1:8090;
hash $request_uri;
hash_method crc32;
}
```

##### 设备状态

`down` 当前服务器不参与请求分配

`weight` 权重，默认 **1**，权重越大，负载越多

`max_fails` 允许请求失败的次数，默认 **1**，若超过最大次数，`proxy_next_upstream` 模块定义错误

`fail_timeout` 超过最大请求失败次数后，再次请求暂停时间

`backup` 备用服务器，当非 **backup** 服务器 **down** 或者 忙的时候，请求备用服务器

```nginx
upstream test {
  #ip_hash;
  server 192.168.3.85:8080;
  server 192.168.3.85:8080 weight=100 down;
  server 192.168.3.86:8080 weight=100;
  server 192.168.3.87:8080 weight=100 backup;
  server 192.168.3.88:8080 weight=100 max_fails=3 fail_timeout=30s;
}
```

注意：当只有一个服务器时，`max_fails` 和 `fail_timeout` 可能不会起作用

#### 动静分离（反向代理）

1. 所有 **jsp** 的页面交由 **tomcat** 处理

```nginx
location ~ .(jsp|jspx|do)?$ {
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_pass http://127.0.0.1:8080;
}
```

2. 所有静态文件由 **nginx** 直接读取，不分配给 **tomcat**

```nginx
location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3) {
  expires 15d;
}
location ~ .*.(js|css)?$ {
  expires 1h;
}
```

3. 定义错误提示页面

```nginx
error_page   500 502 503 504 /50x.html;
location=/50x.html {
   root   html;
}
```

### 集群 Session 共享解决方案

问题：**Session** 产生于单个服务器中，通过 **nginx** 访问同一个页面多次，访问不同服务器，导致创建多个 **Session**，**Session** 不同步问题导致用户需要登录多次

#### Session sticky

使用 `ip_hash`，针对某 **ip** 的访问固定到一台服务器

局限性

* 若某台服务器重新启动，与这台服务器相关的 **Session** 就会丢失，用户还需再次登录
* **nginx** 需要计算，保存，维护 **ip** 和响应服务器之间的映射关系，内存消耗大

#### Session Replication

使用 **web.xml** `<distributable/>`，广播分发同步机制

每个服务器上都有集群中其他服务器的 **Session** 数据副本，同时通过服务器之间的**同步功能**保证 **Session** 在各个服务器的一致性

局限性

* 一旦 **Session** 数据发生变化，所有服务器都需要同步变化的 **Session** 数据，消耗带宽
* 每台服务器存储了其他服务器的 **Session** 副本，冗余数据多

#### Session 数据集中控制

使用 `Redis`，将所有服务器上的 **Session** 集中存储和管理，所有服务器共享一份 **Session** 数据

局限性

* 由于 **Session** 并步存储在本地，对 **Session** 的访问需要通过网络，可能产生访问延时和不稳定性，不过网络通信基本发生在内网，问题不大

#### Cookie based

使用 `URLEncode` ，将 **Session** 数据保存到 **Cookie** 中，浏览器访问时，服务器可以从 **Cookie** 中生成相应的 **Session** 信息

局限性

* **Cookie** 长度限制，**4 KB**，限制 **Sessi  on** 数据长度
* 安全性，写入到外部客户端，暴露给外部，就算加密也不太安全
* 带宽消耗，数据中心整体外部带宽的消耗







