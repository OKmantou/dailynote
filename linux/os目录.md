## /etc

- /etc/pam.d

  包含了Pluggable Authentication Modules（PAM）相关的配置文件。PAM是一种用于管理系统认证的框架，它允许系统管理员通过配置不同的认证模块来规定不同用户的登录过程、密码验证和访问控制。

- /etc/security/

  对用户的资源限制

  - /etc/security/limit.conf

    设置用户或用户组的资源限制，由Pluggable Authentication Modules (PAM) 来实现。用户每次登录都会受到限制。内核启动gdm-session-worker的进程来监控和限制

- /etc/sysconfig/：系统服务和组件的目录，通常以服务或组件命名
  - etc/sysconfig/network-script/

​				ifcfg-{网卡名}:定义网卡的各种信息

​				ifdown/ifup-{}:定义关闭和开启的前置操作

- /etc/systemd/

​		与systemd有关的目录：

​		/usr/lib/systemd/：操作系统设置的

​		/etc/systemd/system/：系统管理员的程序

​		/etc/systemd/user/：用户级的程序

​		/var/lib/systemd/:其中的coredump是转储文件

- /etc/ntp

  时钟

## /sys

一个虚拟文件系统，提供了对内核数据结构和设备驱动程序的访问接口

## /usr


- /usr/bin和/usr/local/bin的区别

  /usr/bin存放通过包管理器安装的程序；/usr/local/bin可以存放用户本地下载的程序

  /usr/bin的优先级大

## /var/log

- /var/log/secure

  涉及输入用户密码的所有软件登录信息

- /var/log/message

  系统发生的错误信息

- /var/log/cron

  定时任务crontab日志