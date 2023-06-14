---
tip: translate by openai@2023-06-03 00:42:47
title: Writing a New Behavior Tree Plugin
---

- [Overview](#overview)
- [Requirements](#requirements)
- [Tutorial Steps](#tutorial-steps)
  - [1- Creating a new BT Plugin](#1--creating-a-new-bt-plugin)
  - [2- Exporting the planner plugin](#2--exporting-the-planner-plugin)
  - [3- Add plugin library name to config](#3--add-plugin-library-name-to-config)
  - [4- Run Your Custom plugin](#4--run-your-custom-plugin)

# Overview

This tutorial shows how to create you own behavior tree (BT) plugin. The BT plugins are used as nodes in the behavior tree XML processed by the BT Navigator for navigation logic.

> 这篇教程展示了如何创建自己的行为树(BT)插件。BT 插件作为行为树 XML 中的节点，由 BT Navigator 处理导航逻辑。

# Requirements

- ROS 2 (binary or build-from-source)
- Nav2 (Including dependencies)
- Gazebo
- Turtlebot3

# Tutorial Steps

## 1- Creating a new BT Plugin

We will create a simple BT plugin node to perform an action in another server. For this example, we\'re going to analyze the simpliest behavior tree action node in the `nav2_behavior_tree` package, the `wait` node. Beyond this example of an action BT node, you can also create custom decorator, condition, and control nodes. Each node type has as unique role in the behavior tree to perform actions like planning, control the flow of the BT, check the status of a condition, or modify the output of other BT nodes.

> 我们将创建一个简单的 BT 插件节点来在另一台服务器上执行操作。对于这个例子，我们将分析`nav2_behavior_tree`包中最简单的行为树动作节点`Wait`。除了这个动作 BT 节点的例子之外，您还可以创建自定义装饰器，条件和控制节点。每种节点类型在行为树中都有独特的作用，可以执行诸如规划，控制 BT 流程，检查条件状态或修改其他 BT 节点输出等操作。

The code in this tutorial can be found in [nav2_behavior_tree](https://github.com/ros-planning/navigation2/tree/main/nav2_behavior_tree) package as the `wait_action` node. This action node can be a considered as a reference for writing other action node plugins.

> 代码在本教程中可以在[nav2_behavior_tree]包中找到，作为 `wait_action` 动作节点。这个动作节点可以作为编写其他动作节点插件的参考。

Our example plugin inherits from the base class `nav2_behavior_tree::BtActionNode`. The base class is a wrapper on the BehaviorTree.CPP `BT::ActionNodeBase` that simplifies BT action nodes that utilize ROS 2 action clients. An `BTActionNode` is both a BT action and uses ROS 2 action network interfaces for calling a remote server to do some work.

> 我们的示例插件继承自基类`nav2_behavior_tree::BtActionNode`。该基类是 BehaviorTree.CPP `BT::ActionNodeBase`的包装器，简化了使用 ROS 2 动作客户端的 BT 动作节点。`BTActionNode`既是 BT 动作，又使用 ROS 2 动作网络接口调用远程服务器执行某些工作。

When working with other types of BT nodes (e.g. decorator, control, condition) use the corresponding BT node, `BT::DecoratorNode`, `BT::ControlNode`, or `BT::ConditionNode`. For BT action nodes that do _not_ utilize ROS 2 action interfaces, use the `BT::ActionNodeBase` base class itself.

> 当使用其他类型的 BT 节点(例如装饰器、控制器、条件)时，请使用相应的 BT 节点`BT::DecoratorNode`、`BT::ControlNode`或`BT::ConditionNode`。对于不使用 ROS 2 动作接口的 BT 动作节点，请使用`BT::ActionNodeBase`基类本身。

The `BTActionNode` class provides 5 virtual methods to use, in addition to the information provided in the constructor. Lets learn more about the methods needed to write a BT action plugin.

> `BTActionNode`类提供了 5 个虚拟方法可供使用，除了构造函数中提供的信息之外。让我们了解更多关于编写 BT 动作插件所需的方法。

---

**method** **Method description** **Required?**

> `方法` `方法描述` `是否必要？`

Constructor Constructor to indicate the corresponding XML tag name to that matches the plugin, the name of the action server to call using the plugin, and any BehaviorTree.CPP special configurations required. Yes

> 构造函数构造函数以指示与插件匹配的 XML 标签名称，使用插件调用的操作服务器的名称以及 BehaviorTree.CPP 所需的任何特殊配置。是的

providedPorts() A function to define the input and output ports a BT node may have. These are analogous to parameters that are defined in the BT XML by hardcoded values or by the value of other output ports of other nodes. Yes

> `providedPorts()`一个函数来定义 BT 节点可能具有的输入和输出端口。这些类似于在 BT XML 中通过硬编码值或其他节点的输出端口的值定义的参数。是的

on_tick() Method is called when this BT node is ticked by the behavior tree while executing. This should be used to get dynamic updates like new blackboard values, input ports, or parameters. May also reset state for the action. No

> `on_tick()` 方法在行为树执行时调用时被调用。这应该用于获取动态更新，如新的黑板值，输入端口或参数。也可以重置动作的状态。

on_wait_for_result() Method is called when the behavior tree node is waiting for a result from the ROS 2 action server it called. This could be used to check for updates to preempt the current task, check for a timeout, or anything to compute while waiting for the action to complete. No

> `on_wait_for_result()` 方法在行为树节点等待 ROS 2 动作服务器返回结果时被调用。可以用它来检查是否有更新来抢占当前任务，检查超时或者等待动作完成时计算任何东西。

on_success() Method is called when the ROS 2 action server returns a successful result. Returns the value the BT node will report back to the tree. No

> `on_success()` 方法在 ROS 2 动作服务器返回成功结果时被调用。返回 BT 节点将报告给树的值。没有

on_aborted() Method is called when the ROS 2 action server returns an aborted result. Returns the value the BT node will report back to the tree. No

> `on_aborted()` 方法在 ROS 2 动作服务器返回中止结果时被调用。返回 BT 节点将向树报告的值。没有

on_cancelled() MMethod is called when the ROS 2 action server returns a cancelled result. Returns the value the BT node will report back to the tree. No

> `on_cancelled()` 方法在ROS 2动作服务器返回取消结果时调用。 返回BT节点将报告给树的值。没有

---

For this tutorial, we will only be using the `on_tick()` method.

> 对于本教程，我们只会使用 `on_tick()` 方法。

In the constructor, we need to get any non-variable parameters that apply to the behavior tree node. In this example, we need to get the value of the duration to sleep from the input port of the behavior tree XML.

> 在构造函数中，我们需要从行为树 XML 的输入端口获得适用于行为树节点的任何非变量参数。在这个例子中，我们需要获得睡眠时长的值。

```c++
WaitAction::WaitAction(
  const std::string & xml_tag_name,
  const std::string & action_name,
  const BT::NodeConfiguration & conf)
: BtActionNode<nav2_msgs::action::Wait>(xml_tag_name, action_name, conf)
{
  int duration;
  getInput("wait_duration", duration);
  if (duration <= 0) {
    RCLCPP_WARN(
      node_->get_logger(), "Wait duration is negative or zero "
      "(%i). Setting to positive.", duration);
    duration *= -1;
  }

  goal_.time.sec = duration;
}
```

Here, we give the input of the `xml_tag_name` which tells the BT node plugin the string in the XML that corresponds to this node. This will be seen later when we register this BT node as a plugin. It also takes in the string name of the action server that it will call to execute some behavior. Finally, a set of configurations that we can safely ignore for the purposes of most node plugins.

> 这里，我们给出`xml_tag_name`的输入，它告诉 BT 节点插件 XML 中与此节点对应的字符串。当我们将此 BT 节点注册为插件时，这将在稍后可见。它还接受它将调用执行某些行为的动作服务器的字符串名称。最后，一组可以安全忽略的配置，对于大多数节点插件来说都是可以忽略的。

We then call the `BTActionNode` constructor. As can be seen, its templated by the ROS 2 action type, so we give it the `nav2_msgs::action::Wait` action message type and forward our other inputs. The `BTActionNode` as the `tick()` method which is called directly by the behavior tree when this node is called from the tree. `on_tick()` is then called along with the action client goal.

> 然后我们调用`BTActionNode`构造函数。正如可以看到的，它是通过 ROS 2 动作类型模板化的，因此我们给它`nav2_msgs::action::Wait`动作消息类型，并转发我们的其他输入。 `BTActionNode`具有`tick()`方法，当此节点从树中调用时，行为树会直接调用此方法。然后调用`on_tick()`以及动作客户端目标。

In the body of the constructor, we get the input port `getInput` of the parameter `wait_duration` which can be configured independently for every instance of the `wait` node in the tree. It is set in the `duration` parameter and inserted into the `goal_`. The `goal_` class variable is the goal that the ROS 2 action client will send to the action server. So in this example, we set the duration to the time we want to wait by so that the action server knows the specifics of our request.

> 在构造函数的主体中，我们获取参数`wait_duration`的输入端口`getInput`，它可以为树中的每个`wait`节点单独配置。它被设置在`duration`参数中，并插入`goal_`中。`goal_`类变量是 ROS 2 动作客户端将发送给动作服务器的目标。因此，在本例中，我们通过设置持续时间来设置我们要等待的时间，以便动作服务器知道我们请求的具体信息。

The `providedPorts()` method gives us the opportunity to define input or output ports. Ports can be thought of as parameters that the behavior tree node has access to from the behavior tree itself. For our example, there is only a single input port, the `wait_duration` which can be set in the BT XML for each instance of the `wait` recovery. We set the type, `int`, the default `1`, the name `wait_duration`, and a description of the port `Wait time`.

> `providedPorts()`方法给了我们定义输入或输出端口的机会。端口可以被认为是行为树节点可以从行为树本身访问的参数。对于我们的示例，只有一个输入端口，`wait_duration`，可以在 BT XML 中为每个`wait`恢复实例设置。我们设置类型`int`，默认`1`，名称`wait_duration`，以及端口`等待时间`的描述。

```c++
static BT::PortsList providedPorts()
{
  return providedBasicPorts(
    {
      BT::InputPort<int>("wait_duration", 1, "Wait time")
    });
}
```

The `on_tick()` method is called when the behavior tree ticks a specific node. For the wait BT node, we simply want to notify a counter on the blackboard that an action plugin that corresponds to a recovery was ticked. This is useful to keep metrics about the number of recoveries executed during a specific navigation run. You could also log or update the `goal_` waiting duration if that is a variable input.

> `on_tick()`方法在行为树滴答指定节点时被调用。对于 Wait BT 节点，我们只想通知黑板上对应于恢复的动作插件的计数器。这有助于在特定导航运行期间跟踪恢复次数的度量。您也可以记录或更新`goal_`等待持续时间，如果它是一个可变输入。

```c++
void WaitAction::on_tick()
{
  increment_recovery_count();
}
```

The remaining methods are not used and not mandatory to override them. Only some BT node plugins will require overriding `on_wait_for_result()` to check for preemption or check a timeout. The success, aborted, and cancelled methods will default to `SUCCESS`, `FAILURE`, `SUCCESS` respectively if not overrided.

> 剩余的方法不被使用，也不是必须重写它们。只有一些 BT 节点插件需要重写`on_wait_for_result()`来检查抢占或检查超时。如果没有重写，成功、中止和取消的方法将默认为`SUCCESS`、`FAILURE`和`SUCCESS`。

## 2- Exporting the planner plugin

Now that we have created our custom BT node, we need to export our plugin so that it would be visible to the behavior tree when it loads a custom BT XML. Plugins are loaded at runtime and if they are not visible, then our BT Navigator server won\'t be able to load it or use it. In BehaviorTree.CPP, exporting and loading plugins is handled by the `BT_REGISTER_NODES` macro.

> 现在我们已经创建了自定义的 BT 节点，我们需要导出我们的插件，以便在加载自定义 BT XML 时可以看到它。插件在运行时加载，如果它们不可见，那么我们的 BT Navigator 服务器将无法加载它或使用它。在 BehaviorTree.CPP 中，导出和加载插件由`BT_REGISTER_NODES`宏处理。

```c++
BT_REGISTER_NODES(factory)
{
  BT::NodeBuilder builder =
    [](const std::string & name, const BT::NodeConfiguration & config)
    {
      return std::make_unique<nav2_behavior_tree::WaitAction>(name, "wait", config);
    };

  factory.registerBuilder<nav2_behavior_tree::WaitAction>("Wait", builder);
}
```

In this macro, we must create a `NodeBuilder` so that our custom action node can have a non-default constructor signature (for the action and xml names). This lambda will return a unique pointer to the behavior tree node we have created. Fill in the constructor with the relavent information, giving it the `name` and `config` given in the function arguments. Then define the ROS 2 action server\'s name that this BT node will call, in this case, its the [wWait]{.title-ref}\` action.

> 在这个宏中，我们必须创建一个`NodeBuilder`，以便我们的自定义动作节点可以具有非默认的构造函数签名(用于动作和 xml 名称)。该 lambda 将返回我们创建的行为树节点的唯一指针。使用函数参数给定的`name`和`config`填写构造函数。然后定义此 BT 节点将调用的 ROS 2 动作服务器的名称，在本例中，它是[wWait]{.title-ref}`动作。

We finally give the builder to a factory to register. `Wait` given to the factory is the name in the behavior tree XML file that corresponds to this BT node plugin. An example can be seen below, where the `Wait` BT XML node specifies a non-variable input port `wait_duration` of 5 seconds.

> 我们最终将构建器交给工厂来注册。`Wait`给出给工厂的是行为树 XML 文件中对应于此 BT 节点插件的名称。下面有一个例子，其中`Wait` BT XML 节点指定了 5 秒钟的非变量输入端口`wait_duration`。

```xml
<Wait wait_duration="5"/>
```

## 3- Add plugin library name to config

In order for the BT Navigator node to discover the plugin we\'ve just registered, we need to list the plugin library name under the bt_navigator node in the configuration YAML file. Configuration should look similar to the one shown below. Take note of nav2_wait_action_bt_node listed under plugin_lib_names.

> 为了让 BT Navigator 节点发现我们刚刚注册的插件，我们需要在配置 YAML 文件中的 bt_navigator 节点下列出插件库名称。配置应该类似下面所示。注意在 plugin_lib_names 下列出的 nav2_wait_action_bt_node。

```yaml
bt_navigator:
  ros__parameters:
    use_sim_time: True
    global_frame: map
    robot_base_frame: base_link
    odom_topic: /odom
    default_bt_xml_filename: "navigate_w_replanning_and_recovery.xml"
    plugin_lib_names:
    - nav2_back_up_action_bt_node # other plugin
    - nav2_wait_action_bt_node    # our new plugin
```

## 4- Run Your Custom plugin

Now you can use a behavior tree with your custom BT node. For example, the `navigate_w_replanning_and_recovery.xml` file is shown below.

> 现在您可以使用自定义 BT 节点使用行为树。例如，下面显示了`navigate_w_replanning_and_recovery.xml`文件。

Select this BT XML file in your specific navigation request in `NavigateToPose` or as the default behavior tree in the BT Navigator\'s configuration yaml file.

> 选择在 `NavigateToPose` 中特定导航请求中的这个 BT XML 文件，或作为 BT Navigator 配置 yaml 文件中的默认行为树。

```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <RecoveryNode number_of_retries="6" name="NavigateRecovery">
      <PipelineSequence name="NavigateWithReplanning">
        <RateController hz="1.0">
          <RecoveryNode number_of_retries="1" name="ComputePathToPose">
            <ComputePathToPose goal="{goal}" path="{path}" planner_id="GridBased"/>
            <ClearEntireCostmap name="ClearGlobalCostmap-Context" service_name="global_costmap/clear_entirely_global_costmap"/>
          </RecoveryNode>
        </RateController>
        <RecoveryNode number_of_retries="1" name="FollowPath">
          <FollowPath path="{path}" controller_id="FollowPath"/>
          <ClearEntireCostmap name="ClearLocalCostmap-Context" service_name="local_costmap/clear_entirely_local_costmap"/>
        </RecoveryNode>
      </PipelineSequence>
      <ReactiveFallback name="RecoveryFallback">
        <GoalUpdated/>
        <SequenceStar name="RecoveryActions">
          <ClearEntireCostmap name="ClearLocalCostmap-Subtree" service_name="local_costmap/clear_entirely_local_costmap"/>
          <ClearEntireCostmap name="ClearGlobalCostmap-Subtree" service_name="global_costmap/clear_entirely_global_costmap"/>
          <Spin spin_dist="1.57"/>
          <Wait wait_duration="5"/>
        </SequenceStar>
      </ReactiveFallback>
    </RecoveryNode>
  </BehaviorTree>
</root>
```
