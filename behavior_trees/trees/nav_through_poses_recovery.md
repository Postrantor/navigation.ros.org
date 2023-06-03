---
tip: translate by openai@2023-06-02 15:30:36
title: Navigate Through Poses
---

This behavior tree implements a navigation behavior from a starting point, through many intermediary hard pose constraints, to a final goal in freespace. It contains both use of custom behaviors for recovery in specific sub-contexts as well as a global recovery subtree for system-level failures. It also provides the opportunity for users to retry tasks multiple times before returning a failed state.

> 这个行为树实现了**从起点经过许多中间硬体姿态约束，到达自由空间中最终目标的导航行为**。它既包括在特定子上下文中使用自定义行为进行恢复，也包括用于系统级故障的全局恢复子树。它还为用户提供了在返回失败状态之前重试任务的机会。

The `ComputePathThroughPoses` and `FollowPath` BT nodes both also specify their algorithms to utilize. By convention we name these by the style of algorithms that they are (e.g. not `DWB` but rather `FollowPath`) such that a behavior tree or application developer need not worry about the technical specifics. They just want to use a path following controller.

> `ComputePathThroughPoses`和`FollowPath`BT 节点也指定它们要使用的算法。按照惯例，我们按照它们所使用的算法类型来命名它们（例如，不是`DWB`而是`FollowPath`），这样行为树或应用程序开发人员就不必担心技术细节。他们只想使用路径跟踪控制器。

> [!NOTE]
> 这里提到了一些明确的需求对于“恢复”的场景！

In this behavior tree, we attempt to retry the entire navigation task 6 times before returning to the caller that the task has failed. This allows the navigation system ample opportunity to try to recovery from failure conditions or wait for transient issues to pass, such as crowding from people or a temporary sensor failure.

> 在这个行为树中，我们**尝试重试整个导航任务 6 次**，然后再向调用者返回任务失败的消息。这**使得导航系统有充足的机会尝试从失败状态中恢复或等待暂时性问题**通过，例如来自人们的拥挤或暂时的传感器故障。

In nominal execution, this will replan the path at every 3 seconds and pass that path onto the controller, similar to the behavior tree in `behavior_trees`{.interpreted-text role="ref"}. The planner though is now `ComputePathThroughPoses` taking a vector, `goals`, rather than a single pose `goal` to plan to. The `RemovePassedGoals` node is used to cull out `goals` that the robot has passed on its path. In this case, it is set to remove a pose from the poses when the robot is within `0.5` of the goal and it is the next goal in the list. This is implemented such that replanning can be computed after the robot has passed by some of the intermediary poses and not continue to try to replan through them in the future. This time, if the planner fails, it will trigger contextually aware recoveries in its subtree, clearing the global costmap. Additional recoveries can be added here for additional context-specific recoveries, such as trying another algorithm.

> 在名义执行中，每 3 秒重新规划一次路径，并将路径发送到控制器，类似于`behavior_trees`中的行为树。规划器现在是`ComputePathThroughPoses`，它**接受一个向量`goals`，而不是单个姿势`goal`来规划**。使用`RemovePassedGoals`节点来剔除机器人已经经过的`goals`。在这种情况下，它被设置为当机器人距离目标在`0.5`之内，并且是列表中的下一个目标时，从姿势中移除一个姿势。这样实现，机器人经过一些中间姿势后，可以重新规划，而不是继续尝试经过它们的规划。这次，**如果规划失败，它将触发其子树中的上下文感知恢复，清除全局成本地图。可以在此处添加其他上下文特定的恢复，例如尝试另一种算法**。

Similarly, the controller has similar logic. If it fails, it also attempts a costmap clearing of the local costmap impacting the controller. It is worth noting the `GoalUpdated` node in the reactive fallback. This allows us to exit recovery conditions when a new goal has been passed to the navigation system through a preemption. This ensures that the navigation system will be very responsive immediately when a new goal is issued, even when the last goal was in an attempted recovery.

> 同样，控制器具有相似的逻辑。**如果失败，它还尝试清除影响控制器的本地成本图**。值得注意的是反应性回退中的`GoalUpdated`节点。这使我们能够在通过抢先行动传递到导航系统的新目标时退出恢复条件。这确保了当新目标被发布时，即使上一个目标处于尝试恢复状态，导航系统也会非常迅速地响应。

If these contextual recoveries fail, this behavior tree enters the recovery subtree. This subtree is reserved for system-level failures to help resolve issues like the robot being stuck or in a bad spot. This subtree also has the `GoalUpdated` BT node it ticks every iteration to ensure responsiveness of new goals. Next, the recovery subtree will tick the costmap clearing operations, spinning, waiting, and backing up. After each of the recoveries in the subtree, the main navigation subtree will be reattempted. If it continues to fail, the next recovery in the recovery subtree is ticked.

> **如果这些上下文恢复失败，这个行为树会进入恢复子树**。这个子树被保留用于系统级别的故障，以帮助解决机器人被卡住或处于不好的位置等问题。这个子树还有 `GoalUpdated` BT 节点，它每次迭代都会滴答，以确保新目标的响应性。**接下来，恢复子树将滴答成本图清理操作、旋转、等待和后退。在子树的每个恢复后，主导航子树将重新尝试。如果继续失败，恢复子树中的下一个恢复将被滴答。**

While this behavior tree does not make use of it, the `PlannerSelector`, `ControllerSelector`, and `GoalCheckerSelector` behavior tree nodes can also be helpful. Rather than hardcoding the algorithm to use (`GridBased` and `FollowPath`), these behavior tree nodes will allow a user to dynamically change the algorithm used in the navigation system via a ROS topic. It may be instead advisable to create different subtree contexts using condition nodes with specified algorithms in their most useful and unique situations. However, the selector nodes can be a useful way to change algorithms from an external application rather than via internal behavior tree control flow logic. It is better to implement changes through behavior tree methods, but we understand that many professional users have external applications to dynamically change settings of their navigators.

> 尽管这个行为树没有使用它，但`PlannerSelector`、`ControllerSelector`和`GoalCheckerSelector`行为树节点也可能有所帮助。它们不是将算法（`GridBased`和`FollowPath`）硬编码，而是**允许用户通过 ROS 主题动态更改导航系统中使用的算法**。可能更建议使用带有指定算法的条件节点来创建不同的子树上下文，以获得最有用和独特的情况。但是，选择器节点可以是从外部应用程序而不是通过内部行为树控制流逻辑更改算法的有用方法。最好通过行为树方法实现更改，但是我们了解**许多专业用户都有外部应用程序来动态更改其导航器的设置**。

> [!NOTE]
> .**动态更改算法**，或者这个也是一种具体的应用场景
> 区别于动态参数管理

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <RecoveryNode number_of_retries="6" name="NavigateRecovery">
      <PipelineSequence name="NavigateWithReplanning">
        <RateController hz="0.333">
          <RecoveryNode number_of_retries="1" name="ComputePathThroughPoses">
            <ReactiveSequence>
              <RemovePassedGoals input_goals="{goals}" output_goals="{goals}" radius="0.5"/>
              <ComputePathThroughPoses goals="{goals}" path="{path}" planner_id="GridBased"/>
            </ReactiveSequence>
            <ReactiveFallback name="ComputePathThroughPosesRecoveryFallback">
              <GoalUpdated/>
              <ClearEntireCostmap name="ClearGlobalCostmap-Context" service_name="global_costmap/clear_entirely_global_costmap"/>
            </ReactiveFallback>
          </RecoveryNode>
        </RateController>
        <RecoveryNode number_of_retries="1" name="FollowPath">
          <FollowPath path="{path}" controller_id="FollowPath"/>
          <ReactiveFallback name="FollowPathRecoveryFallback">
            <GoalUpdated/>
            <ClearEntireCostmap name="ClearLocalCostmap-Context" service_name="local_costmap/clear_entirely_local_costmap"/>
          </ReactiveFallback>
        </RecoveryNode>
      </PipelineSequence>
      <ReactiveFallback name="RecoveryFallback">
        <GoalUpdated/>
        <RoundRobin name="RecoveryActions">
          <Sequence name="ClearingActions">
            <ClearEntireCostmap name="ClearLocalCostmap-Subtree" service_name="local_costmap/clear_entirely_local_costmap"/>
            <ClearEntireCostmap name="ClearGlobalCostmap-Subtree" service_name="global_costmap/clear_entirely_global_costmap"/>
          </Sequence>
          <Spin spin_dist="1.57"/>
          <Wait wait_duration="5"/>
          <BackUp backup_dist="0.15" backup_speed="0.025"/>
        </RoundRobin>
      </ReactiveFallback>
    </RecoveryNode>
  </BehaviorTree>
</root>
```
