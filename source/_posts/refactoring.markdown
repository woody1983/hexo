title: "Ruby Refactoring"
date: 2013-09-25 23:48
tag: Ruby
---

这堆神似的函数 越看越不爽 明天用method_missing重构

``` ruby
require 'yaml'

class Datasource
  
  def initialize()
    @dw = YAML.load(File.open('info_price.yml')) 
  end

  def get_cpu_info(computer_id)
    @dw[computer_id]["cpu_info"]
  end

  def get_cpu_price(computer_id)
    @dw[computer_id]["cpu_price"]
  end

  def get_mouse_info(computer_id)
    @dw[computer_id]["mouse_info"]
  end

  def get_mouse_price(computer_id)
    @dw[computer_id]["mouse_price"]
  end

  def get_memory_info(computer_id)
    @dw[computer_id]["memory_info"]
  end

  def get_memory_price(computer_id)
    @dw[computer_id]["memory_price"]
  end
end


#-- Debug --#
=begin
ds = Datasource.new
puts ds.get_cpu_info(10001)
puts ds.get_cpu_price(10002)
=end
```
<!-- more -->
### 参考

``` ruby
class Computer

  instance_methods.each do |m|
    undef_method m unless m.to_s =~ /^__|method_missing|respond_to?|object_id/
  end

  def initialize(data_source)
    #@pc_id = computer_id
    @data_source = data_source
  end

  def method_missing(name, *args)
      super if !respond_to?(name)
      #puts ">>> #{name}, args: #{args}, args class: #{args.class} and size: #{args.size}"
      
      args.inject(0) do |result,pc_id|
      	info = @data_source.send "get_#{name}_info", pc_id
      	price = @data_source.send "get_#{name}_price", pc_id
      	result = "#{name.capitalize}: #{info}, ($#{price})"
      	result = "* #{result}" if price > 1700
        puts result
      end
  end

  def respond_to?(method)
    @data_source.respond_to?("get_#{method}_info") || super
  end

end
```
