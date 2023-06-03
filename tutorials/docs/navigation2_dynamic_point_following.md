---
tip: translate by openai@2023-06-02 16:56:50
title: Dynamic Object Following
---

- [Overview](#overview)
- [Tutorial Steps](#tutorial-steps)
  - [0- Create the Behavior Tree](#0--create-the-behavior-tree)
  - [1- Setup Rviz clicked point](#1--setup-rviz-clicked-point)
  - [2- Run Dynamic Object Following in Nav2 Simulation](#2--run-dynamic-object-following-in-nav2-simulation)

```{=html}
<h1 align="center">
  <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
    <iframe width="700" height="450" src="https://www.youtube.com/embed/sRodzrrJChA?autoplay=1" frameborder="1" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  </div>
</h1>
```

# Overview

This tutorial shows how to use Nav2 for a different task other than going from point A to point B. In this case, we will use Nav2 to follow a moving object at a distance indefinitely.

> 这个教程展示了如何使用 Nav2 来完成除了从 A 点到 B 点的其他任务。在这种情况下，我们将使用 Nav2 无限期地跟随一个移动的物体。

This task is useful in cases such as following a person or another robot. Below are some sample videos of applications that could be created using this capability. The \"Carry My Luggage\" RoboCup @ Home test, in which the [CATIE Robotics](https://robotics.catie.fr/) team performs the test successfully and this real (future) world application:

> 这个任务在跟随一个人或另一个机器人的情况下很有用。下面是一些可以使用此功能创建的应用程序的示例视频。CATIE Robotics（https://robotics.catie.fr/）团队成功完成了“Carry My Luggage” RoboCup@Home 测试，以及这个真实（未来）的世界应用：

```{=html}
<h1 align="center">
  <div>
    <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
      <iframe width="450" height="300" src="https://www.youtube.com/embed/lTjKO4M7yZc?autoplay=1&mute=1" frameborder="1" allowfullscreen></iframe>
      <iframe width="450" height="300" src="https://www.youtube.com/embed/KgRKyzsja9Q?autoplay=1&mute=1" frameborder="1" allowfullscreen></iframe>
    </div>
  </div>
</h1>
```

The requirements for this task are as follows:

> 以下是此任务的要求：

- Changes are limited to the behavior tree used to navigate. This behavior tree can be selected in the `NavigateToPose` action when required, or it can be the default behavior tree. It is made up of run-time configurable plugins.

> 更改仅限于用于导航的行为树。 当需要时，可以在`NavigateToPose`操作中选择该行为树，或者可以是默认行为树。 它由运行时可配置的插件组成。

- The configuration of the planner and the controller will not be modified.
- The action will indefinitely run until it is canceled by who initiated it.

The detection of the dynamic object (like a person) to follow is outside the scope of this tutorial. As shown in the following diagram, your application should provide a detector for the object(s) of interest, send the initial pose to the `NavigateToPose` action, and update it on a topic for the duration of the task. Many different types of detectors exist that you can leverage for this application:

> 检测动态对象（如人）的跟随超出了本教程的范围。如下图所示，您的应用程序应提供感兴趣的对象的检测器，将初始姿势发送到`NavigateToPose`操作，并在任务持续期间在主题上更新它。您可以利用许多不同类型的检测器来实现此应用程序：`

![image](images/navigation2_dynamic_point_following/main_diagram.png){width="48.0%"}

# Tutorial Steps

## 0- Create the Behavior Tree

Let\'s start from this simple behavior tree. This behavior tree replans a new path at 1 hz and passes that path to the controller to follow:

> 让我们从这个简单的行为树开始。这个行为树每秒重新规划一条新路径，并将该路径传递给控制器以供跟踪：

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <PipelineSequence name="NavigateWithReplanning">
      <RateController hz="1.0">
        <ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased"/>
      </RateController>
      <FollowPath path="{path}" controller_id="FollowPath"/>
    </PipelineSequence>
  </BehaviorTree>
</root>
```

First, let\'s make this behavior run until there\'s a failure. For this purpose, we will use the `KeepRunningUntilFailure` control node.

> 首先，让这个行为一直运行直到出现失败。 为此，我们将使用`KeepRunningUntilFailure`控制节点。

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <PipelineSequence name="NavigateWithReplanning">
      <RateController hz="1.0">
        <ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased"/>
      </RateController>
      <KeepRunningUntilFailure>
        <FollowPath path="{path}" controller_id="FollowPath"/>
      </KeepRunningUntilFailure>
    </PipelineSequence>
  </BehaviorTree>
</root>
```

We will then use the decorator `GoalUpdater` to accept updates of the dynamic object pose we\'re trying to follow. This node takes as input the current goal and subscribes to the topic `/goal_update`. It set the new goal as `updated_goal` if a new goal on that topic is received.

> 我們將使用裝飾器`GoalUpdater`來接受我們正在嘗試跟隨的動態對象姿態的更新。 這個節點作為當前目標輸入，並訂閱主題`/ goal_update`。 如果在該主題上收到新目標，則將新目標設置為`updated_goal`。

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <PipelineSequence name="NavigateWithReplanning">
      <RateController hz="1.0">
        <GoalUpdater input_goal="{goal}" output_goal="{updated_goal}">
          <ComputePathToPose goal="{updated_goal}" path="{path}" planner_id="GridBased"/>
        </GoalUpdater>
      </RateController>
      <KeepRunningUntilFailure>
        <FollowPath path="{path}" controller_id="FollowPath"/>
      </KeepRunningUntilFailure>
    </PipelineSequence>
  </BehaviorTree>
</root>
```

To stay at a certain distance from the target, we will use the action node `TruncatePath`. This node modifies a path making it shorter so we don\'t try to navigate into the object of interest. We can set up the desired distance to the goal using the input port `distance`.

> 使用动作节点`TruncatePath`保持与目标的一定距离。该节点修改路径使其变短，因此我们不会试图导航到感兴趣的对象中。我们可以使用输入端口`distance`设置到目标的期望距离。

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <PipelineSequence name="NavigateWithReplanning">
      <RateController hz="1.0">
        <Sequence>
          <GoalUpdater input_goal="{goal}" output_goal="{updated_goal}">
            <ComputePathToPose goal="{updated_goal}" path="{path}" planner_id="GridBased"/>
          </GoalUpdater>
         <TruncatePath distance="1.0" input_path="{path}" output_path="{truncated_path}"/>
        </Sequence>
      </RateController>
      <KeepRunningUntilFailure>
        <FollowPath path="{truncated_path}" controller_id="FollowPath"/>
      </KeepRunningUntilFailure>
    </PipelineSequence>
  </BehaviorTree>
</root>
```

Now, you may save this behavior tree and use it in our navigation task.

> 现在，您可以保存此行为树并将其用于我们的导航任务。

For reference, this exact behavior tree is [made available](https://github.com/ros-planning/navigation2/blob/main/nav2_bt_navigator/behavior_trees/follow_point.xml) to you batteries included in the `nav2_bt_navigator` package.

> 参考用途，这个行为树可以在`nav2_bt_navigator`软件包中找到，[可以在这里下载](https://github.com/ros-planning/navigation2/blob/main/nav2_bt_navigator/behavior_trees/follow_point.xml)。

## 1- Setup Rviz clicked point

We are going to use RViz instead of a full application so you can test at home without finding a detector to get started. We will use the \"clicked point\" button on the toolbar to substitute object detections to provide goal updates to Nav2. This button allows you to publish coordinates in the topic `/clicked_point`. This point needs to be sent to the behavior tree, using the program `clicked_point_to_pose`, from [this repo](https://github.com/fmrico/nav2_test_utils). Clone this repo in your workspace, build, and type in a terminal.

> 我们将使用 RViz 而不是完整的应用程序，这样您就可以在家中测试而无需找到探测器来开始。我们将使用工具栏上的“点击点”按钮来替代物体检测，以向 Nav2 提供目标更新。此按钮允许您在主题`/clicked_point`中发布坐标。此点需要使用[该 repo](https://github.com/fmrico/nav2_test_utils)中的程序`clicked_point_to_pose`发送到行为树。在您的工作区中克隆此 repo，构建并在终端中键入。

`ros2 run nav2_test_utils clicked_point_to_pose`

Optionally, you can remap this topic in your rviz configuration file to `goal_updates`.

> 可选择性地，您可以在您的 rviz 配置文件中重新映射此主题为`goal_updates`。

## 2- Run Dynamic Object Following in Nav2 Simulation

Start Nav2 in one terminal:

> 启动 Nav2 在一个终端：`等待`

`ros2 launch nav2_bringup tb3_simulation_launch.py default_bt_xml_filename:=/path/to/bt.xml`

Open RViz and, after initialize the robot position, command the robot to navigate to any position. Use the button clicked point to simulate a new detection of the object of interest, as shown in the video in the head of this tutorial.

> 打开 RViz，并在初始化机器人位置后，指令机器人导航到任何位置。使用按钮点击的点来模拟感兴趣物体的新检测，如本教程头部视频所示。`

When you have a detector detecting your obstacle at a higher rate (1 hz, 10 hz, 100 hz) you will see a far more reactive robot following your detected object of interest!

> 当你有一个检测器以更高的速率（1 Hz，10 Hz，100 Hz）检测你的障碍物时，你会看到一个更加反应灵敏的机器人跟随你检测到的感兴趣的物体！

```{=html}
<h1 align="center">
  <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
    <iframe width="700" height="450" src="https://www.youtube.com/embed/r4fIkcktZUM?autoplay=1" frameborder="1" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
  </div>
</h1>
```
