---
layout: post
title: Ruby-Learn
category : lession
tags : Ruby

---
ruby learn 

### 1. 常见的逻辑处理

#### 1.1 if and case
```ruby
    var = true
    if var == true then puts "something"
    elsif if var == false then pust "nothing"
    else puts "error"
    end
```

```ruby
    case var
      when true:  puts true
      when false: puts false
      else  puts "nothing"
    end


    value = case var
      when true: true
      when false: false
      else "nothing"
    end

```

###  1.2 循环（while do/ begin end while .....  break end, upto/downto, until, times）

    while i < 10 do
      if i == false then break
      i += 1
    end

    begin
      i += 1
    end while i < 10

---
    until i > 10 do
    .......
    end

---

    5.times {|i| puts i} / 5.downto(1) {|i| puts i} / 1.upto(5) {|i| puts i}

---

    for i in 1...5(no include 5) / 1..5(include 5) do
    .....
    end

---

## 2. 字符串处理

    a = "hello"
    b = "world"
    a += b
    # a << b

---
    puts a[0], a[-1]
    a =  "345" * 2   # 345345
    a.insert 1, "eeeee"   # a = "3eeeee45"
    #others can by doc

---
    a.each_byte/ a.each_char, a.next / a[0].next

## 3. 数组

#### 3.1 常规使用

    arr = []
    arr.each/ arr[0]/ arr[-1]/ a[1..4]
    arr << "bcd" / arr += "bcd"
    a = [] , b = []
    a = a | b / a & b | a  - b

#### 3.2 一些数据结构

    #栈
    stack = []
    stack.push "a" #入栈
    stack.pop #出栈
    #队列
    queue = []
    queue.push "a" #进队列
    queue.shift #出队列
    queue.unshift ("a", "b") / queue.unshift #在队列最前面加入数值

## 4. 哈希

    hash = {:name => "something", ....... }
    hash = {"name" => "something"}
    hash.each |key, value| do
       ......
    end

## 5. 类

    class Hello
       def initialize(instance)
         @instance = intance
        end
    end

   @instance/@@class_instance/$global/Const

    class Accessor
      attr: :reader, :writer
      attr_accessor :same
      attr_reader: :only_read
      attr_writer: :only_writer
    end
    class Area
      def Area.rect
      ....#类方法
      end
    end

    class parent
    private
    ....
    protected
    ....
    end

    moudle Dice
      def somefunction
      end
    end

    require 'file.rb' / 'file'
    class Sub < parent
       include Dice
       ......
     end