---
title: 如何理解.h 和 .m 文件中的同一个@interface
date: 2016-05-17 20:04:46
categories: iOS
tags: [iOS]
---
在XCode中创建一个新的文件，会在.h和.m文件中自动创建两个几乎完全一样的@interface。比如：创建一个UIViewController的实例：

在.h文件中：

```
#import <UIKit/UIKit.h>

@interface UUTaskDetailController : UIViewController

@end
```

在.m文件中：

```
#import "UUTaskDetailController.h"

@interface UUTaskDetailController ()

@end
```

乍一看，除了高亮的部分外，其他的都一样。 你可以在.h文件中，声明@property和 method；也可以在.m文件中声明这些@property和 method。更为有意思的是，你甚至可以在storyboard中，通过拖拽方式，将一个IBOutlet 拖拽到.h或.m文件中。  
尽管可以随意操作，单从本质上讲，还是存在较大差异的。接下来，让我们看看它们背后的故事。  
如有你有public和private的概念，你可以理解为：.h 文件声明的@property，是公共的，是可以被其他的.m文件访问的；而在.m文件中声明的@property，是私有的，只能在该.m文件中使用。  
再进一步想想，也容易理解。   
因为.h文件可以被其他.m文件#import。自然就可以被其他.m文件访问；而在.m文件中所声明的，其实就是一个static的变量或方法，自然不能被其他文件访问。

小结：  
弄清楚了.h与.m文件的差别后，你完全可以根据自己的意愿，随心所欲地使用。你声明的property，如果不想被其他文件调用，那就声明在.m文件好了；如果想让其他文件调用呢，当然要声明在.h文件中。
