---
tip: translate by baidu@2023-06-03 18:02:41
title: 3. Assisted Teleop
---

**Task description**

> **任务描述**

In mobile robot and autonomous vehicle navigation, there are situations where a human driver is required to intervene to get the vehicle out of a sticky situation. This can be both as a backup in case of autonomy failure as well as the primary function of the robot (e.g. telepresence robots).

> 在移动机器人和自动驾驶汽车导航中，**有时需要人类驾驶员进行干预，以使车辆摆脱困境。这既可以作为自主故障情况下的备份**，也可以作为机器人（例如远程呈现机器人）的主要功能。

This project\'s aim is to create an assisted teleop feature in Nav2 by means of a new behavior tree configuration file (the file that defines the flow of information for the navigation task) and potentially new plugins. This feature should make sure to use the local costmap and/or sensor data in order to avoid obstacles and take position and/or velocity commands to attempt to follow.

> 该项目的目的是通过一个新的行为树配置文件（定义导航任务信息流的文件）和潜在的新插件，在 Nav2 中创建一个辅助遥控器功能。该功能应确保使用本地成本图和/或传感器数据，以避开障碍物并执行位置和/或速度命令。

An example application of this is a telepresence robot, where a human driver is driving the robot through a space to visit in an office building or hospital. Another example would be an autonomous delivery robot stuck requiring a human driver to navigate it back into an open space for the robot to continue its task.

> 这方面的一个示例应用是远程呈现机器人，其中人类驾驶员驾驶机器人穿过办公楼或医院的空间进行访问。另一个例子是，一个自动送货机器人被卡住了，需要人类司机将其导航回一个空地，让机器人继续执行任务。

This will be an excellent chance to make a substantial new feature in the Nav2 system to be used by hundreds of robots in the future. This project could also be a good candidate for a ROSCon talk in future events.

> 这将是一个极好的机会，可以在 Nav2 系统中创造一个实质性的新功能，供未来数百个机器人使用。这个项目也可能是未来活动中 ROSCon 演讲的一个很好的候选者。

**Project difficulty: Medium**

> **项目难度：中等**

**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)

> **项目社区导师：Steve Macenski**[\@Steve Macenski](https://github.com/SteveMacenski)

**Mentor contact details: \[See link above, link in GitHub profile description\]**

> **导师联系方式：\[请参阅上面的链接，GitHub 简介中的链接\]**

**Project output requirements** - Integrations of an assisted teleop feature in Nav2 as either a set of plugins and/or a behavior tree configuration - Robot can successfully navigate a space by a user teleop without collision - If time allots, work on tuning / adding new critics to the DWB local planner to improve safety of its performance for users out of the box

> **项目输出要求**-将 Nav2 中的辅助遥控器功能集成为一组插件和/或行为树配置-机器人可以在没有碰撞的情况下通过用户遥控器成功导航空间-如果时间分配，请调整/添加新的批评者到 DWB 本地规划器，以提高其开箱即用的用户性能的安全性

**Skills required**

> **所需技能**

- C++, XML, ROS
- Mobile robot navigation experience
- Recommended: Gazebo simulation, ROS navigation, Behavior trees

**List of relevant open source software repositories and refs**

> **相关开源软件存储库和参考文献列表**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Github ticket](https://github.com/ros-planning/navigation2/issues/2226)
- [Navigation2](https://navigation.ros.org/)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
