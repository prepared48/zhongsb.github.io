---
layout: post
title:  "python基础—pip指定安装目录"
date:   2020-06-09 15:14:54
categories: 人工智能
tags: AI C#
mathjax: true
---

* content
{:toc}

pip 指定某个路径安装包

## 场景：

有的时候我们安装了annconda环境，有很多的python环境，比如py36, py37, py27。此时，我们使用`pip`安装包的时候，经常可能安装在一个不知道的路径，或者不是我们期望安装的路径。

这就是本文要解决的问题了。

## 方法一

指定安装`numpy`包到固定文件夹下，比如这里“文件夹”是安装路径

    pip install -t 文件夹 numpy
    

## 方法二

设置 pip 默认安装路径

找到 `site.py` 文件。（windows：可以通过自带的查找，或者使用 `everything`软件；Linux直接使用find命令即可）

我的目录：D:\program\Anaconda\envs\py36\Lib\site.py

修改 `USER_SITE` 和 `USER_BASE` 两个字段的值(之前是null).

    #自定义依赖安装包的路径
    USER_SITE = null
    #自定义的启用Python脚本的路径
    USER_BASE = null

我这里修改为

    USER_SITE = "D:\program\Anaconda\envs\py36\Lib\site-packages"
    USER_BASE = "D:\program\Anaconda\envs\py36\Scripts"

使用命令查看、验证

    python -m site

结果

    sys.path = [
        'C:\\Users\\z2010',
        'D:\\program\\Anaconda\\envs\\py36\\python36.zip',
        'D:\\program\\Anaconda\\envs\\py36\\DLLs',
        'D:\\program\\Anaconda\\envs\\py36\\lib',
        'D:\\program\\Anaconda\\envs\\py36',
        'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages',
        'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\torchvision-0.2.1-py3.6.egg',
        'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\tqdm-4.28.1-py3.6.egg',
        'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\pyahocorasick-1.4.0-py3.6-win-amd64.egg',
        'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\win32',
        'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\win32\\lib',
        'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages\\Pythonwin',
    ]
    USER_BASE: 'D:\\program\\Anaconda\\envs\\py36\\Scripts' (exists)
    USER_SITE: 'D:\\program\\Anaconda\\envs\\py36\\Lib\\site-packages' (exists)
    ENABLE_USER_SITE: True


pip安装到conda根目录了，需要安装到py36

公众号：关注【哥妞】，了解技术，学会泡妞~


