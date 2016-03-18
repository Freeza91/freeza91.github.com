---

layout: post
title: omniauth-login
category : [Login, Register, Omniauth, Omniauth-identity]
tags : [Login, Register, Omniauth, Omniauth-identity]

---
统一网站常用注册/登陆和第三方登陆/注册.


### 1. 使用场景

当今第三方登陆注册平台很多，类似qq、wechat、微博这些拥有足够多的群体，自己完全重写其实稍微有点多余。那么以后所有的注册登陆注册似乎是完全可以直接使用这些即：omniauth-xxx来完成。

当然如果有必要的话还是需要写一些站内登陆。

那么站内登陆+第三方登陆的数据库设计以及相互之间的逻辑验证就显得特别混乱。那么有一种方式将站内登陆也做成一种“第三方登陆”，那么采取统一的方式来实现，就会显得特别有意义。


### 2. 常规登陆方式

~~~ ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :github, Settings.github_key, Settings.github_secret
end

~~~

#### 2.1 数据库设计

~~~ ruby 
create_table "authorizations", force: :cascade do |t|
    t.integer  "user_id",    limit: 4
    t.string   "uid",        limit: 255
    t.string   "provider",   limit: 255
    t.datetime "created_at",             null: false
    t.datetime "updated_at",             null: false
end

create_table "users", force: :cascade do |t|
    t.string   "email",           limit: 255
    t.string   "name",            limit: 255
    t.string   "password_digest", limit: 255
    t.datetime "created_at",                  null: false
    t.datetime "updated_at",                  null: false
end

add_index "users", ["email"], name: "index_users_on_email", unique: true, using: :btree

~~~

#### 2.2 models

~~~ ruby 
# user 
class User < ActiveRecord::Base

  has_secure_password
  has_many :authorizations
  validates :password, presence: true, allow_blank: false
  
  def authenticate(params_password)
    # super 是调用父类中的方法，而非是直接获得父类
    password_digest.present? && super(params_password)
  end
  def self.create_with_omniauth(auth_hash)
    # 如果没有返回邮箱，而又必须有邮箱作为唯一验证，随机一个！
    u = User.new(email: "#{Time.now.to_i}#{Random.rand(20)}@example.com", name: auth_hash["info"]["name"])
    u.save!(validate: false) ? u : nil
  end
end

# authorization
class Authorization < ActiveRecord::Base
  belongs_to :user
  def self.find_by_auth(auth_hash)
    find_by_provider_and_uid(auth_hash['provider'], auth_hash['uid']) ||
      create_with_omniauth(auth_hash)
  end
  def self.create_with_omniauth(auth_hash)
    u = User.create_with_omniauth(auth_hash)
    u && create! do |auth|
      auth.provider = auth_hash['provider']
      auth.uid = auth_hash['uid']
      auth.user_id = u.id
    end
  end
end

~~~

#### 2.3 controllers

~~~ ruby 

class SessionsController < ApplicationController
  def new
    @user = User.new
  end
  
  def create
  end
  
  def authenticate
    auth = Authorization.find_by_auth(auth_params)
    if auth
      user = auth.user
      render text: '登陆成功'
    else
      render text: '登陆失败'
    end
  end
  
private
  def auth_params
    auth = request.env["omniauth.auth"]
  end
  def user_params
    params.reuqire(:user).permit(:password, :email)
  end
end

~~~

备注：

> 普通的站内登陆的，是需要将email作为唯一性的索性，故在users表中不可出现重复邮箱！在某些第三方登陆后，不会返回给用户邮箱，或者返回邮箱是曾经已经在users表中存在。这样就无法创建user。只能自己随机一个邮箱创建user！ 此时users表中的password_digest是nil.

>  在自己站内登陆的时候需要进行password的authentic。但是因为有涉及到因为第三方登陆导致的password_digest为nil的情况，故认证之前还需要预先判断。


###  3 使用omniauth-identity
 
#### 3.1 配置
 
 因为[omniauth-identity](https://github.com/intridea/omniauth-identity)没法做登陆时验证码校验，故自己修改后[omniauth-identity](https://github.com/Freeza91/omniauth-identity)。
 
~~~ ruby 
 Rails.application.config.middleware.use OmniAuth::Builder do
  provider :identity, on_failed_registration: lambda { |env|
      IdentitiesController.action(:new).call(env) },
      on_validation: lambda { |env|
        Captcha.valid_captcha?(env)
      }
end


class Captcha

  class << self
    def valid_captcha?(env)
      env = env[:env]
      session = env["rack.session"]
      params = env["rack.request.form_hash"]
      unless verify_rucaptcha?(params['captcha'].downcase, session['captcha'].downcase)
        env["omniauth.identity"].errors.add(:base, '验证码错误')
        session['captcha'] = nil
        return false
      end

      true
    end

    def random_text(n = 4)
      ['a'..'z', 'A'..'Z', 0..9].map(&:to_a).flatten.shuffle[0, n].join
    end

    def create(code)
      command = <<-CODE
        convert -size 100x28 -fill black -background white \
        -draw 'stroke black line #{rand(20)},#{rand(28)} #{rand(30)+50},#{rand(28)}' \
        -draw 'stroke black line #{rand(50)},#{rand(28)} #{rand(100)},#{rand(28)}' \
        -wave #{2+rand(2)}x#{50+rand(20)} \
        -font '#{Rails.root}/public/fonts/segoepr.ttf' \
        -gravity Center -sketch 3x1+#{rand(180)} -pointsize 22 -implode 0.2 label:#{code} png:-
      CODE

      sub = Subexec.run(command)
      sub.output
    end

    def verify_rucaptcha?(a, b)
      a.present? && a == b
    end

  end

end

~~~

#### 3.2 DB
 
~~~ ruby 
 
 ActiveRecord::Schema.define(version: 20160317122458) do
  create_table "authorizations", force: :cascade do |t|
    t.integer  "user_id",    limit: 4
    t.string   "uid",        limit: 255
    t.string   "provider",   limit: 255
    t.datetime "created_at",             null: false
    t.datetime "updated_at",             null: false
  end
  create_table "identities", force: :cascade do |t|
    t.string   "email",           limit: 255
    t.string   "password_digest", limit: 255
    t.string   "name",            limit: 255
    t.datetime "created_at",                  null: false
    t.datetime "updated_at",                  null: false
  end
  create_table "users", force: :cascade do |t|
    t.string   "email",      limit: 255
    t.string   "name",       limit: 255
    t.datetime "created_at",             null: false
    t.datetime "updated_at",             null: false
  end
end

~~~ 

#### 3.3 models

~~~ ruby 
## 其他同上

### 此处特别注意，不是原生的ActiveRecord::Base，是OmniAuth::Identity::Models::ActiveRecord
class Identity < OmniAuth::Identity::Models::ActiveRecord
end

~~~

#### 3.4 views && routes

~~~ ruby 

resources :identities
resources :users
resources :sessions, only: :new
match '/auth/:provider/callback', to: 'sessions#create', via: [:get, :post]
get '/captcha', to: 'captchas#show'


# sessions/ new.html.erb

<p>
  <strong>Don't use these services?</strong>
  <%= link_to "Create an account", new_identity_path %> or login below.
</p>
<%= form_tag "/auth/identity/callback" do %>
  <div class="field">
    <%= label_tag :auth_key, "Email" %><br>
    <%= text_field_tag :auth_key %>
  </div>
  <div class="field">
    <%= label_tag :password %><br>
    <%= password_field_tag :password %>
  </div>
  <div class="actions"><%= submit_tag "Login" %></div>
<% end %>

# identities / new.html.erb

<%= form_tag "/auth/identity/register" do %>
  <% if @identity && @identity.errors.any? %>
    <div class="error_messages">
      <h2><%= pluralize(@identity.errors.count, "error") %> prohibited this account from being saved:</h2>
      <ul>
      <% @identity.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>
  <div class="field">
    <%= label_tag :name %><br>
    <%= text_field_tag :name, @identity.try(:name) %>
  </div>
  <div class="field">
    <%= label_tag :email %><br>
    <%= text_field_tag :email, @identity.try(:email) %>
  </div>
  <div class="field">
    <%= label_tag :password %><br>
    <%= password_field_tag :password %>
  </div>
  <div class="field">
    <img src='/captcha'/>
    <%= text_field_tag :captcha %>
  </div>
  <div class="actions"><%= submit_tag "Register" %></div>
<% end %>
  
~~~ 