# Linux 常用服务类相关命令

## 常用基本命令 - 进程类

> > Service ( centos6)

注册在系统中的标准化程序

有方便统一的管理方式(常用的方法)

```sh
service 服务名 start
service 服务名 stop
service 服务名 restart
service 服务名 reload
service 服务名 status

#查看服务的方法 /etc/init.d/ 服务名
#通过 chkconfig 命令设置自启动
#查看服务 hkconfig --list rabbitmq

chkconfig --level 5 服务名 on
```

------

## 运行级别 runlevel(centos6)

![](http://120.77.237.175:9080/photos/eight/linux/01.png)

> Linux系统有7种运行级别(runlevel): **常用的是级别3和5**

- 运行级别0:系统停机状态，系统默认运行级别不能设为0，否则不能正常启动
- 运行级别1:单用户工作状态，root权限，用于系统维护，禁止远程登陆
- 运行级别2:多用户状态(没有NFS),不支持网络
- 运行级别3:完全的多用户状态(有NFS),登陆后进入控制台命令行模式
- 运行级别4:系统未使用，保留
- 运行级别5: X11控制台，登陆后进入图形GUI模式
- 运行级别6:系统正常关闭并重启，默认运行级别不能设为6,否则不能正常启动.

> Systemctl ( centos7 )

注册在系统中的标准化程序
有方便统一的管理方式(常用的方法)

```sh
systemctl start 服务名(xxx.service
systemct restart 服务名(xxxx.service)
systemctl stop 服务名(xxxx.service)
systemctl reload 服务名(xxxx.service)
systemctl status 服务名(xxxx.service)

#查看服务的方法 /usr/lib/systemd/system
#查看服务的命令
systemctl list-unit-files
systemctl --type service

#通过systemctl命令设置自启动
自启动systemctl enable service_name
不自启动systemctl disable service_name
```

