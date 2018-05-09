# TF学习笔记
tf 库是经常用到的一个库，但是这个库内有很多看起来差不多的class和function，很容易让人眼花缭乱
并且涉及到后来的tf2，让我很长时间摸不到头脑，这个md会零零散散记录一些笔记内容

先记住两个tf namespace 下面的 typedef:

```cpp
1. typedef tf::Transform Pose;
2. typedef tf::Vector3   Point;
```

## [tf::Transform](http://docs.ros.org/jade/api/tf/html/c++/classtf_1_1Transform.html)
一个基本的tf变换，两种构造方式

```cpp
1. tf::Transform(tf::Quaternion(0,0,0,1), tf::Vector3(0,0,0));

2. tf::Transform();
   obj.setOrigin(tf::Vector3(0,0,0));
   obj.setRotation(tf::Quaternion(0,0,0,1));
```
tf::Transform 有两个子类，并且看起来并不像父子 tf::Stamped<tf::Pose\> 以及 tf::StampedTransform.


## [tf::Stamped<T\>](http://docs.ros.org/jade/api/tf/html/c++/classtf_1_1Stamped.html)
tf::Stamped<T\> 是public继承 T 的。

原文不翻译 "The data type which will be cross compatable with geometry_msgs This is the tf datatype equivilant of a MessageStamped. "

按照文档来看 tf::Stamped<tf::Transform\> ≈ tf::StampedTransform. 但是code上
又不是直接的模板特化 Orz.

## [tf::Transformer](http://docs.ros.org/jade/api/tf/html/c++/classtf_1_1Transformer.html)

这个类提供了寻找两个frame直接变换的接口, 常用的函数包括: lookupTransform, canTransform, getParent, waitforTransform ...

简单举一例:
```cpp
void transformPoint(const std::string& target_frame, const Stamped<tf::Point>& stamped_in, Stamped<tf::Point>& stamped_out) const;

void transformPoint(const std::string& target_frame, const Stamped<tf::Pose>& stamped_in, Stamped<tf::Pose>& stamped_out) const;
```

这个函数通过lookupTransform 寻找 两个点之间tf变换树。

## [tf::TransformListener](http://docs.ros.org/jade/api/tf/html/c++/classtf_1_1TransformListener.html)

public继承自 tf::Transformer

listen的对象不止局限在transform:
tranformPoint, transformPose, transformQuaternion, transformVector
分别提供了产生从geometry_msgs::PointStamped PoseStamped QuaternionStamped Vector3Stamped 到target_frame的变换的功能。

其实它的这些函数主要是做类型转换的工作，把输入的msg转为tf 调用继承自父类里面的 tranformPoint 函数
再把输出从tf转为msg输出...
