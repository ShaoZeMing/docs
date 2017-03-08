---
layout: blog
title: Ubuntu环境下SSH的安装和使用
category: blogs
---

SSH分为客户端和服务端。
服务端是一个守护进程，一般是sshd进程，在后台运行并响应来自客户端的请求。提供了对远程请求的处理，一般包括公共密钥认证、密钥交换、对称密钥加密和非安全连接。
客户端一般是ssh进程，另外还包含scp、slogin、sftp等其他进程。

## 工作机制：
1. 客户端发送一个连接请求到远程服务端
2. 服务端检查申请的包和IP地址，再发生密钥给SSH客户端；
3. 客户端再将密钥发回服务端，自此建立连接。

---
# 客户端

## 安装客户端（客户端不是必须的）

```
# apt-get install ssh
```
如果安装失败，则使用下面命令进行安装
```
# apt-get install openssh-client
```
## SSH登录（客户端）
```
$ ssh 192.168.159.128
$ ssh -l weiyg 192.168.159.128
$ ssh weiyg@192.168.159.128
```
---

# 服务端
## 安装服务器

```
# apt-get install openssh-server
```
## 启动服务器

```
# /etc/init.d/ssh stop                　　#停止
# /etc/init.d/ssh start                　　#启动
# /etc/init.d/ssh restart            　 #重启
```
##  SSH配置

修改配置文件`/etc/ssh/sshd_config`，并重启服务

```
# /etc/init.d/ssh restart
```
ssh默认端口是22，需要的话，自行修改

```
Port 20
```
ssh默认配置是允许root登录的，可以修改配置表禁止其登录(注意：ubuntu 16.10 ssh默认是禁止root登陆的。)

```
PermitRootLogin no     //不准root登陆
PermitRootLogin yes    //允许root登陆
```