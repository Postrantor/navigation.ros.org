---
tip: translate by openai@2023-06-02 11:16:13
title: Detailed Behavior Tree Walkthrough
---

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Navigate To Pose With Replanning and Recovery](#navigate-to-pose-with-replanning-and-recovery)
- [Navigation Subtree](#navigation-subtree)
- [Recovery Subtree](#recovery-subtree)

# Overview

This document serves as a reference guide to the main behavior tree (BT) used in Nav2.

> 这份文件是 Nav2 中主要行为树(BT)的参考指南。

There are many example behavior trees provided in `nav2_bt_navigator/behavior_trees`, but these sometimes have to be re-configured based on the application of the robot. The following document will walk through the current main default BT `navigate_to_pose_w_replanning_and_recovery.xml` in great detail.

> 在`nav2_bt_navigator/behavior_trees`中提供了许多示例行为树，但**有时需要根据机器人的应用进行重新配置**。以下文档将详细介绍当前主要默认 BT `navigate_to_pose_w_replanning_and_recovery.xml`。

# Prerequisites

- Become familiar with the concept of a behavior tree before continuing with this walkthrough

  - Read the short explanation in [navigation concepts](../../concepts/index.html)

    > - 阅读[导航概念](../../concepts/index.html)中的简短解释。

  - Read the general tutorial and guide (not Nav2 specific) on the [BehaviorTree CPP V3](https://www.behaviortree.dev/) website. Specifically, the \"Learn the Basics\" section on the BehaviorTree CPP V3 website explains the basic generic nodes that will be used that this guide will build upon.
    > - 请在[BehaviorTree CPP V3](https://www.behaviortree.dev/)网站上阅读通用教程和指南(不是 Nav2 特定的)。具体而言，BehaviorTree CPP V3 网站上的“学习基础”部分解释了本指南将使用的基本通用节点。

- Become familiar with the custom [Nav2 specific BT nodes](nav2_specific_nodes.html)

# Navigate To Pose With Replanning and Recovery

The following section will describe in detail the concept of the main and default BT currently used in Nav2, `navigate_to_pose_w_replanning_and_recovery.xml`. This behavior tree replans the global path periodically at 1 Hz and it also has recovery actions.

> 以下部分将详细描述 Nav2 中当前使用的主要和默认 BT，即`navigate_to_pose_w_replanning_and_recovery.xml`。**这个行为树每秒定期重新计划全局路径，并具有恢复操作**。

> ![image](../images/walkthrough/overall_bt.png)

BTs are primarily defined in XML. The tree shown above is represented in XML as follows.

> BTs 主要是用 XML 定义的。上面的树用 XML 表示如下。

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

This is likely still a bit overwhelming, but this tree can be broken into two smaller subtrees that we can focus on one at a time. These smaller subtrees are the children of the top-most `RecoveryNode`. From this point forward the `NavigateWithReplanning` subtree will be referred to as the `Navigation` subtree, and the `RecoveryFallback` subtree will be known as the `Recovery` subtree. This can be represented in the following way:

> 这可能仍然有点压倒性，但可以将这棵树**分解为两个较小的子树**，我们可以一次处理一个。这些较小的子树是最顶部`RecoveryNode`的子节点。从这一点开始，`NavigateWithReplanning`子树将被称为`Navigation`子树，而`RecoveryFallback`子树将被称为`Recovery`子树。这可以用以下方式表示：

> ![image](../images/walkthrough/overall_bt_w_breakdown.png)

The `Navigation` subtree mainly involves actual navigation behavior:

> `Navigation`子树主要涉及实际的导航行为：

- **calculating a path**
- **following a path**
- **contextual recovery** behaviors for each of the above primary navigation behaviors

The `Recovery` subtree includes behaviors for system level failures or items that were not easily dealt with internally.

The overall BT will (hopefully) spend most of its time in the `Navigation` subtree. If either of the two main behaviors in the `Navigation` subtree fail (path calculation or path following), contextual recoveries will be attempted.

If the contextual recoveries were still not enough, the `Navigation` subtree will return `FAILURE`. The system will move on to the `Recovery` subtree to attempt to clear any system level navigation failures.

This happens until the `number_of_retries` for the parent `RecoveryNode` is exceeded (which by default is 6).

> `Recovery` 子树包括用于**系统级别故障**或无法轻松处理的项目的行为。
> 总体来说，BT(希望)将大部分时间花在`Navigation`子树上。**如果`Navigation`子树中的两个主要行为(路径规划(path calculation)或轨迹跟踪(path following))失败，将尝试上下文恢复(contextual recoveries)。**
> 如果**上下文恢复还不够，`Navigation`子树将返回`FAILURE`。** 系统将转到`Recovery`子树尝试**清除任何系统级导航故障**。
> 这会持续发生，直到父节点`RecoveryNode`的`number_of_retries`被超过(默认是 6 次)。

```xml
<RecoveryNode number_of_retries="6" name="NavigateRecovery">
```

> [!NOTE]
> 这里面涉及 `recovery` 以及 `fallback` 的概念。
> 这里还对 recovery 设置了次数限制，如`number_of_retries`
>
> ---
>
> 进一步了解，在 导航 这棵树中也存在一定的恢复行为，即上下文恢复
> 在真正的右边的 恢复 子树中是系统级别的恢复，即前面的恢复的粒度已经不够了
> 从这点来看，和之前看到的“冗余服务”相关的架构策略还是挺符合的
> 同时，还设置有恢复尝试的次数，超级符合。

# Navigation Subtree

Now that we have gone over the control flow between the `Navigation` subtree and the `Recovery` subtree, let\'s focus on the Navigation subtree.

> 现在我们已经概述了`Navigation`子树和`Recovery`子树之间的控制流，让我们重点关注导航子树。

> ![image](../images/walkthrough/navigation_subtree.png)

The XML of this subtree is as follows:

```xml
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
```

This subtree has two primary actions `ComputePathToPose` and `FollowPath`. If either of these two actions fail, they will attempt to clear the failure contextually. The crux of the tree can be represented with only one parent and two children nodes like this:

> 这个子树有两个主要操作`ComputePathToPose`和`FollowPath`。如果这两个操作中的任何一个失败，它们将尝试以上下文清除失败。树的核心可以用一个父节点和两个子节点表示：

> ![image](../images/walkthrough/navigation_subtree_bare.png)

The parent `PipelineSequence` node allows the `ComputePathToPose` to be ticked, and once that succeeds, `FollowPath` to be ticked. While the `FollowPath` subtree is being ticked, the `ComputePathToPose` subtree will be ticked as well. This allows for the path to be recomputed as the robot moves around.

Both the `ComputePathToPose` and the `FollowPath` follow the same general structure.

> 父节点`PipelineSequence`允许
>
> - **`ComputePathToPose`被勾选，一旦成功，`FollowPath`也会被勾选**。
> - 当`FollowPath`子树被勾选时，`ComputePathToPose`子树也会被勾选。
>
> **这样就可以在机器人移动时重新计算路径**。两个`ComputePathToPose`和`FollowPath`遵循相同的一般结构。

- Do the action
- **If the action fails, try to see if we can contextually recover**

The below is the `ComputePathToPose` subtree:

> ![image](../images/walkthrough/contextual_recoveries.png)

The parent `RecoveryNode` controls the flow between the action, and the contextual recovery subtree. The contextual recoveries for both `ComputePathToPose` and `FollowPath` involve checking if the goal has been updated, and involves clearing the relevant costmap.

> **父`RecoveryNode`控制行动和上下文恢复子树之间的流动**。对于`ComputePathToPose`和`FollowPath`，上下文恢复都涉及检查目标是否已更新，并涉及清除相关的代价地图。

Consider changing the `number_of_retries` parameter in the parent `RecoveryNode` control node if your application can tolerate more attempts at contextual recoveries before moving on to system-level recoveries.

> 考虑更改父节点`RecoveryNode`中的`number_of_retries`参数，如果您的应用程序可以容忍**更多的上下文恢复尝试，然后才进行系统级恢复**。

The only differences in the BT subtree of `ComputePathToPose` and `FollowPath` are outlined below:

> 以下是`ComputePathToPose`和`FollowPath`的 BT 子树之间唯一的差异：

- The action node in the subtree:

  : - The `ComputePathToPose` subtree centers around the `ComputePathToPose` action.
  -The `FollowPath` subtree centers around the `FollowPath` action.

  > - `ComputePathToPose` 子树以`ComputePathToPose`动作为中心。
  > - `FollowPath`子树以`FollowPath`动作为中心。

- The `RateController` that decorates the `ComputePathToPose` subtree

  : The `RateController` decorates the `ComputePathToPose` subtree to keep planning at the specified frequency. The default frequency for this BT is 1 hz. This is done to prevent the BT from flooding the planning server with too many useless requests at the tree update rate (100Hz). Consider changing this frequency to something higher or lower depending on the application and the computational cost of calculating the path. There are other decorators that can be used instead of the `RateController`. Consider using the `SpeedController` or `DistanceController` decorators if appropriate.

  > `RateController`装饰`ComputePathToPose`子树，以保持指定频率的规划。此 BT 的默认频率为 1 Hz。这是为了**防止 BT 以树更新率(100Hz)向规划服务器发送太多无用请求**。考虑根据应用程序和计算路径成本更改此频率为更高或更低的值。**还有其他装饰器可以代替`RateController`。如果适用，考虑使用`SpeedController`或`DistanceController`装饰器**。

- The costmap that is being cleared within the contextual recovery:

  : - The `ComputePathToPose` subtree clears the global costmap. The global costmap is the relevant costmap in the context of the planner
  -The `FollowPath` subtree clears the local costmap. The local costmap is the relevant costmap in the context of the controller

  > -`ComputePathToPose`子树**清除全局代价地图**。全局代价地图是**规划器上下文中的**相关代价地图 -`FollowPath`子树清除**本地代价地图**。本地代价地图是**控制器上下文中的**相关代价地图。

# Recovery Subtree

The `Recovery` subtree is the second big \"half\" of the Nav2 default `navigate_to_pose_w_replanning_and_recovery.xml` tree. In short, this subtree is triggered when the `Navigation` subtree returns `FAILURE` controls the recoveries at the system level (in the case the contextual recoveries in the `Navigation` subtree were not sufficient).

> `Recovery`子树是 Nav2 默认的`navigate_to_pose_w_replanning_and_recovery.xml`树的第二大"半部分"。简而言之，**当`Navigation`子树返回`FAILURE`时，此子树被触发，它控制系统级恢复(在`Navigation`子树中的情境恢复不足的情况下)**。

> ![image](../images/walkthrough/recovery_subtree.png)

And the XML snippet:

```xml
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
```

The top most parent, `ReactiveFallback` controls the flow between the rest of the system wide recoveries, and asynchronously checking if a new goal has been received. If at any point the goal gets updated, this subtree will halt all children and return `SUCCESS`. This allows for quick reactions to new goals and preempt currently executing recoveries. This should look familiar to the contextual recovery portions of the `Navigation` subtree. This is a common BT pattern to handle the situation \"Unless \'this condition\' happens, Do action A\".

> **最高父级`ReactiveFallback`控制系统范围内恢复的流程，并异步检查是否收到新的目标**。如果**在任何时候目标更新，此子树将停止所有子级并返回`SUCCESS`。这样可以快速反应新的目标并优先执行当前正在执行的恢复操作**。这应该看起来与`Navigation`子树的上下文恢复部分相似。这是一种常见的 BT 模式，用于处理 **“除非发生'此条件'，否则执行操作 A”**的情况。

> [!NOTE]
> 控制恢复流程！
> Round Robin 是一种常用的调度算法，用于分配计算资源。通常，Round Robin 调度器在多个任务之间轮流切换，在每一轮中，每个任务都会得到一定的 CPU 时间片，然后被暂停执行，进而切换到下一个任务上。这样，就可以平等地分配计算资源，保证每个任务都可以得到一定的 CPU 时间。Round Robin 调度算法常常被用于多任务操作系统中，以保证每个任务都可以得到一定的处理机资源，从而实现公平性和效率性的平衡。

These condition nodes can be extremely powerful and are typically paired with `ReactiveFallback`. It can be easy to imagine wrapping this whole `navigate_to_pose_w_replanning_and_recovery` tree in a `ReactiveFallback` with a `isBatteryLow` condition \-- meaning the `navigate_to_pose_w_replanning_and_recovery` tree will execute _unless_ the battery becomes low (and then entire a different subtree for docking to recharge).

> 这些条件节点可以非常强大，通常配合`ReactiveFallback`使用。可以很容易地想象将整个`navigate_to_pose_w_replanning_and_recovery`树包裹在`ReactiveFallback`中，其中包含一个`isBatteryLow`条件--这意味着除非电池电量低(然后进入另一个用于充电的子树)，否则`navigate_to_pose_w_replanning_and_recovery`树将执行。

If the goal is never updated, the behavior tree will go on to the `RoundRobin` node. These are the default four system-level recoveries in the BT are:

> 如果目标从未更新，行为树将继续到`RoundRobin`节点。BT 中默认的**四个系统级恢复**是：

- A sequence that **clears both costmaps** (local, and global)
- `Spin` action
- `Wait` action
- `BackUp` action

Upon `SUCCESS` of any of the four children of the parent `RoundRobin`, the robot will attempt to renavigate in the `Navigation` subtree. If this renavigation was not successful, the next child of the `RoundRobin` will be ticked.

> 如果父节点`RoundRobin`的**四个子节点中任意一个成功，机器人将尝试在`Navigation`子树中重新导航**。如果这次重新导航**不成功，则会轮到`RoundRobin`的下一个子节点**。

For example, let\'s say the robot is stuck and the `Navigation` subtree returns `FAILURE`: (for the sake of this example, let\'s assume that the goal is never updated).

> 举个例子，**假设机器人卡住了，而`Navigation`子树返回`FAILURE`**：(为了这个例子，**假设目标永远不会更新**)。

1. The Costmap clearing sequence in the `Recovery` subtree is attempted, and returns `SUCCESS`. The robot now moves to `Navigation` subtree again
2. Let\'s assume that clearing both costmaps was not sufficient, and the `Navigation` subtree returns `FAILURE` once again. The robot now ticks the `Recovery` subtree
3. In the `Recovery` subtree, the `Spin` action will be ticked. If this returns `SUCCESS`, then the robot will return to the main `Navigation` subtree _BUT_ let\'s assume that the `Spin` action returns `FAILURE`. In this case, the tree will _remain_ in the `Recovery` subtree
4. Let\'s say the next action, `Wait` returns `SUCCESS`. The robot will then move on to the `Navigation` subtree
5. Assume the `Navigation` subtree returns `FAILURE` (clearing the costmaps, attempting a spin, and waiting were _still_ not sufficient to recover the system. The robot will move onto the `Recovery` subtree and attempt the `BackUp` action. Let\'s say that the robot attempts the `BackUp` action and was able to successfully complete the action. The `BackUp` action node returns `SUCCESS` and so now we move on to the Navigation subtree again.
6. In this hypothetical scenario, let\'s assume that the `BackUp` action allowed the robot to successfully navigate in the `Navigation` subtree, and the robot reaches the goal. In this case, the overall BT will still return `SUCCESS`.

> 1. 在`Recovery`子树中尝试 Costmap**清理序列，并返回`SUCCESS`。机器人现在再次移动到`Navigation`子树**。
> 2. 假设清除两个 costmaps 都不够，`Navigation`子树再次返回`FAILURE`。机器人现在按`Recovery`子树计时。
> 3. 在`Navigation`子树中，`Spin`动作将被选中。如果返回`SUCCESS`，那么机器人将返回到主要的`Navigation`子树，但是**假设`Spin`动作返回`FAILURE`。在这种情况下，树将 _保持_ 在`Navigation`子树中**。
> 4. **如果下一个动作 `Wait` 返回 `SUCCESS`，机器人就会继续进入 `Navigation` 子树**。
> 5. 假设`Navigation`子树返回`FAILURE`（清除代价地图，尝试旋转，等待仍然不足以恢复系统）。机器人将**移动到`Recovery`子树，尝试`BackUp`动作**。假设机器人尝试了`BackUp`动作并且能够成功完成此操作。**`BackUp`动作节点返回`SUCCESS`，因此现在我们再次移动到导航子树**。
> 6. 在这个假设的情景中，假设`BackUp`动作使机器人成功地在`Navigation`子树中导航，机器人到达目标。在这种情况下，整个 BT 仍将返回`SUCCESS`。

If the `BackUp` action was not sufficient enough to allow the robot to become un-stuck, the above logic will go on indefinitely until the `number_of_retries` in the parent of the `Navigate` subtree and `Recovery` subtree is exceeded, or if all the system-wide recoveries in the `Recovery` subtree return `FAILURE` (this is unlikely, and likely points to some other system failure).

> 如果`BackUp`动作**不足以让机器人解除卡住**，上述逻辑将无限期地继续，**直到`Navigate`子树和`Recovery`子树的`number_of_retries`超出**，或者`Recovery`子树中的所有系统恢复返回`FAILURE`(这是不太可能的，并可能指向其他系统故障)。
