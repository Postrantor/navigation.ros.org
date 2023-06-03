---
tip: translate by openai@2023-06-03 00:28:49
...
---
title: Galactic to Humble
---


Moving from ROS 2 Galactic to Humble, a number of stability improvements were added that we will not specifically address here.

> 从ROS 2 Galactic转移到Humble时，增加了许多稳定性改进，我们不会在这里具体讨论。

# Major improvements to Smac Planners


The Smac Planner was significantly improved, of both the 2D and Hybrid-A\* implementations, making the paths better, faster, and of higher quality.

> `Smac规划器的2D和Hybrid-A\*实现都得到了显著的改进，使路径更好、更快、质量更高。`

- Collision checker rejects collisions faster and queries the costmap for coordinates less often
- Zero-copy collision checking object
- precompute collision checking footprint orientations so no need for trig at runtime
- Only checking full SE2 footprint when the robot is in the possibly inscribed zones

- Computing the possibly inscribed zones, or the cost over which some part of the footprint may be in collision with a boundary to check the full footprint. Else, check center cost since promised to not be in a potential collision state

> 计算可能被插入的区域，或者某部分足迹可能与边界发生碰撞的成本，以检查完整的足迹。否则，检查中心成本，因为承诺不会处于潜在的碰撞状态。
- Renaming Hybrid-A\* planner to SmacPlannerHybrid
- Precomputing the Reedshepp and Dubin paths offline so at runtime its just a lookup table

- Replacing the wavefront heuristic with a new, and novel, heuristic dubbed the obstacle heuristic. This computes a Dijkstra\'s path using Differential A\* search taking into account the 8 connected space, as well as weights for the cost at the positions to guide the heuristic into the center of aisle ways. It also downsamples the costmap such that it can reduce the number of expansions by 75% and have a very small error introduced into the heuristic by being off by at most a partial fraction of a single cell distance

> 替换波前启发式算法为一种新的、创新的启发式算法，称为障碍启发式算法。这种算法使用Differential A\*搜索计算出Dijkstra的路径，考虑到8个连接的空间，以及位置的成本权重，以引导启发式算法进入过道的中心。它还将成本地图进行下采样，以便能够将扩展次数减少75％，并且通过最多仅仅减少单个单元距离的部分分数而对启发式算法引入非常小的误差。

- Improvements to the analytic expansion algorithm to remove the possibility of loops at the end of paths, whenever possible to remove

> 改善分析展開算法，以盡可能減少路徑末端出現迴圈的可能性。

- Improving analytic expansions to provide maximum path length to prevent skirting close to obstacles

> 改善分析展開以提供最大的路徑長度以防止接近障礙物。

- 2D A\* travel cost and heuristic improvements to speed up planning times and also increase the path quality significantly

> `2D A\*旅行成本和启发式改进以加快规划时间，并显著提高路径质量`
- Replaced smoother with a bespoke gradient descent implementation
- Abstract out common utilities of planners into a utils file
- tuned cost functions

- precomputed obstacle heuristic using dynamic programming to expand only the minimum number of nodes

> `使用动态规划预先计算的障碍启发式，仅扩展最少数量的节点`

- A caching heuristic setting to enable 25hz planning rates using cached obstacle heuristic values when the goal remains the same

> `缓存启发式设置，当目标保持不变时，使用缓存的障碍启发式值启用25hz计划速率`

- Leveraging the symmetry in the dubin and reeds-sheep space to reduce cache size by 50% to increase the window size available for heuristic lookup.

> 利用杜宾空间和里兹·希普空间的对称性，将缓存大小减少50％，以增加可用于启发式查找的窗口大小。
- Precompute primitives at all orientation bins
- SmacPlanner2D parameters are now all reconfigurable
- Both Hybrid-A\* and State Lattice planners are now fully admissible
- Hybrid-A\* and State Lattice have had their parameterization for path smoothing readded.
- The smoother now enables kinematically feasible boundary conditions.
- State Lattice supports turning in place primitive types

- Retrospective penalty added to speed up the planner, making it prioritize later search branches before earlier ones, which have negligible chance to improve path in vast majority of situations

> `给规划器新增了回顾性惩罚，使其优先考虑后期搜索分支，而不是早期的分支，在绝大多数情况下，这些分支改善路径的可能性微乎其微。`


The tl;dr of these improvements is:

> 简要总结这些改进：`Wait`

: - Plans are 2-3x as fast as they were before, well under 200ms for nearly all situations, making it as fast as NavFn and Global Planner (but now kinematically feasible). Typical planning times are sub-100ms without even making use of the caching or downsampling features. - Paths are of significantly higher quality via improved smoothers and a novel heuristic that steers the robot towards the center of aisleways implicitly. This makes smoother paths that are also further from obstacles whenever possible. - Using caching or downsampler parameterizations, can easily achieve path planning with sub-50ms in nearly any sized space. - Smoother is now able to do more refinements and can create kinematically feasible boundary conditions, even while reversing.


Additional improvements were made to include a `analytic_expansion_max_length` parameter such that analytic expansions are limited in their potential length. If the length is too far, reject this expansion. This prevents unsafe shortcutting of paths into higher cost areas far out from the goal itself, let search to the work of getting close before the analytic expansion brings it home. This should never be smaller than 4-5x the minimum turning radius being used, or planning times will begin to spike.

> 除了增加`analytic_expansion_max_length`参数以限制分析扩展的潜在长度外，还做了额外的改进。如果长度太远，请拒绝此扩展。这可以防止不安全的路径缩短到更高成本的区域，让搜索在接近目标之前完成工作，分析扩展将其带回家。这不应该小于正在使用的最小转弯半径的4-5倍，否则规划时间将开始攀升。


Further, the traversal cost and heuristic cost computations were updated **requiring retuning of your penalty functions** if you have a previously existing configuration. Defaults of the algorithm were also retuned appropriately to the change for similar our of the box behavior as before (to use as a reference).

> 此外，如果您有先前的配置，则需要重新调整您的惩罚函数，以更新遍历成本和启发式成本计算。算法的默认值也相应地重新调整，以便具有类似于以前的箱外行为（作为参考）。

# Simple (Python) Commander


[This PR 2411](https://github.com/ros-planning/navigation2/pull/2411) introduces a new package to Nav2, called the `nav2_simple_commander`. It is a set of functions in an object, `BasicNavigator`, which can be used to build Nav2-powered autonomy tasks in Python3 without concerning yourself with the Nav2, ROS 2, or Action server details. It contains a simple API taking common types (primarily `PoseStamped`) and handles all of the implementation details behind the hood. For example, this is a simple navigation task using this API:

> 此 PR 2411 引入了一个新的包到 Nav2，称为`nav2_simple_commander`。它是一组对象`BasicNavigator`中的函数，可用于在不关心Nav2，ROS 2或Action服务器细节的情况下使用Python3构建Nav2驱动的自主任务。它包含一个简单的API，接受常见类型（主要是`PoseStamped`），并处理背后的所有实现细节。例如，这是使用此API的一个简单导航任务：`Wait`

```python
    def main():
        rclpy.init()
        navigator = BasicNavigator()

        # Set our demo's initial pose
        initial_pose = PoseStamped()
        ... populate pose ...
        navigator.setInitialPose(initial_pose)

        # Wait for navigation to fully activate
        navigator.waitUntilNav2Active()

        # Go to our demos first goal pose
        goal_pose = PoseStamped()
        ... populate pose ...
        navigator.goToPose(goal_pose)

        while not navigator.isTaskComplete():
            feedback = navigator.getFeedback()
            ... do something with feedback ...

            # Basic navigation timeout
            if Duration.from_msg(feedback.navigation_time) > Duration(seconds=600.0):
                navigator.cancelNav()

        result = navigator.getResult()
        if result == TaskResult.SUCCEEDED:
            print('Goal succeeded!')
        elif result == TaskResult.CANCELED:
            print('Goal was canceled!')
        elif result == TaskResult.FAILED:
            print('Goal failed!')
```


[The full API can be found in the README of the package](https://github.com/ros-planning/navigation2/tree/main/nav2_simple_commander). A number of well commented examples and demos can also be found in the package\'s source code at the link prior.

> [完整的 API 可以在前面链接的软件包的 README 中找到](https://github.com/ros-planning/navigation2/tree/main/nav2_simple_commander)。也可以在前面链接的软件包的源代码中找到许多有注释的示例和演示。`Wait`

# Reduce Nodes and Executors


In order for nav2 to make the best use of ROS 2, we need minimize the number of nodes and executors in nav2, which can improve performance.

> 为了使nav2充分利用ROS 2，我们需要尽可能减少节点和执行器的数量，以提高性能。


This functionality has been discussed in [the ticket #816](https://github.com/ros-planning/navigation2/issues/816), and carried out in

> 这个功能已经在 [the ticket #816](https://github.com/ros-planning/navigation2/issues/816) 中讨论过，并且已经实施 `Wait`


- Remove `client_node_` in `class WaypointFollower` : [PR2441](https://github.com/ros-planning/navigation2/pull/2441)

> 删除类 WaypointFollower 中的 `client_node_`

- Remove `rclcpp_node_` in `class MapSaver` : [PR2454](https://github.com/ros-planning/navigation2/pull/2454)

> 移除 `class MapSaver` 中的 `rclcpp_node_`

- Remove `bond_client_node_` in `class LifecycleManager` : [PR2456](https://github.com/ros-planning/navigation2/pull/2456)

> 移除 `class LifecycleManager` 中的 `bond_client_node_`

- Remove `node_` in `class LifecycleManagerClient` : [PR2469](https://github.com/ros-planning/navigation2/pull/2469)

> 移除类 `LifecycleManagerClient` 中的 `node_`

- Remove `rclcpp_node_` in `class ControllerServer` : [PR2459](https://github.com/ros-planning/navigation2/pull/2459), [PR2479](https://github.com/ros-planning/navigation2/pull/2479)

> 移除 `class ControllerServer` 中的 `rclcpp_node_`：[PR2459](https://github.com/ros-planning/navigation2/pull/2459), [PR2479](https://github.com/ros-planning/navigation2/pull/2479)

- Remove `rclcpp_node_` in `class PlannerServer` : [PR2459](https://github.com/ros-planning/navigation2/pull/2459), [PR2480](https://github.com/ros-planning/navigation2/pull/2480)

> 移除`class PlannerServer`中的`rclcpp_node_`：[PR2459](https://github.com/ros-planning/navigation2/pull/2459)，[PR2480](https://github.com/ros-planning/navigation2/pull/2480)

- Remove `rclcpp_node_` in `class AmclNode` : [PR2483](https://github.com/ros-planning/navigation2/pull/2483)

> 移除 `class AmclNode` 中的 `rclcpp_node_`

- Remove `rclcpp_node_` and `clinet_node_` in `class Costmap2DROS` : [PR2489](https://github.com/ros-planning/navigation2/pull/2489)

> 移除`rclcpp_node_`和`clinet_node_`在`class Costmap2DROS`中：[PR2489](https://github.com/ros-planning/navigation2/pull/2489)

- Remove `rclcpp_node_` in `class LifecycleNode` : [PR2993](https://github.com/ros-planning/navigation2/pull/2993)

> 移除 `class LifecycleNode` 中的 `rclcpp_node_`


some APIs are changed in these PRs:

> 这些 PR 中有一些 API 已经更改了：`Wait`

: - [PR2489](https://github.com/ros-planning/navigation2/pull/2489) removes arguments `client_node`, `rclcpp_node` and adds argument `callback_group` in the initialize function of class `nav2_costmap_2d::Layer`. `callback_group` is used to replace `rclcpp_node`. - [PR2993](https://github.com/ros-planning/navigation2/pull/2993) removes argument `use_rclcpp_node` in the constructor of class `nav2_util::LifecycleNode`.

# API Change for nav2_core


[PR 2976](https://github.com/ros-planning/navigation2/pull/2976) changes the API for `nav2_core::Controller` and `nav2_core::Smoother` by replacing the use of shared pointer references `(const shared_ptr<> &)` to shared pointers `(shared_ptr<>)`. Use of shared pointer references meant that the shared pointer counter was never incremented.

> [PR 2976]更改了nav2_core::Controller和nav2_core::Smoother的API，通过用共享指针`(shared_ptr<>)`替换共享指针引用`(const shared_ptr<> &)`。使用共享指针引用意味着共享指针计数器永远不会增加。

# Extending the BtServiceNode to process Service-Results


[This PR 2481](https://github.com/ros-planning/navigation2/pull/2481) and [PR 2992](https://github.com/ros-planning/navigation2/pull/2992) address [the ticket](https://github.com/ros-planning/navigation2/issues/2467) and [this ticket](https://github.com/ros-planning/navigation2/issues/2968) and adds a virtual `on_completion()` function to the `BtServiceNode` class ([can be found here](https://github.com/ros-planning/navigation2/blob/c417e2fd267e1dfa880b7ff9d37aaaa7b5eab9ca/nav2_behavior_tree/include/nav2_behavior_tree/bt_service_node.hpp)). Similar to the already existing virtual `on_wait_for_result()` function, it can be overwritten in the child class to react to a respective event with some user-defined operation. The added `on_completion()` function will be called after the service interaction of the `BtServiceNode` has been successfully completed.

> 此 PR 2481 和 PR 2992 解决了 [此问题](https://github.com/ros-planning/navigation2/issues/2467) 和 [此问题](https://github.com/ros-planning/navigation2/issues/2968)，并且向 `BtServiceNode` 类添加了一个虚拟的 `on_completion()` 函数（[可在此处找到](https://github.com/ros-planning/navigation2/blob/c417e2fd267e1dfa880b7ff9d37aaaa7b5eab9ca/nav2_behavior_tree/include/nav2_behavior_tree/bt_service_node.hpp)）。与已有的虚拟 `on_wait_for_result()` 函数类似，它可以在子类中重写，以便在相应事件发生时用一些用户定义的操作来做出反应。添加的 `on_completion()` 函数将在 `BtServiceNode` 的服务交互成功完成后调用。

```cpp
    /**
    * @brief Function to perform some user-defined operation upon successful
    * completion of the service. Could put a value on the blackboard.
    * @param response can be used to get the result of the service call in the BT Node.
    * @return BT::NodeStatus Returns SUCCESS by default, user may override to return another value
    */
    virtual BT::NodeStatus on_completion(std::shared_ptr<typename ServiceT::Response>/*response*/)
    {
    return BT::NodeStatus::SUCCESS;
    }
```


The returned `BT::NodeStatus` will set the current status of the BT-Node. Since the function has access to the results of the service, the returned node-status can depend on those service results, for example. The normal behavior of the `BtServiceNode` is not affected by introducing the `on_completion()` function, since the the default implementation still simply returns `BT::NodeStatus::SUCCESS`, if the service interaction completed successfully.

> 返回的`BT::NodeStatus`将设置BT-Node的当前状态。由于该函数可以访问服务结果，因此返回的节点状态可以取决于这些服务结果，例如。引入`on_completion（）`函数不会影响`BtServiceNode`的正常行为，因为默认实现仍然只是在服务交互成功完成时返回`BT::NodeStatus::SUCCESS`。

# Including new Rotation Shim Controller Plugin


[This PR 2718](https://github.com/ros-planning/navigation2/pull/2718) introduces the new `nav2_rotation_shim_controller`. This controller will check the rough heading difference with respect to the robot and a newly received path. If within a threshold, it will pass the request onto the primary controller to execute. If it is outside of the threshold, this controller will rotate the robot towards that path heading. Once it is within the tolerance, it will then pass off control-execution from this rotation shim controller onto the primary controller plugin. At this point, the robot is still going to be rotating, allowing the current plugin to take control for a smooth hand off into path tracking.

> 这个 PR 2718 引入了新的`nav2_rotation_shim_controller`。这个控制器将检查机器人和新收到的路径的粗略航向差异。如果在阈值范围内，它将把请求传递给主控制器执行。如果超出阈值，这个控制器将旋转机器人朝向该路径航向。一旦它在公差范围内，它将把这个旋转 shim 控制器的控制执行转移到主控制器插件上。此时，机器人仍将旋转，允许当前插件获得控制，以实现平稳的路径跟踪手转。


The Rotation Shim Controller is suitable for:

> `活动垫片控制器适用于：`

- Robots that can rotate in place, such as differential and omnidirectional robots.

- Preference to rotate in place rather than \'spiral out\' when starting to track a new path that is at a significantly different heading than the robot\'s current heading.

> 当开始跟踪一条与机器人当前航向显著不同的新路径时，优先在原地旋转而不是`螺旋式移动`。

- Using planners that are non-kinematically feasible, such as NavFn, Theta\*, or Smac 2D (Feasible planners such as Smac Hybrid-A\* and State Lattice will start search from the robot\'s actual starting heading, requiring no rotation).

> 使用非运动可行的规划器，如NavFn、Theta*或Smac 2D（可行规划器如Smac Hybrid-A*和State Lattice将从机器人的实际起始航向开始搜索，无需旋转）。

# Spawning the robot in Gazebo


[This PR 2473](https://github.com/ros-planning/navigation2/pull/2473) deletes the pkg `nav2_gazebo_spawner` inside nav2_bringup directory. Instead of `nav2_gazebo_spawner` the Node [spawn_entity.py](https://github.com/ros-simulation/gazebo_ros_pkgs/blob/ros2/gazebo_ros/scripts/spawn_entity.py) of `gazebo_ros` is recomended to spawn the robot in gazebo. Note that

> 这个PR 2473删除了nav2_bringup目录中的pkg `nav2_gazebo_spawner`。建议使用`gazebo_ros`的节点[spawn_entity.py](https://github.com/ros-simulation/gazebo_ros_pkgs/blob/ros2/gazebo_ros/scripts/spawn_entity.py)来在Gazebo中生成机器人。请注意，`Wait`


- gazebo should be started with both `libgazebo_ros_init.so` and `libgazebo_ros_factory.so` to work correctly.

> 应该使用 `libgazebo_ros_init.so` 和 `libgazebo_ros_factory.so` 来正确启动 Gazebo。

- spawn_entity node could not remap /tf and /tf_static to tf and tf_static in the launch file yet, used only for multi-robot situations. This problem was overcame by adding remapping argument `<remapping>/tf:=tf</remapping>` `<remapping>/tf_static:=tf_static</remapping>` under ros2 tag in each plugin which publishs transforms in the SDF file. It is essential to differentiate the tf\'s of the different robot.

> 当使用多机器人时，spawn_entity节点仍然无法将/tf和/tf_static重映射为tf和tf_static。解决这个问题的方法是在SDF文件中的每个发布变换的插件下添加`<remapping>/tf:=tf</remapping>` `<remapping>/tf_static:=tf_static</remapping>`参数，以便区分不同机器人的tf。

# Recovery Behavior Timeout


Recoveries in Nav2, spin and backup, now have `time_allowance` ports in their BT nodes and request fields in their actions to specify a timeout. This helps ensure that the robot can exit a backup or spin primitive behavior in case it gets stuck or otherwise is unable to backup the full distance over a reasonable block of time.

> Nav2的恢复、旋转和备份现在在其BT节点中拥有`time_allowance`端口，并在其动作中指定超时请求字段。这有助于确保机器人可以在出现陷入困境或无法在合理的时间段内完成备份的情况下退出备份或旋转原语行为。

# New parameter `use_final_approach_orientation` for the 3 2D planners


[Pull request 2488](https://github.com/ros-planning/navigation2/pull/2488) adds a new parameter `use_final_approach_orientation` to the 3 2D planners (Theta\*, SmacPlanner2D and NavFn), `false` by default. If `true`, the last pose of the path generated by the planner will have its orientation set to the approach orientation, i.e. the orientation of the vector connecting the last two points of the path. It allows sending the robot to a position (x,y) instead of a pose (x,y,theta) by effectively ignoring the goal orientation. For example, below, for the same goal with an orientaton pointed left of the screen, `use_final_approach_orientation=false` (left) and `use_final_approach_orientation=true` (right)

> [拉取请求2488]（https://github.com/ros-planning/navigation2/pull/2488）为3个2D规划器（Theta *，SmacPlanner2D和NavFn）添加了一个新参数`use_final_approach_orientation`，默认为`false`。如果设置为`true`，规划器生成的路径的最后一个位置的姿态将设置为接近姿态，即连接路径的最后两个点的向量的姿态。它允许将机器人发送到位置（x,y）而不是姿态（x,y,theta），有效地忽略目标姿态。例如，下图中，对于具有面向屏幕左侧的方向的相同目标，`use_final_approach_orientation=false`（左）和`use_final_approach_orientation=true`（右）

![image](images/use_final_approach_orientation_false.gif){width="45.0%"}
![image](images/use_final_approach_orientation_true.gif){width="45.0%"}

# SmacPlanner2D and Theta\*: fix goal orientation being ignored


[This pull request 2488](https://github.com/ros-planning/navigation2/pull/2488) fixes [the issue](https://github.com/ros-planning/navigation2/issues/2482) of the goal pose orientation being ignored (the end path pose orientation was always set to 0).

> 这个拉取请求#2488修复了`issue #2482`，即忽略了目标位置朝向（最终路径位置朝向总是被设为0）的问题。

# SmacPlanner2D, NavFn and Theta\*: fix small path corner cases


[This PR 2488](https://github.com/ros-planning/navigation2/pull/2488) ensures the planners are not failing when the distance between the start and the goal is small (i.e. when they are on the same costmap cell), and in that case the output path is constructed with a single pose.

> 这个 PR 2488 确保在起点和目标之间距离很小（即在同一个costmap单元格中）时，规划器不会失败，在这种情况下，输出路径是用单个位姿构建的。

# Change and fix behavior of dynamic parameter change detection


[This](https://github.com/ros-planning/navigation2/pull/2576) and [this PR](https://github.com/ros-planning/navigation2/pull/2585) modify the method used to catch the changes of dynamic parameters. The motivation was to fix the issue that `void on_parameter_event_callback(const rcl_interfaces::msg::ParameterEvent::SharedPtr event)` was called for every parameter change of every node leading to unwanted parameter changes if 2 different nodes had the same parameter name.

> 这个[PR](https://github.com/ros-planning/navigation2/pull/2576)和[这个PR](https://github.com/ros-planning/navigation2/pull/2585)修改了捕获动态参数变化的方法。动机是为了解决`void on_parameter_event_callback(const rcl_interfaces::msg::ParameterEvent::SharedPtr event)`会对每个节点的每个参数变化调用的问题，如果两个不同的节点有相同的参数名称，就会导致不必要的参数变化。

# Dynamic Parameters


Newly added dynamic parameters to:

> 新增动态参数至：`Wait`

- [This PR 2592](https://github.com/ros-planning/navigation2/pull/2592) makes most of the Costmap2DROS parameters dynamic
- [This PR 2607](https://github.com/ros-planning/navigation2/pull/2607) makes most of the Regulated Pure Pursuit parameters dynamic
- [This PR 2665](https://github.com/ros-planning/navigation2/pull/2665) makes most of the Theta \* Planner parameters dynamic
- [This PR 2704](https://github.com/ros-planning/navigation2/pull/2704) makes Waypoint Follower, Planner Server, and Controller Server\'s params reconfigurable

# BT Action Nodes Exception Changes


When BT action nodes throw exceptions due to networking or action server failures, they now return a status code of `FAILURE` to fail that particular action in the behavior tree to react to. This is in contrast to prior where the exceptions thrown from networking were sent to the root tree which would count as a task-level failure that the tree could not react to.

> 当行为树中的BT动作节点由于网络或动作服务器故障而抛出异常时，它们现在会返回一个`FAILURE`状态码来使该特定动作失败，以便行为树可以做出反应。这与以前由网络抛出的异常被发送到根树，而根树将计算为树无法做出反应的任务级失败不同。

# BT Navigator Groot Multiple Navigators


[This PR 2627](https://github.com/ros-planning/navigation2/pull/2627) creates separate parameters for groot monitoring for the NavToPose and NavThroughPoses navigator types so you can individually track the state of each behavior tree through the ZMQ publisher. This resolves a long-standing problem after we added multiple navigator types to BT Navigator that you could only view the nav to poses BT execution live. BT.CPP and Groot only support one static ZMQ stream at a time, so there is a bit of a quirk where you must locally reset Groot after switching trees in order to view the live stream of the Nav Through Poses BT, if in use. This is a state of the BT.CPP and Groot libraries and not something we can resolve withing Nav2.

> 这个PR 2627为NavToPose和NavThroughPoses导航器类型创建了单独的Groot监控参数，因此您可以通过ZMQ发布者单独跟踪每个行为树的状态。在我们向BT Navigator添加多个导航器类型后，这解决了一个长期存在的问题，即您只能查看nav to poses BT执行的实时流。BT.CPP和Groot只支持一个静态ZMQ流，因此存在一个小问题，即如果使用Nav Through Poses BT，则必须在切换树之后本地重置Groot才能查看实时流。这是BT.CPP和Groot库的状态，而不是我们可以在Nav2中解决的问题。


There is some thought into the future regarding complete deprecation of live BT monitoring using Groot due to this quirk and the almost-certain infux of tickets on the topic. Groot will however always be supported for visualizing behavior tree XML files and modifications, simply not visualizing the BT execution live during robot navigation.

> 有一些关于完全废弃使用Groot进行实时BT监控的未来想法，由于这个毛病和几乎可以肯定的工单涌入，Groot将永远支持可视化行为树XML文件和修改，只是不再在机器人导航期间实时可视化BT执行`。

# Removed Kinematic Limiting in RPP


The parameters `max_linear_accel` and `max_linear_decel` were removed along with the kinematic limiting in the controller causing instabilities. Instead, use a velocity smoother available in the ROS ecosystem if you would like similar behavior.

> 参数`max_linear_accel`和`max_linear_decel`随着控制器中的运动学限制而被移除，导致不稳定性。如果您想要类似的行为，请使用ROS生态系统中可用的速度平滑器。

# Added Smoother Task Server


A new task server was added which loads smoother plugins and executes them to improve quality of an existing planned path. Smoothing action can be called from a behavior tree using SmoothPath action node. [PR 2569](https://github.com/ros-planning/navigation2/pull/2569) implements and [PR 2875](https://github.com/ros-planning/navigation2/pull/2875) adds in the first of the plugins using it with a simple smoother. Other smoothers are in development and will be added in the future.

> 新的任务服务器已添加，它加载更流畅的插件，并执行它们以改善现有计划路径的质量。可以从行为树中调用平滑操作，使用SmoothPath操作节点。[PR 2569]（https://github.com/ros-planning/navigation2/pull/2569）实施，[PR 2875]（https://github.com/ros-planning/navigation2/pull/2875）使用简单的平滑器添加了第一个插件。其他平滑器正在开发中，将来会添加。

# Removed Use Approach Velocity Scaling Param in RPP


The parameter `use_approach_linear_velocity_scaling` is removed in favor of always on to help in smooth transitions to the goal. [This PR 2701](https://github.com/ros-planning/navigation2/pull/2701) implements.

> 参数`use_approach_linear_velocity_scaling`被移除以便帮助平稳地转换到目标。[此PR 2701](https://github.com/ros-planning/navigation2/pull/2701)实现了。

# Refactored AMCL motion models as plugins


[This PR 2642](https://github.com/ros-planning/navigation2/pull/2642) creates plugins for the different motion models currently used in AMCL. This functionality enables users to use any custom motion model by creating it as a plugin and changing the robot_model_type parameter to the name of the plugin in nav2_params.yaml file. This helps to use custom motion models without the need to modify the AMCL source code.

> 这个PR 2642创建了AMCL当前使用的不同运动模型的插件。此功能使用户可以通过将其作为插件创建，并将robot_model_type参数更改为nav2_params.yaml文件中插件的名称，从而使用任何自定义运动模型。这有助于在不修改AMCL源代码的情况下使用自定义运动模型。

# Dropping Support for Live Groot Monitoring of Nav2

- <https://github.com/ros-planning/navigation2/pull/2642>


It was a great feature idea but never quite panned out, especially after we introduced multiple navigator types in the BT Navigator server. The issue we run into primarily is that Zero-MQ prevents users from producing multiple logger types in the same process. Since BT nav has multiple servers, the swapping between them for viewing has never had a clean hand off causing folks to file tickets or have nasty logs appear or ZMQ crashes in the background. The BT.CPP client for this doesn\'t allow us to have a clean shutdown process so we\'re left with hoping that ZMQ properly handles the situation, which it rarely does. Further, Groot only supports visualizing one type of tree at a time so for applications often switching between navigator types, its not possible to use a single groot client, causing great frustration.

> 这是一个很棒的功能想法，但在我们引入BT Navigator服务器的多种导航器类型后，从未真正实现。我们遇到的主要问题是，Zero-MQ阻止用户在同一进程中生成多个日志记录器类型。由于BT nav具有多个服务器，因此查看它们之间的切换从未有过干净的握手，导致人们提交工单或出现讨厌的日志，或者ZMQ在后台崩溃。这个BT.CPP客户端不允许我们有一个干净的关闭过程，所以我们只能希望ZMQ能够正确处理这种情况，但这很少发生。此外，Groot仅支持同时可视化一种类型的树，因此对于经常在导航器类型之间切换的应用程序，无法使用单个Groot客户端，这会带来巨大的挫折。


So, what I propose here is to remove live monitoring of the BT from Nav2. **We can still use Groot to modify, visualize, and generally work with behavior trees**, the only thing being removed is to live view the executing behavior tree as Nav2 is currently executing it (it used to light up the boxes of the current nodes). This was of dubious value anyhow, since the tree ticks so fast its difficult to visualize and get meaningful insights into things as the system is moving so quickly.

> 所以，我在这里提议移除Nav2的BT的实时监控。**我们仍然可以使用Groot来修改、可视化和一般处理行为树**，唯一被移除的只是当前Nav2正在执行的行为树的实时视图（它曾经点亮了当前节点的框）。而这本身也是有疑问的，因为树的响应速度太快，很难可视化并获得有意义的见解，因为系统运行太快了。

# Replanning Only if Path is Invalid


[This PR 2591](https://github.com/ros-planning/navigation2/pull/2591) creates two new condition BT node to facilitate replanning only if path becomes invalid rather than constantly replanning. These new nodes were integrated into the default BT.

> 此 PR 2591 创建了两个新的条件 BT 节点，以便在路径变得无效时进行重新规划，而不是不断重新规划。这些新节点已集成到默认的 BT 中。

# Fix CostmapLayer clearArea invert param logic


[This PR 2772](https://github.com/ros-planning/navigation2/pull/2772) fixes the invert paramlogic of the CostmapLayer clearArea fonction. Hence correcting the behavior of the clearAroundRobot and clearExceptRegion services and their corresponding BT actions.

> `[这个 PR 2772](https://github.com/ros-planning/navigation2/pull/2772) 修复了 CostmapLayer clearArea 函数的反向参数逻辑。因此纠正了 clearAroundRobot 和 clearExceptRegion 服务及其相应的 BT 操作的行为。`

# Dynamic Composition


[This PR 2750](https://github.com/ros-planning/navigation2/pull/2750) provides a optional bringup based on ROS2 dynamic composition for users. It can be used to compose all Nav2 nodes in a single process instead of launching these nodes separately, which is useful for embedded systems users that need to make optimizations due to harsh resource constraints. it\'s used by default, but can be disabled by using the launch argument `use_composition:=False`.

> 这个PR 2750提供了一个基于ROS2动态组合的可选启动，可用于将所有Nav2节点组合在一个进程中，而不是分别启动这些节点，这对于因为资源紧张而需要做出优化的嵌入式系统用户很有用。默认使用它，但可以通过使用启动参数`use_composition:=False`来禁用它。


Some experiments to show performance improvement of dynamic composition, and the cpu and memory are captured by `psutil` :

> 一些实验来展示动态组合的性能改进，并且CPU和内存由`psutil`捕获：`Wait`

> +------------------------------------------------------------+--------+-----------+
> | CPU: Intel(R) i7-8700 (6Cores 12Threads), Memory: 32GB | cpu(%) | memory(%) |
> +============================================================+========+===========+
> | > normal multiple processes | > 44 | > 0.76 |
> +------------------------------------------------------------+--------+-----------+
> | > dynamic composition (use `component_container_isolated`) | > 38 | > 0.23 |
> +------------------------------------------------------------+--------+-----------+


The way of dynamic composition consumes lower memory(saves \~70%), and lower cpu (saves \~13%) than normal multiple processes.

> 动态组合的方式比正常的多进程消耗更少的内存（节省约70%）和更少的CPU（节省约13%）。

# BT Cancel Node


[This PR 2787](https://github.com/ros-planning/navigation2/pull/2787) caters the users with an abstract node to develop cancel behaviors for different servers present in the Nav2 stack such as the controller_server, recovery_server and so on. As a start, this PR also provides the `CancelControl` behavior to cancel the goal given to the controller_server. As an addition to the `CancelControl` [This PR 2856](https://github.com/ros-planning/navigation2/pull/2856) provides the users with the option to cancel the recoveries such as the `backup`, `spin` and `wait`.

> 这个PR 2787为用户提供了一个抽象节点，用于开发Nav2堆栈中不同服务器（如controller_server，recovery_server等）的取消行为。作为一个开始，这个PR还提供了`CancelControl`行为来取消给定给controller_server的目标。作为对`CancelControl`的补充，PR 2856为用户提供了取消恢复（如`backup`，`spin`和`Wait`）的选项。

# BT PathLongerOnApproach Node


In the [PR](https://github.com/ros-planning/navigation2/pull/2802), a new Decorator BT node known as `PathLongerOnApproach` has been added to provide with the functionality to check and potentially handle longer path generated due to an obstacle in the given goal proximity. To demonstrate this functionality, a new BT `navigate_to_pose_w_replanning_goal_patience_and_recovery.xml` would serve both as an example and ready-to-use BT for a specific application that wishes to optimize their process cycle time. Demo of the developed BT can be seen below, where the robot pauses when close to a goal to see if the dynamic obstacle moves out of the way. Else, it executes the replan:

> 在[PR](https://github.com/ros-planning/navigation2/pull/2802)中，已添加了一个名为`PathLongerOnApproach`的新装饰器BT节点，用于检查并可能处理由于给定目标附近的障碍物而产生的较长路径。为了演示此功能，新的BT `navigate_to_pose_w_replanning_goal_patience_and_recovery.xml`既可以作为示例，也可以作为具体应用的即用BT，以优化其过程周期时间。下面可以看到开发的BT的演示，其中机器人在接近目标时暂停，以查看动态障碍物是否移开。否则，它会执行重新规划：`Wait`。


Obstacle does not clear at all, with [obstacle_clearance_time]{.title-ref} to be 3 seconds:

> 障碍根本不清除，障碍清除时间为3秒`Wait`

![image](images/nav2_patience_near_goal_and_clear_obstacle.gif)


Obstacle clears and you can see the robot pass through the (could have been ideally the) same path:

> 障碍清除，你可以看到机器人经过同样的路径：`Wait`

![image](images/nav2_patience_near_goal_and_go_around.gif)

# BT TruncatePathLocal Node


In the [PR 2753](https://github.com/ros-planning/navigation2/pull/2753), a new Action BT node named `TruncatePathLocal` has been added to extract a bounded-length path section near robot to be used e.g. for collision checking or computationally expensive smoothers

> 在[PR 2753](https://github.com/ros-planning/navigation2/pull/2753)中，添加了一个名为`TruncatePathLocal`的新动作BT节点，用于提取机器人附近的有界长度路径段，例如用于碰撞检查或计算上昂贵的更平滑器。

# Constrained Smoother


In [the PR 2753](https://github.com/ros-planning/navigation2/pull/2753), a new Smoother named `nav2_constrained_smoother::ConstrainedSmoother` has been added to optimize various path criteria such as smoothness or distance from obstacles, maintaining minimum turning radius

> 在[PR 2753](https://github.com/ros-planning/navigation2/pull/2753)中，已添加了一种新的平滑器`nav2_constrained_smoother::ConstrainedSmoother`，可以优化诸如平滑度或与障碍物的距离等各种路径标准，同时保持最小转弯半径`

# Replanning at a Constant Rate and if the Path is Invalid


[This PR 2804](https://github.com/ros-planning/navigation2/pull/2841) introduces a new behavior tree that navigates to pose with consistent replanning and if the path becomes invalid. To facilitate the new behavior tree a new condition node PathExpiringTimer was introduced to trigger replanning at a consistent rate.

> 这个PR 2804引入了一个新的行为树，它可以以一致的重新规划的方式导航到位姿，如果路径变得无效。为了促进新的行为树，引入了一个新的条件节点PathExpiringTimer，以稳定的速率触发重新规划。

# Euclidean Distance 2D


[This PR 2865](https://github.com/ros-planning/navigation2/pull/2865) changes Euclidean distance calculation throughout nav2 to project on to the XY plane (i.e. discard any information related to components in Z). This may potentially subtly change the way certain BT nodes, BT Navigators, controller servers, planner servers, and RPP behave if using custom plugins outside the Nav2 ecosystem.

> 这个PR 2865改变了nav2中的欧几里得距离计算，将其投影到XY平面（即丢弃与Z分量相关的任何信息）。如果使用Nav2生态系统之外的自定义插件，这可能会微妙地改变某些BT节点、BT导航器、控制器服务器、规划器服务器和RPP的行为。

# Recovery To Behavior


[This PR 2867](https://github.com/ros-planning/navigation.ros.org/pull/298) renames the nav2_recoveries to nav2_behaviors.

> 这个PR 2867将nav2_recoveries重命名为nav2_behaviors。


In navigation_launch.py recoveries_server -\> behavior_server and nav2_recoveries -\> nav2_behaviors. In nav2_params.yaml recovery_plugins -\> behavior_plugins and nav2_recoveries -\> nav2_behaviors.

> 在navigation_launch.py中，recoveries_server -\> behavior_server和nav2_recoveries -\> nav2_behaviors。在nav2_params.yaml中，recovery_plugins -\> behavior_plugins和nav2_recoveries -\> nav2_behaviors。

# Respawn Support in Launch and Lifecycle Manager


[PR 2752](https://github.com/ros-planning/navigation2/pull/2910) enables respawn support in Nav2. In the launch files, you may set `use_respawn` to `true` to enable respawning of servers that crash. This is only available in non-composed systems, since in composed systems, all of the nodes are under a single process and a crash anywhere will bring everything down (including the lifecycle manager itself). Even if the container was set to respawn, it would only respawn the empty container, not with all of the components loaded into it.

> [PR 2752](https://github.com/ros-planning/navigation2/pull/2910) 在 Nav2 中启用了重新启动支持。 在启动文件中，您可以将`use_respawn`设置为`true`以启用崩溃的服务器的重新启动。 这仅适用于非组合系统，因为在组合系统中，所有节点都在单个进程中，任何地方的崩溃都会导致所有内容都崩溃（包括生命周期管理器本身）。 即使设置了容器重新启动，它也只会重新启动空容器，而不是加载所有组件的容器。


That PR also enables the lifecycle manager to check if a system goes down due to a crash. If so, it allows the manager to check if the server comes back online within a given timeout period. If it does, it will automatically retransition the system back up to active to continue on its task automatically.

> 如果系统由于崩溃而停止，那么这个PR也可以使生命周期管理器检查。如果是这样，它允许管理周期在给定的超时期内检查服务器是否重新上线。如果是，它会自动将系统重新转换回活动状态，以继续其任务。

# New Nav2 Velocity Smoother


[PR 2964](https://github.com/ros-planning/navigation2/pull/2964) introduces the `nav2_velocity_smoother` for smoothing velocity commands from Nav2 to a robot controller by velocity, acceleration, and deadband constraints. See `configuring_velocity_smoother`{.interpreted-text role="ref"} for more details. It is not included in the default bringup batteries included from `nav2_bringup`.

> [PR 2964]（https://github.com/ros-planning/navigation2/pull/2964）引入了`nav2_velocity_smoother`，用于通过速度，加速度和死区约束平滑Nav2对机器人控制器的速度命令。有关详细信息，请参阅`configuring_velocity_smoother`{.interpreted-text role="ref"}。它不包括在`nav2_bringup`的默认bringup电池中。

# Goal Checker API Changed


[PR 2965](https://github.com/ros-planning/navigation2/pull/2965) adds an extra argument in the initialize function of the [nav2_core::GoalChecker]{.title-ref} class. The extra argument is a costmap_ros pointer. This is used to check if the goal is in collision, so that we can avoid moving towards the goal and replanning can be initiates using some BT plugin.

> [PR 2965](https://github.com/ros-planning/navigation2/pull/2965) 在 [nav2_core::GoalChecker]{.title-ref} 类的初始化函数中添加了一个额外的参数。额外的参数是一个costmap_ros指针。这用于检查目标是否发生碰撞，以便我们可以避免朝向目标移动，并使用一些BT插件启动重新规划。

# Added Assisted Teleop


[PR 2904](https://github.com/ros-planning/navigation2/pull/2904) adds a new behavior for assisted teleop along with two new BT nodes AssistedTeleop and CancelAssistedTeleop.

> [PR 2904](https://github.com/ros-planning/navigation2/pull/2904) 添加了一种新的辅助遥控行为，以及两个新的 BT 节点 AssistedTeleop 和 CancelAssistedTeleop。
