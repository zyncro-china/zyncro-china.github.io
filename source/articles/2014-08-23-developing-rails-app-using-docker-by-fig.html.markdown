---
title: Developing Rails App Using Docker by fig
date: 2014-08-23 03:20 UTC
tags:
---
## 基于虚拟机或容器创建开发环境的优点
1. 可以方便地切换不同的项目
1. 在团队内共享项目的开发环境，避免所有人都浪费时间来设置环境
1. 避免不同项目的设置冲突
1. 可以使开发环境更加接近部署的环境

## Docker vs VirtualBox
以前试过使用VirtualBox虚拟机来做开发环境，因为团队里同时使用Linux和Windows。
但发现有2个问题，一是速度太慢，二是在windows下，编辑器保存文件后并不能触发
VirtualBox里的自动化编译或刷新。

因此尝试使用Docker,因为现在不需要再考虑windows,而且Docker基于Linux容器，性能更好。

## 在国内使用Docker的设置
为了能在国内使用docker，而且防止VPN不稳定的问题，需要找到docker的ip，并设置hosts
文件如下

```bash

# in /etc/hosts
54.205.182.244  get.docker.io
162.242.195.77 cdn-registry-1.docker.io
107.22.52.107 index.docker.io

```
这样才可以顺利地获取docker images。

## 配置文件
需要2个配置文件。

Dockerfile

```bash

FROM ruby
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev mongodb mongodb-dev mongodb-clients
RUN mkdir /myapp
WORKDIR /myapp
ADD Gemfile /myapp/Gemfile
RUN bundle install
ADD . /myapp
```
fig.yml


```yaml

db:
  image: mongo
  ports:
    - "27017"
web:
  build: .
  command: bundle exec rackup -p 3000
  volumes:
    - .:/myapp
  ports:
    - "3000:3000"
  links:
    - db
```
## 设置Gemfile
在Gemfile指定rails的版本

```ruby
source 'https://rubygems.org'
gem 'rails', '4.1.5'
```
## 下载images

运行命令:

```bash
fig run web rails new .

```

会自动下载imgaes，并执行Dockerfile里的命令, 再生产新的rails app。

这会在目录下生产所有的文件，需要重新把文件所有者改为自己：

```bash
chown -R `whoami` *
```
## 设置rails

加入需要的gems到Gemfile，再执行

```bash
fig build
```

## 设置数据库
打开database.yml,把host改为db_1,这就是在另一个容器的db

##启动

```bash
fig up
```
##运行其他命令

```bash
fig run web rake db:create

or

fig run web bash
```

