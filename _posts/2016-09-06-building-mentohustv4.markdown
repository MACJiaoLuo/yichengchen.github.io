---
layout:     post
title:      "小米路由mini交叉编译mentohust v4"
subtitle:   "锐捷路由器实现"
date:       2016-09-06
author:     "yicheng"
header-img: "img/post-bg-hello.jpg"
tags:
    - 网络
---

> 本文主要是本人折腾路由器交叉编译所踩的坑及进行的一些记录备忘。已在福州大学旗山校区进行验证可用。
>
> 有人对mentohust 在github上提供了后续支持，提供了V4版本认证。[Github](https://github.com/hyrathb/mentohust)

# 配置交叉编译环境



- 使用Debian/Ubuntu系统

```
sudo apt-get install autoconf automake bison build-essential flex gawk gettext gperf libtool pkg-config zlib1g-dev libgmp3-dev libmpc-dev libmpfr-dev texinfo python-docutils mc

```

# 下载toolchain

- 小米原生固件 [小米开放平台](http://bigota.miwifi.com/xiaoqiang/sdk/tools/package/sdk_package_r1c.zip)
- padvan固件toolchain需要自己编译，查看教程[HowToMakeFirmware](https://bitbucket.org/padavan/rt-n56u/wiki/EN/HowToMakeFirmware)至第8步。编译需要一定时间，在本人i7 6核心虚拟机上大概需要10分钟.
- openwrt固件 由于官网提供的小米路由mini[固件](https://downloads.openwrt.org/chaos_calmer/15.05.1/ramips/mt7620/openwrt-15.05.1-ramips-mt7620-xiaomi-miwifi-mini-squashfs-sysupgrade.bin)版本是chaos calmer,没有预编译toolchain，可适用trunk版本代替。[下载](https://downloads.openwrt.org/snapshots/trunk/ramips/mt7620/OpenWrt-Toolchain-ramips-mt7620_gcc-5.3.0_musl-1.1.15.Linux-x86_64.tar.bz2)


# 设置环境变量

```
//路径请根据实际情况调整。
//文件名可参照toolchain下bin目录进行调整

export PATH=$PATH:/home/toolchain-mipsel/bin/
export CC=mipsel-openwrt-linux-gcc  
export CPP=mipsel-openwrt-linux-cpp  
export GCC=mipsel-openwrt-linux-gcc  
export CXX=mipsel-openwrt-linux-g++  
export RANLIB=mipsel-openwrt-linux-uclibc-ranlib  
export ac_cv_linux_vers=2.6.24  
export LDFLAGS="-static"  
export CFLAGS="-Os -s"  

//使用openwrt固件需要额外定义此条。
export STAGING_DIR=/home/toolchain-mipsel/
```

# 编译libpcap

```
 ./configure --host=mipsel-linux  --with-pcap=linux
 make
```
注意，make过程中会报错，无需担心，此时我们需要的libpcap.a已经生产

``` Shell
ls | grep libpcap.a
cp libpcap.a /home/libpcap.a   //复制出来
```

# 编译mentohust

```
cd /home/mentohust
 ./configure --host=mipsel-linux  --disable-encodepass --disable-notify --prefix=/etc --with-pcap=/home/libpcap.a
make
```
成功后编译完成的mentohust会在src目录下。

# 上传至路由器

- 使用winscp用scp协议链接路由器并上传。或者直接使用scp命令

```
scp mentohust root@192.168.1.1:/tmp/mentohust
```

- ssh至路由器，执行以下命令

``` Shell
chmod +x mentohust
./mentohust -h //查看帮助
./mentohust -u 用户名 -p 密码 -n eth0.2 -d 1 //测试运行

// 手动配置/etc/mentohust.conf 修改详细配置（由于未知原因mentohust的默认设置向导显示在openwrt下有些残缺，故使用手动配置）

cp mentohust /bin/mentohust //openwrt下，测试成功后拷贝到bin目录。

mentohust -b1 //后台运行
```


# 启动脚本
- 此部分感谢 [mentohust v4版本编译及ipv6的配置教程](http://soundrain.net/2016/04/25/mentohust-v4%E7%89%88%E6%9C%AC%E7%BC%96%E8%AF%91%E5%8F%8Aipv6%E7%9A%84%E9%85%8D%E7%BD%AE/)


创建文件
```
vi /etc/init.d/mentohust
```


```
#!/bin/sh /etc/rc.common
# /init.d/mentohust
START = 50
start()
{
        /bin/mentohust -b1
}

stop()
{
        /bin/mentohust -k &
}
```

赋予权限

```
chmod 755 mentohust
ln -s /etc/init.d/launchMento /etc/rc.d/S97mentohust

```

重启路由器，完成。


