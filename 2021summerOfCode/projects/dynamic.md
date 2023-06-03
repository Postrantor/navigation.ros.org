---
tip: translate by baidu@2023-06-03 18:02:51
title: 1. Navigation Dynamic Obstacle Integration
---

**Task description**

The Navigation Stack has long provided robust navigation in a wide range of environments. Controllers have been developed to operate effectively in the presence of dynamic obstacles without explicitly modeling the characteristics of dynamic obstacles. However, as the field has progressed and we see more and more robots using ROS deployed in human-filled spaces, more consideration must be taken with respect to dynamic obstacles such as people, carts, animals, and vehicles.

> 导航堆栈长期以来一直在各种环境中提供强大的导航功能。控制器已被开发为在存在动态障碍物的情况下有效地操作，而无需对动态障碍物特性进行明确建模。然而，随着该领域的发展，我们看到越来越多的机器人使用 ROS 部署在充满人的空间中，必须更多地考虑人、推车、动物和车辆等**动态障碍**。

Your task will be to create integrations with existing machine learning tools that create dynamic obstacle information (ComplexYolo, Yolo3D, etc) and tie them into the navigation stack for use. It is not in the scope for you to retrain or otherwise become an expert in 3D machine learning, but some basic knowledge will be helpful. We already have a starting point in the project links below that needs to be driven to completion. This includes completing the on-going work to integrate yolact edge into this work to replace detectron2 and benchmark these capabilities on GPUs to verify sufficient run-time performance, as well as other tangental feature development.

> 您的任务是创建与现有机器学习工具的集成，这些工具可以创建动态障碍物信息（ComplexYolo、Yolo3D 等），并将它们绑定到导航堆栈中以供使用。这不在你重新培训或以其他方式成为 3D 机器学习专家的范围内，但一些基本知识会有所帮助。我们在下面的项目链接中已经有了一个起点，需要推动它完成。这包括完成正在进行的工作，将 yolact-edge 集成到这项工作中，以取代 detectron2，并在 GPU 上对这些功能进行基准测试，以验证足够的运行时性能，以及其他切向功能开发。

This task will involve identifying a few techniques that produce position and velocity information about dynamic obstacles that can run on a mobile robot (using high-power Intel CPU, Nvidia Jetson SoC, external GPUs, etc) and get them running with ROS and Navigation. Next, you will help create a new costmap layer to use this information to mark the dynamic obstacle in the costmap to ensure a robot does not collide with a future trajectory of an obstacle.

> 这项任务将涉及识别一些技术，这些技术可以产生关于动态障碍物的位置和速度信息，这些障碍物可以在移动机器人上运行（使用高功率 Intel CPU、Nvidia Jetson SoC、外部 GPU 等），并使其与 ROS 和 Navigation 一起运行。接下来，您将帮助创建一个新的成本图层，使用这些信息在成本图中标记动态障碍物，以确保机器人不会与障碍物的未来轨迹发生碰撞。

If time permits, you may also work to also integrate this dynamic information into a path planner and/or controller to help in direct motion consideration. This will likely be in collaboration with another community member.

> 如果时间允许，您还可以将这些动态信息集成到路径规划器和/或控制器中，以帮助直接考虑运动。这可能会与另一个社区成员合作。

**Project difficulty: High**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**
**Project output requirements** - Integrations of 1 method of dynamic obstacle detection in ROS 2, using machine learning - Test these capabilities on real data or a live robot to demonstrate functionality - 85% test coverage or higher

> **项目输出要求**-集成 ROS 2 中的 1 种动态障碍物检测方法 2，使用机器学习-在真实数据或实时机器人上测试这些能力以演示功能-85% 或更高的测试覆盖率

**Skills required**

- C++, Python, ROS
- Mobile robot navigation experience
- Geometry and statistics
- Recommended: Gazebo simulation, machine learning, ROS navigation

**List of relevant open source software repositories and refs**

> **相关开源软件存储库和参考文献列表**

- [Starting project](https://github.com/ros-planning/navigation2_dynamic/)
- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Github ticket](https://github.com/ros-planning/navigation2/issues/1617)
- [Navigation2](https://navigation.ros.org/)
- [Some related works](https://alyssapierson.files.wordpress.com/2018/05/pierson2018.pdf)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
