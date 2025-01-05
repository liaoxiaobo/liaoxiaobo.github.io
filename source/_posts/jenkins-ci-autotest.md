---
title: 使用Jenkins工具集成自动化测试
date:  2019-03-29
categories:
  - 测试自动化
tags:
  - Jenkins
  - Robot Framework
  - DevOps
---


# 需求背景

1、公司项目搭建了一套CICD，每天可以自动地构建镜像并部署至测试环境，这时需要对接CI自动化冒烟测试
2、Jenkins有着其丰富的插件和job触发机制，易于拓展，能够满足持续测试的大部分场景，尤其是jenkins pipeline使得创建任务非常灵活和简洁，最终选择jenkins作为持续集成测试平台

# 安装和配置Jenkins

## jenkins容器安装

```
# 在docker主机上执行命令：
docker run -u root -d  --name jenkins-ci -p 8081:8080 -p 50000:50000 -v /etc/localtime:/etc/localtime -v /etc/timezone:/etc/timezone -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock  jenkinsci/blueocean
```

> 如果宿主机不存在timezone文件，自己手动创建并写入Asia/Shanghai即可。
> docker run参数的具体说明请看官方文档，介绍得很详细。https://jenkins.io/zh/doc/book/installing/

执行命令截图：
![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk01.png?raw=true)

为了省心方便，在此默认选择了jenkins推荐的插件安装,这需要耐心等待一段时间：
![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk02.png?raw=true)

## 配置邮件通知

这里特别注意的是密码应该填qq邮箱的授权码，填写完毕测试一下邮件发送成功即可
![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk03.png?raw=true)

## 安装插件robot-framework

由于项目里的自动化脚本基于RF编写开发，所以Jenkins需要安装这个插件。该插件在Jenkins中收集并发布Robot Framework测试结果。bfyrjbqkzqvaibjd
至此，jenkins的基本配置工作就算完了，接下来最核心就是编写pipeline脚本。

# 编写Pipeline脚本

直接贴脚本，pipeline支持的语法请查看官方手册。

pipeline流程图：

![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk09.png?raw=true)

```
pipeline {
    agent any
    options {
        // 添加日志打印时间
        timestamps()
        // 设置全局超时间
        // timeout(time:10,unit:'MINUTES')
    }
    environment {
        // GIT_BRANCH = 'master' # 已设置全局变量
        GIT_USER_ID = '76f84542-2dba-4128-8651-bed7a849eddd'
    }
    stages {
        stage('Git Clone') {
            steps {
                // sh  'printenv |sort'
                git branch: "${GIT_BRANCH}", credentialsId: "${GIT_USER_ID}", url: 'https://gitee.com/wisecloud/wise2c-robot.git'
            }
        }
        stage('Build Image') {
            steps {
                sh 'docker build -t ${JOB_NAME}:${GIT_BRANCH} .'
            }
        }
        stage('API Test') {
            steps {
                script{
                    sh 'ls -a'
                    sh 'docker run --rm -v jenkins-data:/robot-results --name ${JOB_NAME} ${JOB_NAME}:${GIT_BRANCH} robot --outputdir /robot-results /wise2c-robot/Project/Suites/resource_manager.robot'
                }
            }
        }
    }
    post {
        always {
            echo 'Publish Test Report'
            robot logFileName: 'log.html', outputFileName: 'output.xml', outputPath: '/var/jenkins_home/', passThreshold: 100.0, reportFileName: 'report.html', unstableThreshold: 90.0
            // deleteDir() /* clean up our workspace */
        }
        success {
            mail bcc: '', body: "API自动化测试通过\n测试版本分支：${GIT_BRANCH}\n测试报告地址：${BUILD_URL}", cc: '', from: '1219199895@qq.com', replyTo: '', subject: "${JOB_NAME}测试报告", to: 'liaoxb@wise2c.com'
        }
        failure {
            mail bcc: '', body: "API自动化测试未通过，请相关模块的同事分析定位问题，谢谢大家。\n测试版本分支：${GIT_BRANCH}\n测试报告地址：${BUILD_URL}", cc: '', from: '1219199895@qq.com', replyTo: '', subject: "${JOB_NAME}测试报告", to: 'liaoxb@wise2c.com'
        }
    }
}
```

# webhook自动触发Jenkins pipeline

1、新建pipeline时，构建触发器选择‘触发远程构建’这项，输入token name。比如apitest
这时就可以获得一个webhhok地址，提供给维护CICD平台的同事即可。
2、把pipeline脚本内容粘贴到流水线里，检查下有没有语法错误，最后点击保存
3、现在可以用curl模拟发送一次webhhok请求，不过记得临时把全局安全配置里的授权策略给放开，改成没有任何限制。

```
curl JENKINS_URL/job/wise2c-robot/build?token=TOKEN_NAME
```



![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk05.png?raw=true)

# 测试结果报告展示

job首页测试结果展示，失败了1条case，同时收到了一封关于api测试失败的邮件通知：

![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk04.png?raw=true)
![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk07.png?raw=true)

测试结果详情页面展示，点击log.html链接可以直接查看日志；

![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk06.png?raw=true)
![jk.png](https://github.com/liaoxiaobo/liaoxiaobo.github.io/blob/blog/source/image/jenkins/jk08.png?raw=true)

注：如果到打开失败，可以参考这个解决办法。https://stackoverflow.com/questions/36607394/error-opening-robot-framework-log-failed
管理jenkins–>脚本命令行输入如下脚本:

```
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP","sandbox allow-scripts; default-src 'none'; img-src 'self' data: ; style-src 'self' 'unsafe-inline' data: ; script-src 'self' 'unsafe-inline' 'unsafe-eval' ;")
```



# 持续改进

1、目前创建的wise2c-robot任务脚本数量500+，全部跑完至少需要2个小时，后期打算按模块拆分成多个子任务job，并行执行测试。

2、邮件通知的正文内容过于简单，希望能把每条case的执行结果直接展示出来
