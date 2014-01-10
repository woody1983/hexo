title: "Ruby-Bits"
date: 2013-09-05 22:10
categories: Ruby
tags: Ruby
---

关于Ruby最佳实践的记录，我几乎已经放弃向周围的人推荐Ruby这门让人开心的语言了，原因其实很奇怪。

关于参数，Ruby的Function中定义的函数或类总是会接收很多参数 不接受参数的Function不觉得很奇葩吗？

传统的语法和实践是

``` ruby
def CHR(username,login_id,message)
 #...
end
```

这个不能说错 但按照CodeSchool老师的原话来说`Location isn't always used, so let's add default.`

``` ruby
def CHR(username,login_id=nil,message=nil)
 #...
end
```

还有一种情况就是参数过多，以致于让function变得毫无可读性`have to keep placeholders for arguments you’re not using`
这个时候可以采用 `HASH Arguments`

``` ruby

def tweet(message, options = {})
  status = Status.new
  status.lat = options[:lat]
  status.long = options[:long]
  status.body = message
  status.reply_id = options[:reply_id]
  status.post
end
```

调用的时候 就可以利用Symbol来实现多参数。

``` ruby
￼tweet("Practicing Ruby-Fu!",
  :lat => 28.55,
  :long => -81.33,
  :reply_id => 227946
￼￼)
```

<!-- more -->

###Using Ruby 1.9 hash syntax

``` ruby
tweet("Practicing Ruby-Fu!", 
    lat: 28.55,
    long: -81.33, 
    reply_id: 227946
)
```

参数的顺序也无所谓了 

``` ruby
￼tweet("Practicing Ruby-Fu!",
  reply_id: 227946,
  lat: 28.55,
  long: -81.33
)
```

一个参数也ok

``` ruby 
tweet("Practicing Ruby-Fu!")
```

``` ruby 
def new_game(name, options={})
  {
    name: name,
    year: options[:year],
    system: options[:system]
  }
end
game = new_game("Street Figher II", year: 1992, system: "SNES")
```

###EXCEPTIONS

关于判断例外EXCEPTIONS的问题 最传统的写法就是先做判断 有值就出来 如果没值的话 就抛除个神器的东西。貌似这么写程序不太好。

``` ruby
def get_tweets(list)
  if list.authorized?(@user)
    list.tweets #如果这个user是验证成功的就把他的tweet列出来
else
    end[] #“magic” return value
end
```

如果没有tweet 查出来是空的话  还要报一个Error 貌似在Ruby里没这个必要。

``` ruby 
tweets = get_tweets(my_list)
if tweets.empty?
  alert "No tweets were found!" + 
  "Are you authorized to access this list?"
end
```

最佳方法应该是用unless先对`@user`这个对象进行判断 如果没有就用`raise`来抛除一个异常并返回登陆.

``` ruby 
￼def get_tweets(list)
  unless list.authorized?(@user)
    raise AuthorizationException.new
  end
list.tweets
end

begin
  tweets = get_tweets(my_list)
rescue AuthorizationException
  warn "You are not authorized to access this list."
end
```

练习

``` ruby 
class InvalidGameError < StandardError; end
def new_game(name, options={})
  {
    name: name,
    year: options[:year],
    system: options[:system]
  }
end
begin
  game = new_game(nil)
rescue InvalidGameError => e
  puts "There was a problem creating your new game: #{e.message}"
end
```
We want to make sure that each game is a valid game object - in this case a simple hash of values. Even still, we wouldn't want to return a hash with a nil name. Raise an InvalidGameError error in the new_game method if name is nil.

``` ruby
class InvalidGameError < StandardError; end
def new_game(name, options={})
	raise InvalidGameError, "You must provide a name for this game." unless name
  {
    name: name,
    year: options[:year],
    system: options[:system]
  }
end
begin
  game = new_game(nil)
rescue InvalidGameError => e
  puts "There was a problem creating your new game: #{e.message}"
end
```

###splat arguments

说实话 我没觉得这个比hash那个要实用

``` ruby
￼def mention(status, *names)
  tweet("#{names.join(' ')} #{status}")
end

mention('Your courses rocked!', 'eallam', 'greggpollack', 'jasonvanlue')

def describe_favorites(*games)
  for game in games
    puts "Favorite Game: #{game}"
  end  
end
describe_favorites('Mario', 'Contra', 'Metroid')
```

为一个二维Array创建一个Class

``` ruby 
￼user_names = [
  ["Ashton", "Kutcher"],
  ["Wil", "Wheaton"],
  ["Madonna"]
]

user_names.each { |n| puts "#{n[1]}, #{n[0]}" }
```

这个要直接打印出来 最后那个就会多一个分隔符。

``` ruby
class Name
  def initialize(first, last = nil)
    @first = first
    @last = last
  end
 
￼￼def format
    [@last, @first].compact.join(', ')
  end 
end
```

