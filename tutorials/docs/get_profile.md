---
tip: translate by openai@2023-06-02 16:42:48
title: Profiling in ROS 2 / Nav2
---

- [Overview](#overview)
- [Preliminaries](#preliminaries)
- [Profile from a Node](#profile-from-a-node)
- [Profile from a Launch File](#profile-from-a-launch-file)
- [From Nav2 Bringup](#from-nav2-bringup)
- [Interpreting Results](#interpreting-results)

# Overview

This document explains one method for profiling applications in ROS 2 / Nav2. The aim of profiling is to generate files that can be analyzed to see where compute time and resources are spent during the execution of a program. This can be useful to determine where the bottlenecks in your program exist and where things might be able to be improved.

> 这份文档解释了一种在 ROS 2/Nav2 中为应用程序分析的方法。分析的目的是生成可以分析的文件，以查看在程序执行期间花费的计算时间和资源。这可以有助于确定程序中的瓶颈所在，以及可能进行改进的地方。

The following steps show ROS 2 users how to modify the Nav2 stack to get profiling information about a particular server / algorithm when they encounter a situation they\'d like to understand better. This tutorial applies to both simulated and physical robots.

> 以下步骤向 ROS 2 用户展示**如何修改 Nav2 堆栈**，以便在遇到想要更好理解的情况时获取关于特定服务器/算法的分析信息。本教程适用于模拟和物理机器人。

# Preliminaries

This tutorial makes use of two tools, callgrind from the `Valgrind` set of tools and `kcachegrind`. Valgrind is used to get the profiling information about the program and kcachegrind is the visualization engine used to interprete this information to do useful work.

> 这个教程使用了两个工具，一个是`Valgrind`工具集中的 callgrind，另一个是`kcachegrind`。Valgrind 用来获取程序的分析信息，而 kcachegrind 是用来解释这些信息并做有用工作的可视化引擎。

Thus, we must install them.

> 因此，我们必须安装它们。

    ```bash
    sudo apt install valgrind kcachegrind
    ```

More information can be found in the [Valgrind manual](https://valgrind.org/docs/manual/cl-manual.html) including additional valgrind arguments that can be used to specify more information.

> 更多信息可以在[Valgrind 手册](https://valgrind.org/docs/manual/cl-manual.html)中找到，包括可用于指定更多信息的其他 Valgrind 参数。

Generally speaking, to use valgrind we need to compile with debugging information. This can be done by passing `-g` as a compiling option or compile `CMAKE_BUILD_TYPE` as `Debug` or `RelWithDebInfo`. Then, we run our program using valgrind to capture the run-time statistics for later analysis. These are stored in `callgrind.out.XXX` files, where the suffix is the PID of the process. kcachegrind is used to visualize and analyze the results of the program execution.

> 一般来说，要使用 valgrind，我们需要使用调试信息编译。这可以通过传递`-g`作为编译选项或编译`CMAKE_BUILD_TYPE`为`Debug`或`RelWithDebInfo`来完成。然后，我们使用 valgrind 运行我们的程序，以**捕获运行时统计信息**，以供以后分析。这些信息存储在`callgrind.out.XXX`文件中，其后缀为进程的 PID。**kcachegrind 用于可视化和分析程序执行的结果**。

    ```cmake
    # CMakeLists.txt
    add_compile_options(-g)
    ```

    ```bash
    cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo
    ```

    ```bash
    valgrind --tool=callgrind [your-program] [program options]
    kcachegrind callgrind.out.XXX
    ```

# Profile from a Node

As in our generic example, for a given node, we need to compile with debug flags to capture the information for profiling with Valgrind. This can be done easily from the commandline. Note that we use `--packages-select` to only compile with this flag for the packages we want to profile nodes within.

> 对于给定的节点，与我们的通用示例一样，我们需要使用调试标志进行编译以便使用 Valgrind 进行分析。这可以从命令行轻松完成。请注意，我们使用`--packages-select`仅为我们要分析的节点中的软件包编译此标志。

    ```bash
    colcon build --packages-select <packages of interest> --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
    ```

Optionally, you may add the following line to the `CMakeLists.txt` of the package you\'re looking to profile. This may be preferable when you have a workspace with many packages but would like to only compile a subset with debug information using a single `colcon build` invokation.

> 如果需要，你可以在你要分析的包的 `CMakeLists.txt` 中添加下面这行。当你有一个有很多包的工作空间，但只想使用一个 `colcon build` 调用来编译一些调试信息时，这可能更好。

It is important that this should be added to both the host server and plugin packages(s) if you would like the results of a plugin\'s run-time profile.

> 这应该添加到主服务器和插件包中，如果你想要插件运行时的结果。

    ```cmake
    add_compile_options(-pg)
    ```

After either compiling method, this node should be run in its own terminal to isolate it from the rest of your system. Thus, this should not be composed in the same process as other nodes. To run a ROS 2 node with valgrind, this can be done from the commandline via:

> 编译完成后，应该在自己的终端中运行此节点，以将其与系统的其余部分隔离开来。 因此，此过程不应与其他节点组合在一起。 要使用 valgrind 运行 ROS 2 节点，可以通过以下命令行执行：

    ```bash
    ros2 run --prefix 'valgrind --tool=callgrind' <pkg> <node> --all-other-launch arguments
    ```

An example of this might be used for profiling the controller server with a particular controller plugin loaded. Both `nav2_controller` and the plugin package of interest are compiled with debug flags. In the example below, we are running a ROS 2 node with remapped topics and a path to its parameter file:

> 一个示例可以用来对具有特定控制插件加载的控制器服务器进行配置。`nav2_controller`和感兴趣的插件包都使用调试标志进行了编译。在下面的示例中，我们正在运行一个具有重映射主题和参数文件路径的 ROS 2 节点：`ros2 run nav2_controller remap __params:=/path/to/params.yaml`

    ```bash
    ros2 run --prefix 'valgrind --tool=callgrind' nav2_controller controller_server --ros-args -r __node:=controller_server -r cmd_vel:=cmd_vel_nav --params-file /path/to/nav2_bringup/params/nav2_params.yaml
    ```

Once sufficient data has been collected, cleanly exit the process with Control+C.

> 当收集到足够的数据时，使用 Control+C 清洁地退出进程。

# Profile from a Launch File

Just as in the Node example, we must also compile with debug flags when profiling a node from launch. We can complete the same valgrind call as from the commandline as within a launch file using launch prefixes.

> 就像在 Node 示例中一样，我们在从启动开始分析节点时也必须使用调试标志进行编译。我们可以在 launch 文件中使用 launch 前缀完成与命令行相同的 valgrind 调用。

As our example before, this is how we\'d launch the `controller_server` node from inside a launch file.

> 如我们之前的示例，这就是我们如何从启动文件内部启动`controller_server`节点的方式。

    ```python
    start_controller_server_node = Node(
        parameters=[
          get_package_share_directory("nav2_bringup") + '/params/nav2_params.yaml',
          {'use_sim_time': use_sim_time}
        ],
        package='nav2_controller',
        executable='controller_server',
        name='controller_server',
        prefix=['xterm -e valgrind --tools=callgrind'],
        output='screen')
    ```

Note that just like before, we should isolate this process from others. So this should not be run with any other nodes in this launch file nor use node composition when profiling a particular node.

Once sufficient data has been collected, cleanly exit the process with Control+C.

> 当收集到足够的数据后，使用 Control+C 清洁地退出进程。

# From Nav2 Bringup

Because Nav2 bringup has more than one node per launch file (and in the case `use_composition=true`, more than one per process), it is necessary to separate out a particular node that you\'re interested in profiling from the rest of the system. As previously described, once they\'re isolated in either a launch file or as a node to be launched on the commandline, they can easily be run to collect the callgrind information.

> 因为 Nav2 启动有每个启动文件（和`use_composition=true`的情况下，每个进程）有多个节点，因此有必要从其余系统中分离出你感兴趣的特定节点。正如前面描述的，一旦它们被隔离在启动文件中或作为一个节点在命令行上启动，就可以轻松地运行它们来收集 callgrind 信息。

The steps within Nav2 are as follows:

> 以下是 Nav2 中的步骤：

- Remove server node from the `navigation_launch.py`, ensuring to remove from both composed and non-composed options within the file

> 从`navigation_launch.py`中移除伺服器节点，确保从该档案中的组合和非组合选项中移除。

- In a separate launch file or using `ros2 run` CLI, start up the node you\'d like to profile using the instructions above

> 使用`ros2 run` CLI 或者在一个单独的启动文件中，根据上面的说明启动你想要分析的节点。

- Launch Nav2 as usual with the missing node

- Once your data has been collected, control+C and cleanly finish the profiled process and the rest of the navigation

> 等你收集完数据后，按 Ctrl+C，干净地结束配置的过程和其他导航。

It is important that the profiler node is launched before Nav2 so that it can take the signals from the lifecycle manager to transition up.

> 重要的是，在启动 Nav2 之前，要先启动 Profiler 节点，以便它能够接收来自生命周期管理器的信号以进行转换。

# Interpreting Results

Once you have your `callgrind` results, regardless of if you did it through a node, launch file, Nav2, or elsewhere, now we can analyze the results from the profiler to identify bottlenecks or potential areas of improvement. Using `kcachegrind`:

> 一旦您有了`callgrind`的结果，无论是通过节点、启动文件、Nav2 还是其他地方，我们现在可以分析分析器的结果以识别瓶颈或潜在的改进领域。使用`kcachegrind`：

    ```bash
    kcachegrind callgrind.out.XXX
    ```

This should open a window looking like below. The left side shows all of the calls and their relative percentages of compute time they and their children functions utilized.

> 这应该会打开一个类似下面的窗口。左侧显示了所有调用及其相对百分比的计算时间，以及它们的子函数使用的计算时间。

> ![image](images/kcachegrind.png){.align-center width="600px" height="450px"}

If you select the top level entry on the left sidebar, then select \"Call Graph\" at the bottom of the right workspace, it should show you a call graph of where the compute time was spent as a graph of method calls. This can be exceptionally helpful to find the methods where the most time is spent.

> 如果您在左侧侧边栏中选择最高级别的条目，然后在右侧工作区的底部选择“调用图”，它应该会以方法调用的图形显示计算时间花费的情况。这可以让您找到花费最多时间的方法，非常有帮助。

> ![image](images/call_graph.png){.align-center width="600px" height="450px"}
