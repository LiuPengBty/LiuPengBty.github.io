---
layout: post
comments: true
categories: iOS
---

# NSPredicate

简述：Cocoa框架中的`NSPredicate`用于查询，原理和用法都类似于SQL中的where，作用相当于数据库的过滤取。
定义(最常用到的方法)：

```
NSPredicate *ca = [NSPredicate prediateWithFormat:(NSString *), ...];
```

Format:

* 比较运算符>,<,==,>=,<=,!=可用于数值及字符串

例：

```
@"number > 100"
```

* 范围运算符: IN、 BETWEEN

例：

```
@"number BETWEEN {1,5}"
@"address IN {'shanghai','beijing'}"
```

* 字符串本身:SELF 

例：

```
@“SELF == ‘APPLE’"
``` 

* 字符串相关：BEGINSWITH、ENDSWITH、CONTAINS

例:

```
@"name CONTAIN[cd] 'ang'"   //包含某个字符串
@"name BEGINSWITH[c] 'sh'"     //以某个字符串开头
@"name ENDSWITH[d] 'ang'"      //以某个字符串结束
注:[c]不区分大小写[d]不区分发音符号即没有重音符号[cd]既不区分大小写，也不区分发音符号。
```

* 通配符：LIKE

例:

```
@"name LIKE[cd] '*er*'"    //*代表通配符,Like也接受[cd].
@"name LIKE[cd] '???er*'"
```

* 正则表达式：MATCHES

例:

```
NSString *regex = @"^A.+e$";   //以A开头，e结尾
@"name MATCHES %@",regex
```


