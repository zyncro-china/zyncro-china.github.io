---
title: Delegate in ruby
date: 2014-03-12 03:03 UTC
tags: ruby, delegate
---

## Serveral ways to delegate in Ruby
There are serveral ways to delegate in Ruby:

1. SimpleDelegator
1. DelegateClass
1. Delegator
1.  forwardable module

### SimpleDelegator

A concrete implementation of Delegator, this class provides the means to
delegate all supported method calls to the object passed into the constructor
and even to change the object being delegated to at a later time with
\#\_\_setobj\_\_.

When using SimpleDelegator, the delegator is passed in the initalize method of SimpleDelegator Class,

Examples:

```ruby

require 'delegate'

class MyArray < SimpleDelegator

end

myarray = MyArray.new []

```

### DelegateClass

There are 2 steps

1. pass the DelegateTo Class in class definition
1. call super(obj\_of\_ClassToDelegateTo in the initialize method

```ruby

class MyClass < DelegateClass(ClassToDelegateTo) # Step 1
  def initialize
    super(obj_of_ClassToDelegateTo)              # Step 2
  end
end
```

### Delegator

We can define a customized delegator like SimpleDelegator

```ruby

class SimpleDelegator < Delegator
  def initialize(obj)
    super                  # pass obj to Delegator constructor, required
    @delegate_sd_obj = obj # store obj for future use
  end

  def __getobj__
    @delegate_sd_obj # return object we are delegating to, required
  end

  def __setobj__(obj)
    @delegate_sd_obj = obj # change delegation object,
                           # a feature we're providing
  end
end
```
### Forwardable module

```ruby

require 'forwardable'
class Meters
  extend Forwardable
  def_delegators :@value, :to_s, :to_int, :to_i
  ...
end

```
