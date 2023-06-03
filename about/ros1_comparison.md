---
tip: translate by openai@2023-06-03 00:13:08
...
---
title: ROS to ROS 2 Navigation
---

`move_base` has been split into multiple components. Rather than a single monolithic state machine, navigation 2 makes use of action servers and ROS 2\'s low-latency, reliable communication to separate ideas. A behavior tree is used to orchestrate these tasks. This allows Navigation2 to have highly configurable navigation behavior without programming by rearranging tasks in a behavior tree xml file.


The _nav2_bt_navigator_ replaces `move_base` at the top level, with an _Action_ interface to complete a navigation task with a tree-based action model. It uses _Behavior Trees_ to make it possible to have more complex state machines and to add in behaviors as additional _Action Servers_. These behavior trees are configurable XMLs and we provide several starting examples.

> `nav2_bt_导航器`取代了最高级别的`move_base`，它拥有一个_Action_接口，可以用一个基于树的动作模型来完成导航任务。它使用_行为树_来使复杂的状态机成为可能，并将行为作为额外的_Action Servers_添加进来。这些行为树是可配置的XML，我们提供了几个起始示例。


The planning, behaviors, smoother, and controller servers are also action servers that the BT navigator can call to compute. All 4 servers can host many plugins of many algorithms each and individually called from the navigation behavior tree for specific behaviors. These servers are called from the BT navigator through their action servers to compute a result or complete a task. The state is maintained by the BT navigator behavior tree.

> 规划、行为、平滑和控制服务器也是BT导航器可以调用的动作服务器。这4个服务器可以各自拥有许多算法的插件，并可以从导航行为树中单独调用。这些服务器通过它们的动作服务器从BT导航器调用，以计算结果或完成任务。状态由BT导航器行为树维护。


All these changes make it possible to replace any of these nodes at launch/run time with any other algorithm that implements that same interface. See each package README.md for more details.

> 所有这些更改都使得可以在启动/运行时用任何实现相同接口的其他算法替换这些节点中的任何一个。有关更多详细信息，请参阅每个包的README.md。

![Move Base -> Navigation2 Overview](images/move_base_compare_2.png){.align-center width="700px"}

Note: `nav2_simple_navigator` no longer exists, it has been replaced by `nav2_bt_navigator`.


**In Summary:**

> `总而言之：`


Ported packages:

> `移植的软件包：`

> - amcl: Ported to nav2_amcl
> - map_server: Ported to nav2_map_server
> - nav2_planner: Replaces global_planner, hosts `N` planner plugins
> - nav2_controller: Replaces local_planner, hosts `N` controller plugins
> - Navfn: Ported to nav2_navfn_planner
> - DWB: Replaces DWA and ported to ROS 2 under nav2_dwb_controller metapackage
> - nav_core: Ported as nav2_core with updates to interfaces
> - costmap_2d: Ported as nav2_costmap_2d


New packages:

> 新包裹：`Wait`

> - nav2_bt_navigator: replaces `move_base` state machine
> - nav2_lifecycle_manager: Handles the server program lifecycles
> - nav2_waypoint_follower: Can take in many waypoints to execute a complex task through
> - nav2_system_tests: A set of integration tests for CI and basic tutorials in simulation
> - nav2_rviz_plugins: An rviz plugin to control the Navigation2 servers, command, cancel, and navigation with
> - navigation2_behavior_trees: wrappers for the behavior tree library to call ROS action servers


among many others.

> 在其他许多之中。
