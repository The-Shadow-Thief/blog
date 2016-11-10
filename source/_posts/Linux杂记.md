---
title: Linux杂记
date: 2016-04-11 16:51:57
toc: true
tags:
  - OS
categories:
  - Linux
---

其实Linux有很多的发行版,用过的有Centos6.5和Ubuntu14.04,想写篇文章用来记录在使用和部署服务器过程中的问题,以及常用的Linux命令和对Linux的理解.

<!--more-->
### **基本目录介绍**

1. `/bin` 该目录下的命令可以被root与一般账号所使用，由于这些命令在挂接其它文件系统之前就可以使用，所以/bin目录必须和根文件系统在同一个分区中。

2. `/sbin`该目录下存放系统命令，即只有系统管理员（俗称最高权限的root）能够使用的命令，系统命令还可以存放在/usr/sbin,/usr /local/sbin目录下，/sbin目录中存放的是基本的系统命令，它们用于启动系统和修复系统等，与/bin目录相似，在挂接其他文件系统之前就 可以使用/sbin，所以/sbin目录必须和根文件系统在同一个分区中。

3. `/dev`该目录下存放的是设备与设备接口的文件，设备文件是Linux中特有的文件类型，在 Linux系统下，以文件的方式访问各种设备，即通过读写某个设备文件操作某个具体硬件。

4. `/etc`该目录下存放着系统主要的配置文件，例如人员的账号密码文件、各种服务的其实文件等。一般 来说，此目录的各文件属性是可以让一般用户查阅的，但是只有root有权限修改。对于PC上的Linux系统，/etc目录下的文件和目录非常多，这些目 录文件是可选的，它们依赖于系统中所拥有的应用程序，依赖于这些程序是否需要配置文件。

5. `/lib`该目录下存放共享库和可加载（驱动程序），共享库用于启动系统。运行根文件系统中的可执行程序。

6. `/home`系统默认的用户文件夹，它是可选的，对于每个普通用户，在/home目录下都有一个以用户名命名的子目录，里面存放用户相关的配置文件。

7. `/root`系统管理员（root）的主文件夹，即是根用户的目录，与此对应，普通用户的目录是/home下的某个子目录。

8. `/usr`该目录的内容可以存在另一个分区中，在系统启动后再挂接到根文件系统中的/usr目录下里存放的是共享、只读的程序和数据，这表明 /usr目录下的内容可以在多个主机间共享。

9. `/var`与/usr目录相反，/var目录中存放可变的数据，比如spool目录（mail,news），log文件，临时文件。

10. `/proc`这是一个空目录，常作为proc文件系统的挂接点，proc文件系统是个虚拟的文件系统，它没有实际的存储设备，里面的目录，文件都是由内核临时生成的，用来表示系统的运行状态，也可以操作其中的文件控制系统。

11. `/mnt`用于临时挂载某个文件系统的挂接点，通常是空目录，用来临时挂载光盘、移动存储设备等。

12. ` /tmp`用于存放临时文件，通常是空目录，一些需要生成临时文件的程序用到的/tmp目录下，所以/tmp目录必须存在并可以访问。

13. `/srv`服务器的目录

### **关于权限**
1. 在Linux的图形界面中,经常可以访问某个目录但是并不能修改目录下的文件的内容,`nautilus`是文件管理器,用`sudo nautilus`以管理员身份打开文件管理器就可以了.


### **关于环境变量**

1. `/etc/profile`：此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行,并从/etc/profile.d目录的配置文件中搜集shell的设置。

2. `/etc/bashrc`: 为每一个运行bash shell的用户执行此文件,当bash shell被打开时,该文件被读取。

3. `~/.bash_profile`: 每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,设置一些环境变量,执行用户的.bashrc文件。

4. `~/.bashrc`: 该文件包含专用于自己的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取。

5. `/etc/profile`中设定的变量(全局)的可以作用于任何用户,而`~/.bashrc`等中设定的变量(局部)只能继承 /etc/profile中的变量,他们是"父子"关系。

6. `/etc/enviroment`是系统的环境变量,`/etc/profile`是所有用户的环境变量。先执行`/etc/environment`，后执行`/etc/profile`。`/etc/environment`是设置整个系统的环境，而`/etc/profile`是设置所有用户的环境，前者与登录用户无关，后者与登录用户有关。

7. 同一个变量在用户环境`/etc/profile`和系统环境`/etc/environment`有不同的值那应该是以用户环境为准

8. 修改environment之后，执行`source /etc/environment`立即生效！

### **解决软件依赖**

`sudo aptitude -f install [package]`

> aptitude 与 apt-get 一样，是 Debian 及其衍生系统中功能极其强大的包管理工具。与 apt-get 不同的是，aptitude 在处理依赖问题上更佳一些。举例来说，aptitude 在删除一个包时，会同时删除本身所依赖的包。这样，系统中不会残留无用的包，整个系统更为干净。

- 以下总结的一些常用 `aptitude` 命令，仅供参考。

```bash
aptitude update 更新可用的包列表
aptitude upgrade 升级可用的包
aptitude dist-upgrade 将系统升级到新的发行版
aptitude install pkgname 安装包
aptitude remove pkgname 删除包
aptitude purge pkgname 删除包及其配置文件
aptitude search string 搜索包
aptitude show pkgname 显示包的详细信息
aptitude clean 删除下载的包文件
aptitude autoclean 仅删除过期的包文件
```

- 统计命令执行时间

`time + [命令内容]` 命令可以用来统计命令执行时间，这部分时间包括总的运行时间，用户空间执行时间，内核空间执行时间，它通过 ptrace 系统调用实现。

### **输出重定向**

`command > file` : 表示输出重定向, 输出重定向会覆盖文件内容

`>>` : 表示不重写文件 , 追加输出

`&>` : `./build/tools/caffe train --solver=examples/mnist/lenet_lr_solver.prototxt &> log.txt` 
