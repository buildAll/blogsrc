---
title: hexo是怎么工作的
excerpt: '你可能用过hexo(或者jekyll)来搭建自己的博客网站, 可是从hexo init 到 hexo deploy之间发生了什么呢...'
date: 2017-08-20 11:02:28
banner:
  url: /images/howhexoworks/hexo_logo.png
  height: 400
categories:
 - HowItWorks
 - hexo
tags:
 - hexo
---
你可能用过hexo(或者jekyll)来搭建自己的博客网站。通常我们在安装、配置完成hexo之后，借助hexo，一般通过以下步骤，就可以完成一篇博客的编写及发布，真是方便极了：

```shell
$ hexo init           // 创建一个新的hexo项目
$ hexo new mynewblog  // 新建一篇标题为mynewblog的文章
$ hexo server         // 为hexo在本地起一个http server, 然后通过浏览器访问博客
$ hexo generate       // 生成将要发布的博客网站包含html在内的静态资源
$ hexo deploy         // 将generate的结果发布到_config.yml中指定的仓库
```
可是，从hexo init到hexo deploy中间发生了什么呢？为了搞清楚这一过程,本文将试着回答以下问题:
- 命令行中的hexo到底是什么
- hexo是怎么将我们写好的markdown转换成html的
- hexo插件是如何工作的
- 你本地的hexo项目和git page有什么关系

本文**不是**:
- hexo的安装、使用教程
- hexo的源码分析
- git page的使用教程

### 命令行中的hexo到底是什么?
[hexo](https://github.com/hexojs/hexo)项目在github上已经有超过17k的star了，但是你知道吗，日常我们在命令行"操作"hexo时所输入的**hexo**(例如hexo init)并不是这个17k个star的项目! 是的，我们在命令行中所输入的"hexo"实际是[**hexo-cli**](https://github.com/hexojs/hexo-cli/tree/master/lib)项目，该项目在github上的star还不足50个。

hexo可以粗略分为三个子项目，分别是:
- hexo-cli
- hexo (下文中用hexo core来指代)
- hexo plugins

其中hexo plugins不是指某一个单独的项目，而是泛指所有的hexo plugin项目。

请看下图:
{% limg howhexoworks/hexo_projects.png  %}
让我们结合这张图来大致看看这三个项目的作用(下面的链接均是指向Github中相关的源码):
- **hexo-cli**: hexo命令行项目，作用是:
  1. 实现hexo命令(进程)，及其参数[解析](https://github.com/hexojs/hexo-cli/blob/5e0969ffa64dec427428a245ab2d65beaf23123b/lib/hexo.js#L49)机制
  2. hexo命令的[三个初始参数(功能)](https://github.com/hexojs/hexo-cli/tree/master/lib/console): init / version / help
  3. 实现了[hexo核心模块加载](https://github.com/hexojs/hexo-cli/blob/5e0969ffa64dec427428a245ab2d65beaf23123b/lib/hexo.js#L85)
- **hexo core**: hexo核心，他的作用如下:
  1. 实现了几个hexo常用的命令[参数(功能)](https://github.com/hexojs/hexo/tree/master/lib/plugins/console)，如: new / generate / publish等
  2. 实现了hexo api
  3. 实现了hexo [plugin加载](https://github.com/hexojs/hexo/blob/master/lib/hexo/load_plugins.js)机制
- **hexo plugins**: 指一些能够扩展hexo的插件。插件大致分两类:
  1. 扩展hexo命令的参数，如[hexo-server](https://github.com/hexojs/hexo-server)(安装这个插件以后才能使用hexo server命令)
  2. 扩展hexo生成静态资源的"能力"，如增加jade模版解析的[hexo-render-jade](https://github.com/hexojs/hexo-renderer-jade)插件

### 从markdown到html的旅程
简单来说，hexo中，从markdown到html的generate过程中做了两件事：
1. 模板渲染
2. 模板渲染

是的，就是这样，就是两次模板渲染。只不过两次渲染的输入、渲染模板的引擎、输出不一样。此处应该有一个表格：
{% limg howhexoworks/render.png  %}
还得有一张图:
{% limg howhexoworks/post.png %}
对表格和图的说明:
- hexo core在generate的过程中会产生一个对象，我们在这里把这个对象称为article。第一次渲染的主要目的就是给这个对象添加title,content等属性。其中:
  1. article.title, article.date, article.tags, article.categories等属性来自yml front的部分
  2. article.content是markdown文章解析后的html片段
- hexo项目目录下包含三个子目录，
  1. source目录，写博客的主要工作目录。这个目录下存放的是我们的markdown文章以及js, images, css
  2. themes目录，主题目录，定义了即将生成的html的layout, 和html中需要加载的css, js, images
  3. public目录, hexo generate的最终输出目录。里面包含了整个博客网站的html, css, js, images
- 第二次渲染的模板引擎

#### post
hexo core在generate的过程中会去处理一个对象，我们在这里把这个对象称为data。我们平时在编辑器里用markdown写的文章，在generate过程中会被解析并绑定到data.content对象上。
#### theme


_赵彪原创，请随意转载，但务必保留作者署名和原文链接_

<script>
  console.log('hihi')
</script>
