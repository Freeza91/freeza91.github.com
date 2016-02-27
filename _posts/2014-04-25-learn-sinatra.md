---
layout: post
title: Learn-Sinatra
tags : ["sinatra", "ruby"]
category : lesssion
---
sinatra learn 

## 1. sinatra 介绍
sinatra是一个更加简洁的ruby web框架

## 2. sinatra [文档](http://www.sinatrarb.com/intro.html)

### 2.1 [初步的文件结构](https://github.com/Useyes/learn-sinatra)

    Rakefile, Gemfile, config.ru, main.rb

### 2.2 activerecord

    rake -T #查看所有的rake命令
    rake db:create_migration NAME=create_posts #此处是生成的一个数据库表
    
    #在db/migrate/xxxxxxx.rb中会有这样的代码
    class CreatePosts < ActiveRecord::Migration
        def change
            #这里面添加一些相关的操作
            #create_table :posts do |t|
                t.string :title
                .....
            #end
            #add_column, remove_column
        end
    end

更多的操作[参考](http://guides.rubyonrails.org/active_record_querying.html)

## 3. [templates](http://www.sinatrarb.com/intro.html#Static%20Files)

### 3.1 erb
    
    erb :index #views/index.erb
    
    code = "this is code text"
    erb code   # return text or 

    erb :index, :layout => 'false'

### 3.2 static files

    set :public_folder, File.dirname(__FILE__) + '/static'

## 4. filter

### 4.1 before, after

    before get 'xxxxxx' do # get can be post
    end

    before do
        #can do some conditions
        
        if request.path_info == 'xxxxx'
        end

    end

一些[request的请求细节](http://www.sinatrarb.com/intro.html#Accessing%20the%20Request%20Object)

## 5. helper

### 5.1 helper
    
    helpers do 
        def xxx
        end
    end

### 5.2 halt
这个是结束当前的动作，直接返回规定的值

### 5.3 set or enable

#### 5.3.1 set 参考上面的静态文件

    set :session_secret, 'super secret'
    set :sessions, :domain => 'foo.com'
    ......

#### 5.3.2 enable

    enable :sessions

#### 5.3.3 [Triggering Another Route](http://www.sinatrarb.com/intro.html#Triggering%20Another%20Route)

#### 5.3.4 Setting Body, Status Code and Headers

### 5.4. redirect

    redirect to('/bar?sum=42')
    redirect back
    redirect 'http://google.com', 'wrong place, buddy'
    


    


    
