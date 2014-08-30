---
layout: post
title: 浅谈用Promise模式来简化Javascript异步编程
category: 技术
tags: javascript
description: 浅谈用Promise模式来简化Javascript异步编程
---

### Javascript中的异步函数

	

### 异步函数所带来的烦恼

>实际上，“烦恼”这个说法是有违公平的。异步函数让像Javascript这样工作在单线程下的语言，能够更好地利用起有限的时间片。因此有一些优化规范建议将大段的同步函数，切成多个小片段，每个小片段用setTimeout(...,0)调用之。
	
引用一段工作在其他栈上的网友(win32 C)对使用异步函数作出的评价：“ 异步调用原理并不复杂，但实际使用时容易出莫名其妙的问题，特别是不同线程共享代码或共享数据时容易出问题，编程时需要时时注意是否存在这样的共享，并通过各种状态标志避免冲突。”
	
>虽说道理都是相通的，但异步函数带来的困扰在Javascript中看起来似乎更为严重。根本原因在上面已经说了，Javascript中使用异步函数的机会比其他语言要高很多————因为它是单线程的。





### 什么是Promise模式

Content

### 用Promise模式来简化Javascript异步编程

Content