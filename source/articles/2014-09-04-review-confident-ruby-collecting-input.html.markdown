---
title: review confident ruby(collecting input)
date: 2014-09-04 04:38 UTC
tags: ruby
---
## 介绍 Collecting Input

method需要input，当决定好method的算法所需要的input，我们需要决定如何获取input。
有3种策略用于确保方法有可靠的collaborators来执行他们的role

1. Coerce objects into the roles we need them to play.
2. Reject unexpected values which cannot play the needed roles.
3. Substitute known-good objects for unacceptable inputs.

要记住只在边界做collectiing input,不要在method内部做。

## Collecting Input的模式

1. Use built-in conversion protocols
1. Conditionally call conversion methods
1. Define your own conversion protocols
1. Define conversions to user-defined types
1. Use built-in conversion functions
1. Use Array() conversion function to array-ify inputs
1. Define conversion function
1. Replace "string typing" with classes
1. Wrap collaborators in Adapters
1. Use transparent adapters to gradually introduce abstraction
1. Reject unworkable values with preconditions
1. Use #fetch to asset the presence of Hash keys
1. Use #fetch for defaults
1. Document assumptions with assertions
1. Handle special cases with Guard Clause
1. Represent special cases as objects
1. Represent do-nothing cases as null objects
1. Substitute a benign value for nil
1. Use symbols as placeholder objects
1. Bundle arguments into parameter objects
1. Yield a parameter builder object
1. Receive policies instead of data


### Use built-in conversion protocols
当需要确保input是特定类型，
使用Ruby定义好的conversion protocols, 如

```ruby
#to_str
#to_i
#to_path
#to_ary
```

这样可以确保类型的同时提高灵活性

#### standard conversion methods

Method|Target Class|Type
 ------------- | ------------- | ------------- 
\#to\_a|Array|Explicit
\#to\_ary|Array|Implicit
\#to\_c|Complex|Explicit
\#to\_enum|Enumerator|Explicit
\#to\_h|Hash|Explicit
\#to\_hash|Hash|Implicit
\#to\_i|Integer|Explicit
\#to\_int|Integer|Implicit
\#to\_io|IO|Implicit
\#to\_open|IO|Implicit
\#to\_path|String|Implicit
\#to\_proc|Proc|Implicit
\#to\_r|Rational|Explicit
\#to\_regexp|Regexp|Implicit
\#to\_s|String|Explicit
\#to\_str|String|Implicit
\#to\_sym|Symbol|Implicit

Implicit 和 Explicit
区别在于，如果定义了实际上是String的类，比如ArticleTitle,那么就应该定义Implicit #to_str,来告诉Ruby
core class接受并转为真正的String类


```ruby
"hello, " + article_title # => "hello, my title"
"the time is now: " + Time.now # =>
# ~> -:1:in `+': can't convert Time into String (TypeError)
# ~>
from -:1:in `<main>'

```

Explicit是用于获取文字的表示形式, 如to_s

Implicit也可以作为assert来用，确保input的类型

### Conditionally call conversion methods
使用respond_to?来决定input是否调用conversion method,
既检查了input，也可扩展其他的类型

```ruby

def my_open(filename)
filename = filename.to_path if filename.respond_to?(:to_path)
# 这里用to_str, 因为我们不仅是获得它的文字表示形式,
# 还要检查输入是不是String或可转化为String的对象
filename = filename.to_str
# ...
end
```

### Define your own conversion protocols

定义自己的conversion protocol,使其他对象也可以定义这个接口来转换

```ruby

# origin and ending should both be [x,y] pairs, or should
# define #to_coords to convert to an [x,y] pair
# 数据在函数内部都表示为[x,y],但输入时允许多种形式
def draw_line(start, endpoint)
  start = start.to_coords if start.respond_to?(:to_coords)
  start = start.to_ary
  # ...
end

class Point
  attr_reader :x, :y, :name
  def initialize(x, y, name=nil)
    @x, @y, @name = x, y, name
  end
  def to_coords
    [x,y]
  end
end

start = Point.new(23, 37)
endpoint = [45,89]
draw_line(start, endpoint)
```  
### Define conversions to user-defined types

当method需要自定义的类作为input，而且也希望其他类也可以implicitly converted
到需要的类型，可以定义自己的conversion protocol来把任意对象转换为目标对象。

```ruby
# 每次计算都通过CustomizedClass.new(value),返回新对象
# 使用delegator把message发给value实例变量
# 每个类都定义conversion protocol
def report_altitude_change(current_altitude, previous_altitude)
change = current_altitude - previous_altitude
# ...
end

require 'forwardable'
class Meters
  extend Forwardable
  def_delegators :@value, :to_s, :to_int, :to_i
  def initialize(value)
    @value = value
  end
  def -(other)
    self.class.new(value - other.to_meters.value)
  end
  def to_meters
    self
  end
  # ...
  protected
  attr_reader :value
end

class Feet
  # ...
  def to_meters
    Meters.new((value*0.348).round)
  end
end

```

新的单位只要定义了 protocol to\_meters就可以计算得正确的结果``

###Use built-in conversion functions

当需要把input对象 convet为core type, 可以使用Ruby的大写conversion函数: Integer,
Array

注意，Hash应使用Hash[]

### Use Array() conversion function to array-ify inputs
Array()比 to\_a 和 to\_ary 适用更多的情况

```ruby
# 3种conversion
Array(value)
value.to_a
value.to_ary
```
### Define conversion function

当public
api可以接受多种形式的input，内部只想用单一的类型。可以定义一个等效(idempotent)的conversion
function

```ruby
# 函数和类可以重名
def Point(*args)
  case args.first
  when Integer then Point.new(*args)
  when String then Point.new(*args.first.split(':').map(&:to_i))
  when ->(arg){ arg.respond_to?(:to_point) }
  args.first.to_point
  when ->(arg){ arg.respond_to?(:to_ary) }
    Point.new(*args.first.to_ary)
  else
    raise TypeError, "Cannot convert #{args.inspect} to Point"
  end
end
# Point class now defines #to_point itself
module Graphics
  module Conversions
    module_function
    Point = Struct.new(:x, :y) do
      def inspect
      "#{x}:#{y}"
      end
      def to_point
        self
      end
    end
    # A Pair class which can be converted to an Array
    Pair = Struct.new(:a, :b) do
      def to_ary
        [a, b]
      end
    end
    # A class which can convert itself to Point
    class Flag
      def initialize(x, y, flag_color)
        @x, @y, @flag_color = x, y, flag_color
      end
      def to_point
        Point.new(@x, @y)
      end
    end
  end
end
include Graphics
include Graphics::Conversions
Point([5,7]) # => 5:7
Point(Pair.new(23, 32)) # => 23:32
Point(Flag.new(42, 24, :red)) # => 42:24

```
支持 to\_ary和to\_point两种conversion prototol

关于 lambda -> {}, Proc的方法 #=== 是 #call 的alias,因此可以像如下使用

```ruby

even = ->(x) { (x % 2) == 0 }
even === 4 # => true
even === 9 # => false

```

关于 module_function, 它把后面的method设为private,并且作为module的singleton
method

### Replace "string typing" with classes
问题代码

```ruby
class TrafficLight
  # Change to a new state
  def change_to(state)
    @state = state
  end
  def signal
    case @state
    when "stop" then turn_on_lamp(:red)
    when "caution"
      turn_on_lamp(:yellow)
      ring_warning_bell
    when "proceed" then turn_on_lamp(:green)
    end
  end
  def next_state
    case @state
    when "stop" then "proceed"
    when "caution" then "stop"
    when "proceed" then "caution"
    end
  end
  def turn_on_lamp(color)
    puts "Turning on #{color} lamp"
  end
  def ring_warning_bell
    puts "Ring ring ring!"
  end
end
```


当需要根据string值来做分支处理时存在的问题

1. 对于相同变量的重复的case statement
1. case没有else分支，当条件不匹配时不会报错
1. 这么多case statement不是好的代码风格

应该使用type system和polymorphic method 来dispatch work，而不是使用case
when分支。这样不仅代码更健壮，可以理清对问题的理解，使得method花更少的时间用在input
checking，更多时间用来讲述清晰的故事。

```ruby
#定义一个类，使用不同对象来表示状态，并把next_state的条件判断移到实例变量@next_state
class TrafficLight
  State = Struct.new(:name, :next_state) do
    def to_s
      name
    end
  end
  VALID_STATES = [
    STOP = State.new("stop", "proceed"),
    CAUTION = State.new("caution", "stop"),
    PROCEED = State.new("proceed", "caution")
  ]
  # ...
  def next_state
    @state.next_state
  end
end
```

但是 “caution” case无法移到实例变量，所以转为使用subclass来表示不同状态

```ruby

class TrafficLight
  class State
    def to_s
      name
    end
    def name
      self.class.name.split('::').last.downcase
    end
    def signal(traffic_light)
      traffic_light.turn_on_lamp(color.to_sym)
    end
  end
  class Stop < State
    def color; 'red'; end
    def next_state; Proceed.new; end
  end
  class Caution < State
    def color; 'yellow'; end
    def next_state; Stop.new; end
    def signal(traffic_light)
      super
      traffic_light.ring_warning_bell
    end
  end
  class Proceed < State
    def color; 'green'; end
  def next_state; Caution.new; end
  end

  def next_state
    @state.next_state
  end
  def signal
    @state.signal(self)
  end
end
```
为了避免下面繁琐的表示形式

```ruby
  light = TrafficLight.new
  light.change_to(TrafficLight::Caution.new)
  light.signal
```

可以使用Symbol或String作为input,转换为类:
self.class.const_get(state.to_s.capitalize).new

```ruby

class TrafficLight
  def change_to(state)
    @state = State(state)
  end
  # ...
  private
  def State(state)
    case state
    when State then state
    else self.class.const_get(state.to_s.capitalize).new
    end
  end
end

light = TrafficLight.new
light.change_to(:caution)
light.signal
puts "Next state is: #{light.next_state}"
Turning on yellow lamp
Ring ring ring!
Next state is: stop
```
### Wrap collaborators in Adapters
method可以接受不同类型的collaborator,他们没有共同的interface，
需要用adapter包住这些对象，使它们有一致的interface
adapter的作用在于隐藏让人分心的特殊情况处理，确保特殊情况只需要处理一次

```ruby
class BenchmarkedLogger
  def initialize(sink=$stdout)
    @sink = sink
  end
  def info(message)
    start_time = Time.now
    yield
    duration = start_time - Time.now
    @sink << ("[%1.3f] %s\n" % [duration, message])
  end
end

```
要兼容IRC的情况, 定义wrapper类，并添加<<方法:

```ruby
class BenchmarkedLogger
  class IrcBotSink
    def initialize(bot)
      @bot = bot
    end
    def <<(message)
      @bot.handlers.dispatch(:log_info, nil, message)
    end
  end
  def initialize(sink)
    @sink = case sink
      when Cinch::Bot then IrcBotSink.new(sink)
      else sink
      end
  end
  def info(message)
    start_time = Time.now
    yield
    duration = start_time - Time.now
    @sink << ("[%1.3f] %s\n" % [duration, message])
  end
end

require 'cinch'
bot = Cinch::Bot.new do
  configure do |c|
    c.nick = "bm-logger"
    c.server = ENV["LOG_SERVER"]
    c.channels = [ENV["LOG_CHANNEL"]]
    c.verbose = true
  end
  on :log_info do |m, line|
    Channel(ENV["LOG_CHANNEL"]).msg(line)
  end
end
bot_thread = Thread.new do
  bot.start
end

```
### Use transparent adapters to gradually introduce abstraction

当需要逐步替代原有对象时，可以使用DelegateClass来替换对象，再逐步改造类的定义

### Reject unworkable values with preconditions

当遇到无法转换的input时应该使用precondition尽早reject

```ruby
class Employee
  attr_accessor :name
  attr_reader :hire_date
  def initialize(name, hire_date)
    @name = name
    self.hire_date = hire_date
  end
  def hire_date=(new_hire_date)
    raise TypeError, "Invalid hire date" unless new_hire_date.is_a?(Date)
    @hire_date = new_hire_date
  end
end
```
### Use #fetch to asset the presence of Hash keys
### Use #fetch for defaults
使用myhash.fetch(key) {default} 而不要使用2个参数的fetch： myhash.fetch(key,
defualt), 因为这种形式每次都执行，可能造成困扰
### Document assumptions with assertions

```ruby
class Account
  def refresh_transactions
    transactions = bank.read_transactions(account_number)
    transactions.is_a?(Array) or raise TypeError, "transactions is not an Array"
    transactions.each do |transaction|
      amount = transaction.fetch("amount")
      amount_cents = (Float(amount) * 100).to_i
      cache_transaction(:amount => amount_cents)
    end
  end
end

```
1. 在边界而不是在内部
1. 使用raise来验证假设
1. Float()而不是to\_f
1. hash.fetch而不是hash[]

### Handle special cases with Guard Clause

```ruby
def log_reading(reading_or_readings))
  if @quiet then return end
  # ...
end

```

### Represent special cases as objects
在特殊情况用对象来表示，而不是用nil，这样可以避免在所有用到的地方做nil检查

```ruby

class User
  def authenticated?
    true
  end
end

class GuestUser
  def initialize(session)
    @session = session
  end

  def name
    'anonymous'
  end

  def authenticated?
    false
  end

end

def current_user
  if session[:user_id]
    User.find(session[:user_id])
  else
    GuestUser.new(session)
  end
end
```
这个模式的缺点是GuestUser必须和User的接口保持同步，可以通过shared
test suite来确保这点

```ruby
shared_examples_for 'a user' do
  it { should respond_to(:name) }
  it { should respond_to(:authenticated?) }
  it { should respond_to(:has_role?) }
  it { should respond_to(:visible_listings) }
  it { should respond_to(:last_seen_online=) }
  it { should respond_to(:cart) }
end
describe GuestUser do
  subject { GuestUser.new(stub('session')) }
  it_should_behave_like 'a user'
end
describe User do
  subject { User.new }
  it_should_behave_like 'a user'
end

```

### Represent do-nothing cases as null objects
当其中一个input是nil时，
这表示是一个特殊情形，处理这个特殊情形的方式是什么也不做，
这时应该把nil替换为一个Null
Object的collaborator，它和一般的collaborator有相同的接口，但是它接受消息，不执行动作

```ruby
class NullObject < BasicObject
  def method_missing(*)
  end
  def respond_to?(name)
    true
  end
end
```

但是，如果遇到method call chain, 或LoD时，需要做些改变

```ruby

class NullObject
  def method_missing(*)
    self
  end
# ...
end
 
```
这样做的缺点是返回一个black hole null object,因此，应该在边界处(public api 返回值时)把对象还原回nil

```ruby

# return either the argument or nil, but never a NullObject
def Actual(object)
  case object
  when NullObject then nil
  else object
  end
end
Actual(User.new)
Actual(nil)
Actual(NullObject.new)
# => #<User:0x00000002218d18>
# => nil
# => nil

def create_widget(attributes={}, data_store=nil)
  data_store ||= NullObject.new
  Actual(data_store.store(Widget.new(attributes)))
end
```

还要让null object是falsey

```ruby

class NullObject < BasicObject
  def method_missing(*)
  end
  def respond_to_missing?(name)
    true
  end
  def nil?
    true
  end
  def !
    true
  end
end

```

但ruby不允许定义我们自己的falsey

```ruby

null ? "truthy" : "falsey" # => "truthy"

```
所以，关键是要记住，不需要去检查null，直接可以调用它的方法，如果它是null对象，它只会安静地什么也不做。


### Substitute a benign value for nil

如果一个不重要的input没有提供, 用一个可用的替代值，不一定需要raise exception

```ruby

def render_member(member, group)
  location = Geolocatron.locate(member.address) ||
  group.city_location # good benign value
  html = ""
  html << "<div class='vcard'>"
  html << " <div class='fn'>#{member.fname} #{member.lname}</div>"
  html << " <img class='photo' src='#{member.avatar_url}'/>"
  html << " <img class='map' src='#{location.map_url}'/>"
  html << "</div>"
end"'"

```

### Use symbols as placeholder objects

根据方法的调用，可选的collaborator可能用到，也可能没用到，我们应该用symbol，而不是nil作为placeholder，这样如果出错，也更容易调试

``` ruby
def list_widgets(options={})
  credentials = options.fetch(:credentials) { :credentials_not_set }
  page_size = options.fetch(:page_size) { 20 }
  page = options.fetch(:page) { 1 }
  if page_size > 20
    user = credentials.fetch(:user)
    password = credentials.fetch(:password)
    url = "https://#{user}:#{password}" +
      "@www.example.com/widgets?page=#{page}&page_size=#{page_size}"
  else
    url = "http://www.example.com/widgets" +
      "?page=#{page}&page_size=#{page_size}"
  end
  puts "Contacting #{url}..."
end

```
### Bundle arguments into parameter objects

如果多个method都传入相同参数列表，应该把参数合到一个新类。比如坐标数组可以用整合到point类

Double Dispatch pattern使Point画出它自己。

```ruby
Point = Struct.new(:x, :y) do
  # ...
  def draw_on(map)
    # ...
  end
end
class Map
  def draw_point(point)
    point.draw_on(self)
  end
  def draw_line(point1, point2)
    point1.draw_on(self)
    point2.draw_on(self)
    # draw line connecting points...
  end
end
```
不要把新类型的信息(如stared)作为参数(例如option)传入，而是定义新的类

StarredPoint只改一点，所以使用继承

```ruby

class StarredPoint < Point
  def draw_on(map)
  # draw a star instead of a dot...
  end
  def to_hash
    super.merge(starred: true)
  end
end

```

FuzzyPoint可以用Decorator模式，用SimpleDelegator把大部分method代理到原对象，再override draw\_on

```ruby

require 'delegate'
class FuzzyPoint < SimpleDelegator
  def initialize(point, fuzzy_radius)
    super(point)
    @fuzzy_radius = fuzzy_radius
  end
  def draw_on(map)
    super # draw the point
    # draw a circle around the point...
  end
  def to_hash
    super.merge(fuzzy_radius: @fuzzy_radius)
  end
end

```

把如何画自己的逻辑都封装在不同的Point类里，因此Point的用户不需要改变就可以支持不同的point

```ruby
Point = Struct.new(:x, :y) do
  # ...
  def draw_on(map)
  # ...
  end
end
class Map
  def draw_point(point)
    point.draw_on(self)
  end
  def draw_line(point1, point2)
    point1.draw_on(self)
    point2.draw_on(self)
    # draw line connecting points...
  end
end
class MapStore
  def write_point(point)
    point_hash = point.to_hash
    # ...
  end
# ...
end
```

把point的信息(star or fuzzy)作为参数的类型, 而不是额外的参数(option),
可以消除所有对不同point属性的条件判断

### Yield a parameter builder object
在上个模式里，用户必须知道很多不同的类才能使用API，因此应该隐藏对象的构建，并yield
parameter object 或 parameter builder object

#### yiled parameter object

```ruby

Point < Struct.new(:x, :y, :name, :magnitude) do
  def initialize(x, y, name='', magnitude=5)
    super(x, y, name, magnitude)
  end
  def magnitude=(magnitude)
    raise ArgumentError unless (1..20).include?(magnitude)
    super(magnitude)
  end
  # ...
end

class Map
  def draw_point(point_or_x, y=:y_not_set_in_draw_point)
    point = point_or_x.is_a?(Integer) ? Point.new(point_or_x, y) : point_or_x
    yield(point) if block_given?
    point.draw_on(self)
  end
  def draw_starred_point(x, y, &point_customization)
    draw_point(StarredPoint.new(x, y), &point_customization)
  end
# ...
end

map.draw_point(7, 9) do |point|
  point.magnitude = 3
end
map.draw_starred_point(18, 27) do |point|
  point.name = "home base"
end

```

#### yiled parameter builder object

```ruby

require 'delegate'
class PointBuilder < SimpleDelegator
  def initialize(point)
    super(point)
  end
  def fuzzy_radius=(fuzzy_radius)
    # __setobj__ is how we replace the wrapped object in a
    # SimpleDelegator
    __setobj__(FuzzyPoint.new(point, fuzzy_radius))
  end
  def point
    # __getobj__ is how we access the wrapped object directly in a
    # SimpleDelegator
    __getobj__
  end
end

class Map
  def draw_point(point_or_x, y=:y_not_set_in_draw_point)
    point = point_or_x.is_a?(Integer) ? Point.new(point_or_x, y) : point_or_x
    builder = PointBuilder.new(point)
    yield(builder) if block_given?
    builder.point.draw_on(self)
  end
  def draw_starred_point(x, y)
    draw_point(StarredPoint.new(x, y))
  end
# ...
end

map.draw_starred_point(7, 9) do |point|
  point.name = "gold buried here"
  point.magnitude = 15
  point.fuzzy_radius = 50
end


my_point = MySpecialPoint.new(123, 321)
map.draw_point(my_point) do |point|
  point.fuzzy_radius = 20
end

```

### Receive policies instead of data

```ruby

def delete_files(files, options={})
  error_policy = options.fetch(:on_error) { ->(file, error) { raise error } }
  symlink_policy = options.fetch(:on_symlink) { ->(file) { File.delete(file) } }
  files.each do |file|
    begin
      if File.symlink?(file)
        symlink_policy.call(file)
      else
        File.delete(file)
      end
    rescue => error
      error_policy.call(file, error)
    end
  end
end

delete_files( 
  ['file1', 'file2'], 
  on_error: ->(file, error) { warn error.message },
  on_symlink: ->(file) { File.delete(File.realpath(file)) })

```
