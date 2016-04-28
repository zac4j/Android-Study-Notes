####常见的滑动冲突场景
+ 外部滑动方向和内部滑动方向不一致（如外部是横向ScrollView，内部是ListView）
+ 外部滑动方向和内部滑动方向一致 (如嵌套ListView)
+ 以上两种情况嵌套

#####第一种滑动冲突解决方式
+ 外部拦截法
点击事件先经过父容器处理，如果父容器需要此事件就拦截，如果不需要就不拦截，这样就可以解决滑动冲突问题，这种方式比较符合点击事件的分发机制，外部拦截法需要重写父容器的onInterceptTouchEvent方法,pseudocode:
``` java
public boolean onInterceptTouchEvent(MotionEvent event) {
  boolean intercepted = false;
  int x = (int) event.getX();
	int y = (int) event.getY();
  switch(event.getAction()) {
	  case MotionEvent.ACTION_DOWN:
			intercepted = false;
			break;
	  case MotionEvent.ACTION_MOVE:
			intercepted = (if intercept current event ? true : false);
			break;
	  case MotionEvent.ACTION_UP:
			intercepted = false;
			break;
	}
	mLastXIntercept = x;
	mLastYIntercept = y;
	return intercepted;
}
```
对于上述代码，首先是ACTION_DOWN事件，父容器必须返回false,即不能拦截ACTION_DOWN事件，因为父容器一旦拦截了ACTION_DOWN,后续的ACTION_MOVE和ACTION_UP事件都会直接交由父容器处理。其次是ACTION_MOVE事件，这个事件可以根据需要来决定是否拦截，如果父容器拦截就返回true,否则返回false;最后是ACTION_UP事件，这里必须返回false,因为ACTION_UP事件本身没有太多意义。
+ 内部拦截法
内部拦截是指父容器不拦截任何事件，所有事件交由子元素，如果子元素需要此事件就直接消耗掉，否则就交给父容器处理，这种方法和Android中的事件分发机制不同，需要配合requestDisallowInterceptTouchEvent方法才能正常工作，使用起来较外部拦截稍微复杂。pseudcode:
``` java
public boolean dispatchTouchEvent(MotionEvent event) {
  int x = (int) event.getX();
	int y = (int) event.getY();
	switch (event.getAction()) {
		case MotionEvent.ACTION_DOWN:
			getParent().requestDisallowInterceptTouchEvent(true);
			break;
	  case MotionEvent.ACTION_MOVE:
			int deltaX = x - mLastX;
			int deltaY = y - mLastY;
			if (parent require intercept current event) {
				getParent().requestDisallowInterceptTouchEvent(false);
			}
			break;
	  case MotionEvent.ACTION_UP:
			intercepted = false;
			break; 
	}
  mLastX = x;
	mLastY = y;
	return super.dispatchTouchEvent(event);
}
```
上述代码同样保持父容器不拦截ACTION_DOWN，原因是ACTION_DOWN不受FLAG_DISALLOW_INTERCEPT这个flag控制，所以一旦父容器拦截ACTION_DOWN事件，那么所有的事件都无法传递到子元素中去，这样内部拦截就无法起作用了。父元素的修改如下：
``` java
public boolean onInterceptTouchEvent(MotionEvent event) {
  int action = event.getAction();
	return action != MotionEvent.ACTION_DOWN;
}
```
