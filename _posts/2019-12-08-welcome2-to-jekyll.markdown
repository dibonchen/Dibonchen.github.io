---
layout: post
title:  "Docker+Jenkins+github 实现 Vue 项目的持续集成"
date:   2018-12-08 20:27:20 +0800
categories: jekyll update
---

#### 环境
1. debian 9

#### 一.基础环境配置
1. 检查、安装jdk --openjdk即可(`apt install -y openjdk-8-jdk`)
2. 检查、安装git(`apt install git`)
3. 安装Docker

```
apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     lsb-release \
     software-properties-common
     
 curl -fsSL get.docker.com -o get-docker.sh
 sudo sh get-docker.sh --mirror Aliyun
```

#### 二.Vue项目配置
1. 项目根目录下新建Dockerfile
2. Dockerfile模板
   
```
FROM node:8-slim
RUN apt-get update  && apt-get install -y nginx
WORKDIR /usr/src/app
COPY ["package.json", "package-lock.json*", "./"]
RUN npm install
COPY . .
RUN npm run build
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
  && ln -sf /dev/stderr /var/log/nginx/error.log
EXPOSE 80
RUN cp -r dist/* /var/www/html \
  && rm -rf /user/src/app
CMD ["nginx","-g","daemon off;"]
```

#### 三.安装 Jenkins
1. 更新数据源
    1. `wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -`
    2. `sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'`
    3. `apt-get update`
 2. 安装Jenkins(`apt-get install jenkins`)
 3. 修改 Jenkins shell脚本运行Docker权限报错
  `gpasswd -a jenkins docker`
  `service jenkins restart`
 4. 浏览器打开 ip+8080端口 网址 初步配置Jenkins
     1. Unlock Jenkins：获取密码(`cat /var/lib/jenkins/secrets/initialAdminPassword`)
     2. 选择安装建议的插件(Install suggested plugins)![](http://pmfqxd845.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202019-02-04%2013.08.54.1.png)
     3. 注册管理员用户![](http://pmfqxd845.bkt.clouddn.com/09931761-84CC-4A70-9582-836EE0B2866E.png)
     4. 修改安全配置
     ![](http://pmfqxd845.bkt.clouddn.com/7B6B00DD-DBD7-4A4F-9F8C-D1D943CA4EEE.png)
     ![](http://pmfqxd845.bkt.clouddn.com/0CC4F955-87D0-4E6C-AFEA-0C4001438FD1.png)
5. 创建任务
    ![](http://pmfqxd845.bkt.clouddn.com/A3D73265-3A34-47ED-B2DA-0FC61CA30911.png)
    ![](http://pmfqxd845.bkt.clouddn.com/5F667F58-1FC1-4E60-8001-C4615154AC3E.png)
    ![](http://pmfqxd845.bkt.clouddn.com/BCB38BB5-C3EA-49D8-8458-6FC6E0BEA520.png)
6. 配置github项目的Webhooks
   ![](http://pmfqxd845.bkt.clouddn.com/4BAE6ACF-4726-4411-8870-F912E8F701C4.png)
   ![](http://pmfqxd845.bkt.clouddn.com/C1F55021-8609-4634-A4A2-113E103137DE.png)
7. 流水线脚本模板
  
```
node {
    sh '''#!/bin/sh

    REPOSITORY_NAME="你的仓库名"
    REPOSITORY_URL="你的仓库地址"
    IMAGE_NAME="给你要构建的镜像起个名字"
    CONTAINER_NAME="给你要构建的容器起个名字"
       
    echo "清除仓库目录"
    rm ${REPOSITORY_NAME} -r
       
    echo "克隆远程仓库"
    git clone ${REPOSITORY_URL}

    echo "删除之前的镜像和容器"
    docker stop ${CONTAINER_NAME}
    docker rm ${CONTAINER_NAME}
    docker rmi ${IMAGE_NAME}
    
    echo "构建镜像"
    cd ${REPOSITORY_NAME}
    docker build -t ${IMAGE_NAME} .
    
    echo "发布应用"
    docker run -d -p 80:3000 --name ${CONTAINER_NAME} ${IMAGE_NAME}'''

}
```



[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
