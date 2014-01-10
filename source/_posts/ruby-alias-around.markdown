title: "Ruby Alias around"
date: 2013-09-25 23:51
tag: Ruby
---

## 关于环绕别名 实际用途还是蛮多的

``` ruby 
class Myclass
  def greet
    puts "Hello!"
  end

  def greet_with_log
    puts ">>>"
    greet_without_log #在这里调用的时候 其实是调用的是greet这个方法
    puts "<<<"
  end

  alias_method :greet_without_log, :greet
  alias_method :greet, :greet_with_log #再把含有Log输出的重新别名给greet这个方法

end

Myclass.new.greet_with_log
```
