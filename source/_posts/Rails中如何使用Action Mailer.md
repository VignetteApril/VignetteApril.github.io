---
title:  Rails中如何使用Action Mailer
date:   2017-04-06
tags: [Rails, Action Mailer]
categories: [Rails]
---

Action Mailer提供给编程人员发送给客户邮件的全部功能，其中包含三个步骤1、配置邮件的发送方式 2、发送邮件 3、电子邮件的内容

## 一、配置邮件的发送方式
电子邮件的配置是Rails应用的环境配置的组成部分，具体是通过Project::Application.configure代码块实现的，如下所示
```ruby
  Project::Application.configure do
    # 填写相关邮件的配置即可
  end
```
如果想在开发、测试和生产环境中使用相同的配置，可以把配置添加到config目录下的environment.rb文件中；否则，可以把不同环境的配置添加到config/environments目录下的不同配置文件中。

下面来看Project::Application.configure代码块中都要填写哪些内容，和意义
```ruby
  Project::Application.configure do
    # action_mailer_delivery_method 这个属性是设置发送邮件的方式
    # :smtp 该选项是开发环境默认的配置，也就是开发环境是默认允许程序发送邮件的，如果想禁止开发环境发送邮件可以在config/enviroments目录下的development.rb文件添加 配置选项为 :test的Project::Application.configure代码块
    # :sendmail 该选项其实是将电子邮件的发送工作委托给了本地系统的sendmail程序，并假定该程序在/usr/bin目录下。由于不同的操作系统sendmail的位置不尽相同，所以该选项的可移植性并不高，所以不建议使用。
    # :test 测试环境使用，当使用该选项是电子邮件并不会真正的发送，而是添加到了一个数组中去，可以通过ActionMailer::Base.deliveries属性访问；该选项也是测试环境邮件的默认选项。
    config.action_mailer_delivery_method = :smtp
  end
```
下面我们需要着重说一下:smtp选项：
使用默认的:smtp选项是为了获得更好的可移植性。为了使用该选项我们还需要提供一些配置，告诉ActionMailer在哪查找SMTP服务器，以及如何处理需要发送的电子邮件。SMTP服务器既可以是运行该Web的应用的服务器，也可以是独立的服务器。
下面我展示下我的163邮箱的SMTP配置
```ruby
  Project::Application.configure do
    config.action_mailer_delivery_method = :smtp

    config.action_mailer.smtp_settings = {
      address: "smtp.163.com",
      port: 465,
      domain: "www.163.com",
      authentication: "plain",
      user_name: "1360****771",
      password: "*********",
      enable_starttls_auto: true
    }
  end
```
## 二、发送邮件
完成配置后我们就可以编写代码来发送邮件了
邮件的代码程序是储存在app/mailers目录下的一个类，其中包含一个或多个方法，每个方法都一个对应的电子邮件模板。通过这些方法对应的视图，可以创建电子邮件的正文（就像是通过控制器动作创建html或xml一样）。
首先我们先使用rails generate命令创建一个mailer
```bash
  rails_project> rails generate mailer Order received shipped
    create app/mailers/order.rb
    invoke erb
    create app/views/order_mailer
    create app/views/order_mailer/received.text.erb
    create app/views/order_mailer/shipped.tex.erb
    invoke test_unit
    create test/mailers/order_test.rb
    create test/mailers/previews/order_mailer_preview.rb
```
输入指令后我们发现Rails首先创建在app/mailers目录下创建了一个order.rb的文件，这个文件是一个名为OrderMailer类所在的文件，该类继承自Application::Mailer，这里需要注意的是OrderMailer是刚才命令自动创建的类。
然后Rails在app/views/下创建了order_mailer文件夹，并在该文件夹里面创建了recevied.text.erb, shipped.text.erb文件，这两个文件对应的是命令中的两个action。
下面我们看看OrderMailer的代码部门
```ruby
class OrderMailer < ApplicationMailer
  default from: 'Zhangjie <13601115771@163.com>'
  # Subject can be set in your I18n file at config/locales/en.yml
  # with the following lookup:
  #
  #   en.order_mailer.received.subject
  #
  def received(order)
    @order = order

    mail to: order.email, subject: 'ACG Store Order Confirmation'
  end

  # Subject can be set in your I18n file at config/locales/en.yml
  # with the following lookup:
  #
  #   en.order_mailer.shipped.subject
  #
  def shipped(order)
    @order = order

    mail to: order.email, subject: 'ACG Store Order Shipped'
  end
end
```
这两有两点需要讲一下
1. mail方法
```ruby
  :subject - 邮件的标题

  :to - 邮件发送的对象, 可以是字符串类型的邮件地址信息, 或者是一个地址信息数组

  :from - 邮件发送者自己的地址信息

  :cc - 抄送的对象，可以是字符串类型的邮件地址信息, 或者是一个地址信息数组

  :bcc - 密件抄送对象, 可以是字符串类型的邮件地址信息, 或者是一个地址信息数组

  :reply_to - 电子邮件的回复标头

  :date - 发送电子邮件的日期
```
2. default方法
  该方法是可以设置默认的mail方法接受的参数
  比如本例中使用了如下default设置
  ```ruby
    default from: 'Zhangjie <13601115771@163.com>'
  ```
  mail方法的from参数被设置默认为*Zhangjie <13601115771@163.com>*
  当然你也可以设置其他的参数默认值

至此我们的controller已经简单的设置好了，下面我们再去看看ActionMailer的view层。在刚才输入指令生成邮件程序的时候，我们看到rails创建了两个后缀为text.erb的文件，这两个就是电子邮件的view层文件，它从controller里获取到数据，然后将数据展示在界面上。
```bash
  create app/views/order_mailer/received.text.erb
  create app/views/order_mailer/shipped.tex.erb
```
我们可以看看app/views/order_mailer/received.text.erb的内部情况
```erb
  Dear <%= @order.name %>

  Thank you for your recent order from the ACG store.

  you ordered the following items:

  <%= render @order.line_items -%>

  we will send you a separate e-mail when you order ships.
```

完成电子邮件模板的设置和邮件程序动作的定义后，就可以在普通的控制器中通过他们来创建和发送电子邮件了
比如当订单创建后我们需要发送邮件告诉顾客，订单已经生效
rails_project/app/controllers/orders_controller.rb
```ruby
def create
    @order = Order.new(order_params)
    @order.add_line_items_from_cart(@cart)

    respond_to do |format|
      if @order.save
        Cart.destroy(session[:cart_id])
        session[:cart_id] = nil
        # 该行即为发送邮件的代码，可以看出我们使用了OrderMailer类，并调用了类方法received，并传入了参数@order，并再在这之上调用了deliver_later方法
        # deliver_later: 使用Active Job将邮件发送的任务加入到队列中去
        # deliver_now: 立刻发送邮件
    ->  OrderMailer.received(@order).deliver_later
        format.html { redirect_to store_index_url(locale: I18n.locale),notice: I18n.t('.thanks') }
        format.json { render :show, status: :created, location: @order }
      else
        format.html { render :new }
        format.json { render json: @order.errors, status: :unprocessable_entity }
      end
    end
  end
```
至此邮件应该已经发送出去了，大家可以自己测一测
