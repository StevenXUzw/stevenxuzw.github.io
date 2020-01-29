---
layout: post
title: python之自动化运维(Paramiko)
---

## 简介
使用开源的Paramiko，我们就可以用Python代码中通过SSH协议对远程服务器执行操作，不需要手敲ssh命令，从而实现自动化运维。
ssh是一个协议，OpenSSH是其中一个开源实现，paramiko库，实现了SSHv2协议(底层使用cryptography)。

项目文档：[点我跳转](http://docs.paramiko.org/en/2.4/index.html)
扩展：ssh协议，OpenSSH

## 上手
1、安装
```powershell
pip install paramiko
```
2、导入模块
```python
import paramiko
```
3、使用

```python
def initSshClinet():
	'''
    初始化,SSH连接账号密码登录服务器
    :return: sshClinet
    '''
    ip = ""#服务器ip地址
    sshClinet = paramiko.SSHClient()
    sshClinet.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    sshClinet.connect(ip, 22, userName, pw, timeout=360)
    return sshClinet
```
```python
def exeCommond(commond):
	'''
	执行shell命令
	'''
	stdin, stdout, stderr = sshClient.exec_command(command)
    outStr = stdout.readlines()
    print("\n".join(outStr))
```

```python
def sftpUploadFile(localPath, remotePath):
    #获取SFTP实例
    sftp = sshClinet.open_sftp()
    #执行上传动作
    sftp.put(localPath, remotePath)

def sftpDownloadFile(localPath, remotePath):
    #获取SFTP实例
    sftp = sshClinet.open_sftp()
    #执行下载动作
    sftp.get(localPath, remotePath) 
```
末尾记得要关闭连接
```python
sshClient.close()
```
也可以使用私钥登录：
```python
# 配置私人密钥文件位置
private = paramiko.RSAKey.from_private_key_file('/Users/ch/.ssh/id_rsa')

#实例化SSHClient
client = paramiko.SSHClient()
 
#自动添加策略，保存服务器的主机名和密钥信息，如果不添加，那么不再本地know_hosts文件中记录的主机将无法连接
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
 
#连接SSH服务端，以用户名和密码进行认证
client.connect(hostname='10.0.0.1',port=22,username='root',pkey=private)
```

## 学习
paramiko包含两个核心组件：SSHClient和SFTPClient。

 - SSHClient的作用类似于Linux的ssh命令，是对SSH会话的封装，该类封装了传输(Transport)，通道(Channel)及SFTPClient建立的方法(open_sftp)，通常用于执行远程命令。
 - SFTPClient的作用类似与Linux的sftp命令，是对SFTP客户端的封装，用以实现远程文件操作，如文件上传、下载、修改文件权限等操作。
 
|名词|解释|
|--|--|
|Channel|是一种类Socket，一种安全的SSH传输通道|
|Transport|是一种加密的会话，使用时会同步创建了一个加密的Tunnels(通道)，这个Tunnels叫做Channel|
|Session|是client与Server保持连接的对象，用connect()/start_client()/start_server()开始会话|

### SSHClient常用的方法介绍
**connect()** ：实现远程服务器的连接与认证，对于该方法只有hostname是必传参数。

|参数|说明|
|--|--|
|hostname|连接的目标主机|
|port=SSH_PORT| 指定端口|
|username=None| 验证的用户名|
|password=None| 验证的用户密码|
|pkey=None| 私钥方式用于身份验证|
|key_filename=None| 一个文件名或文件列表，指定私钥文件|
|timeout=None| 可选的tcp连接超时时间|
|allow_agent=True|是否允许连接到ssh代理，默认为True 允许|
|look_for_keys=True| 是否在~/.ssh中搜索私钥文件，默认为True 允许|
|compress=False|是否打开压缩|

**set_missing_host_key_policy()** ：设置远程服务器没有在know_hosts文件中记录时的应对策略。传入MissingHostKeyPolicy的子类，目前支持三种策略：
设置连接的远程主机没有本地主机密钥或HostKeys对象时的策略，目前支持三种：
|MissingHostKeyPolicy的子类|说明|
|--|--|
|AutoAddPolicy |自动添加主机名及主机密钥到本地HostKeys对象，不依赖load_system_host_key的配置。即新建立ssh连接时不需要再输入yes或no进行确认|
|WarningPolicy |用于记录一个未知的主机密钥的python警告。并接受，功能上和AutoAddPolicy类似，但是会提示是新连接|
|RejectPolicy |自动拒绝未知的主机名和密钥，依赖load_system_host_key的配置。此为默认选项|

**exec_command()** ：在远程服务器执行Linux命令的方法。

**open_sftp()** ：在当前ssh会话的基础上创建一个sftp会话。该方法会返回一个SFTPClient对象。

### SFTPClient常用方法介绍
**from_transport(cls,t)** ：创建一个已连通的SFTP客户端通道
**put(localpath, remotepath, callback=None, confirm=True)** ：将本地文件上传到服务器 参数confirm：是否调用stat()方法检查文件状态，返回ls -l的结果
**get(remotepath, localpath, callback=None)** ：从服务器下载文件到本地
**mkdir()** ：在服务器上创建目录
**remove()** ： 在服务器上删除目录
**rename()** ：在服务器上重命名目录
**stat()** ：查看服务器文件状态
**listdir()** ：列出服务器目录下的文件
