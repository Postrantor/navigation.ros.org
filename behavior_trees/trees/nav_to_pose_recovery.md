---
tip: translate by openai@2023-06-02 16:48:47
title: Navigate To Pose
---

This behavior tree implements a significantly more mature version of the behavior tree on `behavior_trees`. It navigates from a starting point to a single point goal in freespace. It contains both use of custom recovery behaviors in specific sub-contexts as well as a global recovery subtree for system-level failures. It also provides the opportunity for users to retry tasks multiple times before returning a failed state.

> 这个行为树实现了`behavior_trees`中更成熟的版本。它可以**在自由空间中从起点到目标点导航**。它既包含在特定子上下文中**使用自定义恢复行为，也包含用于系统级故障的全局恢复子树**。它还为用户提供了多次重试任务的机会，然后才返回失败状态。

The `ComputePathToPose` and `FollowPath` BT nodes both also specify their algorithms to utilize. By convention we name these by the style of algorithms that they are (e.g. not `DWB` but rather `FollowPath`) such that a behavior tree or application developer need not worry about the technical specifics. They just want to use a path following controller.

> `ComputePathToPose`和`FollowPath` BT 节点都指定了它们要使用的算法。按照惯例，我们按照它们所使用的算法类型来命名它们(例如，不是`DWB`而是`FollowPath`)，这样行为树或应用程序开发者就不必担心技术细节。他们只想使用路径跟踪控制器。

In this behavior tree, we attempt to retry the entire navigation task 6 times before returning to the caller that the task has failed. This allows the navigation system ample opportunity to try to recovery from failure conditions or wait for transient issues to pass, such as crowding from people or a temporary sensor failure.

> 在这个行为树中，我们尝试重试整个导航任务 6 次，然后才返回给调用者任务失败。这样，**导航系统就有充足的机会尝试从失败条件中恢复，或者等待暂时性问题过去，比如人群拥挤或暂时传感器失效**。

> [!NOTE]
> 多次尝试的意义

In nominal execution, this will replan the path at every second and pass that path onto the controller, similar to the behavior tree in `behavior_trees`. However, this time, if the planner fails, it will trigger contextually aware recovery behaviors in its subtree, clearing the global costmap. Additional recovery behaviors can be added here for additional context-specific recoveries, such as trying another algorithm.

> 在名义上的执行中，**这将每秒重新规划路径并将该路径传递给控制器，类似于`behavior_trees`中的行为树。但是，这次，如果规划者失败，它将触发其子树中的上下文感知恢复行为，清除全局成本地图。可以在此处添加其他上下文特定恢复行为，例如尝试另一种算法。**

Similarly, the controller has similar logic. If it fails, it also attempts a costmap clearing of the local costmap impacting the controller. It is worth noting the `GoalUpdated` node in the reactive fallback. This allows us to exit recovery conditions when a new goal has been passed to the navigation system through a preemption. This ensures that the navigation system will be very responsive immediately when a new goal is issued, even when the last goal was in an attempted recovery.

> 同样，控制器具有类似的逻辑。如果失败，它还尝试清除影响控制器的本地成本图。值得注意的是反应回退中的`GoalUpdated`节点。这使我们可以在**通过 _抢占_ 发送新目标到导航系统时退出恢复条件**。这确保导航系统在发出新的目标时会立即非常响应，即使上一个目标正在尝试恢复。

> [!NOTE]
> .**通过 _抢占_ 发送新目标到导航系统时退出恢复条件**

If these contextual recoveries fail, this behavior tree enters the recovery subtree. This subtree is reserved for system-level failures to help resolve issues like the robot being stuck or in a bad spot. This subtree also has the `GoalUpdated` BT node it ticks every iteration to ensure responsiveness of new goals. Next, the recovery subtree will the recoveries: costmap clearing operations, spinning, waiting, and backing up. After each of the recoveries in the subtree, the main navigation subtree will be reattempted. If it continues to fail, the next recovery in the recovery subtree is ticked.

> 如果这些**上下文恢复失败，此行为树将进入恢复子树**。该子树保留**用于解决机器人陷入困境或处境不佳等系统级故障的问题**。此子树还具有每次迭代时都会 tick 的`GoalUpdated` BT 节点，以**确保新目标的响应性**。接下来，恢复子树将恢复以下内容：成本图清除操作、旋转、等待和后退。在子树中的每个恢复之后，将重新尝试主导航子树。如果继续失败，则会 tick 恢复子树中的下一个恢复。

While this behavior tree does not make use of it, the `PlannerSelector`, `ControllerSelector`, and `GoalCheckerSelector` behavior tree nodes can also be helpful. Rather than hardcoding the algorithm to use (`GridBased` and `FollowPath`), these behavior tree nodes will allow a user to dynamically change the algorithm used in the navigation system via a ROS topic. It may be instead advisable to create different subtree contexts using condition nodes with specified algorithms in their most useful and unique situations. However, the selector nodes can be a useful way to change algorithms from an external application rather than via internal behavior tree control flow logic. It is better to implement changes through behavior tree methods, but we understand that many professional users have external applications to dynamically change settings of their navigators.

> 这种行为树没有使用它，但是`PlannerSelector`、`ControllerSelector`和`GoalCheckerSelector`行为树节点可能也会有所帮助。与将算法(`GridBased`和`FollowPath`)硬编码不同，这些行为树节点**将允许用户通过 ROS 主题动态更改导航系统中使用的算法**。可能**更建议使用条件节点创建不同的子树上下文，其中包含指定的算法**，在最有用和独特的情况下。但是，**选择器节点可以是从外部应用程序而不是通过内部行为树控制流程逻辑更改算法的一种有用方法。最好通过行为树方法实施更改**，但是我们理解许多专业用户有外部应用程序来动态更改其导航器的设置。

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <RecoveryNode number_of_retries="6" name="NavigateRecovery">
      <PipelineSequence name="NavigateWithReplanning">
        <RateController hz="1.0">
          <RecoveryNode number_of_retries="1" name="ComputePathToPose">
            <ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased"/>
            <ReactiveFallback name="ComputePathToPoseRecoveryFallback">
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
