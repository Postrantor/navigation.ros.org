---
tip: translate by openai@2023-06-02 11:19:03
title: Lifecycle Manager
---

Source code on [Github](https://github.com/ros-planning/navigation2/tree/main/nav2_lifecycle_manager).

The Lifecycle Manager module implements the method for handling the lifecycle transition states for the stack in a deterministic way. It will take in a set of ordered nodes to transition one-by-one into the configurating and activate states to run the stack. It will then bring down the stack into the finalized state in the opposite order. It will also create bond connections with the servers to ensure they are still up and transition down all nodes if any are non-responsive or crashed.

> 活动期管理模块**实现了以确定性方式处理堆栈生命周期转换状态的方法**。它将**接受一组有序节点，将它们一个接一个地转换到配置和激活状态以运行堆栈**。然后，它将以相反的顺序将堆栈带回最终状态。它还将与服务器建立连接，以确保它们仍然在线，并**在任何节点不可响应或崩溃时将所有节点转换下来**。

> [!NOTE]
> 这里定义的这些参数特备好，可以集成到 beatles 中实现

# Parameters

node_names

:

> +------------------+---------+
> | Type | Default |
> +==================+=========+
> | vector\<string\> | > N/A |
> +------------------+---------+
>
> Description
>
> : Ordered list of node names to bringup through lifecycle transition.
> ：通过生命周期过渡提出的节点名称的订购列表。

autostart

:

> Type Default
>
> ---
>
> bool false
>
> Description
>
> : Whether to transition nodes to active state on startup.
> ：是否要在启动时将节点过渡到活动状态。

bond_timeout

:

> Type Default
>
> ---
>
> double 4.0
>
> Description
>
> : Timeout to transition down all lifecycle nodes of this manager if a server is non-responsive, in seconds. Set to `0` to deactivate. Recommended to be always larger than 0.3s for all-local node discovery. Note: if a server cleanly exits the manager will immediately be notified.
> ：如果服务器在几秒钟内无响应性，则超时降低此管理器的所有生命周期节点。设置为`0`停用。对于全本节点发现，建议始终大于 0.3s。注意：如果服务器干净地退出，将立即通知管理器。

attempt_respawn_reconnection

:

> Type Default
>
> ---
>
> bool true
>
> Description
>
> : Whether to try to reconnect to servers that go down, presumably because respawn is set to `true` to re-create crashed nodes. While default to `true`, reconnections will not be made unless respawn is set to true in your launch files or your watchdog systems will bring up the server externally.
> ：是否试图重新连接到下降的服务器，大概是因为重生设置为“ true”以重新创建崩溃的节点。虽然默认为“ true”，但除非在启动文件中设置为 true，否则将不会进行重新连接，否则您的看门狗系统将在外部启动服务器。

bond_respawn_max_duration

:

> +--------+---------+
> | Type | Default |
> +========+=========+
> | double | > 10.0 |
> +--------+---------+
>
> Description
>
> : When a server crashes or becomes non-responsive, the lifecycle manager will bring down all nodes for safety. This is the duration of which the lifecycle manager will attempt to reconnect with the failed server(s) during to recover and re-activate the system. If this passes, it will stop attempts and will require a manual re-activation once the problem is manually resolved. Units: seconds.
> ：当服务器崩溃或变得无反应时，生命周期管理器将为安全而放下所有节点。这是生命周期管理器将尝试与失败的服务器重新连接以恢复和重新激活系统的持续时间。如果这通过，它将停止尝试，一旦手动解决问题，就需要手动重新激活。单位：秒。

# Example

```yaml
lifecycle_manager:
  ros__parameters:
    autostart: true
    node_names:
      [
        "controller_server",
        "planner_server",
        "behavior_server",
        "bt_navigator",
        "waypoint_follower",
      ]
    bond_timeout: 4.0
    attempt_respawn_reconnection: true
    bond_respawn_max_duration: 10.0
```
