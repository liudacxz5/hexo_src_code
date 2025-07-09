---
title: 算法基础知识
date: 2024-06-26 18:00:00
updated: 2024-06-26 18:00:00
tags: 
 - hexo
categories: 
 - 技术
 - hexo
keywords: 
 - hexo部署
description: hexo+github部署个人博客
top_img: 
comments: true
cover:
highlight_shrink: true
---
# 部署环境
## 1.安装git和node.js
## 2.本地连接github
### 2.1.设置username和email
```
git config --global user.name "username"
git cinfig --global user.email "email"
```
### 2.2.创建ssh密钥文件
#### 2.2.1.创建密钥
```
ssh-keygen -t rsa -C "email" 
```
#### 2.2.2.进入对应目录查看密钥：

c/Users/Administrator/.ssh/id_rsa.pub

#### 2.2.3.在GitHub上建立密钥

Settings => SSH and GPG keys => New SSH Key  

## 3.创建项目
Repository name取名格式：用户名.github.io
## 4.本地安装hexo
### 4.1.安装cnpm
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
### 4.2.安装hexo
#### 4.2.1.安装hexo-cli
```
cnpm install -g hexo-cli
```
#### 4.2.2.初始化hexo
```
hexo init
```
#### 4.2.3.配置 _config.yml
```
deploy:
  type: 'git'
  repository: 'https://github.com/你的地址'
  branch: 'master'
```
#### 4.2.4.发布到GitHub
```
# 安装上传工具
cnpm install hexo-deployer-git
# 上传到GitHub
hexo d
```
### 5.上传文章
#### 5.1.创建新文章
```
hexo new <title>
```
#### 5.2.清除旧数据
```
hexo clean
```
#### 5.3.生成新页面
```
hexo g
```
#### 5.3.文章预览
```
hexo s
```
#### 5.4.上传到GitHub
```
hexo d
```