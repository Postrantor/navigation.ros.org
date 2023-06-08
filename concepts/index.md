---
tip: translate by openai@2023-06-03 00:15:43
title: Navigation Concepts
---

This page is to help familiarize new robotists to the concepts of mobile robot navigation, in particular, with the concepts required to appreciating and working with this project.

> 这个页面旨在帮助新机器人学家熟悉移动机器人导航的概念，特别是理解和使用这个项目所需的概念。

# ROS 2

ROS 2 is the core middleware used for Nav2. If you are unfamilar with this, please visit [the ROS 2 documentation](https://docs.ros.org/en/rolling/) before continuing.

> ROS 2 是用于 Nav2 的核心中间件。如果您对此不熟悉，请在继续之前访问[ROS 2 文档]。

## Action Server

Just as in ROS, action servers are a common way to control long running tasks like navigation. This stack makes more extensive use of actions, and in some cases, without an easy topic interface. It is more important to understand action servers as a developer in ROS 2. Some simple CLI examples can be found in the [ROS 2 documentation](https://docs.ros.org/en/rolling/Tutorials/Understanding-ROS2-Actions.html).

> 正如在 ROS 中一样，**动作服务器是控制诸如导航之类的长时间运行任务的常用方式**。该堆栈在某些情况下更加广泛地使用了动作，而且没有简单的主题接口。作为 ROS 2 的开发人员，更重要的是要理解动作服务器。可以在[ROS 2 文档]中找到一些简单的 CLI 示例。

Action servers are similar to a canonical service server. A client will request some task to be completed, except, this task may take a long time. An example would be moving the shovel up from a bulldozer or ask a robot to travel 10 meters to the right.

> 行动服务器类似于规范服务器。客户端将请求完成某些任务，但是这个任务可能需要很长时间。一个例子是将铲斗从推土机上抬起，或者让机器人向右移动 10 米。

In this situation, action servers and clients allow us to call a long-running task in another process or thread and return a future to its result. It is permissible at this point to block until the action is complete, however, you may want to occasionally check if the action is complete and continue to process work in the client thread. Since it is long-running, action servers will also provide feedback to their clients. This feedback can be anything and is defined in the ROS `.action` along with the request and result types. In the bulldozer example, a request may be an angle, a feedback may be the angle remaining to be moved, and the result is a success or fail boolean with the end angle. In the navigation example, a request may be a position, a feedback may be the time its been navigating for and the distance to the goal, and the result a boolean for success.

> 在这种情况下，**动作服务器和客户端允许我们在另一个进程或线程中调用长时间运行的任务，并返回未来的结果**。在这一点上，可以阻塞，直到动作完成，但是你可能会偶尔检查动作是否完成，并继续在客户端线程中处理工作。由于它是长时间运行的，动作服务器也将向其客户端提供反馈。这种反馈可以是任何东西，并且在 ROS `.action`中与请求和结果类型一起定义。在推土机示例中，**请求可以是一个角度，反馈可以是剩余的角度，结果是一个成功或失败的布尔值，以及最终角度**。在导航示例中，请求可以是一个位置，反馈可以是已经导航的时间和到达目标的距离，结果是一个布尔值，表示是否成功。

Feedback and results can be gathered synchronously by registering callbacks with the action client. They may also be gathered by asychronously requesting information from the shared future objects. Both require spinning the client node to process callback groups.

> **回馈和结果可以通过使用动作客户端注册回调来同步获取**。它们也可以通过从共享未来对象异步请求信息来获取。两者都需要 spin 客户端节点以处理回调组。

Action servers are used in this stack to communicate with the highest level BT navigator through a `NavigateToPose` action message. They are also used for the BT navigator to communicate with the subsequent smaller action servers to compute plans, control efforts, and recoveries. Each will have their own unique `.action` type in `nav2_msgs` for interacting with the servers.

> 行动服务器用于通过`NavigateToPose`动作消息与最高级别的 BT 导航器进行通信。它们也用于 BT 导航器与后续较小的行动服务器之间进行通信，以计算计划、控制努力和恢复。每个服务器都将在`nav2_msgs`中拥有自己独特的`.action`类型来与服务器交互。

## Lifecycle Nodes and Bond

Lifecycle (or Managed, more correctly) nodes are unique to ROS 2. More information can be [found here](https://design.ros2.org/articles/node_lifecycle.html). They are nodes that contain state machine transitions for bringup and teardown of ROS 2 servers. This helps in determinstic behavior of ROS systems in startup and shutdown. It also helps users structure their programs in reasonable ways for commercial uses and debugging.

> **生命周期(或更准确地说，受管理的)节点是 ROS 2 独有的**。更多信息可以在[这里](https://design.ros2.org/articles/node_lifecycle.html)找到。它们是包含有 ROS 2 服务器启动和关闭的状态机转换的节点。这有助于 ROS 系统在启动和关闭时表现出确定性行为。它还有助于用户以合理的方式结构他们的程序，以便商业使用和调试。

> [!NOTE]
> lifecycle node == 受管理的节点

When a node is started, it is in the unconfigured state, only processing the node\'s constructor which should **not** contain any ROS networking setup or parameter reading. By the launch system, or the supplied lifecycle manager, the nodes need to be transitioned to inactive by configuring. After, it is possible to activate the node by transitioning through the activating stage.

> 当节点启动时，它处于未配置状态，只处理节点的构造函数，其中不应包含任何 ROS 网络设置或参数读取。通过启动系统或提供的生命周期管理器，需要通过配置将节点转换为非活动状态。之后，可以通过转换到激活阶段来激活节点。

This state will allow the node to process information and be fully setup to run. The configuration stage, triggering the `on_configure()` method, will setup all parameters, ROS networking interfaces, and for safety systems, all dynamically allocated memory. The activation stage, triggering the `on_activate()` method, will active the ROS networking interfaces and set any states in the program to start processing information.

> 这个状态将允许节点处理信息并完全设置以运行。配置阶段，触发`on_configure()`方法，将设置所有参数，ROS 网络接口，以及安全系统，所有动态分配的内存。激活阶段，触发`on_activate()`方法，将激活 ROS 网络接口并设置程序中的任何状态以开始处理信息。

To shutdown, we transition into deactivating, cleaning up, shutting down and end in the finalized state. The networking interfaces are deactivated and stop processing, deallocate memory, exit cleanly, in those stages, respectively.

> 关机，我们过渡到停用，清理，关闭并结束在最终状态。网络接口被停用并停止处理，释放内存，干净退出，在这些阶段，分别。

The lifecycle node framework is used extensively through out this project and all servers utilize it. It is best convention for all ROS systems to use lifecycle nodes if it is possible.

> 生命周期节点框架在整个项目中被广泛使用，所有服务器都使用它。**如果可能，对于所有 ROS 系统来说，使用生命周期节点是最佳惯例。**

Within Nav2, we use a wrapper of LifecycleNodes, `nav2_util LifecycleNode`. This wrapper wraps much of the complexities of LifecycleNodes for typical applications. It also includes a `bond` connection for the lifecycle manager to ensure that after a server transitions up, it also remains active. If a server crashes, it lets the lifecycle manager know and transition down the system to prevent a critical failure. See [`eloquent_migration`](..\migration\Eloquent.md) for details.

> 在 Nav2 中，我们使用 LifecycleNodes 的一个包装器`nav2_util LifecycleNode`。这个包装器**包装了大多数典型应用程序的复杂性**。它还包括一个 **`bond`连接，用于生命周期管理器**，以确保服务器转换后保持活动状态。如果服务器崩溃，它会让生命周期管理器(lifecycle manager)知道并将系统转换为防止关键失败。有关详细信息，请参阅`eloquent_migration`。

# Behavior Trees

Behavior trees (BT) are becoming increasingly common in complex robotics tasks. They are a tree structure of tasks to be completed. It creates a more scalable and human-understandable framework for defining multi-step or many state applications. This is opposed to a finite state machine (FSM) which may have dozens of states and hundreds of transitions. An example would be a soccer playing robot. Embedding the logic of soccer game play into a FSM would be challenging and error prone with many possible states and rules. Additionally, modeling choices like to shoot at the goal from the left, right, or center, is particularly unclear. With a BT, basic primitives like \"kick\" \"walk\" \"go to ball\" can be created and reused for many behaviors. More information can be found [in this book](https://arxiv.org/abs/1709.00084). I **strongly** recommend reading chapters 1-3 to get a good understanding of the nomenclature and workflow. It should only take about 30 minutes.

> **行为树(BT)正在复杂机器人任务中变得越来越常见**。它们是要完成的任务的树结构。它为**定义多步骤或多状态应用程序创建了一个更可扩展和人类可理解的框架**。这与有限状态机(FSM)形成了对比，它可能有数十个状态和数百个转换。一个例子是踢足球的机器人。将足球比赛的逻辑嵌入 FSM 中将具有挑战性，可能会有许多可能的状态和规则。此外，像从左边、右边或中间射门的选择特别不清楚。**使用 BT，可以创建基本原语，如“踢”、“走”、“去球”，可以用于许多行为**。更多信息可以在[这本书](.\Behavior_Trees_in_Robotics_and_AI_An_Introduction.pdf)中找到。我**强烈**建议阅读 1-3 章，以获得良好的术语和工作流程理解。只需要大约 30 分钟。

Behavior Trees provide a formal structure for navigation logic which can be both used to create complex systems but also be verifiable and validated as provenly correct using advanced tools. Having the application logic centralized in the behavior tree and with independent task servers (which only communicate data over the tree) allows for formal analysis.

> 行为树为导航逻辑提供了一种正式的结构，既可用于创建复杂的系统，也可以通过高级工具验证和验证。将应用程序逻辑集中在行为树中，并使用独立的任务服务器(仅通过树进行数据通信)，可以进行正式分析。

For this project, we use [BehaviorTree CPP V3](https://www.behaviortree.dev/) as the behavior tree library. We create node plugins which can be constructed into a tree, inside the `BT Navigator`. The node plugins are loaded into the BT and when the XML file of the tree is parsed, the registered names are associated. At this point, we can march through the behavior tree to navigate.

> 对于这个项目，我们使用[BehaviorTree CPP V3]作为行为树库。我们创建的节点插件可以构建成一棵树，在`BT Navigator`中。节点插件被加载到 BT 中，当树的 XML 文件被解析时，注册的名称将被关联。在这一点上，我们可以通过行为树来导航。

One reason this library is used is its ability to load subtrees. This means that the Nav2 behavior tree can be loaded into another higher-level BT to use this project as node plugin. An example would be in soccer play, using the Nav2 behavior tree as the \"go to ball\" node with a ball detection as part of a larger task. Additionally, we supply a `NavigateToPoseAction` plugin (among others) for BT so the Nav2 stack can be called from a client application through the usual action interface.

> 一个使用这个库的原因是**它能够加载子树**。这意味着 **Nav2 行为树可以加载到另一个更高级别的 BT 中**，以使用该项目作为节点插件。例如在足球比赛中，使用 Nav2 行为树作为“去球”节点，并将球检测作为更大任务的一部分。此外，我们还提供了`NavigateToPoseAction`插件(以及其他插件)，因此客户端应用程序可以通过通常的操作界面调用 Nav2 堆栈。

Other systems could be used to design complex autonomous behavior, namely Hierarchical FSMs (HFSM). Behavior Trees were selected due to popularity across the robotics and related industries and by largely user demand. However, due to the independent task server nature of Nav2, it is not difficult to offer a `nav2_hfsm_navigator` package in the future, pending interest and contribution.

> 其他系统可以用来设计复杂的自主行为，即**分层状态机(HFSM)**。由于机器人和相关行业的流行程度以及用户的大量需求，行为树被选择。然而，由于 Nav2 的独立任务服务器性质，在未来，只要有兴趣和贡献，就可以提供一个`nav2_hfsm_navigator`包。

# Navigation Servers

Planners and controllers are at the heart of a navigation task. Recoveries are used to get the robot out of a bad situation or attempt to deal with various forms of issues to make the system fault-tolerant. Smoothers can be used for additional quality improvements of the planned path. In this section, the general concepts around them and their uses in this project are analyzed.

> **规划者(Planners)和控制者(controllers)是导航任务的核心**。恢复用于使机器人摆脱糟糕的境况或尝试处理各种问题，以使系统具有容错性。平滑器可用于对计划路径进行额外的质量改进。在本节中，将分析它们的一般概念及其在本项目中的用途。

## Planner, Controller, Smoother and Recovery Servers

Four of the action servers in this project are the planner, behavior, smoother and controller servers.

> 四个行动服务器在本项目中是规划者、行为、平滑器和控制器服务器。

These action servers are used to host a map of algorithm plugins to complete various tasks. They also host the environmental representation used by the algorithm plugins to compute their outputs.

> 这些动作服务器用于托管一张算法插件的映射，以完成各种任务。它们还托管算法插件用于计算其输出的环境表示。

The planner, smoother and controller servers will be configured at runtime with the names (aliases) and types of algorithms to use. These types are the pluginlib names that have been registered and the names are the aliases for the task. An example would be the DWB controller used with name `FollowPath`, as it follows a reference path. In this case, then all parameters for DWB would be placed in that namespace, e.g. `FollowPath.<param>`.

> 在运行时，将使用名称(别名)和算法类型配置计划者、平滑器和控制器服务器。这些类型是已注册的 pluginlib 名称，而名称是任务的别名。例如 DWB 控制器使用名称`FollowPath`，因为它遵循参考路径。在这种情况下，所有 DWB 的参数都将放置在该名称空间中，例如`FollowPath.<param>`。

These three servers then expose an action interface corresponding to its task. When the behavior tree ticks the corresponding BT node, it will call the action server to process its task. The action server callback inside the server will call the chosen algorithm by its name (e.g. `FollowPath`) that maps to a specific algorithm. This allows a user to abstract the algorithm used in the behavior tree to classes of algorithms. For instance, you can have `N` plugin controllers to follow paths, dock with charger, avoid dynamic obstacles, or interface with a tool. Having all of these plugins in the same server allows the user to make use of a single environmental representation object, which is costly to duplicate.

> 这三个服务器会暴露出一个与其任务对应的动作接口。当行为树触及相应的 BT 节点时，它会调用动作服务器来处理其任务。服务器内部的动作服务器回调会按照它的名字(例如`FollowPath`)调用所选择的算法，这使用户能够将行为树中使用的算法抽象成算法类。例如，您可以有`N`个插件控制器来跟踪路径、与充电器对接、避免动态障碍物或与工具进行交互。将所有这些插件放在同一个服务器中，可以让用户利用单个环境表示对象，这种复制成本是高昂的。

For the behavior server, each of the behaviors also contains their own name, however, each plugin will also expose its own special action server. This is done because of the wide variety of behavior actions that may be created cannot have a single simple interface to share. The behavior server also contains a costmap subscriber to the local costmap, receiving real-time updates from the controller server, to compute its tasks. We do this to avoid having multiple instances of the local costmap which are computationally expensive to duplicate.

> 对于行为伺服器，每个行为也包含它们自己的名字，但是每个插件也会暴露出它自己的特殊行为伺服器。这是因为可能创建的各种行为动作无法有单一简单的介面来共享。行为伺服器还包含一个对本地 costmap 的订阅者，接收来自控制伺服器的即时更新，以计算其任务。我们这样做是为了避免本地 costmap 的多个实例，这些实例在复制时计算上是昂贵的。

Alternatively, since the BT nodes are trivial plugins calling an action, new BT nodes can be created to call other action servers with other action types. It is advisable to use the provided servers if possible at all times. If, due to the plugin or action interfaces, a new server is needed, that can be sustained with the framework. The new server should use the new type and plugin interface, similar to the provided servers. A new BT node plugin will need to be created to call the new action server \-- however no forking or modification is required in the Nav2 repo itself by making extensive use of servers and plugins.

> 另外，由于 BT 节点是调用操作的简单插件，因此可以创建新的 BT 节点以调用其他操作服务器，使用其他操作类型。尽可能始终使用提供的服务器。如果由于插件或操作界面需要新的服务器，则可以使用该框架来维护。新服务器应使用新的类型和插件界面，类似于提供的服务器。需要创建新的 BT 节点插件以调用新的操作服务器--但是，通过充分利用服务器和插件，不需要在 Nav2 存储库本身中进行分叉或修改。

If you find that you require a new interface to the pluginlib definition or action type, please file a ticket and see if we can rectify that in the same interfaces.

> 如果您发现需要一个新的接口来定义插件库或操作类型，请提交一个工单，看看我们是否可以在同一接口中纠正这一点。

## Planners

The task of a planner is to compute a path to complete some objective function. The path can also be known as a route, depending on the nomenclature and algorithm selected. Two canonical examples are computing a plan to a goal (e.g. from current position to a goal) or complete coverage (e.g. plan to cover all free space). The planner will have access to a global environmental representation and sensor data buffered into it. Planners can be written to:

> 计划者的任务是计算完成某个目标函数的路径。根据所选择的术语和算法，路径也可以称为路线。两个典型的例子是计算到目标(例如从当前位置到目标)的计划或完全覆盖(例如计划覆盖所有空闲空间)。计划者将访问全局环境表示和缓冲到其中的传感器数据。可以编写计划程序：

- Compute shortest path
- Compute complete coverage path
- Compute paths along sparse or predefined routes

The general task in Nav2 for the planner is to compute a valid, and potentially optimal, path from the current pose to a goal pose. However, many classes of plans and routes exist which are supported.

> 通用的 Nav2 规划器任务是从当前位置计算一条有效且可能是最优的路径到目标位置。但是，支持许多类型的计划和路线。

## Controllers

Controllers, also known as local planners in ROS 1, are the way we follow the globally computed path or complete a local task. The controller will have access to a local environment representation to attempt to compute feasible control efforts for the base to follow. Many controller will project the robot forward in space and compute a locally feasible path at each update iteration. Controllers can be written to:

> 控制器，也称为 ROS 1 中的本地规划器，是我们跟随全局计算路径或完成本地任务的方式。控制器将访问本地环境表示，以尝试计算基础跟随的可行控制努力。许多控制器将机器人投射到空间中，并在每次更新迭代中计算局部可行路径。控制器可以编写用于：

- Follow a path
- Dock with a charging station using detectors in the odometric frame
- Board an elevator
- Interface with a tool

The general task in Nav2 for a controller is to compute a valid control effort to follow the global plan. However, many classes of controllers and local planners exist. It is the goal of this project that all controller algorithms can be plugins in this server for common research and industrial tasks.

> 通用的 Nav2 控制器任务是计算有效的控制努力来遵循全局计划。然而，存在许多类别的控制器和局部规划器。这个项目的目标是所有控制器算法都可以插件到这个服务器中，用于共同的研究和工业任务。

## Behaviors

Recovery behaviors are a mainstay of fault-tolerant systems. The goal of recoveries are to deal with unknown or failure conditions of the system and autonomously handle them. Examples may include faults in the perception system resulting in the environmental representation being full of fake obstacles. The clear costmap recovery would then be triggered to allow the robot to move.

> **恢复行为是容错系统的主要组成部分**。恢复的目标是处理系统的**未知或故障状态，并自主处理它们**。例如，感知系统中的故障可能导致环境表示中充满了**假障碍。清晰的成本图恢复将被触发，以允许机器人移动**。

Another example would be if the robot was stuck due to dynamic obstacles or poor control. Backing up or spinning in place, if permissible, allow the robot to move from a poor location into free space it may navigate successfully.

> 如果机器人由于动态障碍物或控制不佳而卡住，另一个例子就是。如果允许，后退或原地旋转可以让机器人从不好的位置移动到空闲空间，它可能会成功导航。

Finally, in the case of a total failure, a recovery may be implemented to call an operators attention for help. This can be done with email, SMS, Slack, Matrix, etc.

> 最后，如果发生完全失败，可以执行恢复操作以引起运营商的注意，以寻求帮助。这可以通过电子邮件，短信，Slack，Matrix 等完成。

It is important to note that the behavior server can hold any behavior to share access to expensive resources like costmaps or TF buffers, not just recovery behaviors. Each may have their own API.

> 重要的是要注意，行为服务器可以保存任何行为以共享访问昂贵的资源，如成本地图或 TF 缓冲区，而不仅仅是恢复行为。每个可能有自己的 API。

## Smoothers

As criteria for optimality of the path searched by a planner are usually reduced compared to reality, additional path refinement is often beneficial. Smoothers have been introduced for this purpose, typically responsible for reducing path raggedness and smoothing abrupt rotations, but also for increasing distance from obstacles and high-cost areas as the smoothers have access to a global environmental representation.

> 作为规划器搜索路径的最优性标准通常比现实要少，因此通常有必要进行**额外的路径细化**。为此，引入了平滑器，其主要负责减少路径的锯齿和平滑突变旋转，还可以通过平滑器访问全局环境表示，从而增加与障碍物和高成本区域的距离。

Use of a separate smoother over one that is included as a part of a planner is advantageous when combining different planners with different smoothers or when a specific control over smoothing is required, e.g. smoothing ony a specific part of the path.

> 使用单独的平滑器而不是作为规划器的一部分包含的平滑器有利于将不同的规划器与不同的平滑器结合，或者当需要特定的控制平滑时，例如仅对路径的特定部分进行平滑。

The general task in Nav2 for a smoother is to receive a path and return its improved version. However, different input paths, criteria of the improvements and methods of acquiring them exist, creating space for multitude of smoothers that can be registered in this server.

> 通过 Nav2 进行平滑处理的一般任务是接收路径并返回其改进版本。但是，存在不同的输入路径、改进标准和获取它们的方法，为可以在此服务器注册的多种平滑器创造了空间。

## Waypoint Following

Waypoint following is a basic feature of a navigation system. It tells our system how to use navigation to get to multiple destinations.

> 路径跟踪是导航系统的基本功能。它告诉我们的系统如何使用导航抵达多个目的地。

The `nav2_waypoint_follower` contains a waypoint following program with a plugin interface for specific task executors. This is useful if you need to go to a given location and complete a specific task like take a picture, pick up a box, or wait for user input. It is a nice demo application for how to use Nav2 in a sample application.

> `nav2_waypoint_follower` 包含一个具有特定任务执行器插件接口的航点跟踪程序。如果您需要前往指定位置并完成特定任务，例如拍照、拾取箱子或等待用户输入，这将非常有用。它是一个很好的演示程序，用于演示如何在示例应用程序中使用 Nav2。

However, it could be used for more than just a sample application. There are 2 schools of thoughts for fleet managers / dispatchers. - Dumb robot; smart centralized dispatcher - Smart robot; dumb centralized dispatcher

> 然而，它可以用于不仅仅是一个示例应用程序。车队经理/调度员有两种学派。- 愚蠢的机器人; 智能集中调度员 - 智能机器人; 愚蠢的中央调度员

In the first, the `nav2_waypoint_follower` is fully sufficient to create a production-grade on-robot solution. Since the autonomy system / dispatcher is taking into account things like the robot\'s pose, battery level, current task, and more when assigning tasks, the application on the robot just needs to worry about the task at hand and not the other complexities of the system complete the requested task. In this situation, you should think of a request to the waypoint follower as 1 unit of work (e.g. 1 pick in a warehouse, 1 security patrole loop, 1 aisle, etc) to do a task and then return to the dispatcher for the next task or request to recharge. In this school of thought, the waypoint following application is just one step above navigation and below the system autonomy application.

> 首先，**`nav2_waypoint_follower`完全足以创建一个生产级的机器人解决方案**。由于自动化系统/调度程序在分配任务时会考虑机器人的姿态、电池电量、当前任务等因素，因此机器人上的应用程序只需要关注手头的任务，而不必考虑系统的其他复杂性来完成所请求的任务。在这种情况下，你应该将对航点跟踪程序的请求视为 1 个单位的工作(例如仓库中的 1 次拣货、1 次安全巡逻循环、1 条货架等)，然后返回调度程序以获取下一个任务或充电请求。按照这种思路，航点跟踪应用程序只是导航之上、系统自主应用程序之下的一个步骤。

In the second, the `nav2_waypoint_follower` is a nice sample application / proof of concept, but you really need your waypoint following / autonomy system on the robot to carry more weight in making a robust solution. In this case, you should use the `nav2_behavior_tree` package to create a custom application-level behavior tree using navigation to complete the task. This can include subtrees like checking for the charge status mid-task for returning to dock or handling more than 1 unit of work in a more complex task. Soon, there will be a `nav2_bt_waypoint_follower` (name subject to adjustment) that will allow you to create this application more easily. In this school of thought, the waypoint following application is more closely tied to the system autonomy, or in many cases, is the system autonomy.

> 在第二个，`nav2_waypoint_follower`是一个不错的示例应用程序/概念验证，但是你真的需要你的航点跟随/自主系统在机器人上承担更多的责任来制作一个健壮的解决方案。在这种情况下，你**应该使用`nav2_behavior_tree`包来创建一个自定义的应用程序级别的行为树来完成任务**。这可以包括检查中途充电状态以返回充电桩或处理更多的工作单元在一个更复杂的任务中的子树。很快，将有一个`nav2_bt_waypoint_follower`(名称可以调整)，它将允许你更容易地创建这个应用程序。在这种思想中，航点跟随应用程序更紧密地与系统自主性相关，或者在许多情况下，就是系统自主性。

Neither is better than the other, it highly depends on the tasks your robot(s) are completing, in what type of environment, and with what cloud resources available. Often this distinction is very clear for a given business case.

> 没有一个比另一个更好，这取决于您的机器人正在完成的任务，在什么样的环境中，以及可用的云资源。对于给定的业务案例，这种区别通常非常明显。

# State Estimation

Within the navigation project, there are 2 major transformations that need to be provided, according to community standards. The `map` to `odom` transform is provided by a positioning system (localization, mapping, SLAM) and `odom` to `base_link` by an odometry system.

> 在导航项目中，根据社区标准，需要提供两大变换。`map`到`odom`的变换由定位系统(定位、映射、SLAM)提供，`odom`到`base_link`的变换由里程计系统提供。

::: note
::: title
Note
:::

There is **no** requirement on using a LIDAR on your robot to use the navigation system. There is no requirement to use lidar-based collision avoidance, localization, or slam. However, we do provide instructions and support tried and true implementations of these things using lidars. You can be equally as successful using a vision or depth based positioning system and using other sensors for collision avoidance. The only requirement is that you follow the standards below with your choice of implementation.

> 没有要求在机器人上使用激光雷达来使用导航系统。也没有要求使用基于激光雷达的碰撞避免、定位或 SLAM。但是，我们提供了使用激光雷达实现这些事情的说明和支持。您也可以使用视觉或深度定位系统并使用其他传感器进行碰撞避免，同样取得成功。唯一的要求是您根据以下标准选择实施方案。

:::

## Standards

[REP 105](https://www.ros.org/reps/rep-0105.html) defines the frames and conventions required for navigation and the larger ROS ecosystem. These conventions should be followed at all times to make use of the rich positioning, odometry, and slam projects available in the community.

> `REP 105`定义了用于导航和更大的 ROS 生态系统所需的框架和约定。为了利用社区中丰富的定位、odometry 和 slam 项目，应始终遵守这些约定。

In a nutshell, REP-105 says that you must, at minimum, build a TF tree that contains a full `map` -\> `odom` -\> `base_link` -\> `[sensor frames]` for your robot. TF2 are the time-variant transformation library in ROS 2 we use to represent and obtain time synchronized transformations. It is the job of the global positioning system (GPS, SLAM, Motion Capture) to, at minimum, provide the `map` -\> `odom` transformation. It is then the role of the odometry system to provide the `odom` -\> `base_link` transformation. The remainder of the transformations relative to `base_link` should be static and defined in your [URDF](http://wiki.ros.org/urdf).

> REP-105 要求你至少需要构建一个包含完整 `map` -\> `odom` -\> `base_link` -\> `[sensor frames]` 的 TF 树来描述你的机器人。**TF2 是 ROS 2 中用来表示和获取时间同步变换的时变变换库**。它的工作是至少提供 `map` -\> `odom` 变换，由全局定位系统(GPS、SLAM、Motion Capture)完成。然后，接下来的工作是由里程计系统提供 `odom` -\> `base_link` 变换。关于 `base_link` 的其余变换应该是静态的，并在你的 [URDF](http://wiki.ros.org/urdf) 中定义。

## Global Positioning: Localization and SLAM

It is the job of the global positioning system (GPS, SLAM, Motion Capture) to, at minimum, provide the `map` -\> `odom` transformation. We provide `amcl` which is an Adaptive Monte-Carlo Localization technique based on a particle filter for localization of a static map. We also provide SLAM Toolbox as the default SLAM algorithm for use to position and generate a static map.

> GPS、SLAM 和 Motion Capture 的任务至少是提供`map` -\> `odom`转换。我们提供`amcl`，这是一种基于粒子滤波器的自适应蒙特卡罗定位技术，用于定位静态地图。我们还提供 SLAM Toolbox 作为默认的 SLAM 算法，用于定位和生成静态地图。

These methods may also produce other output including position topics, maps, or other metadata, but they must provide that transformation to be valid. Multiple positioning methods can be fused together using robot localization, discussed more below.

> 这些方法也可能产生其他输出，包括位置主题、地图或其他元数据，但它们必须提供转换才是有效的。可以使用机器人定位融合多个定位方法，在下面将进一步讨论。`

## Odometry

It is the role of the odometry system to provide the `odom` -\> `base_link` transformation. Odometry can come from many sources including LIDAR, RADAR, wheel encoders, VIO, and IMUs. The goal of the odometry is to provide a smooth and continuous local frame based on robot motion. The global positioning system will update the transformation relative to the global frame to account for the odometric drift.

> 观测量系统的角色是提供`odom` -\> `base_link`变换。观测量可以来自许多来源，包括 LIDAR，RADAR，轮编码器，VIO 和 IMU。观测量的目标是提供基于机器人运动的平滑和连续的局部坐标系。全局定位系统将更新相对于全局坐标系的变换以解决观测量漂移。

[Robot Localization](https://github.com/cra-ros-pkg/robot_localization/) is typically used for this fusion. It will take in `N` sensors of various types and provide a continuous and smooth odometry to TF and to a topic. A typical mobile robotics setup may have odometry from wheel encoders, IMUs, and vision fused in this manor.

> [Robot Localization]通常用于此融合。它将接受`N`个不同类型的传感器，并向 TF 和主题提供持续平滑的 odometry。典型的移动机器人设置可能会以这种方式融合来自轮编码器、IMU 和视觉的 odometry。

The smooth output can be used then for dead-reckoning for precise motion and updating the position of the robot accurately between global position updates.

> 输出的平滑度可以用于死记算，以精确的运动和准确更新机器人的位置，在全局位置更新之间。

# Environmental Representation

The environmental representation is the way the robot perceives its environment. It also acts as the central localization for various algorithms and data sources to combine their information into a single space. This space is then used by the controllers, planners, and recoveries to compute their tasks safely and efficiently.

> 机器人的环境表示是机器人感知其环境的方式。它还充当各种算法和数据源的中心定位，将它们的信息结合到一个单一的空间中。然后，控制器、规划者和恢复程序使用这个空间来安全有效地计算任务。

## Costmaps and Layers

The current environmental representation is a costmap. A costmap is a regular 2D grid of cells containing a cost from unknown, free, occupied, or inflated cost. This costmap is then searched to compute a global plan or sampled to compute local control efforts.

> **当前的环境表示是一个 costmap**。costmap 是一个普通的 2D 网格，其中的单元格**包含未知、免费、占用或膨胀的成本**。然后搜索这个 costmap 来计算全局计划或采样以计算局部控制努力。

Various costmap layers are implemented as pluginlib plugins to buffer information into the costmap. This includes information from LIDAR, RADAR, sonar, depth, images, etc. It may be wise to process sensor data before inputting it into the costmap layer, but that is up to the developer.

> 各种 costmap 层通过 pluginlib 插件实现，以将信息缓冲到 costmap 中。这包括来自 LIDAR，RADAR，声纳，深度，图像等的信息。在将传感器数据输入到 costmap 层之前，可能有必要对其进行处理，但这取决于开发者。

Costmap layers can be created to detect and track obstacles in the scene for collision avoidance using camera or depth sensors. Additionally, layers can be created to algorithmically change the underlying costmap based on some rule or heuristic. Finally, they may be used to buffer live data into the 2D or 3D world for binary obstacle marking.

> 成本图层可以创建以便使用摄像头或深度传感器检测和跟踪场景中的障碍物以实现避免碰撞。此外，还可以创建图层以根据某些规则或启发式算法对基础成本图进行算法更改。最后，它们可用于将实时数据缓冲到 2D 或 3D 世界中以进行二进制障碍标记。

## Costmap Filters

Imagine, you\'re annotating a map file (or any image file) in order to have a specific action occur based on the location in the annotated map. Examples of marking/annotating might be keep out zones to avoid planning inside, or have pixels belong to maximum speeds in marked areas. This annotated map is called \"filter mask\". Just like a mask overlaid on a surface, it can or cannot be same size, pose and scale as a main map. The main goal of filter mask - is to provide an ability of marking areas on maps with some additional features or behavioral changes.

> 想象一下，您正在注释一个地图文件(或任何图像文件)，以便在注释的地图上的特定位置上发生特定动作。标记/注释的示例可能是避免规划内部的禁止区域，或者让像素属于标记区域的最高速度。这个注释的地图称为“过滤掩码”。就像覆盖在表面上的面具一样，它可以或不可以与主地图的大小、姿态和比例相同。过滤掩码的主要目标是提供在地图上标记具有一些附加功能或行为更改的区域的能力。

Costmap filters - is costmap layer based approach of applying spatial-dependent behavioral changes annotated in filter masks, into Nav2 stack. Costmap filters are implemented as costmap plugins. These plugins are called \"filters\" as they are filtering a costmap by spatial annotations marked on filter masks. In order to make a filtered costmap and change robot\'s behavior in annotated areas, filter plugin reads the data came from filter mask. This data is being linearly transformed into feature map in a filter space. Having this transformed feature map along with a map/costmap, any sensors data and current robot coordinates filters can update underlying costmap and change behavior of the robot depending on where it is. For example, the following functionality could be made by using of costmap filters:

> Costmap 过滤器 - 是一种基于 costmap 层的方法，将空间相关的行为变化注释在过滤掩码中，应用于 Nav2 堆栈中。Costmap 过滤器被实现为 costmap 插件。这些插件被称为“过滤器”，因为它们通过过滤掩码上标记的空间注释来过滤 costmap。为了制作过滤后的 costmap 并更改机器人的行为，过滤器插件从过滤掩码中读取数据。该数据被线性变换为过滤器空间中的特征图。拥有这个转换后的特征图以及地图/ costmap，任何传感器数据和当前机器人坐标过滤器可以更新底层 costmap 并根据机器人所在位置更改其行为。例如，可以通过使用 costmap 过滤器实现以下功能：

- Keep-out/safety zones where robots will never enter.
- Speed restriction areas. Maximum speed of robots going inside those areas will be limited.
- Preferred lanes for robots moving in industrial environments and warehouses.

## Other Forms

Various other forms of environmental representations exist. These include:

> 各种其他形式的环境表示也存在。其中包括：

- gradient maps, which are similar to costmaps but represent surface gradients to check traversibility over
- 3D costmaps, which represent the space in 3D, but then also requires 3D planning and collision checking
- Mesh maps, which are similar to gradient maps but with surface meshes at many angles
- \"Vector space\", taking in sensor information and using machine learning to detect individual items and locations to track rather than buffering discrete points.

> - 梯度图，这类似于成本图，但表示表面梯度以检查可穿越性
> - 3D 成本地图，代表空间的 3D，但也需要 3D 规划和碰撞检查
> - 网格地图，类似于梯度图，但具有许多角度的表面网格
> - `向量空间`，接收传感器信息，使用机器学习来检测单个项目和位置而不是缓冲离散点。

# Nav2 Academic Overview

```{=html}
<h1 align="center">
  <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
    <iframe width="708" height="400" src="https://www.youtube.com/embed/QB7lOKp3ZDQ?autoplay=1&mute=1" frameborder="1" allowfullscreen></iframe>
  </div>
</h1>
```
