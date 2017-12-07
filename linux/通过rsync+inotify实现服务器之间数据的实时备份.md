#通过rsync+inotify实现服务器之间数据的实时备份

我们只使用了rsync来实现数据的备份，但却不能做到实时，因为守护进程触发和执行有一段的时间间隔，如果对于数据量大的网站来说，一旦崩溃，就很难做到数据的完整性。在本篇文章中我们使用rsync+inotify的方式来解决这一问题。 
inotify是一种强大的、细粒度的、异步的文件系统时间监控机制，Linux内核从2.6.13版本开始就加入了对它的支持，通过它可以监控文件系统中添加、删除、修改、移动等各种细微事件。 
接下来使用rsync+inotify搭建实时同步系统： 
现在有两台服务器：A服务器（网站服务器）、B服务器（备份服务器），当inotify工具检测到A服务器上文件发生变化时，就会检测差异，然后同步到B服务器上。 

##1、安装inotify工具inotify-tools

	yum install inotify-tools

##2、配置B服务器上的rsyncd.conf文件

```
# /etc/rsyncd: configuration file for rsync daemon mode

# See rsyncd.conf man page for more options.

# configuration example:

 uid = nobody
 gid = nobody
 use chroot = no
 max connections = 10
 strict modes = no
 pid file = /var/run/rsyncd.pid
 lock file = /var/run/rsyncd.lock
 log file = /var/run/rsyncd.log
# exclude = lost+found/
# transfer logging = yes
# timeout = 900
# ignore nonreadable = yes
# dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2

# [ftp]
#        path = /home/ftp
#        comment = ftp export area
[betterlife]
path = /var/www
comment = betterlife file
ignore errors
read only = no
write only = no
hosts allow = *
list = false
uid = root
gid = root
auth users = backup
secrets file = /etc/server.pass
```
其中server.pass文件内容如下：

	backup:密码
	
修改server.pass文件权限

	chmod 600 /etc/server.pass


然后启动rsync守护进程

	rsync --daemon

将rsync加入到自启问价中

	echo "rsync --daemon" >>/etc/rc.local
	
##3、配置A服务器	

新建同步脚本

```
#!/bin/bash

inotifywait -mrq --timefmt '%d/%m/%y %H:%M' --format '%T %w%f%e' -e close_write,delete,create,attrib 监测目录 \
| while read files
        do
        rsync -vzrtopg --delete --progress --password-file=/etc/server.pass 同步文件目录   backup@服务器地址::模块名
                echo "${files} was rsynced" >>/tmp/rsync.log 2>&1
        done
          
``` 

  
其中server.pass内容如下：

	密码

修改脚本文件权限

	chmod 755 tongbu.sh

运行该脚本，并放入到后台
	
	./tongbu.sh >> tongbu.log &

最后将该命令放入到自启文件中
	
	echo "./inotifyrsync.sh >>/var/www/inotifyrsync.log &" >>/etc/rc.local

至此，实时同步系统就算搭建完毕，但在搭建过程中，遇到一个问题，就是，将A服务器文件rsyncd.conf中的用户名改成其他的名字，执行过程中会出现unauthorized user错误，但将用户名改为backup就不会出现该错误，也不知道是什么原因。 

利用以上方式，结合crontab定时守护进程，可以完成定时数据库备份，并且将备份的数据库同步到另外一台服务器上去。 
定时备份数据库代码如下：

	cronab -e
	0 0 * * 0  mysqldump -uroot -p密码 需要备份的数据库 >> 备份文件存放位置
	