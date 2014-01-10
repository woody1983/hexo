title: "About Blocks With Objective-C"
date: 2013-08-15 11:38
categories: IOS
tags: [IOS, Objective-C]
---

#### 一般的做法
``` objective-c
if(someBOOLVariable){
    NSLog(@"Hello from inside the if statement");
}//如果someBOOLVariable是YES {}里的代码将会被执行
``` 

其实我们想的是能够括号里的代码能够脱离任何特定功能 比如触发条件
Blocks是一组代码 意味着 他可以被传递到方法或像NSArray或NSDictionary中集合

 #### ^ {}块可以创建
``` objective-c
^{
    NSLog(@"Hello from inside the block");
};

^LogMessage = ^ {
    NSLog(@“你好块里面的”);
};
``` 
<!-- more -->
###如果不想其返回任何值 就加一个void
``` objective-c
void ^logMessage = ^{
    NSLog(@"Hello from inside the block");
};
``` 
##练习- 创建一个块  不接受和不返回任何信息
``` objective-c
void (^myFirstBlock)(void) = ^{
    NSLog(@"Hello from inside the block");
};

myFirstBlock();
``` 
##Blocks with arguments
``` objective-c
^(NSUInteger num1, NSUInteger num2){
    NSLog(@"The sum of the numbers is %lu", num1 + num2);
};

void (^sumNumbers)(NSUInteger, NSUInteger) = ^(NSUInteger num1, NSUInteger num2){
    NSLog(@"The sum of the numbers is %lu", num1 + num2);
};

/*
 sumNumbers(45, 89);
 sumNumbers(18, 56);
 */

void (^logCount)(NSArray *) = ^(NSArray *array){
    NSLog(@"There are %lu objects in this array", [array count]);
};

logCount(@[@"Mr.", @"Higgie"]);
logCount(@[@"Mr.", @"Jony", @"Ive", @"Higgie"]);
/*
 challenge[2]: There are 2 objects in this array
 challenge[2]: There are 4 objects in this array
 */
``` 
###练习 
``` objective-c
void (^myFirstBlock)(NSString *) = ^(NSString *str1){
    NSLog(@"%@ from inside the block",str1);
};

myFirstBlock(@"Hello");
myFirstBlock(@"World");
``` 

## Enumerate with blocks
 绝大多数的时候，你会不会块分配给一个变量，调用它自己。相反，你会传递一个块作为参数的消息，像`enumerateObjectsUsingBlock：NSArray`的消息。
 
 `enumerateObjectsUsingBlock：`可以用来作为快速枚举的替代品，因为它基本上是同样的事情：执行一个代码块，在一个数组中的每个对象。但是，而不是使用特殊的语法，它使用正常短信的发送和块。
 

``` objective-c
NSArray *newHats = @[@"Cowboy", @"Conductor", @"Baseball",
@"Beanie", @"Beret", @"Fez"];

for (NSString *hat in newHats) {
    NSLog(@"Trying on hat %@", hat);
}



void (^enumeratingBlock)(NSString *, NSUInteger, BOOL *) =
^(NSString *word, NSUInteger index, BOOL *stop){
    NSLog(@"%@ is a funny word", word);
};

[funnyWords enumerateObjectsUsingBlock:enumeratingBlock];



NSArray *newHats = @[@"Cowboy", @"Conductor", @"Baseball",
@"Beanie", @"Beret", @"Fez"];
void (^enumeratingBlock)(NSString *, NSUInteger, BOOL *) =
^(NSString *hat, NSUInteger index, BOOL *stop){
    NSLog(@"%@ is a funny word", hat);
};

[newHats enumerateObjectsUsingBlock:enumeratingBlock];
```  
