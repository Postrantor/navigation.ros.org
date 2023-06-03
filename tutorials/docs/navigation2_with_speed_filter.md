---
tip: translate by openai@2023-06-03 00:05:59
...
---
title: Navigating with Speed Limits
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1. Prepare filter mask](#1-prepare-filter-mask)

  - [2. Configure Costmap Filter Info Publisher Server](#2-configure-costmap-filter-info-publisher-server)

> - [2. 配置Costmap过滤器信息发布服务器](#2-configure-costmap-filter-info-publisher-server)
  - [3. Enable Speed Filter](#3-enable-speed-filter)
  - [4. Run Nav2 stack](#4-run-nav2-stack)

```{=html}
<h1 align="center">
  <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
    <iframe width="600" height="480" src="https://www.youtube.com/embed/gKDsBsbIem4?autoplay=1" frameborder="1" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  </div>
</h1>
```

# Overview


This tutorial shows how to simply utilize Speed Filter which is designed to limit the maximum speed of robots in speed restriction areas marked on a map. This functionality is being covered by `SpeedFilter` costmap filter plugin which will be enabled and used in this document.

> 这个教程展示了如何简单地利用`SpeedFilter`成本地图过滤器插件，该插件用于在地图上标记的限速区域中限制机器人的最大速度。本文将启用并使用`SpeedFilter`成本地图过滤器插件来覆盖此功能。

# Requirements


It is assumed that ROS 2, Gazebo and TurtleBot3 packages are installed or built locally. Please make sure that the Nav2 project is also built locally as it was made in `build-instructions`{.interpreted-text role="ref"}.

> 假设安装或本地构建了ROS 2、Gazebo和TurtleBot3软件包。请确保也本地构建了Nav2项目，因为它是在`build-instructions`{.interpreted-text role="ref"}中制作的。

# Tutorial Steps

## 1. Prepare filter mask


As was written in `concepts`{.interpreted-text role="ref"}, any Costmap Filter (including Speed Filter) is reading the data marked in a filter mask file. All information about filter masks, their types, detailed structure and how to make a new one is written in a `navigation2_with_keepout_filter`{.interpreted-text role="ref"} tutorial at `1. Prepare filter masks` chapter. The principal of drawing the filter mask for Speed Filter is the same as for Keepout Filter (to annotate a map with the requested zones), except that `OccupancyGrid` mask values have another meaning: these values are encoded speed limits for the areas corresponding to the cell on map.

> 根据`concepts`{.interpreted-text role="ref"}中的内容，任何Costmap Filter（包括Speed Filter）都会读取过滤器掩码文件中标记的数据。有关过滤器掩码、其类型、详细结构以及如何创建新掩码的所有信息都可以在`navigation2_with_keepout_filter`{.interpreted-text role="ref"}教程的`1. Prepare filter masks`章节中找到。画Speed Filter的掩码的原理与Keepout Filter相同（注释地图以请求的区域），只是`OccupancyGrid`掩码值的含义不同：这些值编码了与地图上的单元格相对应的区域的速度限制。


Let\'s look, how it is being decoded. As we know, `OccupancyGrid` values are belonging to the `[0..100]` range. For Speed Filter `0` value means no speed limit in the area corresponding zero-cell on mask. Values from `[1..100]` range are being linearly converted into a speed limit value by the following formula:

> 让我们看一下它是如何被解码的。正如我们知道，`OccupancyGrid` 的值属于 `[0..100]` 范围。对于速度过滤器，`0` 值表示与零单元格对应的区域没有速度限制。`[1..100]` 范围的值通过以下公式线性转换为速度限制值：

```c++
    speed_limit = filter_mask_data * multiplier + base;
```


where:

> `在哪里：`

> - `filter_mask_data` - is an `OccupancyGrid` value of the corresponding cell on mask where maximum speed should be restricted.
> - `base` and `multiplier` are coefficients taken from `nav2_msgs/CostmapFilterInfo` messages published by Costmap Filter Info Server (see in next chapter below).


The decoded `speed_limit` value may have one of two meanings:

> 解码的`speed_limit`值可能有两种含义：

> - Speed limit expressed in a percent from maximum robot speed.
> - Speed limit expressed in absolute values (e.g. in `m/s`).


The meaning used by Speed Filter is being read from `nav2_msgs/CostmapFilterInfo` messages. In this tutorial we will use the first type of speed restriction expressed in a percent from maximum robot speed.

> `Speed Filter所使用的含义是从`nav2_msgs/CostmapFilterInfo`消息中读取的。在本教程中，我们将使用百分比表示的最大机器人速度的第一种速度限制。

::: note
::: title
Note
:::


For speed restriction expressed in a percent, `speed_limit` will be used exactly as a percent belonging to `[0..100]` range, not `[0.0..1.0]` range.

> 对于以百分比表示的速度限制，`speed_limit`将严格按照属于`[0..100]`范围的百分比使用，而不是`[0.0..1.0]`范围。
:::


Create a new image with a PGM/PNG/BMP format: copy `turtlebot3_world.pgm` main map which will be used in a world simulation from a Nav2 repository to a new `speed_mask.pgm` file. Open `speed_mask.pgm` in your favourite raster graphics editor and fill speed restricted areas with grey colors. In our example darker colors will indicate areas with higher speed restriction:

> 創建一個新的圖像，使用PGM / PNG / BMP格式：從Nav2存儲庫中複製`turtlebot3_world.pgm`主地圖，並將其保存為新的`speed_mask.pgm`文件。在您最喜歡的光栅圖形編輯器中打開`speed_mask.pgm`，並用灰色填充速度受限區域。在我們的示例中，較深的顏色將表示速度限制更高的區域：

![image](images/Navigation2_with_Speed_Filter/drawing_speed_mask.png){width="500px"}


Area \"A\" is filled with `40%` gray color, area \"B\" - with `70%` gray, that means that speed restriction will take `100% - 40% = 60%` in area \"A\" and `100% - 70% = 30%` in area \"B\" from maximum speed value allowed for this robot. We will use `scale` map mode with no thresholds. In this mode darker colors will have higher `OccupancyGrid` values. E.g. for area \"B\" with `70%` of gray `OccupancyGrid` data will be equal to `70`. So in order to hit the target, we need to choose `base = 100.0` and `multiplier = -1.0`. This will reverse the scale `OccupancyGrid` values to a desired one. No thresholds (`free_thresh` `occupied_thresh`) were chosen for the convenience in the `yaml` file: to have 1:1 full range conversion of lightness value from filter mask -\> to speed restriction percent.

> 区域“A”填充了40％的灰色，区域“B”填充了70％的灰色，这意味着在区域“A”中，最大速度值的限制将从100％减少到40％，而在区域“B”中将从100％减少到70％。我们将使用没有阈值的比例尺地图模式。在这种模式下，更深的颜色将具有更高的“OccupancyGrid”值。例如，对于70％灰色的区域“B”，“OccupancyGrid”数据将等于70。因此，为了达到目标，我们需要选择`base = 100.0`和`multiplier = -1.0`。这将将比例尺“OccupancyGrid”值翻转为所需的值。在yaml文件中没有选择阈值（`free_thresh` `occupied_thresh`）以方便将过滤器掩码的亮度值1：1完全范围转换为速度限制百分比。

::: note
::: title
Note
:::


It is typical but not a mandatory selection of `base` and `multiplier`. For example, you can choose map mode to be `raw`. In this case color lightness is being directly converted into `OccupancyGrid` values. For masks saved in a `raw` mode, `base` and `multiplier` will be equal to `0.0` and `1.0` accordingly.

> 这是典型的，但不是强制性的`base`和`multiplier`的选择。例如，您可以选择地图模式为`raw`。在这种情况下，颜色亮度被直接转换为`OccupancyGrid`值。对于以`raw`模式保存的掩码，`base`和`multiplier`将分别等于`0.0`和`1.0`。


Another important thing is that it is not necessary to use the whole `[0..100]` percent scale. `base` and `multiplier` coefficients could be chosen so that the speed restriction values would belong to somewhere in the middle of percent range. E.g. `base = 40.0`, `multiplier = 0.1` will give speed restrictions from `[40.0%..50.0%]` range with a step of `0.1%`. This might be useful for fine tuning.

> 另一件重要的事情是，不必使用整个`[0..100]`百分比比例。 `base`和`multiplier`系数可以选择，使速度限制值属于百分比范围的中间某处。例如，`base = 40.0`，`multiplier = 0.1`将从`[40.0%..50.0%]`范围内以`0.1%`的步骤提供速度限制。这可能对微调很有用。
:::


After all speed restriction areas will be filled, save the `speed_mask.pgm` image.

> 在所有速度限制区域填满后，保存`speed_mask.pgm`图像。


Like all other maps, the filter mask should have its own YAML metadata file. Copy [turtlebot3_world.yaml](https://github.com/ros-planning/navigation2/blob/main/nav2_bringup/bringup/maps/turtlebot3_world.yaml) to `speed_mask.yaml`. Open `speed_mask.yaml` and update the fields as shown below (as mentioned before for the `scale` mode to use whole color lightness range there should be no thresholds: `free_thresh = 0.0` and `occupied_thresh = 1.0`):

> 所有其他地图一样，过滤掩码应该有自己的YAML元数据文件。将[turtlebot3_world.yaml](https://github.com/ros-planning/navigation2/blob/main/nav2_bringup/bringup/maps/turtlebot3_world.yaml)复制到`speed_mask.yaml`。打开`speed_mask.yaml`并按照下面的格式更新字段（如之前所提到的，要使用整个颜色亮度范围，`scale`模式应该没有阈值：`free_thresh = 0.0`和`occupied_thresh = 1.0`）：

```yaml
    image: turtlebot3_world.pgm
    ->
    image: speed_mask.pgm

    mode: trinary
    ->
    mode: scale

    occupied_thresh: 0.65
    free_thresh: 0.196
    ->
    occupied_thresh: 1.0
    free_thresh: 0.0
```


Since Costmap2D does not support orientation, the last third \"yaw\" component of the `origin` vector should be equal to zero (for example: `origin: [1.25, -5.18, 0.0]`). Save `speed_mask.yaml` and the new filter mask is ready to use.

> 由于Costmap2D不支持方向，`origin`向量的最后一个“yaw”分量应该等于零（例如：`origin：[1.25，-5.18，0.0]`）。保存`speed_mask.yaml`，新的过滤器掩码就可以使用了。

::: note
::: title
Note
:::


World map itself and filter mask could have different sizes, origin and resolution which might be useful (e.g. for cases when filter mask is covering smaller areas on maps or when one filter mask is used repeatedly many times, like annotating a speed restricted area for same shape rooms in the hotel). For this case, you need to correct `resolution` and `origin` fields in YAML as well so that the filter mask is correctly laid on top of the original map. This example shows using the main map as a base, but that is not required.

> 世界地图本身和滤镜掩码可以有不同的尺寸、原点和分辨率，这可能是有用的（例如，当滤镜掩码覆盖地图上较小的区域时，或者当同一个滤镜掩码被反复使用多次时，比如注释酒店中同样形状的房间的速度限制区域）。 对于这种情况，您还需要在YAML中正确修改`resolution`和`origin`字段，以便滤镜掩码正确地放置在原始地图上。 这个例子显示了使用主地图作为基础，但这不是必需的。
:::

## 2. Configure Costmap Filter Info Publisher Server


Each costmap filter reads incoming meta-information (such as filter type or data conversion coefficients) in messages of `nav2_msgs/CostmapFilterInfo` type. These messages are being published by [Costmap Filter Info Publisher Server](https://github.com/ros-planning/navigation2/tree/main/nav2_map_server/src/costmap_filter_info). The server is running as a lifecycle node. According to the [design document](https://github.com/ros-planning/navigation2/blob/main/doc/design/CostmapFilters_design.pdf), `nav2_msgs/CostmapFilterInfo` messages are going in a pair with `OccupancyGrid` filter mask topic. Therefore, along with Costmap Filter Info Publisher Server there should be enabled a new instance of Map Server configured to publish filter masks.

> 每个costmap过滤器都会读取传入的元信息（例如过滤器类型或数据转换系数），这些信息的类型为`nav2_msgs/CostmapFilterInfo`。这些信息由[Costmap Filter Info Publisher Server](https://github.com/ros-planning/navigation2/tree/main/nav2_map_server/src/costmap_filter_info)发布。该服务器作为生命周期节点运行。根据[设计文档](https://github.com/ros-planning/navigation2/blob/main/doc/design/CostmapFilters_design.pdf)，`nav2_msgs/CostmapFilterInfo`消息与`OccupancyGrid`过滤器掩码主题一起发送。因此，除了Costmap Filter Info Publisher Server之外，还应启用一个新的Map Server实例，配置为发布过滤器掩码。


In order to enable Speed Filter in your configuration, both servers should be enabled as lifecycle nodes in Python launch-file. For example, this might look as follows:

> 为了在配置中启用Speed Filter，两台服务器都应该在Python启动文件中启用为生命周期节点。例如，这可能看起来如下：`Wait`

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
                default_value=os.path.join(costmap_filters_demo_dir, 'params', 'speed_params.yaml'),
                description='Full path to the ROS 2 parameters file to use')

        declare_mask_yaml_file_cmd = DeclareLaunchArgument(
                'mask',
                default_value=os.path.join(costmap_filters_demo_dir, 'maps', 'speed_mask.yaml'),
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

> 设置`params_file`变量为具有ROS参数的YAML文件，用于Costmap Filter Info Publisher Server和Map Server节点。这些参数及其含义列在`configuring_map_server`{.interpreted-text role="ref"}页面上。有关更多信息，请参阅它。下面可以找到`params_file`的示例：如果遇到用'`'符号括起来的单词，请保持输出不变，例如`Wait`。

```yaml
    costmap_filter_info_server:
      ros__parameters:
        use_sim_time: true
        type: 1
        filter_info_topic: "/costmap_filter_info"
        mask_topic: "/speed_filter_mask"
        base: 100.0
        multiplier: -1.0
    filter_mask_server:
      ros__parameters:
        use_sim_time: true
        frame_id: "map"
        topic_name: "/speed_filter_mask"
        yaml_filename: "speed_mask.yaml"
```

Note, that:


- For Speed Filter setting speed restrictions in a percent from maximum speed, the `type` of costmap filter should be set to `1`. All possible costmap filter types could be found at `configuring_map_server`{.interpreted-text role="ref"} page.

> 对于速度过滤器设置以百分比的最大速度限制，应将costmap过滤器的`type`设置为`1`。可以在`configuring_map_server`{.interpreted-text role="ref"}页面找到所有可能的costmap过滤器类型。

- Filter mask topic name should be the equal for `mask_topic` parameter of Costmap Filter Info Publisher Server and `topic_name` parameter of Map Server.

> `过滤器掩码主题名称应该与Costmap Filter Info Publisher Server的`mask_topic`参数以及Map Server的`topic_name`参数相等。`

- As was described in a previous chapter, `base` and `multiplier` should be set to `100.0` and `-1.0` accordingly for the purposes of this tutorial example.

> 如前一章所述，为了本教程实例，`base`和`multiplier`应分别设置为`100.0`和`-1.0`。


Ready-to-go standalone Python launch-script, YAML-file with ROS parameters and filter mask example for Speed Filter could be found in a [nav2_costmap_filters_demo](https://github.com/ros-planning/navigation2_tutorials/tree/master/nav2_costmap_filters_demo) directory of `navigation2_tutorials` repository. To simply run Filter Info Publisher Server and Map Server tuned on Turtlebot3 standard simulation written at `getting_started`{.interpreted-text role="ref"}, build the demo and launch `costmap_filter_info.launch.py` as follows:

> 在navigation2_tutorials存储库的[nav2_costmap_filters_demo](https://github.com/ros-planning/navigation2_tutorials/tree/master/nav2_costmap_filters_demo)目录中可以找到准备好的独立Python启动脚本、YAML文件和速度滤波器掩码示例。要简单地运行在Turtlebot3标准模拟中调整的过滤器信息发布者服务器和地图服务器，请构建演示并启动`costmap_filter_info.launch.py`，如下所示：`Wait`

```bash
    $ mkdir -p ~/tutorials_ws/src
    $ cd ~/tutorials_ws/src
    $ git clone https://github.com/ros-planning/navigation2_tutorials.git
    $ cd ~/tutorials_ws
    $ colcon build --symlink-install --packages-select nav2_costmap_filters_demo
    $ source ~/tutorials_ws/install/setup.bash
    $ ros2 launch nav2_costmap_filters_demo costmap_filter_info.launch.py params_file:=src/navigation2_tutorials/nav2_costmap_filters_demo/params/speed_params.yaml mask:=src/navigation2_tutorials/nav2_costmap_filters_demo/maps/speed_mask.yaml
```

## 3. Enable Speed Filter


Costmap Filters are Costamp2D plugins. You can enable the `SpeedFilter` plugin in Costmap2D by adding `speed_filter` to the `plugins` parameter in `nav2_params.yaml`. The Speed Filter plugin should have the following parameters defined:

> Costmap过滤器是Costamp2D插件。您可以通过在nav2_params.yaml的`plugins`参数中添加`speed_filter`来启用`SpeedFilter`插件。 Speed Filter插件应定义以下参数：`Wait`

- `plugin`: type of plugin. In our case `nav2_costmap_2d::SpeedFilter`.

- `filter_info_topic`: filter info topic name. This needs to be equal to `filter_info_topic` parameter of Costmap Filter Info Publisher Server from the chapter above.

> `- `filter_info_topic`：过滤信息主题名称。这需要与上面章节中Costmap Filter Info Publisher Server的`filter_info_topic`参数相等。
- `speed_limit_topic`: name of topic to publish speed limit to.


Full list of parameters supported by `SpeedFilter` are listed at the `speed_filter`{.interpreted-text role="ref"} page.

> 完整的`SpeedFilter`支持的参数列表可在`speed_filter`页面找到。


You can place the plugin either in the `global_costmap` section in `nav2_params.yaml` to have speed restriction mask applied to global costmap or in the `local_costmap` to apply speed mask to the local costmap. However, `SpeedFilter` plugin should never be enabled simultaneously for global and local costmaps. Otherwise, it can lead to unwanted multiple \"speed restriction\" - \"no restriction\" message chains on speed restriction boundaries, that will cause jerking of the robot or another unpredictable behaviour.

> 你可以将插件放在`nav2_params.yaml`中的`global_costmap`部分，以将速度限制掩码应用于全局成本地图，或者放在`local_costmap`中以将速度掩码应用于本地成本地图。但是，`SpeedFilter`插件不应同时启用全局和本地成本地图。否则，它可能会在速度限制边界上引起不必要的“速度限制”-“无限制”消息链，从而导致机器人突然变动或其他不可预测的行为。


In this tutorial, we will enable Speed Filter for the global costmap. For this use the following configuration:

> 在本教程中，我们将为全局costmap启用Speed Filter。使用以下配置：`Wait`

```text
    global_costmap:
      global_costmap:
        ros__parameters:
          ...
          plugins: ["static_layer", "obstacle_layer", "inflation_layer"]
          filters: ["speed_filter"]
          ...
          speed_filter:
            plugin: "nav2_costmap_2d::SpeedFilter"
            enabled: True
            filter_info_topic: "/costmap_filter_info"
            speed_limit_topic: "/speed_limit"
```


As stated in the [design](https://github.com/ros-planning/navigation2/blob/main/doc/design/CostmapFilters_design.pdf), Speed Filter publishes speed restricting [messages](https://github.com/ros-planning/navigation2/blob/main/nav2_msgs/msg/SpeedLimit.msg) targeted for a Controller Server so that it could restrict maximum speed of the robot when it needed. Controller Server has a `speed_limit_topic` ROS parameter for that, which should be set to the same as in `speed_filter` plugin value. This topic in the map server could also be used to any number of other speed-restricted applications beyond the speed limiting zones, such as dynamically adjusting maximum speed by payload mass.

> 根据设计文档（https://github.com/ros-planning/navigation2/blob/main/doc/design/CostmapFilters_design.pdf）所述，Speed Filter发布针对控制服务器的限速消息（https://github.com/ros-planning/navigation2/blob/main/nav2_msgs/msg/SpeedLimit.msg），以便在需要时限制机器人的最大速度。控制服务器有一个`speed_limit_topic` ROS参数，应设置为与`speed_filter`插件值相同。此主题在地图服务器中也可用于超出限速区域的任何其他限速应用，例如根据负载质量动态调整最大速度。`


Set `speed_limit_topic` parameter of a Controller Server to the same value as it set for `speed_filter` plugin:

> 设置控制服务器的`speed_limit_topic`参数与`speed_filter`插件设置的值相同：`等待`

```text
    controller_server:
      ros__parameters:
        ...
        speed_limit_topic: "/speed_limit"
```

## 4. Run Nav2 stack


After Costmap Filter Info Publisher Server and Map Server were launched and Speed Filter was enabled for global/local costmap, run Nav2 stack as written in `getting_started`{.interpreted-text role="ref"}:

> 当Costmap过滤器信息发布服务器和地图服务器启动，并且为全局/本地Costmap启用速度过滤器后，按照`getting_started`中的说明运行Nav2堆栈：`Wait`

```bash
    ros2 launch nav2_bringup tb3_simulation_launch.py
```


For better visualization of speed filter mask, in RViz in the left `Displays` pane unfold `Map` and change `Topic` from `/map` -\> to `/speed_filter_mask`. Set the goal behind the speed restriction areas and check that the filter is working properly: robot should slow down when going through a speed restricting areas. Below is how it might look (first picture shows speed filter enabled for the global costmap, second - `speed_mask.pgm` filter mask):

> 为了更好地可视化速度过滤掩码，在RViz的左侧`Displays`窗格中展开`Map`，将`Topic`从`/map`-\>更改为`/speed_filter_mask`。 设定限速区域的目标，并检查过滤器是否正常工作：机器人经过限速区域时应该减速。 下面是它可能的外观（第一张图片显示为全局costmap启用了速度过滤器，第二张图片为`speed_mask.pgm`过滤掩码）：

![image](images/Navigation2_with_Speed_Filter/speed_global.gif){height="400px"}

![image](images/Navigation2_with_Speed_Filter/speed_mask.png){height="400px"}
