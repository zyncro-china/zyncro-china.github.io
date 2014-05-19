---
title: tips learn from confident ruby
date: 2014-03-24 03:21 UTC
tags: ruby
---

## The book confident ruby
After reading the book confident ruby, I want the screen cast with that book
and learned following tips

### use each_with_object instead of initialize an empty object before the each

```ruby

topics_list = []
another_list.each do |i|
  topics_list << i
end
```

should turn into

```ruby

result = another_list.each_with_object([]) do |i, topics_list|
  topics_list << i
end

```

### don't use collection.present? to check if collection exist and nonempty ,use Array()

### skip the iteration at the top of the block

```ruby

next unless topics[id]
```

### accept user or user_id with confident

```ruby

user_id = user
user_id = user.id unless user.class == Fixnum
```

To reduce the duplication, refactor to:
```ruby

private
def user_id_for(user_or_id)
  case user_or_id
  when Integer then user_or_id
  else
    user_or_id.id
  end
end
```

if define the private method, then use:

```ruby

def self.user_id_for
...
end
private_class_method :user_id_for
```

### return early(using guard) instead of using if at the top of the method

### rescue at the top level of the method, to avoid the indent and begin block
