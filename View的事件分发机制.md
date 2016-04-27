####点击事件

View的点击事件的分发过程由三个方法共同完成：View.dispatchTouchEvent、ViewGroup.onInterceptTouchEvent和View.onTouchEvent.

+ public boolean dispatchTouchEvent(MotionEvent event)

用来进行事件的分发。如果事件能够传递给当前的View,此方法一定被调用，返回结果受View的onTouchEvent和下级View的dispatchTouchEvent方法的影响，表示comsume当前事件。

+ public boolean onInterceptTouchEvent(MotionEvent event)

在上述方法内部调用，用来拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列中，此方法不会被再次调用，返回结果表示拦截了当前事件。

+ public boolean onTouchEvent(MotionEvent event)

在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否consume当前事件，如果不consume，则在同一个事件序列中，当前View无法再次接收事件。



下面伪代码可以表示以上3者的关系:

``` java

public boolean dispatchTouchEvent(MotionEvent event) {

  boolean consume = false;

  if (onInterceptTouchEvent(event)) {

    consume = onTouchEvent(event);

   } else {

    consume = child.dispatchTouchEvent(event);

  }

  return consume;

}

```

###### 点击事件的传递规则

对于一个**Root ViewGroup**来说，点击事件产生后，首先会传递给它，这时它的`dispatchTouchEvent`就会被调用，如果这个**ViewGroup**的`onInterceptTouchEvent`方法返回true表示它要拦截当前事件，接着事件就交给这个**ViewGroup**处理，即它的`onTouchEvent`方法就会被调用；如果这个ViewGroup的`onInterceptTouchEvent`返回false就表示它不拦截当前事件，这时当前点击事件就会继续传递给它的子元素，接着子元素的`dispatchTouchEvent`方法就会被调用，如此反复直到事件被最终处理。



当一个View需要处理事件时，如设置了`OnTouchListener`,那么`OnTouchListener`中的onTouch方法会被回调。这时的事件处理就需要看onTouch的返回值，如果返回为false,则当前View的`onTouchEvent`方法会被调用；如果返回为true，则当前View的`onTouchEvent`方法不会被调用。另外如果当前View有设置onClickListener，那么它的onClick方法会在onTouchEvent之后被调用。由此对于View来说以上方法的优先级为：OnTouchListener --> onTouchEvent --> OnClickListener



当一个事件产生后，它的传递过程遵循如下顺序，Activity --> Window --> View,即事件总是先传递给Activity,Activity再传递给Window，最后Window再传递给顶级的View,顶级View接到事件后，就会按照View的事件分发机制去分发事件。

﻿
