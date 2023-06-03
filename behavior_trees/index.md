---
tip: translate by baidu@2023-06-06 09:03:29
title: Nav2 Behavior Trees
---

::: {.toctree maxdepth="1"}

overview/nav2_specific_nodes.rst overview/detailed_behavior_tree_walkthrough.rst trees/nav_to_pose_recovery.rst trees/nav_through_poses_recovery.rst trees/nav_to_pose_and_pause_near_goal_obstacle.rst trees/nav_to_pose_with_consistent_replanning_and_if_path_becomes_invalid.rst trees/follow_point.rst trees/odometry_calibration.rst
:::

Nav2 is an incredibly reconfigurable project. It allows users to set many different plugin types, across behavior trees, core algorithms, status checkers, and more! This section highlights some of the example behavior tree xml files provided by default in the project to do interesting tasks. It should be noted that these can be modified for your specific application, or used as a guide to building your own application-specific behavior tree. These are some exemplary examples of how you can reconfigure your navigation behavior significantly by using behavior trees. Other behavior trees are provided by Nav2 in the `nav2_bt_navigator` package, but this section highlights the important ones.

> Nav2 是一个令人难以置信的可重新配置项目。它允许用户设置许多不同的插件类型，包括**行为树、核心算法、状态检查器**等等！本节重点介绍了项目中默认提供的用于执行有趣任务的一些示例行为树 xml 文件。需要注意的是，这些可以针对您的特定应用程序进行修改，也可以用作构建您自己的特定于应用程序的行为树的指南。以下是一些示例，说明如何通过使用行为树来显著地**重新配置导航行为**。Nav2 在“Nav2_bt_navigator”包中提供了其他行为树，但本节重点介绍了重要的行为树。

A **very** basic, but functional, navigator can be seen below.

> 下面可以看到一个非常基本但功能强大的导航器。

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <PipelineSequence name="NavigateWithReplanning">
      <DistanceController distance="1.0">
        <ComputePathToPose goal="{goal}" path="{path}"/>
      </DistanceController>
      <FollowPath path="{path}"/>
    </PipelineSequence>
  </BehaviorTree>
</root>
```

This behavior tree will simply plan a new path to `goal` every 1 meter (set by `DistanceController`) using `ComputePathToPose`. If a new path is computed on the `path` blackboard variable, `FollowPath` will take this `path` and follow it using the server\'s default algorithm.

> 此行为树只需使用`ComputePathToPose`每隔 1 米（由`DistanceController`设置）规划一条通向`goal`的新路径。如果在`path`黑板变量上计算新路径，`FollowPath`将采用该`path`，并使用服务器的默认算法跟随它。

This tree contains:

- No recovery methods
- No retries on failure
- No selected planner or controller algorithms
- No nodes to contextually change settings for optimal performance
- No integration with automatic door, elevator, or other APIs
- No user provided custom BT nodes
- No subtrees for other behaviors like docking, following, etc.

All of this, and more, can be set and configured for your customized navigation logic in Nav2.

> 所有这些，以及更多，都可以在 Nav2 中为您的自定义导航逻辑进行设置和配置。
