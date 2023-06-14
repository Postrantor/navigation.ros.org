---
tip: translate by openai@2023-06-02 13:51:38
title: Using Collision Monitor
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Configuring Collision Monitor](#configuring-collision-monitor)
- [Preparing Nav2 stack](#preparing-nav2-stack)
- [Demo Execution](#demo-execution)

![image](images/Collision_Monitor/collision_monitor.gif){width="800px"}

# Overview

This tutorial shows how to use a Collision Monitor with Nav2 stack. Based on this tutorial, you can setup it for your environment and needs.

> 这个教程展示了如何使用 Nav2 堆栈中的碰撞监视器。根据本教程，您可以根据自己的环境和需求来设置它。

# Requirements

It is assumed ROS2 and Nav2 dependent packages are installed or built locally. Please make sure that Nav2 project is also built locally as it was made in `build-instructions`{.interpreted-text role="ref"}.

> 假设 ROS2 和 Nav2 相关的软件包已经安装或本地构建。请确保 Nav2 项目也是本地构建的，就像在“构建说明”中所述一样。

# Configuring Collision Monitor

The Collision Monitor node has its own `collision_monitor_node.launch.py` launch-file and preset parameters in the `collision_monitor_params.yaml` file for demonstration, though its trivial to add this to Nav2\'s main launch file if being used in practice. For the demonstration, two shapes will be created - an inner stop and a larger slowdown bounding boxes placed in the front of the robot:

> 碰撞监视节点有自己的`collision_monitor_node.launch.py`启动文件和`collision_monitor_params.yaml`文件中的预设参数，尽管在实践中可以将其添加到 Nav2 的主启动文件中。为了演示，将创建两个形状-一个内停止和一个更大的减速边界框放置在机器人前面：

![image](images/Collision_Monitor/polygons.png){width="800px"}

If more than 3 points will appear inside a slowdown box, the robot will decrease its speed to `30%` from its value. For the cases when obstacles are dangerously close to the robot, inner stop zone will work. For this setup, the following lines should be added into `collision_monitor_params.yaml` parameters file. Stop box is named as `PolygonStop` and slowdown bounding box - as `PolygonSlow`:

> 如果在减速框内出现超过 3 个点，机器人将将其速度降低至 30％。当障碍物非常接近机器人时，内部停止区域将起作用。对于此设置，应将以下行添加到`collision_monitor_params.yaml`参数文件中：停止框被命名为`PolygonStop`，减速边界框被命名为`PolygonSlow`：

```yaml
polygons: ["PolygonStop", "PolygonSlow"]
PolygonStop:
  type: "polygon"
  points: [0.4, 0.3, 0.4, -0.3, 0.0, -0.3, 0.0, 0.3]
  action_type: "stop"
  min_points: 4 # max_points: 3 for Humble
  visualize: True
  polygon_pub_topic: "polygon_stop"
PolygonSlow:
  type: "polygon"
  points: [0.6, 0.4, 0.6, -0.4, 0.0, -0.4, 0.0, 0.4]
  action_type: "slowdown"
  min_points: 4 # max_points: 3 for Humble
  slowdown_ratio: 0.3
  visualize: True
  polygon_pub_topic: "polygon_slowdown"
```

::: note
::: title
Note
:::

The circle shape could be used instead of polygon, e.g. for the case of omni-directional robots where the collision can occur from any direction. However, for the tutorial needs, let\'s focus our view on polygons. For the same reason, we leave out of scope the Approach model. Both of these cases could be easily enabled by referencing to the `configuring_collision_monitor`{.interpreted-text role="ref"} configuration guide.

> 圆形可以代替多边形，例如在全向机器人的情况下，可能会发生碰撞。但是，为了教程的需要，让我们将视野集中在多边形上。出于同样的原因，我们将 Approach 模型排除在外。通过参考`configuring_collision_monitor`{.interpreted-text role="ref"}配置指南，这两种情况都可以轻松实现。

:::

::: note
::: title
Note
:::

Both polygon shapes in the tutorial were set statically. However, there is an ability to dynamically adjust them over time using topic messages containing vertices points for polygons or footprints. For more information, please refer to the configuration guide.

> 两个多边形图形在教程中都是静态设置的。但是，可以使用包含多边形顶点点或足迹的主题消息动态调整它们。有关更多信息，请参阅配置指南。

:::

For the working configuration, at least one data source should be added. In current demonstration, it is used laser scanner (though `PointCloud2` and Range/Sonar/IR sensors are also possible), which is described by the following lines for Collision Monitor node:

> 对于工作配置，至少应添加一个数据源。在当前演示中，它使用激光扫描仪（尽管也可以使用`PointCloud2`和 Range/Sonar/IR 传感器），由碰撞监视器节点描述如下：

```yaml
observation_sources: ["scan"]
scan:
  type: "scan"
  topic: "scan"
```

Set topic names, frame ID-s and timeouts to work correctly with a default Nav2 setup. The whole `nav2_collision_monitor/params/collision_monitor_params.yaml` file in this case will look as follows:

> 设置主题名称，帧 ID 和超时以正确使用默认的 Nav2 设置。在这种情况下，整个`nav2_collision_monitor/params/collision_monitor_params.yaml`文件将如下所示：

```yaml
collision_monitor:
  ros__parameters:
    use_sim_time: True
    base_frame_id: "base_footprint"
    odom_frame_id: "odom"
    cmd_vel_in_topic: "cmd_vel_raw"
    cmd_vel_out_topic: "cmd_vel"
    transform_tolerance: 0.5
    source_timeout: 5.0
    stop_pub_timeout: 2.0
    polygons: ["PolygonStop", "PolygonSlow"]
    PolygonStop:
      type: "polygon"
      points: [0.4, 0.3, 0.4, -0.3, 0.0, -0.3, 0.0, 0.3]
      action_type: "stop"
      min_points: 4 # max_points: 3 for Humble
      visualize: True
      polygon_pub_topic: "polygon_stop"
    PolygonSlow:
      type: "polygon"
      points: [0.6, 0.4, 0.6, -0.4, 0.0, -0.4, 0.0, 0.4]
      action_type: "slowdown"
      min_points: 4 # max_points: 3 for Humble
      slowdown_ratio: 0.3
      visualize: True
      polygon_pub_topic: "polygon_slowdown"
    observation_sources: ["scan"]
    scan:
      type: "scan"
      topic: "scan"
```

# Preparing Nav2 stack

The Collision Monitor is designed to operate below Nav2 as an independent safety node. This acts as a filter on the `cmd_vel` topic coming out of the Controller Server. If no such zone is triggered, then the Controller\'s `cmd_vel` is used. Else, it is scaled or set to stop as appropriate. For correct operation of the Collision Monitor with the Controller, it is required to add the `cmd_vel -> cmd_vel_raw` remapping to the `navigation_launch.py` bringup script as presented below:

> 碰撞监视器被设计用作 Nav2 下的独立安全节点。它对来自控制服务器的`cmd_vel`主题进行过滤。如果没有触发此类区域，则使用控制器的`cmd_vel`。否则，按照适当的方式进行缩放或停止。为了正确操作碰撞监视器和控制器，需要将`cmd_vel-> cmd_vel_raw`重映射添加到`navigation_launch.py`带起脚本中，如下所示：

```python
    Node(
        package='nav2_controller',
        executable='controller_server',
        output='screen',
        respawn=use_respawn,
        respawn_delay=2.0,
        parameters=[configured_params],
    +   remappings=remappings + [('cmd_vel', 'cmd_vel_raw')]),
    ...
    ComposableNode(
        package='nav2_controller',
        plugin='nav2_controller::ControllerServer',
        name='controller_server',
        parameters=[configured_params],
    +   remappings=remappings + [('cmd_vel', 'cmd_vel_raw')]),
```

Please note, that the remapped `cmd_vel_raw` topic should match to the input velocity `cmd_vel_in_topic` parameter value of the Collision Monitor node, and the output velocity `cmd_vel_out_topic` parameter value should be actual `cmd_vel` to fit the replacement.

> 请注意，重新映射的`cmd_vel_raw`主题应与碰撞监视器节点的输入速度`cmd_vel_in_topic`参数值匹配，输出速度`cmd_vel_out_topic`参数值应为实际的`cmd_vel`以进行替换。

# Demo Execution

Once Collision Monitor node has been tuned and `cmd_vel` topics remapped, Collision Monitor node is ready to run. For that, run Nav2 stack as written in `getting_started`{.interpreted-text role="ref"}:

> 一旦碰撞监视节点被调整并且`cmd_vel`主题被重映射，碰撞监视节点就准备好运行了。为此，按照`getting_started`中写的运行 Nav2 堆栈：

```bash
ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False
```

In parallel console, launch Collision Monitor node by using its launch-file:

> 在并行控制台中，使用其启动文件启动碰撞监视节点：

```bash
ros2 launch nav2_collision_monitor collision_monitor_node.launch.py
```

Since both `PolygonStop` and `PolygonSlow` polygons will have their own publishers, they could be added to visualization as shown at the picture below:

> 由于 PolygonStop 和 PolygonSlow 多边形都将有自己的发布者，它们可以像下图所示添加到可视化中：

![image](images/Collision_Monitor/polygons_visualization.png){width="800px"}

Set the initial pose and then put Nav2 goal on map. The robot will start its movement, slowing down while running near the obstacles, and stopping in close proximity to them:

> 设置初始姿态，然后将 Nav2 目标放置在地图上。机器人将开始移动，在靠近障碍物时减速，并在靠近它们时停止。

![image](images/Collision_Monitor/collision.png){width="800px"}
