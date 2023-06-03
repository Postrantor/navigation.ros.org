---
tip: translate by openai@2023-06-03 00:52:13
...
---
title: Writing a New Controller Plugin
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1- Create a new Controller Plugin](#1--create-a-new-controller-plugin)
  - [2- Exporting the controller plugin](#2--exporting-the-controller-plugin)

  - [3- Pass the plugin name through the params file](#3--pass-the-plugin-name-through-the-params-file)

> - [3- 通过参数文件传递插件名称](#3--通过参数文件传递插件名称)
  - [4- Run Pure Pursuit Controller plugin](#4--run-pure-pursuit-controller-plugin)

![Animated gif of pure pursuit controller demo](images/Writing_new_nav2controller_plugin/nav2_pure_pursuit_gif.gif){.align-center width="640px"}

# Overview


This tutorial shows how to create your own controller [plugin](https://index.ros.org/p/pluginlib/).

> 这个教程展示了如何创建自己的控制器[插件](https://index.ros.org/p/pluginlib/)。


In this tutorial, we will be implementing the pure pursuit path tracking algorithm based on this [paper](https://www.ri.cmu.edu/pub_files/pub3/coulter_r_craig_1992_1/coulter_r_craig_1992_1.pdf). It is recommended you go through it.

> 在本教程中，我们将根据此[论文](https://www.ri.cmu.edu/pub_files/pub3/coulter_r_craig_1992_1/coulter_r_craig_1992_1.pdf)实施纯追踪路径跟踪算法。建议您阅读它。

Note: This tutorial is based on a previously existing simplifed version of the Regulated Pure Pursuit controller now in the Nav2 stack. You can find the source code matching this tutorial [here](https://github.com/ros-planning/navigation2_tutorials/tree/126902457c5c646b136569886d6325f070c1073d/nav2_pure_pursuit_controller).

# Requirements

- ROS 2 (binary or build-from-source)
- Nav2 (Including dependencies)
- Gazebo
- Turtlebot3

# Tutorial Steps

## 1- Create a new Controller Plugin


We will be implementing the pure pursuit controller. The annotated code in this tutorial can be found in [navigation_tutorials](https://github.com/ros-planning/navigation2_tutorials) repository as the `nav2_pure_pursuit_controller`. This package can be considered as a reference for writing your own controller plugin.

> 我们将实施纯追踪控制器。此教程中的注释代码可以在[navigation_tutorials](https://github.com/ros-planning/navigation2_tutorials)存储库中找到`nav2_pure_pursuit_controller`。这个软件包可以被视为编写自己的控制器插件的参考。


Our example plugin class `nav2_pure_pursuit_controller::PurePursuitController` inherits from the base class `nav2_core::Controller`. The base class provides a set of virtual methods to implement a controller plugin. These methods are called at runtime by the controller server to compute velocity commands. The list of methods, and their descriptions, and necessity are presented in the table below:

> 我们的示例插件类`nav2_pure_pursuit_controller::PurePursuitController`继承自基类`nav2_core::Controller`。基类提供了一组虚拟方法来实现控制器插件。这些方法在运行时由控制器服务器调用，以计算速度命令。下表列出了方法及其描述和必要性：

---


**Virtual method** **Method description** **Requires override?**

> **虚拟方法** **方法描述** **需要重写？**


configure() Method is called when controller server enters on_configure state. Ideally this method should perform declarations of ROS parameters and initialization of controller\'s member variables. This method takes 4 input params: weak pointer to parent node, controller name, tf buffer pointer and shared pointer to costmap. Yes

> `configure() 方法在控制器服务器进入on_configure状态时被调用。理想情况下，此方法应执行ROS参数的声明和控制器成员变量的初始化。此方法接受4个输入参数：父节点的弱指针、控制器名称、tf缓冲区指针和costmap的共享指针。是的。


activate() Method is called when controller server enters on_activate state. Ideally this method should implement operations which are neccessary before controller goes to an active state. Yes

> `activate() 方法在控制服务器进入on_activate状态时被调用。理想情况下，在控制器进入活动状态之前，此方法应实现必要的操作。是的`


deactivate() Method is called when controller server enters on_deactivate state. Ideally this method should implement operations which are neccessary before controller goes to an inactive state. Yes

> `方法deactivate()在控制服务器进入on_deactivate状态时被调用。理想情况下，在控制器进入非活动状态之前，这个方法应该实现必要的操作。是的`


cleanup() Method is called when controller server goes to on_cleanup state. Ideally this method should clean up resources which are created for the controller. Yes

> `cleanup() 方法在控制器服务器进入on_cleanup状态时调用。理想情况下，此方法应清理为控制器创建的资源。是的`


setPlan() Method is called when the global plan is updated. Ideally this method should perform operations that transform the global plan and stores it. Yes

> `setPlan() 方法在全局计划更新时调用。理想情况下，此方法应执行转换全局计划并存储它的操作。是的`


computeVelocityCommands() Method is called when a new velocity command is demanded by the controller server in-order for the robot to follow the global path. This method returns a [geometry_msgs::msg::TwistStamped]{.title-ref} which represents the velocity command for the robot to drive. This method passes 2 parameters: reference to the current robot pose and its current velocity. Yes

> 当控制服务器要求新的速度指令时，就会调用computeVelocityCommands()方法，以便机器人遵循全局路径。该方法返回[geometry_msgs::msg::TwistStamped]{.title-ref}，表示机器人驱动的速度指令。此方法传递2个参数：当前机器人位置的参考和其当前速度。是的。


setSpeedLimit() Method is called when it is required to limit the maximum linear speed of the robot. Speed limit could be expressed in absolute value (m/s) or in percentage from maximum robot speed. Note that typically, maximum rotational speed is being limited proportionally to the change of maximum linear speed, in order to keep current robot behavior to be untouched. Yes

> `setSpeedLimit()` 方法在需要限制机器人的最大线速度时调用。速度限制可以以绝对值（m / s）或最大机器人速度的百分比表示。注意，通常，为了保持当前机器人行为不变，最大转速会相应地随最大线速度的变化而变化。是的。

---


In this tutorial, we will have used the methods `PurePursuitController::configure`, `PurePursuitController::setPlan` and `PurePursuitController::computeVelocityCommands`.

> 在本教程中，我们将使用`PurePursuitController::configure`，`PurePursuitController::setPlan`和`PurePursuitController::computeVelocityCommands`方法。


In controllers, `configure()` method must set member variables from ROS parameters and perform any initialization required,

> 在控制器中，`configure()`方法必须从ROS参数设置成员变量，并执行任何所需的初始化，`Wait`

    ```c++
    void PurePursuitController::configure(
      const rclcpp_lifecycle::LifecycleNode::WeakPtr & parent,
      std::string name, std::shared_ptr<tf2_ros::Buffer> tf,
      std::shared_ptr<nav2_costmap_2d::Costmap2DROS> costmap_ros)
    {
      node_ = parent;
      auto node = node_.lock();

      costmap_ros_ = costmap_ros;
      tf_ = tf;
      plugin_name_ = name;
      logger_ = node->get_logger();
      clock_ = node->get_clock();

      declare_parameter_if_not_declared(
        node, plugin_name_ + ".desired_linear_vel", rclcpp::ParameterValue(
          0.2));
      declare_parameter_if_not_declared(
        node, plugin_name_ + ".lookahead_dist",
        rclcpp::ParameterValue(0.4));
      declare_parameter_if_not_declared(
        node, plugin_name_ + ".max_angular_vel", rclcpp::ParameterValue(
          1.0));
      declare_parameter_if_not_declared(
        node, plugin_name_ + ".transform_tolerance", rclcpp::ParameterValue(
          0.1));

      node->get_parameter(plugin_name_ + ".desired_linear_vel", desired_linear_vel_);
      node->get_parameter(plugin_name_ + ".lookahead_dist", lookahead_dist_);
      node->get_parameter(plugin_name_ + ".max_angular_vel", max_angular_vel_);
      double transform_tolerance;
      node->get_parameter(plugin_name_ + ".transform_tolerance", transform_tolerance);
      transform_tolerance_ = rclcpp::Duration::from_seconds(transform_tolerance);
    }
    ```


Here, `plugin_name_ + ".desired_linear_vel"` is fetching the ROS parameters `desired_linear_vel` which is specific to our controller. Nav2 allows loading of multiple plugins, and to keep things organized, each plugin is mapped to some ID/name. Now, if we want to retrieve the parameters for that specific plugin, we use `<mapped_name_of_plugin>.<name_of_parameter>` as done in the above snippet. For example, our example controller is mapped to the name `FollowPath` and to retrieve the `desired_linear_vel parameter`, which is specific to \"FollowPath", we used `FollowPath.desired_linear_vel`. In other words, `FollowPath` is used as a namespace for plugin-specific parameters. We will see more on this when we discuss the parameters file (or params file).

> 这里，`plugin_name_ +“.desired_linear_vel”`正在获取与我们控制器特定的ROS参数`desired_linear_vel`。Nav2允许加载多个插件，为了保持组织，每个插件都映射到某个ID/名称。现在，如果我们要检索特定插件的参数，我们使用`<mapped_name_of_plugin>.<name_of_parameter>`，如上面的片段所示。例如，我们的示例控制器映射到名称`FollowPath`，要检索特定于“FollowPath”的`desired_linear_vel参数`，我们使用`FollowPath.desired_linear_vel`。换句话说，`FollowPath`用作插件特定参数的命名空间。我们将在讨论参数文件（或参数文件）时看到更多内容。


The passed in arguments are stored in member variables so that they can be used at a later stage if needed.

> 传入的参数被存储在成员变量中，以便在需要时使用。


In `setPlan()` method, we receive the updated global path for the robot to follow. In our example, we transform the received global path into the frame of the robot and then store this transformed global path for later use.

> 在`setPlan()`方法中，我们接收机器人要跟随的更新的全局路径。在我们的示例中，我们将接收的全局路径转换到机器人的框架，然后将这个转换后的全局路径存储起来备用。

    ```c++
    void PurePursuitController::setPlan(const nav_msgs::msg::Path & path)
    {
      // Transform global path into the robot's frame
      global_plan_ = transformGlobalPlan(path);
    }
    ```


The computation for the desired velocity happens in the `computeVelocityCommands()` method. It is used to calculate the desired velocity command given the current velocity and pose. The third argument - is a pointer to the `nav2_core::GoalChecker`, that checks whether a goal has been reached. In our example, this is to be unused. In the case of pure pursuit, the algorithm computes velocity commands such that the robot tries to follow the global path as closely as possible. This algorithm assumes a constant linear velocity and computes the angular velocity based on the curvature of the global path.

> 计算所需速度的计算发生在`computeVelocityCommands()`方法中。它用于计算给定当前速度和姿态的期望速度命令。第三个参数 - 是指向`nav2_core：：GoalChecker`的指针，用于检查是否已达到目标。在我们的示例中，这将不会使用。在纯追踪的情况下，该算法计算速度命令，使机器人尽可能接近全局路径。该算法假设恒定的线性速度，并根据全局路径的曲率计算角速度。

    ```c++
    geometry_msgs::msg::TwistStamped PurePursuitController::computeVelocityCommands(
      const geometry_msgs::msg::PoseStamped & pose,
      const geometry_msgs::msg::Twist & velocity,
      nav2_core::GoalChecker * /*goal_checker*/)
    {
      // Find the first pose which is at a distance greater than the specified lookahed distance
      auto goal_pose = std::find_if(
        global_plan_.poses.begin(), global_plan_.poses.end(),
        [&](const auto & global_plan_pose) {
          return hypot(
            global_plan_pose.pose.position.x,
            global_plan_pose.pose.position.y) >= lookahead_dist_;
        })->pose;

      double linear_vel, angular_vel;

      // If the goal pose is in front of the robot then compute the velocity using the pure pursuit algorithm
      // else rotate with the max angular velocity until the goal pose is in front of the robot
      if (goal_pose.position.x > 0) {

        auto curvature = 2.0 * goal_pose.position.y /
          (goal_pose.position.x * goal_pose.position.x + goal_pose.position.y * goal_pose.position.y);
        linear_vel = desired_linear_vel_;
        angular_vel = desired_linear_vel_ * curvature;
      } else {
        linear_vel = 0.0;
        angular_vel = max_angular_vel_;
      }

      // Create and publish a TwistStamped message with the desired velocity
      geometry_msgs::msg::TwistStamped cmd_vel;
      cmd_vel.header.frame_id = pose.header.frame_id;
      cmd_vel.header.stamp = clock_->now();
      cmd_vel.twist.linear.x = linear_vel;
      cmd_vel.twist.angular.z = max(
        -1.0 * abs(max_angular_vel_), min(
          angular_vel, abs(
            max_angular_vel_)));

      return cmd_vel;
    }
    ```


The remaining methods are not used, but it\'s mandatory to override them. As per the rules, we did override all but left them empty.

> 剩余的方法没有被使用，但是必须覆盖它们。根据规则，我们覆盖了所有，但是留空了。

## 2- Exporting the controller plugin


Now that we have created our custom controller, we need to export our controller plugin so that it would be visible to the controller server. Plugins are loaded at runtime, and if they are not visible, then our controller server won\'t be able to load it. In ROS 2, exporting and loading plugins is handled by `pluginlib`.

> 现在我们已经创建了自定义控制器，我们需要导出我们的控制器插件，以便它对控制器服务器可见。插件在运行时加载，如果它们不可见，那么我们的控制器服务器将无法加载它们。在ROS 2中，导出和加载插件由`pluginlib`处理。


Coming to our tutorial, class `nav2_pure_pursuit_controller::PurePursuitController` is loaded dynamically as `nav2_core::Controller` which is our base class.

> 来到我们的教程中，类`nav2_pure_pursuit_controller::PurePursuitController`动态加载为`nav2_core::Controller`，这是我们的基类。


1.  To export the controller, we need to provide two lines

> 1. 要导出控制器，我们需要提供两行`Wait`。

    ```c++
    #include "pluginlib/class_list_macros.hpp"
    PLUGINLIB_EXPORT_CLASS(nav2_pure_pursuit_controller::PurePursuitController, nav2_core::Controller)
    ```

Note that it requires pluginlib to export out the plugin\'s class. Pluginlib would provide as macro `PLUGINLIB_EXPORT_CLASS`, which does all the work of exporting.


It is good practice to place these lines at the end of the file, but technically, you can also write at the top.

> 这些行最好放在文件末尾，但从技术上讲，也可以写在顶部。


2.  The next step would be to create the plugin\'s description file in the root directory of the package. For example, `pure_pursuit_controller_plugin.xml` file in our tutorial package. This file contains the following information

> 接下來的步驟是在套件的根目錄中創建插件的描述文件。例如，我們教程包中的`pure_pursuit_controller_plugin.xml`文件。該文件包含以下信息`Wait`

- `library path`: Plugin\'s library name and it\'s location.
- `class name`: Name of the class.
- `class type`: Type of class.
- `base class`: Name of the base class.
- `description`: Description of the plugin.

    ```xml
    <library path="nav2_pure_pursuit_controller">
      <class type="nav2_pure_pursuit_controller::PurePursuitController" base_class_type="nav2_core::Controller">
        <description>
          This is pure pursuit controller
        </description>
      </class>
    </library>
    ```


3.  Next step would be to export plugin using `CMakeLists.txt` by using CMake function `pluginlib_export_plugin_description_file()`. This function installs the plugin description file to `share` directory and sets ament indexes to make it discoverable.

> 下一步将使用CMake函数`pluginlib_export_plugin_description_file()`通过`CMakeLists.txt`导出插件。该函数将插件描述文件安装到`share`目录，并设置ament索引以使其可检索。

    ```text
    pluginlib_export_plugin_description_file(nav2_core pure_pursuit_controller_plugin.xml)
    ```


4.  The plugin description file should also be added to `package.xml`

> 4. 同时应将插件描述文件添加到`package.xml`中。

    ```xml
    <export>
      <build_type>ament_cmake</build_type>
      <nav2_core plugin="${prefix}/pure_pursuit_controller_plugin.xml" />
    </export>
    ```


5.  Compile, and it should be registered. Next, we\'ll use this plugin.

> `编译，它应该被注册。接下来，我们将使用这个插件。`

## 3- Pass the plugin name through the params file


To enable the plugin, we need to modify the `nav2_params.yaml` file as below

> 启用插件，我们需要修改`nav2_params.yaml`文件如下：`

    ```text
    controller_server:
      ros__parameters:
        controller_plugins: ["FollowPath"]

        FollowPath:
          plugin: "nav2_pure_pursuit_controller::PurePursuitController"
          debug_trajectory_details: True
          desired_linear_vel: 0.2
          lookahead_dist: 0.4
          max_angular_vel: 1.0
          transform_tolerance: 1.0
    ```


In the above snippet, you can observe the mapping of our `nav2_pure_pursuit_controller/PurePursuitController` controller to its id `FollowPath`. To pass plugin-specific parameters we have used `<plugin_id>.<plugin_specific_parameter>`.

> 在上面的片段中，您可以观察到我们的`nav2_pure_pursuit_controller/PurePursuitController`控制器到其id `FollowPath`的映射。为了传递特定于插件的参数，我们使用了`<plugin_id>.<plugin_specific_parameter>`。

## 4- Run Pure Pursuit Controller plugin


Run Turtlebot3 simulation with enabled Nav2. Detailed instructions on how to make it run are written at `getting_started`{.interpreted-text role="ref"}. Below is a shortcut command for that:

> 运行Turtlebot3模拟，启用Nav2。有关如何使其运行的详细说明可在 `getting_started` 中找到。以下是快捷命令：`Wait`

    ```bash
    $ ros2 launch nav2_bringup tb3_simulation_launch.py params_file:=/path/to/your_params_file.yaml
    ```


Then goto RViz and click on the \"2D Pose Estimate\" button at the top and point the location on the map as it was described in `getting_started`{.interpreted-text role="ref"}. The robot will localize on the map and then click on the \"Nav2 goal\" and click on the pose where you want your robot to navigate to. After that controller will make the robot follow the global path.

> 然后，去RViz，在顶部点击“2D Pose Estimate”按钮，并根据`getting_started`中描述的位置在地图上指向。机器人将定位在地图上，然后点击“Nav2 goal”，点击您想要机器人导航到的位置。之后，控制器将使机器人遵循全局路径。
