---
date:
status: public
title: Activity生命周期
---

###典型情况生命周期分析
+ 一个特定Activity第一次启动，回调为：onCreate -> onStart -> onResume.
+ 打开新的Activity或切换到桌面时，回调为：onPause -> onStop，这里有个特例，加入打来新的Activity采用了透明主题，那么当前Activity不会回调onStop.
+ 新用户再次回到原Activity时，回调为：onRestart -> onStart -> onResume.
+ 用户按back键回退时，回调为：onPause -> onStop -> onDestroy.
+ 从整个生命周期来说，onCreate和onDestroy是配对的，分别标识着Activity的创建和销毁，而且**只可能调用一次**，从Activity **是否可见**来说，onStart和onStop是配对的，随着用户操作或者屏幕点亮和熄灭，这两个方法被多次调用，从Activity **是否在前台**来说，onResume和onPause是配对的，随着用户操作或者设备屏幕的点亮和熄灭，这两个方法被多次调用。

###非典型生命周期分析
+ 系统配置改变导致Activity经历完整生命周期
 + 系统配置改变时，Activity被销毁，其onPause、onStop、onDestroy均会被调用，同时由于Activity是在异常情况下终止的，系统会调用onSaveInstanceState来保存当前Activity的状态。这个方法**调用时机在onStop之前**。当Activity被重新创建后，系统会调用onRestoreInstanceState，并把Activity销毁时onSaveInstanceState所保存的Bundle对象作为参数同时传递给onRestoreInstanceState **和onCreate**方法。onRestoreInstanceState **调用时机在onStart之后**。
 + onRestoreInstanceState和onCreate恢复数据的区别：onRestoreInstanceState一旦被调用，其参数Bundle savedInstanceState一定是有值的，不要额外判断是否为空，而如果是正常启动的话，onCreate的Bundle为null。官方建议使用onRestoreInstanceState来恢复数据。
+ 资源内存不足导致优先级低的Activity被杀死，优先级由高到低，分如下三种：
 + 前台Activity，正在交互的Activity优先级最高
 + 可见但非前台，比如被Dialog覆盖的Activity，导致Activity可见但位于后台无法和用户直接交互。
 + 后台Activity，已经被暂停的Activity，比如执行了onStop，优先级最低。
 
 系统内存不足时，就会按照上述优先级kill目标Activity所在的**进程(process)**，并在后续onSaveInstanceState和onRestoreInstanceState来存储和恢复数据。如果一个进程没有四大组件在执行，那么这个进程将很快被系统kill。比较好的做法是将后台工作放到Service中从而保证有一定优先级，这样不容易被杀死。

####系统配置变化，如何不重建Activity
 + 可以给Activity指定configChanges属性。比如不让Activity在屏幕旋转时重建，就可用`android:configChanges="orientation"` ，如果想指定多个值可以用`android:configChanges="orientation|keyboardHidden"` 。常用的configChanges的项目和含义：
  + locale 设备的本地位置发生了变化，一般指切换了系统语言
  + orientation 屏幕方向发生变化，最常用。
  + keyboardHidden 键盘的可访问性发生变化，如用户调出了键盘