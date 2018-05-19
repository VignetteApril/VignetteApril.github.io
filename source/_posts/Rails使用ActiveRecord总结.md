---
title:  Rails使用AcitiveRecord总结（一）
date:   2017-07-16
tags: [Rails, AcitiveRecord]
categories: [Rails]
---
AcitiveRecord是Rails提供的对象关系映射，是Rails中负责实现应用模型的部分。
本例我们通过一个类讲述下ActiveRecord的一些基本用法

我们先通过命令行生成一个机遇ActiveRecord的模型
```bash
  # 这里我使用rails的generate命令生成了User模型并给它指定了name和age字段
  rails g model User name age:int
```
这里需要注意的一点是，我的name字段并没有指定字段类型，这是因为Rails的默认字段类型设置的是string。
这条命令生成了一个User类，这个类继承自AcitveRecord::Base类，还生成了一个migration保存在db/migrations里面

执行完上述命令后，就可以执行迁移命令，该命令是将刚才创建的模型对应的在数据库创建好表
```bash
 rails db:migrate
```
这里需要注意Rails5之前都是使用rake db:migrate，Rails5之后统一改为rails db:migrate
执行完迁移命令后，Rails会在数据库中生成一个users的表，可以看出users是user的复数形式，Rails会智能的匹配大多数的单复数形式，但是也会偶尔遇到无法正确处理的情况，如果遇到这种情况，可以修改磁性变化文件，让Rails知道某些英文单词的特殊性
rails_project/config/initializers/inflections.rb
```ruby
  ActiveSupport::Inflector.inflections do |inflect|
    inflect.irregular 'tax', 'taxes'
  end
```
在上述代码中可以看到，我们指定了tax的复数为taxes，这样我们在使用迁移命令的时候，名为Tax的模型类就会在数据库创建一个名为taxes的数据表
