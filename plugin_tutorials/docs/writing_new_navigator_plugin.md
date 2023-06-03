---
tip: translate by openai@2023-06-03 01:00:27
...
---
title: Writing a New Navigator Plugin
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1- Create a new Navigator Plugin](#1--create-a-new-navigator-plugin)
  - [2- Exporting the navigator plugin](#2--exporting-the-navigator-plugin)

  - [3- Pass the plugin name through the params file](#3--pass-the-plugin-name-through-the-params-file)

> - [3- 通过参数文件传递插件名称](#3--pass-the-plugin-name-through-the-params-file)
  - [4- Run plugin](#4--run-plugin)

# Overview


This tutorial shows how to create your own behavior-tree navigator [plugin](https://index.ros.org/p/pluginlib/) based on the `nav2_core::BehaviorTreeNavigator` base class.

> 本教程展示如何基于`nav2_core::BehaviorTreeNavigator`基类创建自己的行为树导航[插件](https://index.ros.org/p/pluginlib/)。


In this tutorial, we will be reviewing the `Navigate to Pose` behavior-tree navigator plugin, which is the foundational navigator of Nav2 and complimentary behavior to ROS 1 Navigation. This completes point-to-point navigation. This tutorial will be reviewing the code and structure as of ROS 2 Iron. While small variations may be made over time, this should be sufficient to get started writing your own navigator if you choose as we do not expect major API changes on this system.

> 在本教程中，我们将回顾`导航到姿态`行为树导航插件，它是Nav2的基础导航器和ROS 1导航的补充行为。这完成了点对点导航。本教程将回顾ROS 2 Iron的代码和结构。虽然可能会随时间做出小的变化，但这应该足以让您开始编写自己的导航器，因为我们不期望该系统出现重大的API更改。


It may be beneficial to write your own Navigator if you have a custom action message definition you\'d like to use with Navigation rather than the provided `NavigateToPose` or `NavigateThroughPoses` interfaces (e.g. doing complete coverage or containing additional constraint information). The role of the Navigators are to extract information from requests to pass to the behavior tree / blackboard, populate feedback and responses, and maintain the state of the behavior tree if relevant. The behavior tree XML will define the actual navigation logic used.

> 如果你有一个自定义的动作消息定义，你想要使用导航而不是提供的`NavigateToPose`或`NavigateThroughPoses`接口（例如完成覆盖或包含额外的约束信息），那么编写自己的导航器可能会有益。导航器的作用是从请求中提取信息以传递给行为树/黑板，填充反馈和响应，并维护行为树的状态（如果有关）。行为树XML将定义实际使用的导航逻辑。

# Requirements

- ROS 2 (binary or build-from-source)
- Nav2 (Including dependencies)
- Gazebo
- Turtlebot3

# Tutorial Steps

## 1- Create a new Navigator Plugin


We will be implementing pure point-to-point navigation behavior. The code in this tutorial can be found in [Nav2s BT Navigator package](https://github.com/ros-planning/navigation2/nav2_bt_navigator) as the `NavigateToPoseNavigator`. This package can be considered as a reference for writing your own plugin.

> 我们将实施纯点对点导航行为。本教程中的代码可以在[Nav2s BT Navigator package](https://github.com/ros-planning/navigation2/nav2_bt_navigator)中找到，作为`NavigateToPoseNavigator`。这个包可以被视为编写自己的插件的参考。


Our example plugin class `nav2_bt_navigator::NavigateToPoseNavigator` inherits from the base class `nav2_core::BehaviorTreeNavigator`. The base class provides a set of virtual methods to implement a navigator plugin. These methods are called at runtime by the BT Navigator server or as a response to ROS 2 actions to process a navigation request.

> 我们的示例插件类`nav2_bt_navigator::NavigateToPoseNavigator`继承自基类`nav2_core::BehaviorTreeNavigator`。基类提供了一组虚拟方法来实现导航插件。这些方法可以在运行时由BT Navigator服务器调用，也可以作为对ROS 2操作的响应来处理导航请求。

Note that this class has itself a base class of `NavigatorBase`. This class is to provide a non-templated base-class for use in loading the plugins into vectors for storage and calls for basic state transition in the lifecycle node. Its members (e.g. `on_XYZ`) are implemented for you in `BehaviorTreeNavigator` and marked as `final` so they are not possible to be overrided by the user. The API that you will be implementing for your navigator are the virtual methods within `BehaviorTreeNavigator`, not `NavigatorBase`. These `on_XYZ` APIs are implemented in necessary functions in `BehaviorTreeNavigator` to handle boilerplate logic regarding the behavior tree and action server to minimize code duplication across the navigator implementations (e.g. `on_configure` will create the action server, register callbacks, populate the blackboard with some necessary basic information, and then call a user-defined `configure` function for any additional user-specific needs).


The list of methods, and their descriptions, and necessity are presented in the table below:

> 以下表格提供了方法的列表、描述和必要性：

---


**Virtual method** **Method description** **Requires override?**

> `虚拟方法` `方法描述` `需要重写？`


getDefaultBTFilepath() Method is called on initialization to retrieve the default BT filepath to use for navigation. This may be done via parameters, hardcoded logic, sentinal files, etc. Yes

> `getDefaultBTFilepath() 方法在初始化时被调用，以获取用于导航的默认BT文件路径。 这可以通过参数，硬编码逻辑，哨兵文件等来完成。 是的`


configure() Method is called when BT navigator server enters on_configure state. This method should implement operations which are neccessary before navigator goes to an active state, such as getting parameters, setting up the blackboard, etc. No

> `configure() 方法在BT导航服务器进入on_configure状态时被调用。在导航器进入活动状态之前，该方法应实现必要的操作，例如获取参数、设置黑板等。`Wait`


activate() Method is called when BT navigator server enters on_activate state. This method should implement operations which are neccessary before navigator goes to an active state, such as create clients and subscriptions. No

> `activate() 方法在BT导航服务器进入on_activate状态时被调用。在导航器进入活动状态之前，此方法应实现必要的操作，例如创建客户端和订阅。`Wait`


deactivate() Method is called when BT navigator server enters on_deactivate state. This method should implement operations which are neccessary before navigator goes to an inactive state. No

> `deactivate()`方法在BT导航服务器进入on_deactivate状态时被调用。在导航器进入非活动状态之前，此方法应实施必要的操作。`Wait`


cleanup() Method is called when BT navigator server goes to on_cleanup state. This method should clean up resources which are created for the navigator. No

> `cleanup() 方法在BT导航服务器进入on_cleanup状态时被调用。这个方法应该清理为导航器创建的资源。`Wait`


goalReceived() Method is called when a new goal is received by the action server to process. It may accept or deny this goal with its return signature. If accepted, it may need to load the appropriate parameters from the request (e.g. which BT to use), add request parameters to the blackboard for your applications use, or reset internal state. Yes

> 当动作服务器接收新目标时，将调用goalReceived（）方法。它可以通过其返回签名接受或拒绝此目标。如果接受，则可能需要从请求中加载适当的参数（例如要使用哪个BT），将请求参数添加到黑板上以供应用程序使用，或重置内部状态。是的。


onLoop() Method is called periodically while the behavior tree is looping to check statuses or more commonly to publish action feedback statuses to the client. Yes

> `onLoop() 方法在行为树循环期间周期性地被调用，以检查状态或更常见地将动作反馈状态发布给客户端。是的。`


onPreempt() Method is called when a new goal is requesting preemption over the existing goal currently being processed. If the new goal is viable, it should make all appropriate updates to the BT and blackboard such that this new request may immediately start being processed without hard cancelation of the initial task (e.g. preemption). Yes

> 当新的目标请求抢占现在正在处理的目标时，`onPreempt()` 方法被调用。如果新的目标可行，它应该对 BT 和黑板做出所有适当的更新，以便这个新的请求可以立即开始处理，而不需要对初始任务进行硬取消（例如抢占）。是的。


goalCompleted() Method is called when a goal is completed to populate the action result object or do any additional checks required at the end of a task. Yes

> `goalCompleted() 方法在完成目标时被调用，以填充操作结果对象或在任务结束时执行任何额外的检查。是的.`


getName() Method is called to get the name of this navigator type Yes

> `getName()` 方法被调用来获取这种导航器类型的名称 Yes

---


In the Navigate to Pose Navigator, `configure()` method must determine the blackboard parameter names where the goal and paths are being stored, as these are key values for processing feedback in `onLoop` and for the different behavior tree nodes to communicate this information between themselves. Additionally and uniquely to this navigator type, we also create a client to itself and a subscription to the `goal_pose` topic such that requests from the default configurations of Rviz2 using the _Goal Pose_ tool will be processed.

> 在导航到姿态导航器中，`configure()`方法必须确定存储目标和路径的黑板参数名称，因为这些是处理`onLoop`反馈和不同行为树节点之间进行通信所需的关键值。此外，专门针对此类导航器，我们还创建了一个客户端和一个`goal_pose`主题的订阅，以便处理来自Rviz2的_Goal Pose_工具的默认配置的请求。

    ```c++
    bool NavigateToPoseNavigator::configure(
      rclcpp_lifecycle::LifecycleNode::WeakPtr parent_node,
      std::shared_ptr<nav2_util::OdomSmoother> odom_smoother)
    {
      start_time_ = rclcpp::Time(0);
      auto node = parent_node.lock();

      if (!node->has_parameter("goal_blackboard_id")) {
        node->declare_parameter("goal_blackboard_id", std::string("goal"));
      }

      goal_blackboard_id_ = node->get_parameter("goal_blackboard_id").as_string();

      if (!node->has_parameter("path_blackboard_id")) {
        node->declare_parameter("path_blackboard_id", std::string("path"));
      }

      path_blackboard_id_ = node->get_parameter("path_blackboard_id").as_string();

      // Odometry smoother object for getting current speed
      odom_smoother_ = odom_smoother;

      self_client_ = rclcpp_action::create_client<ActionT>(node, getName());

      goal_sub_ = node->create_subscription<geometry_msgs::msg::PoseStamped>(
        "goal_pose",
        rclcpp::SystemDefaultsQoS(),
        std::bind(&NavigateToPoseNavigator::onGoalPoseReceived, this, std::placeholders::_1));
      return true;
    }
    ```


The values of the blackboard IDs are stored alongside the odometry smoother the BT Navigator provides for populating meaningful feedback later. Complimentary to this, the `cleanup` method will reset these resources. The activate and deactivate methods are not used in this particular navigator.

> 黑板ID的值存储在为填充有意义反馈而提供的测距稳定器旁边。配套使用的`cleanup`方法将重置这些资源。此特定导航器不使用激活和停用方法。

    ```c++
    bool NavigateToPoseNavigator::cleanup()
    {
      goal_sub_.reset();
      self_client_.reset();
      return true;
    }
    ```


In the `getDefaultBTFilepath()`, we use a parameter `default_nav_to_pose_bt_xml` to get the default behavior tree XML file to use if none is provided by the navigation request and to initialize the BT Navigator with a behavior tree hot-loaded. If one is not provided in the parameter files, then we grab a known and reasonable default XML file in the `nav2_bt_navigator` package:

> 在`getDefaultBTFilepath()`中，我们使用参数`default_nav_to_pose_bt_xml`来获取如果导航请求没有提供的话，要使用的默认行为树XML文件，以及用行为树热加载初始化BT Navigator。如果参数文件中没有提供，那么我们在`nav2_bt_navigator`包中抓取一个已知且合理的默认XML文件：`

    ```c++
    std::string NavigateToPoseNavigator::getDefaultBTFilepath(
      rclcpp_lifecycle::LifecycleNode::WeakPtr parent_node)
    {
      std::string default_bt_xml_filename;
      auto node = parent_node.lock();

      if (!node->has_parameter("default_nav_to_pose_bt_xml")) {
        std::string pkg_share_dir =
          ament_index_cpp::get_package_share_directory("nav2_bt_navigator");
        node->declare_parameter<std::string>(
          "default_nav_to_pose_bt_xml",
          pkg_share_dir +
          "/behavior_trees/navigate_to_pose_w_replanning_and_recovery.xml");
      }

      node->get_parameter("default_nav_to_pose_bt_xml", default_bt_xml_filename);

      return default_bt_xml_filename;
    }
    ```


When a goal is received, we need to determine if this goal is valid and should be processed. The `goalReceived` method provides you the `goal` and a return value if it is being processed or not. This information is sent back to the action server to notify the client. In this case, we want to make sure that the goal\'s behavior tree is valid or else we cannot proceed. If it is valid, then we can initialize the goal pose onto the blackboard and reset some state in order to cleanly process this new request.

> 收到目标时，我们需要确定该目标是否有效并且应该被处理。`goalReceived`方法提供了`goal`和一个返回值，以确定是否正在处理它。此信息被发回到动作服务器以通知客户端。在这种情况下，我们要确保目标的行为树是有效的，否则我们无法继续。如果有效，那么我们可以将目标姿势初始化到黑板上，并重置一些状态以清洁地处理这个新请求。

    ```c++
    bool NavigateToPoseNavigator::goalReceived(ActionT::Goal::ConstSharedPtr goal)
    {
      auto bt_xml_filename = goal->behavior_tree;

      if (!bt_action_server_->loadBehaviorTree(bt_xml_filename)) {
        RCLCPP_ERROR(
          logger_, "BT file not found: %s. Navigation canceled.",
          bt_xml_filename.c_str());
        return false;
      }

      initializeGoalPose(goal);

      return true;
    }
    ```


Once this goal is completed, we need to populate the Action\'s result, if required and meaningful. In this navigator\'s case, it contains no result information when the navigation request was completed successfully, so this method is empty. For other navigator types, you may populate the `result` object provided.

> 一旦完成此目标，我们需要填充动作的结果，如果有必要和有意义的话。在这个导航器的情况下，当导航请求成功完成时，它不包含结果信息，因此此方法为空。对于其他导航器类型，您可以填充提供的`result`对象。

    ```c++
    void NavigateToPoseNavigator::goalCompleted(
      typename ActionT::Result::SharedPtr /*result*/,
      const nav2_behavior_tree::BtStatus /*final_bt_status*/)
    {
    }
    ```


If however a goal is preempted (e.g. a new action request comes in while an existing request is being processed), the `onPreempt()` method is called to determine if the new request is genuine and appropriate to preempt the currently processing goal. For example, it might not be wise to accept a preeemption request if that request is fundamentally different in nature from an existing behavior tree task or when your existing task is of a higher priority.

> 如果一个目标被抢占（例如，在处理现有请求时又收到新的动作请求），则会调用`onPreempt()`方法来确定新请求是否真实合适，以抢占当前正在处理的目标。例如，如果请求和现有行为树任务本质上不同，或者当前任务具有更高优先级时，可能不建议接受抢占请求。

    ```c++
    void NavigateToPoseNavigator::onPreempt(ActionT::Goal::ConstSharedPtr goal)
    {
      RCLCPP_INFO(logger_, "Received goal preemption request");

      if (goal->behavior_tree == bt_action_server_->getCurrentBTFilename() ||
        (goal->behavior_tree.empty() &&
        bt_action_server_->getCurrentBTFilename() == bt_action_server_->getDefaultBTFilename()))
      {
        // if pending goal requests the same BT as the current goal, accept the pending goal
        // if pending goal has an empty behavior_tree field, it requests the default BT file
        // accept the pending goal if the current goal is running the default BT file
        initializeGoalPose(bt_action_server_->acceptPendingGoal());
      } else {
        RCLCPP_WARN(
          logger_,
          "Preemption request was rejected since the requested BT XML file is not the same "
          "as the one that the current goal is executing. Preemption with a new BT is invalid "
          "since it would require cancellation of the previous goal instead of true preemption."
          "\nCancel the current goal and send a new action request if you want to use a "
          "different BT XML file. For now, continuing to track the last goal until completion.");
        bt_action_server_->terminatePendingGoal();
      }
    }
    ```

Note that here you can also see the `initializeGoalPose` method called. This method will set the goal parameters for this navigator onto the blackboard and reset important state information to cleanly re-use a behavior tree without old state information, as shown below:

    ```c++
    void
    NavigateToPoseNavigator::initializeGoalPose(ActionT::Goal::ConstSharedPtr goal)
    {
      RCLCPP_INFO(
        logger_, "Begin navigating from current location to (%.2f, %.2f)",
        goal->pose.pose.position.x, goal->pose.pose.position.y);

      // Reset state for new action feedback
      start_time_ = clock_->now();
      auto blackboard = bt_action_server_->getBlackboard();
      blackboard->set<int>("number_recoveries", 0);  // NOLINT

      // Update the goal pose on the blackboard
      blackboard->set<geometry_msgs::msg::PoseStamped>(goal_blackboard_id_, goal->pose);
    }
    ```


The recovery counter and start time are both important feedback terms for a client to understand the state of the current task (e.g. if its failing, having problems, or taking exceptionally long). The setting of the goal on the blackboard is taken by the `ComputePathToPose` BT Action node to plan a new route to the goal (and then who\'s path is communicated to the `FollowPath` BT node via the blackboard ID previously set).

> 恢复计数器和开始时间都是客户理解当前任务状态（例如失败、出现问题或者耗时特别长）的重要反馈术语。黑板上设定的目标被`ComputePathToPose` BT Action节点接收，以便规划新的路线到达目标（然后路径被通过之前设定的黑板ID传达给`FollowPath` BT节点）。


The final function implemented is `onLoop`, which is simplified below for tutorial purposes. While anything can be done in this method, which is called as the BT is looping through the tree, it is common to use this as an opportunity to populate any necessary feedback about the state of the navigation request, robot, or metadata that a client might be interested in.

> 最终实现的函数是`onLoop`，为了便于教程，下面简化了它。虽然可以在此方法中做任何事情，即当BT循环遍历树时称为此方法，但常常使用此方法来填充客户可能感兴趣的有关导航请求、机器人或元数据的任何必要反馈。

    ```c++
    void NavigateToPoseNavigator::onLoop()
    {
      auto feedback_msg = std::make_shared<ActionT::Feedback>();

      geometry_msgs::msg::PoseStamped current_pose = ...;
      auto blackboard = bt_action_server_->getBlackboard();
      nav_msgs::msg::Path current_path;
      blackboard->get<nav_msgs::msg::Path>(path_blackboard_id_, current_path);

      ...

      feedback_msg->distance_remaining = distance_remaining;
      feedback_msg->estimated_time_remaining = estimated_time_remaining;

      int recovery_count = 0;
      blackboard->get<int>("number_recoveries", recovery_count);
      feedback_msg->number_of_recoveries = recovery_count;
      feedback_msg->current_pose = current_pose;
      feedback_msg->navigation_time = clock_->now() - start_time_;

      bt_action_server_->publishFeedback(feedback_msg);
    }
    ```

## 2- Exporting the navigator plugin


Now that we have created our custom navigator, we need to export our plugin so that it would be visible to the BT Navigator server. Plugins are loaded at runtime, and if they are not visible, then our server won\'t be able to load it. In ROS 2, exporting and loading plugins is handled by `pluginlib`.

> 现在我们已经创建了自定义导航器，我们需要导出我们的插件，以便它对BT导航服务器可见。插件在运行时加载，如果它们不可见，那么我们的服务器将无法加载它。在ROS 2中，导出和加载插件由`pluginlib`处理。


Coming to our tutorial, class `nav2_bt_navigator::NavigateToPoseNavigator` is loaded dynamically as `nav2_core::NavigatorBase` which is our base class due to the subtleties previously described.

> 来到我们的教程中，`nav2_bt_navigator::NavigateToPoseNavigator` 类会动态加载为 `nav2_core::NavigatorBase`，这是由于之前描述的细节而成为我们的基类。


1.  To export the controller, we need to provide two lines

> 1. 要导出控制器，我们需要提供两行`Wait`

    ```c++
    #include "pluginlib/class_list_macros.hpp"
    PLUGINLIB_EXPORT_CLASS(nav2_bt_navigator::NavigateToPoseNavigator, nav2_core::NavigatorBase)
    ```

Note that it requires pluginlib to export out the plugin\'s class. Pluginlib would provide as macro `PLUGINLIB_EXPORT_CLASS`, which does all the work of exporting.


It is good practice to place these lines at the end of the file, but technically, you can also write at the top.

> 这些行最好放在文件末尾，但从技术上讲，也可以放在顶部。`Wait`


2.  The next step would be to create the plugin\'s description file in the root directory of the package. For example, `navigator_plugin.xml` file in our tutorial package. This file contains the following information

> 下一步就是在包的根目录中创建插件的描述文件。例如，我们教程包中的`navigator_plugin.xml`文件。此文件包含以下信息`Wait`

- `library path`: Plugin\'s library name and it\'s location.
- `class name`: Name of the class.
- `class type`: Type of class.
- `base class`: Name of the base class.
- `description`: Description of the plugin.

    ```xml
    <library path="nav2_bt_navigator">
      <class type="nav2_bt_navigator::NavigateToPoseNavigator" base_class_type="nav2_core::NavigatorBase">
        <description>
          This is pure point-to-point navigation
        </description>
      </class>
    </library>
    ```


3.  Next step would be to export plugin using `CMakeLists.txt` by using CMake function `pluginlib_export_plugin_description_file()`. This function installs the plugin description file to `share` directory and sets ament indexes to make it discoverable.

> 下一步将使用CMake函数`pluginlib_export_plugin_description_file()`使用`CMakeLists.txt`导出插件。此函数将插件描述文件安装到`share`目录，并设置ament索引以使其可检索。

    ```text
    pluginlib_export_plugin_description_file(nav2_core navigator_plugin.xml)
    ```


4.  The plugin description file should also be added to `package.xml`

> `4. package.xml中也应添加插件描述文件。`

    ```xml
    <export>
      <build_type>ament_cmake</build_type>
      <nav2_core plugin="${prefix}/navigator_plugin.xml" />
    </export>
    ```


5.  Compile, and it should be registered. Next, we\'ll use this plugin.

> 编译，应该就注册了。接下来，我们就可以使用这个插件了。

## 3- Pass the plugin name through the params file


To enable the plugin, we need to modify the `nav2_params.yaml` file as below

> 启用插件，我们需要修改`nav2_params.yaml`文件，如下所示`

    ```text
    bt_navigator:
      ros__parameters:
        use_sim_time: true
        global_frame: map
        robot_base_frame: base_link
        transform_tolerance: 0.1
        default_nav_to_pose_bt_xml: replace/with/path/to/bt.xml
        default_nav_through_poses_bt_xml: replace/with/path/to/bt.xml
        goal_blackboard_id: goal
        goals_blackboard_id: goals
        path_blackboard_id: path
        navigators: ['navigate_to_pose', 'navigate_through_poses']
        navigate_to_pose:
          plugin: "nav2_bt_navigator/NavigateToPoseNavigator"
        navigate_through_poses:
          plugin: "nav2_bt_navigator/NavigateThroughPosesNavigator"
    ```


In the above snippet, you can observe the mapping of our `nav2_bt_navigator/NavigateToPoseNavigator` plugin to its id `navigate_to_pose`. To pass plugin-specific parameters we have used `<plugin_id>.<plugin_specific_parameter>`.

> 在上面的片段中，您可以观察到我们的`nav2_bt_navigator/NavigateToPoseNavigator`插件到其id`navigate_to_pose`的映射。 为了传递特定于插件的参数，我们使用了`<plugin_id>.<plugin_specific_parameter>`。

## 4- Run plugin


Run Turtlebot3 simulation with enabled Nav2. Detailed instructions on how to make it run are written at `getting_started`{.interpreted-text role="ref"}. Below is a shortcut command for that:

> 运行 Turtlebot3 仿真，启用 Nav2。有关如何运行的详细说明写在 `getting_started`{.interpreted-text role="ref"} 中。以下是快捷命令：`Wait`

    ```bash
    $ ros2 launch nav2_bringup tb3_simulation_launch.py params_file:=/path/to/your_params_file.yaml
    ```


Then goto RViz and click on the \"2D Pose Estimate\" button at the top and point the location on the map as it was described in `getting_started`{.interpreted-text role="ref"}. The robot will localize on the map and then click on the \"Nav2 goal\" and click on the pose where you want your robot to navigate to. After that navigator will take over with the behavior tree XML file behavior definition provided to it.

> 然后去RViz，点击顶部的“2D Pose Estimate”按钮，按照`getting_started`中描述的位置在地图上指定。机器人将会定位到地图上，然后点击“Nav2 goal”，然后点击您希望机器人导航到的位置。之后，导航程序将使用提供给它的行为树XML文件行为定义文件来接管。
