---
tip: translate by openai@2023-06-03 00:38:11
...

---

## title: Writing a New Behavior Plugin

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1- Creating a new Behavior Plugin](#1--creating-a-new-behavior-plugin)
  - [2- Exporting the Behavior Plugin](#2--exporting-the-behavior-plugin)
  - [3- Pass the plugin name through params file](#3--pass-the-plugin-name-through-params-file)
  - [4- Run Behavior Plugin](#4--run-behavior-plugin)

# Overview

This tutorial shows how to create you own Behavior Plugin. The Behavior Plugins live in the behavior server. Unlike the planner and controller servers, each behavior will host its own unique action server. The planners and controllers have the same API as they accomplish the same task. However, recoveries can be used to do a wide variety of tasks, so each behavior can have its own unique action message definition and server. This allows for massive flexibility in the behavior server enabling any behavior action imaginable that doesn\'t need to have other reuse.

> 这篇教程展示了如何创建自己的行为插件。行为插件位于行为服务器中。与计划和控制服务器不同，每个行为都将拥有自己独特的动作服务器。计划和控制器具有相同的 API，因为它们完成相同的任务。但是，恢复可以用来完成各种各样的任务，因此每个行为都可以拥有自己独特的动作消息定义和服务器。这样可以极大地提高行为服务器的灵活性，使得任何不需要其他重用的行为动作都可以实现。

# Requirements

- ROS 2 (binary or build-from-source)
- Nav2 (Including dependencies)
- Gazebo
- Turtlebot3

# Tutorial Steps

## 1- Creating a new Behavior Plugin

We will create a simple send sms behavior. It will use Twilio to send a message via SMS to a remote operations center. The code in this tutorial can be found in [navigation_tutorials](https://github.com/ros-planning/navigation2_tutorials) repository as `nav2_sms_behavior`. This package can be a considered as a reference for writing Behavior Plugin.

> 我們將建立一個簡單的發送簡訊行為。它將使用 Twilio 通過簡訊發送消息到遠程操作中心。本教程中的代碼可以在[navigation_tutorials](https://github.com/ros-planning/navigation2_tutorials)存儲庫中找到，作為`nav2_sms_behavior`。這個包可以被認為是寫行為插件的參考。

Our example plugin implements the plugin class of `nav2_core::Behavior`. However, we have a nice wrapper for actions in `nav2_behaviors`, so we use the `nav2_behaviors::TimedBehavior` base class for this application instead. This wrapper class derives from the `nav2_core` class so it can be used as a plugin, but handles the vast majority of ROS 2 action server boiler plate required.

> 我們的示例插件實現了`nav2_core::Behavior`的插件類。但是，我們在`nav2_behaviors`中有一個很好的動作包裝，所以我們為此應用程序使用`nav2_behaviors::TimedBehavior`基類。這個包裝類從`nav2_core`類派生，因此可以用作插件，但可以處理 ROS 2 動作伺服器的絕大部分鍋爐板。

The base class from `nav2_core` provides 4 pure virtual methods to implement a Behavior Plugin. The plugin will be used by the behavior server to host the plugins, but each plugin will provide their own unique action server interface. Lets learn more about the methods needed to write a Behavior Plugin **if you did not use the \`\`nav2_behaviors\`\` wrapper**.

> 基类来自`nav2_core`提供 4 个纯虚拟方法来实现一个行为插件。该插件将由行为服务器使用，但每个插件将提供自己的唯一的动作服务器接口。让我们更多地了解编写行为插件所需的方法，**如果您没有使用\`\`nav2_behaviors\`\`包装器**。

---

**Virtual method** **Method description** **Requires override?**

> `虚拟方法` `方法描述` `需要重写？`

configure() Method is called at when server enters on_configure state. Ideally this methods should perform declarations of ROS parameters and initialization of behavior\'s member variables. This method takes 4 input params: shared pointer to parent node, behavior name, tf buffer pointer and shared pointer to a collision checker Yes

> `configure() 方法在服务器进入on_configure状态时被调用。理想情况下，这些方法应该执行ROS参数的声明和行为成员变量的初始化。此方法需要4个输入参数：指向父节点的共享指针，行为名称，tf缓冲指针和指向碰撞检查器的共享指针。`

activate() Method is called when behavior server enters on_activate state. Ideally this method should implement operations which are neccessary before the behavior to an active state. Yes

> `activate()方法在行为服务器进入on_activate状态时被调用。理想情况下，此方法应实现行为进入活动状态所必需的操作。是的`

deactivate() Method is called when behavior server enters on_deactivate state. Ideally this method should implement operations which are neccessary before behavior goes to an inactive state. Yes

> 方法 deactivate()在行为服务器进入 on_deactivate 状态时被调用。理想情况下，在行为进入非活动状态之前，应该实现必要的操作。是的。

cleanup() Method is called when behavior server goes to on_cleanup state. Ideally this method should clean up resoures which are created for the behavior. Yes

> `清理()方法在行为服务器进入on_cleanup状态时被调用。理想情况下，此方法应清理为行为创建的资源。是的`

---

For the `nav2_behaviors` wrapper, which provides the ROS 2 action interface and boilerplate, we have 4 virtual methods to implement. This tutorial uses this wrapper so these are the main elements we will address.

> 对于提供 ROS 2 动作界面和样板的`nav2_behaviors`包装器，我们有 4 个虚拟方法需要实现。本教程使用此包装器，因此这些是我们将要处理的主要元素。

---

**Virtual method** **Method description** **Requires override?**

> **虚拟方法** **方法描述** **需要重写？**

onRun() Method is called immediately when a new behavior action request is received. Gives the action goal to process and should start behavior initialization / process. Yes

> `onRun() 方法会在收到新行为操作请求时立即调用。给出要处理的行为目标，并应该开始行为初始化/处理。是的。`

onCycleUpdate() Method is called at the behavior update rate and should complete any necessary updates. An example for spinning is computing the command velocity for the current cycle, publishing it and checking for completion. Yes

> `onCycleUpdate() 方法以行为更新率调用，应完成所有必要的更新。旋转的示例是计算当前周期的命令速度，发布它并检查完成情况。是的`

onConfigure() Method is called when behavior server enters on_configure state. Ideally this method should implement operations which are neccessary before behavior goes to an configured state (get parameters, etc). No

> `onConfigure()` 方法在行为服务器进入 on_configure 状态时被调用。理想情况下，这个方法应该实现行为在进入配置状态之前所需的操作（获取参数等）。`Wait`

onCleanup() Method is called when behavior server goes to on_cleanup state. Ideally this method should clean up resoures which are created for the behavior. No

> `onCleanup() 方法在行为服务器进入on_cleanup状态时被调用。理想情况下，这个方法应该清理为行为创建的资源。Wait`

---

For this tutorial, we will be using methods `onRun()`, `onCycleUpdate()`, and `onConfigure()` to create the SMS behavior. `onConfigure()` will be skipped for brevity, but only includes declaring of parameters.

> 对于这个教程，我们将使用方法`onRun()`，`onCycleUpdate()`和`onConfigure()`来创建 SMS 行为。 为了简洁起见，`onConfigure()`将被跳过，但仅包括声明参数。

In recoveries, `onRun()` method must set any initial state and kick off the behavior behavior. For the case of our call for help behavior behavior, we can trivially compute all of our needs in this method.

> 在恢复期间，`onRun()` 方法必须设置任何初始状态并启动行为行为。对于我们求助行为的情况，我们可以在这个方法中很容易地计算出所有需要的内容。

    ```c++
    Status SendSms::onRun(const std::shared_ptr<const Action::Goal> command)
    {
      std::string response;
      bool message_success = _twilio->send_message(
        _to_number,
        _from_number,
        command->message,
        response,
        "",
        false);

      if (!message_success) {
        RCLCPP_INFO(node_->get_logger(), "SMS send failed.");
        return Status::FAILED;
      }

      RCLCPP_INFO(node_->get_logger(), "SMS sent successfully!");
      return Status::SUCCEEDED;
    }
    ```

We receive a action goal, `command`, in which we want to process. `command` contains a field `message` that contains the message we want to communicate to our mothership. This is the \"call for help\" message that we want to send via SMS to our brothers in arms in the operations center.

> 我们收到一个行动目标`命令`，我们想要处理它。`命令`包含一个字段`消息`，其中包含我们想要传达给母舰的消息。这是我们想通过短信发送给作战中心的“求救”信息。`等待`

We use the service Twilio to complete this task. Please [create an account](https://www.twilio.com/) and get all the relavent information needed for creating the service (e.g. `account_sid`, `auth_token`, and a phone number). You can set these values as parameters in your configuration files corresponding to the `onConfigure()` parameter declarations.

> 我们使用 Twilio 服务来完成这项任务。请[创建一个帐户](https://www.twilio.com/)并获取创建服务所需的所有相关信息（例如`account_sid`，`auth_token`和电话号码）。您可以将这些值设置为与`onConfigure()`参数声明相对应的配置文件中的参数。`Wait`

We use the `_twilio` object to send our message with your account information from the configuration file. We send the message and log to screen whether or not the message was sent successfully or not. We return a `FAILED` or `SUCCEEDED` depending on this value to be returned to the action client.

> 我们使用`_twilio`对象，使用您的配置文件中的帐户信息发送消息。我们发送消息并记录到屏幕上，消息是否发送成功。我们根据返回的值返回`FAILED`或`SUCCEEDED`给行动客户端。

`onCycleUpdate()` is trivially simple as a result of our short-running behavior behavior. If the behavior was instead longer running like spinning, navigating to a safe area, or getting out of a bad spot and waiting for help, then this function would be checking for timeouts or computing control values. For our example, we simply return success because we already completed our mission in `onRun()`.

    ```c++
    Status SendSms::onCycleUpdate()
    {
      return Status::SUCCEEDED;
    }
    ```

The remaining methods are not used and not mandatory to override them.

> 剩余的方法没有被使用，也不是必须重写它们。

## 2- Exporting the Behavior Plugin

Now that we have created our custom behavior, we need to export our Behavior Plugin so that it would be visible to the behavior server. Plugins are loaded at runtime and if they are not visible, then our behavior server won\'t be able to load it. In ROS 2, exporting and loading plugins is handled by `pluginlib`.

> 现在我们已经创建了自定义行为，我们需要导出我们的行为插件，以便它对行为服务器可见。插件在运行时加载，如果它们不可见，那么我们的行为服务器就无法加载它。在 ROS 2 中，导出和加载插件由`pluginlib`处理。

Coming to our tutorial, class `nav2_sms_bahavior::SendSms` is loaded dynamically as `nav2_core::Behavior` which is our base class.

> 来到我们的教程中，`nav2_sms_bahavior::SendSms`类动态加载为`nav2_core::Behavior`，这是我们的基类。

1.  To export the behavior, we need to provide two lines

> 1. 要导出行为，我们需要提供两行`等待`。

    ```c++
    #include "pluginlib/class_list_macros.hpp"
    PLUGINLIB_EXPORT_CLASS(nav2_sms_bahavior::SendSms, nav2_core::Behavior)
    ```

Note that it requires pluginlib to export out plugin\'s class. Pluginlib would provide as macro `PLUGINLIB_EXPORT_CLASS` which does all the work of exporting.

It is good practice to place these lines at the end of the file but technically, you can also write at the top.

> 这些行最好放在文件末尾，但技术上也可以放在顶部。

2.  Next step would be to create plugin\'s description file in the root directory of the package. For example, `behavior_plugin.xml` file in our tutorial package. This file contains following information

> 下一步是在包的根目录中创建插件的描述文件。例如，我们教程包中的`behavior_plugin.xml`文件。该文件包含以下信息`Wait`

> - `library path`: Plugin\'s library name and it\'s location.
> - `class name`: Name of the class.
> - `class type`: Type of class.
> - `base class`: Name of the base class.
> - `description`: Description of the plugin.

    ```xml
    <library path="nav2_sms_behavior_plugin">
      <class name="nav2_sms_behavior/SendSms" type="nav2_sms_behavior::SendSms" base_class_type="nav2_core::Behavior">
        <description>This is an example plugin which produces an SMS text message recovery.</description>
      </class>
    </library>
    ```

3.  Next step would be to export plugin using `CMakeLists.txt` by using cmake function `pluginlib_export_plugin_description_file()`. This function installs plugin description file to `share` directory and sets ament indexes to make it discoverable.

> 下一步是使用 cmake 函数 `pluginlib_export_plugin_description_file()` 通过 `CMakeLists.txt` 来导出插件。此函数会安装插件描述文件到 `share` 目录，并设置 ament 索引以使其可被发现。

    ```text
    pluginlib_export_plugin_description_file(nav2_core behavior_plugin.xml)
    ```

4.  Plugin description file should also be added to `package.xml`

> `插件描述文件也应该添加到`package.xml`中

    ```xml
    <export>
      <build_type>ament_cmake</build_type>
      <nav2_core plugin="${prefix}/behavior_plugin.xml" />
    </export>
    ```

5.  Compile and it should be registered. Next, we\'ll use this plugin.

> 编译它，它应该被注册。接下来，我们将使用这个插件。

## 3- Pass the plugin name through params file

To enable the plugin, we need to modify the `nav2_params.yaml` file as below to replace following params .. code-block:: text

> 启用插件，我们需要修改`nav2_params.yaml`文件，如下所示，以替换以下参数.. 代码块：文本`

> behavior_server: (Humble and later) recoveries_server: (Galactic and earlier) ros\_\_parameters: costmap_topic: local_costmap/costmap_raw footprint_topic: local_costmap/published_footprint cycle_frequency: 10.0 behavior_plugins: \[\"spin\", \"backup\", \"wait\"\] (Humble and later) recovery_plugins: \[\"spin\", \"backup\", \"wait\"\] (Galactic and earlier) spin: plugin: \"nav2_behaviors/Spin\" backup: plugin: \"nav2_behaviors/BackUp\" wait: plugin: \"nav2_behaviors/Wait\" global_frame: odom robot_base_frame: base_link transform_timeout: 0.1 use_sim_time: true simulate_ahead_time: 2.0 max_rotational_vel: 1.0 min_rotational_vel: 0.4 rotational_acc_lim: 3.2

with

> 等待

    ```yaml
    behavior_server: (Humble and newer)
    recoveries_server: (Galactic and earlier)
      ros__parameters:
        local_costmap_topic: local_costmap/costmap_raw
        local_footprint_topic: local_costmap/published_footprint
        global_costmap_topic: global_costmap/costmap_raw
        global_footprint_topic: global_costmap/published_footprint
        cycle_frequency: 10.0
        behavior_plugins: ["spin", "backup", "wait","send_sms"] (Humble and newer)
        recovery_plugins: ["spin", "backup", "wait","send_sms"] (Galactic and earlier)
        spin:
          plugin: "nav2_behaviors/Spin"
        backup:
          plugin: "nav2_behaviors/BackUp"
        wait:
          plugin: "nav2_behaviors/Wait"
        send_sms:
          plugin: "nav2_sms_behavior/SendSms"
        account_sid: ... # your sid
        auth_token: ... # your token
        from_number: ... # your number
        to_number: ... # the operations center number
        global_frame: odom
        robot_base_frame: base_link
        transform_timeout: 0.1
        use_sim_time: true
        simulate_ahead_time: 2.0
        max_rotational_vel: 1.0
        min_rotational_vel: 0.4
        rotational_acc_lim: 3.2
    ```

In the above snippet, you can observe that we add the SMS behavior under the `send_sms` ROS 2 action server name. We also tell the behavior server that the `send_sms` is of type `SendSms` and give it our parameters for your Twilio account.

> 在上面的片段中，您可以观察到我们在`send_sms` ROS 2 动作服务器名称下添加了 SMS 行为。我们还告诉行为服务器`send_sms`的类型是`SendSms`，并为您的 Twilio 帐户提供参数。

## 4- Run Behavior Plugin

Run Turtlebot3 simulation with enabled Nav2. Detailed instruction how to make it are written at `getting_started`{.interpreted-text role="ref"}. Below is shortcut command for that:

> 运行 Turtlebot3 模拟，并启用 Nav2。详细的指令如何做到这一点写在`getting_started`中。下面是快捷命令：`等待`

    ```bash
    $ ros2 launch nav2_bringup tb3_simulation_launch.py params_file:=/path/to/your_params_file.yaml
    ```

In a new terminal run:

> 在新的终端中运行：`Wait`

    ```bash
    $ ros2 action send_goal "send_sms" nav2_sms_behavior/action/SendSms "{message : Hello!! Navigation2 World }"
    ```
