title: 让UILabel根据内容自动调整大小
date: 2013-08-01 22:17:00
tags:
 - ios
 - objective-c
categories: develop
---

### 固定__宽度__的UILabel让内容自动换行并根据内容变更__高度__

```objc
NSString *str = @"UILabel中显示的内容，达到UILabel设置的最大宽度时，自动换行";
UIFont *font = [UIFont systemFontOfSize:14]; //内容采用的字体大小

CGFloat height = [str sizeWithFont:font constrainedToSize:CGSizeMake(280, CGFLOAT_MAX) lineBreakMode:NSLineBreakByWordWrapping].height; //根据文本内容计算高度


UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 10, 280, height)]; //根据计算的高度创建UILabel实例
label.font = font;
label.numberOfLines = 0;  //必须定义这个属性，否则UILabel不会换行
label.textColor = RGBCOLOR(102, 153, 102); //内容文本颜色
label.textAlignment = NSTextAlignmentLeft;  //内容文本对齐方式
label.text = str;

[self.view addSubview:label]; 
```

***

### __单行__显示的UILabel根据内容变更__宽度__

```objc
NSString *str = @"UILabel中单行显示的内容，UILabel将会调整宽度为本文长度";
UIFont *font = [UIFont systemFontOfSize:14]; //内容采用的字体大小

CGFloat width = [str sizeWithFont:font constrainedToSize:CGSizeMake(CGFLOAT_MAX, 20)].height; //计算单行显示时文本所需的宽度

UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 10, width, 20)]; //根据计算的宽度创建UILabel实例
label.font = font;
label.textColor = RGBCOLOR(102, 153, 102); //内容文本颜色
label.textAlignment = NSTextAlignmentLeft;  //内容文本对齐方式
label.text = str;

[self.view addSubview:label];
```
