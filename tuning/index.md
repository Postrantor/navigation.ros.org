---
tip: translate by openai@2023-06-03 00:42:44
...
---
title: Tuning Guide
---


This guide is meant to assist users in tuning their navigation system. While `configuration`{.interpreted-text role="ref"} is the home of the list of parameters for all of Nav2, it doesn\'t contain much _color_ for how to tune a system using the most important of them. The aim of this guide is to give more advice in how to setup your system beyond a first time setup, which you can find at `setup_guides`{.interpreted-text role="ref"}. This will by no means cover all of the parameters (so please, do review the configuration guides for the packages of interest), but will give some helpful hints and tips.

> 这份指南旨在帮助用户调整他们的导航系统。虽然`configuration`{.interpreted-text role="ref"}是Nav2的参数列表的主页，但它没有太多的_color_来指导如何使用其中最重要的参数来调整系统。本指南的目的是在第一次设置后（可以在`setup_guides`{.interpreted-text role="ref"}中找到），给出更多的建议来设置您的系统。这当然不会覆盖所有的参数（因此请务必审查感兴趣的软件包的配置指南），但会给出一些有用的提示和技巧。


This tuning guide is a perpetual work in progress. If you see some insights you have missing, please feel free to file a ticket or pull request with the wisdom you would like to share. This is an open section supported by the generosity of Nav2 users and maintainers. Please consider paying it forward.

> 这个调整指南是一个永久的工作在进行中。如果你看到一些你缺少的见解，请随时提交一个工单或拉取请求，分享你的智慧。这是一个由Nav2用户和维护者慷慨支持的开放部分。请考虑回馈。`

# Inflation Potential Fields


Many users and ecosystem navigation configuration files the maintainers find are really missing the point of the inflation layer. While it\'s true that you can simply inflate a small radius around the walls to weight against critical collisions, the _true_ value of the inflation layer is creating a consistent potential field around the entire map.

> 许多维护者发现的用户和生态系统导航配置文件真的没有抓住膨胀层的要点。虽然可以简单地在墙壁周围膨胀一个小半径以防止关键碰撞，但`真正`的膨胀层的价值在于在整个地图周围创建一个一致的潜在场。


Some of the most popular tuning guides for ROS Navigation / Nav2 even [call this out specifically](https://arxiv.org/pdf/1706.09068.pdf) that there\'s substantial benefit to creating a gentle potential field across the width of the map - after inscribed costs are applied - yet very few users do this in practice.

> 一些最流行的ROS导航/Nav2调整指南甚至[特别指出](https://arxiv.org/pdf/1706.09068.pdf)，在地图宽度上创建一个柔和的势场（在应用了描述成本之后）会有相当大的好处，但是实际上很少有用户这样做。


This habit actually results in paths produced by NavFn, Theta\*, and Smac Planner to be somewhat suboptimal. They really want to look for a smooth potential field rather than wide open 0-cost spaces in order to stay in the middle of spaces and deal with close-by moving obstacles better. It will allow search to be weighted towards freespace far before the search algorithm runs into the obstacle that the inflation is caused by, letting the planner give obstacles as wide of a berth as possible.

> 这种习惯实际上导致NavFn、Theta*和Smac Planner生成的路径有些不够优化。他们真的想要寻找一个平滑的势场，而不是宽阔的0成本空间，以便在空间中央保持，并更好地处理附近移动的障碍物。这将允许搜索在搜索算法遇到造成膨胀的障碍之前就向空间重量，让规划者尽可能地远离障碍。


So it is the maintainers\' recommendation, as well as all other cost-aware search planners available in ROS, to increase your inflation layer cost scale and radius in order to adequately produce a smooth potential across the entire map. For very large open spaces, its fine to have 0-cost areas in the middle, but for halls, aisles, and similar; **please create a smooth potential to provide the best performance**.

> 在ROS中，维护者建议，以及所有其他成本意识的搜索规划器，都应该增加您的通货膨胀层成本规模和半径，以便在整个地图上有效地产生平滑的潜力。对于非常大的开放空间，在中间有0成本的区域是可以的，但是对于大厅、过道等，**请创建一个平滑的潜力以提供最佳性能**。

# Robot Footprint vs Radius


Nav2 allows users to specify the robot\'s shape in 2 ways: a geometric `footprint` or the radius of a circle encompassing the robot. In ROS (1), it was pretty reasonable to always specify a radius approximation of the robot, since the global planning algorithms didn\'t use it and the local planners / costmaps were set up with the circular assumption baked in.

> Nav2允许用户以两种方式指定机器人的形状：几何`足迹`或围绕机器人的圆的半径。在ROS（1）中，总是指定机器人的半径近似值是相当合理的，因为全局规划算法不使用它，而本地规划者/成本图都是基于圆的假设进行设置的。


However, in Nav2, we now have multiple planning and controller algorithms that make use of the full SE2 footprint. If your robot is non-circular, it is recommended that you give the planners and controllers the actual, geometric footprint of your robot. This will allow the planners and controllers to plan or create trajectories into tighter spaces. For example, if you have a very long but skinny robot, the circular assumption wouldn\'t allow a robot to plan into a space only a little wider than your robot, since the robot would not fit length-wise.

> 然而，在Nav2中，我们现在有多种规划和控制算法，可以利用完整的SE2足迹。如果您的机器人不是圆形的，建议您将规划者和控制器提供实际的几何足迹。这将允许规划者和控制器计划或创建进入更紧凑空间的轨迹。例如，如果您有一个非常长而细长的机器人，圆形假设不允许机器人计划进入只比机器人稍宽的空间，因为机器人不适合长度。


The kinematically feasible planners (e.g. Smac Hybrid-A\*, Smac State Lattice) will use the SE2 footprint for collision checking to create kinematically feasible plans, if provided with the actual footprint. As of December, 2021 all of the controller plugins support full footprint collision checking to ensure safe path tracking. If you provide a footprint of your robot, it will be used to make sure trajectories are valid and it is recommended you do so. It will prevent a number of \"stuck robot\" situations that could have been easily avoided.

> 如果提供了实际的足迹，运动可行的规划器（例如Smac Hybrid-A *，Smac状态格）将使用SE2足迹进行碰撞检查以创建运动可行的计划。截至2021年12月，所有控制器插件都支持完整的足迹碰撞检查以确保安全的路径跟踪。如果您提供机器人的足迹，它将用于确保轨迹有效，建议您这样做。它将防止许多“卡住的机器人”情况，这些情况可以很容易地避免。


If your robot is truly circular, continue to use the `robot_radius` parameter. The three valid reasons for a non-circular robot to use the radius instead:

> 如果你的机器人真的是圆形的，继续使用`robot_radius`参数。 三个有效的原因使非圆形机器人使用半径而不是：`简化`

- The robot is very small relative to the environment (e.g. RC car in a warehouse)

- The robot has such limited compute power, using SE2 footprint checking would add too much computational burden (e.g. embedded micro-processors)

> 机器人的计算能力有限，使用SE2足迹检查会增加太多的计算负担（例如嵌入式微处理器）。

- If you plan to use a holonomic planner (e.g. Theta\*, Smac 2D-A\*, or NavFn), you may continue to use the circular footprint, since these planners are not kinematically feasible and will not make use of the SE2 footprint anyway.

> 如果您计划使用完全控制型规划器（例如Theta\*，Smac 2D-A\*或NavFn），您可以继续使用圆形足迹，因为这些规划器不具备运动学可行性，并且无论如何也不会使用SE2足迹。

# Rotate in Place Behavior


Using the `configuring_rotation_shim`{.interpreted-text role="ref"}, a robot will simply rotate in place before starting to track a holonomic path. This allows a developer to tune a controller plugin to be optimized for path tracking and give you clean rotations, out of the box.

> 使用`configuring_rotation_shim`{.interpreted-text role="ref"}, 机器人在开始跟踪洛蒙路径之前只需要在原地旋转。这样可以让开发者调整控制插件，使其能够优化路径跟踪，并且可以一次性获得干净的旋转。


This was added due to quirks in some existing controllers whereas tuning the controller for a task can make it rigid \-- or the algorithm simply doesn\'t rotate in place when working with holonomic paths (if that\'s a desirable trait). The result is an awkward, stuttering, or whipping around behavior when your robot\'s initial and path heading\'s are significantly divergent. Giving a controller a better starting point to start tracking a path makes tuning the controllers significantly easier and creates more intuitive results for on-lookers (in one maintainer\'s opinion).

> 这是由于一些现有控制器中存在的怪癖而添加的，即调整控制器以完成任务可能使其变得僵硬--或者说，当使用全向路径时，算法不会在原地旋转（如果这是一个可取的特性）。结果是，当机器人的初始和路径航向明显不同时，会出现笨拙、颤抖或绕行的行为。为跟踪路径提供一个更好的起点可以显着简化控制器的调整，并为旁观者创造更直观的结果（在一个维护者的意见中）。

Note: If using a non-holonomic, kinematically feasible planner (e.g. Smac Hybrid-A\*, Smac State Lattice), this is not a necessary behavioral optimization. This class of planner will create plans that take into account the robot\'s starting heading, not requiring any rotation behaviors.


This behavior is most optimially for:

> 这种行为最适合：

- Robots that can rotate in place, such as differential and omnidirectional robots.

- Preference to rotate in place when starting to track a new path that is at a significantly different heading than the robot's current heading -- or when tuning your controller for its task makes tight rotations difficult.

> 当开始跟踪一个与机器人当前航向显著不同的新路径时，或者当调整控制器以完成其任务时，优先考虑在原地旋转，以避免旋转过紧。

# Planner Plugin Selection


Nav2 provides a number of planning plugins out of the box. For a first-time setup, see `select_algorithm`{.interpreted-text role="ref"} for a more verbose breakdown of algorithm styles within Nav2, and `plugins`{.interpreted-text role="ref"} for a full accounting of the current list of plugins available (which may be updated over time).

> Nav2默认提供了一些规划插件。如果是第一次设置，请参考`select_algorithm`{.interpreted-text role="ref"}，以获取Nav2中算法类型的详细分解，以及`plugins`{.interpreted-text role="ref"}，以获取当前可用的插件列表（随时可能更新）。


In general though, the following table is a good guide for the optimal planning plugin for different types of robot bases:

> 一般而言，下表是不同类型机器人底座的最佳规划插件的良好指南：

```
    -----------------------------------------------------------------------
    Plugin Name        Supported Robot Types
    ------------------ ----------------------------------------------------
    NavFn Planner      Circular Differential, Circular Omnidirectional

    -----------------------------------------------------------------------


  +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| \| Smac Planner 2D \| \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| \| Theta Star Planner \| \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| Smac Hybrid-A\* Planner \| Non-circular or Circular Ackermann, Non-circular or Circular Legged \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| Smac Lattice Planner \| Non-circular Differential, Non-circular Omnidirectional, Arbitrary \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+

> +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| \| Smac Planner 2D \| \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| \| Theta Star Planner \| \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| Smac Hybrid-A\* Planner \| 非圆形或圆形Ackermann，非圆形或圆形腿 \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| Smac Lattice Planner \| 非圆形差动，非圆形全向，任意 \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+

Smac Planner 2D：Theta Star Planner
Smac Hybrid-A* Planner：非圆形或圆形Ackermann，非圆形或圆形腿
Smac Lattice Planner：非圆形差动，非圆形全向，任意
```


If you are using a non-circular robot with very limited compute, it may be worth assessing the benefits of using one of the holonomic planners (e.g. particle assumption planners). It is the recommendation of the maintainers to start using one of the more advanced algorithms appropriate for your platform _first_, but to scale back the planner if need be. The run-time of the feasible planners are typically on par (or sometimes faster) than their holonomic counterparts, so don\'t let the more recent nature of them fool you.

> 如果您使用的是一种非圆形机器人，具有非常有限的计算能力，建议您评估使用完整性规划器（例如，粒子假设规划器）的好处。维护者建议您首先使用适合您的平台的更高级算法，但如果需要，可以缩减规划器。可行规划器的运行时间通常与其完整性对应物相当（有时更快），因此不要让它们更新的性质愚弄您。


Since the planning problem is primarily driven by the robot type, the table accurately summarizes the advice to users by the maintainers. Within the circular robot regime, the choice of planning algorithm is dependent on application and desirable behavior. NavFn will typically make broad, sweeping curves; Theta\* prefers straight lines and supports them at any angle; and Smac 2D is essentially a classical A\* algorithm with cost-aware penalties.

> 由于规划问题主要受机器人类型的驱动，因此表格准确地总结了维护者向用户的建议。在圆形机器人领域，规划算法的选择取决于应用和期望的行为。NavFn通常会产生宽广的弯曲曲线；Theta\*偏爱直线，并以任何角度支持它们；Smac 2D实际上是一种具有成本感知惩罚的经典A\*算法。

::: note
::: title
Note
:::


These are simply the default and available plugins from the community. For a specific application / platform, you may also choose to use none of these and create your own, and that\'s the intention of the Nav2 framework. See the `writing_new_nav2planner_plugin`{.interpreted-text role="ref"} tutorial for more details. If you\'re willing to contribute this work back to the community, please file a ticket or contact a maintainer! They\'d love to hear from you.

> 这些只是社区中默认和可用的插件。 对于特定的应用程序/平台，您也可以选择不使用这些插件，而是自己创建，这就是Nav2框架的意图。 有关详细信息，请参阅`writing_new_nav2planner_plugin`{.interpreted-text role="ref"}教程。 如果您愿意将此工作返回社区，请提交票据或联系维护人员！ 他们很想听到您的消息。
:::

# Controller Plugin Selection


Nav2 provides a number of controller plugins out of the box. For a first-time setup, see `select_algorithm`{.interpreted-text role="ref"} for a more verbose breakdown of algorithm styles within Nav2, and `plugins`{.interpreted-text role="ref"} for a full accounting of the current list of plugins available (which may be updated over time).

> Nav2提供了一些控制器插件。要进行首次设置，请参见`select_algorithm`{.interpreted-text role="ref"}，了解Nav2中算法样式的详细信息，以及`plugins`{.interpreted-text role="ref"}，了解当前可用的插件列表（可能会随时更新）。


In general though, the following table is a good first-order description of the controller plugins available for different types of robot bases:

> 一般而言，以下表格是不同类型机器人底盘可用的控制器插件的良好首要描述：

```
      -----------------------------------------------------------------------------------
      Plugin Name      Supported Robot Types                 Task
      ---------------- ------------------------------------- ----------------------------
      DWB controller   Differential, Omnidirectional         Dynamic obstacle avoidance

      -----------------------------------------------------------------------------------

    +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| \| TEB Controller \| Differential, Omnidirectional, Ackermann, Legged \| Dynamic obstacle avoidance \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| RPP controller \| Differential, Ackermann, Legged \| Exact path following \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+ \| Rotation Shim \| Differential, Omnidirectional \| Rotate to rough heading \| +\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--+
```


All of the above controllers can handle both circular and arbitrary shaped robots in configuration.

> 所有上述控制器都可以处理配置中的圆形和任意形状的机器人。


Regulated Pure Pursuit is good for exact path following and is typically paired with one of the kinematically feasible planners (eg State Lattice, Hybrid-A\*, etc) since those paths are known to be drivable given hard physical constraints. However, it can also be applied to differential drive robots who can easily pivot to match any holonomic path. This is the plugin of choice if you simply want your robot to follow the path, rather exactly, without any dynamic obstacle avoidance or deviation. It is simple and geometric, as well as slowing the robot in the presence of near-by obstacles _and_ while making sharp turns.

> 精确路径跟踪的受控纯追踪很好，通常与运动学上可行的规划器（例如状态格子、混合A*等）配对使用，因为这些路径已知可以在硬物理约束下行驶。但它也可以应用于可以轻松旋转以匹配任何完整路径的差动驱动机器人。如果您只想让机器人沿着路径而不进行任何动态避障或偏离，这将是首选插件。它简单而几何，并且在附近有障碍物和弯道急转时能够减速机器人。


DWB and TEB are both options that will track paths, but also diverge from the path if there are dynamic obstacles present (in order to avoid them). DWB does this through scoring multiple trajectories on a set of critics. These trajectories are also generated via plugins that can be replaced, but support out of the box Omni and Diff robot types within the valid velocity and acceleration restrictions. These critics are plugins that can be selected at run-time and contain weights that may be tuned to create the desired behavior, such as minimizing path distance, minimizing distance to the goal or headings, and other action penalties that can be designed. This does require a bit of tuning for a given platform, application, and desired behavior, but it is possible to tune DWB to do nearly any single thing well.

> DWB和TEB都是可以跟踪路径的选项，但是如果存在动态障碍物，它们也会偏离路径（以避免它们）。DWB通过在一组评论者上评分多条轨迹来实现这一点。这些轨迹也是通过可以替换的插件生成的，但是在有效的速度和加速度限制下支持Omni和Diff机器人类型。这些评论者是可以在运行时选择的插件，其中包含可以调整的权重，以创建所需的行为，例如最小化路径距离，最小化到目标或标题的距离以及可以设计的其他动作惩罚。这确实需要为给定的平台，应用程序和所需行为进行一些调整，但是可以调整DWB来做几乎任何一件事。


TEB on the other hand implements an optimization based approach, generating a graph-solving problem for path tracking in the presence of obstacles. TEB is pretty good at handling dynamic situations well with other moving agents in the scene, but at a much higher compute cost that makes it largely unsuitable for smaller compute platform robots (e.g. minimum i3 and running at 20hz). This typically works pretty well out of the box, but to tune for specific behaviors, you may have to modify optimization engine parameters which are not as intuitive or rooted in something physical as DWB, but have pretty decent defaults.

> TEB另一方面采用基于优化的方法，生成图解决方案，以在存在障碍物的情况下进行路径跟踪。 TEB在处理动态情况方面非常出色，可以在场景中与其他移动代理一起使用，但计算成本要高得多，使其不适合更小的计算平台机器人（例如最低i3，以20hz运行）。 这通常可以很好地工作，但是要调整特定行为，您可能必须修改优化引擎参数，这些参数不像DWB那样直观或植根于某种物理，但具有相当不错的默认值。


Finally, the Rotation Shim Plugin helps assist plugins like TEB and DWB (among others) to rotate the robot in place towards a new path\'s heading before starting to track the path. This allows you to tune your local trajectory planner to operate with a desired behavior without having to worry about being able to rotate on a dime with a significant deviation in angular distance over a very small euclidean distance. Some controllers when heavily tuned for accurate path tracking are constrained in their actions and don\'t very cleanly rotate to a new heading. Other controllers have a \'spiral out\' behavior because their sampling requires some translational velocity, preventing it from simply rotating in place. This helps alleviate that problem and makes the robot rotate in place very smoothly.

> 最后，旋转垫插件有助于像TEB和DWB（以及其他）这样的插件，在开始跟踪路径之前，将机器人旋转到新路径的航向。这样可以调整本地轨迹规划器的行为，而无需担心在很小的欧几里得距离上，角距离会出现重大偏差。当严格调整以实现精确路径跟踪时，一些控制器的行动受到限制，无法平稳地旋转到新航向。其他控制器由于采样需要一定的平动速度，因此会出现“螺旋式外扩”的行为，从而无法只旋转不移动。该插件有助于缓解这个问题，使机器人非常平稳地旋转到原地。

::: note
::: title
Note
:::


These are simply the default and available plugins from the community. For a specific robot platform / company, you may also choose to use none of these and create your own. See the `writing_new_nav2controller_plugin`{.interpreted-text role="ref"} tutorial for more details. If you\'re willing to contribute this work back to the community, please file a ticket or contact a maintainer! They\'d love to hear from you.

> 这些只是社区中默认和可用的插件。 对于特定的机器人平台/公司，您也可以选择不使用这些插件，而是创建自己的插件。 有关详细信息，请参阅`writing_new_nav2controller_plugin`{.interpreted-text role="ref"}教程。 如果您愿意将此工作回贡献给社区，请提交工单或联系维护者！ 他们很乐意收到您的消息。
:::

# Caching Obstacle Heuristic in Smac Planners


Smac\'s Hybrid-A\* and State Lattice Planners provide an option, `cache_obstacle_heuristic`. This can be used to cache the heuristic to use between replannings to the same goal pose, which can increase the speed of the planner **significantly** (40-300% depending on many factors). The obstacle heuristic is used to steer the robot into the middle of spaces, respecting costs, and drives the kinematically feasible search down the corridors towards a valid solution. Think of it like a 2D cost-aware search to \"prime\" the planner about where it should go when it needs to expend more effort in the fully feasible search / SE2 collision checking.

> Smac的混合A*和状态格规划器提供了一个选项`cache_obstacle_heuristic`。它可以用来缓存到相同目标位置的重新规划之间的启发式，这可以显著提高规划器的速度（根据许多因素，可以提高40-300％）。障碍启发式用于引导机器人进入空间的中间，尊重成本，并驱动机械可行的搜索沿着走廊朝向有效的解决方案。把它想象成一个2D成本感知搜索，当它需要在完全可行的搜索/ SE2碰撞检测中投入更多努力时，它可以“预热”规划器去哪里。


This is useful to speed up performance to achieve better replanning speeds. However, if you cache this heuristic, it will not be updated with the most current information in the costmap to steer search. During planning, the planner will still make use of the newest cost information for collision checking, _thusly this will not impact the safety of the path_. However, it may steer the search down newly blocked corridors or guide search towards areas that may have new dynamic obstacles in them, which can slow things down significantly if entire solution spaces are blocked.

> 这有助于加快性能，以获得更好的重新规划速度。但是，如果你缓存这个启发式，它将不会更新成最新的成本地图信息来引导搜索。在规划过程中，规划器仍将利用最新的成本信息进行碰撞检查，`因此这不会影响路径的安全性`。但是，它可能会将搜索引导到新阻塞的走廊或引导搜索到可能有新动态障碍物的区域，如果整个解决方案被阻塞，这会大大减慢进度。


Therefore, it is the recommendation of the maintainers to enable this only when working in largely static (e.g. not many moving things or changes, not using live sensor updates in the global costmap, etc) environments when planning across large spaces to singular goals. Between goal changes to Nav2, this heuristic will be updated with the most current set of information, so it is not very helpful if you change goals very frequently.

> 因此，维护者建议只在大多数静态（例如，没有许多移动的东西或变化，全局成本地图中不使用实时传感器更新等）环境中规划大空间到单一目标时启用此功能。在Nav2的目标变更之间，该启发式算法将使用最新的信息集更新，因此如果您频繁更改目标，则不会很有帮助。

# Nav2 Launch Options


Nav2\'s launch files are made to be very configurable. Obviously for any serious application, a user should use `nav2_bringup` as the basis of their navigation launch system, but should be moved to a specific repository for a users\' work. A typical thing to do is to have a `<robot_name>_nav` configuration package containing the launch and parameter files.

> Nav2的启动文件非常可配置。显然，对于任何严肃的应用，用户应该将`nav2_bringup`作为他们的导航启动系统的基础，但应该移动到用户工作的特定存储库。典型的做法是拥有一个`<robot_name>_nav`配置包，其中包含启动和参数文件。


Within `nav2_bringup`, there is a main entryfile `tb3_simulation_launch.py`. This is the main file used for simulating the robot and contains the following configurations:

> 在`nav2_bringup`中，有一个主要的入口文件`tb3_simulation_launch.py`。这是用于模拟机器人的主要文件，包含以下配置：`Wait`


- `slam` : Whether or not to use AMCL or SLAM Toolbox for localization and/or mapping. Default `false` to AMCL.

> `- `slam`：是否使用AMCL或SLAM Toolbox进行定位和/或映射。默认为`false`到AMCL。

- `map` : The filepath to the map to use for navigation. Defaults to `map.yaml` in the package\'s `maps/` directory.

> - `map`：用于导航的地图的文件路径。默认为包的`maps/`目录中的`map.yaml`。

- `world` : The filepath to the world file to use in simulation. Defaults to the `worlds/` directory in the package.

> - `world`：用于模拟的世界文件的文件路径。默认为包中的`worlds/`目录。

- `params_file` : The main navigation configuration file. Defaults to `nav2_params.yaml` in the package\'s `params/` directory.

> `params_file`：主导航配置文件。默认为包`params/`目录中的`nav2_params.yaml`。

- `autostart` : Whether to autostart the navigation system\'s lifecycle management system. Defaults to `true` to transition up the Nav2 stack on creation to the activated state, ready for use.

> `自动启动`：是否自动启动导航系统的生命周期管理系统。默认为`true`，以便在创建时将Nav2堆栈转换到激活状态，准备使用。

- `use_composition` : Whether to launch each Nav2 server into individual processes or in a single composed node, to leverage savings in CPU and memory. Default `true` to use single process Nav2.

> 使用组合：是否将每个Nav2服务器单独放入单独的进程中，或者放入一个组合节点中，以节省CPU和内存。默认为`true`使用单个进程Nav2。

- `use_respawn` : Whether to allow server that crash to automatically respawn. When also configured with the lifecycle manager, the manager will transition systems back up if already activated and went down due to a crash. Only works in non-composed bringup since all of the nodes are in the same process / container otherwise.

> `使用_respawn`：是否允许服务器崩溃后自动重新启动。如果配置了生命周期管理器，管理器将在已激活并由于崩溃而关闭时将系统重新启动。只能在非组合启动中使用，因为所有节点都在同一个进程/容器中。

- `use_sim_time` : Whether to set all the nodes to use simulation time, needed in simulation. Default `true` for simulation.

> `使用模拟时间`：是否将所有节点设置为使用模拟时间，模拟中需要。默认为模拟时为`true`。

- `rviz_config_file` : The filepath to the rviz configuration file to use. Defaults to the `rviz/` directory\'s file.

> `- `rviz_config_file`：用于加载的rviz配置文件的文件路径。默认为`rviz/`目录中的文件。

- `use_simulator` : Whether or not to start the Gazebo simulator with the Nav2 stack. Defaults to `true` to launch Gazebo.

> `使用模拟器`：是否使用Nav2堆栈启动Gazebo模拟器。默认为`true`以启动Gazebo。

- `use_robot_state_pub` : Whether or not to start the robot state publisher to publish the robot\'s URDF transformations to TF2. Defaults to `true` to publish the robot\'s TF2 transformations.

> 使用机器人状态发布：是否开始机器人状态发布者以发布机器人的URDF变换到TF2。默认为`true`发布机器人的TF2变换。
- `use_rviz` : Whether or not to launch rviz for visualization. Defaults to `true` to show rviz.

- `headless` : Whether or not to launch the Gazebo front-end alongside the background Gazebo simulation. Defaults to `true` to display the Gazebo window.

> `headless`：是否同时启动Gazebo前端和后台Gazebo模拟。默认为`true`，以显示Gazebo窗口。
- `namespace` : The namespace to launch robots into, if need be.

- `use_namespace` : Whether or not to launch robots into this namespace. Default `false` and uses global namespace for single robot.

> `使用命名空間`：是否將機器人發射到此命名空間中。 預設為`false`，並使用全局命名空間為單個機器人。
- `robot_name` : The name of the robot to launch.

- `robot_sdf` : The filepath to the robot\'s gazebo configuration file containing the Gazebo plugins and setup to simulate the robot system.

> `- `robot_sdf` : 模拟机器人系统所需的Gazebo插件和设置的机器人的Gazebo配置文件的文件路径。

- `x_pose`, `y_pose`, `z_pose`, `roll`, `pitch`, `yaw` : Parameters to set the initial position of the robot in the simulation.

> `x_pose`、`y_pose`、`z_pose`、`roll`、`pitch`、`yaw`：用于设置模拟中机器人的初始位置的参数。

# Other Pages We\'d Love To Offer


If you are willing to chip in, some ideas are in <https://github.com/ros-planning/navigation.ros.org/issues/204>, but we\'d be open to anything you think would be insightful!

> 如果你愿意出资，一些想法可以在<https://github.com/ros-planning/navigation.ros.org/issues/204>找到，但我们也欢迎你提出任何你认为有价值的想法！
