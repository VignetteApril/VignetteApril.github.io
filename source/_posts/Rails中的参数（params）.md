---
title: Rails中的参数
date: 2016-06-06
tags: [Rails]
categories: [Rails]
---

# Rails中的Params
```c
# Slim
= form_for(@user) do |f|
        = f.label :name, "姓名"
        = f.text_field :name

        = f.label :email, "邮箱"
        = f.email_field :email

        = f.label :password, "密码"
        = f.password_field :password

        = f.label :password_confirmation, "密码确认"
        = f.password_field :password_confirmation

        = f.submit "创建账户", class: "btn btn-primary"
```

假如我们有这样一个表单，当我们点击创建账户按钮的时候，控制台输出的调试信息是：
 ![图片](RailsWithParams_1.jpg)
本身Params是一个大的哈希，里面还嵌套的着一个哈希，我们看user对应的这个哈希。

```ruby

    "user" => { "name" => "Foo Bar",
                "email" => "foo@invalid",
                "password" => "[FILTERED]",
                "password_confirmation" => "[FILTERED]"
              }
```

这个哈希是params的一部分，会传给Users控制器，前面说过params哈希中包含每次请求的信息，例如向/users/1发送请求时，params[:id]的值是用户的ID，即1，提交表单发送POST请求时，params是一个嵌套哈希。从面的可以看出提交表单后Rails会够将一个名为user的哈希，哈希中键是input标签的name属性的值，键对应的值是用户在字段中填写的内容例如：

```ruby
<input id="user_email" name="user[email]" type="email" />
```

其中，name属性的值是user[email]，表示user哈希中的email元素。
虽然调试信息的键是字符串形式，不过却以符号形式传给Users控制器。params[:user] 这个嵌套好戏实际就是User.new方法创建用户所需的参数:


```ruby
# 上下两种方式其实是等价的，所以现在就好理解，rails是如何通过参数创建一个新用户了
@user = User.new(params[:user])
@user = User.new(name: "Foo Bar", email: "foo@invalid",
                     password: "foo", password_confirmation: "bar")
```



## 健壮参数

像如下代码，用一个哈希去创建一个新用户，但是这样做是十分危险的，会把用户提交的所有数据都传给User.new方法，假设除了这几个属性，User模型中还有一个admin属性，用于标识网站的管理员，如果把这个属性设置为true，要在param[:user]中包含admin= '1'。这个操作可以使用curl等命令HTTP客户端轻易实现，如果把整个params哈希传给User.new,那么网站中的任何用户都可以在请求中包含admin= '1',来获取管理员权限。

```ruby
@user = User.new(params[:user])
```

从Rails 4.0开始，推荐在控制器层使用一种叫做健壮参数的技术，这个技术可以指定需要哪些请求参数，以及允许传入哪些请求参数。而且如果按照上面的传入整个params哈希的方式，应用会抛出异常。
本例，我们需要params哈希包含:user元素，而且只允许传入 name, email, password, password_confirmation属性，这个需求可以如下实现：

```ruby
params.require(:user).permit(:name, :email, :password, :password_confirmation)
```

这行代码会返回一个params的哈希，只包含允许使用的属性，而且，如果没有指定:user元素还会抛出异常。为了使用方便,可以定义一个名为 user_params 的方法,换掉 params[:user],返回初始化所需的哈希:

```ruby
def create
    @user = User.new(user_params) if @user.save
    # 处理注册成功的情况 else
    render 'new'
    end
end

private
    def user_params
    params.require(:user).permit(:name, :email, :password,
                                       :password_confirmation)
    end
end
```
