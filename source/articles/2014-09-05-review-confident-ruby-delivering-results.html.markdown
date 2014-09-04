---
title: review confident ruby(delivering results)
date: 2014-09-05 13:44 UTC
tags: ruby
---

## 介绍 Delivering Results
为method的caller提供output
## Delivering Results的模式
1. Write total functions
1. Call back instead of returning
1. Represent failure with benign value
1. Represent failure with a special case object
1. Return a status object
1. Yield a status object
1. Signal early termination with throw


### Write total functions

当method可以有0，1或多个返回值时，应该都返回一个(有可能长度为0)collection，
这样使得处理结果时不需要考虑特殊情况

```ruby

def find_words(prefix)
  return [] if prefix.empty?
  magic_words = %w[klaatu barada nikto xyzzy plugh]
  words = File.readlines('/usr/share/dict/words').
    map(&:chomp).reject{|w| w =~ /'/}
  result = magic_words.include?(prefix) ? prefix :
    words.select{|w| w =~ /\A#{prefix}/}
  Array(result)
end
find_words('xyzzy')
# => ["xyzzy"]
```

### Call back instead of returning

```ruby
def import_purchase(date, title, user_email, &import_callback)
  user = User.find_by_email(user_email)
  unless user.purchased_titles.include?(title)
    purchase = user.purchases.create(title: title, purchased_at: date)
    import_callback.call(user, purchase)
  end
end
# ...
import_purchase(date, title, user_email) do |user, purchase|
  send_book_invitation_email(user.email, purchase.title)
end
# ...

```

### Represent failure with benign value

```ruby
def render_sidebar
  html = ""
  html << "<h4>What we're thinking about...</h4>"
  html << "<div id='tweets'>"
  html << latest_tweets(3)
  html << "</div>"
end

def latest_tweets(number)
  # ...fetch tweets...
  rescue Net::HTTPError
    ""
end


```
### Represent failure with a special case object
同上一章里的 Represent special cases as objects
### Return a status object
command
method的输出可能多种情况，不仅仅是success/failure,这时要把输出表示为status
object

```ruby
class ImportStatus
  def self.success() new(:success) end
  def self.redundant() new(:redundant) end
  def self.failed(error) new(:failed, error) end
  attr_reader :error
  def initialize(status, error=nil)
    @status = status
    @error = error
  end
  def success?
    @status == :success
  end
  def redundant?
    @status == :redundant
  end
  def failed?
    @status == :error
  end
end

def import_purchase(date, title, user_email)
  user = User.find_by_email(user_email)
  if user.purchased_titles.include?(title)
    ImportStatus.redundant
  else
    purchase = user.purchases.create(title: title, purchased_at: date)
    ImportStatus.success
  end
rescue => error
    ImportStatus.failed(error)
end

result = import_purchase(date, title, user_email)
if result.success?
  send_book_invitation_email(user_email, title)
elsif result.redundant?
  logger.info "Skipped #{title} for #{user_email}"
else
  logger.error "Error importing #{title} for #{user_email}: #{result.error}"
end
```
### Yield a status object
当一个command
method可能有多个输出，不仅仅是success/failure,我们不想它返回值，
我们可以把method的输出表示为一个带有callback style mehtods的status
object,并在caller yield 这个object


```ruby
result = import_purchase(date, title, user_email)
if result.success?
  send_book_invitation_email(user_email, title)
elsif result.redundant?
  logger.info "Skipped #{title} for #{user_email}"
else
  logger.error "Error importing #{title} for #{user_email}: #{result.error}"
end

```

上面的代码有2个问题:

1. 违反了 Command/Query Separation(CQS)原理，command不应返回值。
2. 而且不再能使用batch的方式，要使用batch的方式，我们必须先收集所有status
object到数组里，再迭代。

```ruby
class ImportStatus
  def self.success() new(:success) end
  def self.redundant() new(:redundant) end
  def self.failed(error) new(:failed, error) end
  attr_reader :error
  def initialize(status, error=nil)
    @status = status
    @error = error
  end
  def on_success
    yield if @status == :success
  end
  def on_redundant
    yield if @status == :redundant
  end
  def on_failed
    yield(error) if @status == :error
  end
end

def import_purchase(date, title, user_email)
  user = User.find_by_email(user_email)
  if user.purchased_titles.include?(title)
    yield ImportStatus.redundant
  else
    purchase = user.purchases.create(title: title, purchased_at: date)
    yield ImportStatus.success
  end
rescue => error
  yield ImportStatus.failed(error)
end

import_purchase(date, title, user_email) do |result|
  result.on_success do
    send_book_invitation_email(user_email, title)
  end
  result.on_redundant do
    logger.info "Skipped #{title} for #{user_email}"
  end
  result.on_error do |error|
    logger.error "Error importing #{title} for #{user_email}: #{error}"
  end
end
```
### Signal early termination with throw
