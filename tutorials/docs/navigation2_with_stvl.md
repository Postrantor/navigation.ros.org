---
tip: translate by openai@2023-06-03 00:01:53
title: (STVL) Using an External Costmap Plugin
---

- [Overview](#overview)
- [Costmap2D and STVL](#costmap2d-and-stvl)
- [Tutorial Steps](#tutorial-steps)
  - [0- Setup](#0--setup)
  - [1- Install STVL](#1--install-stvl)
  - [1- Modify Navigation2 Parameter](#1--modify-navigation2-parameter)
  - [2- Launch Navigation2](#2--launch-navigation2)
  - [3- RVIZ](#3--rviz)

```{=html}
<h1 align="center">
  <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
    <iframe width="700" height="450" src="https://www.youtube.com/embed/TGxb1OzgmNQ?autoplay=1" frameborder="1" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  </div>
</h1>
```

# Overview

This tutorial shows how to load and use an external plugin. This example uses the [Spatio Temporal Voxel Layer](https://github.com/SteveMacenski/spatio_temporal_voxel_layer/) (STVL) costmap [pluginlib](http://wiki.ros.org/pluginlib/) plugin as an example. STVL is a demonstrative pluginlib plugin and the same process can be followed for other costmap plugins as well as plugin planners, controllers, and behaviors.

> 这篇教程展示了如何加载和使用一个外部插件。本例使用[时空体素层](https://github.com/SteveMacenski/spatio_temporal_voxel_layer/)（STVL）costmap [pluginlib](http://wiki.ros.org/pluginlib/) 插件作为示例。STVL 是一个演示性的 pluginlib 插件，对于其他 costmap 插件以及 plugin 规划器、控制器和行为，可以采用相同的流程。

Before completing this tutorial, please look at the previous two tutorials on navigation in simulation and physical hardware, if available. This tutorial assumes knowledge of navigation and basic understanding of costmaps.

> 在完成本教程之前，请查看以前关于仿真和物理硬件导航的两个教程（如果有的话）。本教程假定您已经了解导航和成本图的基本概念。

::: note
::: title
Note
:::

For Ubuntu 20.04 users before December 2021, there\'s a known issue with OpenVDB and its binaries with `libjmalloc`. If you see an error such as `Could not load library LoadLibrary error: /usr/lib/x86_64-linux-gnu/libjemalloc.so.2: cannot allocate memory in static TLS block`, it can be resolved with `export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2` until new binaries are released of OpenVDB.

> 对于 Ubuntu 20.04 用户，在 2021 年 12 月之前，OpenVDB 及其二进制文件存在已知问题，与`libjmalloc`有关。如果您看到类似`Could not load library LoadLibrary error: /usr/lib/x86_64-linux-gnu/libjemalloc.so.2: cannot allocate memory in static TLS block`的错误，可以在新的 OpenVDB 二进制文件发布之前，使用`export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2`来解决问题。
> :::

# Costmap2D and STVL

Costmap 2D is the data object we use to buffer sensor information into a global view that the robot will use to create plans and control efforts. Within Costmap2D, there are pluginlib plugin interfaces available to create custom behaviors loadable at runtime. Examples of included pluginlib plugins for Costmap2D are the Obstacle Layer, Voxel Layer, Static Layer, and Inflation Layer.

> Costmap 2D 是我们用来缓冲传感器信息到全局视图的数据对象，机器人将使用它来创建计划和控制努力。在 Costmap2D 中，有可用于创建可在运行时加载的自定义行为的 pluginlib 插件接口。Costmap2D 的 pluginlib 插件示例包括障碍层、体素层、静态层和膨胀层。

However, these are simply example plugins offered by the base implementation. Another available pluginlib plugin for Costmap2D in Navigation2 is STVL.

> 然而，这些只是基本实现提供的示例插件。 Navigation2 中 Costmap2D 的另一个可用的 pluginlib 插件是 STVL。

STVL is another 3D perception plugin similar to the Voxel Layer. A more detailed overview of how it works can be found [in this repo](https://github.com/SteveMacenski/spatio_temporal_voxel_layer/), however it buffers 3D data from depth cameras, sonars, lidars, and more into a sparse volumetic world model and removes voxels over time proportional with a sensor model and time-based expiration. This can be especially useful for robots in highly dynamic envrionments and decreases the resource utilization for 3D sensor processing by up to 2x. STVL also treats 3D lidars and radars as first class citizens for support. The ROSCon talk for STVL can be found [in this video](https://vimeo.com/292699571).

> STVL 是另一种类似于 Voxel Layer 的 3D 感知插件。有关其工作原理的更详细概述可以在[此存储库](https://github.com/SteveMacenski/spatio_temporal_voxel_layer/)中找到，但它可以将深度摄像机、声纳、激光雷达等 3D 数据缓冲到稀疏的体积世界模型中，并根据传感器模型和基于时间的过期时间比例去除体素。这对于在高度动态环境中的机器人尤其有用，可以将 3D 传感器处理的资源利用率降低高达 2 倍。 STVL 还将 3D 激光雷达和雷达作为第一类公民来支持。STVL 的 ROSCon 演讲可以在[此视频](https://vimeo.com/292699571)中找到。

# Tutorial Steps

## 0- Setup

Follow the same process as in `getting_started`{.interpreted-text role="ref"} for installing and setting up a robot for hardware testing or simulation, as applicable. Ensure ROS 2, Navigation2, and Gazebo are installed.

> 遵循`getting_started`{.interpreted-text role="ref"}中的相同过程，安装和设置机器人以进行硬件测试或模拟，取决于情况。确保已安装 ROS 2、Navigation2 和 Gazebo。

## 1- Install STVL

STVL can be installed in ROS 2 via the ROS Build Farm:

> STVL 可以通过 ROS Build Farm 在 ROS 2 中安装：`Wait`

- `sudo apt install ros-<ros2-distro>-spatio-temporal-voxel-layer`

It can also be built from source by cloning the repository into your Navigation2 workspace:

> 可以通过将存储库克隆到您的 Navigation2 工作区来源构建：`Wait`

- `git clone -b <ros2-distro>-devel git@github.com:stevemacenski/spatio_temporal_voxel_layer`

## 1- Modify Navigation2 Parameter

STVL is an optional plugin, like all plugins, in Costmap2D. Costmap Plugins in Navigation2 are loaded in the `plugin_names` and `plugin_types` variables inside of their respective costmaps. For example, the following will load the static and obstacle layer plugins into the name `static_layer` and `obstacle_layer`, respectively:

> `STVL` 是一个可选的插件，就像所有的插件一样，在 `Costmap2D` 中。在 `Navigation2` 中的 `Costmap Plugins` 被加载到各自的 `costmaps` 中的 `plugin_names` 和 `plugin_types` 变量中。例如，以下将静态层和障碍层插件分别加载到 `static_layer` 和 `obstacle_layer` 中：`

```yaml
global_costmap:
  global_costmap:
    ros__parameters:
      use_sim_time: True
      plugins: ["static_layer", "obstacle_layer"]
```

::: note
::: title
Note
:::

For Galactic or later, `plugin_names` and `plugin_types` have been replaced with a single `plugins` string vector for plugin names. The types are now defined in the `plugin_name` namespace in the `plugin:` field (e.g. `plugin: MyPlugin::Plugin`). Inline comments in the code blocks will help guide you through this.

> 对于 Galactic 或更高版本，`plugin_names`和`plugin_types`已经用单个`plugins`字符串向量替换，用于插件名称。类型现在定义在`plugin_name`命名空间中的`plugin:`字段中（例如`plugin: MyPlugin::Plugin`）。代码块中的内联注释将帮助您指引此过程。`Wait`
> :::

To load the STVL plugin, a new plugin name and type must be added. For example, if the application required an STVL layer and no obstacle layer, our file would be:

> 要加载 STVL 插件，必须添加新的插件名称和类型。例如，如果应用程序需要一个 STVL 层和没有障碍层，我们的文件将是：`Wait`

```yaml
global_costmap:
  global_costmap:
    ros__parameters:
      use_sim_time: True
      plugins: ["static_layer", "stvl_layer"]
```

Similar to the Voxel Layer, after registering the plugin, we can add the configuration of the STVL layer under the namespace `stvl_layer`. An example fully-described parameterization of an STVL configuration is:

> 类似 Voxel Layer，在注册插件后，我们可以在名称空间`stvl_layer`下添加 STVL 层的配置。STVL 配置的一个完整描述参数的示例是：`Wait`

```yaml
stvl_layer:
  plugin: "spatio_temporal_voxel_layer/SpatioTemporalVoxelLayer"
  enabled: true
  voxel_decay: 15.
  decay_model: 0
  voxel_size: 0.05
  track_unknown_space: true
  unknown_threshold: 15
  mark_threshold: 0
  update_footprint_enabled: true
  combination_method: 1
  origin_z: 0.0
  publish_voxel_map: true
  transform_tolerance: 0.2
  mapping_mode: false
  map_save_duration: 60.0
  observation_sources: pointcloud
  pointcloud:
    data_type: PointCloud2
    topic: /intel_realsense_r200_depth/points
    marking: true
    clearing: true
    obstacle_range: 3.0
    min_obstacle_height: 0.0
    max_obstacle_height: 2.0
    expected_update_rate: 0.0
    observation_persistence: 0.0
    inf_is_valid: false
    filter: "voxel"
    voxel_min_points: 0
    clear_after_reading: true
    max_z: 7.0
    min_z: 0.1
    vertical_fov_angle: 0.8745
    horizontal_fov_angle: 1.048
    decay_acceleration: 15.0
    model_type: 0
```

Please copy-paste the text above, with the `plugin_names` and `plugin_types` registration, into your `nav2_params.yaml` to enable STVL in your application. Make sure to change both the local and global costmaps.

> 请将上面的文本，包括`plugin_names`和`plugin_types`注册，复制粘贴到你的`nav2_params.yaml`中以启用 STVL。确保更改本地和全局 costmaps。

Note: Pluginlib plugins for other Navigation2 servers such as planning, behavior, and control can be set in this same way.

## 2- Launch Navigation2

Follow the same process as in `getting_started`{.interpreted-text role="ref"} to launch a simulated robot in gazebo with Navigation2. Navigation2 is now using STVL as its 3D sensing costmap layer.

> 跟隨`getting_started`{.interpreted-text role="ref"}中的同樣步驟來啟動 Gazebo 中的模擬機器人，Navigation2 現在使用 STVL 作為 3D 感測成本地圖層。

## 3- RVIZ

With RViz open and `publish_voxel_map: true`, you can visualize the underlying data structure\'s 3D grid using the `{local, global}_costmap/voxel_grid` topics. Note: It is recommended in RViz to set the `PointCloud2` Size to your voxel size and the style to `Boxes` with a neutral color for best visualization.

> 当 RViz 打开并设置`publish_voxel_map: true`时，您可以使用`{local, global}_costmap/voxel_grid`主题可视化底层数据结构的 3D 网格。注意：在 RViz 中，建议将`PointCloud2`大小设置为您的体素大小，样式设置为`Boxes`，颜色为中性色，以获得最佳可视化效果。
