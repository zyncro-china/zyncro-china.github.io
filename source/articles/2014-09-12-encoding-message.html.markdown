---
title: encoding message
date: 2014-09-12 01:45 UTC
tags: ruby, practicingruby
---

## 关于practicingruby
practicingruby是Gregory
Brown定期撰写的关于ruby编程的文章。很有深度，欢迎大家订阅。

## The anatomy of information in software systems
9月份的文章是关于如何把消息编码。内容如下的IRC消息，应该如何解释

```ruby

PRIVMSG #practicing ruby testing :Seasons greetings to you all!\r\n
```
在ruby里可以表示为

```ruby

["PRIVMSG", "#practicing ruby testing", "Seasons greetings to you all!"]
```

### 序列化为字符串
使用[Msgpack](https://github.com/msgpack/msgpack/blob/master/spec.md)序列化上面的消息,得到的HEX是

```

93 a7 50 52 49 56 4d 53 47 b8 23 70 72 61 63 74 69 63 69 6e 67 2d 72 75 62 
79 2d 74 65 73 74 69 6e 67 bd 53 65 61 73 6f 6e 73 20 67 72 65 65 74 69 6e 
67 73 20 74 6f 20 79 6f 75 20 61 6c 6c 21

```

In Ruby:

```ruby

"\x93\xA7PRIVMSG\xB8#practicing-ruby-testing\xBDSeasons greetings to you all!"

```
分解它的结构得到：

![Image](http://i.imgur.com/YAh5olr.png)

其中93, A7, B8, BD有特定含义, 实际上A0到BF表示的是0到31 byte的string

```ruby

>> 0xA7-0xA0
=> 7
>> "PRIVMSG".size
=> 7

>> 0xB8-0xA0
=> 24
>> "#practicing-ruby-testing".size
=> 24

>> 0xBD-0xA0
=> 29
>> "Seasons greetings to you all!".size
=> 29

```

### 结构化
不仅是得到字符串数组，我们还要明确每个部分的意义，可以用MessagePack's
application-specific type

![image](http://i.imgur.com/s3Rjgzz.png)

分解后得到的是

![image](http://i.imgur.com/AubaxCk.png)

ruby decode 代码

```ruby

data_types = { 1 => CommandName, 2 => Parameter, 3 => MessageBody }

command = MessagePackDecoder.unpack(raw_bytes, data_types)
#  [ CommandName <"PRIVMSG">, 
#    Parameter   <"#practicing-ruby-testing">, 
#    MessageBody <"Season greetings to you all!"> ]

```

handler decode的例子代码

```ruby

class Parameter
  def initialize(byte_array)
    @text = byte_array.pack("C*")

    raise ArgumentError if @text.include?(" ")
  end

  attr_reader :text
end
```

### 使用语法规则定义结构
上面的方法虽然让计算机容易理解消息的结构，但让人去理解就困难了。

如果使用Augmented Backus–Naur Form语法规则，可以表示如下

```

message    =  command [ params ] crlf
command    =  1*letter / 3digit
params     =  *14( SPACE middle ) [ SPACE ":" trailing ]
           =/ 14( SPACE middle ) [ SPACE [ ":" ] trailing ]

nospcrlfcl =  %x01-09 / %x0B-0C / %x0E-1F / %x21-39 / %x3B-FF
                ; any octet except NUL, CR, LF, " " and ":"

middle     =  nospcrlfcl *( ":" / nospcrlfcl )
trailing   =  *( ":" / " " / nospcrlfcl )

SPACE      =  %x20        ; space character
crlf       =  %x0D %x0A   ; "carriage return" "linefeed"
letter     =  %x41-5A / %x61-7A       ; A-Z / a-z
digit      =  %x30-39                 ; 0-9

```

在ruby，我们用[citrus](https://github.com/mjackson/citrus)

```ruby
grammar IRC
  rule message
    (command params? endline) {
      { :command => capture(:command).value,
        :params  => capture(:params).value }
    }
  end

  rule command
    letters | three_digit_code 
  end

  rule params
    ( ((space middle)14*14 (space ":"? trailing)?) |
      ((space middle)*14 (space ":" trailing)?) ) {
      captures.fetch(:middle, []) + captures.fetch(:trailing, [])
    }
  end

  rule middle
    non_special (non_special | ":")*
  end

  rule trailing
    (non_special | space | ":")+
  end

  rule letters
    [a-zA-Z]+
  end

  rule three_digit_code
    /\d{3}/ { to_str.to_i }
  end

  rule non_special
    [^\0:\r\n ]
  end

  rule space
    " "
  end

  rule endline
    "\r\n"
  end
end


require 'citrus'
Citrus.load('irc')

msg = "PRIVMSG #practicing-ruby-testing :Seasons greetings to you all!\r\n"

data = IRC.parse(msg).value

p data[:command] 
#=> "PRIVMSG"

p data[:params]
#=> ["#practicing-ruby-testing", "Seasons greetings to you all!"]

```

