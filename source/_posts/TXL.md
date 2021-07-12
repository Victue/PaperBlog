---
title: The Txl Programming Language
author: Li Pie
categories: notes
tags: 
- Txl
- Programming Language
date: 2021-07-10
---



每一个 TXL程序分三个操作阶段:分析阶段,转换阶段和解析输出阶段。

- 分析器读入整个输人,根据词法对输人文本进行标记( tokenize) ,并根据输入语法由标记生成分析树,即抽象语法树(AST)

- 转换规则对输入分析树进行实际变换

- 变换后的分析树由解析器中序遍历输出文本，将原始输入文本解析为结构树

![image-20210707183112488](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210707183112488.png)





通过固定的**语法规则**和**转换规则**将原语言转换为TXL语言

![image-20210707183556107](C:\Users\hasee\AppData\Roaming\Typora\typora-user-images\image-20210707183556107.png)



```
define program
	[expression]
end define
```

