title: "关于HC-SR04 超声波传感器"
date: 2013-05-27 00:21
categories: Arduino
tags: Arduino
---

关于HC-SR04 超声波传感器

淘宝上面最便宜的SRF-04超声波传感器，有四个脚：5v电源脚（Vcc），触发控制端（Trig），接收端（Echo），地端（GND） 我的是HC-SR04

模块工作原理：

1.采用IO触发测距，给至少10us的高电平信号；
2.模块自动发送8个40KHz的方波，自动检测是否有信号返回；
3.有信号返回，通过IO输出一高电平，高电平持续的时间就是超声波从发射到返回的时间．测试距离=(高电平时间*声速(340m/s))/2;
pulseIn函数用于读取引脚脉冲的时间长度，脉冲可以是HIGH或LOW。如果是HIGH，函数将先等引脚变为高电平，然后开始计时，一直到变为低电平为止。返回脉冲持续的时间长短, 单位为ms。如果超时还没有读到的话, 将返回0。

Demo:

发一个10ms的高脉冲去触发TrigPin

``` c
digitalWrite(TrigPin, LOW);
delayMicroseconds(2); //使发出发出超声波信号接口低电平2μs
digitalWrite(TrigPin, HIGH);
delayMicroseconds(10); //使发出发出超声波信号接口高电平10μs，这里是至少10μs
digitalWrite(TrigPin, LOW); //保持发出超声波信号接口低电平
```

####距离换算

* cm = pulseIn(EchoPin, HIGH) / 58.0; //算成厘米
* cm = (int(cm * 100.0)) / 100.0; //保留两位小数

原理理解以后 尽量使用arduino官方推荐的SR04超声波传感器类库

###读出脉冲时间 

``` c
int distance = pulseIn(inputPin, HIGH);
distance= distance/58; // 将脉冲时间转化为距离（单位：厘米）
```
