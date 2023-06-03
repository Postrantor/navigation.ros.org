---
tip: translate by openai@2023-06-03 00:07:45
...
---
title: Navigating with Keepout Zones
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1. Prepare filter mask](#1-prepare-filter-mask)

  - [2. Configure Costmap Filter Info Publisher Server](#2-configure-costmap-filter-info-publisher-server)

> - [2. 配置 Costmap Filter 信息发布服务器](#2-configure-costmap-filter-info-publisher-server)
  - [3. Enable Keepout Filter](#3-enable-keepout-filter)
  - [4. Run Nav2 stack](#4-run-nav2-stack)

# Overview


This tutorial shows how to simply utilize keep-out/safety zones where robots can\'t enter and preferred lanes for robots moving in industrial environments and warehouses. All this functionality is being covered by `KeepoutFilter` costmap filter plugin which will be enabled and used in this document.

> 这篇教程展示了如何简单地利用机器人不能进入的保护/安全区和工业环境和仓库中机器人移动的首选车道。所有这些功能都由`KeepoutFilter`成本地图过滤器插件提供，该插件将在本文中启用和使用。

# Requirements


It is assumed that ROS 2, Gazebo and TurtleBot3 packages are installed or built locally. Please make sure that Nav2 project is also built locally as it was made in `build-instructions`{.interpreted-text role="ref"}.

> 假设ROS 2、Gazebo和TurtleBot3软件包已安装或本地构建。 请确保Nav2项目也已本地构建，因为它是在`build-instructions`{.interpreted-text role="ref"}中制作的。

# Tutorial Steps

## 1. Prepare filter mask


As was written in `concepts`{.interpreted-text role="ref"}, any Costmap Filter (including Keepout Filter) are reading the data marked in a filter mask file. Filter mask - is the usual Nav2 2D-map distributed through PGM, PNG or BMP raster file with its metadata containing in a YAML file. The following steps help to understand how to make a new filter mask:

> 如`concepts`{.interpreted-text role="ref"}中所写，任何Costmap Filter（包括Keepout Filter）都是读取标记在过滤掩码文件中的数据。过滤掩码是通过PGM、PNG或BMP栅格文件及其元数据YAML文件分发的Nav2 2D地图的常见格式。以下步骤有助于理解如何制作新的过滤掩码：`Wait`


Create a new image with a PGM/PNG/BMP format: copy [turtlebot3_world.pgm](https://github.com/ros-planning/navigation2/blob/main/nav2_bringup/bringup/maps/turtlebot3_world.pgm) main map which will be used in a world simulation from a `Nav2` repository to a new `keepout_mask.pgm` file.

> 创建一个具有PGM/PNG/BMP格式的新图像：从Nav2存储库中复制[turtlebot3_world.pgm](https://github.com/ros-planning/navigation2/blob/main/nav2_bringup/bringup/maps/turtlebot3_world.pgm)主地图，用于世界模拟，并将其复制到新的`keepout_mask.pgm`文件中。


Open `keepout_mask.pgm` in your favourite raster graphics editor (as an example could be taken GIMP editor). The lightness of each pixel on the mask means an encoded information for the specific costmap filter you are going to use. Color lightness of each pixel belongs to the `[0..255]` range (or `[0..100]` in percent scale), where `0` means black color and `255` - white. Another term \"darkness\" will be understood as the exact opposite of lightness. In other words `color_darkness = 100% - color_lightness`.

> 打开你最喜欢的光栅图形编辑器（例如GIMP编辑器）中的`keepout_mask.pgm`。掩码上每个像素的亮度表示您将要使用的特定成本图过滤器的编码信息。每个像素的颜色亮度属于`[0..255]`范围（或百分比刻度为`[0..100]`），其中`0`表示黑色，`255`表示白色。另一个术语“黑暗”将被理解为亮度的完全相反。换句话说`color_darkness = 100％- color_lightness`。


In the GIMP lightness is expressed through color components value (e.g. `R` in percent scale) and might be set by moving `L` slider in color changing tool:

> 在GIMP中，亮度通过颜色分量值（例如以百分比表示的`R`）表示，可以通过移动颜色更改工具中的`L`滑块来设置：`Wait

![image](images/Navigation2_with_Keepout_Filter/ligtness_in_GIMP.png)


The incoming mask file is being read by the Map Server and converted into `OccupancyGrid` values from `[0..100]` range (where `0` means free cell, `100` - occupied, anything in between - less or more occupied cells on map) or be equal to `-1` for unknown value. In Nav2 stack each map has `mode` attribute which could be `trinary`, `scale` or `raw`. Depending on `mode` selected, the color lightness of PGM/PNG/BMP is being converted to `OccupancyGrid` by one of the following principles:

> 地图服务器正在读取传入的掩码文件，并将其转换为`[0..100]`范围内的`OccupancyGrid`值（其中`0`表示空白单元，`100`表示占用，其他任何值表示地图上占用程度较低或较高的单元），或者等于`-1`表示未知值。在Nav2堆栈中，每个地图都有`mode`属性，可以是`trinary`、`scale`或`raw`。根据所选择的`mode`，PGM/PNG/BMP的颜色亮度将按以下原则转换为`OccupancyGrid`：


- `trinary` (default mode): Darkness \>= `occupied_thresh` means that map occupied (`100`). Darkness \<= `free_thresh` - map free (`0`). Anything in between - unknown status on map (`-1`).

> 三元（默认模式）：黑暗度 >= `occupied_thresh` 表示地图已被占据（`100`）。黑暗度 <= `free_thresh` 表示地图是空闲的（`0`）。中间的任何值表示地图的状态未知（`-1`）。

- `scale`: Alpha \< `1.0` - unknown. Darkness \>= `occupied_thresh` means that map occupied (`100`). Darkness \<= `free_thresh` - map free (`0`). Anything in between - linearly interpolate to nearest integer from `[0..100]` range.

> `比例：Alpha < `1.0` - 未知。黑暗度 >= `occupied_thresh` 表示地图被占据（`100`）。黑暗度 <= `free_thresh` - 地图空闲（`0`）。中间任何值- 线性插值至最接近 `[0..100]` 范围的整数。`Wait`

- `raw`: Lightness = `0` (dark color) means that map is free (`0`). Lightness = `100` (in absolute value) - map is occupied (`100`). Anything in between - `OccupancyGrid` value = lightness. Lightness \>= `101` - unknown (`-1`).

> 亮度=`0`（暗色）意味着地图是空的（`0`）。亮度=`100`（绝对值）- 地图是占用的（`100`）。中间的任何东西- `OccupancyGrid`值=亮度。亮度> = `101`-未知（`-1`）。


where `free_thresh` and `occupied_thresh` thresholds are expressed in percentage of maximum lightness/darkness level (`255`). Map mode and thresholds are placed in YAML metadata file (see below) as `mode`, `free_thresh` and `occupied_thresh` fields.

> 在YAML元数据文件（见下文）中，`mode`，`free_thresh`和`occupied_thresh`字段表示地图模式和阈值，其中`free_thresh`和`occupied_thresh`阈值以最大亮度/黑暗级别（`255`）的百分比表示。

::: note
::: title
Note
:::


There is another parameter in a YAML metadata file called `negate`. By default it is set to `false`. When it is set to `true`, blacker pixels will be considered as free, whiter pixels - as occupied. In this case we should count color lightness instead of darkness for `trinary` and `scale` modes. `negate` has no effect on `raw` mode.

> 在YAML元数据文件中还有一个参数叫做`negate`。默认设置为`false`。当它设置为`true`时，更黑的像素被视为空闲，更白的像素被视为占用。在这种情况下，我们应该计算`trinary`和`scale`模式的颜色亮度，而不是黑暗度。`negate`对`raw`模式没有影响。

:::


For Keepout Filter `OccupancyGrid` value is proportional to the passibility of area corresponting to this cell: higher values means more impassable areas. Cells with occupied values covers keep-out zones where robot will never enter or pass through. `KeepoutFilter` can also act as a \"weighted areas layer\" by setting the `OccupancyGrid` to something between `[1-99]` non-occupied values. Robot is allowed to move in these areas, however its presence there would be \"undesirable\" there (the higher the value, the sooner planners will try to get the robot out of this area).

> `Keepout Filter` 的 `OccupancyGrid` 值与相应单元格的可通行性成比例：值越高意味着不可通行的区域越多。被占用的单元格表示机器人永远不会进入或穿过的保留区。`KeepoutFilter` 也可以通过将 `OccupancyGrid` 设置为 `[1-99]` 之间的非占用值来作为“加权区域层”来使用。机器人可以在这些区域移动，但其出现在那里是“不受欢迎的”（值越高，规划者就越快试图让机器人离开这个区域）。


Keepout Filter also covers preferred lanes case, where robots should moving only on pre-defined lanes and permitted areas e.g. in warehouses. To use this feaure you need to prepare the mask image where the lanes and permitted areas will be marked with free values while all other areas will be occupied. TIP for drawing the mask in a `trinary` or `scale` mode: typically, amount of pixels belonging to lanes are much less than pixels covering other areas. In this case initially all lanes data might be drawn with a black pencil over white background and then (just before saving a PGM) \"color inversion\" tool in a image raster editor might be used.

> 保持过滤器也覆盖优先车道的情况，其中机器人应该只在预定义的车道和允许区域移动，例如在仓库中。要使用此功能，您需要准备掩码图像，其中车道和允许区域将使用自由值标记，而所有其他区域将被占用。使用`三元`或`比例`模式绘制掩码的提示：通常，属于车道的像素数量比覆盖其他区域的像素少得多。在这种情况下，最初所有车道数据可以用黑色铅笔在白色背景上绘制，然后（在保存PGM之前）可以使用图像光栅编辑器中的“颜色反转”工具。


For simplicity, in the example fill the areas with black color (in `trinary` mode this means occupied map) that you are going to mark as a keep-out zones:

> 为了简化，在示例中，使用黑色填充区域（在`三元`模式下，这意味着占用的地图），您将标记为禁区：

![image](images/Navigation2_with_Keepout_Filter/drawing_keepout_mask.png){width="500px"}


After all keepout areas will be filled save the `keepout_mask.pgm` image.

> 等所有保护区域填满后，保存`keepout_mask.pgm`图像。


Like all other maps, filter mask should have its own YAML metadata file. Copy [turtlebot3_world.yaml](https://github.com/ros-planning/navigation2/blob/main/nav2_bringup/bringup/maps/turtlebot3_world.yaml) to `keepout_mask.yaml`. Open `keepout_mask.yaml` and correct `image` field to a newly made PGM mask:

> 像所有其他地图一样，过滤掩码应该有自己的YAML元数据文件。将[turtlebot3_world.yaml](https://github.com/ros-planning/navigation2/blob/main/nav2_bringup/bringup/maps/turtlebot3_world.yaml)复制到`keepout_mask.yaml`。打开`keepout_mask.yaml`并将`image`字段更正为新制作的PGM掩码：

    ```text
    image: turtlebot3_world.pgm
    ->
    image: keepout_mask.pgm
    ```


Since filter mask image was created as a copy of main map, other fields of YAML-file do not need to be changed. Save `keepout_mask.yaml` and new filter mask is ready to use.

> 由于过滤掩码图像是主图的副本，所以YAML文件的其他字段不需要更改。保存`keepout_mask.yaml`，新的过滤掩码就可以使用了。

::: note
::: title
Note
:::


World map itself and filter mask could have different sizes, origin and resolution which might be useful e.g. for cases when filter mask is covering smaller areas on maps or when one filter mask is used repeatedly many times (like annotating a keepout zone for same shape rooms in the hotel). For this case, you need to correct `resolution` and `origin` fields in YAML as well so that the filter mask is correctly laid on top of the original map.

> 世界地图本身和过滤器掩码可以有不同的尺寸、原点和分辨率，这可能很有用，例如地图上覆盖较小区域的情况，或者当一个过滤器掩码被多次重复使用（比如在酒店里注释同样形状的房间的禁区）。对于这种情况，您还需要在YAML中正确修改`分辨率`和`原点`字段，以便过滤器掩码正确地覆盖在原始地图上。
:::

::: note
::: title
Note
:::


Another important note is that since Costmap2D does not support orientation, the last third \"yaw\" component of the `origin` vector should be equal to zero. For example: `origin: [1.25, -5.18, 0.0]`.

> 另一个重要的注意事项是，由于Costmap2D不支持方向，`origin`向量的最后三个“偏航”分量应该等于零。例如：`origin：[1.25，-5.18，0.0]`。
:::

## 2. Configure Costmap Filter Info Publisher Server


Each costmap filter reads incoming meta-information (such as filter type or data conversion coefficients) in a messages of `nav2_msgs/CostmapFilterInfo` type. These messages are being published by [Costmap Filter Info Publisher Server](https://github.com/ros-planning/navigation2/tree/main/nav2_map_server/src/costmap_filter_info). The server is running as a lifecycle node. According to the [design document](https://github.com/ros-planning/navigation2/blob/main/doc/design/CostmapFilters_design.pdf), `nav2_msgs/CostmapFilterInfo` messages are going in a pair with `OccupancyGrid` filter mask topic. Therefore, along with Costmap Filter Info Publisher Server there should be enabled a new instance of Map Server configured to publish filter mask.

> 每个costmap过滤器都会读取传入的元信息（例如过滤器类型或数据转换系数），这些信息是以`nav2_msgs/CostmapFilterInfo`类型发布的。这些信息由[Costmap Filter Info Publisher Server](https://github.com/ros-planning/navigation2/tree/main/nav2_map_server/src/costmap_filter_info)发布。该服务器作为生命周期节点运行。根据[设计文档](https://github.com/ros-planning/navigation2/blob/main/doc/design/CostmapFilters_design.pdf)，`nav2_msgs/CostmapFilterInfo`消息与`OccupancyGrid`过滤器掩码主题成对发送。因此，除了Costmap Filter Info Publisher Server外，还应启用一个新的Map Server实例，配置为发布过滤器掩码。


In order to enable Keepout Filter in your configuration, both servers should be enabled as a lifecycle nodes in Python launch-file. For example, this might look as follows:

> 为了在您的配置中启用Keepout Filter，两台服务器都应该在Python launch-file中启用为生命周期节点。例如，可能如下所示：`Wait`

    ```python
    import os

    from ament_index_python.packages import get_package_share_directory

    from launch import LaunchDescription
    from launch.actions import DeclareLaunchArgument
    from launch.substitutions import LaunchConfiguration
    from launch_ros.actions import Node
    from nav2_common.launch import RewrittenYaml


    def generate_launch_description():
        # Get the launch directory
        costmap_filters_demo_dir = get_package_share_directory('nav2_costmap_filters_demo')

        # Create our own temporary YAML files that include substitutions
        lifecycle_nodes = ['filter_mask_server', 'costmap_filter_info_server']

        # Parameters
        namespace = LaunchConfiguration('namespace')
        use_sim_time = LaunchConfiguration('use_sim_time')
        autostart = LaunchConfiguration('autostart')
        params_file = LaunchConfiguration('params_file')
        mask_yaml_file = LaunchConfiguration('mask')

        # Declare the launch arguments
        declare_namespace_cmd = DeclareLaunchArgument(
            'namespace',
            default_value='',
            description='Top-level namespace')

        declare_use_sim_time_cmd = DeclareLaunchArgument(
            'use_sim_time',
            default_value='true',
            description='Use simulation (Gazebo) clock if true')

        declare_autostart_cmd = DeclareLaunchArgument(
            'autostart', default_value='true',
            description='Automatically startup the nav2 stack')

        declare_params_file_cmd = DeclareLaunchArgument(
                'params_file',
                default_value=os.path.join(costmap_filters_demo_dir, 'params', 'keepout_params.yaml'),
                description='Full path to the ROS 2 parameters file to use')

        declare_mask_yaml_file_cmd = DeclareLaunchArgument(
                'mask',
                default_value=os.path.join(costmap_filters_demo_dir, 'maps', 'keepout_mask.yaml'),
                description='Full path to filter mask yaml file to load')

        # Make re-written yaml
        param_substitutions = {
            'use_sim_time': use_sim_time,
            'yaml_filename': mask_yaml_file}

        configured_params = RewrittenYaml(
            source_file=params_file,
            root_key=namespace,
            param_rewrites=param_substitutions,
            convert_types=True)

        # Nodes launching commands
        start_lifecycle_manager_cmd = Node(
                package='nav2_lifecycle_manager',
                executable='lifecycle_manager',
                name='lifecycle_manager_costmap_filters',
                namespace=namespace,
                output='screen',
                emulate_tty=True,  # https://github.com/ros2/launch/issues/188
                parameters=[{'use_sim_time': use_sim_time},
                            {'autostart': autostart},
                            {'node_names': lifecycle_nodes}])

        start_map_server_cmd = Node(
                package='nav2_map_server',
                executable='map_server',
                name='filter_mask_server',
                namespace=namespace,
                output='screen',
                emulate_tty=True,  # https://github.com/ros2/launch/issues/188
                parameters=[configured_params])

        start_costmap_filter_info_server_cmd = Node(
                package='nav2_map_server',
                executable='costmap_filter_info_server',
                name='costmap_filter_info_server',
                namespace=namespace,
                output='screen',
                emulate_tty=True,  # https://github.com/ros2/launch/issues/188
                parameters=[configured_params])

        ld = LaunchDescription()

        ld.add_action(declare_namespace_cmd)
        ld.add_action(declare_use_sim_time_cmd)
        ld.add_action(declare_autostart_cmd)
        ld.add_action(declare_params_file_cmd)
        ld.add_action(declare_mask_yaml_file_cmd)

        ld.add_action(start_lifecycle_manager_cmd)
        ld.add_action(start_map_server_cmd)
        ld.add_action(start_costmap_filter_info_server_cmd)

        return ld
    ```


where the `params_file` variable should be set to a YAML-file having ROS parameters for Costmap Filter Info Publisher Server and Map Server nodes. These parameters and their meaning are listed at `configuring_map_server`{.interpreted-text role="ref"} page. Please, refer to it for more information. The example of `params_file` could be found below:

> 在哪里设置`params_file`变量？它应该被设置为一个YAML文件，其中包含ROS参数，用于Costmap Filter Info Publisher Server和Map Server节点。这些参数及其含义列在`configuring_map_server`{.interpreted-text role="ref"}页面上。有关更多信息，请参阅该页面。`params_file`的示例可以在下面找到： `Wait`

    ```yaml
    costmap_filter_info_server:
      ros__parameters:
        use_sim_time: true
        type: 0
        filter_info_topic: "/costmap_filter_info"
        mask_topic: "/keepout_filter_mask"
        base: 0.0
        multiplier: 1.0
    filter_mask_server:
      ros__parameters:
        use_sim_time: true
        frame_id: "map"
        topic_name: "/keepout_filter_mask"
        yaml_filename: "keepout_mask.yaml"
    ```

Note, that:

- For Keepout Filter the `type` of costmap filter should be set to `0`.

- Filter mask topic name should be the equal for `mask_topic` parameter of Costmap Filter Info Publisher Server and `topic_name` parameter of Map Server.

> `过滤掩码主题名称应该与Costmap Filter Info Publisher Server的`mask_topic`参数和Map Server的`topic_name`参数相等。`

- According to the Costmap Filters design, `OccupancyGrid` values are being linearly transformed into feature map in a filter space. For a Keepout Filter these values are directly passed as a filter space values without a linear conversion. Even though `base` and `multiplier` coefficients are not used in Keepout Filter, they should be set to `0.0` and `1.0` accordingly in order to explicitly show that we have one-to-one conversion from `OccupancyGrid` values -\> to a filter value space.

> 根据Costmap Filters设计，`OccupancyGrid`值正在被线性转换为滤波器空间中的特征图。对于Keepout Filter，这些值会直接作为滤波器空间的值而无需线性转换。尽管Keepout Filter中不使用`base`和`multiplier`系数，但为了明确表明我们有一对一的转换从`OccupancyGrid`值-\>到滤波器值空间，它们应该分别设置为`0.0`和`1.0`。


Ready-to-go standalone Python launch-script, YAML-file with ROS parameters and filter mask example for Keepout Filter could be found in a [nav2_costmap_filters_demo](https://github.com/ros-planning/navigation2_tutorials/tree/master/nav2_costmap_filters_demo) directory of `navigation2_tutorials` repository. To simply run Filter Info Publisher Server and Map Server tuned on Turtlebot3 standard simulation written at `getting_started`{.interpreted-text role="ref"}, build the demo and launch `costmap_filter_info.launch.py` as follows:

> 在navigation2_tutorials存储库的[nav2_costmap_filters_demo](https://github.com/ros-planning/navigation2_tutorials/tree/master/nav2_costmap_filters_demo)目录中可以找到一个可以直接使用的独立Python启动脚本、YAML文件以及Keepout Filter的示例掩码。要简单地运行在Turtlebot3标准模拟器上调整过的Filter Info Publisher Server和Map Server，请构建示例并如下启动`costmap_filter_info.launch.py`：`Wait`

    ```bash
    $ mkdir -p ~/tutorials_ws/src
    $ cd ~/tutorials_ws/src
    $ git clone https://github.com/ros-planning/navigation2_tutorials.git
    $ cd ~/tutorials_ws
    $ colcon build --symlink-install --packages-select nav2_costmap_filters_demo
    $ source ~/tutorials_ws/install/setup.bash
    $ ros2 launch nav2_costmap_filters_demo costmap_filter_info.launch.py params_file:=src/navigation2_tutorials/nav2_costmap_filters_demo/params/keepout_params.yaml mask:=src/navigation2_tutorials/nav2_costmap_filters_demo/maps/keepout_mask.yaml
    ```

## 3. Enable Keepout Filter


Costmap Filters are Costamp2D plugins. You can enable the `KeepoutFilter` plugin in Costmap2D by adding `keepout_filter` to the `plugins` parameter in `nav2_params.yaml`. You can place it in the `global_costmap` for planning with keepouts and `local_costmap` to make sure the robot won\'t attempt to drive through a keepout zone. The KeepoutFilter plugin should have the following parameters defined:

> Costmap过滤器是Costamp2D插件。您可以通过将`keepout_filter`添加到`nav2_params.yaml`中的`plugins`参数中来启用`KeepoutFilter`插件。您可以将其放置在`global_costmap`中以进行带有keepout的规划，以及`local_costmap`以确保机器人不会尝试穿过keepout区域。 KeepoutFilter插件应该定义以下参数：`

- `plugin`: type of plugin. In our case `nav2_costmap_2d::KeepoutFilter`.

- `filter_info_topic`: filter info topic name. This need to be equal to `filter_info_topic` parameter of Costmap Filter Info Publisher Server from the chapter above.

> `- `filter_info_topic`：过滤信息主题名称。这需要与上面章节中Costmap Filter Info Publisher Server的`filter_info_topic`参数相等。


Full list of parameters supported by `KeepoutFilter` are listed at `keepout_filter`{.interpreted-text role="ref"} page.

> 完整的`KeepoutFilter`支持的参数列表可在`keepout_filter`{.interpreted-text role="ref"}页面找到。


It is important to note that enabling `KeepoutFilter` for `global_costmap` only will cause the path planner to build plans bypassing keepout zones. Enabling `KeepoutFilter` for `local_costmap` only will cause the robot to not enter keepout zones, but the path may still go through them. So, the best practice is to enable `KeepoutFilter` for global and local costmaps simultaneously by adding it both in `global_costmap` and `local_costmap` in `nav2_params.yaml`. However it does not always have to be true. In some cases keepout zones don\'t have to be the same for global and local costmaps, e.g. if the robot doesn\'t allowed to intentionally go inside keepout zones, but if its there, the robot can drive in and out really quick if it clips an edge or corner. For this case, there is not need to use extra resources of the local costmap copy.

> 重要的是要注意，只启用 `global_costmap` 上的 `KeepoutFilter` 将会导致路径规划器构建计划绕过 keepout 区域。只启用 `local_costmap` 上的 `KeepoutFilter` 将会导致机器人不会进入 keepout 区域，但是路径仍可能穿过它们。因此，最佳实践是通过在 `nav2_params.yaml` 中的 `global_costmap` 和 `local_costmap` 上同时启用 `KeepoutFilter` 来同时启用它们。但是这并不总是正确的。在某些情况下，keepout 区域对于全局和本地代价地图不必相同，例如，如果机器人不允许有意进入 keepout 区域，但如果它在那里，机器人可以快速进入和退出，如果它削减了边缘或角落。对于这种情况，不需要使用本地代价地图副本的额外资源。


To enable `KeepoutFilter` with same mask for both global and local costmaps, use the following configuration:

> 要启用`KeepoutFilter`，使全局和本地Costmaps使用相同的掩码，请使用以下配置：`

    ```text
    global_costmap:
      global_costmap:
        ros__parameters:
          ...
          plugins: ["static_layer", "obstacle_layer", "inflation_layer"]
          filters: ["keepout_filter"]
          ...
          keepout_filter:
            plugin: "nav2_costmap_2d::KeepoutFilter"
            enabled: True
            filter_info_topic: "/costmap_filter_info"
    ...
    local_costmap:
      local_costmap:
        ros__parameters:
          ...
          plugins: ["voxel_layer", "inflation_layer"]
          filters: ["keepout_filter"]
          ...
          keepout_filter:
            plugin: "nav2_costmap_2d::KeepoutFilter"
            enabled: True
            filter_info_topic: "/costmap_filter_info"
    ```

::: note
::: title
Note
:::


All costmap filters should be enabled through a `filters` parameter \-- though it is technically possible to include in the layered costmap itself. This is separated from the layer plugins to prevent interference in the layers, particularly the inflation layer.

> 所有的costmap滤波器都应该通过一个`filters`参数来启用--尽管技术上可以把它们包括在分层costmap本身中。这与图层插件分离是为了防止它们之间的干扰，特别是扩张层。
:::

## 4. Run Nav2 stack


After Costmap Filter Info Publisher Server and Map Server were launched and Keepout Filter was enabled for global/local costmaps, run Nav2 stack as written in `getting_started`{.interpreted-text role="ref"}:

> 等待Costmap Filter Info Publisher Server和Map Server启动，并且为全局/局部costmaps启用Keepout Filter后，按照`getting_started`中的步骤运行Nav2 stack。

    ```bash
    ros2 launch nav2_bringup tb3_simulation_launch.py
    ```


And check that filter is working properly as in the pictures below (first picture shows keepout filter enabled for the global costmap, second - differently-sized `keepout_mask.pgm` filter mask):

> 请检查过滤器是否正常工作，如下图所示（第一张图显示全局成本地图上启用的keepout过滤器，第二张图显示不同大小的`keepout_mask.pgm`过滤器掩码）：

![image](images/Navigation2_with_Keepout_Filter/keepout_global.gif){height="400px"}

![image](images/Navigation2_with_Keepout_Filter/keepout_mask.png){height="400px"}
