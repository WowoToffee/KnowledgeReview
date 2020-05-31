# vargrant入门

## 使用Vagrant创建CentOS虚拟机

Vagrant是一款由HashiCorp公司提供的，用于快速构建虚拟机环境的软件。本节我们将使用Vagrant结合Oracle VM VirtualBox快速地在win10环境下构建CentOS7虚拟机。在此之前需要先安装好 [Vagrant](https://www.vagrantup.com/downloads.html) 和 [VirtualBox](https://www.virtualbox.org/)。

1. 在win10任意盘符下创建vagrant_vm目录（注意目录最好不要有中文和空格），然后在该目录下使用cmd执行`vagrant init centos/7`命令 
2. 执行`vagrant up`启动。（这时候最好也打开VirtualBox）

## 连接虚拟机

`vagrant status`命令查看一下虚拟机的状态

`vagrant halt`来关闭虚拟机

`vagrant suspend`命令来暂停运行中的虚拟机

`vagrant resume`命令来恢复虚拟机

*vagrant ssh* 连接虚拟机

注意：

xshall 登录可能是有问题,默认是无法使用账号密码登录`root`账号，只能使用`vagrant`账号，密码`vagrant`。

设置root的密码：修改 /etc/ssh/sshd_config 文件，修改 ssd_config 里 PermitRootLogin属性 改为yes （登录root 用户， 密码vagrant）

本地使用秘钥方式登录：秘钥地址：`.vagrant\machines\default\virtualbox\private_key`, ip:127.0.0.1  端口：2222

参考：

[使用Vagrant创建CentOS虚拟机]https://mrbird.cc/Create-Virtual-Machine-By-Vagrant.html

[vagrantfile详解](https://blog.csdn.net/raoxiaoya/article/details/93638960)

## 常用命令：

### machine管理：

```shell
$ vagrant init      # 初始化，生成Vagrantfile，可指定box
$ vagrant up        # 启动虚拟机，可指定machine
$ vagrant halt      # 关闭虚拟机，可指定machine
$ vagrant reload    # 重启虚拟机，并重新加载配置参数，可指定machine
$ vagrant ssh       # SSH 至虚拟机，可指定machine
$ vagrant suspend   # 挂起虚拟机，可指定machine
$ vagrant resume    # 唤醒虚拟机，可指定machine
$ vagrant status    # 查看虚拟机运行状态，可指定machine
$ vagrant destroy   # 销毁当前虚拟机，可指定machine
$ vagrant suspend   # 挂起当前的虚拟机
$ vagrant resume    # 恢复前面被挂起的状态
```

### box管理

```shell
$ vagrant box list    # 查看本地box列表
$ vagrant box add     # 添加box到列表
$ vagrant box remove  # 从box列表移除 
```

### machine与box转换

```shell
$ vagrant package        # 对指定machine打包成box
$ vagrant box repackage  # 对指定box重新打包成box，该box的machine会被halt
```

