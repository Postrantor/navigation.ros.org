---
tip: translate by openai@2023-06-03 00:28:29
...
---
title: Foxy to Galactic
---


Moving from ROS 2 Foxy to Galactic, a number of stability improvements were added that we will not specifically address here.

> 从ROS 2 Foxy到Galactic迁移时，我们添加了许多稳定性改进，这里我们不会具体讨论。

# NavigateToPose Action Feedback updates


The NavigateToPose action feedback has two improvements:

> `导航到位置动作反馈有两个改进：`


- `distance_remaining` now integrates path poses to report accurate distance remaining to go. Previously, this field reported the euclidean distance between the current pose and the goal pose.

> `剩余距离`现在集成路径姿态来报告准确的剩余距离。以前，这个字段报告当前姿态和目标姿态之间的欧几里得距离。

- Addition of `estimated_time_remaining` field. This field reports the estimated time remaining by dividing the remaining distance by the current speed.

> `添加`estimated_time_remaining`字段。 该字段通过将剩余距离除以当前速度来报告估计剩余时间。`

# NavigateToPose BT-node Interface Changes


The NavigateToPose input port has been changed to PoseStamped instead of Point and Quaternion.

> `输入端口NavigateToPose已经从Point和Quaternion改为PoseStamped。`


See `bt_navigate_to_pose_action`{.interpreted-text role="ref"} for more information.

> 請參閱 `bt_navigate_to_pose_action`{.interpreted-text role="ref"} 以獲取更多信息。

# NavigateThroughPoses and ComputePathThroughPoses Actions Added


The `NavigateThroughPoses` action has been added analog to the `NavigateToPose`. Rather than going to a single position, this Action will allow a user to specify a number of hard intermediary pose constraints between the start and final pose to plan through. The new `ComputePathThroughPoses` action has been added to the `planner_server` to process these requests through `N goal_poses`.

> `NavigateThroughPoses` 操作已经添加为类似`NavigateToPose`的操作。它不是直接到达一个位置，而是允许用户在起始位置和最终位置之间指定一些硬件中介位置来规划路径。新的`ComputePathThroughPoses`操作已经添加到`planner_server`来处理这些请求，通过`N goal_poses`。


The `ComputePathThroughPoses` action server will take in a set of `N` goals to achieve, plan through each pose and concatenate the output path for use in navigation. The controller and navigator know nothing about the semantics of the generated path, so the robot will not stop or slow on approach to these goals. It will rather continue through each pose as it were any other point on the path continuously. When paired with the `SmacPlanner`, this feature can be used to generate **completely kinematically feasible trajectories through pose constraints**.

> `ComputePathThroughPoses`动作服务器将接受一组`N`目标来实现，计划通过每个姿势，并连接输出路径以用于导航。控制器和导航器不知道生成路径的语义，因此机器人不会在接近这些目标时停止或减速。它会像在路径上的其他点一样继续穿过每个姿势。与`SmacPlanner`结合使用时，此功能可用于通过姿势约束生成**完全运动学可行的轨迹**。


If you wish to stop at each goal pose, consider using the waypoint follower instead, which will stop and allow a user to optionally execute a task plugin at each pose.

> 如果你想在每个目标姿势处停止，考虑使用航点跟踪器，它将停止并允许用户在每个姿势处可选择地执行任务插件。

# ComputePathToPose BT-node Interface Changes


The `start` input port has been added to optionally allow the request of a path from `start` to `goal` instead of from the current position of the robot to `goal`.

> `开始`输入端口已被添加，可选择性地允许从`开始`到`目标`的请求，而不是从机器人的当前位置到`目标`。


See `bt_compute_path_to_pose_action`{.interpreted-text role="ref"} for more information.

> 看`bt_compute_path_to_pose_action`{.interpreted-text role="ref"} 以获取更多信息。

# ComputePathToPose Action Interface Changes

- The goal pose field `pose` was changed to `goal`.
- The PoseStamped field `start` has been added.
- The bool field `use_start` has been added.


These two additional fields have been added to optionally allow, when `use_start` is true, the request of a path from `start` to `goal` instead of from the current position of the robot to `goal`. Corresponding changes have been done of the Planner Server.

> 这两个额外的字段已被添加，以允许在`use_start`为true时，从`start`到`goal`而不是从机器人当前位置到`goal`的路径请求。相应的更改已在Planner Server上完成。

# BackUp BT-node Interface Changes


The `backup_dist` and `backup_speed` input ports should both be positive values indicating the distance to go backward respectively the speed with which the robot drives backward.

> `备份_dist`和`备份_speed`输入端口都应该是正值，分别表示向后行驶的距离和机器人向后行驶的速度。

# BackUp Recovery Interface Changes

`speed` in a backup recovery goal should be positive indicating the speed with which to drive backward. `target.x` in a backup recovery goal should be positive indicating the distance to drive backward. In both cases negative values are silently inverted.

# Nav2 Controllers and Goal Checker Plugin Interface Changes


As of [this PR 2247](https://github.com/ros-planning/navigation2/pull/2247), the `controller` plugins will now be given a pointer to the current goal checker in use of the navigation task in `computeAndPublishVelocity()`. This is geared to enabling controllers to have access to predictive checks for goal completion as well as access to the state information of the goal checker plugin.

> 自[此PR 2247](https://github.com/ros-planning/navigation2/pull/2247)起，在`computeAndPublishVelocity()`中，`controller`插件现在将被给予一个指向当前导航任务中使用的目标检查器的指针。这旨在使控制器能够访问预测完成目标的检查，以及访问目标检查器插件的状态信息。


The `goal_checker` plugins also have the change of including a `getTolerances()` method. This method allows a goal checker holder to access the tolerance information of the goal checker to consider at the goal. Each field of the `pose` and `velocity` represents the maximum allowable error in each dimension for a goal to be considered completed. In the case of a translational tolerance (combined X and Y components), each the X and Y will be populated with the tolerance value because it is the **maximum** tolerance in the dimension (assuming the other has no error). If the goal checker does not contain any tolerances for a dimension, the `numeric_limits<double> lowest()` value is utilized in its place.

> `goal_checker`插件还包括了一个`getTolerances()`方法的改变。这个方法允许目标检查器持有者访问目标检查器的容忍度信息以考虑目标。`pose`和`velocity`的每个字段代表一个维度的最大允许误差，使目标被认为完成。在平移容忍度（组合X和Y分量）的情况下，每个X和Y将被填充为容忍度值，因为它是该维度的最大容忍度（假设另一个没有误差）。如果目标检查器在某个维度上没有任何容忍度，则使用`numeric_limits<double> lowest()`值代替。

# FollowPath goal_checker_id attribute


For example: you could use for some specific navigation motion a more precise goal checker than the default one that it is used in usual motions.

> 例如：您可以使用比通常运动使用的默认检查器更精确的目标检查器来实现某些特定的导航运动。`Wait`

```xml
<FollowPath path="{path}" controller_id="FollowPath" goal_checker_id="precise_goal_checker" server_name="FollowPath" server_timeout="10"/>
```


- The previous usage of the `goal_checker_plugin` parameter to declare the controller_server goal_checker is now obsolete and removed.

> `之前使用`goal_checker_plugin`参数声明controller_server goal_checker的用法已经过时，并已经被移除。`

- The controller_server parameters now support the declaration of a list of goal checkers `goal_checker_plugins` mapped to unique identifier names, such as is the case with `FollowPath` and `GridBased` for the controller and planner plugins, respectively.

> - 现在，控制器服务器参数支持将一组目标检查器`goal_checker_plugins`映射到唯一标识符名称，就像控制器和规划插件（分别为`FollowPath`和`GridBased`）一样。

- The specification of the selected goal checker is mandatory when more than one checker is defined in the controller_server parameter configuration. If only one goal_checker is configured in the controller_server it is selected by default even if no goal_checker is specified.

> 当在controller_server参数配置中定义了多个检查器时，必须指定所选目标检查器的规格。如果在controller_server中只配置了一个goal_checker，即使没有指定goal_checker，也会默认选择它。


Below it is shown an example of goal_checker configuration of the controller_server node.

> 以下是控制器服务器节点的goal_checker配置的示例。

```yaml
    controller_server:
    ros__parameters:
        goal_checker_plugins: ["general_goal_checker", "precise_goal_checker"]
        precise_goal_checker:
            plugin: "nav2_controller::SimpleGoalChecker"
            xy_goal_tolerance: 0.25
            yaw_goal_tolerance: 0.25
        general_goal_checker:
            plugin: "nav2_controller::SimpleGoalChecker"
            xy_goal_tolerance: 0.25
```

# Groot Support


Live Monitoring and Editing of behavior trees with Groot is now possible. Switching bt-xmls on the fly through a new goal request is also included. This is all done without breaking any APIs. Enabled by default.

> 使用Groot现在可以实时监控和编辑行为树。通过新的目标请求可以实时切换bt-xmls，而且不会破坏任何API。默认情况下启用。

# New Plugins

`nav2_waypoint_follower` has an action server that takes in a list of waypoints to follow and follow them in order. In some cases we might want robot to perform some tasks/behaviours at arrivals of these waypoints. In order to perform such tasks, a generic plugin interface [WaypointTaskExecutor]{.title-ref} has been added to `nav2_core`. Users can inherit from this interface to implement their own plugin to perform more specific tasks at waypoint arrivals for their needs.


Several example implementations are included in `nav2_waypoint_follower`. `WaitAtWaypoint` and `PhotoAtWaypoint` plusings are included in `nav2_waypoint_follower` as run-time loadable plugins. `WaitAtWaypoint` simply lets robot to pause for a specified amount of time in milliseconds, at waypoint arrivals. While `PhotoAtWaypoint` takes photos at waypoint arrivals and saves the taken photos to specified directory, the format for taken photos also can be configured through parameters. All major image formats such as `png`, `jpeg`, `jpg` etc. are supported, the default format is `png`.

> 在`nav2_waypoint_follower`中包含了几个示例实现。`WaitAtWaypoint`和`PhotoAtWaypoint`插件都包含在`nav2_waypoint_follower`中，作为运行时可加载的插件。`WaitAtWaypoint`只是让机器人在到达航点时，暂停一段指定的毫秒数。而`PhotoAtWaypoint`则会在到达航点时拍照，并将拍摄的照片保存到指定的目录，拍摄的照片格式也可以通过参数进行配置。所有主要的图像格式，如`png`、`jpeg`、`jpg`等都支持，默认格式为`png`。


Loading a plugin of this type is done through `nav2_bringup/params/nav2_param.yaml`, by specifying plugin\'s name, type and it\'s used parameters.

> 加载此类型的插件是通过`nav2_bringup/params/nav2_param.yaml`来完成的，通过指定插件的名称、类型和使用的参数。

```yaml
    waypoint_follower:
    ros__parameters:
        loop_rate: 20
        stop_on_failure: false
        waypoint_task_executor_plugin: "wait_at_waypoint"
        wait_at_waypoint:
            plugin: "nav2_waypoint_follower::WaitAtWaypoint"
            enabled: True
            waypoint_pause_duration: 0
```


Original GitHub tickets:

> `原始GitHub票证：`

- [WaypointTaskExecutor](https://github.com/ros-planning/navigation2/pull/1993)
- [WaitAtWaypoint](https://github.com/ros-planning/navigation2/pull/1993)
- [PhotoAtWaypoint](https://github.com/ros-planning/navigation2/pull/2041)
- [InputAtWaypoint](https://github.com/ros-planning/navigation2/pull/2049)

# Costmap Filters


A new concept interacting with spatial-dependent objects called \"Costmap Filters\" appeared in Galactic (more information about this concept could be found at `concepts`{.interpreted-text role="ref"} page). Costmap filters are acting as a costmap plugins, applied to a separate costmap above common plugins. In order to make a filtered costmap and change robot\'s behavior in annotated areas, filter plugin reads the data came from filter mask. Then this data is being linearly transformed into feature map in a filter space. It could be passability of an area, maximum speed limit in m/s, robot desired direction in degrees or anything else. Transformed feature map along with the map/costmap, sensors data and current robot position is used in plugin\'s algorithms to make required updates in the resulting costmap and robot\'s behavor.

> {.interpreted-text role="ref"}.

在Galactic中出现了一种新的与空间相关对象交互的概念，称为“Costmap Filters”（更多有关此概念的信息可以在`概念`{.interpreted-text role="ref"}页面找到）。Costmap过滤器充当一个costmap插件，应用于常见插件之上的单独的costmap。为了制作一个过滤后的costmap并改变注释区域中机器人的行为，过滤器插件从过滤掩码中读取数据。然后，该数据在过滤空间中被线性变换为特征图。它可以是一个区域的通行能力，以m/s为单位的最高速度限制，机器人所需的方向以度为单位或其他任何内容。将变换后的特征图与地图/ costmap，传感器数据和当前机器人位置一起使用，可以在插件的算法中对生成的costmap和机器人的行为进行所需的更新。


Architecturally, costmap filters consists from `CostmapFilter` class which is a basic class incorporating much common of its inherited filter plugins:

> 架构上，costmap过滤器由`CostmapFilter`类组成，该类是一个基本类，包含其继承的许多常用过滤插件：`Wait`

- `KeepoutFilter`: keep-out/safety zones filter plugin.
- `SpeedFilter`: slow/speed-restricted areas filter.

- Preferred lanes in industries. This plugin is covered by `KeepoutFilter` (see discussion in [corresponding PR](https://github.com/ros-planning/navigation2/issues/1522) for more details).

> 优先车道在工业中。此插件由`KeepoutFilter`覆盖（有关更多详细信息，请参见[相应的PR]（https://github.com/ros-planning/navigation2/issues/1522））。


Each costmap filter subscribes to filter info topic (publishing by [Costmap Filter Info Publisher Server](https://github.com/ros-planning/navigation2/tree/main/nav2_map_server/src/costmap_filter_info)) having all necessary information for loaded costmap filter and filter mask topic. `SpeedFilter` additionally publishes maximum speed restricting [messages](https://github.com/ros-planning/navigation2/blob/main/nav2_msgs/msg/SpeedLimit.msg) targeted for a Controller to enforce robot won\'t exceed given limit.

> 每个costmap过滤器都订阅过滤器信息主题（由[Costmap Filter Info Publisher Server]（https://github.com/ros-planning/navigation2/tree/main/nav2_map_server/src/costmap_filter_info）发布），其中包含加载的costmap过滤器和过滤器掩码主题所需的所有信息。 `SpeedFilter`还发布针对控制器的最大速度限制[消息]（https://github.com/ros-planning/navigation2/blob/main/nav2_msgs/msg/SpeedLimit.msg），以确保机器人不会超过给定的限制。


High-level design of this concept could be found [here](https://github.com/ros-planning/navigation2/tree/main/doc/design/CostmapFilters_design.pdf). The functionality of costmap filters is being disscussed in [the ticket #1263](https://github.com/ros-planning/navigation2/issues/1263) and carried out by [PR #1882](https://github.com/ros-planning/navigation2/pull/1882). The following tutorials: `navigation2_with_keepout_filter`{.interpreted-text role="ref"} and `navigation2_with_speed_filter`{.interpreted-text role="ref"} will help to easily get involved with `KeepoutFilter` and `SpeedFilter` functionalities.

> 高层设计可以在[这里](https://github.com/ros-planning/navigation2/tree/main/doc/design/CostmapFilters_design.pdf)找到。costmap滤波器的功能正在[问题 #1263](https://github.com/ros-planning/navigation2/issues/1263)中讨论，并通过[PR #1882](https://github.com/ros-planning/navigation2/pull/1882)实施。以下教程：`navigation2_with_keepout_filter`{.interpreted-text role="ref"}和`navigation2_with_speed_filter`{.interpreted-text role="ref"}将有助于您轻松掌握`KeepoutFilter`和`SpeedFilter`功能。

# SmacPlanner


A new package, `nav2_smac_planner` was added containing 4 or 8 connected 2D A\*, and Dubin and Reed-shepp model hybrid-A\* with smoothing, multi-resolution query, and more.

> 一个新的包`nav2_smac_planner`被添加，其中包含4或8个连接的2D A*，以及Dubin和Reed-shepp模型的混合A*，具有平滑处理、多分辨率查询等功能。


The `nav2_smac_planner` package contains an optimized templated A\* search algorithm used to create multiple A\*-based planners for multiple types of robot platforms. We support differential-drive and omni-directional drive robots using the `SmacPlanner2D` planner which implements a cost-aware A\* planner. We support cars, car-like, and ackermann vehicles using the `SmacPlanner` plugin which implements a Hybrid-A\* planner. This plugin is also useful for curvature constrained planning, like when planning robot at high speeds to make sure they don\'t flip over or otherwise skid out of control.

> `nav2_smac_planner` 包含一个优化的模板A*搜索算法，用于创建多个基于A*的规划器，适用于多种机器人平台。我们支持差动驱动和全向驱动机器人，使用 `SmacPlanner2D` 规划器实现成本感知A*规划器。我们支持汽车、汽车类型和阿克曼车辆，使用 `SmacPlanner` 插件实现混合A*规划器。此插件还可用于曲率约束规划，例如在高速度下规划机器人，以确保它们不会翻倒或以其他方式失控。


The `SmacPlanner` fully-implements the Hybrid-A\* planner as proposed in [Practical Search Techniques in Path Planning for Autonomous Driving](https://ai.stanford.edu/~ddolgov/papers/dolgov_gpp_stair08.pdf), including hybrid searching, CG smoothing, analytic expansions and hueristic functions.

> `SmacPlanner` 完全实现了[实用自动驾驶路径规划中的搜索技术](https://ai.stanford.edu/~ddolgov/papers/dolgov_gpp_stair08.pdf)中提出的混合A\*规划器，包括混合搜索、CG平滑、分析扩展和启发式函数。

# ThetaStarPlanner


A new package, `nav2_theta_star_planner` was added containing 4 or 8 connected Theta\* implementation for 2D maps.

> 一个新的包`nav2_theta_star_planner`被添加，其中包含4或8个连接的Theta\*实现用于2D地图。


This package implements an optimized version of the Theta\* Path Planner (specifically the [Lazy Theta\* P](http://idm-lab.org/bib/abstracts/papers/aaai10b.pdf) variant) to plan any-angled paths for differential-drive and omni-directional robots, while also taking into account the costmap costs. This plugin is useful for the cases where you might want to plan a path at a higher rate but without requiring extremely smooth paths around the corners which, for example, could be handled by a local planner/controller.

> 这个包实现了一个优化版本的Theta*路径规划器（特别是[Lazy Theta* P]（http://idm-lab.org/bib/abstracts/papers/aaai10b.pdf）变体），用于为差动驱动和全向机器人规划任意角度的路径，同时也考虑到成本地图的成本。这个插件对于你可能想要以更高的速率规划路径但不需要在拐角处极其平滑的路径的情况很有用，例如可以由局部规划器/控制器处理。

# RegulatedPurePursuitController


A new package, `nav2_regulated_pure_pursuit_controller` was added containing a novel varient of the Pure Pursuit algorithm. It also includes configurations to enable Pure Pursuit and Adaptive Pure Pursuit variations as well.

> 新的包`nav2_regulated_pure_pursuit_controller`已经添加，其中包含一种新颖的Pure Pursuit算法变体。它还包括配置，以启用Pure Pursuit和Adaptive Pure Pursuit变体。


This variation is specifically targeting service / industrial robot needs. It regulates the linear velocities by curvature of the path to help reduce overshoot at high speeds around blind corners allowing operations to be much more safe. It also better follows paths than any other variation currently available of Pure Pursuit. It also has heuristics to slow in proximity to other obstacles so that you can slow the robot automatically when nearby potential collisions. It also implements the Adaptive lookahead point features to be scaled by velocities to enable more stable behavior in a larger range of translational speeds.

> 这个变体专门针对服务/工业机器人的需求。它通过路径的曲率来调节线性速度，从而帮助减少高速下在盲角处的过冲，使操作更加安全。它也比现有的纯追踪变体更好地遵循路径。它还具有启发式算法，可以在接近其他障碍物时减慢速度，从而可以在附近潜在碰撞时自动减慢机器人。它还实现了自适应预览点功能，可以根据速度进行缩放，以实现更大范围的平移速度的稳定行为。


There\'s more this does, that that\'s the general information. See the package\'s `README` for more.

> 这还有更多功能，这只是一般信息。查看`README`了解更多。

# Costmap2D `current_` Usage


In costmap2D, `current_` was used in ROS1 to represent whether a costmap layer was still enabled and actively processing data. It would be turned to `false` only under the situation that the expected update rate of a sensor was not met, so it was getting stale or no messages. It acts as a fail-safe for if a navigation sensor stops publishing.

> 在Costmap2D中，`current_`在ROS1中用于表示一个成本地图层是否仍然启用并且正在处理数据。只有在传感器预期的更新速率没有得到满足的情况下，它才会被设置为`false`，因此它变得过时或没有消息。它充当导航传感器停止发布的故障保护。


In galactic, that will remain turn, however it will also add additional capabilities. It is also now set to `false` when a costmap is reset due to clearing or other navigation recoveries. That stops the robot from creating a plan or control effort until after the costmap has been updated at least once after a reset. This enables us to make sure we cannot ever create a path or control with a completely empty costmap, potentially leading to collisions, due to clearing the costmap and then immediately requesting an algorithm to run.

> 在银河系中，它仍将保持转动，但也会增加额外功能。当成本图因清除或其他导航恢复而重置时，它现在也被设置为`false`。这阻止了机器人在重置后至少更新一次成本图之前创建计划或控制努力。这使我们能够确保我们永远不能使用完全空白的成本图创建路径或控制，可能导致清除成本图并立即请求运行算法而发生碰撞。

# Standard time units in parameters


To follow the SI units outlined in REP-103 to the \"T\" nodes below were modified to use seconds consistently in every parameter. Under each node name you can see which parameters changed to seconds instead of using milliseconds.

> 對於下面的「T」節點，根據REP-103中所列的SI單位，已對參數進行了修改，以使每個參數都以秒為單位進行操作。在每個節點名稱下面，您可以看到哪些參數已改為以秒而不是毫秒為單位。

- lifecycle manager
  - `bond_timeout_ms` became `bond_timeout` in seconds
- smac planner
  - `max_planning_time_ms` became `max_planning_time` in seconds
- map saver
  - `save_map_timeout` in seconds

# Ray Tracing Parameters


Raytracing functionality was modified to include a minimum range parameter from which ray tracing starts to clear obstacles to avoid incorrectly clearing obstacles too close to the robot. This issue was mentioned in [ROS Answers](https://answers.ros.org/question/355150/obstacles-in-sensor-deadzone/). An existing parameter `raytrace_range` was renamed to `raytrace_max_range` to reflect the functionality it affects. The renamed parameters and the plugins that they belong to are mentioned below. The changes were introduced in this [pull request](https://github.com/ros-planning/navigation2/pull/2126).

> 功能Raytracing已修改以包括一个最小范围参数，从该范围开始进行光线追踪以清除障碍物，以避免不正确地清除太接近机器人的障碍物。 这个问题在[ROS Answers](https://answers.ros.org/question/355150/obstacles-in-sensor-deadzone/)中提到。 现有参数`raytrace_range`已重命名为`raytrace_max_range`，以反映其影响的功能。 下面列出了重命名的参数和它们所属的插件。 这些变化在[pull request](https://github.com/ros-planning/navigation2/pull/2126)中引入。

- obstacle_layer plugin

  - `raytrace_min_range` controls the minimum range from which ray tracing clears obstacles from the costmap

> - `raytrace_min_range`控制射线追踪从成本地图中清除障碍物的最小范围。

  - `raytrace_max_range` controls the maximum range to which ray tracing clears obstacles from the costmap

> - `raytrace_max_range`控制从成本地图中清除障碍物的最大范围。
- voxel_layer plugin

  - `raytrace_min_range` controls the minimum range from which ray tracing clears obstacles from the costmap

> - `raytrace_min_range`控制从何处开始射线追踪清除成本地图中的障碍物的最小范围

  - `raytrace_max_range` controls the maximum range to which ray tracing clears obstacles from the costmap

> - `raytrace_max_range` 控制了从成本地图中清除障碍物的最大距离。

# Obstacle Marking Parameters


Obstacle marking was modified to include a minimum range parameter from which obstacles are marked on the costmap to prevent addition of obstacles in the costmap due to noisy and incorrect measurements. This modification is related to the change with the raytracing parameters. The renamed parameters, newly added parameters and the plugins they belong to are given below.

> 修改障碍物标记，包括一个最小范围参数，以防止噪声和不正确的测量在成本图中添加障碍物。此修改与射线跟踪参数的更改有关。下面给出了重命名的参数、新添加的参数和它们所属的插件。

- obstacle_layer plugin
  - `obstacle_min_range` controls the minimum range from which obstacle are marked on the costmap
  - `obstacle_max_range` controls the maximum range to which obstacles are marked on the costmap
- voxel_layer plugin
  - `obstacle_min_range` controls the minimum range from which obstacle are marked on the costmap
  - `obstacle_max_range` controls the maximum range to which obstacles are marked on the costmap

# Recovery Action Changes


The recovery actions, `Spin` and `BackUp` were modified to correctly return `FAILURE` if the recovery action is aborted due to a potential collision. Previously, these actions incorrectly always returned `SUCCESS`. Changes to this resulted in downstream action clients, such as the default behavior tree. The changes were introduced in this [pull request 1855](https://github.com/ros-planning/navigation2/pull/1855).

> 恢复操作`Spin`和`BackUp`已被修改以在潜在冲突导致恢复操作中止时正确返回`FAILURE`。以前，这些操作总是错误地返回`SUCCESS`。这种更改导致了下游操作客户端，如默认行为树。这些更改是在[拉取请求1855](https://github.com/ros-planning/navigation2/pull/1855)中引入的。

# Default Behavior Tree Changes


The default behavior tree (BT) `navigate_w_replanning_and_recovery.xml` has been updated to allow for replanning in between recoveries. The changes were introduced in this [PR 1855](https://github.com/ros-planning/navigation2/pull/1855). Additionally, an alternative BT `navigate_w_replanning_and_round_robin_recovery.xml` was removed due to similarity with the updated default BT.

> 默认行为树（BT）`navigate_w_replanning_and_recovery.xml`已更新，以允许在恢复之间进行重新规划。 这些变化在[PR 1855]（https://github.com/ros-planning/navigation2/pull/1855）中引入。 此外，由于与更新的默认BT相似，因此已删除了另一个BT`navigate_w_replanning_and_round_robin_recovery.xml`。

# NavFn Planner Parameters


The NavFn Planner has now its 3 parameters reconfigurable at runtime (`tolerance`, `use_astar` and `allow_unknown`). The changes were introduced in this [pull request 2181](https://github.com/ros-planning/navigation2/pull/2181).

> NavFn计划现在可以在运行时重新配置3个参数（`容忍度`，`use_astar`和`allow_unknown`）。 这些变化是在[pull request 2181]（https://github.com/ros-planning/navigation2/pull/2181）中引入的。

# New ClearCostmapExceptRegion and ClearCostmapAroundRobot BT-nodes


The ClearEntireCostmap action node was already implemented but the ClearCostmapExceptRegion and ClearCostmapAroundRobot BT nodes calling the sister services `(local_or_global)_costmap/clear_except_(local_or_global)_costmap` and `clear_around_(local_or_global)_costmap` of Costmap 2D were missing, they are now implemented in a similar way. They both expose a `reset_distance` input port. See `bt_clear_costmap_except_region_action`{.interpreted-text role="ref"} and `bt_clear_entire_costmap_around_robot_action`{.interpreted-text role="ref"} for more. The changes were introduced in this [pull request 2204](https://github.com/ros-planning/navigation2/pull/2204).

> ClearEntireCostmap 动作节点已经实现，但是调用 Costmap 2D 的 `(local_or_global)_costmap/clear_except_(local_or_global)_costmap` 和 `clear_around_(local_or_global)_costmap` 服务的 ClearCostmapExceptRegion 和 ClearCostmapAroundRobot BT 节点缺失，现在以类似的方式实现了它们。它们都暴露了一个 `reset_distance` 输入端口。参见 `bt_clear_costmap_except_region_action`{.interpreted-text role="ref"} 和 `bt_clear_entire_costmap_around_robot_action`{.interpreted-text role="ref"} 了解更多。这些改变在 [pull request 2204](https://github.com/ros-planning/navigation2/pull/2204) 中引入了。

# New Behavior Tree Nodes


A new behavior tree node was added and dynamically loadable at run-time using behavior tree cpp v3. See `nav2_behavior_tree` for a full listing, or `plugins`{.interpreted-text role="ref"} for the current list of behavior tree plugins and their descriptions. These plugins are set as default in the `nav2_bt_navigator` but may be overridden by the `bt_plugins` parameter to include your specific plugins.

> 一个新的行为树节点已经添加并且可以使用行为树cpp v3在运行时动态加载。查看`nav2_behavior_tree`获取完整列表，或者查看`plugins`{.interpreted-text role="ref"}获取当前行为树插件及其描述的列表。这些插件被默认设置在`nav2_bt_navigator`中，但是可以通过`bt_plugins`参数来覆盖，以包括你的特定插件。


Original GitHub tickets:

> `原始GitHub票据：`

    - [SingleTrigger](https://github.com/ros-planning/navigation2/pull/2236)
    - [PlannerSelector](https://github.com/ros-planning/navigation2/pull/2249)
    - [ControllerSelector](https://github.com/ros-planning/navigation2/pull/2266)
    - [GoalCheckerSelector](https://github.com/ros-planning/navigation2/pull/2269)
    - [NavigateThroughPoses](https://github.com/ros-planning/navigation2/pull/2271)
    - [RemovePassedGoals](https://github.com/ros-planning/navigation2/pull/2271)
    - [ComputePathThroughPoses](https://github.com/ros-planning/navigation2/pull/2271)


Additionally, behavior tree nodes were modified to contain their own local executors to spin for actions, topics, services, etc to ensure that each behavior tree node is independent of each other (e.g. spinning in one BT node doesn\'t trigger a callback in another).

> 此外，行为树节点被修改为包含自己的本地执行器来执行动作、主题、服务等，以确保每个行为树节点彼此独立（例如，在一个BT节点中旋转不会触发另一个节点中的回调）。

# sensor_msgs/PointCloud to sensor_msgs/PointCloud2 Change


Due to deprecation of [sensor_msgs/PointCloud](https://docs.ros2.org/foxy/api/sensor_msgs/msg/PointCloud.html) the topics which were publishing sensor_msgs/PointCloud are converted to sensor_msgs/PointCloud2. The details on these topics and their respective information are listed below.

> 由于[sensor_msgs/PointCloud](https://docs.ros2.org/foxy/api/sensor_msgs/msg/PointCloud.html)的淘汰，发布sensor_msgs/PointCloud的主题已转换为sensor_msgs/PointCloud2。 下面列出了这些主题及其相关信息的详细信息。

- `clearing_endpoints` topic in `voxel_layer` plugin of `nav2_costmap_2d` package

- `voxel_marked_cloud` and `voxel_unknown_cloud` topic in `costmap_2d_cloud` node of `nav2_costmap_2d` package

> `voxel_marked_cloud` 和 `voxel_unknown_cloud` 主题在 `costmap_2d_cloud` 节点的 `nav2_costmap_2d` 包中
- `cost_cloud` topic of `publisher.cpp` of `dwb_core` package.


These changes were introduced in [pull request 2263](https://github.com/ros-planning/navigation2/pull/2263).

> 这些变化是在[拉取请求2263](https://github.com/ros-planning/navigation2/pull/2263)中引入的。

# ControllerServer New Parameter failure_tolerance


A new parameter `failure_tolerance` was added to the Controller Server for tolerating controller plugin exceptions without failing immediately. It is analogous to `controller_patience` in ROS(1) Nav. See `configuring_controller_server`{.interpreted-text role="ref"} for description. This change was introduced in this [pull request 2264](https://github.com/ros-planning/navigation2/pull/2264).

> 一个新的参数`failure_tolerance`被添加到控制器服务器以容忍控制插件异常而不立即失败。它类似于ROS（1）Nav中的`controller_patience`。有关说明，请参阅`configuring_controller_server`。此更改在此[拉取请求2264]（https://github.com/ros-planning/navigation2/pull/2264）中引入。

# Removed BT XML Launch Configurations


The launch python configurations for CLI setting of the behavior tree XML file has been removed. Instead, you should use the yaml files to set this value. If you, however, have a `path` to the yaml file that is inconsistent in a larger deployment, you can use the `RewrittenYaml` tool in your parent launch file to remap the default XML paths utilizing the `get_shared_package_path()` directory finder (or as you were before in python3).

> 在 CLI 設定行為樹 XML 檔案的 Python 設定已被移除。取而代之的是，您應該使用 yaml 檔案來設定此值。但是，如果您在較大的部署中有一個到 yaml 檔案的`路徑`不一致，您可以在父啟動檔案中使用`RewrittenYaml`工具來重新對應預設的 XML 路徑，使用`get_shared_package_path()`目錄查找器（或者像您之前在 python3 中一樣）。


The use of map subscription QoS launch configuration was also removed, use parameter file. This change was introduced in this [pull request 2295](https://github.com/ros-planning/navigation2/pull/2295).

> 使用映射订阅QoS启动配置已被移除，请使用参数文件。此更改已在此[拉取请求2295](https://github.com/ros-planning/navigation2/pull/2295)中引入。

# Nav2 RViz Panel Action Feedback Information


The Nav2 RViz Panel now displays the action feedback published by `nav2_msgs/NavigateToPose` and `nav2_msgs/NavigateThroughPoses` actions. Users can find information like the estimated time of arrival, distance remaining to goal, time elapsed since navigation started, and number of recoveries performed during a navigation action directly through the RViz panel. This feature was introduced in this [pull request 2338](https://github.com/ros-planning/navigation2/pull/2338).

> 现在，Nav2 RViz 面板显示由`nav2_msgs/NavigateToPose`和`nav2_msgs/NavigateThroughPoses`动作发布的操作反馈。用户可以通过RViz面板直接找到预计到达时间、到达目标的剩余距离、从导航开始的时间以及导航动作期间执行的恢复次数等信息。此功能在[pull request 2338]中引入（https://github.com/ros-planning/navigation2/pull/2338）。

![Navigation feedback in RViz.](/images/rviz/panel-feedback.gif){.align-center width="600px"}
