---
title: vue 组件的生命周期及钩子函数
date: 2019-06-30 09:59:49
tags:
---

vue 组件的生命周期相关
<!-- more -->

关于 vue 组件的生命周期，写简单的项目不需要太多的了解，甚至只需要知道 mounted 这一钩子函数就可以了，但如果项目的逻辑较复杂，就需要对整个组件的生命周期有一些认识
下面就让我们来看看吧。

先看一下官方给出的生命周期图
![](https://cn.vuejs.org/images/lifecycle.png)

图中表明了生命周期的各个阶段和相关的钩子函数。
- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy
- destroyed

可以用以下代码做测试，并在控制台查看输出，对于生命周期有个直观的认识。
```js
 var app = new Vue({
      el: '#app',
      data: {
          message : "xuxiao is boy" 
      },
       beforeCreate: function () {
            console.group('beforeCreate 创建前状态');
            console.log("%c%s", "color:red" , "el     : " + this.$el); //undefined
            console.log("%c%s", "color:red","data   : " + this.$data); //undefined 
            console.log("%c%s", "color:red","message: " + this.message)  
        },
        created: function () {
            console.group('created 创建完毕状态');
            console.log("%c%s", "color:red","el     : " + this.$el); //undefined
            console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化 
            console.log("%c%s", "color:red","message: " + this.message); //已被初始化
        },
        beforeMount: function () {
            console.group('beforeMount 挂载前状态');
            console.log("%c%s", "color:red","el     : " + (this.$el)); //已被初始化
            console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化  
            console.log("%c%s", "color:red","message: " + this.message); //已被初始化  
        },
        mounted: function () {
            console.group('mounted 挂载结束状态');
            console.log("%c%s", "color:red","el     : " + this.$el); //已被初始化
            console.log("%c%s", "color:red","data   : " + this.$data); //已被初始化
            console.log("%c%s", "color:red","message: " + this.message); //已被初始化 
        },
        beforeUpdate: function () {
            console.group('beforeUpdate 更新前状态=');
            console.log("%c%s", "color:red","el     : " + this.$el);
            console.log("%c%s", "color:red","data   : " + this.$data); 
            console.log("%c%s", "color:red","message: " + this.message); 
        },
        updated: function () {
            console.group('updated 更新完成状态');
            console.log("%c%s", "color:red","el     : " + this.$el);
            console.log("%c%s", "color:red","data   : " + this.$data); 
            console.log("%c%s", "color:red","message: " + this.message); 
        },
        beforeDestroy: function () {
            console.group('beforeDestroy 销毁前状态');
            console.log("%c%s", "color:red","el     : " + this.$el);
            console.log("%c%s", "color:red","data   : " + this.$data); 
            console.log("%c%s", "color:red","message: " + this.message); 
        },
        destroyed: function () {
            console.group('destroyed 销毁完成状态');
            console.log("%c%s", "color:red","el     : " + this.$el);
            console.log("%c%s", "color:red","data   : " + this.$data); 
            console.log("%c%s", "color:red","message: " + this.message)
        }
    })
```

下面我介绍一下主要的钩子函数：
- beforeCreate 和 create
  - beforeCreate 
  在 initState 调用之前，而 initState 的作用是初始化 props、data、methods、watch、computed 等属性。因此此时无法获取到 props、data 中定义的值，也不能调用 methods 中定义的函数

  - create 
  initState 调用之后，可以获取到 props、data 中定义的值，也不能调用 methods 中定义的函数
注意 beforeCreate 和 create 执行时还没有渲染 DOM，因此均访问不到 DOM 元素。

- beforeMount 和 mounted
beforeMount 和 mounted 函数执行在 Vue 实例挂载阶段
  - beforeMount
    完成了 el 和 data 初始化 
  - mounted 
    完成挂载

- beforeUpdate 和 updated
beforeUpdate 和 updated 的钩子函数执行时机都是在数据更新的时候

- beforeDestroy 和 destroyed
  beforeDestroy 和 destroyed 钩子函数的执行时机在组件销毁的阶段


以上就是对于各个生命周期的说明了，通过对整个生命周期的了解，我们可以很清晰地知道可以vue 组件在什么阶段做什么事,从而恰当地钩子函数，在恰当的时间做恰当的事情。
