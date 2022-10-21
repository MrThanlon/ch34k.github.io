---
title: ldap-python基本使用
tags:
  - ldap
  - python
abbrlink: 30835
date: 2020-04-20 22:21:32
---

因为[StarSSO](https://github.com/MrThanlon/starsso)这个项目，我不得不专门补了下LDAP的有关知识和概念，这里记录一下。

## LDAP基本概念

LDAP(Lightweight Directory Access Protocol，轻型目录访问协议)是一个协议，但是我们一般认为它用于存储用户数据，比如手机号、电子邮箱、密码（当然是加密后的）、用户名之类的信息。

和大多数数据库一样，它有服务端和客户端，服务端存储，客户端查询。

从名字我们就知道它是一种目录结构，典型的比如一般文件系统的文件夹就是一种目录，每个目录下可以存放多个条目（**entry**），一般来说一个条目就是一个用户或者组织，条目中可以存放多个属性（**attribute**），这些属性就是上面提到的用户名（一般是**cn**，即common name）、密码（一般是**userPassword**）、电子邮箱、手机号等等。一般一个条目必须拥有的属性是objectClass，用来表示条目的类型，比如**person**或者**orgnizationalUnit**，可以认为不同的objectClass可以拥有不同的属性。

每个条目都有一个唯一的**dn**（distinguished name，专有名称），比如说`cn=admin,dc=example,dc=com`就是一条dn，它表示在`com`这个dc下的`example`这个dc下有个cn为admin的条目，通常LDAP在安装之后的管理员账户就是`cn=admin,dc=nodomain`。

关于LDAP的更多知识可以在[这个网站](http://www.ldap.org.cn)找到，这里就不展开了。

## python-ldap

如果我们使用的编程语言是Python，可以使用[python-ldap](https://www.python-ldap.org)这个包来访问LDAP，Linux下安装需要编译，可以安装openldap或者openldap-dev（取决于发行版）来获得相应的头文件和链接库。

然而这个包的文档十分简单粗暴，它本身只是对LDAP的C语言接口做了简单封装，所以各种C语言的函数比如add/add_s都完美继承了下来，大量的函数都带有\_s/\_ext/\_ext\_s，我也是看了C语言接口的文档才知道\_s的意思是同步（Synchronize），也就是说会阻塞知道执行完成，而不带\_s的函数是异步执行，返回一个消息，然后通过另一个函数来获得执行的结果。

使用这个包看文档是不够的，更多时候应该关注官方仓库里的Demo和调试器窗口，因为文档里根本不会告诉你返回值是个什么东西。

下面介绍一些常用的操作：

### [ldap.initialize](https://www.python-ldap.org/en/python-ldap-3.2.0/reference/ldap.html#ldap.initialize)

连接LDAP数据库，返回一个LDAPObject，用于后续的各种操作。

