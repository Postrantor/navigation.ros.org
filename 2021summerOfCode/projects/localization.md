---
tip: translate by baidu@2023-06-03 18:02:58
---

# 6. 2D/3D Localization Improvements {#localization}

**Task description**

The Navigation2 stack uses AMCL as its primary localization engine. Over the last 10 years, essentially no updates to AMCL has been made. This is due to the code base for this implementation of an Adaptive Monte Carlo Localizer is written in embedded C, not well structured, and very sensitive to changes. A-MCL implementations have been a hallmark of localization for over a decade but this particular implementation should be deprecated.

> Navigation2 堆栈使用 AMCL 作为其主要本地化引擎。在过去的 10 年里，基本上没有对 AMCL 进行任何更新。这是因为自适应蒙特卡罗定位器的代码库是用嵌入式 C 编写的，结构不好，对变化非常敏感。十多年来，A-MCL 实现一直是本地化的标志，但这种特定的实现应该被弃用。

Your target involves designing and creating a new localization engine for the Nav2 stack. The requirements of this are: - Support 2D laser scanners - Support 3D laser scanners, where 2D case could potentially be a simplified case - Accurately track the localization of a robot in a given occupancy grid

> 您的目标包括为 Nav2 堆栈设计和创建一个新的本地化引擎。其要求是：-支持 2D 激光扫描仪-支持 3D 激光扫描仪，其中 2D 情况可能是一个简化的情况-在给定的占用网格中准确跟踪机器人的定位

The reason that specific method is left open-ended is to allow for creativity, novelty, or reimplementation of a what you feel is best. We have, however, analyzed other MCL variants as being good options. This may include reimplementing an A-MCL that is designed to be modified with modular components and support sampling from a 3D lidar. Another option is a NDT-MCL using NDT 2D/3D scan matching. Other options may be proposed and discussed with mentors during the application phase. The task involves 3D as well since there is no standard 3D localizer in ROS 2 yet and more and more robust 3D SLAM libraries have emerged over the last 2 years.

> 之所以让特定的方法不受限制，是为了让你有创造力、新颖性，或者重新实现你觉得最好的东西。然而，我们已经分析了其他 MCL 变体是不错的选择。这可能包括重新实现 A-MCL，该 A-MCL 被设计为使用模块化组件进行修改，并支持 3D 激光雷达的采样。另一种选择是使用无损检测 2D/3D 扫描匹配的 NDT-MCL。在申请阶段，可以提出其他选择，并与导师讨论。该任务也涉及 3D，因为 ROS 2 中还没有标准的 3D 定位器，并且在过去两年中出现了越来越多强大的 3D SLAM 库。

An optional but recommended feature of this work would be to also accept the inputs from multiple laser scanners. However it is not strictly required.

> 这项工作的一个可选但推荐的功能是也接受来自多个激光扫描仪的输入。然而，这并不是严格要求的。

**Project difficulty: High**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**
**Project output requirements** - 2D and 3D localization system based on laser scanners - 65% or higher test coverage - Designed with modular components that can be reliably modified over time

> **项目输出要求**-基于激光扫描仪的二维和三维定位系统-65% 或更高的测试覆盖率-采用模块化组件设计，可随着时间的推移进行可靠修改

**Skills required**

- C/C++
- Localization, particle filter, or SLAM techniques
- Ability to read and implement academic works
- Recommended: Gazebo simulation and Navigation experience

**List of relevant open source software repositories and refs**

> **相关开源软件存储库和参考文献列表**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Original Github Issue](https://github.com/ros-planning/navigation2/issues/1391)
- [Navigation2](https://navigation.ros.org/)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
