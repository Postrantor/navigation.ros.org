---
tip: translate by openai@2023-06-03 00:23:53
title: Getting Started
---

This document will take you through the process of installing the binaries and navigating a simulated Turtlebot 3 in the Gazebo simulator.

> 这份文件将引导您安装二进制文件并在 Gazebo 模拟器中导航 Turtlebot 3。

::: note
::: title
Note
:::

See the `build-instructions`{.interpreted-text role="ref"} for other situations such as building from source or working with other types of robots.

> 看一下`build-instructions`{.interpreted-text role="ref"}，了解其他情况，比如源码构建或者处理其他类型的机器人。

:::

::: warning
::: title
Warning
:::

This is a simplified version of the Turtlebot 3 instructions. We highly recommend you follow the [official Turtlebot 3 manual]() if you intend to continue working with this robot beyond the minimal example provided here.

> 这是 Turtlebot 3 说明的简化版本。如果您打算在此处提供的最小示例之外继续使用此机器人，我们强烈建议您遵循[官方 Turtlebot 3 手册]()。

:::

# Installation

1.  Install the [ROS 2 binary packages]() as described in the official docs

> 安装官方文档中描述的[ROS 2 二进制包]()。

2.  Install the packages using your operating system\'s package manager:

> 安装软件包使用您的操作系统的软件包管理器：

    ```bash
    sudo apt install ros-<ros2-distro>-navigation2
    sudo apt install ros-<ros2-distro>-nav2-bringup
    ```

3.  Install the Turtlebot 3 packages (Humble and older):

> 安装 Turtlebot 3 软件包（Humble 和旧版本）：

    ```bash
    sudo apt install ros-<ros2-distro>-turtlebot3-gazebo
    ```

# Running the Example

1.  Start a terminal in your GUI

> 开启你的 GUI 中的终端

2.  Set key environment variables:

> 设置环境变量：

    ```bash
    source /opt/ros/<ros2-distro>/setup.bash
    export TURTLEBOT3_MODEL=waffle
    export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/opt/ros/<ros2-distro>/share/turtlebot3_gazebo/models
    ```

3.  In the same terminal, run

> 在同一个终端中，运行`等待`

    ```bash
    ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False
    ```

    ::: note
    ::: title
    Note
    :::

For `ROS 2 Dashing Diademata` or earlier, use `nav2_simulation_launch.py`. However, it is recommended to use the most recent [ROS 2 LTS distribution](https://ros.org/reps/rep-2000.html) for improved stablity and feature completeness.

> 对于`ROS 2 Dashing Diademata`或更早的版本，请使用`nav2_simulation_launch.py`。但是，建议使用最新的[ROS 2 LTS 发行版](https://ros.org/reps/rep-2000.html)以提高稳定性和功能完整性。

headless defaults to true, if not set to false, gzclient (the 3d view) is not started

> 如果没有设置为 false，headless 默认为 true，gzclient（3D 视图）不会启动。

:::

This launch file will launch Nav2 with the AMCL localizer in the `turtlebot3_world` world. It will also launch the robot state publisher to provide transforms, a Gazebo instance with the Turtlebot3 URDF, and RVIZ.

> 这个启动文件将启动 Nav2，使用`turtlebot3_world`世界中的 AMCL 定位器。它还将启动机器人状态发布器以提供变换，带有 Turtlebot3 URDF 的 Gazebo 实例以及 RVIZ。

If everything has started correctly, you will see the RViz and Gazebo GUIs like this:

> 如果一切都开始正确，你会像这样看到 RViz 和 Gazebo 的 GUI：

![image](/images/rviz/rviz-not-started.png){width="45.0%"}

![image](/images/gazebo/gazebo_turtlebot1.png){width="46.0%"}

4.  If not autostarting, click the \"Startup\" button in the bottom left corner of RViz. This will cause to change to the Active state. It should change appearance to show the map.

> 如果没有自动启动，请在 RViz 左下角点击“启动”按钮。这会导致更改为 Active 状态。它应该改变外观以显示地图。

![Initial appearance of RViz transitioning to the Active state](/images/rviz/rviz_initial.png){.align-center width="700px"}

# Navigating

After starting, the robot initially has no idea where it is. By default, waits for you to give it an approximate starting position. Take a look at where the robot is in the Gazebo world, and find that spot on the map. Set the initial pose by clicking the \"2D Pose Estimate\" button in RViz, and then down clicking on the map in that location. You set the orientation by dragging forward from the down click.

> 开始后，机器人最初不知道自己在哪里。默认情况下，它等待您给出一个大概的起始位置。查看机器人在 Gazebo 世界中的位置，然后在地图上找到该位置。通过在 RViz 中点击“2D Pose Estimate”按钮，然后在该位置的地图上向下单击，可以设置初始姿态。您可以通过从下拉点拖动来设置方向。

If you are using the defaults so far, the robot should look roughly like this.

> 如果你一直使用默认设置，机器人应该大致如下所示。

> ![Approximate starting location of Turtlebot](/images/rviz/rviz-set-initial-pose.png){.align-center width="700px"}

If you don\'t get the location exactly right, that\'s fine. will refine the position as it navigates. You can also, click the \"2D Pose Estimate\" button and try again, if you prefer.

> 如果你没有完全正确地获取位置，没关系。它会随着导航而进行精确定位。你也可以点击“2D Pose Estimate”按钮，再试一次，如果你愿意的话。

Once you\'ve set the initial pose, the transform tree will be complete and is fully active and ready to go. You should see the robot and particle cloud now.

> 一旦你设置了初始姿态，变换树就会完成，并且完全激活，准备好使用了。你现在应该可以看到机器人和粒子云了。

> ![|PN| is ready. Transforms and Costmap show in RViz.](/images/rviz/navstack-ready.png){.align-center width="700px"}

Next, click the \"Navigaton2 Goal\" button and choose a destination. This will call the BT navigator to go to that goal through an action server. You can pause (cancel) or reset the action through the Nav2 rviz plugin shown.

> 接下来，点击“Navigaton2 Goal”按钮并选择目的地。这将通过动作服务器调用 BT 导航器前往该目标。您可以通过显示的 Nav2 rviz 插件暂停（取消）或重置操作。

> ![Setting the goal pose in RViz.](/images/rviz/navigate-to-pose.png){.align-center width="700px"}

Now watch the robot go!

![Navigation2 with Turtlebot 3 Demo](images/navigation_with_recovery_behaviours.gif){.align-center width="700px"}
