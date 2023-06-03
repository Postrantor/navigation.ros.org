---
tip: translate by openai@2023-06-03 00:28:05
title: Eloquent to Foxy
---

Moving from ROS 2 Eloquent to Foxy, a number of stability improvements were added that we will not specifically address here. We will specifically mention, however, the reduction in terminal noise. TF2 transformation timeout errors and warnings on startup have been largely removed or throttled to be more tractable. Additionally, message filters filling up resulting in messages being dropped were resolved in costmap 2d.

> 从 ROS 2 Eloquent 迁移到 Foxy 时，我们添加了许多稳定性改进，这里不做特别说明。不过，我们会特别提到终端噪音的减少。TF2 变换超时错误和启动时的警告已经得到大量移除或限制，以便更容易处理。此外，消息过滤器填满导致消息丢失的问题已经在 costmap 2d 中解决。

# General

The lifecycle manager was split into 2 unique lifecycle managers. They are the `navigation_lifecycle_manager` and `localization_lifecycle_manager`. This gives each process their own manager to allow users to switch between SLAM and localization without effecting Navigation. It also reduces the redundant code in `nav2_bringup`.

> 管理员被分成 2 个独特的管理员。它们是`navigation_lifecycle_manager`和`localization_lifecycle_manager`。这样，每个过程都有自己的管理员，可以让用户在 SLAM 和定位之间切换，而不会影响导航。它还减少了`nav2_bringup`中的冗余代码。

The lifecycle manager also now contains `Bond` connections to each lifecycle server. This means that if a server crashes or exits, the lifecycle manager will be constantly checking and transition down its lifecycle nodes for safety. This acts as a watchdog during run-time to complement the lifecycle manager\'s transitioning up and down from active states. See [this PR for details](https://github.com/ros-planning/navigation2/pull/1894).

> 现在，**生命周期管理器还包含与每个生命周期服务器的`Bond`连接**。这意味着，如果服务器崩溃或退出，生命周期管理器将不断检查并转换其生命周期节点以实现安全性。这在**运行时充当看门狗**，以补充生命周期管理器从活动状态上下转换的功能。有关详细信息，请参见[此 PR]。

> [!NOTE]
> .**运行时充当看门狗**

A fix to the BT navigator was added to remove a rare issue where it may crash due to asynchronous issues. As a result, a behavior tree is created for each navigation request rather than resetting an existing tree. The creation of this tree will add a small amount of latency. Proposals to reduce this latency will be considered before the next release.

> 修复了 BT 导航器以消除一个罕见的问题，即由于异步问题可能会崩溃。因此，为每个导航请求创建了行为树，而不是重置现有树。创建此树将增加一定的延迟。在下一个版本发布之前，将考虑减少这种延迟的建议。

# Server Updates

All plugin servers (controller, planner, recovery) now supports the use of multiple plugins. This can be done by loading a map of plugins, mapping the name of the plugin to its intended use-case. Each server defines a parameter where the list of names for the plugins to be loaded can be defined.

> 所有插件服务器（控制器、计划器、恢复）现在支持使用多个插件。这可以通过加载插件映射来完成，将插件的名称映射到其预期的用例。每个服务器定义一个参数，可以在其中定义要加载的插件名称列表。

```
-------------------------------------------------
Server Name             Plugin Parameter
----------------------- -------------------------
Controller Server       progress_checker_plugin

Controller Server       goal_checker_plugin

Controller Server       controller_plugins

Planner Server          planner_plugins

Recovery Server         recovery_plugins

Costmap Node            plugins
-------------------------------------------------
```

The type of plugin to be mapped to a particular name has to be defined using the `plugin` parameter in the plugin namespace for each name defined in the server plugin list. Each name in the plugin parameter list is expected to have the `plugin` parameter defined.

> 插件要映射到特定名称，必须使用插件命名空间中的`plugin`参数来定义，每个在服务器插件列表中定义的名称均需要定义`plugin`参数。插件参数列表中的每个名称都需要定义`plugin`参数。

An example: `controller_server` defines the parameter `controller_plugins` where a list of plugin names can be defined:

> 一个例子：`controller_server` 定义了参数 `controller_plugins`，可以在其中定义一个插件名称列表：`

```yaml
controller_server:
  ros__parameters:
    controller_plugins: ["FollowPath", "DynamicFollowPath"]
    FollowPath:
      plugin: "dwb_core/DWBLocalPlanner"
      max_vel_x: 0.26
    DynamicFollowPath:
      plugin: "teb_local_planner/TEBLocalPlanner"
      max_vel_x: 0.5
```

`FollowPath` controller is of type `dwb_local_planner/DWBLocalPlanner` and `DynamicFollowPath` of type `teb_local_planner/TEBLocalPlanner`. Each plugin will load the parameters in their namespace, e.g. `FollowPath.max_vel_x`, rather than globally in the server namespace. This will allow multiple plugins of the same type with different parameters and reduce conflicting parameter names.

> `FollowPath`controller 的类型为`dwb_local_planner/DWBLocalPlanner`，而`DynamicFollowPath`的类型为` teb_local_plan ner/TEBLocalPlanner`。每个插件将在其命名空间中加载参数，例如“FollowPath.max_vel_x”，而不是在服务器命名空间中全局加载。这将允许具有不同参数的同一类型的多个插件，并减少冲突的参数名称。

DWB Contains new parameters as an update relative to the ROS 1 updates, [see here for more information](https://github.com/ros-planning/navigation2/pull/1501). Additionally, the controller and planner interfaces were updated to include a `std::string name` parameter on initialization. This was added to the interfaces to allow the plugins to know the namespace it should load its parameters in. E.g. for a controller to find the parameter `FollowPath.max_vel_x`, it must be given its name, `FollowPath` to get this parameter. All plugins will be expected to look up parameters in the namespace of its given name.

> DWB 包含了相对于 ROS 1 更新的新参数，[更多信息请参见此处](https://github.com/ros-planning/navigation2/pull/1501)。此外，控制器和规划器接口也已更新，在初始化时包含一个`std::string name`参数。这个参数被添加到接口中，以允许插件知道它应该加载参数的命名空间。例如，为了获取参数`FollowPath.max_vel_x`，控制器必须被给予其名称`FollowPath`。所有插件都应该在其给定名称的命名空间中查找参数。

# New Plugins

Many new behavior tree nodes were added. These behavior tree nodes are now BT plugins and dynamically loadable at run-time using behavior tree cpp v3. The default behavior trees have been upgraded to stop the recovery behaviours and trigger a replanning when the navigation goal is preempted. See `nav2_behavior_tree` for a full listing, or `plugins`{.interpreted-text role="ref"} for the current list of behavior tree plugins and their descriptions. These plugins are set as default in the `nav2_bt_navigator` but may be overridden by the `bt_plugins` parameter to include your specific plugins.

> 许多新的行为树节点已被添加。这些行为树节点现在是 BT 插件，可以使用行为树 cpp v3 在运行时动态加载。默认的行为树已经升级，以停止恢复行为并在导航目标被抢占时触发重新规划。有关完整列表，请参见`nav2_behavior_tree`，或`plugins`{.interpreted-text role="ref"}以获取当前行为树插件及其描述的列表。这些插件默认设置在`nav2_bt_navigator`中，但可以通过`bt_plugins`参数覆盖，以包括您的特定插件。

Original GitHub tickets:

- [DistanceController](https://github.com/ros-planning/navigation2/pull/1699)
- [SpeedController](https://github.com/ros-planning/navigation2/pull/1744)
- [GoalUpdatedCondition](https://github.com/ros-planning/navigation2/pull/1712)
- [DistanceTraveledCondition](https://github.com/ros-planning/navigation2/pull/1705)
- [TimeExpiredCondition](https://github.com/ros-planning/navigation2/pull/1705)
- [UpdateGoal](https://github.com/ros-planning/navigation2/pull/1859)
- [TruncatePath](https://github.com/ros-planning/navigation2/pull/1859)
- [IsBatteryLowCondition](https://github.com/ros-planning/navigation2/pull/1974)
- [ProgressChecker](https://github.com/ros-planning/navigation2/pull/1857)
- [GoalChecker](https://github.com/ros-planning/navigation2/pull/1857)

# Map Server Re-Work

`map_saver` was re-worked and divided into 2 parts: CLI and server. CLI part is a command-line tool that listens incoming map topic, saves map once into a file and finishes its work. This part is remained to be almost untouched: CLI executable was renamed from `map_saver` to `map_saver_cli` without changing its functionality. Server is a new part. It spins in the background and can be used to save map continuously through a `save_map` service. By each service request it tries to listen incoming map topic, receive a message from it and write obtained map into a file.

> `map_saver`被重新设计并分为 2 个部分：CLI 和服务器。CLI 零件是一种命令行工具，可听取传入的地图主题，将映射保存到文件中并完成其工作。该部分几乎是未触及的：CLI 可执行文件可从“ map_saver”重命名为`map_saver_cli`，而无需更改其功能。服务器是新部分。它在后台旋转，可用于通过`save_map`服务连续保存地图。通过每个服务请求，它试图收听传入的地图主题，从中接收一条消息，然后写入映射到文件中。

`map_server` was dramatically simplified and cleaned-up. `OccGridLoader` was merged with `MapServer` class as it is intended to work only with one `OccupancyGrid` type of messages in foreseeable future.

> `map_server`被大大简化和清理。`ockgridloader`与`Mapserver`类合并，因为它仅在可预见的将来与一种``占用格里德）类型的消息合并。

Map Server now has new `map_io` dynamic library. All functions saving/loading `OccupancyGrid` messages were moved from `map_server` and `map_saver` here. These functions could be easily called from any part of external ROS 2 code even if Map Server node was not started.

> 现在，Map Server 有一个新的`map_io`动态库。所有保存/加载`OccupancyGrid`消息的函数都从`map_server`和`map_saver`移动到了这里。即使没有启动 Map Server 节点，也可以从外部 ROS 2 代码的任何部分轻松调用这些函数。

`map_loader` was completely removed from `nav2_util`. All its functionality already present in `map_io`. Please use it in your code instead.

Please refer to the [original GitHub ticket](https://github.com/ros-planning/navigation2/issues/1010) and [Map Server README](https://github.com/ros-planning/navigation2/blob/main/nav2_map_server/README.md) for more information.

> 请参阅[原始 GitHub 工单](https://github.com/ros-planning/navigation2/issues/1010)和[Map Server README](https://github.com/ros-planning/navigation2/blob/main/nav2_map_server/README.md)了解更多信息。`Wait`

# New Particle Filter Messages

New particle filter messages for particle clouds were added to include the particle weights along with their poses. `nav2_msgs/Particle` defines a single particle with a pose and a weight in a particle cloud. `nav2_msgs/ParticleCloud` defines a set of particles, each with a pose and a weight.

> 新的粒子滤波消息已添加到粒子云中，其中包括粒子权重及其姿态。`nav2_msgs/Particle`定义了一个粒子云中的单个粒子，其中包括姿态和权重。`nav2_msgs/ParticleCloud`定义了一组粒子，每个粒子都有一个姿态和权重。

`AMCL` now publishes its particle cloud as a `nav2_msgs/ParticleCloud` instead of a `geometry_msgs/PoseArray`.

[See here for more information.](https://github.com/ros-planning/navigation2/pull/1677)

> 请点击此处查看更多信息：`https://github.com/ros-planning/navigation2/pull/1677`

# Selection of Behavior Tree in each navigation action

The `NavigateToPose` action allows now to select in the action request the behavior tree to be used by `bt_navigator` for carrying out the navigation action through the `string behavior_tree` field. This field indicates the absolute path of the xml file that will be used to use to carry out the action. If no file is specified, leaving this field empty, the default behavior tree specified in the `default_bt_xml_filename parameter` will be used.

> `NavigateToPose`动作现在可以通过`string behavior_tree`字段在动作请求中选择`bt_navigator`用于执行导航动作的行为树。此字段指示将用于执行此操作的 xml 文件的绝对路径。如果未指定文件，即留空此字段，则将使用`default_bt_xml_filename参数`中指定的默认行为树。

This functionality has been discussed in [the ticket #1780](https://github.com/ros-planning/navigation2/issues/1780), and carried out in [the pull request #1784](https://github.com/ros-planning/navigation2/pull/1784).

> 这个功能已经在[工单#1780](https://github.com/ros-planning/navigation2/issues/1780)中讨论过，并且在[拉取请求#1784](https://github.com/ros-planning/navigation2/pull/1784)中实施了。

# FollowPoint Capability

A new behavior tree `followpoint.xml` has added. This behavior tree makes a robot follow a dynamically generated point, keeping a certain distance from the target. This can be used for moving target following maneuvers.

> 一个新的行为树`followpoint.xml`已经添加。这个行为树使机器人跟随一个动态生成的点，保持一定的距离。这可以用于移动目标跟踪机动。

This functionality has been discussed in [the ticket #1660](https://github.com/ros-planning/navigation2/issues/1660), and carried out in [the pull request #1859](https://github.com/ros-planning/navigation2/issues/1859).

> 这个功能已经在[工单#1660](https://github.com/ros-planning/navigation2/issues/1660)中讨论过，并且在[拉取请求#1859](https://github.com/ros-planning/navigation2/issues/1859)中实施。

# New Costmap Layer

The range sensor costmap has not been ported to navigation2 as `nav2_costmap_2d::RangeSensorLayer"`. It uses the same probabilistic model as the [ROS1](http://wiki.ros.org/range_sensor_layer) layer as well as much of the same interface. Documentation on parameters has been added to docs/parameters and the navigation.ros.org under `Configuration Guide`.

> `范围传感器成本图尚未移植到navigation2，其中`nav2_costmap_2d::RangeSensorLayer`使用与[ROS1](http://wiki.ros.org/range_sensor_layer)层相同的概率模型，以及大部分相同的界面。参数的文档已添加到docs/parameters，以及在`Configuration Guide`下的 navigation.ros.org 中。
