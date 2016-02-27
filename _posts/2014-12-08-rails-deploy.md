---
layout: post
title: Rails deploy
tag : [deploy, rails]
category : lession
---
Rails deploy 

### 1. [创建账号](https://github.com/ruby-china/ruby-china/wiki/Ubuntu-12.04-%E4%B8%8A%E4%BD%BF%E7%94%A8-Nginx-Passenger-%E9%83%A8%E7%BD%B2-Ruby-on-Rails)

    useradd -m -s /bin/bash deploy #-m是创建home，-s是使用bash
    adduser deploy sudo            #加入sudo群组
    passwd deploy                  #密码

切换到deploy:

    su - deploy

查看.ssh文件夹，如果没有则创建

    mkdir .ssh
    cd .ssh
    cat > authorized_keys

使用

    [[ -f ~/.ssh/id_rsa.pub ]] && cat ~/.ssh/id_rsa.pub | pbcopy

将自己的pub key复制，写入authorized_keys

    ssh deploy@xxxxxxxx

Modify SSH settings /etc/ssh/sshd_config.

    PermitRootLogin no
    PasswordAuthentication no

then:

    reload ssh
    
[参考-服务器安全相关](https://www.linode.com/docs/security/securing-your-server/)

### 2. #安装所需的linux包

    sudo apt-get install build-essential bison openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev  libxml2-dev libxslt-dev autoconf libgmp-dev libc6-dev zlib1g-dev libssl-dev build-essential curl git-core libc6-dev g++ gcc libmysqlclient-dev nodejs 
    #更新
    sudo apt-get update

### 3. rvm install

    curl -L get.rvm.io | bash -s stable # will need add gpg --keyserver, it will show, add that and retype this commod 
    source /home/deploy/.rvm/scripts/rvm
    echo "gem: --no-ri --no-rdoc" > ~/.gemrc

### 4. ruby install

    rvm reload
    rvm install 2.1.3
    ruby -v

### 5. nginx install

    sudo apt-get install nginx

#### 5.1 [配置](http://blog.zlxstar.me/blog/2013/02/01/zai-rails-xiang-mu-zhong-ji-cheng-nginx-he-unicorn/)

    sudo mkdir /etc/nginx/conf.d
    sudo ln -s /home/deploy/app/current/config/nginx.conf /etc/nginx/conf.d/nginx.conf

并在 /etc/nginx/nginx.conf 里面 http 模块下面添加

    include conf.d/*.conf;

then:

    sudo service nginx start

or

    cd /usr/sbin
    sudo ./nginx

### 6. [config ssh to access to github](http://www.cyberciti.biz/faq/create-ssh-config-file-on-linux-unix/)

在~/.ssh/config

    Host *
      ForwardAgent yes
      User deploy

在remote server

    ssh -T git@github.com

### 7. 配置Project

[deploy and unicorn](https://www.digitalocean.com/community/tutorials/how-to-deploy-rails-apps-using-unicorn-and-nginx-on-centos-6-5)

> deploy.rb

    require 'mina/unicorn'
    require 'mina/bundler'
    require 'mina/rails'
    require 'mina/git'
    require 'mina/rvm' #rbenv
    require 'mina/whenever'
    require 'mina_sidekiq/tasks'

    set :domain, '119.254.102.200'
    set :branch, 'master'

    set :user, 'deploy'
    set :forward_agent, true
    set :port, 9527

    set :deploy_to, '/home/deploy/doubao'
    set :current_path, 'current'
    set :app_path,  "#{deploy_to}/#{current_path}"

    set :repository, 'git@gitlab.com:hunter/deploy.git'
    set :keep_releases, 20

    set :unicorn_pid, lambda { "#{deploy_to}/#{shared_path}/tmp/pids/unicorn.pid" }
    set :sidekiq_pid, lambda { "#{deploy_to}/#{shared_path}/tmp/pids/sidekiq.pid" }
    # set :unicorn_config, lambda { "#{app_path}/config/unicorn.rb" }

    set :shared_paths, [
      'config/database.yml',
      'config/secrets.yml',
      'config/application.yml',
      'tmp',
      'log'
    ]

    task :environment do
      queue! 'source ~/.bashrc'
      invoke :'rvm:use[ruby-2.1.3]'
    end

    task setup: :environment do
      queue! %[mkdir -p "#{deploy_to}/shared/log"]
      queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/log"]

      queue! %[mkdir -p "#{deploy_to}/shared/config"]
      queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/config"]

      queue! %[mkdir -p "#{deploy_to}/shared/tmp"]
      queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/tmp"]

      queue! %[mkdir -p "#{deploy_to}/shared/public/uploads"]
      queue! %[chmod g+rx,u+rwx "#{deploy_to}/shared/public/uploads"]

      queue! %[touch "#{deploy_to}/shared/config/database.yml"]
      queue! %[touch "#{deploy_to}/shared/config/secrets.yml"]
      queue! %[touch "#{deploy_to}/shared/config/application.yml"]
    end

    desc "Deploys the current version to the server."
    task deploy: :environment do
      deploy do
        invoke :'git:clone'
        invoke :'deploy:cleanup'
        invoke :'deploy:link_shared_paths'
        invoke :'bundle:install'
        invoke :'rails:db_migrate'
        invoke :'rails:assets_precompile'
        invoke :'whenever:update'

        to :launch do
          invoke :'unicorn:restart'
          queue "touch #{deploy_to}/tmp/restart.txt"
          invoke :'sidekiq:restart'
        end
      end
    end

设置 [nginx and unicorn](http://blog.zlxstar.me/blog/2013/02/01/zai-rails-xiang-mu-zhong-ji-cheng-nginx-he-unicorn/)

> nginx.conf

    # nginx config

    upstream unicorn {
      server unix:/tmp/unicorn.sock fail_timeout=0;
    }

    server {
      listen 80 default deferred;
      # server_name example.com;  #此处需要改动
      root /home/deploy/doubao/current/public;  #此处需要改动

      location ~ ^/assets/ {
        gzip_static on;
        expires max;
        add_header Cache-Control public;
      }

      try_files $uri/index.html $uri @unicorn;
      location @unicorn {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://unicorn;
      }

      error_page 500 502 503 504 /500.html;
      client_max_body_size 4G;
      keepalive_timeout 10;
    }

> unicorn.rb

    APP_DIR = File.expand_path("../../", __FILE__)

    preload_app true
    worker_processes 4
    working_directory APP_DIR

    listen "/tmp/unicorn.sock", backlog: 64
    timeout 30

    pid "#{APP_DIR}/tmp/pids/unicorn.pid"
    stderr_path "#{APP_DIR}/log/unicorn.stderr.log"
    stdout_path "#{APP_DIR}/log/unicorn.stdout.log"

    GC.respond_to?(:copy_on_write_friendly=) and
      GC.copy_on_write_friendly = true

    before_fork do |server, worker|
      defined?(ActiveRecord::Base) and
        ActiveRecord::Base.connection.disconnect!

      old_pid = "#{server.config[:pid]}.oldbin"

      if File.exists?(old_pid) && server.pid != old_pid
        begin
          sig = (worker.nr + 1) >= server.worker_processes ? :QUIT : :TTOU
          Process.kill(sig, File.read(old_pid).to_i)
        rescue Errno::ENOENT, Errno::ESRCH
          # someone else did our job for us
        end
      end
    end

    after_fork do |server, worker|
      defined?(ActiveRecord::Base) and
        ActiveRecord::Base.establish_connection
    end


    before_exec do |server| # fix hot restart Gemfile
      ENV["BUNDLE_GEMFILE"] = "#{APP_DIR}/Gemfile"
    end

### 8. 安装mysql
    
    sudo apt-get install mysql-server
    
#### 8.1 mysql not start(Can't connect to local MySQL server through socket '/tmp/mysql.sock)

    mysql.server start
    
sometimes 

    ln -s /var/mysql/mysql.sock /tmp/mysql.sock
    
#### 8.2 [数据库备份](http://leowenyang.github.io/database/2014/06/28/git-backup-mysql.html)

    git init --bare db_backup.git # 创建一个仓库

    git clone db_backup.git # clone a bare 

    # 导出mysql 数据库到这个目录下
    ......


    #init sql 
    git add README.md
    git commit -m "init sql file"
    git push origin master # or git push --set-upstream origin master

    # shell_remote 脚本定期操作如此命令
    cd db_backup;
    echo successful go db_backup files;
    pg_dump db_name > db_file;
    git add . ; git commit -m "update db_file" ; git push ;

    #remote crontab 
    crontab -e  # write a new crontab 

    * * * * * cd  /home/deploy && ./db.sh
    | | | | |_ 星期(0-6)
    | | | |___ 月份(1-12)
    | | |_____ 日期(1-31)
    | |_______ 小时(0-23)
    |_________ 分钟(0-59)


#### 8.3 本地git clone 数据库文件
  
    # 设置ssh config
    Host git-clone # or *
      ForwardAgent yes
      User deploy
      IdentityFile ~/.ssh/id_rsa
      port 9527

    # git clone from remote server 
    git clone deploy@remote_server:db_backup.git

> 本机设置shell脚本定时  lcal.init.shell

    git clone ssh://deploy@xxx:port/~/db_backup.git # or set ssh config

> 本地crontab
    cd db_backup; git pull

### 9. [商业部署参考](https://gorails.com/setup/ubuntu/14.10)

### 10. [unicorn 部署原理](https://raw.githubusercontent.com/ruby-china/unicorn/master/SIGNALS-zh-CN)

    USR2 - 重新启动，将会启动一个新的 Master 进程，当启动完成并且验证通过或，会发送 QUIT 到原始进程上面。

    QUIT - 正常关闭，关闭前会等待子进程完成进行中请求。

    HUP - 重新载入配置文件，并且完整的重启所有子进程。

> shell 

      # 检测是否存在unicorn.pid文件，如果存在则检测是否有这个进程
      # 如果有这个进程，发送 USR2，重新启动，将会启动一个新的 Master 进程
      if [ -e /home/deploy/geek-lab/shared/tmp/pids/unicorn.pid ] && 
        kill -0 `cat /home/deploy/geek-lab/shared/tmp/pids/unicorn.pid` > /dev/null 2>&1; then
        echo "-----> Duplicating Unicorn...";
        kill -s USR2 `cat /home/deploy/geek-lab/shared/tmp/pids/unicorn.pid`;
      else
        # 删除没用的unicorn.pid 文件
        if [ -e "/home/deploy/geek-lab/shared/tmp/pids/unicorn.pid" ]; then
          if kill -0 `cat /home/deploy/geek-lab/shared/tmp/pids/unicorn.pid` > /dev/null 2>&1; then
            echo "-----> Unicorn is already running!";
            exit 0;
          fi;

          rm /home/deploy/geek-lab/shared/tmp/pids/unicorn.pid;
        fi;

      echo "-----> Starting Unicorn...";
      cd "/home/deploy/geek-lab/current" && BUNDLE_GEMFILE=/home/deploy/geek-lab/current/Gemfile RAILS_ENV="production" bundle exec unicorn -c /home/deploy/geek-lab/current/config/unicorn_master.rb -E production -D;

      fi;

      sleep 2; # in order to wait for the (old) pidfile to show up
      
      # 关闭旧的进程, 其实这个倒多余了
      if [ -e /home/deploy/geek-lab/shared/tmp/pids/unicorn.pid.oldbin ] && kill -0 `cat /home/deploy/geek-lab/shared/tmp/pids/unicorn.pid.oldbin` > /dev/null 2>&1; then
        kill -s QUIT `cat /home/deploy/geek-lab/shared/tmp/pids/unicorn.pid.oldbin`;
      fi;

> 备注：
    
    unicorn master (old)
     \_ unicorn worker[0]
     \_ unicorn worker[3]
     \_ unicorn master
        \_ unicorn worker[0]
        \_ unicorn worker[3]


    If everything seems ok, then send QUIT to the old master.  You're done!

    If something is broken, then send HUP to the old master to reload the config and restart its workers.  Then send QUIT to the new master
     process

    

  

