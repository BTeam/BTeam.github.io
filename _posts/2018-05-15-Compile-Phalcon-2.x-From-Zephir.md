---
layout: post
title: 定制属于你自己的Phalcon扩展：从zep文件编译安装Phalcon-2.x
category: 技术
tags: 原创 php phalcon compile
description: customize your own phalcon - compile phalcon 2.x version from the zephir source code
---

原创文章，转载请保留或注明此出处：[http://blog.sunnygo.top/2018/05/15/Compile-Phalcon-2.x-From-Zephir.html](http://blog.sunnygo.top/2018/05/15/Compile-Phalcon-2.x-From-Zephir.html)。

目前公司php服务层大多数项目基本上都用的Phalcon 2.0.13版本。从刚开始用就遇到一些框架固有的缺陷，但因为Phalcon是以php扩展形式安装的框架，感觉修改比较麻烦，就一直没管，而是业务层面绕过框架缺陷来规避问题。

这次遇到一个问题，Phalcon自有的Redis缓存组件，会强制存一份key的副本到_PHCR集合里，而这个集合是没有TTL的，得不到释放（深入的原因本文不再赘述），导致使用一段时间后被这些无用信息塞满了内存。想解决这个问题，在网上调研后，总结了几个可能的解决方案：

1. 定时脚本清除Redis中_PHCR集合内元素：
    * 对每个Redis应用都要设置这个维护脚本，复杂度 = Redis服务器数 * namespace数(默认是_PHCR但实际可以自定义)。
2. 通过在项目里继承来扩展Phalcon\Cache\Redis\Backend：
    * 需要在每个项目里"部署"这个补丁，麻烦。而且觉得通过PHP层来解决框架问题，会破坏Phalcon作为二进制扩展的效率优势。
3. 升级Phalcon
    * 这个缺陷在下一个版本里已经得到了修复，但是不太想为了这么一个小改动而升级。我们用的2.0.13是Phalcon-v2最后一个稳定版，下一个正式版开始就是v3了，升级的代价是大量的changelog需要check。因为升级涉及好多项目，如果升级带来较多接口或者内部行为的改变，那么对已有项目风险比较大。
同时如果我们只能通过升级来修复缺陷，后面会步步掣肘（比如如果必须升级到Phalcon-v4，php还必须升级到v7，我们目前是5.5，升级php同样可能带来项目不兼容的问题）。
4. ~~放着不管~~（误） 修改Phalcon源码

综上看来，修改并重新编译Phalcon源代码才是最好的出路。而且有能力修改Phalcon源码后，我们还可以在底层定制适合我们业务的功能，提高程序处理效率。

### 具体操作

从[Github](https://github.com/phalcon/cphalcon)上clone一份Phalcon源码，检出想要的分支代码。

项目根目录主要结构如下图所示：

![](/public/img/phalcon_dir.png)

Phalcon目前的源码是用一种中间件语言语言zephir来开发的，zephir源码存在根目录/phalcon下，后缀.zep的文件就是。Phalcon完整的编译机制应该是从zep代码编译成c代码，再从c代码编译成php扩展。/ext/phalcon目录则存放被zephir编译后的c文件。

按官网教程，下载项目后直接执行/build下的install脚本来安装。然而经试验我们修改zep源码后install是无法生效的，猜测install的逻辑应该是从/ext/phalcon下的C代码直接编译成扩展。

### 完整的编译步骤

0. 参考[此贴](https://forum.phalconphp.com/discussion/17091/recompiling-phalcon-from-updated-zep-files)

1. 需要安装的依赖清单：
    * re2c
    * Zephir Parser
    * g++ >= 4.4 | clang++ >= 3.x | vc++ >= 11
    * GNU make >= 3.81
    * automake
    * PHP development headers and tools
2. 注意Zephir Parser是一个php扩展（这里可以猜到，Zephir编译php扩展实际上底层应该还是通过phpize或者php-cli来进行的）

    安装

    ```bash
    git clone https://github.com/phalcon/php-zephir-parser
    cd php-zephir-parser
    sudo ./install
    ```

    将phalcon_parser.so这个扩展加入php.ini

    ```bash
    extension=zephir_parser.so
    ```

3. 安装Zephir。

    ```bash
    git clone https://github.com/phalcon/zephir
    cd zephir
    ./install -c
    ```

    注意如果像我们使用的是2.0.x版本，官方release不是使用最新版Zephir来编译的，最好还是跟官方保持一致。(编译用的Zephir版本号从CHANGELOG.md里可以查到)。修改如下：

    ```bash
    git clone https://github.com/phalcon/zephir
    cd zephir
    git checkout 0.9.2
    ./install -c
    ```

4. 切换到phalcon源码根目录，执行zephir build。编译要一定时间。这里有两个注意点：
    * 根编译其他扩展一样，保证php -v的版本跟要挂载phalcon的php版本一致。（貌似可以通过加参数来指定php可执行文件的位置，这个自行百度）
    * 可能会报一个错误php内存限制太小（如果你用php-fpm模式，内存限额不会太大），需要把内存临时改大。

5. 执行完默认会自动拷贝phalcon.so到php安装目录下的扩展目录下。如果路径不对，也可以从/ext/modules/phalcon.so自行拷贝。

6. 还原php.ini配置，zephir_parser.so可以去掉，如果之前调大了内存限额可以改回来。重启php-fpm（如果有需要）

特意整理了下，因为网上关于从zep源码重新编译Phalcon的资料比较少。原创文章，转载请注明出处，谢谢配合。