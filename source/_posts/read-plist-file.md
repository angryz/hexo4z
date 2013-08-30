title: 读取 .plist 文件内的数据
date: 2013-07-31 20:00:00
tags: 
- ios
- objective-c
categories: Develop
---

有时会将静态数据存放到 .plist 文件中，就需要读取 .plist 文件作为数据源。
如果对 .plist 文件打开方式选择Source Code，你会看见它其实是一个xml文件。

读取 .plist 文件的方式如下，

```objc
NSString *plistPath = [[NSBundle mainBundle] pathForResource:@"listFileName" ofType:@"plist"];
NSArray *array = [[NSArray alloc] initWithContentsOfFile:plistPath]; //读取到数组
NSDictionary *dictionary = [[NSDictionary alloc] initWithContentsOfFile:plistPath]; //读取到字典对象
```

就这么简单。
另外，从 NSDictionary 中拿数据的方式如下,

```objc
NSDictionary *userDict = [dictionary objectForKey:@"Users"];
self.userNameLbl.text = [NSString stringWithFormat:@"%@", [userDict objectForKey:@"Name"]];
self.userNumberLbl.text = [NSString stringWithFormat:@"%@", [userDict objectForKey:@"Number"]];
```

