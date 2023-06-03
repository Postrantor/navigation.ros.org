---
tip: translate by openai@2023-06-02 15:50:46
title: Filtering of Noise-Induced Obstacles
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1. Enable Denoise Layer](#1-enable-denoise-layer)
  - [2. Run Nav2 stack](#2-run-nav2-stack)
- [How it works](#how-it-works)

![image](images/Filtering_of_noise-induced_obstacles/title.png){width="1000px"}

# Overview

Noisy sensor measurements can cause to errors in `Voxel Layer` or `Obstacle Layer`. As a result, salt and pepper noise may appear on the costmap. This noise creates false obstacles that prevent the robot from finding the best path on the map. While the images above show both salt and pepper noise as well as error due to mislocalization, this layer will only remove sensor noise, not mislocalized artifacts misaligned with the static map. This tutorial shows how to configure filtering of false obstacles caused by noise. This functionality is provided by the `DenoiseLayer` costmap layer plugin which will be enabled and used in this document.

> 噪声传感器测量可能导致 Voxel Layer 或 Obstacle Layer 中的错误。结果，成本图上可能出现盐和胡椒噪声。这种噪声会产生假障碍，阻止机器人在地图上找到最佳路径。虽然上面的图片显示了盐和胡椒噪声以及与静态地图不一致的定位错误，但此层仅会消除传感器噪声，而不会消除定位错误。本教程将演示如何配置滤除由噪声引起的假障碍。此功能由 DenoiseLayer 成本图层插件提供，并将在本文档中启用和使用。

# Requirements

It is assumed that ROS 2, Gazebo and TurtleBot3 packages are installed or built locally. Please make sure that Nav2 project is also built locally as it was made in `build-instructions`{.interpreted-text role="ref"}.

> 假设 ROS 2、Gazebo 和 TurtleBot3 软件包已安装或本地构建。请确保 Nav2 项目也在本地构建，就像在`build-instructions`中所做的那样。

# Tutorial Steps

## 1. Enable Denoise Layer

Denoise Layer is Costmap2D plugin. You can enable the `DenoiseLayer` plugin in Costmap2D by adding `denoise_layer` to the `plugins` parameter in `nav2_params.yaml`. You can place it in the `global_costmap` and (or) `local_costmap` to filter noise on a global or local map. The DenoiseLayer plugin should have the following parameter defined:

> DenoiseLayer 是 Costmap2D 插件。您可以通过在 nav2_params.yaml 中的 plugins 参数中添加 denoise_layer 来启用 DenoiseLayer 插件。您可以将其放置在 global_costmap 和（或）local_costmap 中，以过滤全局或局部地图上的噪声。 DenoiseLayer 插件应定义以下参数：`Wait`

- `plugin`: type of plugin. In our case `nav2_costmap_2d::DenoiseLayer`.

Full list of parameters supported by `DenoiseLayer` are listed at `denoise`{.interpreted-text role="ref"} page.

> {.interpreted-text role="ref"}

DenoiseLayer 支持的全部参数列表可在`denoise`{.interpreted-text role="ref"}页面找到。

It is important to note that `DenoiseLayer` typically should be placed before the inflation layer. This is required to prevent inflation from noise-induced obstacles. Moreover, `DenoiseLayer` processes only obstacle information in the costmap. Values `INSCRIBED_INFLATED_OBSTACLE`, `LETHAL_OBSTACLE` and optionally `NO_INFORMATION` will be interpreted as obstacle cell. Cells with any other values will be interpreted as `FREE_SPACE` when processed (won\'t be distorted in the cost map). If a cell with an obstacle is recognized as noise, it will be replaced by `FREE_SPACE` after processing.

> 重要的是要注意，`DenoiseLayer`通常应该放在膨胀层之前。这是为了防止噪声诱发的障碍物导致膨胀。此外，`DenoiseLayer`仅处理成本图中的障碍物信息。值`INSCRIBED_INFLATED_OBSTACLE`，`LETHAL_OBSTACLE`和可选的`NO_INFORMATION`将被解释为障碍单元。其他任何值的单元在处理时将被解释为`FREE_SPACE`（不会在成本图中变形）。如果被识别为噪声的障碍单元将在处理后被替换为`FREE_SPACE`。

To enable `DenoiseLayer` for both global and local costmaps, use the following configuration:

> 要启用`DenoiseLayer`用于全局和局部成本图，请使用以下配置：`Wait`

    ```text
    global_costmap:
      global_costmap:
        ros__parameters:
          ...
          plugins: ["static_layer", "obstacle_layer", "denoise_layer", "inflation_layer"]
          ...
          denoise_layer:
            plugin: "nav2_costmap_2d::DenoiseLayer"
            enabled: True
    ...
    local_costmap:
      local_costmap:
        ros__parameters:
          ...
          plugins: ["voxel_layer", "denoise_layer", inflation_layer"]
          ...
          keepout_filter:
            plugin: "nav2_costmap_2d::DenoiseLayer"
            enabled: True
    ```

::: note
::: title
Note
:::

The key to success in filtering noise is to understand its type and choose the right `DenoiseLayer` parameters. The default parameters are focused on fast removal of standalone obstacles. More formally, an obstacle is discarded if there are no obstacles among the adjacent eight cells. This should be sufficient in typical cases.

> 降噪成功的关键在于理解它的类型，并选择正确的`DenoiseLayer`参数。默认参数专注于快速移除独立的障碍物。更正式地说，如果相邻八个单元格中没有障碍物，则会丢弃障碍物。在典型情况下，这应该足够了。

If some sensor generates intercorrelated noise-induced obstacles and small obstacles in the world are unlikely, small groups of obstacles can be removed. To configure the `DenoiseLayer` to such cases and understand how it works, refer to the section [How it works](#how-it-works).

> 如果某些传感器产生相关噪声所引起的小障碍在世界上不太可能出现，可以移除小组障碍。要配置`DenoiseLayer`以适应这种情况并了解它是如何工作的，请参考[如何工作](#how-it-works)部分。
> :::

::: warning
::: title
Warning
:::

Use this plugin to filter the global costmap with caution. It introduces potential performance issues. For example in case of typically-high-range lidars (20+ meters) update window can be massive making processing time unacceptably long. It is worth taking this into account as an application designer.

> 使用此插件谨慎过滤全局成本地图。它引入了潜在的性能问题。例如，对于典型的高范围激光雷达（20+米），更新窗口可能会非常大，使处理时间变得不可接受。作为应用程序设计者，值得考虑这一点。
> :::

## 2. Run Nav2 stack

After Denoise Layer was enabled for global/local costmaps, run Nav2 stack as written in `getting_started`{.interpreted-text role="ref"}:

> 在启用全局/局部成本地图的降噪层之后，按照`getting_started`中的说明运行 Nav2 堆栈：`Wait`

```bash

ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False

> ros2启动nav2_bringup tb3_simulation_launch.py headless:=False `等待`
```

And check that filter is working properly: with the default parameters, no standalone obstacles should remain on the cost map. This can be checked, for example, in RViz main window displaying local and global costmaps after removing unnecessary particles (illustrated at the top of this tutorial).

> 检查过滤器正常工作：使用默认参数，成本地图上不应该有任何孤立的障碍物。可以在 RViz 主窗口中检查，在移除不必要的粒子（本教程顶部有图示）后显示本地和全局成本地图。

# How it works

The plugin is based on two algorithms.

> `插件基于两种算法。`

When parameter `minimal_group_size` = 2, the first algorithm turns on. It apply [erosion](https://docs.opencv.org/3.4/db/df6/tutorial_erosion_dilatation.html) function with kernel from image below (left if `group_connectivity_type` = 4 or right if `group_connectivity_type` = 8) to the costmap. White color of the kernel pixel means to use the value, black means to ignore it.

> 当参数`minimal_group_size`等于 2 时，第一个算法会打开。它使用下图（如果`group_connectivity_type`等于 4，则使用左图，如果`group_connectivity_type`等于 8，则使用右图）中的内核对成本地图应用[腐蚀](https://docs.opencv.org/3.4/db/df6/tutorial_erosion_dilatation.html)函数。内核像素的白色意味着使用该值，黑色意味着忽略它。

![image](images/Filtering_of_noise-induced_obstacles/3x3_kernels.png){width="222px"}

As a result of erosion function the neighbors image is created. Each possible position of the kernel on the costmap corresponds to one pixel of the neighbors image. The pixel value of this image is equal to the maximum of 4/8 costmap pixels corresponding to the white pixels of the mask. In other words, the pixel of the neighbors image is equal to the obstacle code if there is an obstacle nearby, the free space code in other case. After that, obstacles corresponding to free space code on neighbors image are removed.

> 结果，通过侵蚀功能产生邻域图像。成本图上的每个可能的内核位置对应于邻域图像的一个像素。该图像的像素值等于与掩码的白色像素对应的 4/8 成本图像的最大值。换句话说，如果附近有障碍物，邻域图像的像素等于障碍物代码，在其他情况下等于空间代码。然后，删除邻域图像上对应于空间代码的障碍物。

This process is illustrated below. On the left side of the image is a costmap, on the right is a neighbors image. White pixels are free space, black pixels are obstacles, `group_connectivity_type` = 4. Obstacles marked at the end of the animation will be removed.

> 这个过程如下图所示。图像左侧是成本图，右侧是邻居图。白色像素是空闲空间，黑色像素是障碍物，`group_connectivity_type` = 4。动画结尾处标记的障碍物将被移除。

![image](images/Filtering_of_noise-induced_obstacles/dilate.gif){width="600px"}

When parameter `minimal_group_size` \> 2, the second algorithm is executed. This is a generalized solution that allows you to remove groups of adjacent obstacles if their total number is less than `minimal_group_size`. To select groups of adjacent obstacles, the algorithm performs their segmentation. The type of cell connectivity in one segment is determined by the parameter `group_connectivity_type`. Next, the size of each segment is calculated. Obstacles segments with size less than the `minimal_group_size` are replaced with empty cells. This algorithm is about 10 times slower than first, so use it with caution and only when necessary. Its execution time depends on the size of the processed map fragment (and not depend on the value of `minimal_group_size`).

> 当参数`minimal_group_size` > 2 时，执行第二种算法。这是一种通用的解决方案，可以在其总数小于`minimal_group_size`的情况下移除相邻障碍物组。为了选择相邻障碍物组，该算法执行其分割。单个分段中的单元连接类型由参数`group_connectivity_type`确定。接下来，计算每个分段的大小。小于`minimal_group_size`的障碍物分段将被替换为空单元。此算法比第一种算法慢大约 10 倍，因此请谨慎使用，只有在必要时才使用它。其执行时间取决于处理的地图片段的大小（而不取决于`minimal_group_size`的值）。

This algorithm is illustrated in the animation below (`group_connectivity_type` = 8). Obstacles marked at the end of the animation will be removed (groups that size less 3).

> 这个算法在下面的动画中有所说明（`group_connectivity_type` = 8）。在动画结束时标记的障碍物将会被移除（小于 3 的组）。

![image](images/Filtering_of_noise-induced_obstacles/connected_components.gif){width="600px"}
