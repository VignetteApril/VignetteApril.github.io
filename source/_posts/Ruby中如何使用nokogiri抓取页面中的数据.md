---
title:  Ruby中如何使用nokogiri抓取网页中的数据
date:   2017-05-20
tags: [Ruby, nokogiri]
categories: [Ruby]
---

使用背景：老板想抓取搜房网的所有小区的bbs地址和对应的小区名字，最后生成一个excel文件。
技术选用：使用Ruby的gem nokogiri和spreadsheet

nokogiri相关介绍：nokogiri是一个html、xml、sax类型数据的读取和解析器，nokogiri可以通过Xpath和CSS3选择器去在文档中查找数据

spreadsheet相关介绍：Spreadsheet是用来读写excel文件的工具

## 一、使用nokogiri抓取目标网页的数据
首选我们想把需要用的gem包引入进来，open-uri是和nokogiri紧密结合的下面会用到
```Ruby
  require 'rubygems'
  require 'nokogiri'
  require 'open-uri'
  require 'spreadsheet'
```

然后通过nokogiri获取网页的全部数据，其中open方法是open-uri中得到的
```Ruby
  # 要抓取网页的URL
  uri = "http://bbs.fang.com/forumlist.php?_m=search_forum_list"

  # 使用Nokogiri抓取网页的内容，编码格式使用gbk
  doc = Nokogiri::HTML(open(uri), nil, "gbk")
```

然后通过nokogiri的css选择器的查询方式查找对应的数据，可以看到我们调用了nokogiri对象的css查找方法。然后我们通过nokogiri node对象对应的方法的到我们想要的数据。
nokogiri node对象：我们通过doc.css方法匹配到了很多数据，每一条数据都是一个nokogiri node对象，而doc.css返回的真个结果集是一个nokogiri node对象数组，本例中我们用到了node.children方法，获取该节点下所有的子节点的信息
```Ruby
  # 在网页内容里通过css进行搜索
  bbs_data = doc.css("dd ul li")

  # 初始化一个数组用于保存数据
  bbs_arr = []

  bbs_data.each do |bbs|
  # 通过nokogiri node的一些方法获取数据
  bbs_arr << ["#{bbs.children[1]}", "#{bbs.children[0].attribute('href').value}"]
  end
```

## 二、使用spreadsheet生成excel文件
至此我们得到了一个包含我们想要信息的数组：[ ['xxx小区','www.xxx.com'] ...]
注意：该格式是根据spreadsheet所需要的格式生成的。
得到数据之后我通过spreadsheet将数据写入到excel文件中
```Ruby
  # 通过Spreadsheet将获取的数据写入到excel格式的文件中，并保存在本地
  Spreadsheet.client_encoding = 'GBK'
  book = Spreadsheet::Workbook.new
  sheet1 = book.create_worksheet
  sheet1.row(0).concat %w{小区名称 论坛地址}
  bbs_arr.each_with_index do |item, index|
  sheet1.update_row index + 1, item[0], item[1]
  end
  book.write '/Applications/workSpace/sheet1.xls'
```
至此我们就生成了一个名为sheet1.xls文件里面包含了老板想要的所有信息

下面是整体的源码，可以直接在你本地运行的，运行前你需要修改下生成excel文件的地址.
注意：编码的问题，这里我是用的是GBK的编码格式，如果你在测试的时候看到终端都是乱码，那是因为终端的编码格式是UTF-8的
```Ruby
  # encoding : utf-8
  # doc.encoding: gbk
  require 'rubygems'
  require 'nokogiri'
  require 'open-uri'
  require 'spreadsheet'

  puts "get data from http://bbs.fang.com/forumlist.php?_m=search_forum_list"

  # 要抓取网页的URL
  uri = "http://bbs.fang.com/forumlist.php?_m=search_forum_list"

  # 使用Nokogiri抓取网页的内容，编码格式使用gbk
  doc = Nokogiri::HTML(open(uri), nil, "gbk")

  # 在网页内容里通过css进行搜索
  bbs_data = doc.css("dd ul li")

  # 初始化一个数组用于保存数据
  bbs_arr = []

  bbs_data.each do |bbs|
  # 通过nokogiri node的一些方法获取数据
  bbs_arr << ["#{bbs.children[1]}", "#{bbs.children[0].attribute('href').value}"]
  end

  # 通过Spreadsheet将获取的数据写入到excel格式的文件中，并保存在本地
  Spreadsheet.client_encoding = 'GBK'
  book = Spreadsheet::Workbook.new
  sheet1 = book.create_worksheet
  sheet1.row(0).concat %w{小区名称 论坛地址}
  bbs_arr.each_with_index do |item, index|
  sheet1.update_row index + 1, item[0], item[1]
  end
  book.write '/Applications/workSpace/sheet1.xls'
```
