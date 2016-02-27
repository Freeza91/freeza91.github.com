---

layout: post
title: Wechats rewards and ngrok docker
category : [Wechats, ngrok, hongbao, docker]
tags : [Wechats, ngrok, rewards, docker]

---

微信开发过程

### 1. [wechats](https://github.com/Eric-Guo/wechat) gem 

### 2. ngrok and docker

开发微信调试相对不方便，ngrok是一个很好的转发工具，but后面它被墙了。

还好可以它有可以搭建在服务器上面的功能，使用docker完美简单的解决这个问题。

[ngrokd-refer1](http://blog.neofung.org/%E6%9C%AC%E5%9C%B0dockers%E9%83%A8%E7%BD%B2ngrokd%E6%9C%8D%E5%8A%A1/) [ngrokd-refer2](http://blog.sequenceiq.com/blog/2014/10/09/ngrok-docker/)


    # 服务器搭建
    docker run -d --name ngrokd \
      -p 80:80 \
      -p 443:443 \
      -p 4443:4443 \
      sequenceiq/ngrokd \
        -httpAddr=:80 \
        -httpsAddr=:443 \
        -domain=ngrok.yourdomain.com
        
    # 客户端下载
    curl -o /usr/local/bin/ngrok https://s3-eu-west-1.amazonaws.com/sequenceiq/ngrok_linux
    chmod +x /usr/local/bin/ngrok
    
    # 配置文件
    cat > ~/.ngrok <<EOF
    server_addr: ngrok.yourdomain.com:4443
    trust_host_root_certs: false
    EOF
    
    # 本地使用
    ngrok 8000

### 3. 微信红包

#### 3.1 下载相关的认证文件

#### 3.2 配置微信相关文件

#### 3.3 发送微信红包请求

> 3.3.1 构建请求参数

    def build_params(openid, amount, num)
        options = {
          nonce_str: nonce_str ,#随机字符串
          mch_billno: order_num, # 商户订单号
          mch_id: Settings.wechats_mch_id,#商户编号
          wxappid: Settings.wechats_appid, # 公众账号appid
          send_name: '极客实验室', # 提供方名称
          re_openid: openid, # 用户openid
          total_amount: amount, # 付款金额
          total_num: num, # 红包发放总人数
          wishing: '现金红包奖励', # 红包祝福语
          client_ip: Settings.domain_ip, # Ip地址
          act_name: '现金红包', # 活动名称
          remark: '现金红包奖励' # 备注
        }
        options[:sign] = md5_with_partner_key(options)

        options
    end

    def nonce_str
      SecureRandom.hex
    end

    def order_num
      Settings.wechats_mch_id.to_s + Time.now.strftime('%Y%-m%-d') + Time.now.to_i.to_s
    end

    def md5_with_partner_key(params)
      str = params.sort.map { |item| item.join('=') }.join('&')
      str << "&key=#{Settings.wechats_pay_secret}"
      Digest::MD5.hexdigest(str).upcase
    end


> 3.3.2 读取认证文件

    def build_cert_http
      cert = File.read("#{ Rails.root }/cert/apiclient_cert.pem")
      key = File.read("#{ Rails.root }/cert/apiclient_key.pem")
      @uri = URI.parse URL
      http = Net::HTTP.new(@uri.host, @uri.port)
      http.use_ssl = true if @uri.scheme == 'https'
      http.cert = OpenSSL::X509::Certificate.new(cert)
      http.key = OpenSSL::PKey::RSA.new(key, '商户编号')
      http.ca_file = File.join("#{ Rails.root}/cert/rootca.pem")
      http.verify_mode = OpenSSL::SSL::VERIFY_PEER
      
      http
    end


> 3.3.3 构建返回参数

    def build_xml_body(options)
        xml = <<-XML.strip_heredoc
          <xml>
          <sign><![CDATA[#{options[:sign]}]]></sign>
          <mch_billno><![CDATA[#{options[:mch_billno]}]]></mch_billno>
          <mch_id><![CDATA[#{options[:mch_id]}]]></mch_id>
          <wxappid><![CDATA[#{options[:wxappid]}]]></wxappid>
          <send_name><![CDATA[#{options[:send_name]}]]></send_name>
          <re_openid><![CDATA[#{options[:re_openid]}]]></re_openid>
          <total_amount><![CDATA[#{options[:total_amount]}]]></total_amount>
          <total_num><![CDATA[#{options[:total_num]}]]></total_num>
          <wishing><![CDATA[#{options[:wishing]}]]></wishing>
          <client_ip><![CDATA[#{options[:client_ip]}]]></client_ip>
          <act_name><![CDATA[#{options[:act_name]}]]></act_name>
          <remark><![CDATA[#{options[:remark]}]]></remark>
          <nonce_str><![CDATA[#{options[:nonce_str]}]]></nonce_str>
          </xml>
        XML

        Nokogiri::XML xml
    end


> 3.3.4 发送请求并解析

    http.start do
      http.request_post(@uri.path, xml.to_xml) do |res|
        doc = Hash.from_xml(res.body)['xml']
        return '恭喜你获得红包' if doc['return_code'] == 'SUCCESS'
      end
    end
