---
tip: translate by baidu@2023-06-03 18:03:12
title: 6. Semantic Integration
...

**Task description**

This project is to create a semantics library and integrate it into the Nav2 system. The major aim is to enable a generic semantic representation format that navigation (and potentially other ROS projects) may use for representing their environment, objects within it, or points of interest.

> 该项目旨在创建一个语义库，并将其集成到 Nav2 系统中。主要目的是实现导航（以及潜在的其他 ROS 项目）可以用于表示其环境、其中的对象或兴趣点的通用语义表示格式。

After creating the generic representation, your project will be to create demonstrations within Nav2 using this capability including a route planning server to replace the planner server in situations where you have a pre-defined set of potential locales (non-free space planning) and another demonstration of your choice.

> 创建通用表示后，您的项目将使用此功能在 Nav2 中创建演示，包括在您有一组预定义的潜在地点（非自由空间规划）和您选择的另一个演示的情况下，使用路线规划服务器来替换规划器服务器。

**Project difficulty: Hard**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**

**Project output requirements** - Generic semantics standard added to Nav2 documentation - Generic semantics ROS 2 library that implements the standard and makes it easy for applications to get, retreive, or analyze semantic data for custom purposes - A route server to enable navigation-graph and/or route following capabilities - 1 more demonstration using the semantics library of your choice (could be costmap layer with different rules in different rooms or with different objects, a multi-story building demo using semantic info to allow a robot to plan and execute multi-story trajectories, etc)

> **项目输出需求**-添加到 Nav2 文档中的通用语义标准-实现该标准并使应用程序易于获取、检索、，或为自定义目的分析语义数据-一个启用导航图和/或路线跟踪功能的路线服务器-1 个使用您选择的语义库的更多演示（可以是在不同房间或不同对象中具有不同规则的成本图层，使用语义信息允许机器人规划和执行多层轨迹的多层建筑演示，等等）

**Skills required**

- C++, Python, ROS 2
- Mobile robot navigation or manipulation experience
- Perception, semantic information motivation, or similar.
- Recommended: Gazebo simulation, ROS navigation

**List of relevant open source software repositories and refs**

> **相关开源软件存储库和参考文献列表**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Github ticket](https://github.com/ros-planning/navigation2/issues/1595)
- [Github ticket2](https://github.com/ros-planning/navigation2/issues/2229)
- [Navigation2](https://navigation.ros.org/)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
