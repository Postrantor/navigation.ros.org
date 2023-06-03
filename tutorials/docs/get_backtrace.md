---
tip: translate by openai@2023-06-02 16:17:47
title: Get Backtrace in ROS 2 / Nav2
---

- [Overview](#overview)
- [Preliminaries](#preliminaries)
- [From a Node](#from-a-node)
- [From a Launch File](#from-a-launch-file)
- [From Large Project](#from-large-project)
- [From Nav2 Bringup](#from-nav2-bringup)
- [Automatic backtrace on crash](#automatic-backtrace-on-crash)

# Overview

This document explains one set of methods for getting backtraces for ROS 2 and Nav2. There are many ways to accomplish this, but this is a good starting point for new C++ developers without GDB experience.

> 这份文件描述了一组用于获取 ROS 2 和 Nav2 回溯的方法。有许多方法可以实现此目的，但对于没有 GDB 经验的新 C++开发人员来说，这是一个很好的起点。

The following steps show ROS 2 users how to modify the Nav2 stack to get traces from specific servers when they encounter a problem. This tutorial applies to both simulated and physical robots.

> 以下步骤向 ROS 2 用户展示如何**修改 Nav2 堆栈**，以便在遇到问题时从特定服务器获取跟踪记录。此教程适用于模拟和物理机器人。

This will cover how to get a backtrace from a specific node using `ros2 run`, from a launch file representing a single node using `ros2 launch`, and from a more complex orchestration of nodes. By the end of this tutorial, you should be able to get a backtrace when you notice a server crashing in ROS 2.

> 在本教程中，我们将讨论如何使用`ros2 run`从特定节点获取回溯，从使用`ros2 launch`表示单个节点的启动文件，以及从更复杂的节点编排中获取回溯。到本教程结束时，您应该能够在 ROS 2 中发现服务器崩溃时获取回溯。

# Preliminaries

GDB is the most popular debugger for C++ on Unix systems. It can be used to determine the reason for a crash and track threads. It may also be used to add breakpoints in your code to check values in memory a particular points in your software.

> GDB 是 Unix 系统上最流行的 C++调试器。它可用于确定崩溃的原因并跟踪线程。它还可以用来在您的代码中添加断点，以检查内存中特定点的值在您的软件中。

Using GDB is a critical skill for all software developers working on C/C++. Many IDEs will have some kind of debugger or profiler built in, but with ROS, there are few IDEs to choose. Therefore it\'s important to understand how to use these raw tools you have available rather than relying on an IDE to provide them. Further, understanding these tools is a fundamental skill of C/C++ development and leaving it up to your IDE can be problematic if you change roles and no longer have access to it or are doing development on the fly through an ssh session to a remote asset.

> 使用 GDB 是 C/C++开发人员的关键技能。许多 IDE 都会集成一些调试器或分析器，但是在 ROS 中，可供选择的 IDE 很少。因此，了解如何使用这些原始工具而不是依赖 IDE 提供它们是很重要的。此外，了解这些工具是 C/C++开发的基本技能，如果你更换角色而不再使用它，或者通过 ssh 会话到远程资产快速开发，将它留给 IDE 可能会有问题。

Using GDB luckily is fairly simple after you have the basics under your belt. The first step is to add `-g` to your compiler flags for the ROS package you want to profile / debug. This flag builds debug symbols that GDB and valgrind can read to tell you specific lines of code in your project are failing and why. If you do not set this flag, you can still get backtraces but it will not provide line numbers for failures. Be sure to remove this flag after debugging, it will slow down performance at run-time.

> 使用 GDB 幸运的是，一旦掌握了基础知识，就相当简单了。第一步是为您想要分析/调试的 **ROS 包添加`-g`标志**。此标志构建了 GDB 和 valgrind 可以**读取的调试符号**，以告知您项目中哪些特定行代码失败了，以及失败的原因。如果您不设置此标志，您仍然可以获得回溯，但它不会提供故障的行号。**调试后，一定要删除此标志，否则会降低运行时的性能。**

Adding the following line to your `CMakeLists.txt` for your project should do the trick. If your project already has a `add_compile_options()`, you can simply add `-g` to it. Then simply rebuild your workspace with this package `colcon build --packages-select <package-name>`. It may take a little longer than usual to compile.

> 将以下行添加到您的`CMakeLists.txt`中，应该可以解决问题。如果您的项目已经有了`add_compile_options()`，可以简单地添加`-g`。然后用这个包`colcon build --packages-select <package-name>`重新构建您的工作区。编译可能比平常花费更长时间。

    ```cmake
    add_compile_options(-g)
    ```

Now you\'re ready to debug your code! If this was a non-ROS project, at this point you might do something like below. Here we\'re launching a GDB session and telling our program to immediately run. Once your program crashes, it will return a gdb session prompt denoted by `(gdb)`. At this prompt you can access the information you\'re interested in. However, since this is a ROS project with lots of node configurations and other things going on, this isn\'t a great option for beginners or those that don\'t like tons of commandline work and understanding the filesystem.

> 现在你准备好调试你的代码了！如果这是一个非 ROS 项目，在这一点上你可能会做一些像下面这样的事情。这里我们启动一个 GDB 会话，告诉我们的程序立即运行。一旦你的程序崩溃，它会返回一个由`(gdb)`表示的 gdb 会话提示符。在这个提示符下，你可以访问你感兴趣的信息。但是，由于这是一个 ROS 项目，有很多节点配置和其他正在进行的事情，这对于初学者或者不喜欢大量命令行工作和理解文件系统的人来说不是一个很好的选择。

    ```bash
    gdb ex run --args /path/to/exe/program
    ```

Below are sections to describe the 3 major situations you could run into with ROS 2-based systems. Read the section that best describes the problem you\'re attempting to solve.

> 以下是描述基于 ROS 2 的系统可能遇到的 3 个主要情况的部分。请阅读最能描述您正尝试解决的问题的部分。

# From a Node

Just as in our non-ROS example, we need to setup a GDB session before launching our ROS 2 node. While we could set this up through the commandline with some knowledge of the ROS 2 file system, we can instead use the launch `--prefix` option the kind folks at Open Robotics provided for us.

> 正如我们在非 ROS 示例中一样，我们需要在启动 ROS 2 节点之前设置 GDB 会话。虽然可以通过命令行，凭借对 ROS 2 文件系统的一些知识来设置，但我们可以使用 Open Robotics 提供给我们的`--prefix`选项来代替。

`--prefix` will execute some bits of code before our `ros2` command allowing us to insert some information. If you attempted to do `gdb ex run --args ros2 run <pkg> <node>` as analog to our example in the preliminaries, you\'d find that it couldn\'t find the `ros2` command. If you\'re even more clever, you\'d find that trying to source your workspace would also fail for similar reasons.

> `--prefix`将在我们的`ros2`命令之前执行一些代码位，从而允许我们插入一些信息。如果您尝试将`gdb ex run--args ros2 run<pkg><node>`作为与我们在预备部分中的示例类似的操作，您会发现它找不到`ros2`命令。如果你更聪明，你会发现，由于类似的原因，试图获取工作空间的资源也会失败。

Rather than having to revert to finding the install path of the executable and typing it all out, we can instead use `--prefix`. This allows us to use the same `ros2 run` syntax you\'re used to without having to worry about some of the GDB details.

> 而不是不得不查找可执行文件的安装路径并一一输入，我们可以**使用`--prefix`。这样，我们就可以使用你熟悉的`ros2 run`语法**，而不必担心 GDB 的一些细节。

    ```bash
    ros2 run --prefix 'gdb -ex run --args' <pkg> <node> --all-other-launch arguments
    ```

Just as before, this prefix will launch a GDB session and run the node you requested with all the additional commandline arguments. You should now have your node running and should be chugging along with some debug printing.

> 正如以前，这个前缀将启动一个 GDB 会话，并使用所有其他命令行参数运行您请求的节点。您现在应该有节点正在运行，并且应该正在调试打印。

Once your server crashes, you\'ll see a prompt like below. At this point you can now get a backtrace.

> 一旦您的服务器崩溃，您将看到如下提示。此时，您可以获得回溯。

    ```bash
    (gdb)
    ```

In this session, type `backtrace` and it will provide you with a backtrace. Copy this for your needs. For example:

> 在这个会话中，输入`backtrace`，它会给你一个回溯跟踪。复制它以满足你的需求。例如：

    ```bash
    (gdb) backtrace
    #0  __GI_raise (sig=sig@entry=6) at ../sysdeps/unix/sysv/linux/raise.c:50
    #1  0x00007ffff79cc859 in __GI_abort () at abort.c:79
    #2  0x00007ffff7c52951 in ?? () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
    #3  0x00007ffff7c5e47c in ?? () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
    #4  0x00007ffff7c5e4e7 in std::terminate() () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
    #5  0x00007ffff7c5e799 in __cxa_throw () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
    #6  0x00007ffff7c553eb in ?? () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
    #7  0x000055555555936c in std::vector<int, std::allocator<int> >::_M_range_check (
        this=0x5555555cfdb0, __n=100) at /usr/include/c++/9/bits/stl_vector.h:1070
    #8  0x0000555555558e1d in std::vector<int, std::allocator<int> >::at (this=0x5555555cfdb0,
        __n=100) at /usr/include/c++/9/bits/stl_vector.h:1091
    #9  0x000055555555828b in GDBTester::VectorCrash (this=0x5555555cfb40)
        at /home/steve/Documents/nav2_ws/src/gdb_test_pkg/src/gdb_test_node.cpp:44
    #10 0x0000555555559cfc in main (argc=1, argv=0x7fffffffc108)
        at /home/steve/Documents/nav2_ws/src/gdb_test_pkg/src/main.cpp:25
    ```

In this example you should read this in the following way, **starting at the bottom**:

> 在这个例子中，您应该**从底部开始**方式阅读：

- In the main function, on line 25 we call a function VectorCrash.
- In VectorCrash, on line 44, we crashed in the Vector\'s `at()` method with input `100`.
- It crashed in `at()` on STL vector line 1091 after throwing an exception from a range check failure.

> 它在 STL 向量第 1091 行的`at()`函数上崩溃，因范围检查失败而抛出异常。

These traces take some time to get used to reading, but in general, start at the bottom and follow it up the stack until you see the line it crashed on. Then you can deduce why it crashed. When you are done with GDB, type `quit` and it will exit the session and kill any processes still up. It may ask you if you want to kill some threads at the end, say yes.

> 这些跟踪记录需要一些时间来习惯阅读，但**一般来说，从底部开始，然后沿着堆栈跟踪，直到看到它崩溃的行**。然后你可以推断它为什么崩溃。当你完成 GDB 时，**输入`quit`它将退出会话并杀死任何仍然在运行的进程**。它可能会在最后问你是否想杀死一些线程，说 yes。

# From a Launch File

Just as in our non-ROS example, we need to setup a GDB session before launching our ROS 2 launch file. While we could set this up through the commandline, we can instead make use of the same mechanics that we did in the `ros2 run` node example, now using a launch file.

> 正如我们在非 ROS 示例中一样，我们需要在启动 ROS 2 启动文件之前设置 GDB 会话。虽然我们可以通过命令行设置，但我们可以使用与`ros2 run`节点示例中相同的机制，现在**使用启动文件**。

In your launch file, find the node that you\'re interested in debugging. For this section, we assume that your launch file contains only a single node (and potentially other information as well). The `Node` function used in the `launch_ros` package will take in a field `prefix` taking a list of prefix arguments. We will insert the GDB snippet here with one change from our node example, use of `xterm`. `xterm` will pop up a new terminal window to show and interact with GDB. We do this because of issues handling `stdin` on launch files (e.g. if you hit control+C, are you talking to GDB or launch?). See [this ticket](https://github.com/ros2/launch_ros/issues/165) for more information. See below for an example debugging SLAM Toolbox.

> 在你的启动文件中，找到你感兴趣调试的节点。 在这一部分中，我们假设你的启动文件仅包含一个节点(以及可能的其他信息)。 `launch_ros` 包中使用的 `Node` 函数将接受一个 `prefix` 字段，接受一个前缀参数列表。 我们将在此处插入 GDB 代码片段，与我们的节点示例有一处更改，即使用 `xterm`。 `xterm` 将弹出一个新的终端窗口，以显示和与 GDB 交互。 我们这样做是因为启动文件(例如，如果你按下 control+C，你是在与 **GDB 还是 launch 进行交谈？**)中处理 `stdin` 时出现的问题。 有关更多信息，请参见 [此票证](https://github.com/ros2/launch_ros/issues/165)。 有关调试 SLAM Toolbox 的示例，请参见下文。

    ```python
    start_sync_slam_toolbox_node = Node(
        parameters=[
        get_package_share_directory("slam_toolbox") + '/config/mapper_params_online_sync.yaml',
        {'use_sim_time': use_sim_time}
        ],
        package='slam_toolbox',
        executable='sync_slam_toolbox_node',
        name='slam_toolbox',
        prefix=['xterm -e gdb -ex run --args'],
        output='screen')
    ```

> [!NOTE]
> 还有这样的配置参数：`prefix=['xterm -e gdb -ex run --args'],`

Just as before, this prefix will launch a GDB session, now in `xterm` and run the launch file you requested with all the additional launch arguments defined.

> 正如以前一样，这个前缀将启动一个 GDB 会话，现在在`xterm`中运行您请求的启动文件，并使用所有附加的启动参数。

Once your server crashes, you\'ll see a prompt like below, now in the `xterm` session. At this point you can now get a backtrace.

> 一旦你的服务器崩溃，你会看到下面这样的提示，现在在`xterm`会话中。在这一点上，你现在可以得到一个回溯。

    ```bash
    (gdb)
    ```

In this session, type `backtrace` and it will provide you with a backtrace. Copy this for your needs. See the example trace in the section above for an example.

> 在这个会话中，输入`backtrace`，它将为您提供回溯。复制它以满足您的需求。请参阅上面部分的示例跟踪以获得示例。

These traces take some time to get used to reading, but in general, start at the bottom and follow it up the stack until you see the line it crashed on. Then you can deduce why it crashed. When you are done with GDB, type `quit` and it will exit the session and kill any processes still up. It may ask you if you want to kill some threads at the end, say yes.

> 这些跟踪需要一段时间来习惯阅读，但一般来说，从底部开始，沿着堆栈跟踪，直到看到它崩溃的行。然后你就可以推断出它为什么崩溃了。当你完成 GDB 之后，输入`quit`，它会退出会话并杀死任何仍然在运行的进程。它可能会在最后问你是否要杀死一些线程，请说是。

# From Large Project

Working with launch files with multiple nodes is a little different so you can interact with your GDB session without being bogged down by other logging in the same terminal. For this reason, when working with larger launch files, its good to pull out the specific server you\'re interested in and launching it seperately. These instructions are targeting Nav2, but are applicable to any large project with many nodes of any type in a series of launch file(s).

> **使用多个节点的启动文件工作有些不同**，因此您可以在不受其他同一终端登录的影响的情况下与 GDB 会话交互。因此，在使用较大的启动文件时，**最好将您感兴趣的特定服务器拉出来，并单独启动它**。这些说明针对 Nav2，但也适用于任何大型项目，其中包含一系列启动文件中的任何类型的多个节点。

As such, for this case, when you see a crash you\'d like to investigate, its beneficial to separate this server from the others.

> 如此，对于这种情况，当你看到一个你想要调查的崩溃时，**最好将这台服务器与其他服务器分开**。

If your server of interest is being launched from a nested launch file (e.g. an included launch file) you may want to do the following:

> 如果您感兴趣的服务器是从嵌套启动文件(例如包含的启动文件)启动的，您可能想要执行以下操作：

- Comment out the launch file inclusion from the parent launch file
- Recompile the package of interest with `-g` flag for debug symbols
- Launch the parent launch file in a terminal
- Launch the server\'s launch file in another terminal following the instructions in [From a Launch File](#from-a-launch-file).

> 在另一个终端中根据[从启动文件](#from-a-launch-file)中的说明启动服务器的启动文件。

Alternatively, if you server of interest is being launched in these files directly (e.g. you see a `Node`, `LifecycleNode`, or inside a `ComponentContainer`), you will need to seperate this from the others:

> 如果您感兴趣的服务器直接在这些文件中启动(例如您看到一个`Node`，`LifecycleNode`或在`ComponentContainer`中)，您需要将其与其他服务器分开：

- Comment out the node\'s inclusion from the parent launch file
- Recompile the package of interest with `-g` flag for debug symbols
- Launch the parent launch file in a terminal
- Launch the server\'s node in another terminal following the instructions in [From a Node](#from-a-node).

::: note
::: title
Note
:::

Note that in this case, you may need to remap or provide parameter files to this node if it was previously provided by the launch file. Using `--ros-args` you can give it the path to the new parameters file, remaps, or names. See [this ROS 2 tutorial](https://docs.ros.org/en/rolling/Guides/Node-arguments.html) for the commandline arguments required.

> 请注意，在这种情况下，如果启动文件以前提供了该节点，则可能需要重新启动或提供参数文件。使用`-ros-args`您可以为其提供新参数文件，重新映射或名称的路径。有关命令行参数，请参见[此 ROS 2 教程](https://docs.ros.org/en/rolling/guides/node-arguments.html)。

We understand this can be a pain, so it might encourage you to rather have each node possible as a separately included launch file to make debugging easier. An example set of arguments might be `--ros-args -r __node:=<node_name> --params-file /absolute/path/to/params.yaml` (as a template).

> 我们明白这可能会是件麻烦的事，因此可能会鼓励您**将每个节点可能作为单独包含的启动档来使除错更容易**。一个示例参数可能是`--ros-args -r __node:=<node_name> --params-file /absolute/path/to/params.yaml`(作为模板)。

:::

Once your server crashes, you\'ll see a prompt like below in the specific server\'s terminal. At this point you can now get a backtrace.

> 一旦您的服务器崩溃，您将在特定服务器的终端中看到类似下面的提示。此时，您可以获得回溯。

    ```bash
    (gdb)
    ```

In this session, type `backtrace` and it will provide you with a backtrace. Copy this for your needs. See the example trace in the section above for an example.

> 在本次会话中，输入`backtrace`，它将为您提供回溯。复制此内容以满足您的需求。参见上面部分的示例跟踪以获取示例。

These traces take some time to get used to reading, but in general, start at the bottom and follow it up the stack until you see the line it crashed on. Then you can deduce why it crashed. When you are done with GDB, type `quit` and it will exit the session and kill any processes still up. It may ask you if you want to kill some threads at the end, say yes.

> 这些踪迹需要一些时间来习惯阅读，但通常，从底部开始，然后沿着堆栈追踪，直到你看到它崩溃的行。然后你可以推断它为什么崩溃。当你完成 GDB 后，输入`quit`，它将退出会话并杀死任何仍然存在的进程。它可能会在最后问你是否要杀死一些线程，请说是。

# From Nav2 Bringup

To debug directly from the nav2 bringup launch files you may want to do the following:

> 要直接从 nav2 启动文件中调试，您可能需要执行以下操作：

- Add `prefix=['xterm -e gdb -ex run --args']` to the non-composed node in the appropriate launch file.

> 添加`prefix=['xterm -e gdb -ex run --args']`到适当的启动文件中的非组合节点。

- Recompile the package of interest with `-g` flag for debug symbols.

- Launch normally with `ros2 launch nav2_bringup tb3_simulation_launch.py use_composition:=False`. A seperate xterm window will open with the proccess of intrest running in gdb.

> 启动正常，使用`ros2 launch nav2_bringup tb3_simulation_launch.py use_composition:=False`。一个单独的 xterm 窗口将打开，在 gdb 中运行感兴趣的进程。

::: note
::: title
Note
:::

Turning off composition has serious performance impacts. If this is important to you please follow \"From Large Project\".

> 关闭组合会对性能造成严重影响。如果这对你很重要，请遵循“从大项目”。

:::

Once your server crashes, you\'ll see a prompt like below in the xterm window. At this point you can now get a backtrace.

> 一旦你的服务器崩溃，你会在 xterm 窗口中看到如下提示。此时你可以获取回溯。

    ```bash
    (gdb)
    ```

In this session, type `backtrace` and it will provide you with a backtrace. Copy this for your needs. See the example trace in the section above for an example.

> 在这个会话中，输入`backtrace`，它会为您提供一个回溯。将其复制以满足您的需求。请参阅上面部分的示例跟踪以获取示例。

These traces take some time to get used to reading, but in general, start at the bottom and follow it up the stack until you see the line it crashed on. Then you can deduce why it crashed. When you are done with GDB, type `quit` and it will exit the session and kill any processes still up. It may ask you if you want to kill some threads at the end, say yes.

> 这些跟踪需要一些时间来习惯阅读，但一般来说，从底部开始，沿着堆栈跟踪，直到看到它崩溃的行。然后你可以推断它为什么崩溃。当你完成 GDB 时，输入`quit`它将退出会话并杀死任何仍然在运行的进程。它可能会在最后问你是否要杀死一些线程，请说是。

# Automatic backtrace on crash

The [backward-cpp](https://github.com/bombela/backward-cpp) library provides beautiful stack traces, and the [backward_ros](https://github.com/pal-robotics/backward_ros/tree/foxy-devel) wrapper simplifies its integration.

> `backward-cpp`和`backward_ros`库提供漂亮的堆栈跟踪，`backward_ros`包装器可以简化集成。

Just add it as a dependency and [find_package]{.title-ref} it in your CMakeLists and the backward libraries will be injected in all your executables and libraries.

> 把它加为依赖，在你的 CMakeLists 中用`find_package`找到它，这些后向库就会被注入到所有你的可执行文件和库中。
