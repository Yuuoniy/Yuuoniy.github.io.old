---
title: 如何在hexo博客中插入图片
date: 2017-06-06 16:56:22
tags: hexo
---
是很简单的东西啦
<!-- more --> 

首先介绍两种插入图片的语法：  
###### markdown
`![图片说明](图片路径)`

######  HTML 
`<img src="图片路径" alt =""width="" height="" align="" >`
标签的属性可以根据自己的需要来设置
使用`HTML`的`<img>` 标签可以更好地控制图片的显示方式

说完语法，我们来说说图片的存放位置吧  

在线图片直接复制图片地址就好了。  

主要说说本地存放的。  
首先，打开hexo中的 `_config.ym`,将`post_asset_folder` 设为true  
这样在创建新文章后，hexo同时也会创建一个与文章同名并且同层次的文件夹，可以把要插入的图片都放到文件夹中。(其实也可以手动创建与_post同层次的文件夹，然后把所有文章要插入的图片都丢到里面_)

执行命令安装插件：  
`npm install https://github.com/CodeFalling/hexo-asset-image --save`  

接下来使用  
`![图片说明](目录名称/图片名称)` 
HTML 同理，根据上面提供的语法修改就可以了  
真的好简单对不对..(这个用教吗)(不管 我就是写了)  
下面就插入一张图片来结束文章吧  

<img src="pictures/end.jpg" align="center">

7.28补充： 用七牛云作为图床很棒！
还可以用七牛的 imageView接口返回特定尺寸的图片  
配合chrome的插件 qiniu upload files 超级方便(大概是我没有用过什么好东西)
