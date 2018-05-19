---
title:  Rails中如何使用Action Cable
date:   2017-03-06
tags: [Rails, Action Cable]
categories: [Rails]
---
Action Cable 通过提供给客户端JavaScript框架和服务器端Ruby框架把WebSocket协议和Rails应用的其余部分无缝的集成起来，通过Action Cable，可以轻松的为Rails应用提供实时更新功能，不仅性能良好，而且具有可扩展性。

## 一、创建频道
### 1.1 创建products channel
```bash
# 在命令行输入指令创建products channel
rails_project> rails generate channel products
  create app/channels/products_channel.rb
  identical app/assets/javascripts/cable.js
  create  app/assets/javascripts/channels/products.coffee
# 通过返回的系统信息，我们看到rails创建了一个products_channel.rb文件和一个products.coffee文件
```
### 1.2 修改products_channel.rb
```ruby
# 打开products_channel.rb文件
class ProductsChannel < ApplicationCable::Channel
  def subscribe
    # 订阅流名为products的流，值得一提的是一个channel支持订阅多个流，目前咱们先做一个流
>   stream_from "products"
  end

  def unsubcribed
    # 取消订阅做的事情
  end
end
```
*默认情况下Rails只允许从本地主机访问频道，要想多台电脑进行开发的话，必须禁止这项检查。为此，需要在config/enviroments/development.rb中加一行代码*
```ruby
  config.action_cable.disable_request_forgery_protection = true
```
## 二、广播数据
当一个product（产品）更新了之后，就需要对整个目录进行广播，当然也可以发送给部分目录，或发送所需其他的数据。不过，因为我们已经拥有了目录视图，所以就要想办法使用它。

rails_project/app/controllers/products_controller.rb
```ruby
  def update
    response_to do |format|
      if @product.update(product_params)
        format.html { redirect_to @products, notice: 'Product was Successfully updated' }
        format.json { render :show, status: :ok, location: @product }

        # 获取更新后的所有产品数据
        @products = Product.all
        # 将新的数据广播出去
        # 同时调用render_to_string方法，将视图渲染为字符串，并传递layout: false作为参数，这是因为我们需要的只是这个视图，而不是整个页面
        # 广播消息通常是由Ruby的hash结构组成，他们被转换为JSON，然后发送，最终作为JS对象来处理，这里，我们使用html作为hash里的key
        # 所以我们现在广播的数据应该是这样的{html: '...'} 省略的部分是store/index页面字符串
        ActionCable.server.broadcast 'produsts', html: render_to_string('store/index', layout: false)
      else
        format.html { render :edit }
        format.json { render json: @product.errors, status: :unprocessable_entity }
      end
    end
  end
```
## 三、接收数据
最后一步是在客户端上接收数据，以及接收了数据之后要干什么

rails_project/app/assets/javascripts/channels/products.coffee
```ruby
  App.products = App.cable.subscriptions.create "ProductsChannel"
    connect: ->
      # 钩子方法： 当监听准备好时调用
    disconnected: ->
      # 钩子方法： 当监听被终止是调用
    received: (data) ->
      # 钩子方法： 当接受到数据时调用
      # 还记得广播数据中我们说的数据格式么（{html: '...'}）, Ruby通过.方法取出该hash的key(html)
      # 我们通过HTML元素查找，找到class 为store中的id为main的元素，并替换其中的html元素
      # 至此我们就完成使用ActionCable
      $(".store #main").html(data.html)
```
