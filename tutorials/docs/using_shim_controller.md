---
tip: translate by openai@2023-06-02 13:48:06
title: Using Rotation Shim Controller
---

- [Overview](#overview)
- [What is the Rotation Shim Controller?](#what-is-the-rotation-shim-controller)
- [Configuring Rotation Shim Controller](#configuring-rotation-shim-controller)
- [Configuring Primary Controller](#configuring-primary-controller)
- [Demo Execution](#demo-execution)

# Overview

This tutorial will discuss how to set up your robot to use the `RotationShimController` to help create intuitive, rotate-in-place, behavior for your robot while starting out to track a path. The goal of this tutorial is to explain to the reader the value of the controller, how to configure it, how to configure the primary controller with it, and finally an example of it in use.

> 这个教程将讨论如何设置你的机器人来使用`RotationShimController`来帮助**创建直观的旋转原地行为**，以便开始跟踪路径。本教程的目的是向读者解释该控制器的价值，如何配置它，如何配置主控制器以及最后一个使用的示例。

Before starting this tutorial, completing the `getting_started` is highly recommended especially if you are new to ROS and Nav2. The requirements are having the latest install of Nav2 / ROS 2 containing this package.

> 在开始本教程之前，特别是对于 ROS 和 Nav2 新手来说，强烈建议完成“getting_started”。要求是拥有包含此软件包的 Nav2/ROS 2 的最新安装。

# What is the Rotation Shim Controller?

This was developed due to quirks in TEB and DWB, but applicable to any other controller plugin type that you\'d like to have rotation in place behavior with. `TEB`\'s behavior tends to whip the robot around with small turns, or when the path is starting at a very different heading than current, in a somewhat surprising way due to the elastic band approach. `DWB` can be tuned to have any type of behavior, but typically to tune it to be an excellent path follower also makes it less optimally capable of smooth transitions to new paths at far away headings \-- there are always trade offs. Giving both TEB and DWB a better starting point to start tracking a path makes tuning the controllers significantly easier and creates more intuitive results for on-lookers.

> 由于 TEB 和 DWB 的怪癖，开发了这个功能，但也适用于任何其他您想要旋转的控制器插件类型。TEB 的行为往往会用小转弯将机器人带走，或者当路径以与当前非常不同的航向开始时，由于弹性带方法，会以一种令人惊讶的方式进行。DWB 可以调整为任何类型的行为，但通常调整它以成为一个出色的路径跟随者也会使它在过渡到远处航向的新路径时不太理想 - 总是有权衡。为 TEB 和 DWB 提供一个更好的起点来跟踪路径，可以显着简化控制器的调整，并为旁观者创造更直观的结果。

Note that it is not required to use this with **any** plugin. Many users are perfectly successful without using this controller, but if a robot may rotate in place before beginning its path tracking task (or others), it can be advantageous to do so.

> 请注意，不需要与任何插件一起使用它。许多用户在不使用此控制器的情况下完全成功，但是如果在开始其路径跟踪任务(或其他)之前，机器人可能会旋转到位，则这样做可能会很有利。

The `nav2_rotation_shim_controller` will check the rough heading difference with respect to the robot and a newly received path. If within a threshold, it will pass the request onto the `primary_controller` to execute the task. If it is outside of the threshold, this controller will rotate the robot in place towards that path heading. Once it is within the tolerance, it will then pass off control-execution from this rotation shim controller onto the primary controller plugin. At this point, the robot\'s main plugin will take control for a smooth hand off into the task.

> `nav2_rotation_shim_controller`将检查与机器人和新接收路径之间的粗略航向差异。如果在阈值范围内，它将将请求传递给`primary_controller`来执行任务。如果超出阈值，此控制器将在原地旋转机器人朝向该路径航向。一旦它在公差范围内，它将从这个旋转拆卸控制器传递控制执行到主控制插件。在这一点上，机器人的主插件将控制平稳地转交到任务中。

The `RotationShimController` is most suitable for:

- Robots that can rotate in place, such as differential and omnidirectional robots.
- Preference to rotate in place when starting to track a new path that is at a significantly different heading than the robot\'s current heading \-- or when tuning your controller for its task makes tight rotations difficult.
- Using planners that are non-kinematically feasible, such as NavFn, Theta\*, or Smac 2D (Feasible planners such as Smac Hybrid-A\* and State Lattice will start search from the robot\'s actual starting heading, requiring no rotation since their paths are guaranteed drivable by physical constraints).

> - 可以旋转到位的机器人，例如差分和全向机器人。
> - 优先在开始跟踪一个与机器人当前航向显著不同的新路径时在原地旋转，或者当调整控制器以完成其任务时，旋转变得困难时。
> - 使用不可动力可行的规划器，如 NavFn、Theta\* 或 Smac 2D(可行的规划器，如 Smac Hybrid-A\*和 State Lattice，将从机器人实际的起始航向开始搜索，无需旋转，因为它们的路径受物理约束保证可行)。

::: note
::: title
Note
:::

Regulated Pure Pursuit has this built in so it is not necessary to pair with RPP. However, it is applicable to all others. See `plugins` for a full list of current controller plugins.

> 受管制的纯追求已经内置，因此不需要与 RPP 配对。但是，它适用于所有其他控制器。有关当前控制器插件的完整列表，请参见“插件”。

:::

# Configuring Rotation Shim Controller

This controller is a _shim_ because it is placed between the primary controller plugin and the controller server. It takes commands and pre-processes them to rotate to the heading and then passes off execution-control to the primary plugin once that condition is met - acting as a simple pass through.

> 这个控制器是一个 _shim_，因为它位于主控制插件和控制服务器之间。它接收命令并预处理它们以旋转到航向，然后一旦达到该条件，就将执行控制传递给主插件 - 作为一个简单的通过。

As such, its configuration looks very similar to that of any other plugin. In the code block below, you can see that we\'ve added the `RotationShimController` as the plugin for path tracking in the controller server. You can see that we\'ve also configured it below with its internal parameters, `angular_dist_threshold` through `max_angular_accel`.

> 因此，它的配置看起来与任何其他插件非常相似。在下面的代码块中，您可以看到我们已将`RotationShimController`添加为控制器服务器中的路径跟踪插件。您可以看到，我们还在下面配置了其内部参数`angular_dist_threshold`到`max_angular_accel`。

```yaml
controller_server:
  ros__parameters:
    use_sim_time: True
    controller_frequency: 20.0
    min_x_velocity_threshold: 0.001
    min_y_velocity_threshold: 0.5
    min_theta_velocity_threshold: 0.001
    progress_checker_plugins: ["progress_checker"] # progress_checker_plugin: "progress_checker" For Humble and older
    goal_checker_plugins: ["goal_checker"]
    controller_plugins: ["FollowPath"]
    progress_checker:
      plugin: "nav2_controller::SimpleProgressChecker"
      required_movement_radius: 0.5
      movement_time_allowance: 10.0
    goal_checker:
      plugin: "nav2_controller::SimpleGoalChecker"
      xy_goal_tolerance: 0.25
      yaw_goal_tolerance: 0.25
      stateful: True
    FollowPath:
      plugin: "nav2_rotation_shim_controller::RotationShimController"
      angular_dist_threshold: 0.785
      forward_sampling_distance: 0.5
      rotate_to_heading_angular_vel: 1.8
      max_angular_accel: 3.2
      simulate_ahead_time: 1.0
```

The Rotation Shim Controller is very simple and only has a couple of parameters to dictate the conditions it should be enacted.

> 旋转垫片控制器非常简单，只有几个参数来指定应该实施的条件。

- `angular_dist_threshold`: The angular distance (in radians) apart from the robot\'s current heading and the approximated path heading to trigger the rotation behavior. Once the robot is within this threshold, control is handed over to the primary controller plugin.
- `forward_sampling_distance`: The distance (in meters) away from the robot to select a point on the path to approximate the path\'s starting heading at. This is analogous to a \"lookahead\" point.
- `rotate_to_heading_angular_vel`: The angular velocity (in rad/s) to have the robot rotate to heading by, when the behavior is enacted.
- `max_angular_accel`: The angular acceleration (in rad/s/s) to have the robot rotate to heading by, when the behavior is enacted.
- `simulate_ahead_time`: The Time (s) to forward project the rotation command to check for collision

> - `angular_dist_threshold`: 与机器人当前航向和估计路径航向之间的角度距离(以弧度为单位)，以触发旋转行为。一旦机器人在此阈值范围内，控制权将交给主控制器插件。
> - `forward_sampling_distance`：机器人从路径上选取一个点来近似路径的起始航向所需的距离(以米为单位)。这类似于“前瞻”点。
> - `rotate_to_heading_angular_vel`：行为实施时，机器人旋转到航向所需的角速度(以弧度/秒为单位)。
> - `max_angular_accel`：行为执行时，机器人旋转到航向所需的角加速度(以 rad/s/s 为单位)。
> - `simulate_ahead_time`：检查碰撞的旋转命令的前进时间(秒)

# Configuring Primary Controller

There is one more remaining parameter of the `RotationShimController` not mentioned above, the `primary_controller`. This is the type of controller that your application would like to use as the primary modus operandi. It will share the same name and yaml namespace as the shim plugin. You can observe this below with the primary controller set the `DWB` (with the progress and goal checkers removed for brevity).

> 还有一个`RotationShimController`的参数没有提到，那就是`primary_controller`。这是应用程序想要使用的主要操作模式的类型。它将具有与 shim 插件相同的名称和 yaml 命名空间。您可以在下面看到，将 primary controller 设置为`DWB`(删除了**进度和目标检查器**，以简洁起见)。

```yaml
controller_server:
  ros__parameters:
    use_sim_time: True
    controller_frequency: 20.0
    min_x_velocity_threshold: 0.001
    min_y_velocity_threshold: 0.5
    min_theta_velocity_threshold: 0.001
    controller_plugins: ["FollowPath"]
    FollowPath:
      plugin: "nav2_rotation_shim_controller::RotationShimController"
      primary_controller: "dwb_core::DWBLocalPlanner"
      angular_dist_threshold: 0.785
      forward_sampling_distance: 0.5
      rotate_to_heading_angular_vel: 1.8
      max_angular_accel: 3.2
      simulate_ahead_time: 1.0

      # DWB parameters
      ...
      ...
      ...
```

An important note is that **within the same yaml namespace**, you may also include any `primary_controller` specific parameters required for a robot. Thusly, after `max_angular_accel`, you can include any of `DWB`\'s parameters for your platform.

> 注意：**在相同的 yaml 命名空间**中，您还可以包括机器人所需的任何`primary_controller`特定参数。 因此，在`max_angular_accel`之后，您可以包括`DWB`的任何参数以用于您的平台。

# Demo Execution

```html{=html}
<h1 align="center">
  <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
    <iframe width="708" height="400" src="https://www.youtube.com/embed/t-g2CBGByEw?autoplay=1&mute=1" frameborder="1" allowfullscreen></iframe>
  </div>
</h1>
```
