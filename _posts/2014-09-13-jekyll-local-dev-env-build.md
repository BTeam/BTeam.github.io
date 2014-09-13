---
layout: post
title: 在win7x64ch下，搭建Jekyll测试环境时，遇到编码问题的解决方案
category: 工具
tags: jekyll environment encoding 
description: 在win7x64ch下，搭建Jekyll测试环境时，遇到编码问题的解决方案 
---

### **引言**
我很喜欢现在这个用jekyll+markdown在github上写博客的框架，非常方便，但每次都要上传到github才能预览甚是不方便。就算现在用了eclipse的egit来管理本地git仓库，但提交一次至少也需要commit后
push一次，提交多了操作成本非常之高（栗子：目前才发了2篇博文我就提交了73次。。。）

其实之前在老系统（win7-32bit中文版）下已经装过一次本地jekyll，知道装完后用localhost:4000来测试页面方便的多。当时装的也非常顺利。于是就屁颠屁颠想利用周末装一下。

此时的我，真的没有想到，这个问题会一直把我从中午困到晚上，导致我今天没有完全计划中的事项，比如整理数据接口文档和晚上去游泳。。。令人欣慰的是，最后终于基本上成功解决了，强迫症的心情也不再为此纠结。在此分享一下过程：

### **痛苦的过程**

先装了一个ruby1.9.2版本，然后自动下载jekyll，装完jekyll serve看到提示编码冲突什么的。一想自己是64位系统，而ruby只有2.0才有64位版，难道是这原因？

于是又换了ruby2.0x64，问题仍在。

于是baidu+google+stackoverflow，一搜网上各种说法都有啊。。。先看到有人说ruby2.0兼容性不好，别装。于是又换回了1.9.2。。。问题仍在。

之后又以为是一些ruby组件兼容性的问题，于是各种卸载组件，换jekyll版本，换组件版本。。。问题仍在。

之后找到stack上有个似乎比较靠谱的说法，在gem/lib/jekyll下有个convertible.rb里，找到这一句：

    self.content = File.read(File.join(base, name))
    
给它加上第二个参数指定编码就行了，变成酱紫：

    self.content = File.read(File.join(base, name), :encoding => "utf-8")

按po主的说法，问题就引刃而解了。同样，在各种引用此论点的中文博客里，问题也都到这里就引刃而解了。“神马？你还在报编码冲突？检查一下你的文件，它们肯定是GBK！把它们转成UTF-8！引刃而解！”

哥这儿就是照着他们的说法做完了还在报编码错误的那种情况。。。更悲剧的是，哥花了半个多小时一个个到notepad++里去检查网页文件编码，结果全是UTF-8。。。为什么还是通不过呢？问题仍在！

到这里我又纠结了，想放弃，因为游泳时间快要到了。。。换了个老版本的jekyll，胡乱又降级了几个依赖库到jekyll的最低要求，根本没用。jekyll开发组的几个人还在踢皮球，说：“你们只要在_config里加一句encoding=utf-8就ok了啊”——根本没用。。。

### **成功解决**

吃完晚饭上来，已经决定游泳改明天了，于是用一个极蠢的办法来排障：

new了一个新的jekyll工程，它自动生成的，发现到这个工程里能成功jekyll serve。于是我想，这不是jekyll不兼容的问题吧。。。心里感觉有了一点曙光。

然后向这个新工程里一个个的增加我博客的文件，从index.html到_layouts到_includes到_posts，每增加一个就开关一次jekyll serve。一路居然都没问题！直到我添加了disqus.html。

然后我就发现，这个文件里面有中文！于是哥把中文删了——我x，居然又能成功启动jekyll serve了！

这TM不还是一个编码问题吗？！

于是哥回到之前那个convertible.rb，查看刚刚改动的那句代码，发现前面某处有这么一行注释

    # Read the YAML frontmatter.

我x（原谅哥连续爆粗口，被这玩意儿折腾了一下午真的会脾气暴躁）—— 不是吐槽formatter会拼错成frontmatter，而是这地方看起来只是处理那个叫_config.yml的yaml格式文件。

于是我grep了一下那整个目录下所有的rb文件，发现调用File.read方法的还有好几处。一处处添加:encode => "utf-8"参数后，终于找到了问题的根源：

    source = File.read(File.join(includes_dir, @file), :encoding => "utf-8")

这里添加后，jekyll serve终于能在包含中文页面的情况下启动。。。泪崩啊。。。看了下各个页面显示也正常了。

但页面里的中文链接还是有问题（似乎被url编码过但没有解码。。。到github上就没这个问题。。。），总之资源图片什么的还是避免中文名吧。ruby神马的语法都不懂，就不瞎改东西了。

总结就是上面两个地方改一下就好了，而不是网上说的只改第一处就好了。

猜测原因是jekyll中_config.yml的配置并未真正影响到jekyll对页面的解析工作（我没有在ruby代码里看到相关的变量，File.read都是不指定编码的）。收拾回家，明天去游泳。（完）