#centos 安装java, tomcat和mysql

标签（空格分隔）： web开发 环境配置

---

## 安装Java
### 1. 在线安装java
1. 查看yum库中有哪些版本：
```
yum search java | grep jdk
```
2. 选择安装版本（这里选择1.7版本）
```
yum install java-1.7.0-openjdk
```
默认安装目录为/usr/lib/jvm/java-1.7.0-openjdk-*-x86_64 ,"*"对应的内容根据安装目录中的文件名称进行相应的修改
3. 设置环境变量
```
vi /etc/profile
```
在profile文件中添加如下内容：
```
# set java environment
JAVA_HOME=/usr/lib/jvm/java-1.7.0-openjdk-*.x86_64
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```
让修改内容生效
```
source /etc/profile
```
4. 验证
```
java -version
```
如果安装成功，则会显示java版本信息

### 2. 离线安装java
[java安装包下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
离线情况下安装java有两种方法,一种是通过rpm包安装，一种是通过tar.gz压缩包安装。
#### 2.1 通过压缩包安装
1. 在/usr/目录下创建java目录
```
sudo mkdir /usr/java
```
2. 进入java安装所在目录，解压缩
```
tar -zxvf jdk-8u121-linux-i586.tar.gz -C /usr/java
```
3. 设置环境变量
```
vi /etc/profile
```
在profile中添加如下内容:
```
# set java environment
JAVA_HOME=/usr/java/jdk1.8.0_121
JRE_HOME=/usr/java/jdk1.8.0_121/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```
让修改内容生效
```
source /etc/profile
```
4. 验证
```
java -version
```
若安装成功则会输出java版本信息。

#### 2.2 通过rpm包安装
1. 进入rpm安装包所在目录，使用rpm命令进行安装
```
rpm -ivh jdk-8u121-linux-i586.rpm
```
2. rpm命令会将java默认安装在/usr/java目录下，并通过三重链接链接到/usr/bin，配置环境变量：
```
vi /etc/profile
```
添加以下内容
```
JAVA_HOME=/usr/java/jdk-1.8.0_121
JRE_HOME=/usr/java/jdk-1.8.0_121/jre
CLASS_PATH=.$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
```
让修改生效
```
source /etc/profile
```

## 安装Tomcat
1. 准备[tomcat安装包](http://tomcat.apache.org/download-80.cgi#8.0.43)，此处使用的是tomcat 8.0.43安装包
2. 在/usr/local/目录下创建一个名为kencery的文件夹
```
sudo mkdir /usr/local/kencery
```
3. 将压缩包拷贝到该文件夹中
```
sudo cp packge_path/apache-tomcat-8.0.43.tar.gz /usr/local/kencery
```
4. 解压缩并修改文件夹名称 
```
cd /usr/local/kencery
sudo tar -zxvf ./apache-tomcat-8.0.43.tar.gz
sudo mv apache-tomcat-8.0.43 tomcat
```
加sudo的原因是权限可能不够
5. 运行tomcat
```
sudo /usr/local/kencery/tomcat/bin/startup.sh
```
若提示"Tomcat started."则说明tomcat安装成功。
6. 若java已经安装成功，但运行tomcat时提示"Neither the JAVA_HOME nor the JRE_HOME environment variable is defined"，则需要在/usr/local/kencery/tomcat/bin/setclasspath.sh文件中的"# Make sure prerequisite environment variables are set"前面添加以下内容
```
export JAVA_HOME=YourJdkPath
export JRE_HOME=YourJdkPath/jre
```
保存并退出（编辑文件时需要使用sudo权限，否则无法保存）

## 安装MySql(使用root用户)
1. 查看是否已经安装mysql.
```
rpm -qa | grep -i mysql
```
2. 如果有，则需要卸载，否则Mysql-server会安装失败。若没有，则跳至步骤3.
```
rpm -e --nodeps mysql-lib-*
rm -rf /var/lib/mysql
rm /etc/my.cnf
```
查看是否还有mysql软件:
```
rpm -qa | grep mysql
```
有的话继续删除
3. 下载[mysql离线安装包](http://cdn.mysql.com/Downloads/MySQL-5.6/MySQL-5.6.26-1.linux_glibc2.5.x86_64.rpm-bundle.tar),可以使用wget命令直接下载。
4. 解压
```
tar -xvf MySQL-5.6.26-1.linux_glibc2.5.x86_64.rpm-bundle.tar
```
5. 安装
在RHEL系统中，必须先安装“MySQL-shared-compat-5.6.26-1.el6.i686.rpm”这个兼容包，然后才能安装server和client，否则安装时会出错。
```
rpm -ivh MySQL-shared-compat-5.6.26-1.linux_glibc2.5.x86_64.rpm  # RHEL兼容包
rpm -ivh MySQL-server-5.6.26-1.linux_glibc2.5.x86_64.rpm        # MySQL服务端程序
rpm -ivh MySQL-client-5.6.26-1.linux_glibc2.5.x86_64.rpm        # MySQL客户端程序
rpm -ivh MySQL-devel-5.6.26-1.linux_glibc2.5.x86_64.rpm         # MySQL的库和头文件
rpm -ivh MySQL-shared-5.6.26-1.linux_glibc2.5.x86_64.rpm        # MySQL的共享库
```
6. 配置
配置MySQL登录密码
```
cat /root/.mysql_secret  # 获取MySQL安装时生成的随机密码
service mysql start      # 启动MySQL服务
mysql -uroot -p          # 进入MySQL，使用之前获取的随机密码
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('password');  # 在MySQL命令行中设置root账户的密码为password
quit  # 退出MySQL命令行
service mysql restart  # 重新启动MySQL服务
```


参考：
> https://my.oschina.net/JustLoveIT/blog/499208

> http://www.cnblogs.com/hanyinglong/p/5024643.html

> http://os.51cto.com/art/201609/517037.htm?mobile
