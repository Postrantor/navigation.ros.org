---
tip: translate by baidu@2023-06-03 18:02:55
---

# 3. Port Grid Maps to ROS 2 and Environmental Model {#grid_maps}

**Task description** Grid Maps was created by ETH Zurich and later transferred to ANYbotics. It is a universal grid map library for mobile robotic mapping that can be used as the basis of environmental models and various forms of grid maps available in ROS 1. This library is one of the top downloaded ROS packages. Your task will be to work with the community and the mentor to port grid_maps metapackage from ROS 1 to ROS 2 and help develop the next generation environment model in ROS 2 to replace costmap_2d.

> **任务描述**网格地图由苏黎世联邦理工学院创建，后来转移到 ANYbotics。它是一个用于移动机器人地图绘制的通用网格地图库，可作为 ROS 1 中可用的环境模型和各种形式网格地图的基础。这个库是下载量最大的 ROS 软件包之一。您的任务是与社区和导师合作，将 grid_maps 元包从 ROS 1 移植到 ROS 2，并帮助开发 ROS 2 中的下一代环境模型，以取代 costmap_2d。

This will involve porting code from ROS 1 to ROS 2, analyzing uses of the environmental model to define an abstract interface to allow replacement of costmap_2d with grid_map, and building up the basic grid-operations for costmaps. It is not expected to complete the full new model with sensor processing over the course of the summer. If completed early, you may be able to help design a gradient model to complement your implemented costmap model using grid_maps. This will allow robots to select a gradient or a costmap model on startup.

> 这将涉及将代码从 ROS 1 移植到 ROS 2，分析环境模型的用途，以定义一个抽象接口，从而允许用 grid_map 替换 costmap_2d，并建立 costmap 的基本网格操作。预计不会在夏季完成传感器处理的全新型号。如果提前完成，您可能能够帮助设计一个梯度模型，以使用 grid_maps 来补充您实现的成本图模型。这将允许机器人在启动时选择梯度或成本图模型。

**Project difficulty: High**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**
**Project output requirements**

- Grid Maps ported to ROS 2 and merged into the main ROS 2 branch
- Defined plugin interfaces to replace costmap 2D with grid maps
- Implementing low-level operations on top of grid_maps to replace the base costmap_2d object.
- You will not be expected to reimplement the full sensor processing mechanics of costmap_2d.

**Skills required**

- C++ and Git
- ROS recommended ROS 2, but you can pick it up before starting
- Coordinate transformations and basic geometry
- Recommended: Gazebo simulation and Navigation experience

**List of relevant open source software repositories and refs**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Original github issue 1](https://github.com/ros-planning/navigation2/issues/1278)
- [Original github issue 2](https://github.com/ros-planning/navigation2/issues/1517)
- [Navigation2](https://navigation.ros.org/)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
