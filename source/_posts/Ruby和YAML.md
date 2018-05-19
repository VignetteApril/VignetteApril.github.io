---
title: Ruby和YAML
date: 2016-06-12
tags: [Ruby, YAML]
categories: [Ruby]
---

# Ruby中的YAML

## Ruby和YAML的关系

Ruby和YAML的联系，甚至比Java与XML的联系还要紧密。Ruby把YAML用到了和数据相关的方方面面。配置文件的约定格式是YAML。同时YAML还是Ruby的文本序列化格式，就像XML是SDO的文本序列化格式一样。

## 列表

表示一个列表很简单，在每个条目前加入一个短横线。比如

```ruby
- Item1
- Item2
- Item3
```

在Ruby中会被解释为数组对象(Array),上例为：

```ruby
[“Item1”, “Item2”, “Item3”]
```

另外，如果向如下情况

```ruby
indusdata_tables:
  综合:
    - [供应商综合信息表, [收款人名称]]
```

Ruby解析后如下：

```ruby
# indusdata_tables: 对应一个哈希
'indusdata_tables' => { '综合' => [ ['供应商综合信息表', [收款人名称] ] ] }
```

如果想在Ruby代码中使用YAML配置文件中的内容


## 哈希

类似于下列代码

```ruby
key1 : value1
key2 : value2
key3 : value3
```

在Ruby中会被解释为如下的哈希对象

```ruby
{"key1"=>"value1", "key2"=>"value2", "key3"=>"value3"}
```

## 字符串和其他基本类型

YAML会自动判断类型，一般性的文字都会被解释成为字符串。在有可能发生歧义的情况下，可以为字符串加上单引号或者双引号（在双引号下转义字符会被转义，方式与C语言类似）来避免歧义。
下面是一个例子。包含有字符串，正数，浮点数，和日期类型

```ruby
1 : 1.0
1.0 : "1.0"
"1.0" : 2006-01-01
```

上述代码在Ruby中会被解释为

```ruby
{1=>1.0, "1.0"=>#<Date: 4907473/2,0,2299161>, 1.0=>"1.0"}
```

可以看出它的类型被YAML很好的识别出来了

## 字符串块

字符串可能会占据多行，可以用两种方式来处理这种情况，一种是保留换行符，另外一种则是去除换符
用|来表示保留换行符：

```ruby
|
  This is line1.
  This is line2.
  This is line3.
```

它会被Ruby解释为：

```ruby
"This is line1./nThis is line2./nThis is line3."
```

用>表示删除换行符

```ruby
>
  Hello,
  world.
```

它会被Ruby解释为

```ruby
"Hello,  world."
```


除了上面的基本类型，还有更复杂的类型，比如列表中也可以存在另一个列表，哈希中也可能存在另一个哈希，像这种嵌套的方式在YAML中 也是支持的

对于重复出现的项目，可以用&进行定义，然后用*进行引用。下面用一个比较完整的例子来说明这些内容

```ruby
logEvent:    Purchase Invoice
date:        2007-08-06
customer:
    given:   Dorothy
    family:  Gale

bill-to:  &id001
    street: |
            123 Tornado Alley
            Suite 16
    city:   East Westville
    state:  KA

ship-to:  *id001   

items:
    - part_no:   A4786
      descrip:   Water Bucket (Filled)
      price:     1.47
      quantity:  4

    - part_no:   E1628
      descrip:   High Heeled "Ruby" Slippers
      price:     100.27
      quantity:  1

specialDelivery:  >
    Follow the Yellow Brick
    Road to the Emerald City.
    Pay no attention to the
    man behind the curtain.
```

在Ruby中解释如下

```ruby
{
  "items"=>[
    {"price"=>1.47, "quantity"=>4, "part_no"=>"A4786", "descrip"=>"Water Bucket (Filled)"},
    {"price"=>100.27, "quantity"=>1, "part_no"=>"E1628", "descrip"=>"High Heeled /"Ruby/" Slippers"}
  ],
  "bill-to"=>{"city"=>"East Westville", "street"=>"123 Tornado Alley/nSuite 16/n", "state"=>"KA"},
  "specialDelivery"=>"Follow the Yellow Brick Road to the Emerald City. Pay no attention to the  man behind the curtain.",
  "date"=>#<Date: 4908637/2,0,2299161>,
  "ship-to"=>{"city"=>"East Westville", "street"=>"123 Tornado Alley/nSuite 16/n", "state"=>"KA"},
  "customer"=>{"given"=>"Dorothy", "family"=>"Gale"},
  "logEvent"=>"Purchase Invoice"
}
```

## YAML和JSON

注：近来火热的JSON可以看作是（几乎是）YAML的一个子集，一般说来，YAML的解析器都可以解析JSON文档。
