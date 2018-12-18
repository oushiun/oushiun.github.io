---
title: CentOS 搭建 shadowsocks

toc: true
tags:
 - CentOS
 - shadowsocks
categories:
 - 工具
date: 2018-05-09 22:20:38
thumbnail: https://static.oushiun.com/blog/banner/shadowsocks.png
---

Shadowsocks(ss) 是由 Clowwindy 开发的一款软件，其作用本来是加密传输资料。当然，也正因为它加密传输资料的特性，使得 GFW 没法将由它传输的资料和其他普通资料区分开来，也就不能干扰我们访问那些「不存在」的网站了。

<!-- more -->

### 安装 shadowsocks

#### 安装 pip

Pip 是 Python 的包管理工具，这里我们用 pip 安装 shadowsocks。

``` bash
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
pip -V #pip 10.0.1 from /usr/lib/python2.7/site-packages/pip (python 2.7)
```

#### 通过 pip 安装 shadowsocks

``` bash
pip install --upgrade pip
pip install shadowsocks
```

#### 配置 shadowsocks

``` bash
# vi /etc/shadowsocks.json
{
  "server":"x.x.x.x",             #你的 ss 服务器 ip
  "server_port":0,                #你的 ss 服务器端口
  "local_address": "127.0.0.1",   #本地ip
  "local_port":0,                 #本地端口
  "password":"password",          #连接 ss 密码
  "timeout":300,                  #等待超时
  "method":"aes-256-cfb",         #加密方式
  "workers": 1                    #工作线程数
}
```

#### systemctl 设置

``` bash
# vi /etc/systemd/system/shadowsocks.service
[Unit]
Description=Shadowsocks

[Service]
Type=forking
PIDFile=/run/shadowsocks/server.pid
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /run/shadowsocks
ExecStart=/usr/bin/ssserver --pid-file /var/run/shadowsocks/server.pid --log-file /var/log/shadowsocks.log -c /etc/shadowsocks.json -d start
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

#### 配置 firwall 规则

输入以下命令开启一个端口，如果不是 9002 端口，修改成自己需要添加的端口（–permanent 永久生效，没有此参数重启后失效）。

``` bash
firewall-cmd --zone=public --add-port=9002/tcp --permanent
```

添加端口后系统返回 success 说明添加成功。这个时候需要重新载入 firewall，输入以下命令，返回 success 后此端口就可使用

``` bash
firewall-cmd --reload
```

#### shadowsocks 其他配置

``` bash
ssserver -h
usage: ssserver [OPTION]...
A fast tunnel proxy that helps you bypass firewalls.

You can supply configurations via either config file or command line arguments.

Proxy options:
  -c CONFIG              path to config file
  -s SERVER_ADDR         server address, default: 0.0.0.0
  -p SERVER_PORT         server port, default: 8388
  -k PASSWORD            password
  -m METHOD              encryption method, default: aes-256-cfb
  -t TIMEOUT             timeout in seconds, default: 300
  --fast-open            use TCP_FASTOPEN, requires Linux 3.7+
  --workers WORKERS      number of workers, available on Unix/Linux
  --forbidden-ip IPLIST  comma seperated IP list forbidden to connect
  --manager-address ADDR optional server manager UDP address, see wiki

General options:
  -h, --help             show this help message and exit
  -d start/stop/restart  daemon mode
  --pid-file PID_FILE    pid file for daemon mode
  --log-file LOG_FILE    log file for daemon mode
  --user USER            username to run as
  -v, -vv                verbose mode
  -q, -qq                quiet mode, only show warnings/errors
  --version              show version information

Online help: <https://github.com/shadowsocks/shadowsocks>
```
