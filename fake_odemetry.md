# Fake Odemetry
ros-planning/navigation里面有这么一个包，里面用了fake的odemetry信息，模拟了
真实的场景。

## 参数
* odom_frame_id: "odem" 里程计信息

* base_frame_id: "base_link" 机器人本体

* global_frame_id: "/map" 地图

## 成员函数
* tf2::Transform m_offsetTf

* tf2_ros::MessageFilter<nav_msgs::Odometry\>* filter_
