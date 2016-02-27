---
layout: post
title: Sidekiq and Redis 
category : [sidekiq, redis, learn]
tags : [sidekiq, redis, learn]

---
sidekiq and redis 

## Redis install
    brew install redis
    

启动：

    redis-server /usr/local/etc/redis.conf

默认启动状态为： localhost:6379

## Sidekiq

    gem 'sidekiq'

    # 下面是web ui 必须得
    gem 'sinatra', require: false
    gem 'slim' # or rails-slim

###1. 创建一个worker

    class PygmentsWorker
    include Sidekiq::Worker
    sidekiq_options queue: "high" # queue name, by yourself defined and other
                                  # like weight, retry, 
                                  # and  backtrace(default is false)
    # sidekiq_options retry: false
  
      def perform(xxxxx)
        p xxxxx
      end
    end

### 2. 启动

    bundle exec sidekiq # default is started
    
    bundle exec sidekiq -q test # start test quene 

    bundle exec sidekiq -q high,5 -q default # 5 is weight

或者可以创建一个配置文件，以配置文件启动

> config/sidekiq.yml

    ---
    :concurrency: 5
    :pidfile: tmp/pids/sidekiq.pid
    staging: # work model
      :concurrency: 10  # By default, one sidekiq process creates 25 threads.
                        # Set the pool setting to something close or equal to 
                        # the number of threads:
                        # pool: 10
    production: # work model
      :concurrency: 20
    :queues:
      - default
      - [myqueue, 2]

start: 
    
    bundle exec sidekiq -C config/sidekiq.yml

    bundle exec sidekiq -C config/sidekiq.yml -d 后台启动方式
    
    bundle exec sidekiq -C config/sidekiq.yml -d -e production 指定环境启动

### 3. Web UI

> route.rb

    require 'sidekiq/web'
    mount Sidekiq::Web, at: '/sidekiq'

[add auth](https://github.com/mperham/sidekiq/wiki/Monitoring), for example:

    require "sidekiq/web"
    Sidekiq::Web.use Rack::Auth::Basic do |username, password|
      username == ENV["SIDEKIQ_USERNAME"] && password == ENV["SIDEKIQ_PASSWORD"]
    end if Rails.env.production?
    mount Sidekiq::Web, at: "/sidekiq"

### 4. [Use Redis](https://github.com/mperham/sidekiq/wiki/Using-Redis)

> config/initializers/sidekiq.rb

    Sidekiq.configure_server do |config|
      config.redis = { url: 'redis://localhost:6379' }
    end

    Sidekiq.configure_client do |config|
      config.redis = { url: 'redis://localhost:6379' }
    end

### 5. [Active Job](https://github.com/mperham/sidekiq/wiki/Active-Job)

> config/application.rb
  
    config.active_job.queue_adapter = :sidekiq

    class ExampleJob < ActiveJob::Base
      rescue_from(ErrorLoadingSite) do
        retry_job wait: 5.minutes, queue: :low_priority 
      end 

      def perform(*args)
        # Perform Job
      end
    end

### Action Mailer

#### 1. Worker

#### 2  Job

