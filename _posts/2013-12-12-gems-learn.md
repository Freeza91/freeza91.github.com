---
layout: post
title: Gems_Learn
tag: Gems
category: Learn
---
gems list

![image](http://l.ruby-china.org/photo/2013/129c58d301965734647dd67c69039634.jpg)


1. [官方Ruby-Tool](https://www.ruby-toolbox.com/)

2. [常用gem列表1](http://lilulife.com/recommends/)

3. [常用gem列表2](http://ruby-china.org/wiki/gems)


### 常见的gem列表
```ruby
    gem "devise"
    gem 'omniauth'  #系列版本
    gem "cancan"
    gem "simple_form", "~> 3.0.0.rc"
    gem 'bootstrap-sass', '2.3.2.0'
    gem 'will_paginate'
    gem 'bootstrap-will_paginate'
    gem "cells"
    gem 'carrierwave'
    gem 'mini_magick'
    gem "jquery-fileupload-rails"
    gem 'qiniu-rs'
    gem 'rails_admin'  #依赖分页
    gem "Whenever"
    gem "Resque or Delayed_job"
    gem "rails-dev-boost"
    gem "ClientSideValidations"
    gem "Seed Fu"
    gem "Better Errors"

```

### 1. [devise](https://github.com/plataformatec/devise#the-devise-wiki)
    gem "devise", "~> 3.2.2"
安装：

    rails g devise:install #之后会生成两个文件，并在控制台中会有提示，将一些属性设置好。
配置Model:

    gem g devise model #会产生对应model文件，可以在对应的db文件中修改生成的值
    rake g migation add_conculm_to_model conclum:type
    rake db:migrate    # 生成数据库
一些常用方法：

    before_filter :authenticate_user!

    user_signed_in?
    current_user
    user_session
一些方法可以覆写,用来调控：

覆写方法

    after_sign_in_path_for
    after_sign_out_path_for
强化参数：

    class ApplicationController < ActionController::Base
        before_filter :configure_permitted_parameters, if: :devise_controller?
在原始基础上添加：

    protected
        def configure_permitted_parameters
            devise_parameter_sanitizer.for(:sign_up) << :username
        end
全部重新定义：

    protected
        def configure_permitted_parameters
           devise_parameter_sanitizer.for(:sign_in) { |u| u.permit(:username, :email) }
        end
#### 配置view
    gem g devise:views
        confirmations #邮件重新发送
        mailer:
            1. 注册确认提醒
            2. 重设密码验证
            3. 锁定提醒
        passwords：
            1. 邮件密码更新
            2. 忘记密码，发送邮件
        registertion:
            1. 注册
            2. 更新注册信息
        sessions #登录
        shared #提示
        unlocks #锁定

备注：

> 可以直接rails g devise:views model

#### routes控制

初级定制：

```ruby
devise_for :users, :path => "auth",
:path_names => { :sign_in => 'login', :sign_out => 'logout',
                 :password => 'secret', :confirmation => 'verification',
                 :unlock => 'unblock', :registration => 'register',
                 :sign_up => 'cmon_let_me_in'
                }
```

深度定制：

```ruby
devise_scope :user do
    get "sign_in", :to => "devise/sessions#new" #is sign_in not users/sign_in
    .....
end
```

#### 设置提醒信息

利用I18n,进行定义即可

### 2. [CanCan](https://github.com/ryanb/cancan)

#### 安装：

    gem "cancan", "~> 1.6.10"
    rails g cancan:ability #ability.rb
    class Ability
        include CanCan::Ability
        def initialize(user)
            user ||= User.new # guest user
            if user.role? :admin
                can :manage, :all
            else
                can :read, :all
                can :create, Comment
                can :update, Comment do |comment|
                  comment.try(:user) == user || user.role?(:moderator)
                end
            end
            if user.role?(:author)
                can :create, Article
                can :update, Article do |article|
                    article.try(:user) == user
                end
            end
        end
    end

备注：

##### can action Model

    can :manage, Article  # user can perform any action on the article
    can :read, :all       # user can read any object
    can :manage, :all     # user can perform any action on any object

    can [:update, :destroy], [Article, Comment]

    alias_action :create, :read, :update, :destroy, :to => :crud
    can :crud, User


#### 是否可行 & 可行性验证:

可行处理：

    <% if can? :update, @article %>
        <%= link_to "Edit", edit_article_path(@article) %>
    <% end %>

可行性验证：

    # def show
    #   @article = Article.find(params[:id])
    #   authorize! :read, Article
    # end
    # if use load_and_authorize_resource, the same as above
    # def show
    #    # @article is already loaded and authorized
    # end
    load_and_authorize_resource # load auto and authorize!
    load_resource
    authorize_resource

    # authorize!(params[:action], @product || Product)
    authorize! :action, @resource  #这个也进行了页面跳转控制
    skip_authorize_resource :only => :new  #

    load_resource :find_by => :permalink
    load_and_authorize_resource :only => [:index, :show]

    authorize! :read, Article, :message => "Unable to read this article."

#### 异常处理：

    class ApplicationController < ActionController::Base
        rescue_from CanCan::AccessDenied do |exception|
        render :file => "#{Rails.root}/public/403.html", :status => 403, :layout => false
            ## to avoid deprecation warnings with Rails 3.2.x (and incidentally using Ruby 1.9.3 hash syntax)
            ## this render call should be:
            # render file: "#{Rails.root}/public/403", formats: [:html], status: 403, layout: false
        end
    end


    class ApplicationController < ActionController::Base
        rescue_from CanCan::AccessDenied do |exception|
            redirect_to main_app.root_url, :alert => exception.message
        end
    end
    exception.action # => :read
    exception.subject # => Article
    exception.default_message = "Default error message"  #can modify by locals
    exception.message # => "Default error message"

#### 全局验证：

    class ApplicationController < ActionController::Base
        check_authorization
    end

### 3. 七牛云存储-[qiniu-rs](http://docs.qiniutek.com/v2/sdk/ruby/)(可以有版本选择！！)

#### 3.1 配置：

在初始化配置中进行这样的设置-YOUR_RAILS_APP/config/initializers/qiniu-rs.rb：

    Qiniu::RS.establish_connection! :access_key => YOUR_APP_ACCESS_KEY,
                                :secret_key => YOUR_APP_SECRET_KEY

#### 3.2 客户端上传：

>3.2.1 旧方法

    Qiniu::RS.put_auth(expires_in = nil, callback_url = nil)
    expires_in: 可选，整型，用于设置上传 URL 的有效期，单位：秒，缺省为 3600 秒
    callback_url:可选，字符串类型(String)，用于设置文件往这个 URL 上传成功后，七牛云存储服务端要回调客户方的业务服务器地址。

备注：

此方法仅仅是简单的客户端上传！

>3.2.2 新方法：

    	Qiniu::RS.generate_upload_token :scope      => 'bokmarket',
                                :expires_in         => 3600,
                                :callback_url       => 'http://localhost:3000/callback',
                                :customer           => '123',
								:callback_body			=> '{"save": "save"}',
								:callback_body_type => 'application/json',
                                :escape             => 0,
                                :async_options      => async_callback_api_commands,
								:return_url			=> "http://localhost:3000/sucssful"
                                :return_body        =>'{"foo": "bar"}'
        配合：
        <form method="post" action="http://up.qiniu.com/" enctype="multipart/form-data">
          <input name="x:location" type="hidden" value="beijing">
          <input name="key" type="hidden" value="test.zip">   <!-- 可书写保存的文件名-->
          <input name="token" type="hidden" value="<%= @uploadToken %>">
          <input name="file" type="file"/>
          <input type="submit" value="Upload File" />
        </form>

备注：

这个方法的功能点比较多：

1. 在生成opload_token的时候设置了很多属性，其中比较不错的就是先上传uploadtoken(并携带很多自定义数据).
保存后在向自己的服务器发送post(return_url)，之后服务器在向云存储期返回自定义格式数据，在通过callback_url(返回到客户端)。
2. 在具体的表单提交过程中，可以定义保存的文件名，一些自定义的参数。


#### 3.3 客户端下载

    hash = Qiniu::RS.get(bucket, filename, save_as = downloadedfilename, expires_in = 3600, version = nil)
    返回值：
    {
     "fsize"    => 3053,
    "hash"     => "Fu9lBSwQKbWNlBLActdx8-toAajv",
    "mimeType" => "application/x-ruby",
    "url"      => "http://iovip.qbox.me/file/<an-authorized-token>", #下载链接
    "expires"  => 3600
    }

    只获取下载链接：
    download_url = Qiniu::RS.download(bucket, key, save_as = nil, expires_in = nil, version = nil)


#### 4 其他

还涉及到一些批量操作和云端处理（尤其是对视频和图片的处理操作）
[参考](https://github.com/qiniu/ruby-sdk)

### 4. [第三方登录](https://github.com/intridea/omniauth) [Wiki](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview)

#### 4.1 github

基本上是在使用内置的devise:

    rails g migration AddColumnsToUsers provider uid name
    rake db:migrate

在配置中添加devise.rb：

    config.omniauth :github, "APP_ID", "APP_SECRET"

增加并自定义callback函数：

    devise :omniauthable
    devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }

>备注：

在app/controllers/users/omniauth_callbacks_controller.rb下添加：

    class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
        ......
    end

回调认证方法可参考[Wiki](https://github.com/plataformatec/devise/wiki/OmniAuth:-Overview)

对一些默认的设置：

    devise_scope :user do
        get 'sign_in', :to => 'devise/sessions#new', :as => :new_user_session
        get 'sign_out', :to => 'devise/sessions#destroy', :as => :destroy_user_session
    end


### 5. [simple_form](https://github.com/plataformatec/simple_form) [wiki](http://blog.plataformatec.com.br/2012/02/simpleform-2-0-bootstrap-for-you-with-love/)

#### 5.1 安装：

    gem "simple_form", "~> 3.0.0.rc"
    gem 'bootstrap-sass', '2.3.2.0'

>出现过因为版本问题导致无法正常使用，还是按照版本来操作。

    rails generate simple_form:install --bootstrap


#### 5.2 example

    <%= simple_form_for @user do |f| %>
        <%= f.input :username, label: 'Your username please' %>
        <%= f.input :password, hint: 'No special characters.' %>
        <%= f.input :email, placeholder: 'user@domain.com' %>
        <%= f.input :remember_me, inline_label: 'Yes, remember me' %>
        <%= f.button :submit %>
    <% end %>

>统一的样式可参考[ruby-china](https://github.com/ruby-china/ruby-china/blob/master/config/initializers/simple_form.rb)


### 6. [Cells](https://github.com/apotonick/cells)

#### 6.1 安装：

    rails generate cell cart show -e haml  # -e 之后是表示生成的格式，默认情况下是erb（删除-e haml）

        create  app/cells/
        create  app/cells/cart
        create  app/cells/cart_cell.rb
        create  app/cells/cart/show.html.haml
        create  test/cells/cart_test.rb

#### 6.2 前台控制和后台数据提取：

    <%= render_cell :cart, :show, :user => @current_user %>

在app/cells/cart_cell.rb中：

    class CartCell < Cell::Rails
        def show(args)
        user    = args[:user]    #获取自定义的参数
        @items  = user.items_in_cart

        render  # renders show.html.haml
      end
    end

#### 6.3 caching

    class CartCell < Cell::Rails
      cache :show, :expires_in => 10.minutes

#### 6.4 其他

可在controller，view中添加使用
参考：[How Cells Improves your Rails Architecture](http://nicksda.apotomo.de/2010/10/10-points-how-cells-improves-your-rails-architecture/)

### 7. [Ajax](http://ihower.tw/rails3/assets-and-ajax.html)

    remote:ture
        form_for
        link_to
        button_to
        .....

#### 7.1 使用

    <%= javascript_include_tag "jquery", "jquery_ujs" %>   #一定得写上，之前出各种问题！！

申明：

    remote: true
    "data-type" => "script"  or :format => :js......

[ajax的格式](http://www.alfajango.com/blog/rails-3-remote-links-and-forms-data-type-with-jquery/)

#### 7.2 后台返回值

    respond_to do |format|
        format.js {render xxxx}
        format.html
        format.xml
        format.json
        format.text
    end

#### 7.3 创建后台返回的文件

需要创建当前方法名 for example: create.js.erb
这里使用频繁的一个方法：

    <%= escape_javascript(xxxxx) %>


### 8. [上传文件](https://github.com/blueimp/jQuery-File-Upload/wiki/Rails-setup-for-V6)

上传文件主要采用的就是插件形式，可以上传任何形式的文件，同时也能自定义上传文件。

#### 8.1 配置

    gem 'carrierwave'
    gem "jquery-fileupload-rails"

    gem 'mini_magick' #如果是图片的话，使用这个更好，同默认的里面会有代码注释

#### 8.2 [生成](https://github.com/carrierwaveuploader/carrierwave)  [demo](https://github.com/Phifo/jquery-fileupload-rails-carrierwave)

>备注： 这个过程需要把握好其中的逻辑关系，需要has_many, belongs_to

>在生成的文件中，可以设置路径，文件名和上传白名单等等的事情。



#### 8.3 代码初始化

在js和css中设置下：

    //= require jquery-fileupload
    *= require jquery.fileupload-ui

在model中设置：

    mount_uploader :avatar, DocumentUploader

    def to_jq_upload
        {
        "name" => read_attribute(:avatar),
         "size" => avatar.size,
         "url" => avatar.url,
         "delete_url" => picture_path(:id => id),
         "delete_type" => "DELETE"
        }
   end

在controller中设置：

     respond_to do |format|
      if @document.save
        format.html {
          render :json => [@document.to_jq_upload].to_json,
          :content_type => 'text/html',
          :layout => false
        }
        format.json { render json: {files: [@document.to_jq_upload]}, status: :created, location: @document }
      else
        format.html { render action: "new" }
        format.json { render json: @document.errors, status: :unprocessable_entity }
      end
    end

#### 8.4 创建[view](https://github.com/Useyes/gems/blob/master/app/views/document/new.html.erb)

> 备注：此过程中有一些是自己改动过的，bootstrap版本不一样问题导致显示样式和结果不同，需要特别注意。同时这个代码需分开放置；在上传文件格式的设置上，可以自定义，也可以使用默认的设置。[demo参考](http://blueimp.github.io/jQuery-File-Upload/)

##### 8.5 上传到云服务器

### 9. [后台管理](https://github.com/sferik/rails_admin)

#### 9.1 安装

安装的时候需要依赖下面的几个

    gem 'rails_admin'
    gem "activesupport"
    gem "will-paginate" #但是它版本过低，有些方法不适用了。

于是在初始化代码中加入[refer](https://github.com/mislav/will_paginate/issues/174)：

    # config/initializers/will_paginate.rb
    if defined?(WillPaginate)
        module WillPaginate
          module ActiveRecord
            module RelationMethods
              alias_method :per, :per_page
              alias_method :num_pages, :total_pages
            end
           end
        end
    end

或者改用另一个分页技巧：[Kaminari](https://github.com/amatsuda/kaminari)

#### 9.2 使用

    rails g rails_admin:install

它会提示，是否是新建一个用户管理员还是在已有的基础上管理
同时也可以配合cancan这类的管理工具
因为它依赖在devise中，所以它也需要一些认证。同时也需要 devise_for :admins

之后在利用：

    rake db:migrate

后台还是挺强大的！

#### 9.3 [定制](https://github.com/sferik/rails_admin/wiki)
