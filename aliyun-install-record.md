[TOC]

# 一、安装前的准备工作

​	新购买的阿里云服务器ECS，只用root用户；如果没有指定主机名，那么给定一个默认的随机串作为主机名，比如：`iZ2zeccmg1u7pjn8jm0wg4Z`这样的字符串。

​	**注意：以下内容针对CentOS7版本操作系统**

## 修改主机名

```bash
[root@iZ2zeccmg1u7pjn8jm0wg4Z ~]# hostnamectl set-hostname emon
```

修改后需要**重新登录**root用户，才能看到新的主机名。

```bash
[root@emon ~]# cat /etc/hostname 
emon
```



## 修改本地域名解析

打开`/etc/hosts`文件，编辑主机名`iZ2zeccmg1u7pjn8jm0wg4Z`为`emon`即可。



## 更换yum源配置为阿里云源配置

1. 备份

```bash
[root@emon ~]# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

2. 下载新的`CentOS-Base.repo`到`/etc/yum.repos.d/`目录

```bash
[root@emon ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

3. 生成缓存

```bash
[root@emon ~]# yum makecache
```



## 创建具有sudo权限的普通用户

1. 创建普通用户

```bash
[root@emon ~]# useradd -c "Web Site User" emon
```

2. 修改密码

```bash
[root@emon ~]# passwd emon
```

3. 赋权sudo

root用户以`visudo`命令打开文件，在99行添加： `emon    ALL=(ALL)       ALL`即可。



## 开启SELinux

阿里云默认关闭SELinux，请开启。

```bash
[root@emon ~]# vim /etc/selinux/config 
```

找到`SELINUX=disabled`修改为`SELINUX=enforcing`，这种修改**需要重启**系统才能生效。



## 开启firewalld防火墙

阿里云默认关闭firewalld服务，请开启。

```bash
[root@emon ~]# systemctl start firewalld
```



## 修改vim的缩进为4个空格

```bash
[root@emon ~]# vim /etc/vimrc 
```

打开文件后，在最后一行追加如下内容：

```
set tabstop=4
set softtabstop=4
set shiftwidth=4
set expandtab
```

​	以上设置，是一个新的阿里云服务器ECS在安装软件之前，需要预先设置的内容；设置完成后，重启系统使SELinux配置生效。

​	别忘记**创建快照**哦！



# 二、安装项目所需软件

---

由于下面采用emon进行安装，安装目录在`/usr/local/`，这里先修改目录的属主。

```bash
[emon@emon ~]$ sudo chown -R emon /usr/local/
[emon@emon ~]$ ll -d /usr/local/
drwxr-xr-x. 13 emon root 4096 Feb 24  2017 /usr/local/
```



下面安装时，默认切换到`/usr/local/`目录执行命令。

---



## 安装JDK

1. 检查是否已安装

```bash
[emon@emon local]$ rpm -qa|grep jdk
```

2. 下载

下面的下载地址，可以通过ORACLE官网下载页，登录后获取：

官网下载页地址： http://www.oracle.com/technetwork/java/javase/downloads/index.html

```bash
[emon@emon local]$ wget -cP /usr/local/src/ http://111.1.62.31/files/216700000680F6BC/download.oracle.com/otn-pub/java/jdk/8u161-b12/2f38c3b165be4555a1fa6e98c45e0808/jdk-8u161-linux-x64.tar.gz
```

3. 创建安装目录

```bash
[emon@emon local]$ mkdir /usr/local/Java
```

4. 解压安装

```bash
[emon@emon local]$ tar -zxvf /usr/local/src/jdk-8u161-linux-x64.tar.gz -C /usr/local/Java/
```

5. 创建软连接

```bash
[emon@emon local]$ ln -s /usr/local/Java/jdk1.8.0_161/ /usr/local/java
```

6. 配置环境变量

```bash
[emon@emon local]$ sudo vim /etc/profile
```

打开文件，在最后一行追加如下内容：

```bash
export JAVA_HOME=/usr/local/java
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
```

使之生效：

```bash
[emon@emon local]$ source /etc/profile
```

7. 校验

```bash
[emon@emon local]$ java -version
java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)
```



## 安装Tomcat

1. 下载

下载地址获取页面： https://tomcat.apache.org/whichversion.html

```bash
[emon@emon local]$ wget -cP /usr/local/src/ http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.28/bin/apache-tomcat-8.5.28.tar.gz
```

2. 创建安装目录

```bash
[emon@emon local]$ mkdir /usr/local/Tomcat
```

3. 解压安装

```bash
[emon@emon local]$ tar -zxvf /usr/local/src/apache-tomcat-8.5.28.tar.gz -C /usr/local/Tomcat
```

4. 创建软连接

```bash
[emon@emon local]$ ln -s /usr/local/Tomcat/apache-tomcat-8.5.28/ /usr/local/tomcat
```

5. 配置环境变量

```bash
[emon@emon local]$ sudo vim /etc/profile
```

打开文件，在最后一行追加如下内容：

```bash
export CATALINA_HOME=/usr/local/tomcat
export PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$PATH
```

使之生效：

```bash
[emon@emon local]$ source /etc/profile
```

6. 配置UTF-8字符集

打开文件`/usr/local/tomcat/conf/server.xml ` 找到8080默认端口的配置位置，在xml节点末尾增加`URIEncoding=UTF-8` ，修改后的内容如下：

```xml
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" URIEncoding="UTF-8"/>
```

7. 校验

   **关于阿里云安装tomcat无法启动8005端口问题的修复**

如果使用`startup.sh` 命令启动后，再使用`shutdown.sh` 命令停止会发现如下错误：

......

SEVERE: Could not contact [localhost:[8005]]. Tomcat may not be running.
Mar 10, 2018 11:19:24 PM org.apache.catalina.startup.Catalina stopServer
SEVERE: Catalina.stop: 
java.net.ConnectException: Connection refused (Connection refused)

......

问题的症结在于80005端口没有启动，导致关闭时无法连接到8005端口。

修复如下：

- 修改`$JAVA_HOME/jre/lib/security/java.security` 文件中的securerandom.source配置项

```
securerandom.source=file:/dev/random
-->修改为：
securerandom.source=file:/dev/urandom
```

- 使用`killall` 命令杀掉java进程

```bash
[emon@emon local]$ killall java
```

- 重新启动tomcat

```bash
[emon@emon local]$ startup.sh 
[emon@emon local]$ sudo netstat -tnlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      2780/java     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1240/sshd     
tcp        0      0 127.0.0.1:8005          0.0.0.0:*               LISTEN      2780/java     
tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      2780/java  
```



## 安装Maven

1. 下载

下载地址获取页面： https://maven.apache.org/download.cgi

```bash
[emon@emon local]$ wget -cP /usr/local/src/ http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.5.3/binaries/apache-maven-3.5.3-bin.tar.gz
```

2. 创建安装目录

```bash
[emon@emon local]$ mkdir /usr/local/Maven
```

3. 解压安装

```bash
[emon@emon local]$ tar -zxvf /usr/local/src/apache-maven-3.5.3-bin.tar.gz -C /usr/local/Maven/
```

4. 创建软连接

```bash
[emon@emon local]$ ln -s /usr/local/Maven/apache-maven-3.5.3/ /usr/local/maven
```

5. 配置环境变量

```bash
[emon@emon local]$ sudo vim /etc/profile
```

打开文件，在最后一行追加如下内容：

```bash
export MAVEN_HOME=/usr/local/maven
export PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$MAVEN_HOME/bin:$PATH
```

使之生效：

```bash
[emon@emon local]$ source /etc/profile
```

6. 校验

```bash
[emon@emon local]$ mvn -v
Apache Maven 3.5.3 (3383c37e1f9e9b3bc3df5050c29c8aff9f295297; 2018-02-25T03:49:05+08:00)
Maven home: /usr/local/maven
Java version: 1.8.0_161, vendor: Oracle Corporation
Java home: /usr/local/Java/jdk1.8.0_161/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-514.6.2.el7.x86_64", arch: "amd64", family: "unix"
```



## 安装vsftpd

1. 检查是否安装

```bash
[emon@emon local]$ rpm -qa|grep vsftpd
```

2. 使用yum安装

```bash
[emon@emon local]$ sudo yum -y install vsftpd
```

3. 备份`vsftpd.conf` 配置文件

```bash
[emon@emon local]$ sudo cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
```

4. 创建文件服务器根目录`/fileserver`

首先，`fileserver` 并非ftp专享的目录，而是ftp、ftps、sftp这三种文件服务器共享的根目录。

```bash
[emon@emon local]$ sudo mkdir /fileserver
```

5. 创建ftp本地用户

```bash
[emon@emon local]$ sudo useradd -d /fileserver/ftproot -s /sbin/nologin -c "Ftp User" ftpuser
```

创建用户后，自动创建了`/fileserver/ftproot/` 目录，但是该目录权限为700，需要修改为755

```bash
[emon@emon local]$ sudo chmod -R 755 /fileserver/ftproot/
```

为了创建本地用户模式+虚拟用户模式，都可以登录ftp服务器，这里设置ftpuser用户的密码

```bash
[emon@emon local]$ sudo passwd ftpuser
```

6. 虚拟用户模式需要如下准备

   - 配置虚拟用户

   ```bash
   [emon@emon local]$ sudo vim /etc/vsftpd/virtual_user_list
   ```

   文件内容说明： 奇数行是虚拟用户名，偶数行是前一行用户名对应的密码。

   ```
   ftp
   [这里输入密码]
   extra
   [这里输入密码]
   ```

   - 根据配置的虚拟用户，生效虚拟用户数据库文件

   ```bash
   [emon@emon local]$ sudo db_load -T -t hash -f /etc/vsftpd/virtual_user_list /etc/vsftpd/virtual_user_list.db
   [emon@emon local]$ file /etc/vsftpd/virtual_user_list.db 
   /etc/vsftpd/virtual_user_list.db: Berkeley DB (Hash, version 9, native byte-order)
   ```

   - 配置支持虚拟用户的PAM认证文件，引用生成的虚拟用户数据库文件（默认带`.db` 后缀，无需指定）

   ```bash
   [emon@emon local]$ sudo vim /etc/pam.d/vsftpd 
   ```

   打开文件，在文件头非注释行开始，插入如下内容（插入的内容必须第一行开始）：

   ```
   auth sufficient pam_userdb.so db=/etc/vsftpd/virtual_user_list
   account sufficient pam_userdb.so db=/etc/vsftpd/virtual_user_list
   ```

7. 配置`vsftpd.conf`

```bash
[emon@emon local]$ sudo vim /etc/vsftpd/vsftpd.conf
```



```
# 不允许匿名用户登录【修改】
anonymous_enable=NO
# 允许本地用户登录
local_enable=YES
# 本地用户可以在自己家目录中进行读写操作
write_enable=YES
# 本地用户新增档案时的umask值
local_umask=022
# 如果启动这个选项，那么使用者第一次进入一个目录时，会检查该目录下是否有.message这个档案，如果有，则会出现此档案的内容，通常这个档案会放置欢迎话语，或是对该目录的说明。默认值为开启
dirmessage_enable=YES
# 是否启用上传/下载日志记录。如果启用，则上传与下载的信息将被完整纪录在xferlog_file 所定义的档案中。预设为开启。
xferlog_enable=YES
# 指定FTP使用20端口进行数据传输，默认值为YES
connect_from_port_20=YES
# 如果启用，则日志文件将会写成xferlog的标准格式
xferlog_std_format=YES
# 这里用来定义欢迎话语的字符串【新增】
ftpd_banner=Welcome to emon FTP service.
# 用于指定用户列表文件中的用户是否允许切换到上级的目录【新增】
chroot_local_user=NO
# 用于设置是否启用chroot_list_file配置项指定的用户列表文件【新增】
chroot_list_enable=YES
# 用于指定用户列表文件【新增】
chroot_list_file=/etc/vsftpd/chroot_list
listen=NO
listen_ipv6=YES

# 设置PAM使用的名称，默认值为/etc/pam.d/vsftpd
pam_service_name=vsftpd
# 是否启用vsftpd.user_list文件，黑名单，白名单都可以的
userlist_enable=YES
tcp_wrappers=YES

# 虚拟用户创建文档的umask值
anon_umask=022
# 是否启用虚拟用户，默认值为NO。【新增】
guest_enable=YES
# 这里用来映射虚拟用户，默认为ftp。【新增】
guest_username=ftpuser
# 当不允许本地用户+虚拟用户切换到主目录上级时，对于虚拟用户而言，可以登录；对于本地用户而言，会报错： 500 OOPS: vsftpd: refusing to run with writable root inside chroot()
# 两种做法，第一种是去掉用户主目录的写权限，第二种是增加如下属性
allow_writeable_chroot=YES
# 默认是GMT时间，改成使用本机系统时间【新增】
use_localtime=YES
# 为虚拟用户设置独立的权限【新增】
user_config_dir=/etc/vsftpd/virtual_user_dir

# 被动模式及其使用的端口范围【新增】
pasv_enable=YES
pasv_min_port=61001
pasv_max_port=62000
```

8. 创建配置属性`chroot_list_file` 和`user_config_dir` 所需要的目录和文件，并创建ftp服务器根目录`/fileserver/ftproot/`下一个index.html文件

   - `chroot_list_file`所需

   ```bash
   [emon@emon local]$ sudo vim /etc/vsftpd/chroot_list
   ```

   文件内容：

   ```
   ftp
   extra
   ```

   - `user_config_dir`所需

   首先，`user_config_dir`属性指定的值是一个目录，在该目录下需要为虚拟用户创建同名的权限文件，比如虚拟用户`ftp`的权限文件，命名为`ftp`。

   创建指定目录：

   ```bash
   [emon@emon local]$ sudo mkdir /etc/vsftpd/virtual_user_dir
   ```

   为虚拟用户`ftp` 和`extra` 创建权限控制文件：

   ```bash
   [emon@emon local]$ sudo vim /etc/vsftpd/virtual_user_dir/ftp
   ```

   文件内容：

   ```
   anon_upload_enable=YES
   anon_mkdir_write_enable=YES
   anon_other_write_enable=YES
   ```

   ```bash
   [emon@emon local]$ sudo vim /etc/vsftpd/virtual_user_dir/extra
   ```

   文件内容：

   ```
   #先不填写，预留。
   ```

   - 创建`index.html`文件

   ```bash
   [emon@emon local]$ sudo vim /fileserver/ftproot/index.html
   ```

   ​

   **由于sudo创建的，属于root用户，最好修改为ftpuser用户所有**

   > [emon@emon local]$ sudo chown ftpuser:ftpuser /fileserver/ftproot/index.html 

   ```html
   <html>
       <head>
           <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
           <link href="https://cdn.bootcss.com/bootstrap/4.0.0/css/bootstrap.min.css" rel="stylesheet">
       </head>
       <body>
           <h1>
           </h1>
           <div class="container">
               <div class="row clearfix">
                   <div class="col-md-12 column">
                       <div class="jumbotron">
                           <h1>
                               Welcome to emon FTP service.
                           </h1>
                           <p>
                               <h3>
                                   为了可以预见的忘却，为了想要进阶的自己。
                               </h3>
                               <h3>
                                   种一棵树最好的时间是十年前，其次是现在。
                               </h3>
                           </p>
                       </div>
                   </div>
               </div>
           </div>
       </body>
   </html>
   ```

9. 配置SELinux对ftp服务器目录`/fileserver/ftproot/` 的限制

查看限制情况：

```bash
[emon@emon local]$ getsebool -a|grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
```

放开限制：

```bash
[emon@emon local]$ sudo setsebool -P ftpd_full_access=on
```

10. 校验

- 启动vsftpd

```bash
[emon@emon local]$ sudo systemctl start vsftpd
```

为了ftp登录，需要安装ftp客户端：

```bash
[emon@emon local]$ yum list ftp|tail -n 2
Available Packages
ftp.x86_64                           0.17-67.el7                            base
[emon@emon local]$ sudo yum -y install ftp
```

- 登录ftp验证

```bash
[emon@emon local]$ ftp 127.0.0.1
Connected to 127.0.0.1 (127.0.0.1).
220 Welcome to emon FTP service.
Name (127.0.0.1:emon): ftp
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (127,0,0,1,241,215).
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001         1514 Mar 11 01:20 index.html
226 Directory send OK.
ftp> mkdir lm
257 "/lm" created
ftp> ls
227 Entering Passive Mode (127,0,0,1,240,163).
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001         1514 Mar 11 01:20 index.html
drwxr-xr-x    2 1001     1001         4096 Mar 11 01:41 lm
226 Directory send OK.
ftp> exit
221 Goodbye
```



## 安装ftps服务器

​	**ftps=ftps+ssl**

- 为ftp添加ssl功能的方式
  - 使用自签名证书
  - 使用私有CA签名证书
  - 使用公信CA签名证书

### openssl安装检查

由于要使用到openssl，这里先检查openssl安装情况。

```bash
[emon@emon ~]$ ldd which vsftpd|grep ssl
	libssl.so.10 => /lib64/libssl.so.10 (0x00007fd8b3113000)
```

或者：

```bash
[emon@emon ~]$ yum list openssl|tail -n 4
Installed Packages
openssl.x86_64                   1:1.0.1e-60.el7_3.1                    @updates
Available Packages
openssl.x86_64                   1:1.0.2k-8.el7                         base    
```



### 方式一：使用自签名证书

1. 切换目录

```bash
[emon@emon ~]$ cd /etc/ssl/certs/
[emon@emon certs]$ ls
ca-bundle.crt  ca-bundle.trust.crt  make-dummy-cert  Makefile  renew-dummy-cert
```

2. 生成RSA私钥和自签名证书

```bash
[emon@emon certs]$ sudo openssl req -newkey rsa:2048 -nodes -keyout rsa_private.key -x509 -days 365 -out cert.crt
[sudo] password for emon: 
Generating a 2048 bit RSA private key
..+++
......+++
writing new private key to 'rsa_private.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:ZheJiang
Locality Name (eg, city) [Default City]:HangZhou
Organization Name (eg, company) [Default Company Ltd]:HangZhou emon Technologies,Inc.
Organizational Unit Name (eg, section) []:IT emon
Common Name (eg, your name or your server's hostname) []:*.emon.vip
Email Address []:[邮箱]
```

3. 查看生成的RSA私钥和自签名证书

```bash
[emon@emon certs]$ ls
ca-bundle.crt  ca-bundle.trust.crt  cert.crt  make-dummy-cert  Makefile  renew-dummy-cert  rsa_private.key
```

4. 配置`vsftpd.conf`

```
# ssl config
# 是否使用ssl
ssl_enable=YES
# 是否允许匿名用户使用ssl
allow_anon_ssl=NO
# 强制本地用户登录使用ssl
force_local_logins_ssl=YES
# 强制本地用户数据使用ssl传输
force_local_data_ssl=YES
# 强制匿名/虚拟用户登录使用ssl
force_anon_logins_ssl=YES
# 强制匿名/虚拟用户数据使用ssl传输
force_anon_data_ssl=YES
# 允许 TLS v1 协议连接
ssl_tlsv1=YES
# 允许 SSL v2 协议连接
ssl_sslv2=YES
# 开启sslv3
ssl_sslv3=YES

# 是否启用隐式SSL功能，不建议开启，而且默认是关闭的
implicit_ssl=NO
# 隐式ftp端口设置，如果不设置，默认还是21，但是当客户端以隐式SSL连接时，默认会使用990端口，导致连接失败！！！
# listen_port=990
# 输出SSL相关的日志信息
# debug_ssl=YES
# Disable SSL session reuse(required by WinSCP)
require_ssl_reuse=NO
# Select which SSL ciphers vsftpd will allow for encrypted SSL connections（required by FileZilla）
ssl_ciphers=HIGH
# 自签证书：证书文件
rsa_cert_file=/etc/ssl/certs/cert.crt
# 自签证书：RSA私钥文件
rsa_private_key_file=/etc/ssl/certs/rsa_private.key
```

ssl有显式`explicit`和隐式`implicit`之分：

- 显式配置

```
implicit_ssl=NO
```

- 隐式配置

```
implicit_ssl=YES
listen_port=990
```

5. 重启vsftpd服务

```bash
[emon@emon certs]$ sudo systemctl restart vsftpd
```

6. 校验

对于ftps的校验，无法使用ftp命令校验了：

```bash
[emon@emon certs]$ ftp 127.0.0.1
Connected to 127.0.0.1 (127.0.0.1).
220 Welcome to emon FTP service.
Name (127.0.0.1:emon): ftp
530 Anonymous sessions must use encryption.
Login failed.
421 Service not available, remote server has closed connection
ftp> 
```

如果是显式`explicit` 的ftps，还可以使用lftp测试：

- 安装lftp

```bash
[emon@emon certs]$ sudo yum install -y lftp
[emon@emon certs]$ lftp ftp@127.0.0.1:21
Password: 
lftp ftp@127.0.0.1:~> ls          
ls: Fatal error: Certificate verification: Not trusted
lftp ftp@127.0.0.1:~> 
```

- 编辑`/etc/lftp.conf`

在最后一行追加：

```
set ssl:verify-certificate no
```

再次校验：

```bash
[emon@emon certs]$ lftp ftp@127.0.0.1:21
Password: 
lftp ftp@127.0.0.1:~> ls
-rw-r--r--    1 1001     1001         1514 Mar 11 09:20 index.html
lftp ftp@127.0.0.1:/> 
```

如果是隐式的ftps，lftp就无法校验了，除非lftp是` compiled with OpenSSL (configure --with-openssl)`：

```bash
[emon@emon certs]$ lftp ftp@127.0.0.1:990
Password: 
lftp ftp@127.0.0.1:~> ls
`ls' at 0 [FEAT negotiation...]
```

怎么办呢？ **推荐使用Windows操作系统的FlashFXP软件验证。**



### 方式二：使用私有CA签名证书

私有CA签名证书的使用与自签名证书一样的，这里不再赘述，主要讲解如何生成私有CA签名证书。

#### 相关知识点

> - 证书签发机构CA
>   - 公共信任CA
>
>     大范围维护大量证书企业使用OpenCA（对openssl进行了二次封装，更加方便使用）
>
>   - 私有CA
>
>     小范围测试使用openssl
>
> - openssl配置文件
>
>   > /etc/pki/tls/openssl.cnf

#### 创建私有证书签发机构CA步骤

​	在确定配置为CA的服务器主机上生成一个自签证书，并为CA提供所需要的目录及文件。在真正的通信过程中CA服务器主机不需要网络参与，只需要参与到签名中，不需要提供服务。

1. 生成私钥

因为在默认配置文件中CA自己的私钥配置在`/etc/pki/CA/private/cakey.pem`，所以指定目录和文件名要和配置文件一致。

```bash
[emon@emon certs]$ sudo bash -c "umask 077;openssl genrsa -out /etc/pki/CA/private/cakey.pem 4096"
Generating RSA private key, 4096 bit long modulus
.....................++
...........................................++
e is 65537 (0x10001)
[emon@emon certs]$ sudo ls -l /etc/pki/CA/private/cakey.pem
-rw-------. 1 root root 3243 Mar 11 12:52 /etc/pki/CA/private/cakey.pem
```

2. 生成CA自签证书

```bash
[emon@emon certs]$ sudo openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 3655
[sudo] password for emon: 
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:ZheJiang
Locality Name (eg, city) [Default City]:HangZhou
Organization Name (eg, company) [Default Company Ltd]:HangZhou emon Technologies,Inc.
Organizational Unit Name (eg, section) []:IT emon
Common Name (eg, your name or your server's hostname) []:*.emon.vip
Email Address []:[邮箱] 
```

命令解释：

- `/etc/pki/CA/cacert.pem` : CA自签证书默认位置
- `-new` : 生成新证书签署请求
- `-x509` ： 生成自签格式证书，专用于创建私有CA时
- `-key` ： 生成请求时用到的私有文件路径
- `-out` ： 生成的请求文件路径，如果自签操作将直接生成签署过的证书
- `-days` ： 证书的有效时长，单位是day

注意：

- `-key /etc/pki/CA/private/cakey.pem` 指明的是私钥的位置，只是因为此处会自动抽取出私钥中的公钥。
- req只能发起签署请求，需要加-x509参数实现自己发出请求，自己签署。非自签无需此参数。

3. 为CA提供所需的目录及文件

   当不存在时需要创建签发证书、吊销证书、新证书目录

```bash
[emon@emon certs]$ sudo mkdir -pv /etc/pki/CA/{certs,crl,newcerts}
```

​	创建证书序列号文件、证书索引文件

```bash
[emon@emon certs]$ sudo touch /etc/pki/CA/{serial,index.txt}
```

​	第一次创建的时候需要给予证书序列号

```bash
[emon@emon certs]$ echo 01 | sudo tee /etc/pki/CA/serial 
01
```



#### OpenSSL：服务申请证书签署实现SSL安全通信

​	要用到证书进行安全通信的服务器，需要向CA请求签署证书，需要签署的服务无需和CA证书签署机构主机在同一台服务器上。

1. 用到证书的服务器生成私钥

生成vsftpd服务的私钥创建时候无需在`/etc/pki/CA/private`目录创建，该目录仅在创建CA主机时需要的。

```bash
[emon@emon certs]$ sudo mkdir /etc/vsftpd/ssl
[emon@emon certs]$ cd /etc/vsftpd/ssl/
[emon@emon ssl]$ sudo bash -c "umask 077; openssl genrsa -out /etc/vsftpd/ssl/vsftpd.key 2048"
Generating RSA private key, 2048 bit long modulus
.................................+++
...............................................................+++
e is 65537 (0x10001)
[emon@emon ssl]$ ll
total 4
-rw-------. 1 root root 1679 Mar 11 14:01 vsftpd.key
```

2. 生成证书签署请求

```bash
[emon@emon ssl]$ sudo openssl req -new -key /etc/vsftpd/ssl/vsftpd.key -out /etc/vsftpd/ssl/vsftpd.csr -days 365
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:ZheJiang
Locality Name (eg, city) [Default City]:HangZhou   
Organization Name (eg, company) [Default Company Ltd]:HangZhou emon Technologies,Inc.
Organizational Unit Name (eg, section) []:IT emon
Common Name (eg, your name or your server's hostname) []:*.emon.vip
Email Address []:[邮箱]
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

命令解释：

- `*.csr` ： 表示证书签署请求文件
- 要保证和签署机构CA签署机构信息一致

生成签名请求时，有两项额外的信息需要填写：

| 字段                       | 说明      | 示例   |
| ------------------------ | ------- | ---- |
| A challenge password     | 高强度的密码  | 无需填写 |
| An optional company name | 可选的公司名称 | 无需填写 |

3. 将请求通过可靠方式发送给CA主机

> [emon@emon ssl]$ sudo scp /etc/vsftpd/ssl/vsftpd.csr root@127.0.0.1:/tmp/
> [sudo] password for emon: 
> The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
> ECDSA key fingerprint is 24:9a:cf:6b:82:1c:1a:12:2c:ef:bd:e9:8d:6e:83:cc.
> Are you sure you want to continue connecting (yes/no)? yes
> Warning: Permanently added '127.0.0.1' (ECDSA) to the list of known hosts.
> root@127.0.0.1's password: 
> vsftpd.csr                                                                                100% 1098     1.1KB/s   00:00    

4. 在CA主机上签署证书

```bash
[root@emon ssl]# cd /tmp/
[root@emon tmp]# openssl ca -in /tmp/vsftpd.csr -out /etc/pki/CA/certs/vsftpd.crt -days 365
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Mar 11 06:34:53 2018 GMT
            Not After : Mar 11 06:34:53 2019 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = ZheJiang
            organizationName          = HangZhou emon Technologies,Inc.
            organizationalUnitName    = IT emon
            commonName                = *.emon.vip
            emailAddress              = [邮箱]
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                78:06:3A:9F:2B:36:E3:30:13:FE:EE:A7:1D:34:6B:D2:BD:D3:62:CC
            X509v3 Authority Key Identifier: 
                keyid:CF:94:F2:84:94:6B:28:30:4D:AB:61:3B:D3:EC:C4:BE:DD:8B:50:13

Certificate is to be certified until Mar 11 06:34:53 2019 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

5. 查看所签署的证书信息

- 方法一

```bash
[root@emon tmp]# cat /etc/pki/CA/index.txt
V	190311063453Z		01	unknown	/C=CN/ST=ZheJiang/O=HangZhou emon Technologies,Inc./OU=IT emon/CN=*.emon.vip/emailAddress=[邮箱]
```

`V` ： 表示已经签署的

`01` ： 表示证书序列号

`/C=CN/ST=ZheJiang/O=......`： 表示主题信息

- 方法二



```bash
[root@emon tmp]# openssl x509 -in /etc/pki/CA/certs/vsftpd.crt -noout -serial -subject
serial=01
subject= /C=CN/ST=ZheJiang/O=HangZhou emon Technologies,Inc./OU=IT emon/CN=*.emon.vip/emailAddress=[邮箱]
```

`serial`： 序列号

`subject` ： 主题信息

6. 将CA签署机构的.crt证书发送给服务器

```bash
[root@emon tmp]# scp /etc/pki/CA/certs/vsftpd.crt root@127.0.0.1:/etc/vsftpd/ssl/
root@127.0.0.1's password: 
vsftpd.crt                                                                                100% 6020     5.9KB/s   00:00    
```

7. 删除服务器和CA主机上签署前的`*.csr`文件，确保安全

CA主机： 

```bash
[root@emon tmp]# rm -rf /tmp/vsftpd.csr 
```

vsftpd主机：

```bash
[emon@emon ssl]$ sudo rm -rf /etc/vsftpd/ssl/vsftpd.csr 
```



### 方式三：使用公信CA签名证书

其实，方式二已经讲解了如何向CA申请证书，只不过那个是私有CA而已。

步骤如下：

1. 用到证书的服务器生成私钥
2. 生成证书签署请求
3. 将请求通过可靠方式发送给CA主机

## 安装sftp

​	sftp是Secure File Transfer Protocol的缩写，安全文件传输协议。sftp没有单独的守护进程，它必须使用sshd守护进程（默认端口号是22）来完成相应的连接和答复操作。

1. sftp用户和sftp用户组的规划

| 用户        | 所属分组       | 宿主目录                                     |
| --------- | ---------- | ---------------------------------------- |
| sftpadmin | sftpadmin  | /fileserver/sftproot/sftpadmin/sftpadmin |
| sftpuser1 | sftpnormal | /fileserver/sftproot/sftpnormal/sftpuser1 |
| sftpuser2 | sftpnormal | /fileserver/sftproot/sftpnormal//sftpuser2 |

- 敲黑板，划重点：
  - `/sftpadmin`和/`sftpnormal`及上级目录的属主必须是root，否则Chroot会拒绝连接。
  - `/sftpadmin` 目录规划了高级组的用户组目录；属主是root，属组是sftpadmin。
  - `/sftpnormal` 目录规划了普通组的用户组目录；属主是root，属组是sftpnormal。
  - `/sftpadmin` 和`/sftpnormal`的子目录对应sftp用户；属主与属组归具体用户所有。

2. 创建用户组

```bash
[emon@emon local]$ sudo groupadd sftpadmin
[emon@emon local]$ sudo groupadd sftpnormal
```

3. 创建用户

创建用户所需目录：

```bash
[emon@emon local]$ sudo mkdir -p /fileserver/sftproot/{sftpadmin,sftpnormal}
```

创建sftp用户：

```bash
[emon@emon local]$ sudo useradd -g sftpadmin -d /fileserver/sftproot/sftpadmin/sftpadmin -s /sbin/nologin -c "Sftp User" sftpadmin
[emon@emon local]$ sudo useradd -g sftpnormal -d /fileserver/sftproot/sftpnormal/sftpuser1 -s /sbin/nologin -c "Sftp User" sftpuser1
[emon@emon local]$ sudo useradd -g sftpnormal -d /fileserver/sftproot/sftpnormal/sftpuser2 -s /sbin/nologin -c "Sftp User" sftpuser2
```

设置密码：

```bash
[emon@emon local]$ sudo passwd sftpadmin
[emon@emon local]$ sudo passwd sftpuser1
[emon@emon local]$ sudo passwd sftpuser2
```

查看权限：

```bash
[emon@emon local]$ ll /fileserver/sftproot/
total 8
drwxr-xr-x. 3 root root 4096 Mar 12 22:23 sftpadmin
drwxr-xr-x. 4 root root 4096 Mar 12 22:25 sftpnormal
[emon@emon local]$ ll /fileserver/sftproot/sftpadmin/
total 4
drwx------. 2 sftpadmin sftpadmin 4096 Mar 12 22:23 sftpadmin
[emon@emon local]$ ll /fileserver/sftproot/sftpnormal/
total 8
drwx------. 2 sftpuser1 sftpnormal 4096 Mar 12 22:25 sftpuser1
drwx------. 2 sftpuser2 sftpnormal 4096 Mar 12 22:25 sftpuser2
```

可以看到，用户目录`sftpadmin`、 `sftpuser1`、`sftpuser2` 的权限是700，它们的上级目录权限是755。

4. 配置`sshd_config`

```bash
[emon@emon local]$ sudo vim /etc/ssh/sshd_config 
```

注释掉下面这一行：

```
# Subsystem   sftp    /usr/libexec/openssh/sftp-server
```

在文件末尾追加sftp的配置

```
Subsystem   sftp    internal-sftp
Match Group sftpadmin
       ForceCommand internal-sftp
       ChrootDirectory /fileserver/sftproot/sftpadmin
Match Group sftpnormal
       X11Forwarding no
       AllowTcpForwarding no
       ForceCommand internal-sftp
       ChrootDirectory /fileserver/sftproot/sftpnormal
```

5. 重启`sshd`

```bash
[emon@emon local]$ sudo systemctl restart sshd
```

6. 校验

```bash
[emon@emon local]$ sftp sftpadmin@127.0.0.1
sftpadmin@127.0.0.1's password: 
Connected to 127.0.0.1.
sftp> ls
sftpadmin  
sftp> cd sftpadmin/
sftp> pwd
Remote working directory: /sftpadmin
sftp> mkdir lm
sftp> ls
lm  
sftp> 
```



## 安装Nginx

1. 下载

下载页： http://nginx.org/en/download.html

```bash
[emon@emon local]$ wget -cP /usr/local/src/ http://nginx.org/download/nginx-1.12.2.tar.gz
```

2. 安装依赖

```bash
[emon@emon local]$ yum list gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel open openssl-devel
[emon@emon local]$ sudo yum -y install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel open openssl-devel
```

3. 创建解压目录

```bash
[emon@emon local]$ mkdir /usr/local/Nginx
```

4. 解压

```bash
[emon@emon local]$ tar -zxvf /usr/local/src/nginx-1.12.2.tar.gz -C /usr/local/Nginx/
```

5. 执行配置脚本，并编译安装

   - 切换目录并执行配置脚本生成Makefile

   ```bash
   [emon@emon local]$ cd /usr/local/Nginx/nginx-1.12.2/
   [emon@emon nginx-1.12.2]$ ./configure --prefix=/usr/local/Nginx/nginx
   ```

   - 编译

   ```bash
   [emon@emon nginx-1.12.2]$ make
   ```

   - 安装

   ```bash
   [emon@emon nginx-1.12.2]$ make install 
   [emon@emon nginx-1.12.2]$ cd /usr/local/
   [emon@emon local]$ ls /usr/local/Nginx/nginx
   conf  html  logs  sbin
   ```

6. 备份主配置文件`nginx.conf`

```bash
[emon@emon local]$ cp -a /usr/local/Nginx/nginx/conf/nginx.conf /usr/local/Nginx/nginx/conf/nginx.conf.bak
```

7. 创建软连接

```bash
[emon@emon local]$ ln -s /usr/local/Nginx/nginx/ /usr/local/nginx
```

8. 配置环境变量【特殊】

​        由于nginx启动的是1024以下的端口，需要root权限，而sudo又不能引用`/etc/profile`和`~/.bash_rc`配置

的环境变量，就会导致`sudo: nginx: command not found`。

​	所以，采用软连接的方式：

```bash
[emon@emon local]$ sudo ln -s /usr/local/nginx/sbin/nginx /usr/sbin/nginx
```

9. 校验

```bash
[emon@emon local]$ nginx -V
nginx version: nginx/1.12.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC) 
configure arguments: --prefix=/usr/local/Nginx/nginx
[emon@emon local]$ nginx -v
nginx version: nginx/1.12.2
```

10. 配置`nginx.conf`

```bash
[emon@emon local]# vim /usr/local/nginx/conf/nginx.conf
```

打开文件，找到`HTTPS server`上一行，大约95行，添加如下内容：

```
    include vhost/*.conf;
```

创建文件夹`vhost` ：

```bash
[emon@emon local]# mkdir /usr/local/nginx/conf/vhost
```

创建一个虚拟主机，映射到ftp服务器：

```bash
[emon@emon local]# vim /usr/local/nginx/conf/vhost/file.emon.vip.conf
```



```nginx
server {
    listen 80;
    autoindex on;
    server_name 39.107.97.197 file.emon.vip;
    access_log /usr/local/nginx/logs/access.log combined;
    index index.html index.htm index.jsp index.php;
    #error_page 404 /404.html;
    if ( $query_string ~* ".*[\;'\<\>].*" ){
        return 404;
    }

    location / {
        root /fileserver/ftproot;
        add_header Access-Control-Allow-Origin *;
    }
}
```

11. 测试、启动、重新加载、停止

- 测试

`-t` Nginx服务器配置文件是否有语法错误，可以与`-c`一起使用，使输出内容更详细，这对查找配置文件中错误语法很有帮助。

```bash
[emon@emon local]$ sudo nginx -t -c /usr/local/nginx/conf/nginx.conf
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

- 启动

```bash
[emon@emon local]$ sudo nginx
```

- 重新加载

```bash
[emon@emon local]$ sudo nginx -s reload
```

- 停止

```bash
[emon@emon local]$ sudo nginx -s quit
```

12. 访问

因为是阿里云，所以确保`本实例安全组`开放了80端口的访问。

http://39.107.97.197/



## 安装MySQL

1. 检查是否安装

```bash
[emon@emon local]$ sudo rpm -qa|grep mysql
```

2. 下载

下载页地址： https://dev.mysql.com/downloads/mysql/

```bash
[emon@emon local]$ sudo wget -cP /usr/local/src/ https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz
```

3. 创建安装目录

```bash
[emon@emon local]$ mkdir /usr/local/MySQL
```

4. 解压安装

```bash
[emon@emon local]$ tar -zxvf /usr/local/src/mysql-5.7.21-linux-glibc2.12-x86_64.tar.gz -C /usr/local/MySQL/
```

5. 创建软连接

```bash
[emon@emon local]$ ln -s /usr/local/MySQL/mysql-5.7.21-linux-glibc2.12-x86_64/ /usr/local/mysql
```

6. 配置环境变量并使之生效

```bash
[emon@emon local]$ sudo vim /etc/profile
```

打开文件，在最后一行追加如下内容：

```bash
export PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$MAVEN_HOME/bin:/usr/local/mysql/bin:$PATH
```

使之生效：

```bash
[emon@emon local]$ source /etc/profile
```

7. 数据库目录规划

| 文件说明                     | 软连接位置                                    | 实际存储位置                        |
| ------------------------ | ---------------------------------------- | ----------------------------- |
| 数据datadir                | /usr/local/mysql/data                    | /data/mysql/data              |
| 参考文件my.cnf               | /usr/local/mysql/etc/my.cnf              |                               |
| 错误日志log-error            | /usr/local/mysql/log/mysql_error.log     |                               |
| 二进制日志log-bin             | /usr/local/mysql/binlogs/mysql-bin       | /data/mysql/binlogs/mysql-bin |
| 慢查询日志slow_query_log_file | /usr/local/mysql/log/mysql_slow_query.log |                               |
| 套接字socket文件              | /usr/local/mysql/run/mysql.sock          |                               |
| pid文件                    | /usr/local/mysql/run/mysql.pid           |                               |

备注：考虑到数据和二进制日志比较大，需要软链接：

```bash
[emon@emon local]$ sudo mkdir -p /data/mysql/{data,binlogs,log,etc,run}
[emon@emon local]$ sudo ln -s /data/mysql/data /usr/local/mysql/data
[emon@emon local]$ sudo ln -s /data/mysql/binlogs /usr/local/mysql/binlogs
[emon@emon local]$ sudo ln -s /data/mysql/log /usr/local/mysql/log
[emon@emon local]$ sudo ln -s /data/mysql/etc /usr/local/mysql/etc
[emon@emon local]$ sudo ln -s /data/mysql/run /usr/local/mysql/run
```

创建mysql用户，为`/data/mysql`和`/usr/local/mysql/{data,binlogs,log,etc,run}`赋权：

```bash
[emon@emon local]$ sudo useradd -s /sbin/nologin -M -c "MySQL User" mysql
[emon@emon local]$ sudo chown -R mysql.mysql /data/mysql/
[emon@emon local]$ sudo chown -R mysql.mysql /usr/local/mysql/{data,binlogs,log,etc,run}
```

8. 配置`my.cnf`

备份移除系统自带的my.cnf文件：

```bash
[emon@emon local]$ sudo mv /etc/my.cnf /etc/my.cnf.bak
```

在`/usr/local/mysql/etc/`下创建`my.cnf`文件并配置如下：

```bash
[emon@emon local]$ sudo vim /usr/local/mysql/etc/my.cnf
```



```
[client]
port = 3306
socket = /usr/local/mysql/run/mysql.sock

[mysqld]
port = 3306
socket = /usr/local/mysql/run/mysql.sock
pid_file = /usr/local/mysql/run/mysql.pid
datadir = /usr/local/mysql/data
default_storage_engine = InnoDB
max_allowed_packet = 512M
max_connections = 2048
open_files_limit = 65535

skip-name-resolve
lower_case_table_names=1

character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'

innodb_buffer_pool_size = 1024M
innodb_log_file_size = 2048M
innodb_file_per_table = 1
innodb_flush_log_at_trx_commit = 0

key_buffer_size = 64M

log-error = /usr/local/mysql/log/mysql_error.log
log-bin = /usr/local/mysql/binlogs/mysql-bin
slow_query_log = 1
slow_query_log_file = /usr/local/mysql/log/mysql_slow_query.log
long_query_time = 5

tmp_table_size = 32M
max_heap_table_size = 32M
query_cache_type = 0
query_cache_size = 0

server-id=1
```

9. 初始化数据库

```bash
[emon@emon local]$ sudo /usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/
```

在日志文件里会提示一个临时密码，记录这个密码：

```bash
[emon@emon local]$ sudo grep 'temporary password' /usr/local/mysql/log/mysql_error.log 
2018-03-13T00:20:48.579480Z 1 [Note] A temporary password is generated for root@localhost: .8%>1t.gv_;I
```

10. 生成SSL

```bash
[emon@emon local]$ sudo /usr/local/mysql/bin/mysql_ssl_rsa_setup --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/
Generating a 2048 bit RSA private key
.................................................+++
............+++
writing new private key to 'ca-key.pem'
-----
Generating a 2048 bit RSA private key
.....................................+++
...................................+++
writing new private key to 'server-key.pem'
-----
Generating a 2048 bit RSA private key
.........+++
.............................................................+++
writing new private key to 'client-key.pem'
-----
```

11. 设置启动项

```bash
[emon@emon local]$ sudo vim /usr/lib/systemd/system/mysqld.service
```



```
# Copyright (c) 2015, 2016, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
#
# systemd service file for MySQL forking server
#

[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql

Type=forking

PIDFile=/usr/local/mysql/run/mysqld.pid

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Execute pre and post scripts as root
PermissionsStartOnly=true

# Needed to create system tables
#ExecStartPre=/usr/bin/mysqld_pre_systemd

# Start main service
ExecStart=/usr/local/mysql/bin/mysqld --daemonize --pid-file=/usr/local/mysql/run/mysqld.pid $MYSQLD_OPTS

# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 65535

Restart=on-failure

RestartPreventExitStatus=1

PrivateTmp=false
```

加载启动项：

```bash
[emon@emon local]$ sudo systemctl daemon-reload
```

12. 启动mysql

```
[emon@emon local]$ sudo systemctl start mysqld.service
```

13. 初始化mysql服务程序


```bash
[emon@emon local]$ mysql_secure_installation 

Securing the MySQL server deployment.

Enter password for user root: `[输入mysqld --initialize时生成的临时密码]`

The existing password for the user account root has expired. Please set a new password.

New password: `[设置新密码]`

Re-enter new password: `[确认新密码]`

VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: y`[设置密码强度校验]`

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 2`[使用最强密码校验]`
Using existing password for root.

Estimated strength of the password: 100 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : n`[是否修改root用户密码]`

 ... skipping.
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y`[是否移除匿名账户]`
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y`[是否禁止root用户的远程登录]`
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y`[是否移除测试数据库]`
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y`[是否刷新权限表]`
Success.

All done! 
```

14. 测试

```bash
[emon@emon local]$ mysqladmin version -uroot -p
```

查看变量：

```bash
[emon@emon local]$ mysqladmin variables -uroot -p|wc -l
Enter password: 
507
```

登录：

```
[emon@emon local]$ mysql -uroot -p
```

停止：

```bash
[emon@emon ~]$ sudo systemctl stop mysqld
```

## 安装Git

1. 检查安装情况

```bash
[emon@emon local]$ yum list git|tail -n 2
Available Packages
git.x86_64                       1.8.3.1-12.el7_4                        updates
```

2. 下载

下载地址：  https://www.kernel.org/pub/software/scm/git/

```bash
[emon@emon local]$ wget -cP /usr/local/src/ https://www.kernel.org/pub/software/scm/git/git-2.16.2.tar.gz
```

3. 安装依赖

```bash
[emon@emon local]$ yum list gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel gcc gcc-c+ curl-devel expat-devel perl-ExtUtils-MakeMaker perl-ExtUtils-CBuilder cpio
[emon@emon local]$ sudo yum install -y gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel gcc gcc-c+ curl-devel expat-devel perl-ExtUtils-MakeMaker perl-ExtUtils-CBuilder cpio
```

4. 创建解压目录

```bash
[emon@emon local]$ mkdir /usr/local/Git
```

5. 解压

```bash
[emon@emon local]$ tar -zxvf /usr/local/src/git-2.16.2.tar.gz -C /usr/local/Git/
```

6. 执行配置脚本，并编译安装

   - 切换目录并执行配置脚本

   ```bash
   [emon@emon local]$ cd /usr/local/Git/git-2.16.2/
   [emon@emon git-2.16.2]$ ./configure --prefix=/usr/local/Git/git
   ```

   - 编译

   ```bash
   [emon@emon git-2.16.2]$ make
   ```

   - 安装

   ```bash
   [emon@emon git-2.16.2]$ make install
   [emon@emon git-2.16.2]$ cd /usr/local/
   [emon@emon local]$ ls /usr/local/Git/git
   bin  lib64  libexec  share
   ```

7. 创建软连接

```bash
[emon@emon local]$ ln -s /usr/local/Git/git/ /usr/local/git
```

8. 配置环境变量

```bash
[emon@emon local]$ sudo vim /etc/profile
```

打开文，在最后一行追加如下内容：

```bash
export GIT_HOME=/usr/local/git
export GIT_EDITOR=vim
export PATH=$JAVA_HOME/bin:$CATALINA_HOME/bin:$MAVEN_HOME/bin:/usr/local/mysql/bin:$GIT_HOME/bin:$PATH
```

使之生效：

```bash
[emon@emon local]$ source /etc/profile
```

9. 设置账户信息

```bash
[emon@emon local]$ git config --global user.name "emon"
[emon@emon local]$ git config --global user.email "[邮箱]"
```

10. 配置SSH信息

- 检查SSH keys是否存在：

```bash
[emon@emon local]$ ls -a ~/.ssh/
.  ..  known_hosts
```

- 如果不存在，生成SSH keys：

```bash
[emon@emon local]$ ssh-keygen -t rsa -b 4096 -C "[邮箱]"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/emon/.ssh/id_rsa): `[默认]`
Enter passphrase (empty for no passphrase): `[输入口令，其他用户切换到emon会提示输入]`
Enter same passphrase again: `[确认口令]`
Your identification has been saved in /home/emon/.ssh/id_rsa.
Your public key has been saved in /home/emon/.ssh/id_rsa.pub.
The key fingerprint is:
99:ad:4b:f3:9e:56:8a:36:0f:7f:94:9d:15:db:2c:80 [邮箱]
The key's randomart image is:
+--[ RSA 4096]----+
|           .     |
|          E .  . |
|             . .+|
|         +    ..+|
|        S .  o + |
|         .  + o  |
|        =. +     |
|       .+*o..    |
|       ..=*.     |
+-----------------+
[emon@emon local]$ ls -a ~/.ssh/
.  ..  id_rsa  id_rsa.pub  known_hosts
```

- 配置自动加载ssh-agent：

把下面的内容放入`~/.bashrc`或`~/.bash_profile` 即可。

```bash
[emon@emon local]$ vim ~/.bash_profile 
```

以下是关于SSH keys中私钥加载到ssh-agent的自动配置，无需每次登陆配置。

```bash
#以下是关于SSH keys中私钥加载到ssh-agent的自动配置，无需每次登陆配置
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2= agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add
fi

unset env
```

- 拷贝公钥到GitHub上【需要有GitHub账户才可以配置】

```bash
[emon@emon local]$ cat ~/.ssh/id_rsa.pub
```

拷贝了公钥，打开GitHub配置SSH keys的页面： https://github.com/settings/keys 【Settings->SSH and GPG keys->New SSH key->写入Title，粘贴Key】

| Title           | Key       |
| --------------- | --------- |
| aliyun-emon-rsa | 【刚才拷贝的公钥】 |

点击Add SSH key，确定添加。

- 验证SSH连接

```bash
[emon@emon local]$ ssh -T git@github.com
The authenticity of host 'github.com (192.30.253.113)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.253.113' (RSA) to the list of known hosts.
Enter passphrase for key '/home/emon/.ssh/id_rsa': `[生成SSH keys时设置的口令]`
Hi Rushing0711! You've successfully authenticated, but GitHub does not provide shell access.
[emon@emon local]$ ls -a ~/.ssh/
.  ..  id_rsa  id_rsa.pub  known_hosts
```
11. 校验

```bash
[emon@emon local]$ git --version
git version 2.16.2
```



##安装Redis 

1. 检查依赖环境

```bash
[emon@emon local]$ yum list gcc gcc-c++
```

2. 下载

下载地址页： https://redis.io/download

```bash
[emon@emon local]$ wget -cP /usr/local/src/ http://download.redis.io/releases/redis-4.0.8.tar.gz
```

3. 创建解压目录

```
[emon@emon local]$ mkdir /usr/local/Redis
```

4. 解压

```bash
[emon@emon local]$ tar -zxvf /usr/local/src/redis-4.0.8.tar.gz -C /usr/local/Redis/
```

5. 切换目录并执行编译

```bash
[emon@emon local]$ cd /usr/local/Redis/redis-4.0.8/
[emon@emon redis-4.0.8]$ make MALLOC=libc
cd src && make all
make[1]: Entering directory `/usr/local/Redis/redis-4.0.8/src'
    CC Makefile.dep
......`[省略输出]`
Hint: It's a good idea to run 'make test' ;)

make[1]: Leaving directory `/usr/local/Redis/redis-4.0.8/src'
```

​	注意：make命令执行完成编译后，会在src目录下生成6个可执行文件，分别是：

 - redis-server
 - redis-sentinel
 - redis-cli
 - redis-benchmark
 - redis-check-rdb
 - redis-check-aof 

6. 编译测试


```bash
[emon@emon redis-4.0.8]$ make test
cd src && make test
make[1]: Entering directory `/usr/local/Redis/redis-4.0.8/src'
    CC Makefile.dep
make[1]: Leaving directory `/usr/local/Redis/redis-4.0.8/src'
make[1]: Entering directory `/usr/local/Redis/redis-4.0.8/src'
You need tcl 8.5 or newer in order to run the Redis test
make[1]: *** [test] Error 1
make[1]: Leaving directory `/usr/local/Redis/redis-4.0.8/src'
make: *** [test] Error 2
```

看到错误，说需要tcl 8.5或更新的版本，安装：

```bash
[emon@emon redis-4.0.8]$ sudo yum -y install tcl
```

再次执行测试：

```bash
[emon@emon redis-4.0.8]$ make test
......
......`[省略输出]`
\o/ All tests passed without errors!

Cleanup: may take some time... OK
make[1]: Leaving directory `/usr/local/Redis/redis-4.0.8/src'
[emon@emon redis-4.0.8]$ cd /usr/local/
```

7. 配置化安装

严格意义上，只是把一些文件归集到一个目录，作为安装内容，所以叫做`配置化安装`。

   - 创建安装目录

   ```bash
   [emon@emon local]$ mkdir /usr/local/Redis/redis
   ```

   - 复制`redis-server`、 `redis-cli` 到安装目录

   ```bash
   [emon@emon local]$ cp /usr/local/Redis/redis-4.0.8/src/{redis-server,redis-cli} /usr/local/Redis/redis/
   ```

   - 复制`redis.config` 到安装目录

   ```bash
   [emon@emon local]$ cp /usr/local/Redis/redis-4.0.8/redis.conf /usr/local/Redis/redis/
   ```

   - 在安装目录创建RDB文件目录与`redis.log` 日志文件

   ```bash
   [emon@emon local]$ mkdir /usr/local/Redis/redis/redis_rdb
   [emon@emon local]$ touch /usr/local/Redis/redis/redis_rdb/redis.log
   ```

8.  创建软连接


```bash
[emon@emon local]$ ln -s /usr/local/Redis/redis/ /usr/local/redis
```

9. 配置`redis.conf`

```bash
[emon@emon local]$ vim /usr/local/redis/redis.conf 
```



```
# 守护进程方式运行【修改】
daemonize yes
# 这个文件需要手动提前创建好，表示日志文件【修改】
logfile "/usr/local/redis/redis_rdb/redis.log"
# RDB文件目录【修改】
dir /usr/local/redis/redis_rdb/
# 最大内存200M【新增】
maxmemory 200M
# 设置Redis连接密码，客户端在链接Redis时需要通过auth <password>命令提供密码，默认关闭。【新增】
requirepass [这里输入密码]
```

10. 增加开机服务

```bash
[emon@emon local]$ sudo vim /usr/lib/systemd/system/redisd.service
```

```
[Unit]
Description=The redis-server Process Manager
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/redis_6379.pid
ExecStart=/usr/local/redis/redis-server /usr/local/redis/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

加载启动项：

```bash
[emon@emon local]$ sudo systemctl daemon-reload
```

11. 启动与停止

```
[emon@emon local]$ sudo systemctl start redisd.service
[emon@emon local]$ sudo netstat -tnlp|grep redis
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      10237/redis-server 
[emon@emon local]$ sudo systemctl stop redisd.service
```

12. 校验

在Redis启动情况下：

连接：

```bash
[emon@emon local]$ /usr/local/redis/redis-cli 
127.0.0.1:6379> get name
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth `[输入密码]`
OK
127.0.0.1:6379> set name emon
OK
127.0.0.1:6379> get name
"emon"
127.0.0.1:6379> exit
```



#三、项目自动化部署

## 创建SpringBoot项目

利用IDEA创建SpringBoo的Web项目



## 提交到GitHub

1. 首先，在GitHub上创建` a new repository` 

   GitHub地址： https://github.com/

![GitHub上创建repository](C:\Users\LiMing\Downloads\1.png)

2. 拷贝生成的 `LICENSE` 、`README.md`到上一步创建的SpringBoot项目

![2](C:\Users\LiMing\Downloads\2.png)

3. 提交本地SpringBoot项目到GitHub上

- 在IDEA的note项目中打开Terminal
- 初始化仓库

```bash
$ git init
Initialized empty Git repository in D:/Job/JobResource/IdeaProjects/Idea2017/note/.git/
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        .gitignore
        .mvn/
        LICENSE
        README.md
        mvnw
        mvnw.cmd
        pom.xml
        src/

nothing added to commit but untracked files present (use "git add" to track)
```

- 添加note项目到暂存区

```bash
$ git add .
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   .gitignore
        new file:   .mvn/wrapper/maven-wrapper.jar
        new file:   .mvn/wrapper/maven-wrapper.properties
        new file:   LICENSE
        new file:   README.md
        new file:   mvnw
        new file:   mvnw.cmd
        new file:   pom.xml
        new file:   src/main/java/vip/emon/note/NoteApplication.java
        new file:   src/main/resources/application.properties
        new file:   src/test/java/vip/emon/note/NoteApplicationTests.java
```

- 提交到本地

```bash
$ git commit -am "first commit init project"
[master (root-commit) 49cf15e] first commit init project
 11 files changed, 686 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 .mvn/wrapper/maven-wrapper.jar
 create mode 100644 .mvn/wrapper/maven-wrapper.properties
 create mode 100644 LICENSE
 create mode 100644 README.md
 create mode 100644 mvnw
 create mode 100644 mvnw.cmd
 create mode 100644 pom.xml
 create mode 100644 src/main/java/vip/emon/note/NoteApplication.java
 create mode 100644 src/main/resources/application.properties
 create mode 100644 src/test/java/vip/emon/note/NoteApplicationTests.java
```

- 添加远程仓库

```bash
$ git remote add origin git@github.com:emonnote/note.git
```

- 提交到远程仓库

- ```bash
  $ git push -u -f origin master
  Counting objects: 27, done.
  Delta compression using up to 8 threads.
  Compressing objects: 100% (16/16), done.
  Writing objects: 100% (27/27), 50.50 KiB | 0 bytes/s, done.
  Total 27 (delta 0), reused 0 (delta 0)
  To github.com:emonnote/note.git
   + 2e21b04...49cf15e master -> master (forced update)
  Branch master set up to track remote branch master from origin.
  ```

- GitHub上查看

![3](C:\Users\LiMing\Downloads\3.png)

- 在本地查看分支

```bash
$ git branch -a
* master
  remotes/origin/master
```

## 采用Git flow方式开发

- 创建develop分支并推送到GitHub

```bash
$ git branch develop
$ git push -u origin develop
Total 0 (delta 0), reused 0 (delta 0)
To github.com:emonnote/note.git
 * [new branch]      develop -> develop
Branch develop set up to track remote branch develop from origin.
```





## 创建自动化发布脚本

1. 创建Git仓库目录

```bash
[emon@emon local]$ mkdir /usr/local/src/git-repository
```

2. 克隆GitHub上的仓库

```bash
[emon@emon local]$ cd /usr/local/src/git-repository/
[emon@emon git-repository]$ git clone git@github.com:emonnote/note.git
```

3. 创建脚本

```bash
[emon@emon git-repository]$ cd /usr/local/src/
[emon@emon src]$ vim deploy.sh 
```

```
echo "===========进入git项目note目录============="
cd /usr/local/src/git-repository/note

echo "==========git切换分之到note-develop==============="
git checkout develop

echo "==================git fetch======================"
git fetch

echo "==================git pull======================"
git pull

echo "===========编译并跳过单元测试===================="
mvn clean package -Dmaven.test.skip=true

echo "====================启动note====================="
mvn spring-boot:run
```

4. 执行自动化发布

```bash
[emon@emon src]$ ./deploy.sh 
```







# 四、服务器用户信息

## 阿里云服务器用户信息

| 用户名  | 密码            |
| ---- | ------------- |
| root | [密码] |
| emon | [密码]     |

## 文件服务器用户信息

### FTP服务器

| 用户名     | 密码           |
| ------- | ------------ |
| ftpuser | FtpUser@0711 |
| ftp     | ftp@163      |
| extra   | extra@123456 |



### SFTP服务器

| 用户名       | 密码           |
| --------- | ------------ |
| sftpadmin | sftp@163     |
| sftpuser1 | extra@123456 |
| sftpuser2 | extra@123456 |



## MySQL用户信息

| 用户名  | 密码       |
| ---- | -------- |
| root | root@163 |



## Git用户信息

| 用户名          | 密码       |
| ------------ | -------- |
| SSH切换到emon用户 | emon@163 |



## Redis用户信息

| 用户名            | 密码        |
| -------------- | --------- |
| 在redis.conf启动后 | redis@163 |

