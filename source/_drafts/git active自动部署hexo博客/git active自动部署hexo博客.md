---
abbrlink: ''
categories:
- - hexo
date: '2025-07-10T14:38:31.305093+08:00'
description: git active自动部署hexo博客
keywords: hexo,git
tags:
- git
- hexo
title: git active自动部署hexo博客
updated: '2025-07-10T14:38:31.651+08:00'
---
## 生成git的token

## 修改hexo的_config.yml

```
deploy:
  - type: git
    repository:
      github: https://{$GH_TOKEN}@github.com/yourname/repo
      branch: 'master'
      ...
```

## Git Action 脚本

```
name: liuda's Blog CI/CD # 脚本 workflow 名称

on:
  push:
    branches: [main, master] # 当监测 main,master 的 push
    paths: # 监测所有 source 目录下的文件变动，所有 yml,json 后缀文件的变动。
      - '*.json'
      - '**.yml'
      - '**/source/**'

jobs:
  blog: # 任务名称
    timeout-minutes: 30 # 设置 30 分钟超时
    runs-on: ubuntu-latest # 指定最新 ubuntu 系统
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
      - name: Cache node_modules
        uses: actions/cache@v3 
        env:
          cache-name: cache-node-modules
        with:
          path: ~/.npm
          key: ${{ runner.os }}-build-${{ env.cache-name }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-build-${{ env.cache-name }}-
            ${{ runner.os }}-build-
            ${{ runner.os }}-
      - name: Init Node.js # 安装源代码所需插件
        run: |
          npm install
          echo "init node successful"
      - name: Install Hexo-cli # 安装 Hexo
        run: |
          npm install -g hexo-cli --save
          echo "install hexo successful"
      - name: Build Blog # 编译创建静态博客文件
        run: |
          hexo clean
          hexo g
          echo "build blog successful"
      - name: Deploy DoubleAm's Blog # 设置 git 信息并推送静态博客文件
        run: |
          git config --global user.name "liudacxz5"
          git config --global user.email "liudacxz5@163.com"
          hexo deploy

      - run: echo "Deploy Successful!"
```
