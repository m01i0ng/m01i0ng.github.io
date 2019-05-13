---
title: Frp 内网穿透使用说明
date: 2019-01-20 20:08:58
tags:
    - frp
categories:
---

## 使用 frp 内网穿透

> 背景：内网服务器为 ESXI 环境，希望从外网通过域名直接访问对应服务器的服务。一开始用花生壳配合路由器做端口转发，但端口数量以及服务类型据局限较大，故寻求新的内网穿透方式。

- frp：[Github](https://github.com/fatedier/frp)
- 系统：CentOS 7 x64
- 需求：一台固定公网 IP 的云服务器，域名
- 内网服务器 IP 段：192.168.1.201-254

## 下载

```bash
cd /root
wget -c https://github.com/fatedier/frp/releases/download/v0.23.1/frp_0.23.1_linux_amd64.tar.gz
tar zxvf frp*.tar.gz
```

## frps 服务端

**安装在云服务器上**

```bash
cd /root/frp
chmod +x frps
yum install -y supervisor
systemctl disable firewalld && systemctl stop firewalld

# 定义 frps 配置
cat > /root/frp/frps.ini <<EOF
[common]
bind_port = 30000
vhost_http_port = 30100
vhost_https_port = 30143
subdomain_host = dev.demo.cc
dashboard_port = 30200
dashboard_user = admin
dashboard_pwd = admin
EOF

# 定义运行守护配置
cat > /etc/supervisord.d/frps.ini <<EOF
[program:frps]
directory=/root/frp
command=/root/frp/frps -c /root/frp/frps.ini
autostart=true
autorestart=false
stderr_logfile=/var/log/supervisor/frps_stderr.log
stdout_logfile=/var/log/supervisor/frps_stdout.log
EOF

# 开启服务
systemctl enable supervisord && systemctl start supervisord
```

如对已配置的 `frps.ini` 进行修改，`vim /root/frp/frps.ini` 编辑保存之后，执行 `supervisorctl reload frps`

访问 `IP:30200` 如能进入 frp 运行面板则成功，否则请从头检查配置

## frpc 客户端

**客户端程序运行在想要被穿透的内网服务器上，用来与服务端沟通，一个客户端只可以与一个服务端连接**

服务器 IP：192.168.1.206
服务：

- web，端口 80
- ssh，端口 22

```bash
cd /root/frp
chmod +x frpc
yum install -y supervisor
systemctl disable firewalld && systemctl stop firewalld

cat < /root/frp/frpc.ini <<EOF
[common]
server_addr = 云服务器公网 IP
server_port = 30000

[206_ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 30106
use_encryption = true
use_compression = true

[206_web]
type = http
local_port = 80
subdomain = 206-web
use_encryption = true
use_compression = true
EOF

# 定义运行守护配置
cat > /etc/supervisord.d/frpc.ini <<EOF
[program:frpc]
directory=/root/frp
command=/root/frp/frpc -c /root/frp/frpc.ini
autostart=true
autorestart=false
stderr_logfile=/var/log/supervisor/frpc_stderr.log
stdout_logfile=/var/log/supervisor/frpc_stdout.log
EOF

# 开启服务
systemctl enable supervisord && systemctl start supervisord
```

更多服务转发配置请看 [官方说明](https://github.com/fatedier/frp/blob/master/README_zh.md#%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B)

如对已配置的 `frpc.ini` 进行修改，`vim /root/frp/frpc.ini` 编辑保存之后，执行 `supervisorctl reload frpc`

访问 `206-web.dev.demo.cc:30100` 如能成功访问则配置成功
ssh 连接命令：`ssh -oPort=30106 root@云服务器公网 IP`
