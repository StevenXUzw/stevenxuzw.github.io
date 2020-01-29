---
layout: post
title: python操作Jinkins
---

> Jenkins不仅有web界面交互，也提供了RestAPI的方式来使用代码（Python、Java）控制Jenkins的工作流;
> 可以直接使用curl命令发送接口触发， [Jenkins官网的wiki](https://wiki.jenkins-ci.org/display/JENKINS/Remote+access+API)

## 简介
python有现成的轮子，使用python-jenkins库实现代码操作jinkins，参考：[项目文档](https://pypi.org/project/python-jenkins/)

|功能| 说明 |
|--|--|
| Create new jobs | 创建任务 |
| Copy existing jobs| 复制已有任务| 
| Delete jobs| 删除任务| 
| Update jobs| 修改任务| 
| Get a job’s build information| 获取任务构建信息| 
| Get Jenkins master version information| 获取Jenkins master版本信息| 
| Get Jenkins plugin information| 获取插件信息| 
| Start a build on a job| 构建任务| 
| Create nodes| 创建节点| 
| Enable/Disable nodes| 启用/禁用节点| 
| Get information on nodes| 获取节点信息| 
| Create/delete/reconfig views| 创建/删除/编辑 视图| 
| Put server in shutdown mode (quiet down)|进入关机模式 | 
| List running builds| 获取正在运行的所有构建| 
| Delete builds| 删除构建| 
| Wipeout job workspace| 清除构建工作区| 
|Create/delete/update folders |创建/删除/更新 文件夹，需要安装插件[Cloudbees Folders Plugin](https://wiki.jenkins-ci.org/display/JENKINS/CloudBees+Folders+Plugin)| 
|Set the next build number|设置下一个构建版本号，需要安装插件[Next Build Number Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Next+Build+Number+Plugin)|
|Install plugins|安装插件|
|and many more..|...|

## Jenkins设置
### 修改Jenkins的“安全设置”
![修改全局安全配置为‘登录用户可以做任何事’](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MDU5NDAwLWVlY2FjZTgwNGRmYTc1N2MucG5n?x-oss-process=image/format,png)
修改完后，重启Jenkins服务
### Jenkins用户的API Token
![Jenkins中设置当前登录用户的 API Token](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MDU5NDAwLWMxYTMyYjU1NTRlOTY4N2QucG5n?x-oss-process=image/format,png)
![修改用户所属的API Token](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91cGxvYWQtaW1hZ2VzLmppYW5zaHUuaW8vdXBsb2FkX2ltYWdlcy80MDU5NDAwLWVlMTliOWQ1ZGQ4OTE5MDIucG5n?x-oss-process=image/format,png)

## 上手
### 安装
```Shell
pip install python-jenkins
```
### 使用
```Python
	'''
	使用用户api token登录
	'''
    url = "http://{}:{}/jenkins/".format(host, port)
    jenkinsApi = jenkins.Jenkins(url, username="admin", password=apiToken)
    
    #获取任务的最新一次构建版本号
    number = jenkinsApi.get_job_info(jobName)['lastBuild']['number']
    
    #构建任务
    jenkinsApi.build_job(jobName)
    
    #带参数构建任务
    jenkinsApi.build_job(jobName, {'Tag': tag})
```
### 常用案例封装
```Python
def queryTaskStatus(jenkinsApi, jobName:str, number:int):
	#查询任务状态
    printLog("query jenkins task status")
    try:
        #获取构建信息
        # queue_info["building"]=True：构建完成
        # queue_info["result"] == 'SUCCESS'：构建成功
        queue_info = jenkinsApi.get_build_info(jobName, number)
        if queue_info["building"]:
            return 0
        elif "testcase" not in jobName and not queue_info["result"] == 'SUCCESS':
            dingTalkNotice("jenkins build fail :{}_{}".format(jobName, number))
        else:
            return 1
    except Exception:
        return 0
```
```Python
def stopLastProcessingTest(jenkinsApi, jobName):
    '''
    获取最新一次构建，如未完成，停止构建
    '''
    lastBuild = jenkinsApi.get_job_info(jobName)['lastBuild']
    if queryTaskStatus(jenkinsApi, jobName, lastBuild['number']) == 0:

        jenkinsApi.stop_build(jobName, lastBuild['number'])
        printLog("stop the last running jenkins task")
        time.sleep(2)
```
```Python
def getJobInfo(jenkinsServer, jobName):
	'''
	查询任务详情，获取最新一次构建的action参数中的tag
	'''
    number = jenkinsServer.get_job_info(jobName)['lastBuild']['number']
    info = jenkinsServer.get_build_info(jobName, number)
    tag = info["actions"][0]['parameters'][0]['value']
    printLog("{} server lasted tag :{}".format(name, tag))
    return tag
```

### 踩坑记录
1、没有操作权限，报错：
```Shell
jenkins.BadHTTPException: Error communicating with server[http://192.168.1.1:8080/]”
```
解决：照上文，修改全局安全配置为‘登录用户可以做任何事’
