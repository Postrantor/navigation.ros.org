---
tip: translate by baidu@2023-06-03 18:03:21
title: 8. Convert Twist to TwistStamped in Ecosystem and Run-Time Configuration
---

**Task description**

This project is comprised of 2 smaller projects that can be easily worked on in parallel.

> 该项目由两个较小的项目组成，可以很容易地并行进行。

Subproject A: Convert Twist to TwistStamped in Ecosystem

> 子项目 A：在生态系统中将扭曲转化为扭曲印记

The aim of this project is to identify places in the ROS 2 ecosystem that make use of `Twist` in the context of `cmd_vel` coming out of Navigation. This includes things like Nav2, ROS 2 Control, Gazebo ROS Plugins, Yuk\'s Velocity Filter, major robot drivers, etc. A set of previously identified places is shown in the ticket linked below.

> 该项目的目的是确定 ROS 2 生态系统中在 Navigation 中出现“cmd_vel”的情况下使用“Twist”的地方。这包括 Nav2、ROS 2 Control、Gazebo ROS 插件、Yuk's Velocity Filter、主要机器人驱动程序等。下面链接的票证中显示了一组先前确定的位置。

Once you\'ve created a list of places in the ecosystem where it is used, your project will be to submit PRs on their ROS 2 branches to change the interface to make use of a `TwistStamped` instead of a `Twist`.

> 一旦你在生态系统中创建了一个使用位置列表，你的项目将在其 ROS 2 分支上提交 PR，以更改界面，使用“TwistStamped”而不是“Twist”。

Subproject B: Run-time Reconfiguration of Parameters

> 子项目 B：运行时参数重新配置

In the meantime while you\'re waiting for PRs to be merged or blocked by reviews on converting all of the ecosystems `cmd_vel` use of `Twist` to `TwistStamped`, your project will be to enable run-time reconfiguration of the major parameters in Nav2. In ROS 2 this is done via the parameter change event callbacks. See tickets below for a list of plugins or servers needing dynamically reconfigurable parameter support added.

> 与此同时，当您正在等待 PR 被关于将所有生态系统“cmd_vel”使用“Twist”转换为“TwistStamped”的审查合并或阻止时，您的项目将启用 Nav2 中主要参数的运行时重新配置。在 ROS 2 中，这是通过参数更改事件回调来完成的。请参阅下面的票证，以获取需要添加动态可重新配置参数支持的插件或服务器的列表。

**Project difficulty: Medium**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**
**Project output requirements** - Convert all the major ecosystem projects into TwistStamped - Enable run-time reconfiguration of the remaining plugins and servers in Nav2 missing - Ensure that reconfiguration is thread-safe by using locks, atomic variables, or callback groups

> **项目输出要求**-将所有主要的生态系统项目转换为 TwistStamped-启用 Nav2 中剩余插件和服务器的运行时重新配置-通过使用锁、原子变量或回调组确保重新配置是线程安全的

**Skills required**

- C++, Python3
- ROS 2

**List of relevant open source software repositories and refs**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Github ticket](https://github.com/ros-planning/navigation2/issues/956)
- [Github ticket2](https://github.com/ros-planning/navigation2/issues/1594)
- [Navigation2](https://navigation.ros.org/)
- [Some related works](https://alyssapierson.files.wordpress.com/2018/05/pierson2018.pdf)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
