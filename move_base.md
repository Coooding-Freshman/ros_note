# MoveBase Insight
move base 是ros navigation最上层的一个包它里面包含了global&local planner
costmap等多个navigation核心组件。给定一个goal，move base可以给出一个可靠
移动路径。

## 成员变量
* MoveBaseState state_

&emsp; 有三种状态，plan-controlling-clearing 收到目标会置为为plan, plan完
变为controlling, reset的时候变成clearing, 如果我们make plan失败并且state_
还是 planning状态依旧会执行clearing，也算是一种自恢复操作

* MoveBaseActionServer* as_

&emsp; 监听goal请求，收到请求触发MoveBase::executeCb

* tf::TransformListener tf_

&emsp; MoveBase对象的构造需要的唯一参数一个TransformListener对象，这个变量是用来
构造costmap的

* vector<geometry::PoseStample\>* planner_plan_

&emsp; plan的时候用于接受costmap产生的最新plan，得到后给latest_plan_

* vector<geometry::PoseStample\>* latest_plan_

&emsp; 最新的规划得到后将上次的规划结果给planner_plan_

* vector<geometry::PoseStample\>* controller_plan_

&emsp; TODO

* boost::shared_ptr<nav_core::BaseGlobalPlanner> planner_

&emsp; 前面的数组里存的都是GlobalPlanner的结果

* boost::shared_ptr<nav_core::BaseLocalPlanner> tc_

&emsp; 接受global plann的结果并调用自身的setPlan的结果

* costmap_2d::Costmap2DROS* planner_costmap_ros_

* costmap_2d::Costmap2DROS* controller_costmap_ros_

* 还有很多成员变量，大多都是辅助性质，比如锁，条件变量，标志位等等

## 成员函数
* void MoveBase::executeCb(const move_base_msgs::MoveBaseGoalConstPtr& move_base_goal)

&emsp; as_的回调函数将得到的posestamp进行tf变换，publish等操作，唤醒planThead
线程进行plan，相当于整个规划程序的入口，它会处理所有的规划请求，如果有了新的
goal也可以停用对旧的goal的规划。

* void planThread();

&emsp; MoveBase一构造就会开启这个线程，析构时关闭，用于不断的make plan
其实底层的planner已经写好了，这里只需要不断地检查条件并且调用global&local
plan的make_plan函数就可以了。这里存在两个个标志位runPlanner_，给出goal的
时候runPlanner会置为true wait_for_wake会等刷新频率到了置为true

* bool makePlan(const geometry_msgs::PoseStamped& goal, std::vector<geometry_msgs::PoseStamped>& plan)

&emsp; 其实就是调用global plan的makePlan 检查一些条件做点tf变换而已

* bool executeCycle(goal, global_plan)

&emsp; 这个函数的注释是"the real work on pursuing a goal is done here"，当
我们得到一个path之后这个函数会进行一系列的检查，在适当状态下会通过
tc_(local_planner)计算velocity cmd 并将其广播出去

* 还有好多个recovery相关的函数，在这里就不探讨了。

## 总结

其实navigation的主要功能是在plan和costmap里面完成的，MoveBase已经是最外层的
接口，它的作用在用不到接受goal请求，检查状态，调用planner和map的 api
最后发送结果。无非就是actionserver接受goal唤醒planThread进行plan然后进入
controlling模式 检测current pose和目标之间的差距，不断控制agent移动。
