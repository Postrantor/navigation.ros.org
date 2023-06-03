---
tip: translate by baidu@2023-06-03 18:02:48
...

# 2. Create New Planner and Controller Plugins {#create_plugins}

**Task description**

The ROS 2 Navigation Stack has a number of plugin interfaces to help users create or select specific plugins for planning, control, and behaviors for their applications. Two specific areas that the Nav2 stack could use more algorithm plugins for is for path planning (referred to as a planner plugin) and local trajectory generation (referred to as controller plugins). A simple tutorial for creating a [planner plugin can be found here.](https://navigation.ros.org/tutorials/docs/writing_new_nav2planner_plugin.html) Currently, we have one planner, NavFn which implements an A\* and Dijkstra\'s planner. It also has two controllers, DWB and TEB which implement a DWA and timed elastic-band optimization techniques. There is also a Hybrid-A\* and OMPL planner in development.

> ROS 2 导航堆栈有许多插件接口，可以帮助用户创建或选择特定的插件，用于为其应用程序进行规划、控制和行为。Nav2 堆栈可以使用更多算法插件的两个特定领域是路径规划（称为规划器插件）和局部轨迹生成（称为控制器插件）。创建[计划器插件的简单教程可以在这里找到。](https://navigation.ros.org/tutorials/docs/writing_new_nav2planner_plugin.html)目前，我们有一个规划师 NavFn，它实现了 A\*和 Dijkstra 的规划师。它还有两个控制器，DWB 和 TEB，它们实现了 DWA 和定时弹性带优化技术。还有一个 Hybrid-a\*和 OMPL 计划器正在开发中。

Your task will be to create a high-quality implementation of one of the following algorithms for the Nav2 plugin interfaces. Alternative algorithms may also be considered upon approval, please ask \@steve in the application phase. Please select only one to discuss.

> 您的任务是为 Nav2 插件接口创建以下算法之一的高质量实现。批准后也可以考虑替代算法，请在申请阶段询问\@steve。请仅选择一个进行讨论。

- Planner Plugin Options: D\* or variant, Vornoi planner, Navigation graph route planner, State Lattice planner, kinodynamic planner, and any planning method given a set of dynamic and static obstacles.

> - 规划器插件选项：D\*或变体、Vornoi 规划器、导航图路线规划器、State Lattice 规划器、kinodynamic 规划器，以及给定一组动态和静态障碍物的任何规划方法。

- Controller Plugin Options: CiLQR, iLQR, MPC, Splines, path following or dynamic obstacle following controllers.

> - 控制器插件选项：CiLQR、iLQR、MPC、样条曲线、路径跟随或动态障碍物跟随控制器。

- Additional options: helping in completing the OMPL or Hybrid-A\* planner.

**Project difficulty: High**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**
**Project output requirements**

- A functional planner or controller plugin for the Nav2 stack
- Plugin should be optimized for run-time performance with 50% or greater test coverage

**Skills required**

- C++
- Path planning or motion planning
- Algorithm optimization
- ROS / Pluginlib
- Recommended: Gazebo simulation and Navigation experience

**List of relevant open source software repositories and refs**

> **相关开源软件存储库和参考文献列表**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Github issue page](https://github.com/ros-planning/navigation2/issues/1710)
- [Nav2](https://navigation.ros.org/)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
