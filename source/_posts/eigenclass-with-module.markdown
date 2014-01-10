title: "eigenclass with Module"
date: 2013-09-25 23:23
categories: Ruby
tag: Ruby
---

我用了一个很笨的方法才把这个搞明白 那就是... 

# Module的eigenclass 不会被任何事物继承  绝对封闭！

`元编程#129Pages#Quiz`

### 例子弄错了~在线编辑器害死人  凸

``` ruby 
module MyModule 
  def my_method; 'hello';end
end

class Myclass 
  include MyModule
end

irb(main):004:0> MyModule.my_method
#NoMethodError: undefined method `my_method' for MyModule:Module
irb(main):010:0> puts Myclass.my_method
#NoMethodError: undefined method `my_method' for Myclass:Class
irb(main):011:0> obj = Myclass.new
#=> #<Myclass:0x861fa78>
irb(main):012:0> puts obj.my_method
#hello
#=> nil
```

在Module中定义一个普通的实例方法 Myclass Include该Module 只有Myclass的实例obj可以调用
也就是说`my_method`一直作为实例方法在ancestors中流窜

<!-- more -->

``` ruby
module MyModule 
	def self.my_method; 'hello';end
end

class Myclass
  include MyModule
end

irb(main):007:0> MyModule.my_method
#=> "hello"
irb(main):008:0> Myclass.my_method
#NoMethodError: undefined method `my_method' for Myclass:Class
irb(main):009:0> obj = Myclass.new
#=> #<Myclass:0x8627980>
irb(main):010:0> obj.my_method
#NoMethodError: undefined method `my_method' for #<Myclass:0x8627980>
```

第二次  将Module中的方法改成是了实例方法  结果正常 只有MyModule可以正常调用  但Myclass并没有继承到
难道在Myclass的eigenclass中并没有得到这个继承？

``` ruby
module MyModule 
  def my_method; 'hello';end
end

class Myclass 
  class << self
	include MyModule
  end
end

irb(main):009:0> MyModule.my_method
#NoMethodError: undefined method `my_method' for MyModule:Module
irb(main):010:0> Myclass.my_method
#=> "hello"
irb(main):011:0> obj = Myclass.new
#=> #<Myclass:0x8627570>
irb(main):012:0> obj.my_method
#NoMethodError: undefined method `my_method' for #<Myclass:0x8627570>
```

第三次和第一次的修改差异在 将MyModule植入到 Myclass的eigenclass中了~ Myclass可以正常调用该类方法
### 可是作为 Myclass的实例对象obj却不能调用这个my_method了

``` ruby
module MyModule 
	def self.my_method; 'hello';end
end

class Myclass 
  class << self
	include MyModule
  end
end

irb(main):009:0> MyModule.my_method
#=> "hello"
irb(main):010:0> Myclass.my_method
#NoMethodError: undefined method `my_method' for Myclass:Class
irb(main):011:0> obj = Myclass.new
#=> #<Myclass:0x862bfa8>
irb(main):012:0> puts obj.my_method
#NoMethodError: undefined method `my_method' for #<Myclass:0x862bfa8>

# my_method就算变成Module的类方法以后 还是被继承下来了 和普通的Include看起来效果好像是一样的
```

第四次和第二次 得到的结果是一样的 也就是进没进Myclass的eigenclass都一样~ 

我把第四部分重写了一下 加入了另一个实例方法在Module中

``` ruby 
module MyModule
  def self.my_method; 'hello';end
  def mine_method; 'another method'; end
end

class Myclass
  class << self
    include MyModule
  end
end

irb(main):012:0* Myclass.mine_method
#=> "another method"
irb(main):013:0> Myclass.my_method
#NoMethodError: undefined method `my_method' for Myclass:Class

```

看来`class << self`这种方法只能把MyModule中的实例方法植入到Myclass的eigenclass中 类方法不行

## 难道下面这么做是让 MyModule仅仅植入到 Myclass的eigenclass中 而在Myclass中是不可见的？

``` ruby 
class Myclass 
  class << self
    include MyModule
  end
end
```
