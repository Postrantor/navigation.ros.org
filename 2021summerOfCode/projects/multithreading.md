---
tip: translate by baidu@2023-06-03 18:03:01
title: 4. Navigation MultiThreading
---

**Task description**

The aim of this project is the significantly improve the run-time performance of Nav2 making sure to leverage the full capabilities of multi-processor core CPUs. We seek to identify areas in the Nav2 stack that could leverage multi-threading or parallel processing to speed up computations and improve overall user performance on a broad range of compute platforms.

> 该项目的目的是**显著提高 Nav2 的运行时性能，确保充分利用多处理器核心 CPU 的全部功能**。我们试图在 Nav2 堆栈中确定可以利用多线程或并行处理来加速计算并提高广泛计算平台上的整体用户性能的区域。

Some examples include: - AMCL particle cloud updates - Costmap layer updates - Costmap sensor data population - Controller critic evalulation - Collision checking - Voxel grid ray casting - and more.

> 一些示例包括：-AMCL 粒子云更新-成本贴图层更新-成本映射传感器数据填充-控制器评论家评估-碰撞检查-体素网格光线投射-等等。

We are seeking a student interested in learning about multi-threading and parallel processing, ideally with some exposure to these concepts and libraries already, to analyze potential areas for parallel computing. Then, select the top candidates and implement them with parallel processing and benchmark the improvements to the Nav2 stack they provide.

> 我们正在寻找一位对学习多线程和并行处理感兴趣的学生，最好是已经接触了这些概念和库，以分析并行计算的潜在领域。然后，选择最优秀的候选者，并通过并行处理实现它们，并对它们提供的 Nav2 堆栈的改进进行基准测试。

This will be an excellent chance to apply (or obtain) C++ parallel computing skills while also learning a great deal about how to build mobile robot navigation systems \-- both very valuable skillsets.

> 这将是一个应用（或获得）C++ 并行计算技能的绝佳机会，同时也将学习大量关于如何构建移动机器人导航系统的知识——这两种技能都非常有价值。

**Project difficulty: Medium**
**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)
**Mentor contact details: \[See link above, link in GitHub profile description\]**
**Project output requirements** - Analysis of potential areas in the Nav2 stack that can benefit from parallel processing - Integrations of 3-5 areas in the Nav2 stack that non-trivially improves run-time performance of the stack using multithreading - Benchmark performance improvements

> **项目输出要求**-分析 Nav2 堆栈中可以从并行处理中受益的潜在区域-集成 Nav2 堆栈的 3-5 个区域，使用多线程显著提高堆栈的运行时性能-基准性能改进

**Skills required**

- C++, ROS
- Mobile robot navigation experience
- Working knowledge (or ability to quickly obtain) on one or more of: TBB, OpenMP, OpenCL, Cuda, and similar

> -关于以下一种或多种的工作知识（或快速获得的能力）：TBB、OpenMP、OpenCL、Cuda 等

- Recommended: Gazebo simulation, ROS navigation

**List of relevant open source software repositories and refs**

> **相关开源软件存储库和参考文献列表**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Github ticket](https://github.com/ros-planning/navigation2/issues/2042)
- [Navigation2](https://navigation.ros.org/)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
