---
tip: translate by openai@2023-06-02 16:40:23
title: Navigate To Pose With Consistent Replanning And If Path Becomes Invalid
---

This behavior tree implements a significantly more mature version of the behavior tree on `behavior_trees`. It navigates from a starting point to a single point goal in freespace. It contains both use of custom recoveries in specific sub-contexts as well as a global recovery subtree for system-level failures. It also provides the opportunity for users to retry tasks multiple times before returning a failed state.

> 这个行为树实现了`behavior_trees`中更成熟的版本。它在自由空间中从起始点导航到单个目标点。它既包含在特定子上下文中使用自定义恢复，也包含用于系统级故障的全局恢复子树。它还**为用户提供了在返回失败状态之前重试任务的机会**。

The `ComputePathToPose` and `FollowPath` BT nodes both also specify their algorithms to utilize. By convention we name these by the style of algorithms that they are (e.g. not `DWB` but rather `FollowPath`) such that a behavior tree or application developer need not worry about the technical specifics. They just want to use a path following controller.

> `ComputePathToPose`和`FollowPath`BT 节点都指定了它们要使用的算法。按照惯例，我们按照它们使用的算法类型来命名它们(例如不是`DWB`而是`FollowPath`)，这样行为树或应用开发者就不必担心技术细节。他们只想使用路径跟随控制器。

In this behavior tree, we attempt to retry the entire navigation task 6 times before returning to the caller that the task has failed. This allows the navigation system ample opportunity to try to recovery from failure conditions or wait for transient issues to pass, such as crowding from people or a temporary sensor failure.

> 在这个行为树中，我们**尝试重试整个导航任务 6 次，然后再回调调用者，表明任务失败了。这使得导航系统有充足的机会尝试恢复失败的情况**，或者等待暂时性的问题通过，比如人群拥挤或者暂时的传感器失效。

In nominal execution, replanning can be triggered by an a invalid previous path, a new goal or if a new path has not been created for 10 seconds. If the planner or controller fails, it will trigger contextually aware recoveries in its subtree. Currently, the recoveries will clear the global costmap if the planner fails and clear the local costmap if the controller fails. Additional context-specific recoveries can be added to these subtrees.

> 在名义上的执行中，**重新规划可以由无效的先前路径、新的目标或如果 10 秒内没有创建新的路径而被触发**。如果**规划者或控制器失败，它将触发其子树中的上下文感知恢复**。目前，如果规划者失败，恢复将清除全局成本图，如果控制器失败，恢复将清除本地成本图。可以向这些子树添加其他特定于上下文的恢复。

> [!NOTE]
> 这个重试次数还是很有意义，也是 lifecycle 的一个重要价值
> 这里还给出了一些重试的场景！！

If these contextual recoveries fail, this behavior tree enters the recovery subtree. This subtree is reserved for system-level failures to help resolve issues like the robot being stuck or in a bad spot. This subtree has the `GoalUpdated` BT node which ticks every iteration to ensure responsiveness of new goals. Next, the recovery subtree will attempt the following recoveries: costmap clearing operations, spinning, waiting, and backing up. After each of the recoveries in the subtree, the main navigation subtree will be reattempted. If it continues to fail, the next recovery in the recovery subtree is ticked.

> 如果**这些上下文恢复失败，此行为树将进入恢复子树**。此子树保留用于系统级故障的恢复，以帮助**解决机器人被卡住或处于不良位置**等问题。此子树具有每次迭代时都会tick的`GoalUpdated` BT 节点，以确保新目标的响应性。接下来，恢复子树将尝试以下恢复操作：清理成本地图操作、旋转、等待和后退。在子树中的每个恢复操作之后，将重新尝试主导航子树。如果继续失败，则将tick恢复子树中的下一个恢复操作。

While this behavior tree does not make use of it, the `PlannerSelector`, `ControllerSelector`, and `GoalCheckerSelector` behavior tree nodes can also be helpful. Rather than hardcoding the algorithm to use (`GridBased` and `FollowPath`), these behavior tree nodes will allow a user to dynamically change the algorithm used in the navigation system via a ROS topic. It may be instead advisable to create different subtree contexts using condition nodes with specified algorithms in their most useful and unique situations. However, the selector nodes can be a useful way to change algorithms from an external application rather than via internal behavior tree control flow logic. It is better to implement changes through behavior tree methods, but we understand that many professional users have external applications to dynamically change settings of their navigators.

> 虽然这个行为树没有使用它，但`PlannerSelector`、`ControllerSelector`和`GoalCheckerSelector`行为树节点也可能有帮助。与硬编码使用的算法(`GridBased`和`FollowPath`)不同，这些行为树节点将允许用户通过 ROS 主题动态更改导航系统中使用的算法。可能更建议使用条件节点创建不同的子树上下文，其中指定的算法在其最有用和独特的情况下。但是，选择器节点可以是从外部应用程序而不是通过内部行为树控制流逻辑更改算法的有用方法。最好通过行为树方法实施更改，但是我们理解许多专业用户有外部应用程序来动态更改其导航器的设置。

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <RecoveryNode number_of_retries="6" name="NavigateRecovery">
      <PipelineSequence name="NavigateWithReplanning">
    <RateController hz="2.0">
      <RecoveryNode number_of_retries="1" name="ComputePathToPose">
        <Fallback>
          <ReactiveSequence>
            <Inverter>
              <PathExpiringTimer seconds="10" path="{path}"/>
            </Inverter>
            <Inverter>
              <GlobalUpdatedGoal/>
            </Inverter>
            <IsPathValid path="{path}"/>
          </ReactiveSequence>
          <ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased"/>
        </Fallback>
        <ClearEntireCostmap name="ClearGlobalCostmap-Context" service_name="global_costmap/clear_entirely_global_costmap"/>
      </RecoveryNode>
    </RateController>
    <RecoveryNode number_of_retries="1" name="FollowPath">
      <FollowPath path="{path}" controller_id="FollowPath"/>
      <ClearEntireCostmap name="ClearLocalCostmap-Context" service_name="local_costmap/clear_entirely_local_costmap"/>
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
      <BackUp backup_dist="0.30" backup_speed="0.05"/>
    </RoundRobin>
      </ReactiveFallback>
    </RecoveryNode>
  </BehaviorTree>
</root>
```
