---
layout: post
title: Pm2 Deploy Node Using Mina
category : [Pm2, deploy, Node, Mina]
tags : [Pm2, deploy, Node, Mina]

---
deploy about node、pm2 and mina 

### 前言
  很早之前就想具体实际写一下node，最近尝试了一下

  作为Rubyist的补充技能，多多掌握也很好。

  一直觉得mina部署相当不错, pm2自带部署还是觉得不如[mina](https://github.com/mina-deploy/mina)直观和便捷。

### mina部署

#### 1. 安装Ruby和rvm
[wiki](https://ruby-china.org/wiki/rvm-guide), 只需要安装好rvm和ruby即可

#### 2. [mina](https://github.com/mina-deploy/mina)
mina 是我接触Rails以来一直使用的工具，很是便捷，但最新发现了一个[问题](https://github.com/mina-deploy/mina/pull/349)，我修复后可用这种方式安装： 

> 2.1 touch Gemfile
    
    source 'https://ruby.taobao.org/'
    group :development do
      gem 'mina', git: "https://github.com/Freeza91/mina.git", require: false
    end

> 2.2 bundle
 
>> 会生成Gemfile.lock 文件，本地可ignore

> 2.3 mina init && mina deploy 

>>完全可参考[官方wiki](https://github.com/mina-deploy/mina)

>> [我个人的配置参考](https://github.com/Freeza91/wechat-shake-game/blob/master/config%2Fdeploy.rb.example)


#### 3. [pm2-deploy](http://pm2.keymetrics.io/docs/usage/deployment/)

        {
          apps : [
            {
              name      : "wechats",
              script    : "/home/deploy/wechats/current/app.js",
              // or script    : "~/wechats/current/app.js",
              env: {
                COMMON_VARIABLE: "true"
              },
              env_production : {
                NODE_ENV: "production"
              }
            }
          ],
          deploy : {
            production : {
              /**
              * 这些可完全不用设置，在mina中已经设置成功
                user : "deploy",
                host : "test.geeklab.cc",
                port: '9527',
                ref  : "origin/master",
                repo : "git@git.coding.net:rudyboy/wechats-shake-game.git",
                path : "/home/deploy/wechats",
                "post-deploy" : "pm2 ecosystem.json5 --env production"
              **/
              /**
              * http://pm2.keymetrics.io/docs/usage/application-declaration/
              * 相关的所有设置可以参考以上链接
              */
            }
          }
        }

> 3.1 小技巧

    task :logs do 
      queue! %w{
        cd #{app_path}
        pm2 logs | grep #{app_name}
      }
    end

> 本地服务器直接运行 mina logs，即可直接打开服务器的logs 查看

### pm2 部署

pm2 部署要相对简单些

    {
      apps : [

        {
          name      : "huobi-trade",
          script    : "app.js",
          env: {
            COMMON_VARIABLE: "true"
          },
          env_production : {
            NODE_ENV: "production"
          }
        },
      ],

      /*
      *
      * pm2 ecosystem
      * 
      + deploy setup
      + pm2 deploy ecosystem.json5 production setup
      *
      + 必须是绝对路径
      + ln -nfs ~/huobi-trade/shared/config/application.js ~/huobi-trade/current/config/application.js
      *
      + deploy
      + pm2 deploy ecosystem.json5 production
      *
      */

      deploy : {
        production : {
          user : "deploy",
          host : "test.geeklab.cc",
          port : "9527",
          ref  : "origin/master",
          repo : "git@github.com:Freeza91/btc-auto-trade.git",
          path : "~/huobi-trade",
          "post-deploy" : "npm install && pm2 startOrRestart ecosystem.json5 --env production"
        }
      }
    }
