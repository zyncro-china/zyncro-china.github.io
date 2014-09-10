---
title: review confident ruby(handling failure)
date: 2014-09-06 06:19 UTC
tags: ruby
---
## 介绍 Handling Failure
自信地处理错误情况。这本书只讨论BRE(begin/rescue/end) block,因为BRE在代码里是如此醒目,
会把注意力引离method要做的事，所以需要想办法不要让它出现在程序的主要逻辑里。
## Handling Failure的模式
1. Prefer top-level rescue clause
1. Use checked methods for risky operations
1. Use bouncer methods


### Prefer top-level rescue clause
当代码里有begin/rescue/end block时，应尽量使用top level的形式,
这样可以清晰地划分出happy path和错误处理部分。

```ruby

def bar
  # happy path goes up here
rescue #---------------------------
  # failure scenarios go down here
end

```
### Use checked methods for risky operations
如果一个method里包含需要处理从lib或system
call的调用出现的错误，那么应该单独写一个函数把它包住,并处理可能出现的异常，这样可以避免重复处理底层的异常

```ruby
def filter_through_pipe(command, message)
  results = nil
  IO.popen(command, "w+") do |process|
    results = begin
      process.write(message)
      process.close_write
      process.read
    rescue Errno::EPIPE
      message
    end
  end
  results
end
```

可以用"Receive policies instead of data"模式重构为

```ruby
def checked_popen(command, mode, error_policy=->{raise})
  IO.popen(command, mode) do |process|
    return yield(process)
  end
rescue Errno::EPIPE
  error_policy.call
end

def filter_through_pipe(command, message)
  checked_popen(command, "w+", ->{message}) do |process|
    process.write(message)
    process.close_write
    process.read
  end
end

```
### Use bouncer methods
