---
title: ContOS 7 安装 BBR

tags:
 - BBR
 - CentOS
categories:
 - 工具
date: 2018-05-12 19:02:51
banner: https://static.oushiun.com/blog/banner/google-bbr.jpg
---

Google 开源了其 TCP BBR 拥塞控制算法，并提交到了 Linux 内核，从 4.9 开始，Linux 内核已经用上了该算法。根据以往的传统，Google 总是先在自家的生产环境上线运用后，才会将代码开源，此次也不例外。根据实地测试，在部署了最新版内核并开启了 TCP BBR 的机器上，网速甚至可以提升好几个数量级。

<!-- more -->

#### CentOS 7.3

1.  yum 系统更新（更新到 CentOS 7.3）

```bash
yum update
```

2.  查看系统版本

```bash
cat /etc/redhat-release
```

输出如下（release 数值大于 7.3 即可），则表示已升级到 7.3

```bash
CentOS Linux release 7.3.1611 (Core)
```

3.  安装 elrepo 并升级内核

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml -y
```

正常情况下将输出如下信息：

```bash
Transaction Summary
================================================================================
Install  1 Package
Total download size: 39 M
Installed size: 169 M
Downloading packages:
kernel-ml-4.9.0-1.el7.elrepo.x86_64.rpm                    |  39 MB   00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : kernel-ml-4.9.0-1.el7.elrepo.x86_64                          1/1
  Verifying  : kernel-ml-4.9.0-1.el7.elrepo.x86_64                          1/1
Installed:
  kernel-ml.x86_64 0:4.9.0-1.el7.elrepo
Complete!
```

4.  更新 grub 文件并重启（reboot 后，ssh 会断开，稍等一会儿重新连接）

```bash
egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
grub2-set-default 0
reboot
```

5.  开机后查看内核是否已更换为 4.9

```bash
uname -r
```

输出如下内容则表示内核 4.9 已经启动了（数值大于 4.9 即可）

```bash
4.9.0-1.el7.elrepo.x86_64
```

#### CentOS 7.4

导入 elrepo 软件源的 GPG 公钥

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

导入 elrepo 软件源

说明：该地址可以自动下载该源的最新的软件列表，无需修改地址。

```bash
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```

下载并安装新的内核

启用软件源并下载安装最新稳定版内核

```bash
yum -y --enablerepo=elrepo-kernel install kernel-ml
```

设定 Grub 默认启动新内核

说明：如果手动修改过 Grub 的配置文件，以下命令可能无法执行成功。请自行修改 Grub 配置文件。

```bash
grub2-set-default 0
```

使用新的内核重启

```bash
reboot
```

检查设置 & 删除旧的内核

查看当前系统的内核版本号

```bash
uname -r
```

如果输出是 4.10 以上的版本，说明安装成功。

```bash
4.14.13-1.el7.elrepo.x86_64
```

删除旧内核

说明：删除旧内核的目的是为了防止 yum 更新旧版内核之后覆盖了 grub 默认启动项

```bash
yum -y remove kernel kernel-tools
```

#### 开启 BBR

```bash
vim /etc/sysctl.conf
```

添加如下内容

```bash
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

加载系统参数（正常情况下会输出我们之前加入的内容）

```bash
sysctl -p
```

验证 bbr 是否已经开启

a.若

```bash
sysctl net.ipv4.tcp_available_congestion_control
```

返回

```bash
net.ipv4.tcp_available_congestion_control = bbr cubic reno
```

则成功

b.若

```bash
lsmod | grep bbr
```

返回形如如下形式的信息即成功。

```bash
tcp_bbr                16384  1
tcp_bbr                20480  0
```
