---
title: RecoveryNode
---

The RecoveryNode is a control flow node with two children. It returns SUCCESS if and only if the first child returns SUCCESS. The second child will be executed only if the first child returns FAILURE. If the second child SUCCEEDS, then the first child will be executed again. The user can specify how many times the recovery actions should be taken before returning FAILURE. In nav2, the RecoveryNode is included in Behavior Trees to implement recovery actions upon failures.

> `RecoveryNode` 是一个具有两个子节点的控制流节点。
>
> - 只有当第一个子节点返回 SUCCESS 时，它才会返回 SUCCESS。
> - **只有在第一个子节点返回 FAILURE 时，才会执行第二个子节点**。
> - 如果**第二个子节点成功，那么会重新执行第一个子节点**。
>
> 用户可以指定在返回 FAILURE 之前，**恢复动作应该重复执行多少次**。在 nav2 中，**RecoveryNode 包含在行为树中**，用于在失败时执行恢复操作。

# Input Ports

number_of_retries

:

> Type Default
>
> ---
>
> int 1
>
> Description
>
> : Number of retries.

# Example

```xml
<RecoveryNode number_of_retries="1">
    <!--Add tree components here--->
</RecoveryNode>
```
