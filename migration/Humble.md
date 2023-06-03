---
tip: translate by openai@2023-06-03 00:29:11
title: Humble to Iron
---

Moving from ROS 2 Humble to Iron, a number of stability improvements were added that we will not specifically address here.

> 从 ROS 2 Humble 到 Iron 的迁移，增加了一些稳定性改进，我们不会在这里具体讨论。

# New Behavior-Tree Navigator Plugins

New in [PR 3345](https://github.com/ros-planning/navigation2/pull/3345), the navigator types are exposed to users as plugins that can be replaced or new navigator types added. The default behaviors of navigate to pose and navigate through poses continue to be default behavior but are now customizable with new action interface definitions. These plugins implement the `nav2_core::BehaviorTreeNavigator` base class, which must process the action request, feedback, and completion messages. The behavior tree is handled by this base class with as much general logic as possible abstracted away from users to minimize repetition.

> 在[PR 3345](https://github.com/ros-planning/navigation2/pull/3345)中，导航器类型被暴露给用户作为可替换或添加新导航器类型的插件。导航到姿态和导航通过姿态的默认行为仍然是默认行为，但现在可以通过新的动作接口定义进行定制。这些插件实现`nav2_core::BehaviorTreeNavigator`基类，必须处理动作请求，反馈和完成消息。该基类处理行为树，尽可能将一般逻辑抽象出来，以减少重复。

See `writing_new_nav2navigator_plugin` for a tutorial about writing new navigator plugins.

> 见`writing_new_nav2navigator_plugin`，以了解有关如何编写新的导航插件的教程。

# Added Collision Monitor

[PR 2982](https://github.com/ros-planning/navigation2/pull/2982) adds new safety layer operating independently of Nav2 stack which ensures the robot to control the collisions with near obstacles, obtained from different sensors (LaserScan, PointCloud, IR, Sonars, etc\...). See `configuring_collision_monitor` for more details. It is not included in the default bringup batteries included from `nav2_bringup`.

> [PR 2982](https://github.com/ros-planning/navigation2/pull/2982) 添加了一个独立于 Nav2 堆栈的新安全层，可以确保机器人控制来自不同传感器（激光扫描，点云，红外，声纳等）的附近障碍物的碰撞。有关详细信息，请参阅`configuring_collision_monitor`。它不包括在`nav2_bringup`中的默认 bringup 电池中。

# Removed use_sim_time from yaml

[PR #3131](https://github.com/ros-planning/navigation2/pull/3131) makes it possible to set the use_sim_time parameter from the launch file for multiple nodes instead of individually via the yaml files. If using the Nav2 launch files, you can optionally remove the use_sim_time parameter from your yaml files and set it via a launch argument.

> [PR #3131](https://github.com/ros-planning/navigation2/pull/3131) 可以从启动文件中为多个节点设置 use_sim_time 参数，而不需要通过 yaml 文件单独设置。如果使用 Nav2 启动文件，您可以选择从 yaml 文件中删除 use_sim_time 参数，并通过启动参数设置。

# Run-time Speed up of Smac Planner

The core data structure of the graph implementation in the Smac Planner framework was swapped out in [PR 3201](https://github.com/ros-planning/navigation2/pull/3201) to using a specialized unordered map implementation. This speeds up the planner by 10% on trivial requests and reports up to 30% on complex plans that involve numerous rehashings.

> 在[PR 3201](https://github.com/ros-planning/navigation2/pull/3201)中，Smac Planner 框架中图形实现的核心数据结构已替换为使用专用的无序映射实现。这可以使计划程序在简单请求上提速 10％，并在涉及大量重新散列的复杂计划上报告高达 30％的加速。

# Recursive Refinement of Smac and Simple Smoothers

The number of recursive refinements for the Simple and Smac Planner Smoothers have been exposed under the `refinement_num` parameter. Previous behavior had this hardcoded if `do_refinement = True` to `4`. Now, default is `2` to help decrease out-of-the-box over smoothing reducing in paths closer to collision than probably ideal, but old behavior can be achieved by changing this to `4`.

> 简单和 Smac 规划器平滑器的递归细化数量已经暴露在`refinement_num`参数下。如果`do_refinement = True`，以前的行为将其硬编码为`4`。现在，默认值为`2`，以帮助减少开箱即用的过度平滑，从而减少接近碰撞的路径，但是可以通过将其更改为`4`来实现旧行为。

# Simple Commander Python API

[PR 3159](https://github.com/ros-planning/navigation2/pull/3159) and follow-up PRs add in Costmap API in Python3 simple commander to receive `OccupancyGrid` messages from Nav2 and be able to work with them natively in Python3, analog to the C++ Costmap API. It also includes a line iterator and collision checking object to perform footprint or other collision checking in Python3. See the Simple Commander API for more details.

> [PR 3159](https://github.com/ros-planning/navigation2/pull/3159) 和后续的 PRs 添加了 Python3 中的 Costmap API，以便能够从 Nav2 中接收 `OccupancyGrid` 消息并在 Python3 中本地处理它们，类似于 C++ Costmap API。它还包括一个线迭代器和碰撞检查对象，以在 Python3 中执行足迹或其他碰撞检查。有关更多详细信息，请参阅 Simple Commander API。

# Smac Planner Start Pose Included in Path

[PR 3168](https://github.com/ros-planning/navigation2/pull/3168) adds the starting pose to the Smac Planners that was previously excluded during backtracing.

> `[PR 3168](https://github.com/ros-planning/navigation2/pull/3168)`添加了在回溯时以前被排除的起始姿势到 Smac 规划器中。

# Parameterizable Collision Checking in RPP

[PR 3204](https://github.com/ros-planning/navigation2/pull/3204) adds makes collision checking for RPP optional (default on).

> [PR 3204](https://github.com/ros-planning/navigation2/pull/3204) 添加了对RPP的碰撞检测可选（默认开启）。

# Expanded Planner Benchmark Tests

[PR 3218](https://github.com/ros-planning/navigation2/pull/3218) adds launch files and updated scripts for performing objective random planning tests across the planners in Nav2 for benchmarking and metric computation.

> `[PR 3218](https://github.com/ros-planning/navigation2/pull/3218) 添加了启动文件和更新的脚本，用于在Nav2中的规划器之间执行目标随机规划测试，用于基准测试和度量计算。`

# Smac Planner Path Tolerances

[PR 3219](https://github.com/ros-planning/navigation2/pull/3219) adds path tolerances to Hybrid-A\* and State Lattice planners to return approximate paths if exact paths cannot be found, within a configurable tolerance aroun the goal pose.

> [PR 3219](https://github.com/ros-planning/navigation2/pull/3219) 添加路径容差到 Hybrid-A\*和 State Lattice 规划器，以便在无法找到精确路径的情况下，在目标位置周围的可配置容差内返回近似路径。

# costmap_2d_node default constructor

[PR #3222](https://github.com/ros-planning/navigation2/pull/3222) changes the constructor used by the standalone costmap node. The new constructor does not set a name and namespace internally so it can be set via the launch file.

> [PR #3222](https://github.com/ros-planning/navigation2/pull/3222) 改变了独立成本地图节点使用的构造函数。新的构造函数不再在内部设置名称和命名空间，因此可以通过启动文件来设置。

# Feedback for Navigation Failures

[PR #3146](https://github.com/ros-planning/navigation2/pull/3146) updates the global planners to throw exceptions on planning failures. These exceptions get reported back to the planner server which in turn places a error code on the Behavior Tree Navigator\'s blackboard for use in contextual error handling in the autonomy application.

> [PR #3146](https://github.com/ros-planning/navigation2/pull/3146) 更新全局规划器以在规划失败时抛出异常。这些异常会被报告给规划服务器，然后规划服务器会将错误码放置在行为树导航器的黑板上，以供自主应用程序中的上下文错误处理使用。

The following errors codes are supported (with more to come as necessary): Unknown, TF Error, Start or Goal Outside of Map, Start or Goal Occupied, Timeout, or No Valid Path Found.

> 以下错误代码受支持（必要时会增加更多）：`未知`、TF 错误、起点或终点在地图之外、起点或终点被占用、`Wait` 超时或未找到有效路径。

[PR #3248](https://github.com/ros-planning/navigation2/pull/3248) updates the compute path through poses action to report planning failures. These exceptions get reported back to the planner server which in turn places a error code on the Behavior Tree Navigator\'s blackboard for use in contextual error handling in the autonomy application.

> [PR #3248]（https://github.com/ros-planning/navigation2/pull/3248）更新了通过姿势操作计算路径以报告规划失败。这些异常会被报告回计划服务器，然后在行为树导航器的黑板上放置一个错误代码，以用于自主应用程序中的上下文错误处理。

The following errors codes are supported (with more to come as necessary): Unknown, TF Error, Start or Goal Outside of Map, Start or Goal Occupied, Timeout, No Valid Path Found and No Waypoints given.

> 以下错误代码支持（必要时可添加更多）：`Unknown`，`TF Error`，`Start or Goal Outside of Map`，`Start or Goal Occupied`，`Timeout`，`No Valid Path Found` 和 `No Waypoints given`。

[PR #3227](https://github.com/ros-planning/navigation2/pull/3227) updates the controllers to throw exceptions on failures. These exceptions get reported back to the controller server which in turn places a error code on the Behavior Tree Navigatior\'s blackboard for use in contextual error handling in the autonomy application.

> [PR #3227](https://github.com/ros-planning/navigation2/pull/3227) 更新控制器在出现失败时抛出异常。这些异常会被报告给控制服务器，然后控制服务器会在行为树导航器的黑板上设置一个错误码，以便在自主应用程序中进行上下文错误处理。

The following error codes are supported (with more to come as necessary): Unknown, TF Error, Invalid Path, Patience Exceeded, Failed To Make Progress, or No Valid Control.

> 以下错误码已支持（必要时将添加更多）：`Unknown`、TF 错误、`Invalid Path`、耐心超限、无法取得进展或无有效控制。

[PR #3251](https://github.com/ros-planning/navigation2/pull/3251) pipes the highest priority error code through the bt_navigator and defines the error code structure.

> `[PR #3251](https://github.com/ros-planning/navigation2/pull/3251)` 将最高优先级的错误代码管道传递到 bt_navigator 并定义错误代码结构。

A new parameter for the the BT Navigator called \"error_code_id_names\" was added to the nav2_params.yaml to define the error codes to compare. The lowest error in the \"error_code_id_names\" is then returned in the action request (navigate to pose, navigate through poses waypoint follower), whereas the code enums increase the higher up in the software stack - giving higher priority to lower-level failures.

> 一个叫做“error_code_id_names”的新参数已经添加到 nav2_params.yaml 中，用于定义要比较的错误代码。在“error_code_id_names”中最低的错误将会在行动请求（导航到位置，通过位置导航航点跟随者）中返回，而代码枚举会随着软件栈的上升而增加——给较低级别的失败提供更高的优先级。

The error codes produced from the servers follow the guidelines stated below. Error codes from 0 to 9999 are reserved for nav2 while error codes from 10000-65535 are reserved for external servers. Each server has two \"reserved\" error codes. 0 is reserved for NONE and the first error code in the sequence is reserved for UNKNOWN.

> 服务器产生的错误码遵循下面所述的指南。从 0 到 9999 的错误码保留给 nav2，而从 10000 到 65535 的错误码保留给外部服务器。每个服务器有两个“保留”的错误码。 0 保留给 NONE，序列中的第一个错误码保留给 UNKNOWN。

The current implemented servers with error codes are:

> 当前实施的服务器及其错误代码为：

- Controller Server: NONE:0, UNKNOWN:100, server error codes: 101-199
- Planner Server(compute_path_to_pose): NONE:0, UNKNOWN:201, server error codes: 201-299
- Planner Server(compute_path_through_poses): NONE:0, UNKNOWN:301, server error codes: 301-399
- Smoother Server: NONE: 0, UNKNOWN: 501, server error codes: 501-599
- Waypoint Follower Server: NONE: 0, UNKNOWN: 601, server error codes: 601-699

This pr also updates the waypoint follower server to throw exceptions on failures. These exceptions get reported back to the server which in turn places a error code on the Behavior Tree Navigator\'s blackboard for use in contextual error handling in the autonomy application. The following errors codes are supported (with more to come as necessary): Unknown and Task Executor Failed. See `adding_a_nav2_task_server` and the PR for additional information.

> 这个 PR 也更新了路径跟踪服务器，以在失败时抛出异常。这些异常会被报告回服务器，然后服务器会在行为树导航器的黑板上放置一个错误码，以用于自主应用程序中的上下文错误处理。支持以下错误代码（随着需要会有更多）：未知和任务执行器失败。有关更多信息，请参见`adding_a_nav2_task_server`和 PR。

# Costmap Filters

Costmap Filters now are have an ability to be enabled/disabled in run-time by calling `toggle_filter` service for appropriate filter ([PR #3229](https://github.com/ros-planning/navigation2/pull/3229)).

> 现在，Costmap 过滤器可以通过调用`toggle_filter`服务来实时启用/禁用（[PR #3229](https://github.com/ros-planning/navigation2/pull/3229)）。

Added new binary flip filter, allowing e.g. to turn off camera in sensitive areas, turn on headlights/leds/other safety things or switch operating mode when robot is inside marked on mask areas ([PR #3228](https://github.com/ros-planning/navigation2/pull/3228)).

> 新增二元翻转滤波器，可以在敏感区域关闭摄像头，打开大灯/ LED /其他安全装置，或在标记遮罩区域内切换工作模式（[PR＃3228]（https://github.com/ros-planning/navigation2/pull/3228））。

# Savitzky-Golay Smoother

Adding a new smoother algorithm, the Savitzky-Golay smoother to the smoother server plugin list. See the configuration guide `configuring_savitzky_golay_filter_smoother` for more details.

> 加入一种新的平滑算法，Savitzky-Golay 平滑器，到平滑服务插件列表中。更多详情请参见`configuring_savitzky_golay_filter_smoother` 配置指南。

# Changes to Map yaml file path for map_server node in Launch

[PR #3174](https://github.com/ros-planning/navigation2/pull/3174) adds a way to set the path to map yaml file for the map_server node either from the yaml file or using the launch configuration parameter `map` giving priority to the launch configuration parameter. `yaml_filename` is no longer strictly required to be present in `nav2_params.yaml`.

> [PR #3174](https://github.com/ros-planning/navigation2/pull/3174) 增加了一种方法，可以通过 yaml 文件或使用启动配置参数`map`来设置 map_server 节点的 map yaml 文件路径，优先级更高。`yaml_filename`不再严格要求存在于`nav2_params.yaml`中。

# SmootherSelector BT Node

[PR #3283](https://github.com/ros-planning/navigation2/pull/3283) adds a BT node to set the smoother based on a topic or a default. See the configuration guide `configuring_simple_smoother` for more details.

> [PR #3283](https://github.com/ros-planning/navigation2/pull/3283) 添加了一个 BT 节点，用来根据主题或默认值设置更平滑的参数。有关更多细节，请参阅配置指南`configuring_simple_smoother`。

# Publish Costmap Layers

[PR #3320](https://github.com/ros-planning/navigation2/pull/3320) adds the ability for the nav2_costmap_2d package to publish out costmap data associated with each layer.

> [PR #3320](https://github.com/ros-planning/navigation2/pull/3320) 添加了 nav2_costmap_2d 包可以发布与每个图层相关的成本地图数据的能力。

# Give Behavior Server Access to Both Costmaps

[PR #3255](https://github.com/ros-planning/navigation2/pull/3255) addes the ability for a behavior to access the local and global costmap.

> [PR #3255](https://github.com/ros-planning/navigation2/pull/3255) 增加了行为访问局部和全局 costmap 的能力。

To update behaviors, any reference to the global_frame must be updated to the local_frame parameter along with the `configuration` method which now takes in the local and global collision checkers. Lastly, `getResourceInfo` must be overriden to return `CostmapInfoType::LOCAL`. Other options include `GLOBAL` if the behavior useses global costmap and/or footprint) or `BOTH` if both are required. This allows us to only create and maintain the minimum amount of expensive resources.

> 更新行为，任何涉及到 global_frame 的引用都需要更新为 local_frame 参数，同时`configuration`方法现在需要接受本地和全局碰撞检测器。最后，`getResourceInfo`必须被重写以返回`CostmapInfoType::LOCAL`。其他选项包括`GLOBAL`（如果行为使用全局 costmap 和/或足迹）或`BOTH`（如果需要两者）。这样可以最小化创建和维护昂贵的资源。

# New Model Predictive Path Integral Controller

The new Nav2 MPPI Controller is a predictive controller - a successor to TEB and pure path tracking MPC controllers - with Nav2. It uses a sampling based approach to select optimal trajectories, optimizing between successive iterations. It contains plugin-based objective functions for customization and extension for various behaviors and behavioral attributes.

> 新的 Nav2 MPPI 控制器是一种预测控制器 - 是 TEB 和纯路径跟踪 MPC 控制器的后继者 - 使用 Nav2。它采用基于采样的方法来选择最优轨迹，在连续迭代之间进行优化。它包含基于插件的目标函数，用于定制和扩展各种行为和行为属性。

See the README.md and `configuring_mppic` page for more detail.

> 见 README.md 和`configuring_mppic`页面以获取更多细节。

# Behavior Tree Uses Error Codes

[PR #3324](https:https://github.com/ros-planning/navigation2/pull/3324) adds three new condition nodes to check for error codes on the blackboard set by action BT nodes which contain them.

> [PR #3324](https:https://github.com/ros-planning/navigation2/pull/3324) 添加了三个新的条件节点，用于检查黑板上由包含它们的动作 BT 节点设置的错误代码。

The `AreErrorCodesPresent` condition node allows the user to specify the error code from the server along with the error codes to match against. The `WouldAControllerRecoveryHelp` checks if the active error code is UNKNOWN, PATIENCE_EXCEEDED, FAILED_TO_MAKE_PROGRESS or NO_VALID_CONTROL. If the error code is a match, the condition returns `SUCCESS`. These error code are potentially able to be cleared by a controller recovery.

> `AreErrorCodesPresent` 条件节点允许用户指定服务器的错误码以及要匹配的错误码。`WouldAControllerRecoveryHelp` 检查活动错误码是否为 UNKNOWN、PATIENCE_EXCEEDED、FAILED_TO_MAKE_PROGRESS 或 NO_VALID_CONTROL。如果错误码匹配，该条件将返回 `SUCCESS`。这些错误码可能可以通过控制器恢复清除。

The `WouldAPlannerRecoveryHelp` checks if the active error code is UNKNOWN, NO_VALID_CONTROL, or TIMEOUT. If the error code is a match, the condition returns `SUCCESS`. These error code are potentially able to be cleared by a planner recovery.

> `WouldAPlannerRecoveryHelp`检查活动错误代码是否为 UNKNOWN、NO_VALID_CONTROL 或 TIMEOUT。如果错误代码匹配，条件将返回`SUCCESS`。这些错误代码可能能够通过规划器恢复清除。

The `WouldASmootherRecoveryHelp` checks if the active error code is UNKNOWN, TIMEOUT, FAILED_TO_SMOOTH_PATH, or SMOOTHED_PATH_IN_COLLISION. If the error code is a match, the condition returns `SUCCESS`. These error code are potentially able to be cleared by a smoother recovery.

> `WouldASmootherRecoveryHelp`检查活动错误代码是否为 UNKNOWN、TIMEOUT、FAILED_TO_SMOOTH_PATH 或 SMOOTHED_PATH_IN_COLLISION。如果错误代码匹配，则条件返回`SUCCESS`。这些错误代码可能能够通过更平滑的恢复来清除。

# Load, Save and Loop Waypoints from the Nav2 Panel in RViz

[PR #3165](https:https://github.com/ros-planning/navigation2/pull/3165) provides three new functionalities for the nav2 panel in RViz, they are:

> [PR #3165](https:https://github.com/ros-planning/navigation2/pull/3165) 提供了三项新功能，可以在 RViz 的 nav2 面板中使用，它们是：`Wait`。

- load and save waypoints in a yaml file for waypoint following (initial pose can also be stored if required)

> 载入和保存 `waypoints` 到 `yaml` 文件以便进行路径跟踪（如果需要，也可以保存初始位置）

- loop functionality to revisit the waypoints
- pause and resume button for stopping and continuing through the waypoints

Looping functionality is not specific to the nav2 panel in RViz. Users utilizing nav2_waypoint_follower can take advantage of the changes made to the FollowWaypoint action, by specifying the desired number of loops in the action request that will be eventually sent to the nav2_waypoint_follower server.

> 循环功能不仅限于 RViz 中的 nav2 面板。使用 nav2_waypoint_follower 的用户可以利用 FollowWaypoint 操作所做的更改，通过在最终发送到 nav2_waypoint_follower 服务器的操作请求中指定所需的循环次数来实现。

# DWB Forward vs Reverse Pruning

[PR #3374](https://github.com/ros-planning/navigation2/pull/3374) adds a new `forward_prune_distance` parameter in the DWB controller. It replaces the `prune_distance` for forward path shortening, enabled through the `shorten_transformed_plan` boolean parameter. This change allows to use different values for forward and backward path shortening.

> [PR #3374](https://github.com/ros-planning/navigation2/pull/3374) 添加了一个新的`forward_prune_distance`参数到 DWB 控制器中。它用于替换`prune_distance`，用于通过`shorten_transformed_plan`布尔参数启用前向路径缩短。此更改允许使用不同的值来缩短前向和后向路径。

# More stable regulation on curves for long lookahead distances

[PR #3414](https://github.com/ros-planning/navigation2/pull/3414) adds a new `use_fixed_curvature_lookahead` parameter to the RPP controller. This makes slowing down on curve not dependent on the instantaneous lookahead point, but instead on a fixed distance set by the parameter `curvature_lookahead_dist`.

> [PR #3414](https://github.com/ros-planning/navigation2/pull/3414) 添加了一个新的`use_fixed_curvature_lookahead`参数到 RPP 控制器中。 这使得在曲线上减速不再依赖于瞬时的前瞻点，而是依赖于由参数`curvature_lookahead_dist`设置的固定距离。

# Publish Collision Monitor State

[PR #3504](https://github.com/ros-planning/navigation2/pull/3504) adds a new `state_topic` parameter to the CollisionMonitor. If specified, this optional parameter enables the state topic publisher. The topic reports the currently activated polygon action type and name.

> [PR #3504](https://github.com/ros-planning/navigation2/pull/3504) 增加了一个新的`state_topic`参数到 CollisionMonitor。如果指定，这个可选参数可以启用状态主题发布者。主题报告当前激活的多边形操作类型和名称。

# Renamed ROS-parameter in Collision Monitor

[PR #3513](https://github.com/ros-planning/navigation2/pull/3513) renames `max_points` parameter to `min_points` and changes its meaning. Formerly `max_points` meant the maximum number of points inside the area still not triggering the action, while `min_points` - is a minimal number of points starting from the action to be initiated. In other words `min_points` now should be adjusted as `max_points + 1`.

> [PR #3513](https://github.com/ros-planning/navigation2/pull/3513) 将`max_points`参数重命名为`min_points`，并改变其含义。以前，`max_points`意味着仍未触发操作的区域内的最大点数，而`min_points`则是从操作开始的最小点数。换句话说，现在`min_points`应该调整为`max_points + 1`。

# New safety behavior model \"limit\" in Collision Monitor

[PR #3519](https://github.com/ros-planning/navigation2/pull/3519) adds a new collision monitor behavior model `limit` that restricts maximum linear and angular speed to specific values (`linear_limit` and `angular_limit`) if enough points are in the given shape.

> [PR #3519](https://github.com/ros-planning/navigation2/pull/3519) 添加了一个新的碰撞监视行为模型`limit`，如果给定形状中有足够的点，它会限制最大的线性和角度速度到特定的值（`linear_limit`和`angular_limit`）。

# Velocity smoother applies deceleration when timeout

[PR #3512](https://github.com/ros-planning/navigation2/pull/3512) makes the VelocitySmoother apply the deceleration when the input command timeout.

> [PR #3512](https://github.com/ros-planning/navigation2/pull/3512) 使得 VelocitySmoother 在输入命令超时时应用减速。

# PoseProgressChecker plugin

[PR #3530](https://github.com/ros-planning/navigation2/pull/3530) adds a new `nav2_controller::PoseProgressChecker` plugin. It builds on the behavior of the `SimpleProgressChecker` by adding a new parameter `required_movement_angle`, allowing the plugin to considers that there is still progress when there is no translation movement, from the moment there is a rotation movement superior to `required_movement_angle` within the `movement_time_allowance`.

> [PR #3530](https://github.com/ros-planning/navigation2/pull/3530) 添加了一个新的`nav2_controller::PoseProgressChecker`插件。它建立在`SimpleProgressChecker`的行为之上，通过添加一个新的参数`required_movement_angle`，允许插件在没有平移运动的情况下仍然认为有进展，从`movement_time_allowance`内的旋转运动超过`required_movement_angle`的时刻开始。

# Allow multiple goal checkers and change parameter progress_checker_plugin(s) name and type

[PR #3555](https://github.com/ros-planning/navigation2/pull/3555) initializes the progress checker plugin(s) in the same way as for the goal checker and controller plugins: it is now a list of string and was renamed from `progress_checker_plugin` to `progress_checker_plugins`, and the type changed from `string` to `vector<string>`. This allows the initialization of multiple progress checkers that can be chosen from the added `progress_checker_id field` of the `FollowPath` action. Beware that it is a breaking change and that configuration files will need to be updated.

> [PR #3555](https://github.com/ros-planning/navigation2/pull/3555) 将进度检查插件以与目标检查器和控制器插件相同的方式进行初始化：现在它是一个字符串列表，并且从`progress_checker_plugin`重命名为`progress_checker_plugins`，类型从`string`更改为`vector<string>`。这允许从添加的`FollowPath`动作的`progress_checker_id字段`中选择多个进度检查器进行初始化。请注意，这是一个突破性的变化，配置文件需要更新。

# IsBatteryChargingCondition BT Node

[PR #3553](https://github.com/ros-planning/navigation2/pull/3553) adds a BT node to check if the battery is charging. See the configuration guide `bt_is_battery_charging_condition` for more details.

> [PR #3553](https://github.com/ros-planning/navigation2/pull/3553) 增加了一个 BT 节点来检查电池是否充电。有关更多详细信息，请参见配置指南`bt_is_battery_charging_condition`。

# Behavior Server Error Codes

[PR #3569](https://github.com/ros-planning/navigation2/pull/3539) updates the behavior server plugins to provide error codes on failure.

> [PR #3569](https://github.com/ros-planning/navigation2/pull/3539) 更新行为服务器插件以在失败时提供错误代码。

- Spin: NONE: 0, UNKNOWN: 701, server error codes: 701-709
- BackUp: NONE: 0, UNKNOWN: 801, server error codes: 710-719
- DriveOnHeading: NONE: 0, UNKNOWN: 901, server error codes: 720-729
- AssistedTeleop: NONE: 0, UNKNOWN: 1001, server error codes: 730-739

# New Denoise Costmap Layer Plugin

[PR #2567](https://github.com/ros-planning/navigation2/pull/2567) adds the new plugin for filtering noise on the costmap.

> [PR #2567](https://github.com/ros-planning/navigation2/pull/2567)添加了新的插件来过滤成本地图上的噪声。

Due to errors in `Voxel Layer` or `Obstacle Layer` measurements, salt and pepper noise may appear on the `costmap <configuring_cosmaps>`. This noise creates false obstacles that prevent the robot from finding the best path on the map. The new `Denoise Layer` plugin is designed to filter out noise-induced standalone obstacles or small obstacles groups. This plugin allows you to add layer that will filter local or global costmap. More information about `Denoise Layer` plugin and how it works could be found `here <filtering_of_noise-induced_obstacles>`.

> 由于`Voxel Layer`或`Obstacle Layer`测量中存在错误，`costmap <configuring_cosmaps>`上可能出现椒盐噪声。这种噪声会产生虚假的障碍物，阻碍机器人在地图上找到最佳路径。新的`Denoise Layer`插件旨在过滤出噪声产生的独立障碍物或小障碍物组。该插件允许您添加将过滤本地或全局 costmap 的图层。有关`Denoise Layer`插件及其工作原理的更多信息可以在`这里 <filtering_of_noise-induced_obstacles>`找到。

# SmacPlannerHybrid viz_expansions parameter

[PR #3577](https://github.com/ros-planning/navigation2/pull/3577) adds a new paremeter for visualising SmacPlannerHybrid expansions for debug purpose.

> [PR #3577](https://github.com/ros-planning/navigation2/pull/3577) 添加了一个新的参数，用于调试目的，用于可视化 SmacPlannerHybrid 扩展。
