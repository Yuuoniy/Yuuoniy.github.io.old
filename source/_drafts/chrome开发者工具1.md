
#### element 
- html结构面板
- 操作dom样式，结构，时间的显示面板

`event listeners` 观察绑定的事件

`click` 是事件名称

`.div1` 事件是索引名称（也就是通过什么绑定的）

`attachment` 事件来源 

`handler`里面包含事件的毁掉主体内容 ？？

`useCapture`表示该事件是否向上冒泡


`Add attribu`t : 为该元素添加属性
`Edit attribute`：修改该元素的属性
`Force element state`： 为元素激活某种状态（主要用在可以几乎的元素比如a、input、button等）
Edit as HTML：编辑该元素（你可以重写它的整个content）甚至修改它的标签名称
中间简单的掠过.......
Break on：为该元素添加dom操作事件监听。包含三个选项（树结构改变、属性改变、节点移除）。这个选项的作用是帮助我们监控和定位操作元素的代码 


ctrl+F 查找 

#### network 
1.`Network`是一个监控当前网页所有的http请求的面版，它主体部分展示的是每个http请求，每个字段表示着该请求的不同属性和状态   
`Name`：请求文件名称
`Method`：方法（常见的是get post）
`Status`：请求完成的状态
`Type`：请求的类型
`Initiator`：请求源也就是说该链接通过什么发送（常见的是Parser、Script）
`Size`：下载文件或者请求占的资源大小
`Time`：请求或下载的时间
`Timeline`：该链接在发送过程中的时间状态轴（我们可以把鼠标移动到这些红红绿绿的时间轴上，对应的会有它的详细信息：开始下载时间，等待加载时间，自身下载耗时）


单击面板中的任意一条http信息，会在底部弹出一个新的面板，其中记录了该条http请求的详细参数header（表头信息、返回信息、请求基本状态---请参看http1.1协议内容对号入座）、Preview（返回的格式化转移后文本信息）、response（转移之前的原始信息）、Cookies（该请求带的cookies）、Timing（请求时间变化）

3.在主面板的顶部，有一些按钮从左到右它们的功能分别是：是否启用继续http监控（默认高亮选中过）、清空主面板中的http信息、是否启用过滤信息选项（启用后可以对http信息进行筛选）、列出多种属性、只列出name和time属性、preserve log（目前不清楚啥用）、Dishable cahe（禁用缓存，所有的304返回会和fromm cahe都回变成正常的请求忽视cache conctrol 设定）


####  sources部分
展示了本界面所加载的资源列表。还有cookie和local storage 、SESSION 等本地存储信息，在这里，我们可以自由地修改、增加、删除本地存储。
