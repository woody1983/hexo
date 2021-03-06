title: "Ruby On Kettle"
date: 2013/09/09 22:34
categories: Ruby
tags: [Ruby ,Kettle]
---

``` ruby
1.9.3-p194 :009 > "1111111111".to_i(2)
 => 1023 
1.9.3-p194 :010 > 1023.to_s(2)
 => "1111111111" 
```

总要为Kettle做点什么 用二进制来完成一对多线路分发 监控就成了主要的问题。

``` ruby
1.9.3-p194 :013 > "1111111111".size
 => 10 
1.9.3-p194 :014 > "1111111111".length
 => 10 
```

#### 10位对应10个分发线路

一个最常见的问题 我需要充值某条数据的某个线路重新发送 或者紧急止停某条线路的一批数据下发，so 要定位

``` ruby
1.9.3-p194 :016 > "1111111111".split('')
 => ["1", "1", "1", "1", "1", "1", "1", "1", "1", "1"] 
```

#### 重置Reset 0号线路 也就是Array最右边那个 

``` ruby 
1.9.3-p194 :021 > rok = "1111111111".split('')
 => ["1", "1", "1", "1", "1", "1", "1", "1", "1", "1"] 
1.9.3-p194 :022 > rok
 => ["1", "1", "1", "1", "1", "1", "1", "1", "1", "1"] 
1.9.3-p194 :023 > rok[10-1]
 => "1" 
1.9.3-p194 :024 > rok[10-1] = "0"
 => "0" 
1.9.3-p194 :025 > rok
 => ["1", "1", "1", "1", "1", "1", "1", "1", "1", "0"] 
```

`明天抽时间写一个Class 封装这个动作`

### 我想要的功能应该有 

* Preview Kettle当前运行状况和`质量`
* 单独查看某个线路或某张表的下发情况 `时间` & `Error` & `Count`
* Error 单独分析和Email寄送功能
* Reset 下发Status
