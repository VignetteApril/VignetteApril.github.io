---
title:  Rails中项目的目录结构
date:   2017-05-07
tags: [Rails]
categories: [Rails]
---

Rails对运行时的布局有特定的假设，而且为了这一布局提供了应用和脚手架生成器
使用rails new my_rails_project命令之后, 会生成如下的rails目录
```bash
  rails my_rails_project
```
my_rails_project/
  app/        -> 用来存放模型（model）控制器（controller）和视图（view）
  bin/        -> 用来存放包装的脚本
  config/     -> 用来存放配置文件和数据连接的配置文件等等
  db/         -> 用来存放数据库的迁移文件
  lib/        -> 用来存放共享的代码
  log/        -> 用来存放应用生成的日志文件(development.log, production.log, test.log)
  public/     -> 用来存放公开可访问的文件。应用也是从这里运行的
  test/       -> 用来存放测试代码
  tmp/        -> 用来存放运行时的临时文件
  vendor/     -> 用来存放导入的代码（gem的cache文件夹也在里面）
  Gemfile     -> gem的依赖情况
  Rakefile    -> 构建脚本
  README.md   -> 系统的说明情况
  config.ru   -> Rack服务器配置
