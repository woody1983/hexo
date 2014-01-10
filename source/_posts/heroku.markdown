title: "heroku"
date: 2013-09-05 21:20
tags: [Deploy, Heroku]
---

一开始是在 [Coursera](https://class.coursera.org/startup-001/class/index)上的公开课学到的 留个记录 以后会天天用到的。

###安装命令很简单 Gem包

`gem install heroku`

`$ heroku keys` 可以查看目前主机中存有的key 如果没有你当前主机的话 就用`heroku keys:add`

Heroku只是一个托管平台而已 Code还是要保存在Github上 需要部署的时候部署过去。我还真不知道怎么从Heroku直接拖Code下来。

```
$ heroku create semisconblog
Creating semisconblog... done, stack is cedar
http://semisconblog.herokuapp.com/ | git@heroku.com:semisconblog.git
Git remote heroku added
```
回头再写 Heroku还是有点不熟悉


