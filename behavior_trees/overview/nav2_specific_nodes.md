---
tip: translate by openai@2023-06-02 11:32:46
title: Introduction To Nav2 Specific Nodes
---

::: warning
::: title
Warning
:::

Vocabulary can be a large point of confusion here when first starting out. - A `Node` when discussing BT is entirely different than a `Node` in the ROS2 context

- An `ActionNode` in the context of BT is not necessarily connected to an Action Server in the ROS2 context (but often it is)

> 词汇可能是初学者最容易混淆的一大要点。**当讨论 BT 时，“节点”与 ROS2 环境中的“节点”完全不同**。
> 在 BT 的上下文中，**一个 ActionNode 不一定要连接到 ROS2 上的 Action Server(但通常是这样)**

> [!NOTE]
> 这个就是之前看到的，是从真正的行为上定义的 action。

:::

There are quite a few custom Nav2 BT nodes that are provided to be used in the Nav2 specific fashion. Some commonly used Nav2 nodes will be described below. The full list of custom BT nodes can be found in the [nav2_behavior_tree plugins folder](https://github.com/ros-planning/navigation2/tree/main/nav2_behavior_tree/plugins). The [configuration guide](../../configuration/packages/configuring-bt-xml.html) can also be quite useful.

> 有相当多的自定义 Nav2 BT 节点可供以 Nav2 特定方式使用。下面将描述一些常用的 Nav2 节点。可以在[nav2_behavior_tree plugins folder]中找到完整的自定义 BT 节点列表。[配置指南]也可能非常有用。

# Action Nodes

- `ComputePathToPose` - ComputePathToPose Action Server Client (Planner Interface)
- `FollowPath` - FollowPath Action Server Client (Controller Interface)
- `Spin, Wait, Backup` - Behaviors Action Server Client
- `ClearCostmapService` - ClearCostmapService Server Clients

Upon completion, these action nodes will return `SUCCESS` if the action server believes the action has been completed correctly, `RUNNING` when still running, and will return `FAILURE` otherwise. Note that in the above list, the [ClearCostmapService]{.title-ref} action node is _not_ an action server client, but a service client.

> 完成后，如果动作服务器认为动作已经正确完成，这些动作节点将返回`SUCCESS`；如果仍在运行，将返回`RUNNING`；否则将返回`FAILURE`。请注意，在上面的列表中，**[ClearCostmapService]动作节点不是动作服务器客户端，而是服务客户端。**

# Condition Nodes

- `GoalUpdated` - Checks if the goal on the goal topic has been updated
- `GoalReached` - Checks if the goal has been reached
- `InitialPoseReceived` - Checks to see if a pose on the `intial_pose` topic has been received
- `isBatteryLow` - Checks to see if the battery is low by listening on the battery topic

The above list of condition nodes can be used to probe particular aspects of the system. Typically they will return `SUCCESS` is `TRUE` and `FAILURE` when `FALSE`. The key condition that is used in the default Nav2 BT is `GoalUpdated` which is checked asynchronously within particular subtrees. This condition node allows for the behavior described as \"If the goal has updated, then we must replan\". Condition nodes are typically paired with `ReactiveFallback` nodes.

> 以上**条件节点列表可用于探查系统的特定方面**。通常，如果为真，它们会返回“成功”，如果为假，则会返回“失败”。默认 Nav2 BT 中使用的关键条件是“GoalUpdated”，它**在特定的子树中异步检查**。此条件节点允许描述为 **“如果目标已更新，则必须重新计划”** 的行为。条件节点通常与`ReactiveFallback`配对。

# Decorator Nodes

- `Distance Controller` - Will tick children nodes every time the robot has traveled a certain distance
- `Rate Controller` - Controls the ticking of it\'s child node at a constant frequency. The tick rate is an exposed port
- `Goal Update`r - Will update the goal of children nodes via ports on the BT
- `Single Trigger` - Will only tick it\'s child node once, and will return `FAILURE` for all subsequent ticks
- `Speed Controller` - Controls the ticking of it\'s child node at a rate proportional to the robot\'s speed

> - 距离控制器 - 每次机器人行驶一定距离时，将会计时子节点。
> - 速率控制器 - 以恒定频率控制其子节点的 tick 声。tick 率是一个公开的端口。
> - 目标更新器 - 将通过 BT 上的端口更新子节点的目标
> - 单触发 - 只会触发它的子节点**一次**，对于所有后续的触发都会返回“失败”。
> - 速度控制器 - 根据机器人的速度比例控制其子节点的 tick 声。

# Control: PipelineSequence

> [!NOTE]
> 感觉这里在“树”的形式上定义了丰富的“行为”

The `PipelineSequence` control node re-ticks previous children when a child returns `RUNNING`. This node is similar to the `Sequence` node, with the additional property that the children prior to the \"current\" are re-ticked, (resembling the flow of water in a pipe). If at any point a child returns `FAILURE`, all children will be halted and the parent node will also return `FAILURE`. Upon `SUCCESS` of the **last node** in the sequence, this node will halt and return `SUCCESS`.

> `PipelineSequence`控制节点在子节点返回`RUNNING`时会**重新触发前面的子节点**。该节点类似于`Sequence`节点，但具有额外的属性，即在“当前”之前的子节点将被重新触发(类似水流在管道中流动)。如果任何子节点返回`FAILURE`，所有子节点都将被停止，父节点也将返回`FAILURE`。在序列中最后一个节点**成功**完成后，此节点将停止并返回`SUCCESS`。

To explain this further, here is an example BT that uses PipelineSequence.

> ![image](../images/control_pipelineSequence.png)

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <PipelineSequence>
      <Action_A/>
      <Action_B/>
      <Action_C/>
    </PipelineSequence>
  </BehaviorTree>
</root>
```

1.  `Action_A`, `Action_B`, and `Action_C` are all `IDLE`.
2.  When the parent PipelineSequence is first ticked, let\'s assume `Action_A` returns `RUNNING`. The parent node will now return `RUNNING` and no other nodes are ticked.

> ![image](../images/control_pipelineSequence_RUNNING_IDLE_IDLE.png)

3.  Now, let\'s assume `Action_A` returns `SUCCESS`, `Action_B` will now get ticked and will return `RUNNING`. `Action_C` has not yet been ticked so will return `IDLE`.

> ![image](../images/control_pipelineSequence_SUCCESS_RUNNING_IDLE.png)

4.  `Action_A` gets ticked again and returns `RUNNING`, and `Action_B` gets re-ticked and returns `SUCCESS` and therefore the BT goes on to tick `Action_C` for the first time. Let\'s assume `Action_C` returns `RUNNING`. The retick-ing of `Action_A` is what makes PipelineSequence useful.

> ![image](../images/control_pipelineSequence_RUNNING_SUCCESS_RUNNING.png)

5.  All actions in the sequence will be re-ticked. Let\'s assume `Action_A` still returns `RUNNING`, where as `Action_B` returns `SUCCESS` again, and `Action_C` now returns `SUCCESS` on this tick. The sequence is now complete, and therefore `Action_A` is halted, even though it was still `RUNNING`.

> ![image](../images/control_pipelineSequence_RUNNING_SUCCESS_SUCCESS.png)

> 1. `Action_A`、`Action_B` 和 `Action_C` 都处于空闲状态。
> 2. 当父管道序列首次被 tick 时，假设`Action_A`返回`RUNNING`。父节点现在将返回`RUNNING`，并且没有其他节点被 tick。
> 3. 现在，假设`Action_A`返回`SUCCESS`，`Action_B`将会被打勾并返回`RUNNING`。`Action_C`尚未被打勾，因此将返回`IDLE`。
> 4. 当`Action_A`被重新标记并返回`RUNNING`，而`Action_B`被重新标记并返回`SUCCESS`，因此 BT 会继续第一次标记`Action_C`。假设`Action_C`返回`RUNNING`。**重新标记`Action_A`正是使 PipelineSequence 变得有用的原因**。
> 5. **所有动作在序列中都会重新被检查**。假设`Action_A`仍然返回`RUNNING`，而`Action_B`再次返回`SUCCESS`，而`Action_C`在这次 tick 中返回`SUCCESS`。序列现在完成了，因此`Action_A`被停止，尽管它仍然是`RUNNING`。

Recall that if `Action_A`, `Action_B`, or `Action_C` returned `FAILURE` at any point of time, the parent would have returned `FAILURE` and halted any children as well.

> 如果 `Action_A`、`Action_B` 或 `Action_C` 在任何时间点返回 **FAILURE，父级将返回 FAILURE 并停止任何子级**。

For additional details regarding the `PipelineSequence` please see the [PipelineSequence configuration guide](../../configuration/packages/bt-plugins/controls/PipelineSequence.html).

> 关于`PipelineSequence`的更多细节，请参阅[PipelineSequence 配置指南]。

# Control: Recovery

The Recovery control node has only two children and returns `SUCCESS` if and only if the first child returns `SUCCESS`. If the first child returns `FAILURE`, the second child will be ticked. This loop will continue until either:

> **恢复控制节点**只有两个子节点，只有当第一个子节点返回`SUCCESS`时才会返回`SUCCESS`。如果**第一个子节点返回`FAILURE`，则会激活第二个子节点**。此循环将持续直到：

- The first child returns `SUCCESS` (which results in `SUCCESS` of the parent node)
- The second child returns `FAILURE` (which results in `FAILURE` of the parent node)
- The `number_of_retries` input parameter is violated

> - 第一个子返回“SUCCESS”（导致父节点的“SUCCESS'”）
> - 第二个子返回“FAILURE”（导致父节点的“FAILUE”）
> - 违反了“number_of_reries”输入参数

This node is usually used to link together an action, and a recovery action as the name suggests. The first action will typically be the \"main\" behavior, and the second action will be something to be done in case of `FAILURE` of the main behavior. Often, the ticking of the second child action will promote the chance the first action will succeed.

> 这个节点**通常用于将一个动作(action)和一个恢复动作(recovery action)连接在一起**，正如其名称所暗示的那样。第一个动作通常是“主要”行为，**第二个动作将是在主行为失败时要做的事情**。通常，第二个子动作的触发会**增加第一个动作成功的机会**。

> ![image](../images/control_recovery_node.png)

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <RecoveryNode number_of_retries="1">
      <ComputePathToPose/>
      <ClearLocalCostmap/>
    </RecoveryNode>
  </BehaviorTree>
</root>
```

In the above example, let\'s assume `ComputePathToPose` fails. `ClearLocalCostmap` will be ticked in response, and return `SUCCESS`. Now that we have cleared the costmap, lets\' say the robot is correctly able to compute the path and `ComputePathToPose` now returns `SUCCESS`. Then, the parent RecoveryNode will also return `SUCCESS` and the BT will be complete.

> 在上面的例子中，**假设`ComputePathToPose`失败了。回应地，`ClearLocalCostmap`会被勾选，并且返回`SUCCESS`**。现在我们已经清除了成本地图，假设机器人能够正确地计算路径，并且`ComputePathToPose`现在返回`SUCCESS`。然后，父级 RecoveryNode 也会返回`SUCCESS`，BT 也完成了。

For additional details regarding the `RecoveryNode` please see the [RecoveryNode configuration guide](../../configuration/packages/bt-plugins/controls/RecoveryNode.md).

> 关于`RecoveryNode`的更多细节，请参见[RecoveryNode 配置指南]。

# Control: RoundRobin

The RoundRobin control node ticks it\'s children in a round robin fashion until a child returns `SUCCESS`, in which the parent node will also return `SUCCESS`. If all children return `FAILURE` so will the parent RoundRobin.

> 轮询控制节点以轮询方式对其子节点进行 tick，**直到一个子节点返回“成功”，父节点也会返回“成功”。如果所有子节点都返回“失败”，父轮询也会失败。**

Here is an example BT we will use to walk through the concept.

> ![image](../images/control_round_robin.png)

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <RoundRobin>
      <Action_A/>
      <Action_B/>
      <Action_C/>
    </RoundRobin>
  </BehaviorTree>
</root>
```

1.  All the nodes start at `IDLE`

> ![image](../images/control_round_robin_IDLE_IDLE_IDLE.png)

1. Upon tick of the parent node, the first child (`Action_A`) is ticked. Let\'s assume on tick the child returns `RUNNING`. In this case, no other children are ticked and the parent node returns `RUNNING` as well.

> ![image](../images/control_round_robin_RUNNING_IDLE_IDLE.png)

1. Upon the next tick, let\'s assume that `Action_A` returns `FAILURE`. This means that `Action_B` will get ticked next, and `Action_C` remains unticked. Let\'s assume `Action_B` returns `RUNNING` this time. That means the parent RoundRobin node will also return `RUNNING`.

> ![image](../images/control_round_robin_FAILURE_RUNNING_IDLE.png)

1. Upon this next tick, let\'s assume that `Action_B` returns `SUCCESS`. The parent RoundRobin will now halt all children and returns `SUCCESS`. The parent node retains this state information, and will tick `Action_C` upon the next tick rather than start from `Action_A` like Step 2 did.

> ![image](../images/control_round_robin_FAILURE_SUCCESS_IDLE.png)

1.  On this tick, let\'s assume `Action_C` returns `RUNNING`, and so does the parent RoundRobin. No other nodes are ticked.

> ![image](../images/control_round_robin_FAILURE_SUCCESS_RUNNING.png)

1.  On this last tick, let\'s assume `Action_C` returns `FAILURE`. The parent will circle and tick `Action_A` again. `Action_A` returns `RUNNING` and so will the parent RoundRobin node. This pattern will continue indefinitely unless all children return `FAILURE`.

> ![image](../images/control_round_robin_RUNNING_IDLE_FAILURE.png)

> 1. 所有节点都从`IDLE`开始
> 1. 当父节点被勾选时，第一个子节点(`Action_A`)也被勾选。假设勾选子节点后返回“RUNNING”，**此时不会勾选其他子节点**，父节点也会返回“RUNNING”。
> 1. 在下一次 tick 之后，**假设`Action_A`返回`失败`。这意味着`Action_B`将被下一个 tick**，而`Action_C`保持 unticked 状态。假设`Action_B`这次返回`RUNNING`。这意味着父 RoundRobin 节点也会返回`RUNNING`。
> 1. 在下一个 tick 时，**假设 `Action_B` 返回 SUCCESS。现在 RoundRobin 父节点将停止所有子节点，并返回 SUCCESS**。父节点保留这个状态信息，**并在下一个 tick 时跳转到 `Action_C`** 而不是像第 2 步那样从 `Action_A` 开始。
> 1. 在这一次 tick 中，假设 `Action_C` 返回 `RUNNING，父` RoundRobin 也是如此。没有其他节点被 tick。
> 1. 在最后一次检查中，**假设 `Action_C` 返回失败。父节点会再次勾选 `Action_A`**。`Action_A` 返回运行中，父节点 RoundRobin 也会返回运行中。除非所有子节点都返回失败，这种模式将持续下去。

> [!NOTE]
> 这里的轮询也是可以做出不同的 `action_*` 每个都有不同的策略，这样轮询去尝试解决故障！

For additional details regarding the `RecoveryNode` please see the [RoundRobin configuration guide](../../configuration/packages/bt-plugins/controls/RoundRobin.html).

> 关于`RecoveryNode`的更多细节，请参见[RoundRobin 配置指南](../../configuration/packages/bt-plugins/controls/RoundRobin.html)。
