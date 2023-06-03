---
tip: translate by openai@2023-06-03 00:27:42
...
---
title: Dashing to Eloquent
---


Moving from ROS 2 Dashing to Eloquent, a number of stability improvements were added that we will not specifically address here.

> 从ROS 2 Dashing到Eloquent迁移时，添加了一些稳定性改进，我们不会在此特别讨论。

# New Packages


Navigation2 now includes a new package `nav2_waypoint_follower`. The waypoint follower is an action server that will take in a list of waypoints to follow and follow them in order. There is a parameter `stop_on_failure` whether the robot should continue to the next waypoint on a single waypoint failure, or to return fail to the action client. The waypoint follower is also a reference application for how to use the Navigation2 action server to complete a basic autonomy task.

> Navigation2 现在包括一个新的包 `nav2_waypoint_follower`。Waypoint follower是一个动作服务器，它将接收一系列的路点并按顺序跟随它们。有一个参数`stop_on_failure`，表示在单个路点失败时，机器人是否继续到下一个路点，或者返回失败给动作客户端。Waypoint follower也是一个如何使用Navigation2动作服务器来完成基本自主任务的参考应用程序。


Navigation2 now supports new algorithms for control and SLAM. The Timed-Elastic Band (TEB) controller was implemented [and can be found here](https://github.com/rst-tu-dortmund/teb_local_planner). It is its own controller plugin that can be used instead of the DWB controller. Nav2 also supports SLAM Toolbox as the default SLAM implementation for ROS 2. This replaces the use of Cartographer.

> Nav2现在支持新的控制和SLAM算法。实现了定时弹性带（TEB）控制器，[可以在这里找到](https://github.com/rst-tu-dortmund/teb_local_planner)。它是自己的控制插件，可以代替DWB控制器使用。Nav2还支持SLAM Toolbox作为ROS 2的默认SLAM实现。这取代了使用Cartographer。

# New Plugins


Eloquent introduces back in pluginlib plugins to the navigation stack. `nav2_core` defines the plugin header interfaces to be used to implement controller, planner, recovery, and goal checker plugins. All algorithms (NavFn, DWB, recoveries) were added as plugin interfaces and the general packages for servers were created. `nav2_planner` is the action server for planning that hosts a plugin for the planner. `nav2_controller` is the action server for controller that hosts a plugin for the controller. `nav2_recovery` is the action server for recovery that hosts a plugin for recovery.

> Eloquent引入了在pluginlib插件中返回到导航堆栈中。 `nav2_core`定义了用于实现控制器、规划器、恢复和目标检查器插件的插件头接口。 所有算法（NavFn、DWB、恢复）都被添加为插件接口，并且创建了用于服务器的通用包。 `nav2_planner`是规划的动作服务器，它拥有一个规划器的插件。 `nav2_controller`是控制器的动作服务器，它拥有一个控制器的插件。 `nav2_recovery`是恢复的动作服务器，它拥有一个恢复的插件。


New recovery plugins were added including backup, which will take in a distance to back up, if collision-free. Additionally, the wait recovery was added that will wait a configurable period of time before trying to navigate again. This plugin is especially helpful for time-dependent obstacles or pausing navigation for a scene to become less dynamic.

> 新增了恢复插件，包括备份，可以远距离备份，如果无碰撞。另外，增加了等待恢复插件，可以在尝试导航之前等待可配置的时间段。这个插件对时间依赖的障碍或暂停导航以使场景变得不那么动态特别有用。


Many new behavior tree nodes were added. These behavior tree nodes are hard-coded in the behavior tree engine. Behavior tree cpp v3 supports plugins and will be converted in the next release.

> 许多新的行为树节点已经添加。这些行为树节点被硬编码在行为树引擎中。行为树cpp v3支持插件，并将在下一个版本中转换。

# Navigation2 Architectural Changes


The `nav2_world_model` package was removed. The individual `nav2_planner` and `nav2_controller` servers now host their relevent costmaps. This was done to reduce network traffic and ensure up-to-date information for the safety-critical elements of the system. As above mentions, plugins were introduced into the stack and these servers each host plugins for navigation, control, and costmap layers.

> `nav2_world_model` 包已被删除。单独的 `nav2_planner` 和 `nav2_controller` 服务器现在拥有它们的相关成本地图。这样做是为了减少网络流量，并确保系统安全关键元素的最新信息。正如上面提到的，插件被引入到堆栈中，这些服务器每个都拥有用于导航、控制和成本地图层的插件。


Map server was substantially refactored but the external API remains the same. It now uses the SDL library for image loading.

> 地图服务器经过大幅重构，但外部API保持不变。它现在使用SDL库来加载图像。


TF-based positioning is now used for pose-estimation everywhere in the stack. Prior, some elements of the navigation stack only updated its pose from the `/amcl_pose` topic publishing at an irregular rate. This is obviously low-accuracy and high-latency. All positioning is now based on the TF tree from the global frame to the robot frame.

> TF基于定位现在被用于姿态估计在堆栈的各个地方。之前，导航堆栈的一些元素只从`/amcl_pose`主题上发布以不规则的速率更新其姿态。这显然是低精度和高延迟。所有定位现在都是基于从全局框架到机器人框架的TF树。


Prior to Eloquent, there were no ROS 2 action servers and clients available. Navigation2, rather, used an interface we called Tasks. Eloquent now contains actions and a simple action server interface was created and is used now throughout the stack. Tasks were removed.

> 在Eloquent之前，没有ROS 2动作服务器和客户端可用。Navigation2，而是使用我们称为Tasks的接口。Eloquent现在包含了动作，并且创建了一个简单的动作服务器接口，现在在整个堆栈中使用。任务被移除了。
