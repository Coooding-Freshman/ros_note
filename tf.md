# TF学习笔记
tf 库是经常用到的一个库，但是这个库内有很多看起来差不多的class和function，很容易让人眼花缭乱
并且涉及到后来的tf2，让我很长时间摸不到头脑，这个md会零零散散记录一些笔记内容

先记住两个tf namespace 下面的 typedef:

```cpp
1. typedef tf::Transform Pose;
2. typedef tf::Vector3   Point;
```

### [tf::Transform](http://docs.ros.org/jade/api/tf/html/c++/classtf_1_1Transform.html)
一个基本的tf变换，两种构造方式

```cpp
1. tf::Transform(tf::Quaternion(0,0,0,1), tf::Vector3(0,0,0));

2. tf::Transform();
   obj.setOrigin(tf::Vector3(0,0,0));
   obj.setRotation(tf::Quaternion(0,0,0,1));
```
tf::Transform 有两个子类，并且看起来并不像父子 tf::Stamped<tf::Pose\> 以及 tf::StampedTransform.


### [tf::Stamped<T\> and tf::StampedTransform](http://docs.ros.org/jade/api/tf/html/c++/classtf_1_1Stamped.html)
tf::Stamped<T\> 是public继承 T 的。

原文不翻译
>The data type which will be cross compatable with geometry_msgs. This is the tf datatype equivilant of a MessageStamped.

按照文档来看 tf::Stamped&lt;tf::Transform&gt; ≈ tf::StampedTransform. 但是code上
又不是直接的模板特化 Orz.

### [tf::Transformer](http://docs.ros.org/jade/api/tf/html/c++/classtf_1_1Transformer.html)

这个类提供了寻找两个frame直接变换的接口, 常用的函数包括: lookupTransform, canTransform, getParent, waitforTransform ...

简单举一例:
```cpp
void transformPoint(const std::string& target_frame, const Stamped<tf::Point>& stamped_in, Stamped<tf::Point>& stamped_out) const;

void transformPoint(const std::string& target_frame, const Stamped<tf::Pose>& stamped_in, Stamped<tf::Pose>& stamped_out) const;
```

这个函数通过lookupTransform 寻找 两个点之间tf变换树。

### [tf::TransformListener](http://docs.ros.org/jade/api/tf/html/c++/classtf_1_1TransformListener.html)

public继承自 tf::Transformer

listen的对象不止局限在transform:
tranformPoint, transformPose, transformQuaternion, transformVector
分别提供了产生从geometry_msgs::PointStamped PoseStamped QuaternionStamped Vector3Stamped 到target_frame的变换的功能。

其实它的这些函数主要是做类型转换的
工作，把输入的msg转为tf 调用继承自父类里面的 tranformPoint 函数
再把输出从tf转为msg输出...

<br />
***
tf库的内容到此位置我们介绍一下tf2这个库，tf库其实也用到了很多tf2的组件

### [tf2::TransformStorage](https://github.com/ros/geometry2/blob/melodic-devel/tf2/include/tf2/transform_storage.h)

其实上面说的所有类都是接口，并没有存储真正的数据，tf2里面的TransformStorage
里面存储真正的数据，有五个成员变量:
```cpp
tf2::Quaternion rotation_;
tf2::Vector3 translation_;
ros::Time stamp_;
CompactFrameID frame_id_;
CompactFrameID child_frame_id_;
```

### [tf2::TimeCache and tf2::StaticCache](https://github.com/ros/geometry2/blob/melodic-devel/tf2/include/tf2/time_cache.h)
这两个类继承自同一个虚基类，但是TimeCache里面存的是
```cpp
typedef std::deque<TransformStorage> L_TransformStorage;
L_TransformStorage storage_;
```
StaticCache里面只有一个TransformStorage
说不清直接引用了：
>For greater efficiency tf now has a static transform topic "/tf_static".
>This topic has the same format as "/tf" however it is expected that
>any transform on this topic can be considered true for all time.
> Internally any query for a static transform will return true.

这里个里面的函数都是一些简单的get数据 insert数据的方法。

### [tf2::BufferCore](https://github.com/ros/geometry2/blob/melodic-devel/tf2/include/tf2/buffer_core.h) and [tf2_ros::Buffer](https://github.com/ros/geometry2/blob/melodic-devel/tf2_ros/include/tf2_ros/buffer.h)
tf::Transformer 里面有这么一句代码
```cpp
tf2_ros::Buffer tf2_buffer_;
```
其实 transformer里面的所有lookupTransform canTransform之类的函数都是直接在
里面调用tf2_buffer_.xxx() 而tf2_ros::Buffer是public继承自tf2::BufferCore的
这些都是BufferCore里面的接口


## 总结
这个库里面还有很多TFtoMsg MsgtoTF的函数还有一些transformBroadcaster transformListener
之类的类，这些无非就是默认的advertise 和 subscribe 用tf_msgs::TFMessage实例化模板
再做一些变换， 核心内容就是如上几个类, 封装的蛮深的不好看，呵呵。
