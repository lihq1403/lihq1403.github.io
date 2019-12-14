---
cover: https://image.lihq.xyz/imgs/2019/12/b79fbd1e8b632c33.jpg
title: Vagrant +Virtualbox 打造跨平台开发环境
date: 2019-08-20 17:54
tags:
  - tools
categories:
  - 分享
keywords:
description:
---

# Vagrant +Virtualbox 打造跨平台开发环境
> 为啥说是跨平台呢，因为这几个工具软件，每个系统都有相应的软件安装。还有一个原因是，作为一名程序员，应该==多多使用Linux嘛==，哈哈哈！！！


## 安装所需工具
- [Vagrant](https://www.vagrantup.com/)
- [Virtualbox](https://www.virtualbox.org/)
- [Centos7 box](https://cloud.centos.org/centos/7/vagrant/x86_64/images/)


> 全都下载最新版本的就好啦，哈哈哈，先将工具都下载好了，后面的事就好办了

## 安装软件
> vagrant + virtualbox 的安装就不说了（so easy），记得vagrant安装完，如果点击yes，会重启电脑的（ps：亲测，貌似不重启也可以使用的）

### vagrant 常用命令
> 先了解一些基本常识 

```
vagrant init      # 初始化，生成Vagrantfile

vagrant up        # 启动虚拟机
vagrant halt      # 关闭虚拟机
vagrant reload    # 重启虚拟机
vagrant ssh       # SSH 至虚拟机
vagrant suspend   # 挂起虚拟机
vagrant resume    # 唤醒虚拟机
vagrant status    # 查看虚拟机运行状态
vagrant destroy   # 销毁当前虚拟机

#box管理命令
vagrant box list    # 查看本地box列表
vagrant box add     # 添加box到列表
vagrant box remove  # 从box列表移除 

# 修改了配置需要启动或重启
vagrant provision
vagrant reload --provision
```

## 添加box

```
$ vagrant box add [box名称] [下载的box文件路径]
```

示例：
```
$ vagrant box add centos7 CentOS-7-x86_64-Vagrant-1907_01.VirtualBox.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos7' (v0) for provider:
    box: Unpacking necessary files from: file://E:/work/box/CentOS-7-x86_64-Vagrant-1907_01.VirtualBox.box
    box:
==> box: Successfully added box 'centos7' (v0) for 'virtualbox'!
```

### box 初始化

> 新建一个目录，用来保存虚拟机配置文件，以后的操作都是在这里进行了，我这里是E:/work/box/

```
$ vagrant init
```

示例：
```
$ vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

### Vagrantfile 文件小修改

- [具体配置说明](https://www.baidu.com/s?wd=Vagrantfile%E6%96%87%E4%BB%B6%E8%AF%B4%E6%98%8E)

> 我这里就是改了几个地方

```
# box名称修改
config.vm.box = "centos7"

# 将网络变成私有网络，这样的话，就不需要去做端口转发了，具体ip值的话，建议先去网络共享中心看看网卡是啥，一般是这个：VirtualBox Host-Only Network
config.vm.network "private_network", ip: "192.168.56.10" 

# 共享目录，主要是放代码的目录，在centos里面会变成777的 (ps:nfs模式的共享磁盘，可以加快共享目录文件读取，需要先vagrant plugin install vagrant-winnfsd)
config.vm.synced_folder "./../www", "/home/wwwroot", type: "nfs"
```

## box 启动

> 在当前目录启动哦，因为要读取你刚刚修改的Vagrantfile；第一次的话，会比较慢，因为要创建虚拟机，后面启动就很快啦

示例：
```
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'centos7'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: box_default_1566287914381_38916
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: No guest additions were detected on the base box for this VM! Guest
    default: additions are required for forwarded ports, shared folders, host only
    default: networking, and more. If SSH fails on this machine, please install
    default: the guest additions and repackage the box to continue.
    default:
    default: This is not an error message; everything may continue to work properly,
    default: in which case you may ignore this message.
==> default: Configuring and enabling network interfaces...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Rsyncing folder: /cygdrive/e/work/box/ => /vagrant
==> default: Mounting shared folders...
    default: /home/wwwroot => E:/work/www
Vagrant was unable to mount VirtualBox shared folders. This is usually
because the filesystem "vboxsf" is not available. This filesystem is
made available via the VirtualBox Guest Additions and kernel module.
Please verify that these guest additions are properly installed in the
guest. This is not a bug in Vagrant and is usually caused by a faulty
Vagrant box. For context, the command attempted was:

mount -t vboxsf -o uid=1000,gid=1000 home_wwwroot /home/wwwroot

The error output from the command was:

mount: unknown filesystem type 'vboxsf'

```

> 好像看到不得了的东西，出问题了！！！

1. 不要慌，问题不大
2. 打开浏览器
3. 飞快的键入问题"mount: unknown filesystem type 'vboxsf"
4. 回车，会发现原来有很多人跟咱们一样，哈哈哈，心理平衡了


### 解决 mount: unknown filesystem type 'vboxsf'

> 其实就是缺少了挂载插件，装上就好了

###### 1、我们先关闭虚拟机
```
$ vagrant halt
==> default: Attempting graceful shutdown of VM...
```

###### 2、安装插件
```
$ vagrant plugin install vagrant-vbguest
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Installed the plugin 'vagrant-vbguest (0.19.0)'!
```

###### 3、再次启动
```
$ vagrant up
安装过程略多，此处省略...

最后没有报错，就启动成功了
```

## 进入box

> vagrant的box安装好了之后，默认的账号有俩个【vagrant vagrant (普通权限)】【root vagrant (root权限)】

> 启动的时候，有没有发现一行 SSH address: 127.0.0.1:2222 没错，他会自己开启一个本地连接端口的

#### 尝试ssh账号密码
```
$ ssh -p 2222 root@127.0.0.1
The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
ECDSA key fingerprint is SHA256:sLcP3X75TMzTjmeolLXX7BqpD+Psrq6vU7haTYyM//4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[127.0.0.1]:2222' (ECDSA) to the list of known hosts.
root@127.0.0.1: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

> 咦，进不去。不要慌，其实只是因为是新的box，并没有开启ssh密码访问模式，我们换个方式进去

#### 尝试vagrant ssh
```
$ vagrant ssh
[vagrant@10 ~]$
```

> 666，进去了，接下里就是开启一下密码访问模式（其实不开也没问题，用ssh密钥去连接就好了，我是出于习惯，所以想开启密码访问）

##### 开启密码模式

###### 1、打开ssh配置
```
# vi /etc/ssh/sshd_config
```

###### 2、修改内容
```
PasswordAuthentication no 改为 PasswordAuthentication yes
```

###### 3、保存退出
```
:wq
```
###### 4、重启ssh
```
# systemctl restart sshd.service
```

#### 再次尝试ssh账号密码
```
$ ssh -p 2222 root@127.0.0.1
root@127.0.0.1's password:
Last login: Tue Aug 20 08:30:58 2019 from 10.0.2.2
[root@10 ~]#
```

> 完美！！！

## 进行box更新

- [第一步当然是修改yum源啦](https://www.baidu.com/s?wd=centos7%E4%BF%AE%E6%94%B9yum%E6%BA%90)

更新系统：
```
# yum -y update
```

## PHP开发环境搭建

> 我使用过三套方案，各有各的优势，所以我选择宝塔 -.-

1. nginx/apache php mysql 一个一个编译安装
2. [lnmp一键包](https://lnmp.org/)
3. [宝塔服务器运维面板](https://www.bt.cn/)

#### 宝塔安装

> 因为我们装的是centos7，所以十分契合宝塔面板

开始漫长的安装之旅
```
# yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
略...

Starting Bt-Panel... Bt-Panel (pid 3757 3776) already running
Starting Bt-Tasks... Bt-Tasks (pid 3790) already running
==================================================================
Congratulations! Installed successfully!
==================================================================
Bt-Panel: http://116.18.22.120:8888/0dafbc37
username: c0ryrxbk
password: fde1ff6a
Warning:
If you cannot access the panel,
release the following port (8888|888|80|443|20|21) in the security group
==================================================================
Time consumed: 2 Minute!
```
> 原来只花了俩分钟，666。接下来，在浏览器上访问这个地址就好了，因为是虚拟机的原因，那个ip地址是不对的，我这里是http://192.168.56.10:8888/0dafbc37。输入账号密码，熟悉的画面

![image](http://mine.lihq.xyz/image/201908/20/vagrant_20190820170611.pngvagrant_20190820170611.png)

> 才发现，原来默认的内存这么小，不行，要调大一点 （因为会不能安装mysql的）

###### 修改box内存

打开Vagrantfile 将这部分开启，内存值的单位是MB，我这里相当于2GB
```
  config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
      vb.memory = "2048"
  end
```

###### 重启
```
$ vagrant reload --provision
```

#### 必要软件安装
- nginx 1.16
- mysql 5.7
- php 7.2

> 勾选，然后一键安装，贼简单

#### 扩展软件
- PM2管理器
- Redis
- PHP守护
- ...

> 都在软件商店里面，根据自己的需求安装就好啦，安装好软件之后。1、根据自己需要装php扩展 2、根据自己需要，开启一些禁用函数

#### php扩展
- fileinfo
- redis
- Swoole
- bz2
- ...

#### php禁用函数
- putenv
- symlink
- proc_ope n
- ...

#### 漫长的安装之旅，都是后台运行的，切出去看个剧吧，哈哈哈

本次大致耗时情况，仅供参考，肯定是性能越好，安装的越快啦。我这里才1核2G

软件 | 耗时
---|---
nginx-1.16 | 159秒
mysql-5.7 | 81秒
php-7.2 | 10秒
redis-5.0|77秒
pm2-2.6|28秒
php扩展-fileinfo-72|24秒
php扩展-redis-72|13秒
php扩展-swoole4-72|86秒
php扩展-bz2-72|3秒

#### composer配置

> 作为一名phper，composer是必须的。安装php的时候，会自己帮我们装上composer的。感谢阿里大大提供镜像源

- [阿里云 Composer 全量镜像](https://developer.aliyun.com/composer)

更新自己先：
```
# composer self-update
Updating to version 1.9.0 (stable channel).
   Downloading (100%)
Use composer self-update --rollback to return to version 1.8.5
```
全局更换源：
```
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

#### 测试开发环境
1. 网站
2. 添加站点
3. 域名，根目录，php版本
4. 宿主机host文件修改
5. 访问域名
6. bingo，成功
7. 宝塔的功能还需要自己去实验实验就好了

## box打包

> 这就是方便之处，只要你配置好了一次，再打包成自己的box镜像，那么下次开箱即用

在开发环境的目录，关闭centos7
```
$ vagrant halt 
==> default: Attempting graceful shutdown of VM...
```
确认centos7关机
```
$ vagrant status
Current machine states:

default                   poweroff (virtualbox)

The VM is powered off. To restart the VM, simply run `vagrant up`
```
打包
```
$ vagrant package --output centos7-20190820.box
==> default: Clearing any previously set forwarded ports...
==> default: Exporting VM...
==> default: Compressing package to: E:/work/box/centos7-20190820.box
```
分发发去给其他人用的时候只需要
```
1、vagrant box add [box名称] [下载的box文件路径]
2、vagrant init
3、修改Vagrantfile配置
4、vagrant up 开箱即用
```