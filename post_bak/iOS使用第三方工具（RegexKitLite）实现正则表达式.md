---
title: iOS使用第三方工具（RegexKitLite）实现正则表达式
date: 2016-05-17 20:04:40
categories: iOS
tags: [iOS]
---
iOS应用中，经常要输入数据，就要校验数据的合法性，这时我们很自然的联想到Perl应用中的正则表达式。然而Cocoa支持的正则表达式使用起来相对比较麻烦。这时我们可以使用第三方工具（RegexKitLite）来实现正则表达式。
* 1、下载[RegexKitLite](https://github.com/wezm/RegexKitLite)类库，将RegexKitLite.h/ RegexKitLite.m两个文件添加到您的项目中；
* 2、在您的工程中添加libicucore.dylib frameworks(iOS8之后为tbd后缀)，ARC项目需加-fno-objc-arc；
* 3、在您要校验的数据中使用RegexKitLite，这里假设校验一个电子邮箱

```
NSString *email = @"iMilo@163.com";
NSString *regex = @"\\b([a-zA-Z0-9%_.+\\-]+)@([a-zA-Z0-9.\\-]+?\\.[a-zA-Z]{2,6})\\b";
if ([email isMatchedByRegex:regex]) {
     NSLog(@"通过校验！");
} else {
     NSLog(@"未通过校验，数据格式有误，请检查！");
}
```

说明：查看RegexKitLite源代码，您会发现其实是对NSString的扩展，所以校验的数据必须是NSString类型的。

注意：
* 1、在写正则表达式时：所有的’\’都需要转义，即：’\\’;
