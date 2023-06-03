---
tip: translate by baidu@2023-06-03 18:03:08
title: 5. Navigation Safety Node
---

**Task description**

The aim of this project is to create a safety watchdog node to ensure the robot is acting properly and not about to collide with an obstacle. Typical safety-rated lidars will contain \"safety zones\" whereas if any sensor points are located in a box around the lidar, then the lidar will send a signal to the robot to stop due to a potential collision. However, less and less people are using safety-rated lidars as consumer available lidars are dropping in cost and 3D lidars are seeing more use in mobile robotics.

> 该项目的目的是创建一个安全监督节点，以确保机器人动作正常，不会与障碍物相撞。典型的安全级激光雷达将包含“安全区”，而如果任何传感器点位于激光雷达周围的盒子中，则激光雷达将向机器人发送信号，使其因潜在碰撞而停止。然而，随着消费者可用的激光雷达成本下降，3D 激光雷达在移动机器人中的应用越来越多，使用安全级激光雷达的人越来越少。

Your project will be to re-create this logic at the Navigation level. While this wouldn\'t be \"safety certified\", this is a significant functional improvement on safety that could potentially safe real people from real injuries in the real-world. The project will be to create a node that sits below the navigation stack but above the robot controller to do the following:

> 您的项目将在导航级别重新创建此逻辑。虽然这不会是“安全认证”，但这是对安全的一个重大功能改进，可能会使现实世界中的真人免受真正的伤害。该项目将创建一个位于导航堆栈下方但位于机器人控制器上方的节点，以执行以下操作：

- Take in the current command velocity from navigation and the most recent laser or RGBD scan
- Projecting the velocity forward in time `N` seconds, check if that velocity will result in a collision with any sensor measurements

> -在时间“N”秒内向前投射速度，检查该速度是否会导致与任何传感器测量值发生碰撞

- If not, allow the velocity command through to the base
- If it does collide, scale back the velocity such that the robot will always be at minimum `N` seconds from a collision

> -如果它确实发生了碰撞，则缩小速度，使机器人在碰撞后始终处于最小“N”秒

- Optionally if a flag is set, if `M` or more points are in defined bounding boxes around the robot, send only `0` commands to enact an emergency stop.

> -可选地，如果设置了标志，如果“M”或更多点位于机器人周围定义的边界框中，则仅发送“0”命令以实施紧急停止。

This will be an excellent chance to make mobile robots and Nav2 users significantly safer and run at higher speeds in their production or research environments. It goes a long way for functional safety for those not using safety-rated lidars which contain similar features.

> 这将是一个极好的机会，使移动机器人和 Nav2 用户在其生产或研究环境中更加安全，并以更高的速度运行。对于那些没有使用具有类似功能的安全级激光雷达的人来说，这对功能安全有很大帮助。

**Project difficulty: Medium**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**

**Project output requirements** - Creation of a ROS 2 node that will prevent robot collision based on lidar and/or RGBD data - Node should be able to run at lidar data speed (40hz+) and adjust velocity commands accordingly - If time allots, work on tuning / adding new critics to the DWB local planner to improve safety of its performance for users out of the box

> **项目输出要求**-创建一个 ROS 2 节点，该节点将基于激光雷达和/或 RGBD 数据防止机器人碰撞-节点应能够以激光雷达数据速度（40hz+）运行，并相应地调整速度命令-如果时间分配，请调整 DWB 本地规划器/添加新的批评者，以提高其性能的安全性，为用户提供开箱即用的服务

**Skills required**

- C++, ROS
- Mobile robot navigation experience
- Geometry and statistics
- Recommended: Gazebo simulation, ROS navigation

**List of relevant open source software repositories and refs**

> **相关开源软件存储库和参考文献列表**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Github ticket](https://github.com/ros-planning/navigation2/issues/1899)
- [Navigation2](https://navigation.ros.org/)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
