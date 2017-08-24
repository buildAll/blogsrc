---
title: hexo是怎么工作的
tags:
  - hexo
excerpt: '你可能用过hexo(或者jekyll)来搭建自己的博客网站, 可是从hexo init 到 hexo deploy之间发生了什么呢...'
banner:
  url: /images/howhexoworks/hexo_logo.png
  height: 400
categories:
  - HowItWorks
  - hexo
date: 2017-08-20 11:02:28
---
你可能用过hexo(或者jekyll)来搭建自己的博客网站。通常我们在安装、配置完成hexo之后，借助hexo，一般通过以下步骤，就可以完成一篇博客的编写及发布，真是方便极了：

```shell
$ hexo init           // 创建一个新的hexo项目
$ hexo new mynewblog  // 新建一篇标题为mynewblog的文章
$ hexo server         // 为hexo在本地起一个http server, 然后通过浏览器访问博客
$ hexo generate       // 生成将要发布的博客网站包含html在内的静态资源
$ hexo deploy         // 将generate的结果发布到_config.yml中指定的仓库
```
可是，从hexo init到hexo deploy中间发生了什么呢？为了搞清楚这一过程、理解hexo的工作原理，本文将试着回答以下问题:
- 命令行中的hexo是什么
- hexo是怎么将我们写好的markdown转换成html的
- hexo插件是如何工作的
- 本地的hexo项目和git page有什么关系

本文**不是**:
- hexo的安装、使用教程
- git page的使用教程

## 命令行中的hexo是什么?
[hexo](https://github.com/hexojs/hexo)项目在github上已经有超过17k的star了，但是你知道吗，日常我们在命令行"操作"hexo时所输入的**hexo**(例如hexo init)并不是这个17k个star的项目! 是的，我们在命令行中所输入的"hexo"实际是[**hexo-cli**](https://github.com/hexojs/hexo-cli/tree/master/lib)项目，该项目在github上的star还不足50个。

> hexo可以粗略分为三个子项目，分别是:
> - hexo-cli
> - hexo (下文中用hexo core来指代)
> - hexo plugins
>
> 其中hexo plugins不是指某一个单独的项目，而是泛指所有的hexo plugin项目。
>
> 请看下图:
> {% limg howhexoworks/hexo_projects.png  %}
让我们结合这张图来大致看看这三个项目的作用(下面的链接均是指向Github中相关的源码):
- **hexo-cli**: hexo命令行项目，作用是:
  1. 启动hexo命令(进程)，及其参数[解析](https://github.com/hexojs/hexo-cli/blob/5e0969ffa64dec427428a245ab2d65beaf23123b/lib/hexo.js#L49)机制。每次我们输入'hexo xxx'命令后，都会通过node调用hexo-cli中的[entry函数](https://github.com/hexojs/hexo-cli/blob/5e0969ffa64dec427428a245ab2d65beaf23123b/lib/hexo.js#L13)(比如，可以把'hexo init'视为'node hexo-cli/entry.js init')
  2. 实现hexo命令的[三个初始参数(功能)](https://github.com/hexojs/hexo-cli/tree/master/lib/console): init / version / help
  3. [加载](https://github.com/hexojs/hexo-cli/blob/5e0969ffa64dec427428a245ab2d65beaf23123b/lib/hexo.js#L85)hexo核心模块，并[初始化](https://github.com/hexojs/hexo-cli/blob/5e0969ffa64dec427428a245ab2d65beaf23123b/lib/hexo.js#L47)
- **hexo core**: hexo核心，他的主要作用如下:
  1. 实现了hexo功能[扩展对象](https://github.com/hexojs/hexo/blob/master/lib/hexo/index.js#L59)
  2. 实现了hexo核心功能, 如new, publish, generate等（其实是一些hexo插件，下文中会详细分析）
- **hexo plugins**: 指一些能够扩展hexo的插件。插件可以按功能分成两类:
  1. 扩展hexo命令的参数，如[hexo-server](https://github.com/hexojs/hexo-server)(安装这个插件以后才能使用hexo server命令)
  2. 扩展hexo解析文件的"能力"，如增加jade模版解析功能的[hexo-render-jade](https://github.com/hexojs/hexo-renderer-jade)插件

## 从markdown到html的旅程
简单来说，hexo中，从markdown到html的generate过程中做了两件事：
1. 模板渲染
2. 模板渲染

是的，就是这样，就是两次模板渲染。只不过两次渲染的输入、渲染模板的引擎、输出不一样。此处应该有一个表格：
{% limg howhexoworks/render.png  %}
还得有一张图:
{% limg howhexoworks/post.png %}
> 对上面表格和图的说明:
> - hexo core在generate的过程中会产生一个对象，我们在这里把这个对象称为article。第一次渲染的主要目的就是给这个对象添加title,content等属性。其中:
>   1. article.title, article.date, article.tags, article.categories等属性来自yml front的部分
>   2. article.content是markdown文章解析后的html片段
> - hexo项目目录下包含三个子目录，
>   1. source目录，写博客的主要工作目录。这个目录下存放的是我们的markdown文章以及js, images, css
>   2. themes目录，主题目录，定义了即将生成的html的layout, 和html中需要加载的css, js, images
>   3. public目录, hexo generate的最终输出目录。里面包含了整个博客网站的html, css, js, images
> - 第二次渲染，需要引入对应模板文件格式的插件，如.ejs文件就需要使用hexo-render-ejs插件，.jade文件需要使用hexo-render-jade插件，而.sass文件则需要hexo-render-sass插件来转换成css文件。hexo的这一设计有点类似webpack中的loader。

## hexo插件是如何工作的
hexo和webpack还有一点类似的地方就是插件驱动理念。即hexo(和webpack)是先实现一套(插件)扩展系统，然后再往扩展系统中添加插件来实现自身的功能。即我们日常使用的hexo init, hexo new，hexo generate等等功能都是通过一个个插件(其实就是一个个function)实现的。

> 具体来讲就是:
> 1. [hexo.extend](https://github.com/hexojs/hexo/blob/master/lib/hexo/index.js#L59)这个对象的每个属性都是一个用来绑定(特定)插件的对象。（所谓"绑定"，其实就是对象的register方法）
> 2. hexo[初始化](https://github.com/hexojs/hexo/blob/master/lib/hexo/index.js#L153)过程中先加载内部插件，再加载外部插件
>
> 而这些插件的功能分为两大类: 命令行插件和generate过程相关功能，例如：
> - 命令行插件, hexo [new](https://github.com/hexojs/hexo/blob/master/lib/plugins/console/index.js#L47), 是在hexo.extend.console对象上绑定的一个[插件](https://github.com/hexojs/hexo/blob/master/lib/plugins/console/new.js)
> - generate过程相关的插件，如上文提到的往article对象添加title,content等属性的功能，是通过往hexo.extend.processer对象上[绑定](https://github.com/hexojs/hexo/blob/master/lib/plugins/processor/index.js#L13)post插件来[实现](https://github.com/hexojs/hexo/blob/master/lib/plugins/processor/post.js#L52)的
>
> 所以，当我们想自己动手写插件时，就是像hexo官网给出的[这样](https://hexo.io/api/console.html),调用某个对象的register方法，如hexo.extend.console.register。

## hexo和git page
{% limg howhexoworks/deploy.png %}
如上图，(用户通过浏览器访问到的)git page上的博客网站其实是hexo generate之后生成的public目录下的内容。
> 所以，一个hexo博客项目应该有两个仓库:
> 1. (基于hexo init结果的)[博客编写仓库](https://github.com/buildAll/blogsrc)。可以把这个项目看成一个代码库，用来"开发"博客网站(包含写博客，生成博客等任务)
> 2. 存放(hexo generate结果的)[public目录仓库](https://github.com/buildAll/buildall.github.io)。这个项目是"只读"的，我们不会直接修改这个仓库的内容，我们也不会对这个仓库直接进行git pull、git commit、git push等常规操作。这个仓库的内容就是public目下的内容，即是通过hexo generate产生、hexo deploy提交的。

## 总结
hexo简洁、强大的功能来自于自身优雅的系统设计:
1. hexo进程启动、hexo核心对象封装、插件系统分别独立
2. 自身采用插件驱动，生来就具备高可扩展性

希望读完这篇文章你能对hexo本身有更深入的理解，也能通过hexo的代码设计，对自己以后写出更优雅的代码有所启发。

_赵彪原创，请随意转载，但务必保留作者署名和原文链接_

{% limg support.png %}

<script>
console.log('hello, how are you?')
</script>
