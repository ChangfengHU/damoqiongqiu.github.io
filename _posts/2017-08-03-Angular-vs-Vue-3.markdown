---
layout: post
title:  "Vue从Angular里面抄了哪些东西？"
author: "大漠穷秋"
comments: true
date:   2017-08-04 07:10:11 +0800
category: "web前端"
published: true
excerpt: Vue从Angular里面抄了哪些东西？
---

大家都知道，Vue有大量的设计思想（甚至代码）是从Angular(JS)里面抄过来的。在Vue的官方文档里面也大量出现了用Angular(JS)来类比解释的文本。当然，Vue一般都会说自己是“借鉴”。

那么，Vue里面到底有多少东西是抄袭的Angular呢？本文将会做出详细的列举和解释。

### 1.双向数据绑定
“双向数据绑定”是现代前端框架必备的特性，也是8年前AngualrJS 1.x出来的时候最受开发者欢迎的特性，之一。

“双向数据绑定”的设计思想最早是在微软的.NET实现的，2009年，AngularJS 1.x第一个把这种思想应用到了前端开发领域。现在，几乎所有前端开发框架都接受了“双向数据绑定”的设计思想，当然也包括一直不断抄袭的Vue。

那么问题就来了：

- 什么是“双向数据绑定”？
- 双向数据绑定有什么优点？
- 要实现双向数据绑定有什么技术难点？
- AngularJS 1.x里面是如何实现双向数据绑定的？
- Angular新版本里面是如何实现双向数据绑定的？
- Vue里面是如何实现双向数据绑定的？

我尽量用人话来解释以上问题（一边解释案例一边回答，没有顺序）。

假设你有这样的一个非常简单的登录表单：

<img src="{{ "/assets/img/2017-08-04/login.png" | relative_url }}" alt="login-form" width="100%" class="img-thumbnail rounded"/>

这个表单里面有2个输入项，你最终想要的数据结构是这样的：

    {userName:'damoqiongqiu',password:'123456'}

在没有双向数据绑定的日子里，以jQuery为例，你需要自己去一个一个获取输入项的值，然后再整理成一个JS对象，就像这样：

    var userName=$("#userName").val();
    var password=$("#pwd').val();
    var user={userName:userName,password:password};

但是这样会有一个很明显的问题，如果输入项的数量非常多，这种取值和赋值的代码很快就会变成长长的“面条”。在实际的项目开发中，尤其是那种后台管理型的系统，经常会出现大量的输入项，比如下面这个典型的界面：

<img src="{{ "/assets/img/2017-08-04/complex-form.png" | relative_url }}" alt="complex-form" width="100%" class="img-thumbnail rounded"/>

而且你不要忘了，当用户需要编辑一条数据记录的时候，你又不得不再写一遍赋值操作，就像这样：

    var user=$.ajax({...})                  //到server端去取一条记录
    $("#userName").val(user.userName);
    $("#pwd').val(user.password);           //真实应用一般不会传输密码字段，这里只是示例

很明显，一直用人肉的方式去写这么多取值和赋值代码是非常傻的。这时候，大家都会有一个很自然的想法，如果有一种底层机制，能把这么多输入项“自动绑定”到一个JS对象上就好了！

当用户在表单里面输入了新内容的时候，自动把值同步到JS对象上；当JS对象上的值被修改之后，表单界面自动刷新，而不再需要我们手动去写那么多赋值语句---这就是“双向数据绑定”最简单的解释。

<img src="{{ "/assets/img/2017-08-04/binding-2.png" | relative_url }}" alt="data-binding-2" width="100%" class="img-thumbnail rounded"/>

如果真的实现了这样一种“绑定机制”，你会发现，上面那种“面条”代码全部都会被消除掉，前端代码量会突然被大幅度简化。从这个角度说，“双向数据绑定”是目前评判一款前端框架是否优秀的核心要素。

思想看起来很不错，但是如果你真正去实现的话，就会遇到成吨的麻烦，可能这也是一直以来其它前端框架不去实现“双向数据绑定”的原因。因为要实现“双向数据绑定”，必须实现一个叫做“脏值检测”（也有人翻译成“变更检测”）的机制。而“脏值检测”的实现是比较难的，典型的问题列举如下：

    - 如何才能检测到某个JS对象的值被修改了：JS对象只是POJO，而且不能派发事件（最近Google的几个人把Object.observe也从ES7草案里面删掉了），当JS对象被修改了之后，框架如何才能检测到这种改动呢？
    - 脏值检测的效率问题：对于一个超大的JS对象（实际上是巨大的树形结构），假如非常深层次的某个叶子节点被修改了，你将会被迫进行递归比较。虽然当前JS引擎的执行效率已经足够快，但是这种递归操作如果非常频繁，运行效率会显著下降。

还有一些琐碎的细节问题这里不一一列举，后面写一篇全新的文章来解释。

所以你可以发现，想要实现一套稳定健壮“双向绑定系统”并不是一件轻松的事情。那么，对于以上典型的2个问题，早期的AngularJS 1.x是如何解决的呢？

    - 如何才能检测到某个JS对象的值被修改了：AnguarJS 1.x里面给了一个很粗暴的实现来解决这个问题，AngularJS 1.x会为每一个进行数据绑定的HTML标签创建一个watcher。然后，当发生任何可能会导致数据模型被修改的情况下，就会启动一轮$digest循环，把所有watcher全部运行一遍，看看有没有东西需要自己更新（目前Vue的实现思想与此类似，它底层使用了Object.defineProperty）。AngularJS 1.x在这个地方还有一个实现细节也非常糟糕，它为了保证数据模型的改动不会丢失，可能会重复运行多次$digest循环，而且搞了一个TTL机制，如果$digest循环检查了10次，发现模型还没有稳定下来，直接抛异常。
    - 脏值检测的效率问题：在AngularJS 1.x里面，这个问题没有得到很好的解决。它每次都会生成一份对象副本，当发生修改之后，会把新的对象和原来的对象进行“深比较”。当JS对象的结构非常深的时候，Tree型遍历会消耗大量的CPU时间，因为Tree型遍历一般需要递归，而在JS里面做递归运行效率非常差。同时因为每次都拷贝一个对象副本，内存占用也比较大，这就是你们在网上看到很多吐槽说AngularJS 1.x的数据绑定有效率问题的根本原因。

再来看新版本的Angular里面是如何解决以上2个问题的：

    - 如何才能检测到某个JS对象的值被修改了：新版本的Angular引入了Zones.js，并在此基础上做了自己的封装，Zones会拦截所有可能修改数据模型的操作。实际上在浏览器环境下，只有3个典型的异步回调才可能造成数据模型被修改：事件回调、Ajax回调、定时器回调。所以，Zones.js拦截了所有这些回调函数，自己做了代理。这样，它就可以在数据发生修改的时候精确地给Angular发出通知，因此，在Angular新版本里面，不会再出现更新丢失的问题，你再也不用自己去手动调用该死的$apply()方法了。
    - 脏值检测的效率问题：新版本的Angular引入了RxJS+Immutable的设计，非常有效地避开了这个问题。根据Victor Savkin的说法，这个全新的实现把运行效率从O(N)降低到了O(logN)。所以，在新版本的Angular里面，不再存在所谓的“绑定效率”问题了，有一些无脑黑的文章还在扯这个问题，明显是胡说。

如果你想要更加深入地研究脏值检测的原理，有3篇非常好的文章推荐给你（都是英文版，第一篇有油管上的视频）：

http://blog.thoughtram.io/angular/2016/02/22/angular-2-change-detection-explained.html

http://teropa.info/blog/2015/03/02/change-and-its-detection-in-javascript-frameworks.html

https://vsavkin.com/change-detection-in-angular-2-4f216b855d4c

不管怎么说，AngularJS 1.x总算是第一个把双向数据绑定的思想带到了前端开发领域，虽然实现得没有最新版本的Angular那么优雅。

有了“双向数据绑定”这个神器之后，在某些情况下我们甚至可以一行代码都不用写就可以进行数据绑定操作了，以下是AngularJS 1.x里面的一个典型案例：

<img src="{{ "/assets/img/2017-08-04/binding-3.png" | relative_url }}" alt="data-binding-3" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-04/binding.gif" | relative_url }}" alt="data-binding-4" width="100%" class="img-thumbnail rounded"/>

再来看Vue里面双向数据绑定的写法：

<img src="{{ "/assets/img/2017-08-04/binding-5.png" | relative_url }}" alt="binding-5" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-04/vue-bind-2.gif" | relative_url }}" alt="vue-bind-2" width="100%" class="img-thumbnail rounded"/>

怎么样，是不是长得很像？Angular里面是ng-model，Vue里面用的v-model，把前缀ng-换成了v-，插值都是用的双花括号（Mustache，八字胡）语法。

当然，Vue搞了一套自己的简化版实现，如果你有兴趣请参考这篇文章：

https://gist.github.com/coderek/e656129d9ef48a27124f06a39c957f10

请注意，Vue最近的版本又从React里面抄来了VDom，这部分的机制发生了一些小变化，它会在检测到变化之后驱动VDom去刷新UI。

<img src="{{ "/assets/img/2017-08-04/vdom.png" | relative_url }}" alt="vdom" width="100%" class="img-thumbnail rounded"/>

恭喜Vue，以后VDom就是你的“核心特性”了，可以挺直腰板对外宣称，Vue比React的渲染速度还要快！

也恭喜React社区，你们迎来了史上最强大的抄袭者。它会把最核心的设计思想全部抄走，然后再把自己宣传得比原版还要强大好多倍。

这里要善意地提醒一下Vue，最好去研究一下Facebook的FreeBSD+Patent的开源协议，别把自己抄到大坑里。

### 2.HTML解析器
大家都知道，在AngularJS 1.x里面，指令（组件）的运行的大致流程是这样的：

- 浏览器加载完ng核心库
- ng启动并找到ng-app根指令
- 查找ng-app指令内层所有的自定义指令（组件）
- 调用$compiler动态编译这些指令，做一大堆的预处理工作
- //继续后续动作...

Vue对指令（组件）的处理流程类似。

这里面有一步操作需要“编译”开发者自定义的指令（组件），所以，我们需要一个parser，然后我们非常欣喜地看到Vue-2.0.0-alpha.1里面引入了和AngularJS 1.x同一款开源的htmlparser，请看图：

<img src="{{ "/assets/img/2017-08-04/htmlparser-vue.png" | relative_url }}" alt="htmlparser-vue" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-04/htmlparser-ng.png" | relative_url }}" alt="htmlparser-ng" width="100%" class="img-thumbnail rounded"/>

当然，这个地方有可能是个巧合，毕竟英雄所见略同嘛。对于Vue历次发布的版本，我正在和AngularJS 1.x的各个版本做详细的源代码交叉对比。我会尽量详细阅读每个版本的代码，看看Vue是如何从一个只有2760行代码（VueJS 0.6.0，含注释）的小玩具，一步一步抄到今天这么庞大的，希望源代码里面不要出现让人更加尴尬的情况。

### 3.Filters

在AngularJS 1.x里面，Filter（过滤器）是用来对数据进行格式化的小工具。

AngularJS 1.x里面是这样用的：

<img src="{{ "/assets/img/2017-08-04/ng-filter-1.png" | relative_url }}" alt="ng-filter-1" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-04/ng-filter-2.png" | relative_url }}" alt="ng-filter-2" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-04/ng-filter-3.png" | relative_url }}" alt="ng-filter-3" width="100%" class="img-thumbnail rounded"/>

再来看Vue里面Filter的用法：

<img src="{{ "/assets/img/2017-08-04/vue-filter.png" | relative_url }}" alt="vue-filter" width="100%" class="img-thumbnail rounded"/>

怎么样，很像吧？

但是，问题来了，在Angular的新版本中，Filter这个概念已经被改成了Pipe（管道），功能还是一样的，同样用来对数据做格式化。在新版本的Angular里面，Pipe（管道）是这样用的：

<img src="{{ "/assets/img/2017-08-04/ng-pipe-1.png" | relative_url }}" alt="ng-pipe-1" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-04/ng-pipe-2.png" | relative_url }}" alt="ng-pipe-2" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-04/ng-pipe-3.png" | relative_url }}" alt="ng-pipe-3" width="100%" class="img-thumbnail rounded"/>

Vue会不会在未来的某个版本里面也把Filter改成Pipe呢？

我们拭目以待。

### 3.组件和指令分离
在AngularJS 1.5之前没有突出Component（组件）的概念，当时只有“指令”：

<img src="{{ "/assets/img/2017-08-04/ng-directive-1.png" | relative_url }}" alt="ng-directive-1" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-04/ng-directive-2.png" | relative_url }}" alt="ng-directive-2" width="100%" class="img-thumbnail rounded"/>

在Vue早期的版本里面，抄袭了同样的设计，也是通过directive的方式来定义组件的：

<img src="{{ "/assets/img/2017-08-04/vue-directive.png" | relative_url }}" alt="vue-directive" width="100%" class="img-thumbnail rounded"/>

这种把directive和component混在一起的设计有一个非常大的问题，它导致了很多开发者滥用Directive（指令），出现了到处都是指令的情况。为了彻底解决这个问题，同时也为了突出“组件化”的核心思想，新版本的Angular把“组件”和“指令”这两个概念彻底分离开了。

<img src="{{ "/assets/img/2017-08-04/component-directive-angular2.png" | relative_url }}" alt="component-directive" width="100%" class="img-thumbnail rounded"/>

在新版本的Angular里面，Component（组件）是带有模板的Directive（指令），而Directive（指令）是不能带有模板的，新版本的Angular是这样写组件的：

<img src="{{ "/assets/img/2017-08-04/ng-component.png" | relative_url }}" alt="ng-component" width="100%" class="img-thumbnail rounded"/>

由于Vue跟的不够紧，于是你会发现，当前的版本里面，依然保留了浓重的AngularJS 1.x版本的痕迹：

<img src="{{ "/assets/img/2017-08-04/vue-component.png" | relative_url }}" alt="vue-component" width="100%" class="img-thumbnail rounded"/>

新版本的Angular是这样写Directive（指令）的：

<img src="{{ "/assets/img/2017-08-04/ng-directive-3.png" | relative_url }}" alt="ng-directive-3" width="100%" class="img-thumbnail rounded"/>

而Vue的指令依然很好地保持了AngularJS 1.x时代的风格：

<img src="{{ "/assets/img/2017-08-04/vue-directive-2.png" | relative_url }}" alt="vue-directive-2" width="100%" class="img-thumbnail rounded"/>

### 一直在抄袭，几乎没原创
至于模仿Angular，把项目名称从VueJS改成Vue；模板语法；在文档里面大量利用Angular来对比解释，诸如此类还有非常多的细节，满满都是“高仿”的痕迹，你可以在Vue的文档里面找到更多直观的体验。

很明显，从设计思想、API语法糖，甚至到底层的htmlparser库，Vue都从Angular里面抄了大量的内容，当然最近还从React里面抄来了VDom。

<img src="{{ "/assets/img/2017-08-04/ng-vue.png" | relative_url }}" alt="ng-vue" width="100%" class="img-thumbnail rounded"/>

如果前端框架也算学术研究的话，相似度远远超过50%，请问这还能叫“借鉴”吗？这不是板上钉钉的“抄袭”是什么？Vue里面有多少东西是原创？

所以，把用以下这句话来评价Vue再合适不过了：Vue的优秀之处并非原创，Vue的原创之处并不优秀。

而Vue在它的文档里面宣称的“比Angular和React都简单”，背后的事实是，有好些东西还没有来得及抄，或者抄起来没有那么快。

对于那些刚入前端的兄弟（姐妹？），你说是学一手的设计更好，还是学一个“高仿品”更加面向未来？

当然，就算抄成这样也是没有问题的，毕竟都是开源项目，而且设计思想也没有申请专利。但是，像Vue这样一直跟着抄，抄袭的幅度如此之大，而且抄得心安理得，抄得光荣和骄傲，实在是世所罕见。

恭喜Vue，你为国内抄袭成风的恶劣局面又竖立了一个成功案例。

但是不要忘了，不断跟风抄袭一定会出现一个问题，那就是总有一天会跟不上。

比如，Vue现在的版本感觉就有点抄不动了，因为新版本的Angular用TypeScript进行了彻底的重写。那么，在新版本的Angular里面有哪些全新的概念和设计思想Vue还没有来得及抄袭，或者说暂时不方便抄呢？

请看下一篇技术分析：《新版本的Angular里面还有哪些核心特性目前还没有被Vue抄袭？》

以及稍晚一些的（我需要时间详细读代码）：

《Vue从React里面抄了哪些东西？》
《React里面还有哪些核心特性目前还没有被Vue抄袭？》

### 附：学习资源

想了解一下Angular？请参考官方网站<a href="https://angular.io/" target="_blank">Angular Docs</a>。还有我自己写的一个系列开源项目可以帮助你快速入手：<a href="http://git.oschina.net/mumu-osc/NiceFish" target="_blank">NiceFish</a> 。这个开源项目包含了典型的To C型布局的实例、To B型管理后台系统的示例、移动端Ionic实例、配合Electron打包成桌面端应用的实例。而且我给NiceFish这个项目录制了完整的视频教程，编写了详细的文档。所以，你可以发现，它的star数量已经超过了1.7K，作为一个没啥技术含量的教学项目，还是挺不错的。