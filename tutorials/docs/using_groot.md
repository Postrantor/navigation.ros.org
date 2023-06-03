---
tip: translate by openai@2023-06-02 15:40:29
title: Groot - Interacting with Behavior Trees
---

![](images/Groot/groot_start_menu.png){#groot_startup_menu .align-center}

- [Overview](#overview)
- [Visualize Behavior Trees](#visualize-behavior-trees)
- [Edit Behavior Trees](#edit-behavior-trees)
- [Adding A Custom Node](#adding-a-custom-node)

# Overview

```html{=html}
<h1 align="center">
  <div>
    <div style="position: relative; padding-bottom: 0%; overflow: hidden; max-width: 100%; height: auto;">
      <iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/Z6xCat0zaWU?autoplay=1&mute=1" frameborder="0" allowfullscreen></iframe>
    </div>
  </div>
</h1>
```

[Groot](https://github.com/BehaviorTree/Groot) is the companion application of the [BehaviorTree.CPP](https://github.com/BehaviorTree/BehaviorTree.CPP) library used to create, edit, and visualize behavior trees. Behavior Trees are deeply integrated into Nav2, used as the main method of orchestrating task server logic across a complex navigation and autonomy stack. Behavior Trees, in short BTs, consist of many nodes completing different tasks and control the flow of logic, similar to a Heirarchical or Finite State Machine, but organized in a tree structure. These nodes are of types: [Action]{.title-ref}, [Condition]{.title-ref}, [Control]{.title-ref}, or [Decorator]{.title-ref}, and are described in more detail in `concepts` and [BehaviorTree.CPP](https://github.com/BehaviorTree/BehaviorTree.CPP).

> **Groot 是[BehaviorTree.CPP]库的伴侣应用程序，用于创建、编辑和可视化行为树**。行为树深度集成到 Nav2 中，用作在复杂的导航和自主堆栈中协调任务服务器逻辑的主要方法。简而言之，**行为树由许多节点完成不同的任务并控制逻辑流，类似于分层或有限状态机，但组织在树结构中**。这些节点的类型为[Action]{.title-ref}、[Condition]{.title-ref}、[Control]{.title-ref}或[Decorator]{.title-ref}，并在`concepts`和[BehaviorTree.CPP](https://github.com/BehaviorTree/BehaviorTree.CPP)中有详细描述。

> [!NOTE]
> 行为树由许多节点完成不同的任务并控制逻辑流，类似于分层或有限状态机，但组织在树结构中

`writing_new_nbt_plugin` offers a well written example of creating a simple `Action` node if creating new BT nodes are of interest. This tutorial will focus solely on launching Groot, visualizing a Behavior Tree, and modifying that tree for a given customization, assuming a library of BT nodes. Luckily, Nav2 provides a robust number of BT nodes for your use out of the box, enumerated in `plugins`.

> `writical_new_nbt_plugin` {.interpreted-text role =“ ref”}提供了一个很好的书面示例，即创建一个简单的`action`节点''，如果创建新的 bt 节点很感兴趣。本教程将仅着眼于启动 Groot，可视化行为树，并假定为给定的自定义修改该树，并假定 BT 节点库。幸运的是，NAV2 为您的使用提供了强劲的 BT 节点，并在“插件” {.interpreted-text 角色=“ ref”}中列举。

A BT configuration file in BehaviorTree.CPP is an XML file. This is used to dynamically load the BT node plugins at run-time from the appropriate libraries mapped to their names. The XML format is defined [in detail here](https://www.behaviortree.dev/xml_format/). Therefore, Groot needs to have a list of nodes it has access to and important metadata about them like their type and ports (or parameters). We refer to this as the \"pallet\" of nodes later in the tutorial.

> 一个 BehaviorTree.CPP 中的 BT 配置文件是一个 XML 文件。这用于在运行时从相应的库中动态加载 BT 节点插件，并将它们映射到它们的名称。XML 格式在这里详细定义[详细信息](https://www.behaviortree.dev/xml_format/)。因此，Groot 需要有一个它可以访问的节点列表以及有关它们的重要元数据，如它们的类型和端口(或参数)。我们稍后在教程中将其称为 **“节点板”**。

In the video above you can see Groot side-by-side with RVIz and a test platform 100% equipped with ROS-enabled hardware from SIEMENS. Groot not only displays the current Behavior Tree while the robot is operating. Note: Before ROS 2 Humble, live Groot behavior tree monitoring during execution was supported in Nav2. This was removed due to buggy support in BT.CPP / Groot for changing behavior trees on the fly, see `galactic_migration` for more details.

> 在上面的视频中，您可以看到 Groot 与 RVIz 和一个完全配备有 SIEMENS ROS 启用硬件的测试平台并排显示。Groot 不仅在机器人操作时显示当前行为树。注意：在 ROS 2 Humble 之前，活动 Groot 行为树监视器在 Nav2 中得到支持。由于 BT.CPP / Groot 中的改变行为树的功能存在错误，因此已删除，有关详细信息，请参见`galactic_migration`。

# Visualize Behavior Trees

To display a Behavior Tree like that in `groot_nav2_default_bt`, we will first start the Groot executable. Out of the box, Groot can only display Behavior Trees and nodes that are from the defaults in BT.CPP, since it does not know anything about Nav2 or your other projects. Therefore, we must point Groot to our pallet, or index, of Nav2 / custom behavior tree nodes:

> 要像`groot_nav2_default_bt`中那样显示行为树，我们首先要启动 Groot 可执行文件。出厂时，Groot 只能显示 BT.CPP 中的默认行为树和节点，因为它对 Nav2 或其他项目一无所知。因此，我们必须将 Groot 指向我们的 Nav2/自定义行为树节点目录：`Wait`

1. Open Groot in editor mode. Now, Groot should look like in `groot_bt_editor`.
2. Select the [Load palette from file]{.title-ref} option either via the context menu or the import icon in the top middle of the menu bar.
3. Open the file [/path/to/navigation2/nav2_behavior_tree/nav2_tree_nodes.xml]{.title-ref} to import all the custom behavior tree nodes used for navigation. This is the pallet of Nav2 custom behavior tree nodes. Now, Groot should look like in `groot_bt_editor_with_nodes`.
4. Select [Load tree]{.title-ref} option near the top left corner
5. Browse the tree you want to visualize, then select [OK]{.title-ref}. The Nav2 BTs exist in [/path/to/navigation2/nav2_bt_navigator/behavior_trees/]{.title-ref}

> 打开 Groot 编辑模式。现在，Groot 应该看起来像`groot_bt_editor`一样。
> 选择[从文件加载调色板]{.title-ref}选项，可以通过上下文菜单或菜单栏顶部中央的导入图标。
> 打开文件[/path/to/navigation2/nav2_behavior_tree/nav2_tree_nodes.xml]{.title-ref}，以导入所有用于导航的自定义行为树节点。这是 Nav2 自定义行为树节点的调色板。现在，Groot 应该像`groot_bt_editor_with_nodes`一样。
> 选择左上角附近的[加载树]{.title-ref} 选项
> 浏览你想要可视化的树，然后选择[OK]{.title-ref}。Nav2 BTs 存在于[/path/to/navigation2/nav2_bt_navigator/behavior_trees/]{.title-ref}

---

![Default Editor View](images/Groot/groot_bt_editor.png){#groot_bt_editor}
![Editor with Custom Nodes loaded in blue](images/Groot/groot_with_nav2_custom_nodes.png){#groot_bt_editor_with_nodes}

---

If you select the default tree [navigate_w\_replanning_and_recovery.xml]{.title-ref}, then a Groot editor should look like `groot_nav2_default_bt`.

> 如果您选择默认树[navigate_w\_replanning_and_recovery.xml]{.title-ref}，那么 Groot 编辑器应该看起来像`groot_nav2_default_bt`。

---

![Full Nav2 Default BehaviorTree](images/Groot/bt_w_replanning_and_recovery.png){#groot_nav2_default_bt}

---

::: note
::: title
Note
:::

If a tree cannot be visualized because some nodes are missing in the pallet, you might need to add it to your pallet. While we try to keep Nav2\'s BT nodes and pallets in sync, if you notice one is missing, please file a ticket or pull request and we should have that updated quickly.

> 如果在棕榈树中缺少一些节点，无法可视化，您可能需要将其添加到您的棕榈树中。虽然我们尽量让 Nav2 的 BT 节点和棕榈树保持同步，但如果您发现缺少一些节点，请提交工单或拉取请求，我们应该可以迅速更新。`Wait`

:::

# Edit Behavior Trees

Now that you have a Nav2 BT open in Groot in editor mode, you should be able to trivially modify it using the GUI. Starting from a screen like that shown in `groot_nav2_default_bt`, you can pull in new nodes from the side panel to add them to the workspace. You may then connect the nodes using a \"drag and drop\" motion between the node\'s input and output ports to assemble the new nodes into the tree.

> 现在，您已经在 Groot 中以编辑模式打开了 Nav2 BT，您应该能够使用 GUI 轻松修改它。 从`groot_nav2_default_bt`中显示的屏幕开始，您可以从侧面板中拉入新节点，将它们添加到工作区中。 然后，您可以使用“拖放”动作在节点的输入和输出端口之间连接节点，以组装新节点成树。

If you select a given node, you can change metadata about it such as its name or values of parameterizable ports. When you\'re done modifying, simply save the new configuration file and use that on your robot the next time!

> 如果你选择了一个给定的节点，你可以更改其元数据，比如它的名字或可参数化端口的值。当你完成修改后，只需保存新的配置文件，下次在你的机器人上使用！

# Adding A Custom Node

Each node in the behavior tree holds a specialized function. Sometimes, its useful to create new nodes and add them to your pallet during the design process - perhaps before the implementations themselves exist. This helps designers abstract away the implementation specifics of the nodes from the higher level logic of the tree itself and how they\'d like to interact with a given node (e.g. type, ports, etc). Within Groot, you may create new custom nodes to add to your tree and export these new nodes back to your pallet. Implementing the node itself needs to be done separately from Groot, which is described in `writing_new_nbt_plugin`.

> 每个行为树节点都具有专门的功能。有时，在设计过程中创建新节点并将它们添加到您的调色板中是很有用的 - 也许在实现本身存在之前。这有助于设计师将节点的实现特定细节从树本身的更高级逻辑中抽象出来，以及他们如何与给定节点交互(例如类型，端口等)。在 Groot 中，您可以创建新的自定义节点来添加到树中，并将这些新节点导出回您的调色板。需要单独从 Groot 中实现节点，具体方法参见`writing_new_nbt_plugin`。

---

![Create a new Custom Node](images/Groot/groot_create_custom_node.png){#groot_create_custom_node}

---

Creating a new custom node can be started by clicking the orange marked icon in `groot_create_custom_node`, while Groot is in Editor mode. This should load a new window, similar to `groot_interactive_node_creation`. In this new window, it asks you to fill in the metadata about this new node, in order to create it. It will ask you for standard information such as name (green box), type of node (orange box), and any optional ports for parameterization or access to blackboard variables (blue box).

> 点击`groot_create_custom_node`中标有橙色图标，可以开始创建新的自定义节点。这将加载一个新窗口，类似于`groot_interactive_node_creation`。在这个新窗口中，它会要求你填写有关此新节点的元数据，以便创建它。它会要求您提供标准信息，如名称(绿色框)，节点类型(橙色框)以及任何可选端口用于参数化或访问黑板变量(蓝色框)。如果您遇到用'`'符号括起来的单词，请将输出保持不变，如`Wait`。

After completing, select [OK]{.title-ref} in `groot_interactive_node_creation`, the new custom node should appear in blue in the [TreeNode Palette]{.title-ref} as in `groot_export_new_node`.

> 完成后，在`groot_interactive_node_creation`中选择[OK]{.title-ref}，新的自定义节点应在[TreeNode Palette]{.title-ref}中以蓝色出现，如`groot_export_new_node`中所示。

---

![UI to describing new Nodes](images/Groot/groot_interactive_node_creation.png){#groot_interactive_node_creation} ![Exporting the new Custom Node](images/Groot/groot_export_new_node.png){#groot_export_new_node width="180.0%"}

---

Before starting to create a new BT based on the new custom nodes, it is recommend to export the newly created nodes to save in case of Groot crashing. This can be performed with the icon highlighted in green from `groot_export_new_node`. The resulting XML output from the node created in `groot_interactive_node_creation` can be seen below. You can see more examples in [Nav2\'s BT Node Pallet XML](https://github.com/ros-planning/navigation2/blob/main/nav2_behavior_tree/nav2_tree_nodes.xml).

> 在开始根据新的自定义节点创建新的 BT 之前，建议从 `groot_export_new_node` 中导出用绿色标记的图标来保存新创建的节点，以防 Groot 崩溃。以下是从 `groot_interactive_node_creation` 创建的节点的 XML 输出。您可以在[Nav2 的 BT 节点拼板 XML](https://github.com/ros-planning/navigation2/blob/main/nav2_behavior_tree/nav2_tree_nodes.xml)中查看更多示例。如果遇到用'`'符号括起来的单词，请保持输出，例如`Wait`。

```xml
<root>
  <TreeNodesModel>
    <Action ID="MyAwesomeNewNode">
      <input_port name="key_name" default="false">coffee</input_port>
      <output_port name="key_name2" default="42">Sense of life</output_port>
      <inout_port name="next_target" default="pancakes">rolling target</inout_port>
    </Action>
  </TreeNodesModel>
</root>
```
