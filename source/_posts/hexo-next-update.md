---
title: hexo | NEXT 主题更新,从 5.1 到 6.0
date: 2018-09-11 16:53:30
tags:
---
我说...我的标题怎么没有居中呢,我的代码高亮怎么这么丑呢..我的主题看起来怎么和别人的不一样呢,突然发现自己的主题没有更新下
<!-- more -->

[官方更新说明](https://github.com/theme-next/hexo-theme-next/blob/master/docs/UPDATE-FROM-5.1.X.md)

主要是更新时保留原来的配置

首先:
下载新版本 next:
  `https://github.com/theme-next/hexo-theme-next themes/next-reloaded `

复制原版本next的`config.yml`文件到 `next-reloaded` 中
报错:
`Unhandled rejection TypeError: path.substring is not a function`
哎呀..还是重新配置一遍好了哈哈哈哈哈,也没花多大功夫
设置 `auto_excerpt` 为`true`,避免直接在主页显示全文
### 设置评论:
disqus:true

### 百度统计
注册账号获取id,修改配置文件即可
### 阅读量统计

参考[为NexT主题添加文章阅读量统计功能](https://notes.doublemine.me/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)

[添加文章阅读量](http://www.jeyzhang.com/hexo-next-add-post-views.html)

最新的next主题已经配置好`leancloud`的，只需要在配置文件中设置为true,并且配置 `appid appkey`就好了，我乱七八糟搞了一堆还弄错了orz
[文章字数统计](https://github.com/theme-next/hexo-symbols-count-time)
`npm install hexo-symbols-count-time --save`
在配置文件中设置`symbols_count_time`相关属性

不知道为什么突然出现了 `Page build failed`. 错误  ,原来是github的问题..

想着顺便更新一下hexo，生成的时候就崩了，有点难受... 尝试了没有解决
提示：
`Error: Cannot find module 'highlight.js/lib/languages/jboss-cli`

所以我...又回退了 npm uninstall hexo 后，重新下载 解决问题...
