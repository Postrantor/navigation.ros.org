---
tip: translate by openai@2023-06-02 15:29:47
title: Follow Dynamic Point
...

This behavior tree implements a navigation behavior from a starting point, attempting to follow a dynamic point over time. This \"dynamic point\" could be a person, another robot, a virtual carrot, anything. The only requirement is that the pose you\'d like to follow is published to the topic outlined in the `GoalUpdater` BT node.

> 这个行为树实现了从一个起点开始的导航行为，试图随着时间跟随一个动态点。这个“动态点”可以是一个人，另一个机器人，一个虚拟的胡萝卜，任何东西。唯一的要求是你**想要跟随的姿态被发布到`GoalUpdater` BT 节点中提出的主题**。

In this tree, we replan at 1 hz just as we did in `behavior_tree_nav_to_pose`{.interpreted-text role="ref"} using the `ComputePathToPose` node. However, this time when we replan, we update the `goal` based on the newest information in on the updated goal topic. After we plan a path to this dynamic point, we use the `TruncatePath` node to remove path points from the end of the path near the dynamic point. This behavior tree node is useful so that the robot always remains at least `distance` away from the obstacle, even if it stops. It also smooths out any off path behavior involved with trying to path plan towards a probably occupied space in the costmap.

> 在这棵树中，我们就像在`behavior_tree_nav_to_pose`中一样以 **1 Hz 重新规划**。但是，这一次，我们更新了`goal`，根据更新的目标主题中的最新信息。在为这个动态点规划路径后，我们使用`TruncatePath`节点从路径末尾的动态点附近删除路径点。这个行为树节点很有用，因此即使机器人停下来，它也总是保持至少`distance`的距离离障碍物。它也平滑了尝试在代价地图中路径规划向可能占用空间的行为。

After the new path to the dynamic point is computed and truncated, it is again passed to the controller via the `FollowPath` node. However, note that it is under a `KeepRunningUntilFailure` decorator node ensuring the controller continues to execute until a failure mode. This behavior tree will execute infinitely in time until the navigation request is preempted or cancelled.

> **当计算出新的动态点的路径并截断之后，它会通过`FollowPath`节点再次传递给控制器。** 但是，请注意，它处于`KeepRunningUntilFailure`装饰节点之下，确保控制器继续执行直到失败模式。此行为树将无限期执行，直到导航请求被抢占或取消。

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <PipelineSequence name="NavigateWithReplanning">
      <RateController hz="1.0">
        <Sequence>
          <GoalUpdater input_goal="{goal}" output_goal="{updated_goal}">
            <ComputePathToPose goal="{updated_goal}" path="{path}" planner_id="GridBased"/>
          </GoalUpdater>
        <TruncatePath distance="1.0" input_path="{path}" output_path="{truncated_path}"/>
        </Sequence>
      </RateController>
      <KeepRunningUntilFailure>
        <FollowPath path="{truncated_path}" controller_id="FollowPath"/>
      </KeepRunningUntilFailure>
    </PipelineSequence>
  </BehaviorTree>
</root>
```
