---
tip: translate by openai@2023-06-03 00:47:17
...
---
title: Writing a New Costmap2D Plugin
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1- Write a new Costmap2D plugin](#1--write-a-new-costmap2d-plugin)
  - [2- Export and make GradientLayer plugin](#2--export-and-make-gradientlayer-plugin)
  - [3- Enable the plugin in Costmap2D](#3--enable-the-plugin-in-costmap2d)
  - [4- Run GradientLayer plugin](#4--run-gradientlayer-plugin)

![Animated gif with gradient demo](images/Writing_new_Costmap2D_plugin/gradient_layer_preview.gif){.align-center width="700px"}

# Overview


This tutorial shows how to create your own simple [plugin](http://wiki.ros.org/pluginlib) for Costmap2D.

> 这个教程展示了如何为Costmap2D创建自己的简单`plugin`。


Before starting the tutorial, please check this [video](https://vimeo.com/106994708) which contains information about Costmap2D layers design and plugins basic operational principals.

> 在开始教程之前，请查看此[视频](https://vimeo.com/106994708)，其中包含有关Costmap2D图层设计和插件基本操作原理的信息。

# Requirements


It is assumed that ROS 2, Gazebo and TurtleBot3 packages are installed or built locally. Please make sure that Navigation2 project is also built locally as it was made in `build-instructions`{.interpreted-text role="ref"}.

> 假設ROS 2、Gazebo和TurtleBot3軟件包已安裝或在本地構建。請確保Navigation2項目也在本地構建，就像在`build-instructions`{.interpreted-text role="ref"}中所做的那樣。

# Tutorial Steps

## 1- Write a new Costmap2D plugin


For a demonstration, this example will creates a costmap plugin that puts repeating costs gradients in the costmap. The annotated code for this tutorial can be found in [navigation2_tutorials](https://github.com/ros-planning/navigation2_tutorials) repository as the `nav2_gradient_costmap_plugin` ROS 2-package. Please refer to it when making your own layer plugin for Costmap2D.

> 为了演示，这个例子将创建一个costmap插件，它将重复的成本梯度放入costmap中。此教程的注释代码可在[navigation2_tutorials](https://github.com/ros-planning/navigation2_tutorials)存储库中找到，作为`nav2_gradient_costmap_plugin` ROS 2-package。在制作Costmap2D的自己的层插件时，请参阅它。


The plugin class `nav2_gradient_costmap_plugin::GradientLayer` is inherited from basic class `nav2_costmap_2d::Layer`:

> 插件类`nav2_gradient_costmap_plugin::GradientLayer`继承自基础类`nav2_costmap_2d::Layer`：

    ```c
    namespace nav2_gradient_costmap_plugin
    {

    class GradientLayer : public nav2_costmap_2d::Layer
    ```


The basic class provides the set of virtual methods API for working with costmap layers in a plugin. These methods are calling in runtime by `LayeredCostmap`. The list of methods, their description and necessity to have these methods in plugin\'s code is presented in the table below:

> 基础类提供了用于在插件中处理成本图层的虚拟方法API集。 这些方法在运行时由`LayeredCostmap`调用。 下表列出了方法的列表，其描述以及插件代码中必须具有这些方法的必要性：

---


**Virtual method** **Method description** **Requires override?**

> **虚拟方法** **方法描述** **需要重写？**


onInitialize() Method is called at the end of plugin initialization. There is usually declarations of ROS parameters. This is where any required initialization should occur. No

> `onInitialize() 方法在插件初始化结束时被调用。通常会声明ROS参数。这是应该发生任何必要初始化的地方。`


updateBounds() Method is called to ask the plugin: which area of costmap layer it needs to update. There are 3 input parameters of method: robot position and orientation and 4 output parameters: pointers to window bounds. These bounds are used for performance reasons: to update the area inside the window where is new info available, avoiding updates of whole costmap on every iteration. Yes

> updateBounds() 方法被调用以询问插件：它需要更新哪个costmap层的区域。该方法有3个输入参数：机器人的位置和方向，以及4个输出参数：指向窗口边界的指针。这些边界是出于性能原因而使用的：更新窗口内新信息可用的区域，避免每次迭代时更新整个costmap。是的。


updateCosts() Method is called each time when costmap re-calculation is required. It updates the costmap layer only within its bounds window. There are 4 input parameters of method: calculation window bounds and 1 output parameter: reference to a resulting costmap `master_grid`. The `Layer` class provides the plugin with an internal costmap, `costmap_`, for updates. The `master_grid` should be updated with values within the window bounds using one of the following update methods: `updateWithAddition()`, `updateWithMax()`, `updateWithOverwrite()` or `updateWithTrueOverwrite()`. Yes

> `updateCosts()` 方法每次需要重新计算costmap时调用。它仅在其边界窗口内更新costmap层。该方法有4个输入参数：计算窗口边界和1个输出参数：对结果costmap `master_grid`的引用。 `Layer` 类为插件提供了内部costmap `costmap_` 以进行更新。应使用以下更新方法之一在窗口边界内更新 `master_grid`：`updateWithAddition()`、`updateWithMax()`、`updateWithOverwrite()` 或 `updateWithTrueOverwrite()`。是的。


matchSize() Method is called each time when map size was changed. No

> `matchSize() 方法每次地图大小改变时被调用。`Wait`


onFootprintChanged() Method is called each time when footprint was changed. No

> 当足迹发生变化时，`onFootprintChanged()` 方法就会被调用。


reset() It may have any code to be executed during costmap reset. Yes

> `重置() 在成本地图重置期间可能有任何要执行的代码。是的`

---


In our example these methods have the following functionality:

> 在我们的例子中，这些方法具有以下功能：


1.  `GradientLayer::onInitialize()` contains declaration of ROS parameter with its default value:

> GradientLayer::onInitialize()中包含ROS参数的声明及其默认值：`Wait`

    ```c
    declareParameter("enabled", rclcpp::ParameterValue(true));
    node_->get_parameter(name_ + "." + "enabled", enabled_);
    ```


and sets `need_recalculation_` bounds recalculation indicator:

> 并设置`need_recalculation_`界限重新计算指示器：`Wait`

    ```c
    need_recalculation_ = false;
    ```


2.  `GradientLayer::updateBounds()` re-calculates window bounds if `need_recalculation_` is `true` and updates them regardless of `need_recalculation_` value.

> GradientLayer::updateBounds()，如果need_recalculation_为true，就重新计算窗口边界，不管need_recalculation_的值如何，都会更新它们。

3.  `GradientLayer::updateCosts()` - in this method the gradient is writing directly to the resulting costmap `master_grid` without merging with previous layers. This is equal to working with internal `costmap_` and then calling `updateWithTrueOverwrite()` method. Here is the gradient making algorithm for master costmap:

> 在这个方法中，梯度直接写入结果costmap `master_grid`，而不需要与之前的图层合并。这等同于使用内部`costmap_`，然后调用`updateWithTrueOverwrite()`方法。这里是主costmap的梯度制作算法：

    ```c
    int gradient_index;
    for (int j = min_j; j < max_j; j++) {
      // Reset gradient_index each time when reaching the end of re-calculated window
      // by OY axis.
      gradient_index = 0;
      for (int i = min_i; i < max_i; i++) {
        int index = master_grid.getIndex(i, j);
        // setting the gradient cost
        unsigned char cost = (LETHAL_OBSTACLE - gradient_index*GRADIENT_FACTOR)%255;
        if (gradient_index <= GRADIENT_SIZE) {
          gradient_index++;
        } else {
          gradient_index = 0;
        }
        master_array[index] = cost;
      }
    }
    ```


where the `GRADIENT_SIZE` is the size of each gradient period in map cells, `GRADIENT_FACTOR` - decrement of costmap\'s value per each step:

> 在地图格子中，`GRADIENT_SIZE`是每个梯度周期的大小，`GRADIENT_FACTOR`是每一步骤成本地图的减少量。

![image](images/Writing_new_Costmap2D_plugin/gradient_explanation.png)


These parameters are defined in plugin\'s header file.

> 这些参数定义在插件的头文件中。


4.  `GradientLayer::onFootprintChanged()` just resets `need_recalculation_` value.

> GradientLayer::onFootprintChanged() 只是重置 `need_recalculation_` 的值。

5.  `GradientLayer::reset()` method is dummy: it is not used in this example plugin. It remaining there since pure virtual function `reset()` in parent `Layer` class required to be overriden.

> GradientLayer::reset()方法是虚拟的：在这个示例插件中没有使用它。它仍然存在，因为父类Layer中的纯虚函数reset()需要被覆盖。

## 2- Export and make GradientLayer plugin


The written plugin will be loaded in runtime as it\'s basic parent class and then will be called by plugin handling modules (for costmap2d by `LayeredCostmap`). Pluginlib opens a given plugin in run-time and provides methods from exported classes to be callable. The mechanism of class exporting tells pluginlib which basic class should be used during these calls. This allows to extend an application by plugins without knowing application source code or recompiling it.

> 插件将在运行时加载，作为它的基本父类，然后由插件处理模块（对于costmap2d由`LayeredCostmap`调用）调用。 Pluginlib在运行时打开给定的插件，并提供可调用的导出类的方法。类导出机制告诉pluginlib在这些调用期间应使用哪个基本类。这允许通过插件而不知道应用程序源代码或重新编译它来扩展应用程序。


In our example the `nav2_gradient_costmap_plugin::GradientLayer` plugin\'s class should be dynamically loaded as a `nav2_costmap_2d::Layer` basic class. For this the plugin should be registered as follows:

> 在我们的示例中，`nav2_gradient_costmap_plugin::GradientLayer`插件的类应该作为`nav2_costmap_2d::Layer`基类动态加载。为此，插件应该如下注册：`Wait`


1.  Plugin\'s class should be registered with a basic type of loaded class. For this there is a special macro `PLUGINLIB_EXPORT_CLASS` should be added to any source-file composing the plugin library:

> `插件类应该使用已加载的基本类型进行注册。为此，应将特殊宏`PLUGINLIB_EXPORT_CLASS`添加到组成插件库的任何源文件中：`

    ```text
    #include "pluginlib/class_list_macros.hpp"
    PLUGINLIB_EXPORT_CLASS(nav2_gradient_costmap_plugin::GradientLayer, nav2_costmap_2d::Layer)
    ```


This part is usually placed at the end of cpp-file where the plugin class was written (in our example `gradient_layer.cpp`). It is good practice to place these lines at the end of the file but technically, you can also place at the top.

> 这部分通常放在插件类被编写的cpp文件的末尾（在我们的示例`gradient_layer.cpp`中）。把这些行放在文件末尾是一个好的做法，但从技术上讲，也可以放在顶部。


2.  Plugin\'s inormation should be stored to plugin description file. This is done by using separate XML (in our example `gradient_plugins.xml`) in the plugin\'s package. This file contains information about:

> 2. 插件信息应存储在插件描述文件中。 这是通过在插件包中使用单独的XML（在我们的示例`gradient_plugins.xml`）来实现的。此文件包含以下信息：`Wait`

> - `path`: Path and name of library where plugin is placed.
> - `name`: Plugin type referenced in `plugin_types` parameter (see next section for more details). It could be whatever you want.
> - `type`: Plugin class with namespace taken from the source code.
> - `basic_class_type`: Basic parent class from which plugin class was derived.
> - `description`: Plugin description in a text form.

    ```xml
    <library path="nav2_gradient_costmap_plugin_core">
      <class name="nav2_gradient_costmap_plugin/GradientLayer" type="nav2_gradient_costmap_plugin::GradientLayer" base_class_type="nav2_costmap_2d::Layer">
        <description>This is an example plugin which puts repeating costs gradients to costmap</description>
      </class>
    </library>
    ```


The export of plugin is performed by including `pluginlib_export_plugin_description_file()` cmake-function into `CMakeLists.txt`. This function installs plugin description file into `share` directory and sets ament indexes for plugin description XML to be discoverable as a plugin of selected type:

> 导出插件可以通过在`CMakeLists.txt`中包含`pluginlib_export_plugin_description_file()` cmake函数来完成。该函数将插件描述文件安装到`share`目录，并为插件描述XML设置ament索引，使其作为所选类型的插件可被发现。

    ```text
    pluginlib_export_plugin_description_file(nav2_costmap_2d gradient_layer.xml)
    ```


Plugin description file is also should be added to `package.xml`. `costmap_2d` is the package of the interface definition, for our case `Layer`, and requires a path to the xml file:

> 插件描述文件也应该添加到`package.xml`中。 `costmap_2d`是接口定义的包，对于我们的例子`Layer`，需要一个到xml文件的路径：`

    ```text
    <export>
      <costmap_2d plugin="${prefix}/gradient_layer.xml" />
      ...
    </export>
    ```


After everything is done put the plugin package into `src` directory of a certain ROS 2-workspace, build the plugin package (`colcon build --packages-select nav2_gradient_costmap_plugin --symlink-install`) and source `setup.bash` file when it necessary.

> 完成所有事情后，将插件包放入某个ROS 2工作空间的`src`目录中，构建插件包（`colcon build --packages-select nav2_gradient_costmap_plugin --symlink-install`），并在必要时源`setup.bash`文件。


Now the plugin is ready to use.

> 现在插件已经准备好使用了。

## 3- Enable the plugin in Costmap2D


At the next step it is required to tell Costmap2D about new plugin. For that the plugin should be added to `plugin_names` and `plugin_types` lists in `nav2_params.yaml` optionally for `local_costmap`/`global_costmap` in order to be enabled in run-time for Controller/Planner Server. `plugin_names` list contains the names of plugin objects. These names could be anything you want. `plugin_types` contains types of listed in `plugin_names` objects. These types should correspond to `name` field of plugin class specified in plugin description XML-file.

> 在下一步，需要告诉Costmap2D有关新插件的信息。为此，在`nav2_params.yaml`中，应将插件添加到`plugin_names`和`plugin_types`列表中，可选择针对`local_costmap`/`global_costmap`以在运行时为控制器/规划服务器启用。`plugin_names`列表包含插件对象的名称。这些名称可以是您想要的任何内容。`plugin_types`包含列出在插件描述XML文件中指定的插件类的`name`字段中的类型。

::: note
::: title
Note
:::


For Galactic or later, `plugin_names` and `plugin_types` have been replaced with a single `plugins` string vector for plugin names. The types are now defined in the `plugin_name` namespace in the `plugin:` field (e.g. `plugin: MyPlugin::Plugin`). Inline comments in the code blocks will help guide you through this.

> 对于Galactic或更高版本，`plugin_names`和`plugin_types`已被单一的`plugins`字符串向量所取代，用于插件名称。类型现在定义在`plugin_name`命名空间中的`plugin：`字段（例如`plugin：MyPlugin :: Plugin`）。代码块中的内联注释将帮助您指引此过程。`Wait`
:::


For example:

> 例如：`Wait`

    ```diff
    --- a/nav2_bringup/bringup/params/nav2_params.yaml
    +++ b/nav2_bringup/bringup/params/nav2_params.yaml
    @@ -124,8 +124,8 @@ local_costmap:
          width: 3
          height: 3
          resolution: 0.05
    -      plugins: ["obstacle_layer", "voxel_layer", "inflation_layer"]
    +      plugins: ["obstacle_layer", "voxel_layer", "gradient_layer"]
          robot_radius: 0.22
          inflation_layer:
            cost_scaling_factor: 3.0
    @@ -171,8 +171,8 @@ global_costmap:
          robot_base_frame: base_link
          global_frame: map
          use_sim_time: True
    -      plugins: ["static_layer", "obstacle_layer", "voxel_layer", "inflation_layer"]
    +      plugins: ["static_layer", "obstacle_layer", "voxel_layer", "gradient_layer"]
          robot_radius: 0.22
          resolution: 0.05
          obstacle_layer:
    ```


YAML-file may also contain the list of parameters (if any) for each plugin, identified by plugins object name.

> `YAML-文件也可能包含每个插件的参数列表（如果有），由插件对象名称标识。`


NOTE: there could be many simultaneously loaded plugin objects of one type. For this, `plugin_names` list should contain different plugins names whether the `plugin_types` will remain the same types. For example:

> 注意：一种类型的插件可能同时加载多个对象。为此，`plugin_names`列表应包含不同的插件名称，而`plugin_types`将保持相同的类型。例如：`等待`

    ```text
    plugins: ["obstacle_layer", "gradient_layer_1", "gradient_layer_2"]
    ```


In this case each plugin object will be handled by its own parameters tree in a YAML-file, like:

> 在这种情况下，每个插件对象将由其自己的参数树在YAML文件中处理，如：`Wait`

    ```text
    gradient_layer_1:
      plugin: nav2_gradient_costmap_plugin/GradientLayer
      enabled: True
      ...
    gradient_layer_2:
      plugin: nav2_gradient_costmap_plugin/GradientLayer
      enabled: False
      ...
    ```

## 4- Run GradientLayer plugin


Run Turtlebot3 simulation with enabled Nav2. Detailed instructuction how to make it are written at `getting_started`{.interpreted-text role="ref"}. Below is shortcut command for that:

> 运行Turtlebot3模拟，启用Nav2。详细的指令如何做到这一点可以在`getting_started`中找到。以下是该操作的快捷命令：`Wait`

    ```bash
    $ ros2 launch nav2_bringup tb3_simulation_launch.py
    ```


Then goto RViz and click on the \"2D Pose Estimate\" button at the top and point the location on map as it was described in `getting_started`{.interpreted-text role="ref"}. Robot will be localized on map and the result should be as presented at picture below. There is could be seen the gradient costmap. There are also 2 noticeable things: dynamically updated by `GradientLayer::updateCosts()` costmap within its bounds and global path curved by gradient:

> 然后去RViz，点击顶部的“2D Pose Estimate”按钮，并按照`getting_started`中描述的地图位置进行指向。机器人将在地图上定位，结果如下图所示。可以看到梯度成本图。还有两个显着的事情：`GradientLayer :: updateCosts（）`在其范围内动态更新的成本图，以及由梯度曲线曲线的全局路径：`

![Image of gradient costmap used](images/Writing_new_Costmap2D_plugin/gradient_layer_run.png){.align-center width="700px"}
