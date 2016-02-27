---
layout: post
title: MongoDB-Learn
tags: ["MongoDB", "Learn"]
category: Learn
---
mongodb 

### 1. MongoDB Install

    $ mkdir -p /data/db
    $ chmod -R $USER:$USER /data/db

在下载相应文件，并解压

启动：

    bin/mongod --dbpath 自定义
    bin/mongod #默认是在 /data/db

    bin/mongo

### 2. MongoDB 数据类型

> 特有特征： 文档， 集合（无模式）

> 基本数据类型: null， 布尔， 数字， 字符串， 数组，对象, hash

>> 数字： 32整数，64整数和浮点数

>>> 在shell（javascript）中仅仅支持一种数据类型，MongoDB支持三种，因此在shell中写入数据库的时候会更改数据类型。

>> 内嵌文档

>> 数组

### 3. 常见操作

    use foobar

    db.blog.insert({...})
    db.blog.update({old}, {new})
    db.blog.update({old}, {new}, true) #原来没有数据的时候创建，若有直接更新
    db.blog.update({old}, {new}, false, true) 更新全部的匹配文档

    db.blog.remove({item}) or db.blog.remove()

### 4. [mongodid rails](http://mongoid.org/en/mongoid/docs/installation.html)

    gem "mongoid", "~> 4.0.0"

    rails g mongoid:config

> myapp/config/application.rb

remove require "rails/all" instead of

    require "action_controller/railtie"
    require "action_mailer/railtie"
    require "rails/test_unit/railtie"
    require "sprockets/railtie" # Uncomment this line for Rails 3.1+

> myapp/config/environments/development.rb.

Make sure any references to active_record

    # Rails 4.0+
    # config.active_record.migration_error = :page_load


或者省略以上步骤：

    rails new app_name --skip-active-record
    rails new app -O

### 5. [mongodb继承](http://codecampo.com/topics/66)

在这个过程中需要注意一点，必须这么做才可以正常保存

    #parent.build_childs xxxx
    parent.childs = Child.new xxx
    if parent.save
      child.save #can not child.save first!!
    end

### 6. [rails操作数据库指南(CRUD)](http://mongoid.org/en/origin/docs/selection.html#symbol)

