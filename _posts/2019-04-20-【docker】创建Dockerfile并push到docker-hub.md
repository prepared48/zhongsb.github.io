---
layout: post
title:  "【docker】创建Dockerfile并push到docker-hub"
date:   2019-04-20 10:31:35
categories: 大数据
tags: 大数据
mathjax: true
---

* content
{:toc}

怎么将自己的程序包打包成dockerfile，让其他人可以通过docker下载你的程序包





# 创建 dockerfile

Dockerfile

PS: 	文件名字必须是这样`Dockerfile`	 

    FROM tomcat:8.0
    COPY index.jsp  /usr/local/tomcat/webapps/ROOT
    EXPOSE 8080
    
index.jsp

    <%
    out.println("Hello World, V1");
    %>
    
build 并测试

The docker build command processes this file generating a Docker Image in your Local Image Cache, which you can then start-up using the docker run command, or push to a permanent Image Repository.

docker build 创建一个docker镜像，这个镜像可以在你本地运行。

    $docker build -t mytomcat:1.0 .
    
    $docker run -p 8080:8080 -d mytomcat:1.0
    
    $curl localhost:8080
    
# push 到 docker hub

注册一个账号

Build image 



Docker login

输入用户名、密码

Docker push
    
第一步：修改 tag 名称

    // 必须先修改 tag 名称
    docker tag firstimage YOUR_DOCKERHUB_NAME/firstimage
    
    举例：
    docker tag mytomcat:1.0 preparedman/mytomcat:1.0
    
第二步：push

    docker push YOUR_DOCKERHUB_NAME/firstimage

    举例：docker push preparedman/mytomcat:1.0

Docker 测试

	https://cloud.docker.com/u/preparedman/repository/docker/preparedman/mytomcat

https://stackoverflow.com/questions/39508018/docker-driver-failed-programming-external-connectivity-on-endpoint-webserver