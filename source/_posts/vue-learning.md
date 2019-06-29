---
title: 如何使用vue 及相关技术栈快速构建前端
date: 2019-06-29 21:41:10
tags:
---
在本文中，我将给大家介绍一些 vue 的基础知识、相关工具、生态等内容。相信通过该文章你会对 vue 有大概的认识，并且可以快速地找到入门的方向。
一起来学习吧！

<!-- more -->

首先介绍一下本次系分项目前端使用的技术栈：
- vue-cli
- vuex
- element-ui
- vue-router
- axios
- webpack
- yarn

我将一一简单介绍:

## vue
一开始学 vue, 主要熟悉一些组件的概念，数据，生命周期。
写的时候主要也就是 data, mounted, methods 这几项。
布局方面使用 v-for/v-if/v-show 。
还有学一些路由之间的跳转，传递参数这些内容。
基本够用了... 
还有可以学一下怎么父子组件的相关概念。
其他的还有computed/watch 等，但是简单的项目也可以不用。

## vuex
用来做状态管理，简单来说，可以维护一些全局变量。

## vue-cli 
使用  vue-cli  你可以快速构建一个 vue 项目。目前的版本是 3.0。不同先前版本，构建完项目后需要自己创建配置文件，文件名为 `vue.config.js`
创建： `vue create project-name`

## element-ui
UI 框架。算是 vue 生态中最常用的UI框架了。其他还有Muse-UI 等等。
我用 element-ui 比较多，就介绍一下我常用的组件：
布局方面，使用 `<el-container>` ` <el-header>`   `<el-main>` 将页面分成几大部分。然后使用 `el-row` `el-col` 进行布局，设置好 `offset` 和 `span` 属性就行
使用 `message` 来弹出消息，主要有 `success/error/warning` 几类。
使用 `el-menu` 设置导航
此外，我喜欢用 `el-card` 将页面卡片化，布局也很方便。
我常用的还有 form/button/icon/divider 
在系分项目中我使用 `Upload` 来实现头像上传。
新版的 element-ui 也引入了一些非常方便的组件，包括：
`avatar`,`loading`。我在系分项目中也用到了。


## vue-router
这个是 vue 官方的路由管理器。(不是前后端路由的那个路由)
基本看一下示例就会写了，主要就是写下 路径/你想渲染的组件。复杂一点的，可以学一下嵌套路由。也很简单
在布局的时候使用 `router-view` 进行占位，

## axios
这是用来处理网络请求的。 vue-resource 也是，但已经不被官方推荐了。使用  axios 好一些。
网上教程很多，我就不说了。

## webpack
可以在 vue.config.js 中编写 webpack 配置文件。如
```json
 configureWebpack: {
    devServer: {
      proxy: {
        '/api': {
          target: 'sever address',
          pathRewrite: {"^/api" : ""}
        }
      }
    }
  },
```
用来设置请求代理，将请求发给服务端。

## yarn
依赖管理工具，替代 npm，速度方面有些优势。
`yarn run server`  运行项目
`yarn add xxx` 安装依赖

## 其他组件

vue 有很多第三方组件，可以用来实现一些小功能，如表单验证/分页/富文本编辑器等等，可以自己探索。


## 工具相关
再介绍一下工具！可以大大提高生产力。
### Vue.js devtools 
chrome 的拓展，vue 调试神器，真的很好用。可以用来查看运行时 vue 各个组件的布局以及数据，真的很好用。

### 其他插件
写代码怎么能少了高亮和自动补全，格式化插件呢。
我使用的编辑器 `vscode`, 用的相关插件有：`Vetur/Vue 2 Snippets/vue-format`。推荐


## 总结
以上就是使用 vue 构建前端常用的技术栈了，有了宏观的认识后，入门真的很容易，但是我上面所说的内容都是比较简单的，还是有很多高级的内容需要你自己去探索啦。可以找个基础的项目看下，
都是大同小异（写多了感觉真的纯搬砖...)
