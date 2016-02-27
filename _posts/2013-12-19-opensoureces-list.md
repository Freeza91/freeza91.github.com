---
layout: post
title: OpenSources-List
tag: OpenSources
category: learn
---
开源项目list 

### [开源项目](http://www.opensourcerails.com/)

下面将会列举出一些开源项目和github地址，并挨个分析

#### 1. github: https://github.com/malclocke/fulcrum

> [Restful](http://ihower.tw/rails3/restful.html)

    resouces :items
    GET    /items        #=> index
    GET    /items/1      #=> show
    GET    /items/new    #=> new
    GET    /items/1/edit #=> edit
    PUT    /items/1      #=> update
    POST   /items        #=> create
    DELETE /items/1      #=> destroy

    match "user/:user_id" => "user/articles#index", :as => :user #隐藏用户的id等敏感信息


备注:

>根据不同的设计提交请求，它自动的载入相对应的方法

    items_path => index
    item_path(@item) => show
    若item_path(@item, :params => "something")   则会出现  http:url?params=something,要是加上#的话则就能直接进行页面页面内部跳转
    new_item_path => new
    edit_item_path(@item) => update
    {item_path(@item), :method => 'delte', :confrim =>"something"} => destory
    在进行这个之前，先通过new_item_path得到@item = Item.new,会自动render下面这个
    form_for @item do |f| ... end
    在提交后，它会自动匹配create（获取id），要是失败在render 'new'

> [Helper方法](http://ihower.tw/rails3/actionview-helpers.html)

>>1 用于帮助controller处理

>>2 Helper-View

    content_for, content_tag....

>>> 利用这个可以更精细化局部布局:

    #layout
    <%= yield :title%>
    <%= yield %>
    #html
    <%content_for :title do %>
        <%= render ...... %>
    <% end %>

>>> 这类方法可以在视图显示的时候起很大作用！处理显示的情况

> [rails异常](http://zhangcaiyanbeyond.iteye.com/blog/976762)

    rescue_from ActiveRecord::RecordNotFound, :with => :render_404
    def render_404
    respond_to do |format|
      format.html do
        render :file => Rails.root.join('public', '404.html'),
          :status => '404'
      end
      format.xml do
        render :nothing => true, :status => '404'
      end
    end
  end

备注：

>所有异常比如权限控制异常啥的，根据当时使用的gem或者在controller中便可定义

更常见的异常可如此处理，application.rb中定义：

    rescue_from Exception, with: :render_execption
    rescue_from ActionController::RoutingError, with: :render_not_found
    rescue_from ActionController::UnknownController, with: :render_not_found
    rescue_from ActionController::UnknownAction, with: :render_not_found
    rescue_from Mongoid::Errors::DocumentNotFound, with: :render_not_found

    def render_not_found
        render template: "/errors/404", status: :not_found, layout: 'error'
    end

    def render_execption
        render template: "/errors/500", status: :not_found, layout: 'error'
    end

     protected
         def not_found
        	raise ActionController::RoutingError.new('Not Found')
    	end
    end



#### 2. github: https://github.com/hooopo/rubyist

> 本地化

>> 在application.rb中加入如下代码：

    config.time_zone = 'Beijing'
    config.i18n.default_locale = "zh-CN"

>> 系统默认的会自动成汉字提醒，要是在额外加单独插件的话，需要自己手动在加入：

    devise.yml  => devise.zh-CN.yml

> [数据库查询优化](http://ihower.tw/rails3/performance.html)

    belongs_to :user :counter_cache = > true

    Aritcle.includes(:user)

    Event.select([:id, :title, :description]).limit(10) #仅仅提取用到的信息

    使用scope来进行更清晰化的自定义查询：

    scope :short, :select => "id, name, email" ====   user.short.limit(10)
    scope :visiable_by_all, where("visiable is ?", true) ==== user.visiable_by_all
    scope :hottst, order("ranking DESC")  === user.hottst

> 过滤危险字符

    content.html_safe


