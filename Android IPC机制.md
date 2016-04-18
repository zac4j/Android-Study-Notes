---
date: 2016-04-17 11:13
status: public
title: IPC机制
---

####IPC概念基础
#####Serializable和Parcelable接口
+ Serializable是Java标准序列化接口，想让一个对象实现序列化，只需要实现Serializavble接口并声明一个`private static final long serialVersionUID `。通过Serializable方式来实现对象的序列化，实现起来非常简单，只需要采用ObjectOutputStream和ObjectInputStream即可实现。方式如下：
``` java
// Serialization
User user = new User(0, "Jake",true);
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("cache.text"));
out.writeObject(user);
out.close();

// Deserialization
ObjectInputStream in = new ObjectInputStream(new FileInputStream("cache.text"));
User newUser = (User)in.readObject();
in.close();
```
+ serialVersionUID的工作机制：
 序列化的时候系统会把当前类的serialVersionUID写入序列化的文件中，当发序列化的时候系统会去检测文件中的serialVersionUID，看它是否和当前类的serialVersionUID一致，如果信息一致，这个时候就可以反序列化；否则就无法完成，并 throw` java.io.InvalidClassException`.有两点需要注意：
 + 静态成员变量不属于对象，所以不参与序列化过程
 + 有transient关键字标记的成员变量不参与序列化过程。

+ Parcelable接口
实现这个接口，一个对象就可以实现序列化并可以通过Intent和Binder传递。Parcel内部包装了可序列化的数据，可以在Binder中自由传输。
 + Parcel序列化功能主要由writeToParcel方法完成，最终是通过Parcel中的一系列write方法实现。
 + Parcel反序列化由CREATOR完成，其内部表明了如何创建序列化对象和数组，并通过Parcel的一系列read方法完成反序列化过程；内容描述功能由describeContents方法来完成。
 
 Android系统提供了很多实现了Parcelable接口的类，它们都是可以直接序列化的，如Intent, Bundle, Bitmap等，同时List和Map也可以序列化，前提是它们里面的每个元素都是可序列化的。
+ 两者的区别，Serializable是Java中提供的standard序列化接口，其使用起来简单但开销很大，序列化和反序列化过程需要大量的I/O操作。而Parcelable是Android优化了的序列化接口，缺点是使用起来稍微麻烦，但是效率很高。

####Binder
+ 一般来说Binder是Android的一个类，它实现了IBinder接口。
 + 从IPC(Inter-Process Communication)角度来说，Binder是Android中的一种进程间通信方式。Binder可以理解为一种虚拟的物理设备，它的设备驱动是/dev/binder，这种通信方式在Linux上没有。
 + 从Android Framework角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManager，等）和相应ManagerService的桥梁
 + 从Android应用层来说，Binder是客户端和服务器端进行通信的媒介，当bindService的时候，服务端会返回一个包含了服务端业务调用Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务或者数据，这里的服务包括普通服务和基于AIDL的服务。
+ Android开发中，Binder主要在Service中，包括AIDL和Messager，其中普通Service中的Binder不涉及进程间通信，所以较为简单，而Messager的底层就是AIDL。