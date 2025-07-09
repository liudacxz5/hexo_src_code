---
abbrlink: ''
categories:
- - 开发心得
date: '2025-07-08T10:28:17.213516+08:00'
description: 离线环境获取docker包，以mysql为例
keywords: docker
tags:
- docker
title: 离线环境获取docker包
updated: '2025-07-08T10:30:28.253+08:00'
---
## 在联网机器下载镜像

`docker pull mysql:8.0.36`

## 保存镜像为离线包

`docker save -o mysql8.tar mysql:8.0.36`

## 传输文件到离线环境后加载镜像

`docker load -i mysql8.tar`
