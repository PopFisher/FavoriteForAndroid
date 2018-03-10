# 本文框架
- 什么是热修复？
- 热修复框架分类
- 技术原理及特点
- Tinker框架解析

&emsp;&emsp;通过阅读本文，你会对热修复技术有更深的认知，本文会列出各类框架的优缺点以及技术原理，文章末尾简单描述一下Tinker的框架结构。

# 一、什么是热修复？
## 正常开发流程

----------
![](/picture/hotfix/1.png)

##  热修复开发流程

----------
![](/picture/hotfix/2.png)

##  热修复优势

----------

![](/picture/hotfix/3.png)

##  修复什么？

----------

![](/picture/hotfix/4.png)

# 二、热修复框架分类
##  现状：百花齐放百家争鸣

----------
![](/picture/hotfix/5.png)

##  简单分类

----------
![](/picture/hotfix/6.png)

##  更合理的分类

----------
![](/picture/hotfix/7.png)

# 三、技术原理及特点


## 3.1 阿里Dexposed -- native解决方案 ##
### 原理：

- 直接在native层进行方法的结构体信息对换，从而实现完美的方法新旧替换，从而实现热修复功能

&emsp;&emsp;他的思想完全来源于Xposed框架，完美诠释了AOP编程，这里用到最核心的知识点就是在native层获取到指定方法的结构体，然后改变他的nativeFunc字段值，而这个值就是可以指定这个方法对应的native函数指针，所以先从Java层跳到native层，改变指定方法的nativeFunc值，然后在改变之后的函数中调用Java层的回调即可。实现了方法的拦截功能。

- 基于开源框架Xposed实现，是一种AOP解决方案
- 只Hook App本身的进程，不需要Root权限

----------
![](/picture/hotfix/8.png)

![](/picture/hotfix/9.jpg)

![](/picture/hotfix/10.png)


### 优点：

- 即时生效
- 不需要任何编译器的插桩或者代码改写，对正常运行不引入任何性能开销。这是AspectJ之类的框架没法比拟的优势；
- 对所改写方法的性能开销也极低（微秒级），基本可以忽略不计；
- 从工程的角度来看，热补丁仅仅是牛刀小试，它真正的威力在于『线上调试』；
- 基于Xposed原理实现的AOP不仅可以hook自己的代码，还可以hook同进程的Android SDK代码，这也就可以让我们有能力在App中填上Google自己挖的坑。

### 缺点：

- Dalvik上近乎完美，不支持ART（需要另外的实现方式），所以5.0以上不能用了；
- 最大挑战在于稳定性与兼容性，而且native异常排查难度更高；
- 由于无法增加变量与类等限制，无法做到功能发布级别；

### 相关链接：

- 文章：[https://www.zhihu.com/question/31894163](https://www.zhihu.com/question/31894163 "如何看待阿里开源的dexposed框架?")
- 文章：
[http://www.wjdiankong.cn/android%E4%B8%AD%E5%85%8Droot%E5%AE%9E%E7%8E%B0hook%E7%9A%84dexposed%E6%A1%86%E6%9E%B6%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90%E4%BB%A5%E5%8F%8A%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0/](http://www.wjdiankong.cn/android%E4%B8%AD%E5%85%8Droot%E5%AE%9E%E7%8E%B0hook%E7%9A%84dexposed%E6%A1%86%E6%9E%B6%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86%E8%A7%A3%E6%9E%90%E4%BB%A5%E5%8F%8A%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0/ "Android中免Root实现Hook的Dexposed框架实现原理解析以及如何实现应用的热修复")
- Dexposed源码：[https://github.com/alibaba/dexposed](https://github.com/alibaba/dexposed)
- Xposed源码：[https://github.com/rovo89/Xposed](https://github.com/rovo89/Xposed)

## 3.2 阿里AndFix -- native解决方案 ##
### 原理： ###

- 与Dexposed一样都基于开源框架Xposed实现，是一种AOP解决方案

### 优点： ###

- 即时生效
- 支持dalvik和art（AndFix supports Android version from 2.3 to 7.0, both ARM and X86 architecture, both Dalvik and ART runtime, both 32bit and 64bit.）
- 与Dexposed框架相比AndFix框架更加轻便好用，在进行热修复的过程中更加方便了

### 缺点： ###

- 面临稳定性与兼容性问题
- AndFix不支持新增方法，新增类，新增field等

### AndFix（Dexpsed）框架不稳定的原因（痛点）

![](/picture/hotfix/11.png)

![](/picture/hotfix/12.png)

### 相关链接： ###

- 文章：[https://zhuanlan.zhihu.com/p/23935568](https://zhuanlan.zhihu.com/p/23935568 "Android热修复框架AndFix原理解析及使用")
- AndFix源码：[https://github.com/alibaba/AndFix](https://github.com/alibaba/AndFix)


## 3.3 QQ空间--Dex插桩方案（大众点评的Nuwa参考其实现并开源） ##
###原理：###

- 原理是Hook了ClassLoader.pathList.dexElements[]。因为ClassLoader的findClass是通过遍历dexElements[]中的dex来寻找类的。当然为了支持4.x的机型，需要打包的时候进行插桩。
- 越靠前的Dex优先被系统使用，基于类级别的修复

![](/picture/hotfix/13.jpg)

###优点：###

- 不需要考虑对dalvik虚拟机和art虚拟机做适配
- 代码是非侵入式的，对apk体积影响不大

###缺点：###

- 需要下次启动才会生效
- 最大挑战在于性能，即Dalvik平台存在插桩导致的性能损耗，Art平台由于地址偏移问题导致补丁包可能过大的问题
- 虚拟机在安装期间为类打上CLASS_ISPREVERIFIED标志是为了提高性能的，我们强制防止类被打上标志是否会影响性能？这里我们会做一下更加详细的性能测试．但是在大项目中拆分dex的问题已经比较严重，很多类都没有被打上这个标志。

###插桩方案性能上的痛点：###

![](/picture/hotfix/14.png)

###相关链接：###

- 文章：[https://zhuanlan.zhihu.com/magilu/20308548](https://zhuanlan.zhihu.com/magilu/20308548 "安卓App热补丁动态修复技术介绍")
- 文章：[http://blog.csdn.net/sbsujjbcy/article/details/50812674](http://blog.csdn.net/sbsujjbcy/article/details/50812674 "Android 热修复Nuwa的原理及Gradle插件源码解析")
- Nuwa源码：[https://github.com/jasonross/Nuwa](https://github.com/jasonross/Nuwa)
- HotFix源码：[https://github.com/dodola/HotFix](https://github.com/dodola/HotFix)
- DroidFix源码：[https://github.com/bunnyblue/DroidFix](https://github.com/bunnyblue/DroidFix)

## 3.4 美团Robust -- Instant Run 热插拔原理 ##
###原理：###

- Robust插件对每个产品代码的每个函数都在编译打包阶段自动的插入了一段代码，插入过程对业务开发是完全透明
- 编译打包阶段自动为每个class都增加了一个类型为ChangeQuickRedirect的静态成员，而在每个方法前都插入了使用changeQuickRedirect相关的逻辑，当 changeQuickRedirect不为null时，可能会执行到accessDispatch从而替换掉之前老的逻辑，达到fix的目的。

![](/picture/hotfix/17.png)

###优点：###

- 几乎不会影响性能（方法调用，冷启动）
- 支持Android2.3-8.x版本
- 高兼容性（Robust只是在正常的使用DexClassLoader）、高稳定性，修复成功率高达99.9%
- 补丁实时生效，不需要重新启动
- 支持方法级别的修复，包括静态方法
- 支持增加方法和类
- 支持ProGuard的混淆、内联、优化等操作

###缺点：###

- 代码是侵入式的，会在原有的类中加入相关代码
- so和资源的替换暂时不支持
- 会增大apk的体积，平均一个函数会比原来增加17.47个字节，10万个函数会增加1.67M。
- 会增加少量方法数，使用了Robust插件后，原来能被ProGuard内联的函数不能被内联了

###相关链接：###

- 美团技术文章：[https://tech.meituan.com/android_robust.html](https://tech.meituan.com/android_robust.html "Android热更新方案Robust")
- 美团技术文章：[https://tech.meituan.com/android_autopatch.html](https://tech.meituan.com/android_autopatch.html "Android热更新方案Robust开源，新增自动化补丁工具")
- Robust源码：[https://github.com/Meituan-Dianping/Robust](https://github.com/Meituan-Dianping/Robust)


## 3.5 微信Tinker ##

###原理：###

- 服务端做dex差量，将差量包下发到客户端，在ART模式的机型上本地跟原apk中的classes.dex做merge，merge成为一个新的merge.dex后将merge.dex插入pathClassLoader的dexElement，原理类同Q-Zone，为了实现差量包的最小化，Tinker自研了DexDiff/DexMerge算法。Tinker还支持资源和So包的更新，So补丁包使用BsDiff来生成，资源补丁包直接使用文件md5对比来生成，针对资源比较大的（默认大于100KB属于大文件）会使用BsDiff来对文件生成差量补丁。

![](/picture/hotfix/15.png)

![](/picture/hotfix/16.png)

###优点：###

- 支持动态下发代码
- 支持替换So库以及资源

###缺点：###

- 不能即时生效，需要下次启动

###Tinker已知问题：###

- Tinker不支持修改AndroidManifest.xml，Tinker不支持新增四大组件(1.9.0支持新增非export的Activity)；
- 由于Google Play的开发者条款限制，不建议在GP渠道动态更新代码；
- 在Android N上，补丁对应用启动时间有轻微的影响；
- 不支持部分三星android-21机型，加载补丁时会主动抛出"TinkerRuntimeException:checkDexInstall failed"；
- 对于资源替换，不支持修改remoteView。例如transition动画，notification icon以及桌面图标。

###Tinker性能痛点：###

- Dex合并内存消耗在vm head上，容易OOM，最后导致合并失败。
- 如果本身app占用内存已经比较高，可能容易导致app本系统杀掉。

###相关链接：###

- 文章：[http://geek.csdn.net/news/detail/104000](http://geek.csdn.net/news/detail/104000 "微信 Tinker 的一切都在这里，包括源码")
- 文章：[Tinker官方文章](https://github.com/Tencent/tinker/wiki "wiki")
- Tinker源码：[https://github.com/Tencent/tinker](https://github.com/Tencent/tinker)

## 3.6 阿里Sophix ##

###原理(双剑合璧)：###
![](/picture/hotfix/18.png)

###优化Andfix（突破底层结构差异，解决稳定性问题）：###

**Andfix底层ArtMethod结构时采用内部变量一一替换，倒是这个各个厂商是会修改的，所以兼容性不好。**

![](/picture/hotfix/20.png)

----------

**Sophix改变了一下思路，采用整体替换方法结构，忽略底层实现，从而解决兼容稳定性问题。**

![](/picture/hotfix/19.png)

###突破QQ和Tinker的缺陷###

**QQ和Tinker的缺陷**

![](/picture/hotfix/21.png)

----------

**Sophix对dex的解决方案**

- Dalvik下采用阿里自研的全量dex方案：不是考虑把补丁包的dex插到所有dex前面（dex插桩），而是想办法在原理的dex中删除（只是删除了类的定义）补丁dex中存在的类，这样让系统查找类的时候在原来的dex中找不到，那么只有补丁中的dex加载到系统中，系统自然就会从补丁包中找到对应的类。
- Art下本质上虚拟机以及支持多dex的加载，Sophix的做法仅仅是把补丁dex作为主dex（classes.dex）而已，相当于重新组织了所有的dex文件：把补丁包的dex改名为classes.dex，以前apk的所有dex依次改为classes2.dex、classes3.dex ... classesx.dex，如下图所示。

![](/picture/hotfix/22.png)

###资源修复另辟蹊径###
**常用方案（Instant Run技术）：这种方案的兼容问题在于替换AssetManager的地方**
![](/picture/hotfix/23.png)

**Sophix资源修复方案**
![](/picture/hotfix/24.png)

###SO修复另辟蹊径###
![](/picture/hotfix/25.png)

# 四、Tinker框架解析
&emsp;&emsp;之所以只贴了Tinker的代码框架，是因为目前开源的方案中是最好的，当然除了Robust。
## 代码结构
![](/picture/hotfix/26.png)

## 修复流程
![](/picture/hotfix/27.png)

**这里后续再补一个详细的源码分析，敬请期待**

# 对比图（来自不同的地方）
## 来自Tinker的对比
![](http://images2017.cnblogs.com/blog/823551/201801/823551-20180119173848396-1135737571.png)

## 来自Sophix的对比
![](/picture/hotfix/28.png)

## 来自[蘑菇街 Android 热修复探索之路](https://mp.weixin.qq.com/s/GuzbU1M1LY1VKmN7PyVbHQ)
![](http://images2017.cnblogs.com/blog/823551/201801/823551-20180119181035646-1613402870.png)

## 其他文章 ##

浅谈Android热修复：

- [http://blog.csdn.net/caihongdao123/article/details/52051799](http://blog.csdn.net/caihongdao123/article/details/52051799)

- Android 热修复专题：支付宝、淘宝、微信、QQ空间、饿了么、美丽说蘑菇街、美团大众点评方案集 [https://zhuanlan.zhihu.com/p/25863920](https://zhuanlan.zhihu.com/p/25863920 "Android 热修复专题：支付宝、淘宝、微信、QQ空间、饿了么、美丽说蘑菇街、美团大众点评方案集合")
![](http://images2017.cnblogs.com/blog/823551/201801/823551-20180119174807209-784087153.jpg)

# 总结

&emsp;&emsp;如果不考虑增大apk的体积，选择Robust是最稳定的，否则的话选择Tinker是一个不错的方案。虽然阿里Sophix横空出世，但是它不开源，而且商业收费，所以一般不是很赚钱的app选择收费的可能就很小了。不过它确实各方面都做了大量的优化，本文中的很多知识点也来源于阿里的《Android热修复技术原理.pdf》一书，本书值得一读，里面就是基于Sophix框架来编排的。