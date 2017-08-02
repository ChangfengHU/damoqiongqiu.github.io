---
layout: post
title:  "Angular有哪些地方比Vue更优秀？"
author: "大漠穷秋"
comments: true
date:   2017-08-02 09:26:13 +0800
category: "web前端"
published: true
excerpt: Angular有哪些地方比Vue更优秀？
---
大家都知道，Vue一直紧跟Angular（含AngularJS 1.x）的脚步，借鉴了大量的特性。但是，借鉴毕竟是借鉴，复制品不太可能比正品行货做得更精致。最近一个月仔细研究了Vue的所有文档和常见的Demo项目，本文将会进行非常细致全面的对比。

### Round 1：打包工具
大家都知道，Angular有自己的打包工具@angular/cli，Vue也有自己的打包工具vue-cli。看起来挺像的，但是你仔细使用一下就会发现，这两个东西完全就不是一个级别。为了给出一个直观的对比，我们简化一下，就看看这两款工具分别实现了多少功能吧。

以下是vue-cli能支持的功能：

<img src="{{ "/assets/img/2017-08-02/vue-cli.jpg" | relative_url }}" alt="vue-cli" width="100%" class="img-thumbnail rounded"/>

包括help在内，一共只有可怜巴巴的4个命令。

我们再来看@angular/cli能做什么：

<img src="{{ "/assets/img/2017-08-02/ng-cli-1.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-2.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-3.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-4.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-5.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-6.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-7.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-8.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-9.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-10.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-11.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-12.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>
<img src="{{ "/assets/img/2017-08-02/ng-cli-13.jpg" | relative_url }}" alt="ng-cli" width="100%" class="img-thumbnail rounded"/>

怎么样，有直观感受了吧？简而言之，如果说@angular/cli是一架坦克的话，vue-cli就是一架玩具坦克：

<img src="{{ "/assets/img/2017-08-02/ng-cli-tank.png" | relative_url }}" alt="ng-cli-tank" width="100%" class="img-thumbnail rounded"/>

<img src="{{ "/assets/img/2017-08-02/vue-cli-tank.png" | relative_url }}" alt="vue-cli-tank" width="100%" class="img-thumbnail rounded"/>

vue-cli要做到@angular/cli这么完善基本上没有可能，因为人力和资金都够不上。也正是因为vue-cli是如此简陋，导致很多日常开发必备的功能都需要开发者自己去下载配置第三方的Node模块。

噢，这里要特别提一下，@angular/cli的底层集成了webpack，Angular项目组在此基础上做了自己的封装，如果你有兴趣去看<a href="https://github.com/angular/angular-cli" target="_blank">@angular/cli</a>的源代码，应该已经发现了这一点。

### Round 2：异步加载模块
在现在WEB开发中，前端的代码量已经变得非常庞大，不可能再像以前那样，把所有JS全部压缩成一个文件，然后一次性全部加载。所以，无论你用什么样的前端框架，异步加载模块都是一个必备的功能。为了帮助初学者理解最基本的思路，画图示例如下：

<img src="{{ "/assets/img/2017-08-02/async-module.jpg" | relative_url }}" alt="async-module" width="100%" class="img-thumbnail rounded"/>

以下是代码块和路由之间的对应关系（虚构的）：

- http://localhost:4200/home          chunk-0.js(10k)
- http://localhost:4200/user          chunk-1.js(10k)
- http://localhost:4200/role          chunk-2.js(50k)
- http://localhost:4200/permission    chunk-3.js(80k)

就算你不是前端开发者，你也能看出来，这样好多了对不对？你不需要一上来就把所有js文件（一共150k）全部加载进来，只有当用户开始使用某个菜单的时候，才去异步加载。很明显，这是一个必备的特性，我们来看在Vue和Angular里面分别是怎么实现异步模块的。

我们先来看Vue里面繁琐的做法：

<img src="{{ "/assets/img/2017-08-02/async-vue-1.jpg" | relative_url }}" alt="async-vue-1" width="100%" class="img-thumbnail rounded"/>

根据以上文档，你首先要写一下你已经写好的Vue组件，然后自己再去利用webpack的code splitting特性自己写配置文件。很明显，当你的项目规模已经做得很大的时候，比如你已经写了500个Vue组件，这时候你想切一下异步模块，然后你就开始恶心了。首先你要把这500个组件都改一下，虽然每个组件改动的幅度不大，但是要改的数量还是挺多的！或者，当你发现某一些组件本来被打包在chunk-0.js里面的，但是由于业务上的修改，你现在必须把它们移动到chunk-1.js里面去，那对不起，请你自己到配置文件里面去慢慢改吧！

然后当你点开以上文档里面的链接，你会看到以下翻译了半拉子的中英混杂文档：

<img src="{{ "/assets/img/2017-08-02/async-vue-2.jpg" | relative_url }}" alt="async-vue-2" width="100%" class="img-thumbnail rounded"/>

这时候你会突然意识到，在Vue表面简单优雅的文档背后，隐藏着填不满的天坑，用一幅你们都喜欢的图来表达这种心情：

<img src="{{ "/assets/img/2017-08-02/front-end-and-back-end-2.jpg" | relative_url }}" alt="front-end-and-back-end" width="100%" class="img-thumbnail rounded"/>

我们再来看Angular里面是怎么做异步模块加载的，非常简单，你不需要修改任何组件，你也不需要自己去搭建webpack然后编写繁琐的配置文件，你只要在声明路由的时候，用loadChildren配置项告诉Angular这是一个异步模块，done!

<img src="{{ "/assets/img/2017-08-02/angular-router-2.png" | relative_url }}" alt="angular-router-2" width="100%" class="img-thumbnail rounded"/>

@angular/cli在编译的时候会自动检查所有路由配置，然后根据这些配置自动帮你切分成多个chunk。注意：所有这些动作都是自动完成的，不需要你再去单独写什么配置项！很明显，当项目规模变得很大的时候，Angular这种全自动实现的优势就开始凸显了。你到底是更相信人肉的做法？还是更相信机器的精确性？

### Round 3：单元测试和集成测试
2009年，NodeJS的出现，标志着前端开发进入了工业化时代。因为，在没有NodeJS的那段黑暗岁月里面，前端开发里面有很多事情都是没法做的，比如：JS代码的合并和和压缩、CSS的预处理、图片路径的自动处理、自动为每个资源块生成Hash码避免浏览器缓存，等等。

那时候，有一个巨大的痛点，基本上没人能解决，那就是前端的自动化测试，包括单元测试和集成测试。大家都知道，前端代码有一个很大的特点，它是依赖于浏览器环境的，所以在NodeJS出现之前，大部分企业都只能采用人肉测试的方式。当然，现在还有很多企业在人肉测试，这些企业要注意，你们距离现代前端开发有点远了！

目前，市面上有很多Node模块可以用来做单元测试，Karma+Jasmine的方案基本上是事实标准。

所以，Vue也是用的Karma+Jasmine的方案，以下是Vue官方文档里面的描述：

<img src="{{ "/assets/img/2017-08-02/vue-ut-1.jpg" | relative_url }}" alt="vue-unit-test" width="100%" class="img-thumbnail rounded"/>

这里面有两点要注意：

- 第一点：Vue的单元测试需要你自己去安装配置Karma+Jasmine，以上图片红框中的内容表明，Vue本身对单元测试的支持还在“计划”中。
- 第二点：Vue的单元测试对你编写的组件是有要求的，为了能编写单元测试用例，你必须尽量用props属性来表达组件的数据。

我们再来看Angular里面是如何做单元测试的：

由于我们有强大的@angular/cli，它在生成每个组件的时候会自动生成对应的单元测试用例文件，同时，它还会把一些模板性质的代码都自动生成好：

<img src="{{ "/assets/img/2017-08-02/ng-ut-1.jpg" | relative_url }}" alt="angular-unit-test" width="100%" class="img-thumbnail rounded"/>

请注意：

- @angular/cli里面内置了Karmar+Jasmine，所以你不需要自己去安装配置，你也不需要为了版本问题焦头烂额，你只要编写测试用例就好了。
- 每个组件对应的测试用例模板都是自动生成的，你不需要手动去拷贝文件然后进行修改。
- 写完测试用例之后，你只要运行ng test，@angular/cli会自动帮你去跑测试用例，你也没有必要自己去写启动脚本了。

<img src="{{ "/assets/img/2017-08-02/ng-ut-2.jpg" | relative_url }}" alt="angular-unit-test" width="100%" class="img-thumbnail rounded"/>

如果说Angular在单元测试方面的得分是80分，那么Vue最多就20分，因为它把一堆麻烦事都丢给开发者，自己什么也没有做。而在集成测试方面，Vue就只能是0分了，因为压根什么都没做。

单元测试只是验证每一个组件自身是不是有问题，但是即使每一个组件本身都没有问题，把这些组件都整合起来形成完成的系统之后还是会存在各种各样的问题。所以，在整个系统都开发完成之后，我们还需要测试一下整体的功能有没有问题，这叫“集成测试”、“场景测试”，或者叫“端到端测试”。

Angular团队开发了一款集成测试工具，叫做Protractor（量角器）：

<img src="{{ "/assets/img/2017-08-02/protractor.png" | relative_url }}" alt="angular-protractor" width="100%" class="img-thumbnail rounded"/>

这是一款专门针对Angular的集成测试工具，它可以自动把你本地的浏览器启动起来，模拟用户输入和鼠标点击等等功能，有了Protractor之后，你不再需要人肉去点页面了！

<img src="{{ "/assets/img/2017-08-02/protractor.gif" | relative_url }}" alt="angular-protractor-gif" width="100%" class="img-thumbnail rounded"/>

请注意，以上动图里面的输入和页面滑动都是protractor自动完成的，当然，是根据你编写的集成测试用例运行的。@angular/cli在生成新项目的时候，会自动生成一个e2e目录，还有一些模板文件，你只要在里面编写用例就好了：

<img src="{{ "/assets/img/2017-08-02/angular-e2e-1.jpg" | relative_url }}" alt="angular-protractor-gif" width="100%" class="img-thumbnail rounded"/>

编写集成测试用例的时候，同样使用的Jasmine语法，你不需要学习任何新东西。

从对集成测试的支持大家应该可以看出来，在那些技术含量不太高的地方，Vue确实可以抄袭得有模有样，但是一旦技术门槛提高，Vue就没法抄了，或者说抄得没那么快了。

想要了解Protractor更多的细节，请访问官方网站：<a href="https://github.com/angular/protractor" target="_blank">Protractor</a>

### Round 4：TypeScript
TypeScript（简称TS）是微软开发的一门脚本语言，它是JavaScript的超集。
<img src="{{ "/assets/img/2017-08-02/ts-1.png" | relative_url }}" alt="TypeScript-1" width="100%" class="img-thumbnail rounded"/>

TS的编译过程如下：
<img src="{{ "/assets/img/2017-08-02/ts-2.png" | relative_url }}" alt="TypeScript-2" width="100%" class="img-thumbnail rounded"/>

Angular初学者有一个常见的问题，那就是在学习Angular之前是否要先学一学TypeScript。

我的看法是不需要！

因为TypeScript的语法和ES6的语法相似度非常高，如果你已经熟悉了ES6，可以直接上手写TypeScript，一边玩一边学就好了。对于已经掌握了一门后端编程语言的开发者，例如Java和C#，那就更加没有难度了，除了一些小小的语法习惯要适应一下之外，几乎没有任何难度。

关于TypeScript，比较有意思的一点是，这门语言本身也是一个开源项目，它在github上的链接：https://github.com/Microsoft/TypeScript 。有一件事特别值得一提，C#之父Anders Hejlsberg是TypeScript项目组的核心代码提交者。

对于代码规模很大的前端项目来说，有两件事情非常重要：

- 如何合理地切分模块，从而达到分而治之的目标。
- 如何管理或者说约束开发人员写出健壮的代码。
- 项目上线之后，如何保证可维护性。

TypeScript带来的强类型约束刚好可以在这两个方面带来巨大的帮助，引用你们都喜欢的一张图：
<img src="{{ "/assets/img/2017-08-02/ts-3.jpg" | relative_url }}" alt="TypeScript-3" width="100%" class="img-thumbnail rounded"/>


### Round 5：AOT and TreeShaking
由于Angular应用是基于TypeScript编写的，而TypeScript是强类型的，这就带来了另一个好处：TS编译器可以做大量的静态检查和编译优化，比如AOT和Tree Shaking，把你用不到的那些模块和代码都自动消除掉，从而可以保证你的应用足够小巧和高效。

随着你的项目规模越来越大，你会越来越体会到有AOT是多么的重要，而Vue在这一点上根本无能为力，开发者只能自求多福了。

<img src="{{ "/assets/img/2017-08-02/aot.jpg" | relative_url }}" alt="angular-aot" width="100%" class="img-thumbnail rounded"/>

我看到还有一些人在误导开发者，说Angular体积大，这基本上就是闭着眼睛胡说了，看上图。

### Round 6：团队
来看两张图，重新定义一下什么叫“团队”。

以下是angular项目组的提交记录图：

<img src="{{ "/assets/img/2017-08-02/ng-github-1.jpg" | relative_url }}" alt="angular-github" width="100%" class="img-thumbnail rounded"/>

以下是vue的提交记录图，你会发现基本上都是一个人在提交，其大部分都是打酱油的：

<img src="{{ "/assets/img/2017-08-02/vue-github.jpg" | relative_url }}" alt="vue-github" width="100%" class="img-thumbnail rounded"/>

如果你知道“独行快、众行远”的话，应该知道我在说什么。

### 更多待总结的内容

还有很多非常细节的内容，我会继续研究，然后详细扩充进来，比如：

- 由于Vue没有依赖注入机制，你无法mock后台服务，所以你没法优雅地解决单元测试的mock问题。
- Vue没有内置对Form表单校验的支持，你必须自己去把第三方的表单校验工具集成到项目里面。
- Vue没有对i18n做内置的支持，如果你的项目要做国际化，很抱歉，还是你自己去解决。

### 结语
Angular从入手到编码、测试，全部一条龙服务，就像你们说的，它是一个“全家桶”式的设计，你只要安心编写你的应用就好了。

而Vue貌似简单文档本后，隐藏这一个接一个的大坑，当你真正深入使用之后，你就会猛然发现其总体学习成本比Angular高出非常多！

Vue只是一个新手玩具，而Angular才是真正的战斗坦克。

所以，拜托不要再拿Vue来和Angular比了，完全就不是一个量级的东西好不好啊？

### 附：学习资源

想了解一下Angular？请参考官方网站<a href="https://angular.io/" target="_blank">Angular Docs</a>。还有我自己写的一个系列开源项目可以帮助你快速入手：<a href="http://git.oschina.net/mumu-osc/NiceFish" target="_blank">NiceFish</a> 。这个开源项目包含了典型的To C型布局的实例、To B型管理后台系统的示例、移动端Ionic实例、配合Electron打包成桌面端应用的实例。而且我给NiceFish这个项目录制了完整的视频教程，编写了详细的文档。所以，你可以发现，它的star数量已经超过了1.7K，作为一个没啥技术含量的教学项目，还是挺不错的。