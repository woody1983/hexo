title: "Ruby Block & Proc"
date: 2013-09-25 23:19
categories: Ruby
tag: Ruby
---



``` ruby
def math(a,b)
    yield(a,b)
end
#math写成这样子 不跟block是会报错的 因为功能单一 也没必要加上 block_given?

def teach_math(a,b,&opertion)
    puts math(a,b,&opertion)
end

#opertion加上&意思就是 opertion是个Proc对象 但我当Block来用

teach_math(2,3) {|x,y| x * y } 
#调用的时候2和3会当作参数穿给a和b两个变量 后面挂着的Block会传给opertion对象
```

<!-- more -->

``` ruby
# 稍微修改一下就看出对比了
#因为math定义了一个yield 所以就必须有一个block挂上去一起玩或者把一个Proc当做block来挂着玩
#懂？


# 这么写的话  opertion_again就必须是一个proc的对象 使用其自身的call方法来回调
def math(a,b,opertion_again)
    opertion_again.call(a,b)
end

def teach_math(a, b, &opertion)
    puts math(a,b,opertion)#在这里就传进去一个proc对象而不是一个当做block的proc
end

teach_math(2,3) { |x,y| x * y } 

#-------------------Block--->Proc-------------------------------------- 
def my_method(&the_proc)
    the_proc #去掉了&就变成了一个普通的proc对象并返回出去！
end

cmp = my_method {|name| "Hello,#{name}!"}#cmp其实是me_method的返回值就是一个proc对象
#所以要call回调
cmp.call("Johnny")

#-------------------Proc--->Block-------------------------------------- 

def my_method(greeting)
    puts "#{greeting},#{yield}"#看到yield了就意味着这个方法调用的时候需要跟一个block
end

my_proc = proc { "Woody" } #声明一个proc对象

my_method("Hello",&my_proc)#把Proc当Block ！again!
    
```

``` ruby
def me_method()
  yield
end
# puts my_method { "xXx" }   => "xXx"

def me_method2(&my_proc)
  my_proc.call
end
#me_method2 { "xXx" }   => "xXx"

#当你会把Block和Proc串起来玩的时候  可调用对象的概念应该了解的差不多了

def me_method()
  yield
end

def me_method2(&my_proc)
  me_method(&my_proc)
end

me_method2 { "_why" } # => "_why"
```

> 这两个方法在调用的时候都需要跟一个block

### 大招

``` ruby 
["1", "2", "3"].map(&:to_i)
#其效果和下面这个一样
["1", "2", "3"].map {|i| i.to_i }
```
> Reg Braithwaite简单地介绍了to_proc是怎样让 (1..100).inject(&:+) 这样的代码工作的:


& 操作符用来把Proc对象转化成块，或者把块转化成Proc对象。这此例中，它试图把符号 :+ 转换成一个块。此转换过程使用了Ruby内建的强制机制。这个机制会检查我们是否有一个Proc对象。如果没有，它就把#to_proc方法传递到参数中 来生成一个Proc。如果符号 :+ 有#to_proc方法，就调用它。在Ruby 1.9中，它就有一个#to_proc方法。此方法使用了第一个参数，然后返回一个Proc，并把 + 方法和其他参数传递给它。
由此可见，&:+ 实际上就是 { |x, y| x + y }


``` ruby 
(1..5).map(&:to_s) 
#=> ["1", "2", "3", "4", "5"]
(1..5).map{|x| x.to_s }
#=> ["1", "2", "3", "4", "5"]

 ["woody","johnny"].map(&:capitalize) 
#=> ["Woody", "Johnny"]
```

### 另一种可调用对象的方法  .method

``` ruby 
class MyClass
    def initialize(value)
        @name = value
    end
    
    def my_method
        @name.capitalize
    end
end
    
object = MyClass.new("woody")
cmp = object.method :my_method
cmp.class #=> Method
cmp.call #=> "Woody"
#String.instance_methods
```

### instance_eval 
`object#instance_eval` 可以在对象的上下文中执行一个Block

``` ruby
@setups = []
# 初始化
def setup(&block)
  @setups << block
end

setup do
  puts "[\033[1;36;40m Setting up \033[m] [\033[1;37;40m Sky \033[m]"
  @sky_height = 100
end

setup do
  puts "[\033[1;36;40m Setting up \033[m] [\033[1;37;40m Mountains \033[m]"
  @mountains_height = 200
end

 @setups.each {|setup| env.instance_eval &setup }
```


