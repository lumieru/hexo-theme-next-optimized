---
title: VirtualBox安装Ubuntu教程
date: 2018-01-01 23:00:31
tags:
- VBox
categories:
- Linux
---

最近因某些原因重装了Win10, 虚拟机也需要重装, 记录一下过程, 供以后查阅, 以免走更多弯路

# 需准备的工具和材料

- 虚拟机软件
- Ubuntu : 我用的是16.04版本的ubuntu的server版本(不是desktop桌面版) 
- 建议给虚拟机分配20G硬盘空间
- 建议给虚拟机分配4G内存


**. . .**<!-- more -->



# 网络设置

**在安装ubuntu-server之前需知** : 

你的那个虚拟机里的网络设置里只留两张网卡, 注意这两张网卡的顺序最好别颠倒, 免得麻烦

**网卡1**是 : “网络地址转换（NAT）”， 不是那个“NAT网络”噢， 这张网卡是用来访问宿主机和外网的
**网卡2**是 : “仅主机（Host-Olny）网络”， 这张网卡是用来让宿主机是访问你的这个虚拟机的， 这样当虚拟机装了openssh-server (`sudo apt-get install openssh-server`) 之后就能用ssh工具从宿主机连到你的这个虚拟机了


安装ubuntu的时候有一步是需要你选择主网卡, 此时得记住两张网卡的名字, 比如我的是 `enp0s3`对应网卡1 和 `enp0s8`对应网卡2, 那主网卡应该选“网络地址转换（NAT）”的那张, 即网卡1, 因为要上网.

**安装完毕之后** : 

使用ifconfig命令查看会发现只有一个网卡工作，因为第二块网卡还没有进行配置。

使用vim编辑`/etc/network/interfaces`，添加第二块网卡的网络配置，宿主机需要长期连接虚拟机，需要为Host-Only网络配置静态IP，IP需要和宿主机的Host-Only网段一致, 到windows的控制面板的网络适配器页面查看`VirtualBox Host-Only`这个网络适配器的网段, 比如我的是`192.168.80.1` , 
则 : 

    auto enp0s8
    iface enp0s8 inet static
    address 192.168.80.8

重启网络(`service networking restart`)或者系统。



# 更换源


见[原网站](https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/)

Ubuntu 的软件源配置文件是 /etc/apt/sources.list。将系统自带的该文件做个备份，将该文件替换为下面内容，即可使用 TUNA 的软件源镜像。替换之后记得 ` sudo apt-get update `

    # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
    deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse

    # 预发布软件源，不建议启用
    # deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse
    # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-proposed main restricted universe multiverse


# bash增强

可参考 {% post_link bash_enhance %}


# 必装软件

- sudo apt-get install openssh-server
- sudo apt-get install g++
- sudo apt-get install cmake
- sudo apt-get install gdb

装完gdb之后添加一个pstack脚本方便查看运行时的程序堆栈(用法 : pstack pid) : 

`sudo vi /usr/bin/pstack` 
`sudo chmod +x /usr/bin/pstack`

pstack脚本的内容如下 :

```
#!/bin/sh

if test $# -ne 1; then
    echo "Usage: `basename $0 .sh` <process-id>" 1>&2
    exit 1
fi

if test ! -r /proc/$1; then
    echo "Process $1 not found." 1>&2
    exit 1
fi

# GDB doesn't allow "thread apply all bt" when the process isn't
# threaded; need to peek at the process to determine if that or the
# simpler "bt" should be used.

backtrace="bt"
if test -d /proc/$1/task ; then
    # Newer kernel; has a task/ directory.
    if test `/bin/ls /proc/$1/task | /usr/bin/wc -l` -gt 1 2>/dev/null ;                                                                                                      then
        backtrace="thread apply all bt"
    fi
elif test -f /proc/$1/maps ; then
    # Older kernel; go by it loading libpthread.
    if /bin/grep -e libpthread /proc/$1/maps > /dev/null 2>&1 ; then
        backtrace="thread apply all bt"
    fi
fi

GDB=${GDB:-/usr/bin/gdb}

if $GDB -nx --quiet --batch --readnever > /dev/null 2>&1; then
    readnever=--readnever
else
    readnever=
fi

# Run GDB, strip out unwanted noise.
$GDB --quiet $readnever -nx /proc/$1/exe $1 <<EOF 2>&1 |
set width 0
set height 0
set pagination no
$backtrace
EOF
/bin/sed -n \
    -e 's/^\((gdb) \)*//' \
    -e '/^#/p' \
    -e '/^Thread/p'
```



若是ubuntu桌面版的话可以安装一下增强功能


<h1 id="桌面版ubuntu安装增强功能">桌面版ubuntu安装增强功能</h1>

<p><strong>注 : 如果在侧边找到如下图加载的虚拟光驱，就需要先右击，点击弹出，然后才可正常安装增强功能</strong></p>
<p><img src="/img/vbox_install_ubuntu_tutorial/20150116222056924.jpg" alt="" /><br /></p>
<p>点击安装增强功能</p>
<p><img src="/img/vbox_install_ubuntu_tutorial/20150116222128125.jpg" alt="" /><br /></p>
<p>点击“运行”</p>
<p><img src="/img/vbox_install_ubuntu_tutorial/20150116222246498.jpg" alt="" /><br /></p>
<p>输入登录系统的密码，点击授权，就开始自动安装了</p>
<p><img src="/img/vbox_install_ubuntu_tutorial/20150116222308759.jpg" alt="" /><br /></p>
<p>如图，为安装界面，安装完成后按下回车键，就按照成功了。</p>
<p><img src="/img/vbox_install_ubuntu_tutorial/20150116222336153.jpg" alt="" /><br /></p>
<p>安装好后关闭ubuntu再次启动ubuntu的时候，虚拟机就可以在无缝模式和自动显示尺寸下运行了。</p>

