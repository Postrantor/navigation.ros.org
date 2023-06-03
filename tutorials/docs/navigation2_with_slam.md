---
tip: translate by openai@2023-06-03 00:08:13
...
---
title: (SLAM) Navigating While Mapping
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [0- Launch Robot Interfaces](#0--launch-robot-interfaces)
  - [1- Launch Navigation2](#1--launch-navigation2)
  - [2- Launch SLAM](#2--launch-slam)
  - [3- Working with SLAM](#3--working-with-slam)
  - [4- Getting Started Simplification](#4--getting-started-simplification)

# Overview


This document explains how to use Nav2 with SLAM. The following steps show ROS 2 users how to generate occupancy grid maps and use Nav2 to move their robot around. This tutorial applies to both simulated and physical robots, but will be completed here on physical robot.

> 这份文件解释了如何使用Nav2配合SLAM。以下步骤向ROS 2用户展示如何生成占用网格地图，并使用Nav2来控制他们的机器人移动。本教程适用于模拟和物理机器人，但我们将在物理机器人上完成。


Before completing this tutorial, completing the `getting_started`{.interpreted-text role="ref"}. is highly recommended especially if you are new to ROS and Navigation2.

> 在完成本教程之前，特别是对于ROS和Navigation2新手，强烈建议完成`getting_started`{.interpreted-text role="ref"}。


In this tutorial we\'ll be using SLAM Toolbox. More information can be found in the [ROSCon talk for SLAM Toolbox](https://vimeo.com/378682207)

> 在本教程中，我们将使用SLAM Toolbox。更多信息可以在[ROSCon讲座中找到SLAM Toolbox](https://vimeo.com/378682207)。

# Requirements


You must install Navigation2, Turtlebot3, and SLAM Toolbox. If you don\'t have them installed, please follow `getting_started`{.interpreted-text role="ref"}.

> 你必须安装Navigation2、Turtlebot3和SLAM Toolbox。如果你还没有安装，请按照`getting_started`{.interpreted-text role="ref"}的指示进行操作。


SLAM Toolbox can be installed via:

> SLAM Toolbox 可以通过以下方式安装：`Wait`

> `sudo apt install ros-<ros2-distro>-slam-toolbox`

or from built from source in your workspace with:

> `git clone -b <ros2-distro>-devel git@github.com:stevemacenski/slam_toolbox.git`

# Tutorial Steps

## 0- Launch Robot Interfaces


For this tutorial, we will use the turtlebot3. If you have another robot, replace with your robot specific interfaces. Typically, this includes the robot state publisher of the URDF, simulated or physical robot interfaces, controllers, safety nodes, and the like.

> 对于本教程，我们将使用turtlebot3。如果您有另一台机器人，请用您的机器人特定接口替换。通常，这包括URDF的机器人状态发布者，模拟或物理机器人接口，控制器，安全节点等等。


Run the following commands first whenever you open a new terminal during this tutorial.

> 在本教程期间，每次打开新终端时，首先运行以下命令。`等待`

- `source /opt/ros/<ros2-distro>/setup.bash`
- `export TURTLEBOT3_MODEL=waffle`


Launch your robot\'s interface and robot state publisher,

> 启动你的机器人的界面和机器人状态发布者，`Wait`

> `ros2 launch turtlebot3_bringup robot.launch.py`

## 1- Launch Navigation2


Launch Navigation without nav2_amcl and nav2_map_server. It is assumed that the SLAM node(s) will publish to /map topic and provide the map-\>odom transform.

> 启动导航而不使用nav2_amcl和nav2_map_server。假设SLAM节点将发布到/map主题并提供map-\>odom变换。

> `ros2 launch nav2_bringup navigation_launch.py`

## 2- Launch SLAM


Bring up your choice of SLAM implementation. Make sure it provides the map-\>odom transform and /map topic. Run Rviz and add the topics you want to visualize such as /map, /tf, /laserscan etc. For this tutorial, we will use [SLAM Toolbox](https://github.com/SteveMacenski/slam_toolbox).

> 带来你选择的SLAM实现。确保它提供map-\>odom变换和/map主题。运行Rviz并添加你想要可视化的主题，如/map，/tf，/laserscan等。对于本教程，我们将使用[SLAM Toolbox]（https://github.com/SteveMacenski/slam_toolbox）。

> `ros2 launch slam_toolbox online_async_launch.py`

## 3- Working with SLAM


Move your robot by requesting a goal through RViz or the ROS 2 CLI, ie:

> 通过RViz或ROS 2 CLI请求目标来移动你的机器人，例如：`等待`。

```bash
    ros2 topic pub /goal_pose geometry_msgs/PoseStamped "{header: {stamp: {sec: 0}, frame_id: 'map'}, pose: {position: {x: 0.2, y: 0.0, z: 0.0}, orientation: {w: 1.0}}}"
```


You should see the map update live! To save this map to file:

> 你应该看到地图实时更新！要保存此地图到文件：`等待`

> `ros2 run nav2_map_server map_saver_cli -f ~/map`

![Navigation2 with SLAM](images/Navigation2_with_SLAM/navigation2_with_slam.gif){.align-center width="700px"}

## 4- Getting Started Simplification


If you\'re only interested in running SLAM in the turtlebot3 getting started sandbox world, we also provide a simple way to enable SLAM as a launch configuration. Rather than individually launching the interfaces, navigation, and SLAM, you can continue to use the `tb3_simulation_launch.py` with `slam` config set to true. We provide the instructions above with the assumption that you\'d like to run SLAM on your own robot which would have separated simulation / robot interfaces and navigation launch files that are combined in `tb3_simulation_launch.py` for the purposes of easy testing.

> 如果您只对在turtlebot3入门沙盒世界中运行SLAM感兴趣，我们还提供了一种简单的方法来启用SLAM作为启动配置。不是单独启动接口、导航和SLAM，而是可以继续使用`tb3_simulation_launch.py`，将`slam`配置设置为true。我们提供了上面的说明，假设您想在自己的机器人上运行SLAM，该机器人将具有分离的模拟/机器人接口和导航启动文件，为了方便测试而组合在`tb3_simulation_launch.py`中。

```bash
    ros2 launch nav2_bringup tb3_simulation_launch.py slam:=True
```
