---
tip: translate by openai@2023-06-02 15:46:52
title: Adding a Smoother to a BT
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [0- Familiarization with the Smoother BT Node](#0--familiarization-with-the-smoother-bt-node)
  - [1- Specifying a Smoother Plugin](#1--specifying-a-smoother-plugin)
  - [2- Modifying your BT XML](#2--modifying-your-bt-xml)

![image](images/smoothing_path.png){.align-center width="60.0%"}

# Overview

This tutorial shows how to add a smoother to your behavior tree to smooth output paths from a path planner. Before completing this tutorials, completing `getting_started`{.interpreted-text role="ref"} is highly recommended especially if you are new to ROS and Nav2.

> 这个教程展示了如何向行为树中添加平滑器来平滑路径规划器的输出路径。在完成本教程之前，特别是如果你是 ROS 和 Nav2 的新手，强烈建议你先完成`getting_started`{.interpreted-text role="ref"}。

# Requirements

You must install Nav2, Turtlebot3. If you don\'t have them installed, please follow `getting_started`{.interpreted-text role="ref"}. You must also have a working behavior tree, such as those provided by the Nav2 BT Navigator package, for editing. You should also have a copy of the `nav2_params.yaml` file for your system to edit as well.

> 你必须安装 Nav2 和 Turtlebot3。如果你没有安装，请按照`getting_started`{.interpreted-text role="ref"}。你还必须有一个可用的行为树，比如 Nav2 BT Navigator 包提供的行为树，用于编辑。你也应该有一份`nav2_params.yaml`文件，用于编辑你的系统。

# Tutorial Steps

## 0- Familiarization with the Smoother BT Node

The `bt_smooth_action`{.interpreted-text role="ref"} BT node is a behavior tree node that interacts with the smoother task server similar to that which you may expect to find for the planner or controller servers. It contains the action client to call the server and specifies its arguments and return types as BT ports. It too calls the server via an action interface that may be seperately interacted with via other servers and client library languages.

> `bt_smooth_action`{.interpreted-text role="ref"}节点是一个行为树节点，它与更平滑的任务服务器交互，就像您可能期望在计划程序或控制器服务器上找到的那样。它包含调用服务器的动作客户端，并将其参数和返回类型指定为 BT 端口。它也通过可以通过其他服务器和客户端库语言单独与之交互的动作界面调用服务器。

Please see the BT node\'s configuration page to familiarize yourself with all aspects, but the core ports to note are the `unsmoothed_path` input port and the `smoothed_path` output port. The first takes in a raw path from a planning algorithm and the latter will set the value of the smoothed output path post-smoothing. Other ports are available that fully implements the Smoother Server\'s action API.

> 请查看 BT 节点的配置页面，以熟悉所有方面，但要注意的核心端口是`unsmoothed_path`输入端口和`smoothed_path`输出端口。前者接收来自规划算法的原始路径，后者将在平滑后设置平滑输出路径的值。还有其他端口可以完全实现 Smoother Server 的动作 API。

## 1- Specifying a Smoother Plugin

In order to use a smoother in your BT node, you must first configure the smoother server itself to contain the smoother plugins of interest. These plugins implement the specific algorithms that you would like to use.

> 要在您的 BT 节点中使用平滑器，您必须首先配置平滑器服务器本身以包含感兴趣的平滑器插件。这些插件实现您想要使用的特定算法。

For each smoother plugin you would like to use, a name must be given to it (e.g. `simple_smoother`, `curvature_smoother`). This name is its `smoother_id` for other servers to interact with this algorithm from a request to the Smoother Server\'s action interface.

> 对于每一个您想要使用的平滑插件，必须给它起一个名字（例如`simple_smoother`、`curvature_smoother`）。这个名字就是它的`smoother_id`，其他服务器通过向平滑服务器的动作接口发出请求，来与这个算法交互。

Under each name, the parameters for that particular algorithm must be specified along with the `plugin` name for pluginlib to load a given algorithm\'s library. An example configuration of 2 smoother plugins is shown below that could be used in the `nav2_params.yaml` for your robot.

> 在每个名称下，必须指定该特定算法的参数以及 pluginlib 加载给定算法库所需的`plugin`名称。 下面展示了可用于机器人的`nav2_params.yaml`中的 2 个平滑插件的示例配置。`

    ```yaml
    smoother_server:
      ros__parameters:
        costmap_topic: global_costmap/costmap_raw
        footprint_topic: global_costmap/published_footprint
        robot_base_frame: base_link
        transform_timeout: 0.1
        smoother_plugins: ["simple_smoother", "curvature_smoother"]
        simple_smoother:
          plugin: "nav2_smoother::SimpleSmoother"
          tolerance: 1.0e-10
          do_refinement: True
        curvature_smoother:
          plugin: "nav2_ceres_costaware_smoother/CeresCostawareSmoother"
    ```

## 2- Modifying your BT XML

Now that you have selected and configured the smoother server for your given plugin(s), it is time to use those smoother(s) in your behavior tree for navigation behavior. While there are many places / ways to use this in a BT, what is shown below is probably the most likely situation you would want to use the smoother in (to smooth a path returned by the path planner and then using that smoothed path for path tracking).

> 现在您已经为给定的插件选择并配置了平滑服务器，是时候在行为树中使用这些平滑器了。虽然在 BT 中有很多地方/方式可以使用这个，但下面显示的可能是您想要使用平滑器的最可能的情况（用于平滑路径规划器返回的路径，然后使用该平滑路径进行路径跟踪）。

Note: If you use only a single type of smoothing algorithm, there is no need to specify the `smoother_id` in the BT XML entry. Since there is only a single option, that will be used for any request that does not specifically request a smoother plugin. However, if you leverage multiple smoother plugins, you **must** populate the `smoother_id` XML port.

A given behavior tree will have a line:

> 给定的行为树将有一行：`Wait`

```xml
<ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased" error_code_id="{compute_path_error_code}"/>
```

This line calls the planner server and return a path to the `path` blackboard variable in the behavior tree. We are going to replace that line with the following to compute the path, smooth the path, and finally replace the `path` blackboard variable with the new smoothed path that the system will now interact with:

> 这一行调用规划服务器，并将路径返回到行为树中的`path`黑板变量。我们将用以下语句替换它，以计算路径、平滑路径，最后用新的平滑路径替换`path`黑板变量，系统现在将与之交互：`Wait`

```xml
<Sequence name="ComputeAndSmoothPath">
  <ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased" error_code_id="{compute_path_error_code}"/>
  <SmoothPath unsmoothed_path="{path}" smoothed_path="{path}" error_code_id="{smoother_error_code}"/>
</Sequence>
```

If you wish to have recoveries for the smoother error codes, such as triggering the system recoveries branch of a behavior tree:

> 如果你希望拥有更平滑的错误代码恢复，比如触发行为树的系统恢复分支：`Wait`

```xml
<Sequence name= "TryToResolveSmootherErrorCodes">
  <WouldASmootherRecoveryHelp error_code="{smoother_error_code}">
  <!-- recovery to resolve smoother error code goes here -->
<Sequence/>
```

And its as simple as that! You can now compile or use this behavior tree in your system and see that the plans are now smoothed and the controllers are now tracking this smoothed path. The image at the top of the tutorial shows the unsmoothed path from NavFn (red) and the smoothed path (black). Note the smoother approach to goal, turns, and transitions in the straight-line segments.

> 简而言之，就是这样！您现在可以在系统中编译或使用此行为树，并看到计划现在已经平滑，控制器现在正在跟踪这条平滑路径。教程顶部的图像显示了 NavFn（红色）的未平滑路径和平滑路径（黑色）。注意接近目标，转弯和转折处的更平滑的方法。

If you wish to see the difference, but not track the smoothed path, you may wish to remove the `smoothed_path="{path}"` portion to compute the smoothed path, but not replace the original path with it. Instead, the topic `/smoothed_path` contains this information published by the smoother server for visualization or use by other parts of the system. You may also remap the smoothed path to another blackboard variable to interact with it in other parts of the behavior tree (e.g. `smoothed_path="{smoothed_path}"`).

> 如果您想要看到差异，但不跟踪平滑的路径，您可能需要删除`smoothed_path="{path}"`部分以计算平滑的路径，但不替换原始路径。相反，主题`/smoothed_path`包含由平滑服务器发布的用于可视化或由系统的其他部分使用的信息。您还可以将平滑的路径重新映射到另一个黑板变量，以在行为树的其他部分中与它交互（例如`smoothed_path="{smoothed_path}"`）。
