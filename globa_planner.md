# Global Planner Insight
navigation包里面有多个和 global plan相关的库，其中我们可以发现nav_core 里面
包含了一个纯虚基类[BaseGlobalPlanner](https://github.com/ros-planning/navigation/blob/kinetic-devel/nav_core/include/nav_core/base_global_planner.h), 
如果我们想构造的Global Planner能够被move_base使用 那么必须继承这个虚基类。

然后就可以发现其实整个库里面有三个继承自这个虚基类的Global Planner. 
carrot_planner, global_planner以及navfn里面的NavfunRos, 通过查阅move_base
的cfg文件我们知道，move_base默认使用的是[NavfunROS](https://github.com/ros-planning/navigation/blob/kinetic-devel/navfn/include/navfn/navfn_ros.h).
我们就主要来阅读一下这个库。

我们先看一下BaseGlobalPlanner里面的函数:
```cpp
virtual bool makePlan(const geometry_msgs::PoseStamped& start, 
       const geometry_msgs::PoseStamped& goal, std::vector<geometry_msgs::PoseStamped>& plan) = 0;
virtual void initialize(std::string name, costmap_2d::Costmap2DROS* costmap_ros) = 0;
```

初始化函数很普通ros库的套路，在构造函数里面调用这个。makePlan是GlobalPlanner的核心，讲Plan存在最后一个vector里面。

让后我们介绍一下NavfunROS这个类, 我们先明确一下求取路径分两步，首先我们要先计算到各个点的势能(Potential), 然后选一条势能最小的路径。

我们主要到其里面有这样一个成员变量
```cpp
boost::shared_ptr<NavFn> planner_;
```
在makePlan里面我们可以发现这样几行和其相关的代码
```cpp
254 planner_->setNavArr(costmap_->getSizeInCellsX(), costmap_->getSizeInCellsY());
255 planner_->setCostmap(costmap_->getCharMap(), true, allow_unknown_);
286 planner_->setStart(map_goal); 
287 planner_->setGoal(map_start);      
290 planner_->calcNavFnDijkstra(true);
```
然后余下的大部分就是条件检查，wordToMap之类的调用了,所以其实真正的工作都在[NavFn](https://github.com/ros-planning/navigation/blob/kinetic-devel/navfn/include/navfn/navfn.h)
这个类里面。NavfnROS只是一个对外的接口，除了makePlan外这个类里面只有一个getPathFromPotential函数比较重要，makePlan在290行计算完势能之后会调用这个函数，
这个函数会继续调用planner_里面的calcPath函数从得到的有势能梯度的数组里面找出一个最好的路径。

然后我们看一下 NavFn 这个类吧，这个类写的真的很烂...各种摸不到头脑的变量名, 比如pb1, pb2, pb3, 这种3p三兄弟，吐血.jpg
但是我们只要把握住一点，这个类是用来计算势能得到路径的，它里面必定要有一个member用来储存势能矩阵，一个member储存路径向量
顺着这个思路我们就找到了这三个成员变量：
```cpp
float *potarr;   /**< potential array, navigation function potential */ 
float *pathx, *pathy;     /**< path points, as subpixel cell coordinates */
```
一个将二维矩阵一维化形成创建了这个 nx * ny 长度的势能数组, 一个用两个数组表示起了一个二维向量组,
calcNavFnAstar 以及 calcNavFnDijkstra 的时候会更新potarr, calcPath的时候会更新 pathx pathy

一些设置函数我就不看了，算法也没仔细看，我感觉一维化后的Dijkstra是不是有什么特殊的技巧可以用啊，写的太辣眼睛，看不下去啊

## 总结

看起来似乎默认情况下会一直用Dijkstra寻路，并不知道为啥不用Astar，其实本质上就是在cosmap上的寻路算法应用，寻路算法这种东西搞不出来花啊 
