---
tip: translate by openai@2023-06-02 13:46:03
title: Adding a New Nav2 Task Server
---

A nav2 task server consists of server side logic to complete different types of requests, usually called by the autonomy system or through the Behavior Tree Navigator. In this guide, we will discuss the core components needed to add a new task server to Nav2 (ex. Controller, Behavior, Smoother, Planner Servers). Namely, how to set up your new Lifecycle-Component Node for launch and state management and the communication of semantically meaningful error codes (if necessary).

> 一个 Nav2 任务服务器由服务器端逻辑组成，可以完成不同类型的请求，通常由自主系统或行为树导航器调用。在本指南中，我们将讨论添加新任务服务器到 Nav2 所需的核心组件（例如控制器、行为、平滑器和规划服务器）。也就是说，如何设置新的生命周期组件节点以进行启动和状态管理，以及语义上有意义的错误码的通信（如果需要）。

While this tutorial does not cover how to add the complementary Behavior Tree Node to interact with this new Task Server, that is covered at length in `writing_new_nbt_plugin` so this Task Server can be invoked in the BTs in BT Navigator.

> 这个教程不涵盖如何添加补充的行为树节点来与这个新的任务服务器交互，这在`writing_new_nbt_plugin`中有详细介绍，因此可以在 BT Navigator 中的 BTs 中调用这个任务服务器。

If you\'ve created a new Task Server that may have general reuse for the community, consider contacting the maintainers to add it to the Nav2 project! Nav2 gets better by contributions by users like you!

> 如果您创建了一个可能具有普遍重用性的新任务服务器，请考虑联系维护人员将其添加到 Nav2 项目中！Nav2 通过像您这样的用户的贡献变得更好！

# Lifecycle Nodes

The Lifecycle node is the first key component of a nav2 task server. Lifecycle nodes were introduced in ROS 2 to systematically manage the bringup and shutdown of the different nodes involved in the robot\'s operation. The use of Lifecycle nodes ensures that all nodes are successfully instantiated before they begin their execution and Nav2 shuts down all nodes if there is any unresponsive node.

> 生命周期节点是 Nav2 任务服务器的第一个关键组件。**在 ROS 2 中引入了生命周期节点，以系统地管理机器人操作中涉及的不同节点的启动和关闭。使用生命周期节点可确保在开始执行之前成功实例化所有节点，如果有任何不可响应的节点，Nav2 将关闭所有节点。**

Lifecycle nodes contain state machine transitions that enable deterministic behavior in ROS 2 servers. The Lifecycle node transitions in Nav2 are handled by the `Lifecycle Manager`. The Lifecycle Manager transitions the states of the Lifecycle nodes and provides greater control over the state of a system.

> **Lifecycle 节点包含状态机转换，可以在 ROS 2 服务器中实现确定性行为。Nav2 中的 Lifecycle 节点转换由 Lifecycle Manager 处理。Lifecycle Manager 转换 Lifecycle 节点的状态，可以更好地控制系统的状态**。

The primary states of a Lifecycle node are `Unconfigured`, `Inactive`, `Active`, and `Finalized`. A Lifecycle node starts in an `Unconfigured` state after being instantiated. The Lifecycle Manager transitions a node from `Unconfigured` to `Inactive` by implementing the `Configurating` transition. The `Configurating` transition sets up all configuration parameters and prepares any required setup such as memory allocation and the set up of the static publication and subscription topics. A node in the `Inactive` state is allowed to reconfigure its parameters and but cannot perform any processing. From the `Inactive` state, the Lifecyle Manager implements the `Activating` transition state to transition the node from `Inactive` to `Active`, which is the main state. A node in the `Active` state is allowed to perform any processing operation. In case a node crashes, the Lifecycle Manager shuts down the system to prevent any critical failures. On shutdown, the necessary cleanup operations are performed and the nodes are transitioned to the `Finalized` state via `Deactivating`, `CleaningUp`, and `ShuttingDown` transition states.

> 一个生命周期节点的主要状态是`未配置`、`未激活`、`活动`和`完成`。生命周期节点实例化后会处于`未配置`状态。生命周期管理器会通过实施`配置`转换，将节点从`未配置`状态转换为`未激活`状态。`配置`转换会设置所有配置参数，并准备必要的设置，如内存分配和静态发布订阅主题的设置。`未激活`状态下的节点可以重新配置参数，但不能执行任何处理操作。从`未激活`状态，生命周期管理器会通过`激活`转换状态将节点从`未激活`状态转换为`活动`状态，这是主要状态。`活动`状态下的节点可以执行任何处理操作。如果节点崩溃，生命周期管理器会关闭系统以防止任何严重故障。在关闭时，会执行必要的清理操作，并通过`停用`、`清理`和`关闭`转换状态将节点转换为`完成`状态。

::: seealso

For more information on Lifecycle management, see the article on [Managed Nodes](https://design.ros2.org/articles/node_lifecycle.html).

:::

You may wish to integrate your own nodes into the Nav2 framework or add new lifecycle nodes to your system. As an example, we will add a new notional lifecycle node `sensor_driver`, and have it be controlled via the Nav2 Lifecycle Manager to ensure sensor feeds are available before activating navigation. You can do so by adding a `sensor_driver` node in your launch file and adding it to the list of nodes to be activated by the `lifecycle_manager` before navigation, as shown in the example below.

> 您可以将自己的节点集成到 Nav2 框架中，或向系统中添加新的生命周期节点。例如，我们将添加一个新的虚拟生命周期节点“sensor_driver”，并通过 Nav2 生命周期管理器控制，以确保在激活导航之前可以获取传感器信息。您可以通过在启动文件中添加一个“sensor_driver”节点并将其添加到由“lifecycle_manager”激活的节点列表中，如下所示，来实现这一目的。

```python
lifecycle_nodes = ['sensor_driver',
                'controller_server',
                'smoother_server',
                'planner_server',
                'behavior_server',
                'bt_navigator',
                'waypoint_follower']

...

Node(
    package='nav2_sensor_driver',
    executable='sensor_driver',
    name='sensor_driver',
    output='screen',
    parameters=[configured_params],
    remappings=remappings),

Node(
    package='nav2_lifecycle_manager',
    executable='lifecycle_manager',
    name='lifecycle_manager_navigation',
    output='screen',
    parameters=[{'autostart': autostart},
                {'node_names': lifecycle_nodes}]),
```

In the snippet above, the nodes to be handled by the Lifecycle Manager are set using the `node_names` parameter. The `node_names` parameter takes in an ordered list of nodes to bringup through the Lifecycle transition. As shown in the snippet, the `node_names` parameter takes in `lifecycle_nodes` which contains the list of nodes to be added to the Lifecycle Manager. The Lifecycle Manager implements bringup transitions (`Configuring` and `Activating`) to the nodes one-by-one and in order, while the nodes are processed in reverse order for shutdown transitions. Hence, the `sensor_driver` is listed first before the other navigation servers so that the sensor data is available before the navigation servers are activated.

> 在上面的片段中，使用`node_names`参数设置要由生命周期管理器处理的节点。 `node_names`参数接受要通过生命周期转换带起来的节点的有序列表。如片段中所示，`node_names`参数接受`lifecycle_nodes`，其中包含要添加到生命周期管理器的节点列表。生命周期管理器实现了一次一个，按顺序提升转换（`配置`和`激活`）到节点，而关闭转换时节点以相反的顺序处理。因此，在激活其他导航服务器之前，首先列出`sensor_driver`，以便在激活导航服务器之前可以获取传感器数据。

Two other parameters of the Lifecycle Manager are `autostart` and `bond_timeout`. Set `autostart` to `true` if you want to set the transition nodes to the `Active` state on startup. Otherwise, you will need to manually trigger Lifecycle Manager to transition up the system. The `bond_timeout` sets the waiting time to decide when to transition down all of the nodes if a node is not responding.

> Lifecycle Manager 的另外兩個參數是 `autostart` 和 `bond_timeout`。如果你想在啟動時將轉換節點設置為 `Active` 狀態，請將 `autostart` 設置為 `true`。否則，你需要手動觸發 Lifecycle Manager 以將系統轉換為上升狀態。`bond_timeout` 設置等待時間，以便在節點不回應時決定何時將所有節點轉換為下降狀態。

::: note
::: title
Note
:::

More information on Lifecycle Manager parameters can be found in the [Configuration Guide of Lifecycle Manager](https://navigation.ros.org/configuration/packages/configuring-lifecycle.html)

> 更多有关 Lifecycle Manager 参数的信息可以在 Lifecycle Manager 配置指南中找到（https://navigation.ros.org/configuration/packages/configuring-lifecycle.html）。
> :::

# Composition

Composition is the second key component nav2 task servers that was introduced to reduce the memory and CPU resources by putting multiple nodes in a single process. In Nav2, Composition can be used to compose all Nav2 nodes in a single process instead of launching them separately. This is useful for deployment on embedded systems where developers need to optimize resource usage.

> 组合是第二个关键组件 Nav2 任务服务器，旨在通过在单个进程中放置多个节点来减少内存和 CPU 资源。在 Nav2 中，可以使用组合来将所有 Nav2 节点组合在一个进程中，而不是将它们单独启动。这对于在嵌入式系统上部署很有用，开发人员需要优化资源使用。

::: seealso

More information on Composition can be found [here](https://docs.ros.org/en/rolling/Tutorials/Intermediate/Composition.html).

> 更多关于组合的信息可以在[这里](https://docs.ros.org/en/rolling/Tutorials/Intermediate/Composition.html)找到。
> :::

In the following section, we give an example on how to add a new Nav2 server, which we notionally call the `route_server`, to our system.

> 在下一节中，我们举例说明如何将一个我们称之为“route_server”的新 Nav2 服务器添加到我们的系统中。

We make use of the launch files to compose different servers into a single process. The process is established by the `ComposableNodeContainer` container that is populated with composition nodes via `ComposableNode`. This container can then be launched and used the same as any other Nav2 node.

> 我们使用启动文件将不同的服务器组合成一个进程。这个进程由`ComposableNodeContainer`容器建立，通过`ComposableNode`填充组合节点。然后，该容器可以像其他 Nav2 节点一样启动和使用。

1.  Add a new `ComposableNode()` instance in your launch file pointing to the component container of your choice.

> 1. 在您的启动文件中添加一个新的`ComposableNode()`实例，指向您选择的组件容器。

> ```python
> container = ComposableNodeContainer(
>     name='my_container',
>     namespace='',
>     package='rclcpp_components',
>     executable='component_container',
>     composable_node_descriptions=[
>         ComposableNode(
>             package='nav2_route_server',
>             plugin='nav2_route_server::RouteServer',
>             name='nav2_route_server'),
>     ],
>     output='screen',
> )
> ```
>
> ::: seealso
> See example in composition demo\'s [composition_demo.launch.py](https://github.com/ros2/demos/blob/master/composition/launch/composition_demo.launch.py).
> :::

2.  Add the package containing the server to your `package.xml` file.

> 将包含服务器的包添加到您的`package.xml`文件中。

> ```xml
> <exec_depend>nav2_route_server</exec_depend>
> ```

# Error codes

Your nav2 task server may also wish to return a \'error_code\' in its action response (though not required). If there are semantically meaningful and actionable types of failures for your system, this is a systemic way to communicate those failures which may be automatically aggregated into the responses of the navigation system to your application.

> 您的 nav2 任务服务器也可能希望在其动作响应中返回一个'error_code'（尽管不是必需的）。如果您的系统有语义上有意义且可操作的失败类型，这是一种系统性的方式来传达这些失败，可以自动聚合到导航系统对您的应用程序的响应中。

It is important to note that error codes from 0-9999 are reserved for internal nav2 servers with each server offset by 100 while external servers start at 10000 and end at 65535. The table below shows the current servers along with the expected error code structure.

> 重要提示：错误代码从 0-9999 专门为内部 nav2 服务器保留，每个服务器偏移 100，而外部服务器从 10000 开始，到 65535 结束。下表显示了当前服务器以及预期的错误代码结构。

```table
---

Server Name Reserved RANGE

---

\... NONE=0, UNKNOWN=1 2-99

[Controller Server](https://github.com/ros-planning/navigation2/blob/main/nav2_controller/src/controller_server.cpp) NONE=0, UNKNOWN=100 101-199

[Planner Server](https://github.com/ros-planning/navigation2/blob/main/nav2_planner/src/planner_server.cpp) (compute_path_to_pose) NONE=0, UNKNOWN=200 201-299

[Planner Server](https://github.com/ros-planning/navigation2/blob/main/nav2_planner/src/planner_server.cpp) (compute_path_through_poses) NONE=0, UNKNOWN=300 301-399

\... \...

[Smoother Server](https://github.com/ros-planning/navigation2/blob/main/nav2_smoother/src/nav2_smoother.cpp) NONE=0, UNKNOWN=500 501-599

[Waypoint Follower Server](https://github.com/ros-planning/navigation2/blob/main/nav2_waypoint_follower/src/waypoint_follower.cpp) NONE=0, UNKNOWN=600 601-699

[Behavior Server](https://github.com/ros-planning/navigation2/blob/main/nav2_behaviors/src/behavior_server.cpp) NONE=0 701-799

\... \...

Last Nav2 Server NONE=0, UNKNOWN=9900 9901-9999

First External Server NONE=0, UNKNOWN=10000 10001-10099

\... \...

---
```

Error codes are attached to the response of the action message. An example can be seen below for the route server. Note that by convention we set the error code field within the message definition to `error_code`.

> 错误代码附加到动作消息的响应中。下面是路由服务器的一个示例。请注意，按照惯例，我们将消息定义中的错误代码字段设置为“error_code”。

```bash
# Error codes
# Note: The expected priority order of the errors should match the message order
uint16 NONE=0 # 0 is reserved for NONE
uint16 UNKNOWN=10000 # first error code in the sequence is reserved for UNKNOWN

# User Error codes below
int16 INVILAD_START=10001
int16 NO_VALID_ROUTE=10002

#goal definition
route_msgs/PoseStamped goal
route_msgs/PoseStamped start
string route_id
---
#result definition
nav_msgs/Route route
builtin_interfaces/Duration route_time
uint16 error_code
---
```

As stated in the message, the priority order of the errors should match the message order, 0 is reserved for NONE and the first error code in the sequence is reserved for UNKNOWN. Since the the route server is a external server, the errors codes start at 10000 and go up to 10099.

> 根据消息指定，错误优先级应与消息顺序相匹配，0 保留给无，序列中的第一个错误代码保留给未知。由于路由服务器是外部服务器，错误代码从 10000 开始，一直到 10099。

In order to propigate your server\'s error code to the rest of the system it must be added to the nav2_params.yaml file. The [error_code_id_names]{.title-ref} inside of the BT Navigator define what error codes to look for on the blackboard by the server. The lowest error code of the sequence is then returned - whereas the code enums increase the higher up in the software stack - giving higher priority to lower-level failures.

> 为了将您服务器的错误代码传播到系统的其他部分，必须将其添加到 nav2_params.yaml 文件中。 BT Navigator 中的[error_code_id_names]{.title-ref}定义了服务器要在黑板上查找的错误代码。然后返回序列中的最低错误代码，而代码枚举值会随软件堆栈的提升而增加，从而给低级故障提供更高的优先级。

```yaml
error_code_id_names:
  - compute_path_error_code_id
  - follow_path_error_code_id
  - route_error_code_id
```

# Conclusion

In this section of the guide, we have discussed Lifecycle Nodes, Composition and Error Codes which are new and important concepts in ROS 2. We also showed how to implement Lifecycle Nodes, Composition and Error Codes to your newly created nodes/servers with Nav2. These three concepts are helpful to efficiently run your system and therefore are encouraged to be used throughout Nav2.

> 在本指南的这一部分，我们讨论了 ROS 2 中的新而重要的概念——生命周期节点、组合和错误码。我们还展示了如何使用 Nav2 将生命周期节点、组合和错误码应用到新创建的节点/服务器中。这三个概念有助于有效地运行系统，因此鼓励在 Nav2 中使用它们。
