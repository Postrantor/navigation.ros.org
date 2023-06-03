---
tip: translate by baidu@2023-06-03 18:03:15
---

## title: 7. Reduce ROS 2 Nodes and Determinism

**Task description**

This project is admittedly abstract to explain to someone unfamiliar with the inner-details of ROS 2 and its layers. If you\'re interested in working on this, you will become one of the few that truly understand the inner workings of it and be a very marketable skill. We do not expect anyone applying for this project to have that knowledge beforehand and we will help you learn the necessary items.

> 无可否认，这个项目是抽象的，可以向不熟悉 ROS 2 及其层的内部细节的人解释。如果你对此感兴趣，**你将成为少数真正了解其内部运作的人之一，并成为一项非常有市场的技能**。我们不希望任何申请该项目的人事先掌握这些知识，我们将帮助您学习必要的项目。

ROS 2 architecturally was changed before Foxy in order to ensure that any single process containing multiple ROS 2 node objects will share the same DDS participant on the network. This is important due to the overhead that each additional DDS participant has on the system.

> 在 Foxy 之前，ROS 2 在架构上进行了更改，以确保包含多个 ROS 2 节点对象的任何单个进程在网络上共享相同的 DDS 参与者。这一点很重要，因为每个额外的 DDS 参与者在系统上都有开销。

In order for nav2 to leverage this the best, we need to adjust our usage of ROS 2 nodes and executors to further minimize the number of node objects in existance. In the early days of ROS 2 when Nav2 was being built, we were required to have many nodes in a single server in order to handle action requests and other callbacks. Now, we can make use of multi-threaded spinners, callback groups, and individual executors for specific tasks.

> 为了让 nav2 最好地利用这一点，**我们需要调整 ROS 2 节点和执行器的使用，以进一步减少现有节点对象的数量。在 ROS 2 的早期，当 Nav2 被构建时，我们被要求在一个服务器中有许多节点，以便处理操作请求和其他回调。现在，我们可以为特定任务使用多线程微调器、回调组和单个执行器**。

This project will involve identifying all of the Node objects in the stack (control+F makes this easy) and work with mentors to ensure by the end of the summer each server contains only a single node. Additionally, the behavior tree plugins should be updated to leverage callback groups to ensure that any single BT node spinning to check if any new messages are on its callback will **only** trigger its own callback by the same mechanisms.

> 该项目将涉及识别堆栈中的所有 Node 对象（控制 +F 使这变得容易），并与导师合作，以确保在夏末之前，每个服务器只包含一个节点。此外，应该更新行为树插件以利用回调组，以确保任何一个 BT 节点旋转以检查其回调中是否有任何新消息，都将**仅**通过相同的机制触发其自己的回调。

More details about this project can be supplied if interested, but the tickets linked below also provide more context. Trust me to say this is a very achievable goal over the course of the summer and will also let you look under the hood of both Nav2 and rclcpp, giving you valuable insight future in your career (and put you in the top 10% of ROS 2 developers that know it to this degree).

> 如果感兴趣，可以提供有关该项目的更多详细信息，但下面链接的票证也提供了更多上下文。相信我，这是整个夏天非常可以实现的目标，也会让你了解 Nav2 和 rclcpp 的情况，让你在职业生涯中有宝贵的见解（并使你在 ROS 2 开发人员中排名前 10%）。

**Project difficulty: Medium**

> **项目难度：中等**

**Project community mentor: Steve Macenski** [\@SteveMacenski](https://github.com/SteveMacenski)

> **项目社区导师：Steve Macenski**[\@Steve Macenski](https://github.com/SteveMacenski)

**Mentor contact details: \[See link above, link in GitHub profile description\]**

> **导师联系方式：\[请参阅上面的链接，GitHub 简介中的链接\]**

**Project output requirements** - Remove excess ROS 2 nodes from stack and replace with executors and multi-threaded executors - Replace Behavior Tree node spinners with local executors to ensure deterministic execution & processing only callback groups of the current BT node - If time allots, create a single `main()` function Nav2 executable composing all Nav2 processes and benchmarking improvements in performance

> **项目输出要求**-从堆栈中删除多余的 ROS 2 节点，并用执行器和多线程执行器替换-用本地执行器替换行为树节点微调器，以确保确定性执行和仅处理当前 BT 节点的回调组-如果时间分配，创建一个单独的“main（）”函数 Nav2 可执行文件，组成所有 Nav2 进程并确定性能改进的基准

**Skills required**

- C++, ROS

**List of relevant open source software repositories and refs**

- [ROS](https://www.ros.org/)
- [Gazebo Simulator](http://gazebosim.org/)
- [Github ticket](https://github.com/ros-planning/navigation2/issues/2251)
- [Github ticket2](https://github.com/ros-planning/navigation2/issues/816)
- [Navigation2](https://navigation.ros.org/)
- [Some related works](https://alyssapierson.files.wordpress.com/2018/05/pierson2018.pdf)

**Licensing** - All contributions will be under the Apache 2.0 license. - No other CLA\'s are required.

> **许可证\***-所有贡献都将在 Apache 2.0 许可证下。-不需要其他 CLA。
