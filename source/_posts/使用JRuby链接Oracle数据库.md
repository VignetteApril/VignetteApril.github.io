---
title:  使用JRuby链接Oracle数据库
date:   2016-07-05
tags: [JRuby, Oracle]
categories: [JRuby]
---

1. 下载JDK
2. 下载JDBC
3. 下载JRuby:可以使用RVM安装，然后将第二步下载好的ojdbc-x移动到/path_to_rvm/.rvm/rubies/jruby-9.0.5.0/lib目录下
4. 拷贝代码jdbc_connection.rb和test_oracle.rb，这两rb文件要放在同一个目录下，然后在终端执行 ruby test_oracle.rb就可以和oracle建立连接了

### jdbc_connection.rb 代码如下
```ruby
  # jdbc_connection.rb

  require 'java'

  java_import 'oracle.jdbc.OracleDriver'
  java_import 'java.sql.DriverManager'

  class OracleConnection

  @conn = nil

  def initialize (user, passwd, url)
    @user = user
    @passwd = passwd
    @url = url

    # Load driver class
    oradriver = OracleDriver.new

    DriverManager.registerDriver oradriver
    @conn = DriverManager.get_connection url, user, passwd
    @conn.auto_commit = false
    puts "链接成功"
  end

  # Add getters and setters for all attrributes we wish to expose
  attr_reader :user, :passwd, :url, :connection

  def close_connection()
    @conn.close() unless @conn
    puts "链接关闭"
  end

  def prepare_call(call)
    @conn.prepare_call call
  end

  def create_statement()
    @conn.create_statement
  end

  def prepare_statement(sql)
    @conn.prepare_statement sql
  end

  def commit()
    @conn.commit
  end

  def self.create(user, passwd, url)
    conn = new(user, passwd, url)
  end

  def to_s
    "OracleConnection [user=#{@user}, url=#{@url}]"
  end
  alias_method :to_string, :to_s

  end
```


### test_oracle代码如下
```ruby
  $LOAD_PATH.unshift(File.dirname(__FILE__))

  # test_oracle.rb

  require 'jdbc_connection'

  # Edit these for your database schema
  # 服务器用户名称
  user   = "redouble"
  # 用户密码
  passwd = "victory"
  # 服务器的地址，注意写法jdbc:oracle:thin:@url:端口号:服务器ming'che
  url    = "jdbc:oracle:thin:@192.168.31.221:1521:orcl"

  print "Run at #{Time.now} using JRuby #{RUBY_VERSION}\n\n"

  begin
    # 建立链接
    conn = OracleConnection.create(user, passwd, url)
    # Display connection using the to_s method of OracleConnection
    puts conn, "\n"

    # 写sql语句
    query_sql = "SELECT owner, table_name from dba_tables where owner = 'REDOUBLE'"
    #query_stmt = conn.prepare_statement(query_sql)
    # 创建一个任务
    query_stmt = conn.create_statement
    # 使该任务执行上述的sql语句
    rs = query_stmt.executeQuery(query_sql)
    # 由于是查询语句，得到是一个结果集,需要使用next操作
    while (rs.next) do
      ...
    end
  end
```
