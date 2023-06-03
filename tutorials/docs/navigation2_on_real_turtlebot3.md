---
tip: translate by openai@2023-06-03 00:07:12
...
---
title: Navigating with a Physical Turtlebot 3
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [0- Setup Your Enviroment Variables](#0--setup-your-enviroment-variables)
  - [1- Launch Turtlebot 3](#1--launch-turtlebot-3)
  - [2- Launch Nav2](#2--launch-nav2)
  - [3- Launch RVIZ](#3--launch-rviz)
  - [4- Initialize the Location of Turtlebot 3](#4--initialize-the-location-of-turtlebot-3)
  - [5- Send a Goal Pose](#5--send-a-goal-pose)

```{=html}
<h1 align="center">
  <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
    <iframe width="700" height="450" src="https://www.youtube.com/embed/ZeCds7Sv-5Q?autoplay=1" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  </div>
</h1>
```

# Overview


This tutorial shows how to control and navigate Turtlebot 3 using the ROS 2 Nav2 on a physical Turtlebot 3 robot. Before completing this tutorials, completing `getting_started`{.interpreted-text role="ref"} is highly recommended especially if you are new to ROS and Nav2.

> 这个教程展示了如何使用ROS 2 Nav2在物理Turtlebot 3机器人上控制和导航Turtlebot 3。在完成本教程之前，特别是如果您是ROS和Nav2的新手，强烈建议完成`getting_started`{.interpreted-text role="ref"}。


This tutorial may take about 1 hour to complete. It depends on your experience with ROS, robots, and what computer system you have.

> 这个教程可能需要大约1小时来完成。这取决于您对ROS、机器人和您拥有的计算机系统的经验。`

# Requirements


You must install Nav2, Turtlebot3. If you don\'t have them installed, please follow `getting_started`{.interpreted-text role="ref"}.

> 你必须安装Nav2和Turtlebot3。如果你还没有安装，请按照`getting_started`{.interpreted-text role="ref"}指示操作。

# Tutorial Steps

## 0- Setup Your Enviroment Variables


Run the following commands first whenever you open a new terminal during this tutorial.

> 在本教程中，每次打开新终端时，请首先运行以下命令。`等待`

- `source /opt/ros/<ros2-distro>/setup.bash`
- `export TURTLEBOT3_MODEL=waffle`

## 1- Launch Turtlebot 3


You will need to launch your robot\'s interface,

> 你需要启动你的机器人的界面。

> `ros2 launch turtlebot3_bringup robot.launch.py  use_sim_time:=False`

## 2- Launch Nav2


You need to have a map of the environment where you want to Navigate Turtlebot 3, or create one live with SLAM.

> 你需要有一张你想要导航Turtlebot 3的环境地图，或者用SLAM实时创建一张。


In case you are interested, there is a use case tutorial which shows how to use Nav2 with SLAM. `navigation2-with-slam`{.interpreted-text role="ref"}.

> 如果您感兴趣，有一个用例教程可以演示如何使用Nav2与SLAM。`navigation2-with-slam`。


Required files:

> `所需文件：`

> - `your-map.map`
> - `your-map.yaml`

`<your_map>.yaml` is the configuration file for the map we want to provide Nav2. In this case, it has the map resolution value, threshold values for obstacles and free spaces, and a map file location. You need to make sure these values are correct. More information about the map.yaml can be found [here](http://wiki.ros.org/map_server).


Launch Nav2. If you set autostart:=False, you need to click on the start button in RViz to initialize the nodes. Make sure [use_sim time]{.title-ref} is set to **False**, because we want to use the system time instead of the time simulation time from Gazebo.

> 如果你将autostart设置为False，你需要点击RViz中的开始按钮来初始化节点。确保[use_sim time]{.title-ref}设置为**False**，因为我们想要使用系统时间而不是来自Gazebo的模拟时间。启动Nav2。

`ros2 launch nav2_bringup bringup_launch.py use_sim_time:=False autostart:=False map:=/path/to/your-map.yaml`

Note: Don\'t forget to change **/path/to/your-map.yaml** to the actual path to the your-map.yaml file.

## 3- Launch RVIZ


Launch RVIZ with a pre-defined configuration file.

> 启动RVIZ，使用预定义的配置文件。

> `ros2 run rviz2 rviz2 -d $(ros2 pkg prefix nav2_bringup)/share/nav2_bringup/rviz/nav2_default_view.rviz`


Now, you should see a shadow of Turtlebot 3 robot model in the center of the plot in Rviz. Click on the Start button (Bottom Left) if you set the auto_start parameter to false. Then, the map should appear in RViz.

> 现在，您应该在Rviz中的中心看到Turtlebot 3机器人模型的阴影。如果您将auto_start参数设置为false，请单击Start按钮（左下角）。然后，地图应该出现在RViz中。

![image](images/Navigation2_on_real_Turtlebot3/rviz_after_launch_view.png){width="48.0%"}

![image](images/Navigation2_on_real_Turtlebot3/rviz_slam_map_view.png){width="45.0%"}

## 4- Initialize the Location of Turtlebot 3


First, find where the robot is on the map. Check where your robot is in the room.

> 首先，在地图上找到机器人的位置。检查你的机器人在房间里的位置。


Set the pose of the robot in RViz. Click on the 2D Pose Estimate button and point the location of the robot on the map. The direction of the green arrow is the orientation of Turtlebot.

> 设置RViz中机器人的姿态。点击2D姿态估计按钮，在地图上指出机器人的位置。绿色箭头的方向是Turtlebot的方向。

![Set initial pose in RViz](images/Navigation2_on_real_Turtlebot3/rviz_set_initial_pose.png){.align-center width="700px"}


Now, the 3D model of Turtlebot should move to that location. A small error in the estimated location is tolerable.

> 现在，Turtlebot的3D模型应该移动到那个位置。估计位置的小误差是可以容忍的。

## 5- Send a Goal Pose


Pick a target location for Turtlebot on the map. You can send Turtlebot 3 a goal position and a goal orientation by using the **Nav2 Goal** or the **GoalTool** buttons.

> 选择一个Turtlebot在地图上的目标位置。您可以通过使用**Nav2 Goal**或**GoalTool**按钮向Turtlebot 3发送目标位置和目标方向。

Note: Nav2 Goal button uses a ROS 2 Action to send the goal and the GoalTool publishes the goal to a topic.

![Send goal pose in RViz](images/Navigation2_on_real_Turtlebot3/rviz_send_goal.png){.align-center width="700px"}


Once you define the target pose, Nav2 will find a global path and start navigating the robot on the map.

> 一旦你定义了目标姿势，Nav2 将会找到一条全局路径，并开始在地图上导航机器人。

![Robot navigating in RViz](images/Navigation2_on_real_Turtlebot3/rviz_robot_navigating.png){.align-center width="700px"}


Now, you can see that Turtlebot 3 moves towards the goal position in the room. See the video below.

> 现在，您可以看到Turtlebot 3朝着房间中的目标位置移动。请参见下面的视频。
