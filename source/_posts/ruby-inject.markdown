title: "Ruby inject"
date: 2013-09-08 23:26
categories: Ruby
tags: Ruby
---

很早之前再Ruby China问的一个问题 就是1到10 可以创建多少个3位数不重复的组合

现在有了答案!

``` ruby
(1..10).to_a.combination(3).to_a
```

说实话 这个函数我也没见过 但效果还不错 目测了一把基本是我要的，和数据打交道时间久了就总觉得号码是有规律可言的。上周研习了一下`inject`的用法 感觉可以做一点小东西出来玩一下。

大乐透的前五位数有多少种组合呢？ 呵呵 

``` ruby
(1..35).to_a.combination(5).to_a
```
