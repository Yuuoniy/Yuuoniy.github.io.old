---
title: 博客配置及hexo学习 　
date: 2017-04-09 17:27:09  　　
tags: hexo  　
categories: 学习


---
记录搭建博客的注意事项，hexo基本操作插件等内容
<!--more-->


首先说说搭建博客过程中学习到的几点吧， 希望能帮助到别人

- 要理解一下hexo的各个命令 不要只是根据教程输入 不然后期会出现因为漏了某些步骤而出现的问题  
- 注意yml语法 尤其是: 后面应该有一个空格，很多报错信息都是因为这个
- 配置信息中不要出现中文(也可能是我语言设置的问题），我加了中文就导致了一连串的错误
- 记得deploy 而不是push到github github pages 就会自动更新了
影响github pages 的是你有没有deploy （即使有些commit还没有push 到github 上
配置过程中发现自己对git还不是很熟
- pull 的时候可以加上 --allow-unrelated-histories
- 报错信息和实际错误可能是没有关系的，不要钻进去


#### 基本操作 ####

  这部分只记录一些我觉得会用到但是还不太熟悉的操作

##### 删除 #####
    　　先删除本地文件，然后通过生成和部署命令进而将远程仓库中的文件也一并删除。具体来说，  
    以最开始默认形成的helloworld.md这篇文章为例。  
    首先进入到source / _post 文件夹中，找到helloworld.md文件，在本地直接执行删除。然后依次执行hexo g，  
    hexo d，就成功删除文章了。

##### 草稿 #####
```
hexo new draft "new draft"
```
　　草稿可以当作私密文章，一般不在页面中显示，如果想把某一篇文章移除显示，又不舍得删除，可以把它移动到_drafts目录之中。 
如果想要预览草稿，有两种方法：   
1.在配置文件中设置render_drafts: true 2.hexo server --drafts  

通过以下命令将草稿变成正式文章

```
hexo publish [layout] <filename>
```

另外，对于一些文章的修改只需要刷新一下就可以显示了。

##### 文件 ####
**node_modules**： 包含整个hexo的依赖包  
**public**：执行hexo g后hexo根据我们的主题以及文章内容生成的一个文件。可以看到对应的时间及博客内容  
**scaffolds**：里面是模板，默认有draft,page,post，我们生成文章时可以指定采用哪个模板。
**db.json**： 待补充
**package.json**： 待补充

##### 插件 ####
 若使用Algolia的搜索服务, 增加新的文章后需要执行` hexo algolia` 来更新

##### 编辑 ####
- 正文中可以使用<!--more-->设置文章摘要
（要注意使用标题时#后不加空格无法渲染）
