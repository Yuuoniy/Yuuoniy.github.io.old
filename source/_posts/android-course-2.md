---
title: android | 手机应用开发基础知识
date: 2018-09-12 16:27:12
tags:
---
关于 android 基本 UI 界面设计的常用空间等内容
<!-- more -->
记得修改 build.gradle 文件
android 应用中的所有用户界面元素都是使用 View 和 ViewGroup 对象构建而成。
打开 res/layout/*.xml 进行布局，首先看到图形显示界面为空白。

修改 res/value/styles.xml [stackoverflow](https://stackoverflow.com/questions/44449275/failed-to-load-appcompat-actionbar-with-unknown-error-in-android-studio)

## UI 设计
### ConstraintLayout
约束布局
形式：app:layout_constraint[组件本身的位置]_to[目标位置]Of="[目标id]"

1：设计和蓝图:选择您想如何在编辑器中查看您的布局。设计视图显示一个颜色预览你的布局,而蓝图视图显示只对每个视图轮廓。或者你可以查看设计+并排蓝图.
提示:您可以按下B就这些视图之间切换。

#### shape
自定义圆角按钮

## 事件处理

### Intent、Bundle 的使用以及 RecyclerView、ListView 的应用
```c++
  Intent mIntent = new Intent(this,SerializableDemo.class);    
        Bundle mBundle = new Bundle();    
        mBundle.putSerializable("data",object);    
        mIntent.putExtras(mBundle);    
          
        startActivity(mIntent);   
```
取值
```c++
Bundle bundle = this.getIntent().getExtras();  
```

### broadcast
静态广播 动态广播
发送广播：将数据装入一个Intent对象，调用Context.sendBroadcast() 方法发出去。
接受广播：所有已经注册的
`BroadcastReceiver` 会检查注册时的 `IntentFilter` 是否与发送的
`Intent` 相匹配，若匹配则就会调用 `BroadcastReceiver` 的 
`void onReceive(Context curContext, Intent broadcastMsg)`

broadcast 例程：
1. 在`AndroidManifest.xml`配置文件中，用`<Receiver>`标签注册一个`BroadcastReceiver`，
还需要有一个字符串作为过滤`filter`，通过`filter`选择接收广播的类。
2. `TestActivity.java`中将`filter`字符串放入`intent`中，再通过广播发出去，等待系统接收。
3. 系统通过`xml`文件，查找到对应的`filter`，映射到对应的`BroadcastReceiver`类。


notification:
1. `Notification.Builder:`用于动态的设置`Notification`的一些属性。
2. `NotificationManager:`负责将`Notification`在状态显示出来和取消。
3. `Notification` 设置`Notification`的相关属性。


eventbus

```c++
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    EventBus.getDefault().register(this);

}
@Override
protected void onDestroy() {
    super.onDestroy();
    EventBus.getDefault().unregister(this);
}
```

