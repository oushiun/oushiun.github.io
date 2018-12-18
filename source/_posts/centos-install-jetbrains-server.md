---
title: CentOS 搭建 jetbrains 破解服务器

toc: true
tags:
 - CentOS
 - jetbrains
categories:
 - 软件
date: 2018-05-08 17:18:37
thumbnail: https://static.oushiun.com/blog/banner/jetbrains.png
---

## CentOS 搭建 jetbrains 破解服务器

<!-- more -->

### 安装步骤

1.  检查是否安装了 vsftpd

``` bash
rpm -qa |grep vsftpd
```

3.  Linux 系统：CentOS 7 x86_64

``` bash
cat /etc/os-release
```

2.  通过 yum 来安装 vsftpd

``` bash
yum -y install vsftpd
```

3.  设置为开机启动

``` bash
systemctl enable vsftpd

systemctl start vsftpd #启动vsftpd命令
systemctl stop vsftpd #停止vsftpd命令
systemctl status vsftpd #查看vsftpd状态
systemctl restart vsftpd #重启vsftpd命令
```

4.  修改配置文件

``` bash
vi /etc/vsftpd/vsftpd.conf
```

### 添加用户及额外配置

1.  启用 root 用户进入 `/etc/vsftpd` 目录下修改 `ftpusers` & `user_list`
    将 **root** 用户注释 #

2.  开放 21 端口

``` bash
firewall-cmd --zone=public --add-port=21/tcp --permanent
```

``` bash
yum install firewalld #安装firewalld 防火墙

systemctl start firewalld #开启防火墙
systemctl stop firewalld #关闭防火墙
systemctl enable firewalld #开机自动启动
systemctl disable firewalld #禁止开机制动启动

firewall-cmd --state #running 表示运行
firewall-cmd --reload #重新载入以生效
firewall-cmd --complete-reload #更新规则，重启服务
```

### 上传文件

下载[IntelliJ IDEA License Server v1.6](http://blog.lanyus.com/archives/326.html)(当前使用是 v1.6，新版本还请及时关注[lanyus](http://blog.lanyus.com/))

解压到 某个目录下(任意即可), IntelliJIDEALicenseServer 目录下涵盖了很多平台(mac linux windows)。当前服务器是 x86_64 GNU/Linux，so 给 IntelliJIDEALicenseServer_linux_amd64 赋可执行权限

``` bash
chmod +x IntelliJIDEALicenseServer_linux_amd64
```

### 安装配置 Nginx

1.  通过 yum 安装 Nginx

``` bash
yum install epel-release
yum install nginx
```

常用命令

``` bash
systemctl start nginx
systemctl enable nginx
systemctl status nginx
```

2.  配置防火墙

``` bash
firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --zone=public --permanent --add-service=https
firewall-cmd --reload
```

3.  测试 Nginx 是否正常访问
    [http://SERVER_DOMAIN_NAME_OR_IP ](http://SERVER_DOMAIN_NAME_OR_IP)

4.  修改 Nginx 配置文件

``` bash
#vi /usr/local/nginx/conf/nginx.conf
server {
    listen 80;
    server_name idea.oushiun.com;
    root /usr/local/nginx/html;

    location / {
        proxy_pass http://127.0.0.1:1027;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    access_log /dev/null;
    error_log /dev/null;
}
```

注意事项：需要展示破解地址文档，通过 nginx 反向代理需要将 IntelliJIDEALicenseServer.html 放在/目录，如果是通过直接运行脚本需要和脚本在同一目录。

5.  systemd 设置

``` bash
# vim /etc/systemd/system/intellij.service
[Unit]
Description= IntelliJIDEALicenseServe Service
After=network.target

[Service]
ExecStart=/root/jetbrains/IntelliJIDEALicenseServer_linux_amd64
PrivateTmp=true

[Install]
WantedBy=default.target

# systemctl daemon-reload # 重载
# systemctl start intellij # 启动
# systemctl enable intellij # 开机启动
# systemctl disable intellij # 撤销开机启动
```

6.  IntelliJIDEALicenseServer 帮助

``` bash
./IntelliJIDEALicenseServer_linux_amd64 -h
-l string 绑定的host，基本默认
       bind on host (default "0.0.0.0")
 -p int 监听端口，建议改下
       port (default 1027)
 -prolongationPeriod string 过期时间
       prolongationPeriod (default "607875500")
 -u string 当未设置-u参数，且计算机用户名为^[a-zA-Z0-9]+$时，使用计算机用户名作为idea用户名
       username (default "ilanyu")
```
