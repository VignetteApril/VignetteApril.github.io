---
title:  离线状态下配置Ruby环境
date:   2016-06-06
tags: [Ruby]
categories: [Ruby]
---

# 离线配置应用环境 copy

## 一、下载安装RVM

```bash
#1 下载RVM的稳定版安装包
curl -sSL https://github.com/rvm/rvm/tarball/stable -o rvm-stable.tar.gz

#2 创建RVM文件夹并进入该文件夹
mkdir rvm && cd rvm

#3 解压缩tar包
tar --strip-components=1 -xzf ../rvm-stable.tar.gz

#4 安装rvm
./install --auto-dotfiles

#5 添加环境变量 (如果是在centOS安装的话就是这个路径，如果是其他系统，请改为当前的rvm路径)
source ~/usr/local/rvm/scripts/rvm

#6 安装RVM所需要的依赖包
rvm requirements
```

## 二、编译安装ruby

```bash
#1 下载源代码
https://cache.ruby-lang.org/pub/ruby/2.x/ruby-2.x.x.tar.gz
注：自行替换x为具体的数字，比如我想下载2.2.5版本的ruby，我可以输入如下地址：
https://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.5.tar.gz

#2 解压缩源代码
tar zxf ruby-2.x.x.tar.gz

#3 进入解压缩出来的文件夹中
cd ruby-2.x.x

#4 执行配置命令（直接复制粘贴即可）
./configure --prefix=/usr/local --disable-install-doc --with-opt-dir=/usr/local/lib

#5 编译安装
make && make install

#6 添加ruby的环境变量
vim ~/.bash_profile
# 添加一行：
export PATH=/usr/local/ruby/bin:$PATH
# 保存退出，并且source
source ~/.bash_profile
```

## 三、RVM挂载ruby

```bash
# 使用rvm mount 命令可以挂载ruby
rvm mount /usr/local/ruby
```

## 四、下载安装rubygems

```bash
#1 下载最新的rubygems
https://rubygems.org/pages/download

#2 如果是zip格式的直接解压缩文件并进入

#3 安装rubygems
ruby setup.rb
```

## 五、下载安装bundler

```bash
#1 下载最新的bundler
https://rubygems.org/pages/download

# 本地安装bundler
进入到bundler.gem所在的文件夹输入：
gem install bundler -l
```
