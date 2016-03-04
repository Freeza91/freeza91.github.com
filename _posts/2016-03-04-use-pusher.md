---

layout: post
title: Pusher
category : [Pusher, Websocket, Gon]
tags : [Pusher, Websocket, Gon]

---
pusher(websocket第三方平台) 使用

### 1. [pusher](https://github.com/pusher/pusher-http-ruby)

pusher 是在看[peatio](https://github.com/peatio/peatio)源码的时候接触到的一个第三方websocket中转服务商，避免了自己单独搭建websocket服务器的概念和原理都很不错,是一个很不错的平台。

### 2. 使用
##### 2.1 创建 pusher.rb --server 端

~~~ ruby 
require 'pusher'
Pusher.url = Settings.pusher_url
pusher_url = "https://#{key}:#{secret}@api.pusherapp.com/apps/#{app_id}"

# or 
Pusher.app_id = 'your-pusher-app-id'
Pusher.key = 'your-pusher-key'
Pusher.secret = 'your-pusher-secret'
Pusher.encrypted = true
~~~ 

##### 2.2 创建channel & event

~~~ html
<!DOCTYPE html>
<head>
  <title>Pusher Test</title>
  <script src="https://js.pusher.com/3.0/pusher.min.js"></script>
  <script>
    // Enable pusher logging - don't include this in production
    Pusher.log = function(message) {
      if (window.console && window.console.log) {
        window.console.log(message);
      }
    };

    var pusher = new Pusher('5ed6030e7452e4b57957', {
      encrypted: true
    });

    var channel = pusher.subscribe('test_channel');
    channel.bind('my_event', function(data) {
      alert(data.message);
    });
  </script>
</head>
~~~

##### 2.3 push message

~~~ ruby
pusher_client.trigger('test_channel', 'my_event', {
  message: 'hello world'
})
~~~

### 3 技巧

#### 3.1 pusher channel 的类型

##### 3.1.1 [public](https://pusher.com/docs/client_api_guide/client_public_channels)
对所有人开放，即不需要认证。
##### 3.1.2 [private](https://pusher.com/docs/client_api_guide/client_private_channels) 
对socket需要认证。创建的channel名字必须是"private-"开头
##### 3.1.3 [presence](https://pusher.com/docs/client_api_guide/client_presence_channels)
在私有的基础上面，可以获取到在某个channel上面的所有在线用户的相关信息(members.count、members.each、members.get(userId)、members.me等等)。创建的channel名字必须是"presence-"开头。

备注：
[认证相关](https://pusher.com/docs/authenticating_users)

#### 3.2 定制socket方式

websocket 有emit/send/broadcast三种方式

``` ruby
### send
Pusher.trigger('pay_channel', 'pay_notify', { message: 'pay success!!' })
```

``` html
### broadcast

var pusher = new Pusher('5ed6030e7452e4b57957', {
      encrypted: true
  });

var socketId = null;

pusher.connection.bind('connected', function() {
    socketId = pusher.connection.socket_id;
    
    var channel = pusher.subscribe('pay_channel' + socketId);
    channel.bind('pay_notify', function(data) {
      alert(data.message);
    });
    console.log(socketId);
}); 

#此处的socket_id 可有client通过ajax来获得。发给除了自己以外的所有人
Pusher.trigger('pay_channel186586.403132', 'pay_notify', { message: 'pay success ???? ' }, { socket_id: '186674.374456' })
```
特别说明如果想要push到单独某个client的时候可以采用如下方式(参考peatio)：

> 1. [gon](https://github.com/gazay/gon),在网页加载之前生成一个唯一的channel，单独为某个用户开设，binding的event还是之前的逻辑。

> 2. 在client绑定这个通过gon获取来的private-channel-id,如果有需要并完成auth。

> 3. 服务器push 参数

``` ruby
Pusher.trigger(xxxxxx)

# or using workers，可使用如下方法
Pusher.get_async
Pusher.post_async
Pusher.trigger_async
```