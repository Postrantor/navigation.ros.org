---
tip: translate by openai@2023-06-03 00:56:43
...
---
title: Writing a New Planner Plugin
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1- Creating a new Planner Plugin](#1--creating-a-new-planner-plugin)
  - [2- Exporting the planner plugin](#2--exporting-the-planner-plugin)
  - [3- Pass the plugin name through params file](#3--pass-the-plugin-name-through-params-file)
  - [4- Run StraightLine plugin](#4--run-straightline-plugin)

![Animated gif with gradient demo](images/Writing_new_nav2planner_plugin/nav2_straightline_gif.gif){.align-center width="700px"}

# Overview


This tutorial shows how to create you own planner plugin.

> 这个教程展示了如何创建你自己的计划者插件。

# Requirements

- ROS 2 (binary or build-from-source)
- Nav2 (Including dependencies)
- Gazebo
- Turtlebot3

# Tutorial Steps

## 1- Creating a new Planner Plugin


We will create a simple straight line planner. The annotated code in this tutorial can be found in [navigation_tutorials](https://github.com/ros-planning/navigation2_tutorials) repository as the `nav2_straightline_planner` This package can be a considered as a reference for writing planner plugin.

> 我们将创建一个简单的直线规划器。本教程中的注释代码可以在[navigation_tutorials](https://github.com/ros-planning/navigation2_tutorials)存储库中找到`nav2_straightline_planner`这个包可以作为编写规划器插件的参考。


Our example plugin inherits from the base class `nav2_core::GlobalPlanner`. The base class provides 5 pure virtual methods to implement a planner plugin. The plugin will be used by the planner server to compute trajectories. Lets learn more about the methods needed to write a planner plugin.

> 我们的示例插件继承自基类`nav2_core::GlobalPlanner`。该基类提供了5个纯虚拟方法来实现规划器插件。该插件将由规划器服务器用于计算轨迹。让我们来了解更多有关编写规划器插件所需的方法。

---


**Virtual method** **Method description** **Requires override?**

> **虚拟方法** **方法描述** **需要覆盖？**


configure() Method is called at when planner server enters on_configure state. Ideally this methods should perform declarations of ROS parameters and initialization of planner\'s member variables. This method takes 4 input params: shared pointer to parent node, planner name, tf buffer pointer and shared pointer to costmap. Yes

> `configure() 方法在规划服务器进入on_configure状态时被调用。理想情况下，该方法应该执行ROS参数的声明和规划师成员变量的初始化。该方法需要4个输入参数：指向父节点的共享指针、规划师名称、tf缓冲区指针和指向costmap的共享指针。是的。


activate() Method is called when planner server enters on_activate state. Ideally this method should implement operations which are neccessary before planner goes to an active state. Yes

> `activate()` 方法在计划程序服务器进入on_activate状态时被调用。理想情况下，在计划程序进入活动状态之前，应该实现必要的操作。是的。


deactivate() Method is called when planner server enters on_deactivate state. Ideally this method should implement operations which are neccessary before planner goes to an inactive state. Yes

> `deactivate() 方法在计划者服务器进入on_deactivate状态时被调用。理想情况下，该方法应该实现在计划者进入非活动状态之前所必需的操作。是的`


cleanup() Method is called when planner server goes to on_cleanup state. Ideally this method should clean up resoures which are created for the planner. Yes

> `cleanup() 方法在计划程序服务器进入on_cleanup状态时被调用。理想情况下，此方法应清理为计划程序创建的资源。是的`


createPlan() Method is called when planner server demands a global plan for specified start and goal pose. This method returns [nav_msgs::msg::Path]{.title-ref} carrying global plan. This method takes 2 input parmas: start pose and goal pose. Yes

> `createPlan()` 方法在规划服务器要求指定起始和目标位置的全局计划时被调用。此方法返回[nav_msgs::msg::Path]{.title-ref}，携带全局计划。此方法需要2个输入参数：起始位置和目标位置。是的。

---


For this tutorial, we will be using methods `StraightLine::configure()` and `StraightLine::createPlan()` to create straight line planner.

> 对于本教程，我们将使用方法`StraightLine::configure()`和`StraightLine::createPlan()`来创建直线规划器。


In planners, `configure()` method must set member variables from ROS parameters and any initialization required,

> 在规划器中，`configure()`方法必须从ROS参数设置成员变量，并进行任何必要的初始化`

    ```c++
    node_ = parent;
    tf_ = tf;
    name_ = name;
    costmap_ = costmap_ros->getCostmap();
    global_frame_ = costmap_ros->getGlobalFrameID();

    // Parameter initialization
    nav2_util::declare_parameter_if_not_declared(node_, name_ + ".interpolation_resolution", rclcpp::ParameterValue(0.1));
    node_->get_parameter(name_ + ".interpolation_resolution", interpolation_resolution_);
    ```


Here, `name_ + ".interpolation_resolution"` is fetching the ROS parameters `interpolation_resolution` which is specific to our planner. Navigation2 allows loading of multiple plugins and to keep things organized each plugin is mapped to some ID/name. Now if we want to retrieve the parameters for that specific plugin, we use `<mapped_name_of_plugin>.<name_of_parameter>` as done in the above snippet. For example, our example planner is mapped to the name \"GridBased\" and to retrieve the `interpolation_resolution` parameter which is specific to \"GridBased\", we used `Gridbased.interpolation_resolution`. In other words, `GridBased` is used as a namespace for plugin-specific parameters. We will see more on this when we discuss the parameters file (or params file).

> 这里，`name_ +“.interpolation_resolution”` 正在提取ROS参数`interpolation_resolution`，这是我们规划器特有的。Navigation2允许加载多个插件，为了保持整洁，每个插件都映射到某个ID/名称。现在，如果我们想要检索特定插件的参数，我们使用`<mapped_name_of_plugin>.<name_of_parameter>`，就像上面的片段中所做的那样。例如，我们的示例规划器映射到名称“GridBased”，为了检索特定于“GridBased”的`interpolation_resolution`参数，我们使用了`GridBased.interpolation_resolution`。换句话说，`GridBased`用作插件特定参数的命名空间。当我们讨论参数文件（或参数文件）时，我们会看到更多内容。


In `createPlan()` method, we need to create a path from the given start to goal poses. The `StraightLine::createPlan()` is called using start pose and goal pose to solve the global path planning problem. Upon succeeding, it converts the path to the `nav_msgs::msg::Path` and returns to the planner server. Below annotation shows the implementation of this method.

> 在`createPlan()`方法中，我们需要根据给定的起始位置和目标位置创建一条路径。使用起始位置和目标位置调用`StraightLine::createPlan()`来解决全局路径规划问题。成功后，它将路径转换为`nav_msgs::msg::Path`并返回给规划器服务器。下面的注释显示了该方法的实现。

    ```c++
    nav_msgs::msg::Path global_path;

    // Checking if the goal and start state is in the global frame
    if (start.header.frame_id != global_frame_) {
      RCLCPP_ERROR(
        node_->get_logger(), "Planner will only except start position from %s frame",
        global_frame_.c_str());
      return global_path;
    }

    if (goal.header.frame_id != global_frame_) {
      RCLCPP_INFO(
        node_->get_logger(), "Planner will only except goal position from %s frame",
        global_frame_.c_str());
      return global_path;
    }

    global_path.poses.clear();
    global_path.header.stamp = node_->now();
    global_path.header.frame_id = global_frame_;
    // calculating the number of loops for current value of interpolation_resolution_
    int total_number_of_loop = std::hypot(
      goal.pose.position.x - start.pose.position.x,
      goal.pose.position.y - start.pose.position.y) /
      interpolation_resolution_;
    double x_increment = (goal.pose.position.x - start.pose.position.x) / total_number_of_loop;
    double y_increment = (goal.pose.position.y - start.pose.position.y) / total_number_of_loop;

    for (int i = 0; i < total_number_of_loop; ++i) {
      geometry_msgs::msg::PoseStamped pose;
      pose.pose.position.x = start.pose.position.x + x_increment * i;
      pose.pose.position.y = start.pose.position.y + y_increment * i;
      pose.pose.position.z = 0.0;
      pose.pose.orientation.x = 0.0;
      pose.pose.orientation.y = 0.0;
      pose.pose.orientation.z = 0.0;
      pose.pose.orientation.w = 1.0;
      pose.header.stamp = node_->now();
      pose.header.frame_id = global_frame_;
      global_path.poses.push_back(pose);
    }

    global_path.poses.push_back(goal);

    return global_path;
    ```


The remaining methods are not used but its mandatory to override them. As per the rules, we did override all but left them blank.

> 剩下的方法没有被使用，但是必须要重写它们。根据规定，我们重写了所有方法，但是留下了空白。

## 2- Exporting the planner plugin


Now that we have created our custom planner, we need to export our planner plugin so that it would be visible to the planner server. Plugins are loaded at runtime and if they are not visible, then our planner server won\'t be able to load it. In ROS 2, exporting and loading plugins is handled by `pluginlib`.

> 现在我们已经创建了自定义计划器，我们需要导出计划器插件，以便它对计划器服务器可见。插件在运行时加载，如果它们不可见，那么我们的计划器服务器就无法加载它。在ROS 2中，导出和加载插件由`pluginlib`处理。


Coming to our tutorial, class `nav2_straightline_planner::StraightLine` is loaded dynamically as `nav2_core::GlobalPlanner` which is our base class.

> 来到我们的教程中，`nav2_straightline_planner::StraightLine` 类会动态加载为 `nav2_core::GlobalPlanner`，它是我们的基类。


1.  To export the planner, we need to provide two lines

> 1. 要导出计划者，我们需要提供两行`Wait`

    ```c++
    #include "pluginlib/class_list_macros.hpp"
    PLUGINLIB_EXPORT_CLASS(nav2_straightline_planner::StraightLine, nav2_core::GlobalPlanner)
    ```

Note that it requires pluginlib to export out plugin\'s class. Pluginlib would provide as macro `PLUGINLIB_EXPORT_CLASS` which does all the work of exporting.


It is good practice to place these lines at the end of the file but technically, you can also write at the top.

> 这些行的放置最好放在文件末尾，但从技术上讲，你也可以把它们写在顶部。


2.  Next step would be to create plugin\'s description file in the root directory of the package. For example, `global_planner_plugin.xml` file in our tutorial package. This file contains following information

> 接下来的步骤是在包的根目录中创建插件的描述文件。例如，我们教程包中的`global_planner_plugin.xml`文件。此文件包含以下信息`

> - `library path`: Plugin\'s library name and it\'s location.
> - `class name`: Name of the class.
> - `class type`: Type of class.
> - `base class`: Name of the base class.
> - `description`: Description of the plugin.

    ```xml
    <library path="nav2_straightline_planner_plugin">
      <class name="nav2_straightline_planner/StraightLine" type="nav2_straightline_planner::StraightLine" base_class_type="nav2_core::GlobalPlanner">
        <description>This is an example plugin which produces straight path.</description>
      </class>
    </library>
    ```


3.  Next step would be to export plugin using `CMakeLists.txt` by using cmake function `pluginlib_export_plugin_description_file()`. This function installs plugin description file to `share` directory and sets ament indexes to make it discoverable.

> 下一步是使用cmake函数 `pluginlib_export_plugin_description_file()` 通过 `CMakeLists.txt` 来导出插件。 该函数将插件描述文件安装到`share`目录，并设置ament索引，使其可以发现。

    ```text
    pluginlib_export_plugin_description_file(nav2_core global_planner_plugin.xml)
    ```


4.  Plugin description file should also be added to `package.xml`

> 4. 同时也应该将插件描述文件添加到`package.xml`中。

    ```xml
    <export>
      <build_type>ament_cmake</build_type>
      <nav2_core plugin="${prefix}/global_planner_plugin.xml" />
    </export>
    ```


5.  Compile and it should be registered. Next, we\'ll use this plugin.

> 编译它，它应该被注册。接下来，我们将使用此插件。

## 3- Pass the plugin name through params file


To enable the plugin, we need to modify the `nav2_params.yaml` file as belowto replace following params

> 要启用插件，我们需要修改`nav2_params.yaml`文件，如下所示来替换以下参数`

::: note
::: title
Note
:::


For Galactic or later, `plugin_names` and `plugin_types` have been replaced with a single `plugins` string vector for plugin names. The types are now defined in the `plugin_name` namespace in the `plugin:` field (e.g. `plugin: MyPlugin::Plugin`). Inline comments in the code blocks will help guide you through this.

> 对于Galactic或更新版本，`plugin_names`和`plugin_types`已被单个`plugins`字符串向量所取代，用于插件名称。类型现在定义在`plugin_name`命名空间中的`plugin:`字段中（例如`plugin: MyPlugin::Plugin`）。代码块中的内联注释将帮助您指引这一过程。
:::

    ```text
    planner_server:
      ros__parameters:
        plugins: ["GridBased"]
        use_sim_time: True
        GridBased:
          plugin: "nav2_navfn_planner/NavfnPlanner" # For Foxy and later
          tolerance: 2.0
          use_astar: false
          allow_unknown: true
    ```


with

> 等待

    ```text
    planner_server:
      ros__parameters:
        plugins: ["GridBased"]
        use_sim_time: True
        GridBased:
          plugin: "nav2_straightline_planner/StraightLine"
          interpolation_resolution: 0.1
    ```


In the above snippet, you can observe the mapping of our `nav2_straightline_planner/StraightLine` planner to its id `GridBased`. To pass plugin-specific parameters we have used `<plugin_id>.<plugin_specific_parameter>`.

> 在上面的片段中，您可以观察到我们的`nav2_straightline_planner/StraightLine`规划器到其id `GridBased`的映射。为了传递插件特定的参数，我们使用了`<plugin_id>.<plugin_specific_parameter>`。

## 4- Run StraightLine plugin


Run Turtlebot3 simulation with enabled navigation2. Detailed instruction how to make it are written at `getting_started`{.interpreted-text role="ref"}. Below is shortcut command for that:

> 运行Turtlebot3模拟，启用navigation2。详细的指令如何做到这一点可以在`getting_started`中找到。下面是快捷命令：`Wait`

    ```bash
    $ ros2 launch nav2_bringup tb3_simulation_launch.py params_file:=/path/to/your_params_file.yaml
    ```


Then goto RViz and click on the \"2D Pose Estimate\" button at the top and point the location on map as it was described in `getting_started`{.interpreted-text role="ref"}. Robot will localize on the map and then click on \"Navigation2 goal\" and click on the pose where you want your planner to consider a goal pose. After that planner will plan the path and robot will start moving towards the goal.

> 然后去RViz，在顶部点击“2D Pose Estimate”按钮，按照`getting_started`中所述的位置在地图上指示。机器人将在地图上定位，然后点击“Navigation2 goal”，点击您希望计划者考虑目标姿态的姿态。之后，计划者将规划路径，机器人将开始朝目标移动。
