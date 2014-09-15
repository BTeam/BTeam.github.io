---
layout: sec
title: 淘学岛新项目数据接口草案
category: 淘学岛
tags: doc
description: 淘学岛新项目数据接口草案
---

## 1. 首页

### 1.0 公用

#### 1.0.1 省份

读数据库：省份表 	省份名-省份id

#### 1.0.2 学校信息对象

格式如下，其他字段待补充：

    {
    	name : String, //学校名
    	...
    }

### 1.1 高考成绩查询

输入：

* location int/String 省份id或url编码后的省份名称（下同）

输出：

    {
    	admission : {
    		human : {
				pro1 : int, //本1
				pro2 : int, //本2
			},
			science : {
				pro1 : int,
				pro2 : int,
			},
		},
		schoolInfo : {
    		num : {
				in985 : int,
				in211 : int，
				key : int,		//重点大学
				normal : int，		//普通本科
				independ : int,		//本科独立
				civilian : int,		//民办本科
				academy : int,		//专科
				civilianAcdm : int	//专科民办
    		}
    	}
    }

### 1.2 地区分数线查询

输入：

* year int 年份
* location 同1.1
* majorType String human:文科 science:理科

输出：

    {
		line : {
			human : int //文科分数线（可选）
			science : int //理科分数线（可选）
		}
	}

### 1.3 高校招生分数线查询

输入：

* myLoc 同1.1 location
* schoolLoc 同1.1 location
* majorType 同1.2
* school String/id 学校名/id

输出：

    {
		line : { //字段都可选
			pro : int, //本科线
			pro1: int, //本1
			pro2: int,
			pro3: int,
			academy : int //专科
		}
	}

### 1.4 估分选学校

输入：

* score int //估分（设计里有0.5？可能为1位精度float）
* location 同1.1 
* examCate	String  human:文科  science:理科  complex:综合
* schoolLoc	同1.1 location
* priority int  0:提前录取  1:本一  2:本二  3:本三
* stategy  int  0:稳健型  1:冒险型

输出：
	
    [
		学校信息对象_1, //见 1.0.2
		学校信息对象_2,
	    ... //more schools' info
    ]
    
### 1.5 院校查询

输入：

* school String //院校名称
* location 同1.1
* type String/int 院校类型，啥东西？？？
* rank Sting/int  院校性质，啥东西？？？

输出：

	学校信息对象 //见1.0.2

### 1.6 院校分数线查询

输入：

* major String //专业名称
* location 同1.1
* year int 年份
* examCate 同1.4
* school String //院校名称

输出：

    {
		line : int //分数线
	}

### 1.7 专业查询

前置： 获得专业大小类

* 读数据库？

输入：

* 专业名称  String
* 专业大类  String/int  大类名/id
* 专业小类  String/int  小类名/id

输出： ？？？ **这里输出貌似是到专业榜？是的话见1.9**

### 1.8 推荐院校

输入：

* 用户id/cookie标示符(可选)

输出：

    {
    	suggestion : [ //推荐学校，至少3个 
    					{
    						学校名_1,
    						校徽_1,
    					},
    					{
    						学校名_2,
    						校徽_2,
    					},
    					{
    						学校名_3,
    						校徽_3,
    					},
    					...
    				],
    	hot_seach : [ //热搜学校，多个
    					{
    						学校名_1,
    						搜索人数_1,
    					},
    					...
    				],
    	hot_signup :[ //报考最热，多个
    					{
    						学校名_1,
    						报考人数_1,
    					},
    					...
    				]
    }

### 1.9 专业榜

输入：

* 未知？？？

输出：

    [
    	学校名_1， //学校名
    	学校名_2，
    	...
    ]

### 1.10 资讯

人工编辑

### 1.11 高考进程（最上面那个进度条）

人工录入当年数据？
