从零配置，需要的基础环境包括：
### 0.    基础环境
            
虚拟机（virtual Box或者VMWare都可以），Ubuntu或者其他OS的iso镜像，安装好虚拟机。或者真实环境的公司服务器（若此种情况，可能一些配置已经不需要了，比如jdk、ssh，即这节内容可以忽略而过，可以直接跳到安装二）
            
选3台机器或者更多机器作为集群环境，一台作为Master节点，>=2台作为Slave节点
            
Hadoop集群主要有两个节点类型：NameNode、DataNode。当然还有secondaryNameNode和TaskTracker

### 1.    Java JDK环境
        
java环境不必说必须要有的，若没有，可以去官网下载jdk7或者jdk8。Linux下载xx.tar.gz，可以使用迅雷或者别的下载工具，或者在linux环境下wget url方式进行压缩包下载，或者直接使用apt-get方式下载安装。个人不太喜欢用apt-get方式下载JDK，速度可能比较慢。
        
如果是压缩包，安装包下载完之后，进行解压，-C选项解压到指定目录（/usr/local/）

```
tar zxvf xxx.tar.gz -C /usr/local/
```

然后添加一个软链

```
ln -s /usr/local/jdk1.8.0_101 jdk
```

如果想要采用apt-get方式下载，命令如下：

```
apt-get update
apt-get install default-jdk
```

Java环境变量设置，在/etc/profile文件下新增如下内容

```
export JAVA_HOME=/usr/local/jdk
export PATH=$PATH:$JAVA_HOME/bin
```

使设置立即生效

```
source /etc/profile
```


###   2.    SSH环境    
SSH是Hadoop集群环境下进行相互通讯的媒介工具。若无的话，需要进行安装。注：ssh分为客户端和服务端两种，ubuntu等Linux一般默认会安装client。
        
ssh安装如下

```
sudo apt-get openssh-server
```

ssh安装完之后会启动，通过22端口进行监听，可以通过netstat -nat进行验证是否服务开启。可通过ssh localhost进行测试，需要输入用户密码。

###    3.    建立hadoop用户
为了方便以后Hadoop操作，我们添加一个hadoop账号（以root用户）

```
useradd hadoop
```

若在/home/下没有生成hadoop，那么就创建一个hadoop目录

```
# cd /home
# mkdir hadoop
```

同时更改用户和用户组

```
chown -R hadoop:hadoop hadoop
```

注：若无hadoop用户组，则进行添加groupadd hadoop
  

其他注意事项：
            
当我们添加完hadoop之后，此时是root用户身份。若遇到如下情况：
            
在切换用户时，比如切换hadoop用户（su hadoop）后，再通过sudo -s切换root时，需要输入当前hadoop的密码，但是第一次添加后不知道密码没添加或者忘记各种情况下，只能先exit退出到root用户。或者切换用户时，不想输入密码。

可进行如下操作：

设置sudoers文件

```
vim /etc/sudoers
```

添加如下内容

```
hadoop  ALL=(ALL:ALL) NOPASSWD:ALL  #若每次验证密码可以将NOPASSWD去掉 hadoop ALL=(ALL:ALL) ALL
```

保存后即可以无密码进行用户随意切换了。

### 4.    机器间通信的SSH设置
若想要机器间进行通讯，则需要借助SSH工具。设置hadoop用户在机器间无密码访问。
            首先选定一台作为主机Master，其他作为Slave，即通过Master机器可以连接Slave机器。在Master上，进行如下操作：
            切换至hadoop用户，生成密钥

```
hadoop@yy-vm:/home$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): 
Created directory '/home/hadoop/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
6c:1e:bc:9e:8e:83:50:da:5d:33:83:1c:0d:00:41:8a hadoop@yy-vm
The key's randomart image is:
+--[ RSA 2048]----+
| .+o...o         |
|..    . .        |
|E    . o         |
|    . oo=        |
|   + . .S+       |
|  o . .o o       |
|   . .  o        |
|    . .o .       |
|      .o+        |
+-----------------+
```

一直回车就可以了，起初会询问是否生成到默认路径（/home/hadoop/.ssh），后面是询问密码设置，可忽略

.ssh是隐藏文件，可通过ls -la进行查看，或通过ll（需设置别名alias，在/etc/bash.bashrc下追加alias ll='ls -la --color=auto'，注：不同系统文件名可能不同）查看。文件夹下有id_rsa密钥和id_rsa.pub公钥两个文件。
            在其他Slave机器上同样执行命令生成密钥。
       
执行完成后，回到Master，执行命令

```
cat /home/hadoop/.ssh/id_rsa.pub >> authorized_keys
```

将Master的公钥信息加入到authorized_keys文件中，此命令会自动生成authorized_keys文件。然后将此文件传输到其他Slave机器上

```
scp /home/hadoop/.ssh/authorized_keys targethost:/home/hadoop/.ssh
```

最后通过Master机器进行验证ssh无密码登陆Slave，

```
ssh targethost #Eg: ssh 172.26.67.3
```

第一次连接可能需要输入hadoop用户密码，之后就不需要了。
    至此Hadoop的安装准备工作就完成了。

说明：新环境可能需要上述步骤进行设置。若是在公司服务器上，可能这些基本环境已经搭建过了。就不需要此环节。特别说明下，有些博客里面提到设置/etc/hosts，这个不是必须的，虚拟机可以自己设置下，其实就是设置更便捷的访问方式，通过更直观的域名进行IP映射。

比如对于172.26.67.3这个IP来说，在hosts文件下追加如下内容：

```
sudo vim /etc/hosts
172.26.67.3  Master.Hadoop
```

通过访问Master.Hadoop和172.26.67.3是一样的效果，可以通过ssh Master.Hadoop进行验证。

第一次连接可能会出现下面的情况，询问是否需要继续连接，输入yes就可以了，之后再连接就可直接进去了。

```
Are you sure you want to continue connecting (yes/no)?
```

如果在Master需要访问其他的Slave机器，可在Master的/etc/hosts继续追加 slaveIP Slave.Hadoop 这样的映射关系，反过来Slave访问Master也是一样的操作。
