---
tip: translate by baidu@2023-06-03 18:02:45
...

# 1. Create a Configuration Assistant (Analog to MoveIt) {#create_moveit_analog}

**Task description**

[Moveit](https://moveit.ros.org/) has long has a QT [configuration assistant](http://docs.ros.org/kinetic/api/moveit_tutorials/html/doc/setup_assistant/setup_assistant_tutorial.html). This setup assistent helps the user configure their UDRF and needs to setup MoveIt configuration files.

> [移动](https://moveit.ros.org/)长期拥有 QT [配置助手](http://docs.ros.org/kinetic/api/moveit_tutorials/html/doc/setup_assistant/setup_assistant_tutorial.html)。此设置助手可帮助用户配置其 UDRF，并需要设置 MoveIt 配置文件。

A configuration assistant could be extremely beneficial to Navigation2 users as a way to minimize friction. We should provide a gui tool to cover the following configurations:

> 配置助手对于 Navigation2 用户来说是非常有益的，可以最大限度地减少摩擦。我们应该提供一个 gui 工具来涵盖以下配置：

- the broad strokes with the costmap, with a visualizer to show the user what it will look like
- Select configurable costmap layers
- Select recovery behavior parameters
- URDF, footprint, and frame selection to make sure the options comply with standards, planner, and controller
- Set minimum and maximum speed and other kinematic parameters
- Select from a dropdown of possible planners and controllers
- Helpful notes throughout the prompts to aid in selecting appropriate parameters
- Selecting at behavior tree
- \@steve please add more specific options

After the items are configured, there should be a preview to see how the parameters effect the robot.

> 配置项目后，应该有一个预览，**看看参数如何影响机器人**。

**Project difficulty: High**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**
**Contact information for the cooperating mentor (optional): juzhenatpku@gmail.commailto:juzhenatpku@gmail.com**
**Project output requirements**

- A QT based GUI configuration assistant that support the parameters listed above
- A preview panel to display the parameters\' effection on the robot

**Skills required**

- C++, Python3, QT framework
- JSON/XML parsing
- 3D programming (maybe needed in the preview)
- Recommended: Gazebo simulation, ROS, and Navigation experience

**List of relevant open source software repositories and refs**

> **相关开源软件存储库和参考文献列表**

- [QT](https://www.qt.io/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Original github issue page](https://github.com/ros-planning/navigation2/issues/1721)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
