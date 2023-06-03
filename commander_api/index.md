translate by baidu@2023-06-07 15:22:16
---
---

title: Simple Commander API

> 标题：简单指挥官API
---

# Overview


The goal of the Nav2 Simple (Python3) Commander is to provide a \"navigation as a library\" capability to Python3 users. We provide an API that handles all the ROS 2 and Action Server tasks for you such that you can focus on building an application leveraging the capabilities of Nav2 (after you\'ve configured it to your liking with your plugins of choice). [We also provide you with demos and examples of API usage](https://github.com/ros-planning/navigation2/tree/main/nav2_simple_commander) to build common basic capabilities in autonomous mobile robotics in the `nav2_simple_commander` package.

> Nav2 Simple（Python3）Commander的目标是为Python4用户提供“作为库导航”功能。我们提供了一个API，可以为您处理所有ROS 2和Action Server任务，这样您就可以集中精力构建一个利用Nav2功能的应用程序（在您使用所选插件进行配置后）。[我们还为您提供API使用的演示和示例](https://github.com/ros-planning/navigation2/tree/main/nav2_simple_commander)在“nav2_simple_commander”包中构建自主移动机器人的通用基本功能。


A simple demonstration is shown below. Note: `goToPose()`, `goThroughPoses()`, `followWaypoints()` and similar are **non-blocking** such that you can receive and process feedback in a single-threaded application. As such while waiting for a task to be completed, the `while not nav.isTaskComplete()` design is necessary to poll for changes in the navigation completion, and if not complete some tasks of interest to your application (like processing feedback, doing something with the data the robot is collecting, or checking for faults).

> 下面显示了一个简单的演示。注意：“goToPose（）”、“goThroughPoses（）”和“followWaypoints（）”等都是**非阻塞**，因此您可以在单线程应用程序中接收和处理反馈。因此，在等待任务完成时，“while not nav.isTaskComplete（）”设计对于轮询导航完成中的更改是必要的，如果没有完成应用程序感兴趣的一些任务（如处理反馈、对机器人正在收集的数据进行处理或检查故障）。


You may use this simple commander preempt commands of the same type (e.g. you can preempt a `goToPose()` with another `goToPose()`) but you must explicitly cancel a current command and issue a new one if switching between `goToPose()`, `goThroughPoses()`, or `followWaypoints()`.

> 您可以使用这个简单的命令器抢占相同类型的命令（例如，您可以用另一个“goToPose（）”抢占一个“goToPose”），但如果在“goToPose（））”、“goThroughPoses（）”或“followWaypoints（）”之间切换，则必须明确取消当前命令并发出新命令。

``` python3

from nav2_simple_commander.robot_navigator import BasicNavigator

> 从nav2_simple_commander.robotnavigator导入BasicNavigator

import rclpy

> 导入rcpy


rclpy.init()

> rclpy.init（）

nav = BasicNavigator()

> nav=基本导航程序（）

# ...


nav.setInitialPose(init_pose)

> 导航设置初始姿势（init_pose）

nav.waitUntilNav2Active() # if autostarted, else use lifecycleStartup()

> nav.waitUntilNav2Active（）#如果自动启动，则使用lifecycleStartup（）

# ...


path = nav.getPath(init_pose, goal_pose)

> path=nav.getPath（init_pose，goal_pose）

smoothed_path = nav.smoothPath(path)

> smoothd_path=导航.soothPath（路径）

# ...


nav.goToPose(goal_pose)

> nav.goToPose（目标姿势）

while not nav.isTaskComplete():

> 而不是导航。isTaskComplete（）：

  feedback = nav.getFeedback()

> feedback=导航.getFeedback（）

  if feedback.navigation_duration > 600:

> 如果反馈.导航持续时间>600：
    nav.cancelTask()

# ...


result = nav.getResult()

> result=导航.getResult（）

if result == TaskResult.SUCCEEDED:

> 如果结果==任务结果。成功：
    print('Goal succeeded!')

elif result == TaskResult.CANCELED:

> elif结果==任务结果。取消：
    print('Goal was canceled!')

elif result == TaskResult.FAILED:

> elif结果==任务结果失败：
    print('Goal failed!')
```

# Commander API


The methods provided by the basic navigator are shown below, with inputs and expected returns. If a server fails, it may throw an exception or return a [None]{.title-ref} object, so please be sure to properly wrap your navigation calls in try/catch and check returns for [None]{.title-ref} type.

> 基本导航器提供的方法如下所示，包括输入和预期的返回。如果服务器失败，它可能会引发异常或返回[None]｛.title-ref｝对象，因此请确保在try/catch中正确包装导航调用，并检查[None]{.title-ref｝类型的返回。

  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  Robot Navigator Method                                                            Description

> 机器人导航器方法描述
  --------------------------------------------------------------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  setInitialPose(initial_pose)                                                      Sets the initial pose (`PoseStamped`) of the robot to localization.

> setInitialPose（initial_pose）将机器人的初始姿势（“PoseStamped”）设置为定位。


  goThroughPoses(poses, behavior_tree=\'\')                                         Requests the robot to drive through a set of poses (list of `PoseStamped`).

> goThroughPoses（poses，behavior_tree=\'\'）请求机器人通过一组姿势（“PoseStamped”列表）。


  goToPose(pose, behavior_tree=\'\')                                                Requests the robot to drive to a pose (`PoseStamped`).

> goToPose（pose，behavior_tree=\'\'）请求机器人驾驶到一个姿势（“PoseStamped”）。


  followWaypoints(poses)                                                            Requests the robot to follow a set of waypoints (list of `PoseStamped`). This will execute the chosen `TaskExecutor` plugin at each pose.

> followWaypoints（poses）请求机器人遵循一组路点（“PoseStamped”列表）。这将在每个姿势执行选定的“任务执行器”插件。


  followPath(path, controller_id=\'\', goal_checker_id=\'\')                        Requests the robot to follow a path from a starting to a goal `PoseStamped`, `nav_msgs/Path`.

> followPath（path，controller_id=\'\'，goal_checker_id=\'\`）请求机器人遵循从起点到目标的路径`PoseStamped`，`nav_msgs/path`。


  spin(spin_dist=1.57, time_allowance=10)                                           Requests the robot to performs an in-place rotation by a given angle.

> spin（spin_dist=1.57，time_allowance=10）请求机器人执行给定角度的原地旋转。


  backup(backup_dist=0.15, backup_speed=0.025, time_allowance=10)                   Requests the robot to back up by a given distance.

> backup（backup_dist=0.15，backup_speed=0.025，time_allowance=10）请求机器人后退给定距离。


  assistedTeleop(time_allowance=30)                                                 Requests the robot to run the assisted teleop action.

> assistantedTeleop（time_allowance=30）请求机器人运行辅助遥操作。


  cancelTask()                                                                      Cancel an ongoing task, including route tasks.

> cancelTask（）取消正在进行的任务，包括路由任务。


  isTaskComplete(trackingRoute=False)                                               Checks if task is complete yet, times out at `100ms`. Returns `True` if completed and `False` if still going. If checking a route tracking task, set default argument to `True`.

> isTaskComplete（trackingRoute=False）检查任务是否已完成，超时时间为“100ms”。如果已完成，则返回“True”；如果仍在继续，则返回”False“。如果检查路线跟踪任务，请将默认参数设置为“True”。


  getFeedback(trackingRoute=False)                                                  Gets feedback from task, returns action server feedback msg. If getting feedback on a tracking task, set default argument to `True`.

> getFeedback（trackingRoute=False）从任务中获取反馈，返回操作服务器反馈消息。如果获得跟踪任务的反馈，请将默认参数设置为“True”。


  getResult()                                                                       Gets final result of task, to be called after `isTaskComplete` returns `True`. Returns action server result msg.

> getResult（）获取任务的最终结果，在“isTaskComplete”返回“True”后调用。返回操作服务器结果消息。


  getPath(start, goal, planner_id=\'\', use_start=False)                            Gets a path from a starting to a goal `PoseStamped`, `nav_msgs/Path`.

> getPath（start，goal，planner_id=\'\'，use_start=False）获取从起点到目标的路径`PoseStamped`，`nav_msgs/path`。


  getPathThroughPoses(start, goals, planner_id=\'\', use_start=False)               Gets a path through a starting to a set of goals, a list of `PoseStamped`, `nav_msgs/Path`.

> getPathThroughPoses（start，goals，planner_id=\'\'，use_start=False）获取一个从起点到一组目标的路径，即“PoseStamped”、“nav_msgs/path”的列表。


  getRoute(start, goal, use_start=False)                                            Gets a sparse route and dense path from start to goal, where start and goal may be of type `PoseStamped` or `int` for known NodeIDs.

> getRoute（start，goal，use_start=False）获取从开始到目标的稀疏路由和密集路径，其中对于已知NodeID，start和goal的类型可以是“PoseStamped”或“int”。


  getandTrackRoute(start, goal, use_start=False)                                    Gets and tracks a sparse route and dense path from start to goal, where start & goal may be of type `PoseStamped` or `int` for known NodeIDs.

> getandTrackRoute（start，goal，use_start=False）获取并跟踪从开始到目标的稀疏路径和密集路径，其中对于已知的NodeID，start&goal的类型可以是“PoseStamped”或“int”。


  smoothPath(path, smoother_id=\'\', max_duration=2.0, check_for_collision=False)   Smooths a given path of type `nav_msgs/Path`.

> smoothPath（path，smooter_id=\'\'，max_duration=2.0，check_for_collision=False）平滑给定的“nav_msgs/path”类型的路径。


  changeMap(map_filepath)                                                           Requests a change from the current map to [map_filepath]{.title-ref}\'s yaml.

> changeMap（map_filepath）请求将当前映射更改为[map_filepath]｛.title-ref｝\的yaml。


  clearAllCostmaps()                                                                Clears both the global and local costmaps.

> clearAllCostmaps（）清除全局和本地成本图。


  clearLocalCostmap()                                                               Clears the local costmap.

> clearLocalCostmap（）清除本地成本图。


  clearGlobalCostmap()                                                              Clears the global costmap.

> clearGlobalCostmap（）清除全局成本图。


  getGlobalCostmap()                                                                Returns the global costmap, `nav2_msgs/Costmap`.

> getGlobalCostmap（）返回全局成本图，`nav2_msgs/costmap`。


  getLocalCostmap()                                                                 Returns the local costmap, `nav2_msgs/Costmap`.

> getLocalCostmap（）返回本地成本图，`nav2_msgs/costmap`。


  waitUntilNav2Active( navigator=\'bt_navigator\', localizer=\'amcl\')              Blocks until Nav2 is completely online and lifecycle nodes are in the active state. To be used in conjunction with autostart or external lifecycle bringup. Custom navigator and localizer nodes can be specified

> waitUntilNav2Active（navigator=\'bt_anavigator\'，localizer=\'amcl\'）阻止，直到Nav2完全联机并且生命周期节点处于活动状态。与自动启动或外部生命周期启动一起使用。可以指定自定义导航器和定位器节点


  lifecycleStartup()                                                                Sends a request to all lifecycle management servers to bring them into the active state, to be used if autostart is `False` and you want this program to control Nav2\'s lifecycle.

> lifecycleStartup（）向所有生命周期管理服务器发送一个请求，使其进入活动状态，如果autostart为“False”并且您希望此程序控制Nav2\的生命周期，则使用该请求。


  lifecycleShutdown()                                                               Sends a request to all lifecycle management servers to shut them down.

> lifecycleShutdown（）向所有生命周期管理服务器发送关闭请求。


  destroyNode()                                                                     Releases the resources used by the object.

> destroyNode（）释放对象使用的资源。
  -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Costmap API


This is a Python3 API for costmap 2d messages from the stack. It provides the basic conversion, get/set, and handling semantics found in the costmap 2d C++ API.

> 这是一个Python3 API，用于堆栈中的costmap 2d消息。它提供了costmap 2d C++API中的基本转换、获取/设置和处理语义。

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  Costmap Method                Description

> 成本图方法说明
  ----------------------------- -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  getSizeInCellsX()             Get map width in cells.

> getSizeInCellsX（）获取单元格中的贴图宽度。


  getSizeInCellsY()             Get map height in cells.

> getSizeInCellsY（）获取单元格中的贴图高度。


  getSizeInMetersX()            Get x axis map size in meters.

> getSizeInMetersX（）获取以米为单位的x轴贴图大小。


  getSizeInMetersY()            Get y axis map size in meters.

> getSizeInMetersY（）获取以米为单位的y轴贴图大小。


  getOriginX()                  Get the origin x axis of the map \[m\].

> getOriginX（）获取地图\[m\]的原点x轴。


  getOriginY()                  Get the origin y axis of the map \[m\].

> getOriginY（）获取地图\[m\]的原点y轴。


  getResolution()               Get map resolution \[m/cell\].

> getResolution（）获取地图分辨率\[m/cell\]。


  getGlobalFrameID()            Get global frame_id.

> getGlobalFrameID（）获取全局frame_id。


  getCostmapTimestamp()         Get costmap timestamp.

> getCostmapTimestamp（）获取成本图时间戳。


  getCostXY(mx, my)             Get the cost (`np.uint8`) of a cell in the costmap using mx (`int`) , my (`int`) of Map Coordinate.

> getCostXY（mx，my）使用Map Coordinate的mx（`int`），my（`int'）获取成本图中单元格的成本（`np.uint8`）。


  getCostIdx(index)             Get the cost (`np.uint8`) of a cell in the costmap using index (`int`)

> getCostIdx（index）使用index（`int`）获取成本图中单元格的成本（`np.uint8`）


  setCost(mx, my, cost)         Set the cost (`np.uint8`) of a cell in the costmap using mx (`int`) , my (`int`) of Map Coordinate.

> setCost（mx，my，cost）使用Map Coordinate的mx（`int`），my（`int'）设置成本图中单元格的成本（`np.uint8`）。


  mapToWorld(mx, my)            Get the wx (`float`) \[m\], wy (`float`) \[m\] of world coordinate XY using mx (`int`), my (`int`) of map coordinate XY

> mapToWorld（mx，my）使用地图坐标XY的mx（`int`），my（`int'）获取世界坐标XY的wx（`float`）\[m\]，wy（`floate`）\[m\]


  worldToMapValidated(wx, wy)   Get the mx (`int`), my (`int`) of map coordinate XY using wx (`float`) \[m\], wy (`float`) \[m\] of world coordinate XY. If wx wy coordinates are invalid, (None,None) is returned.

> worldToMapValidated（wx，wy）使用世界坐标XY的wx（`float`）\[m\]，wy（`float `）\[m]获取地图坐标XY的mx（`int`），my（`int'）。如果wx-wy坐标无效，则返回（None，None）。


  getIndex(mx, my)              Get the index (`int`) of the cell using mx (`int`), my (`int`) of map coordinate XY

> getIndex（mx，my）使用映射坐标XY的mx（`int`），my（`int'）获取单元格的索引（`int`'）
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Footprint Collision Checker API


This is a Python3 API for a Footprint Collision Checker. It provides the needed methods to manipulate the coordinates and calculate the cost of a Footprint in a given map.

> 这是足迹碰撞检查器的Python3 API。它提供了在给定地图中操作坐标和计算足迹成本所需的方法。

  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  Footprint Collision Checker Method            Description

> 足迹碰撞检查器方法描述
  --------------------------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  footprintCost(footprint)                      Checks the footprint (`Polygon`) for collision at its implicit provided coordinate pose.

> footprintCast（footprint）检查footprint'（“多边形”）在其隐式提供的坐标姿势处是否发生碰撞。


  lineCost(x0, x1, y0, y1, step_size=0.5)       Iterate over all the points along a line and check for collision. The line is defined by x0, y0, x1, y1, step_size (`int`) or (`float`).

> lineCost（x0，x1，y0，y1，step_size=0.5）在沿线的所有点上迭代，并检查是否存在碰撞。该行由x0、y0、x1、y1、step_size（`int`）或（`float`）定义。


  worldToMapValidated(wx, wy)                   Get the mx (`int`), my (`int`) of map coordinate XY using wx (`float`) \[m\], wy (`float`) \[m\] of world coordinate XY. If wx wy coordinates are invalid, (None,None) is returned. Returns None if costmap is not defined yet through (`setCostmap(costmap)`).

> worldToMapValidated（wx，wy）使用世界坐标XY的wx（`float`）\[m\]，wy（`float `）\[m]获取地图坐标XY的mx（`int`），my（`int'）。如果wx-wy坐标无效，则返回（None，None）。如果尚未通过（`setCostmap（costmap）`）定义成本图，则返回None。


  pointCost(x, y)                               Get the cost of a point in the costmap using map coordinates XY. (`int`)

> pointCost（x，y）使用地图坐标XY获取成本图中某个点的成本。（`int`）


  setCostmap(costmap)                           Specify which costmap to use with the footprint collision checker. (`PyCostmap2D`)

> setCostmap（成本图）指定要与封装外形碰撞检查器一起使用的成本图。（`PyCostmap2D`）


  footprintCostAtPose(x, y, theta, footprint)   Get the cost of a footprint at a specific Pose in map coordinates. x, y, theta (`float`) footprint (`Polygon`).

> footprintCostAtPose（x，y，theta，footprint）获取地图坐标中特定Pose的足迹成本。x、 y，θ（`float`）足迹（`Polygon`）。
  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Examples and Demos


All of these can be found in the [package](https://github.com/ros-planning/navigation2/tree/main/nav2_simple_commander).

> 所有这些都可以在[包]中找到(https://github.com/ros-planning/navigation2/tree/main/nav2_simple_commander)。

![Alternative text](readme.gif){.align-center width="800px"}


The `nav2_simple_commander` has a few examples to highlight the API functions available to you as a user:

> “nav2_simple_commander”有几个示例来突出显示用户可用的API函数：

-   `example_nav_to_pose.py` - Demonstrates the navigate to pose capabilities of the navigator, as well as a number of auxiliary methods.
-   `example_nav_through_poses.py` - Demonstrates the navigate through poses capabilities of the navigator, as well as a number of auxiliary methods.
-   `example_waypoint_follower.py` - Demonstrates the waypoint following capabilities of the navigator, as well as a number of auxiliary methods.
-   `example_follow_path.py` - Demonstrates the path following capabilities of the navigator, as well as a number of auxiliary methods like path smoothing.
-   `example_assisted_teleop.py` - Demonstrates the assisted teleop capabilities of the navigator.
-   `example_route.py` - Demonstrates the Route server capabilities of the navigator.


The `nav2_simple_commander` has a few demonstrations to highlight a couple of simple autonomy applications you can build using the API:

> “nav2_simple_commander”有几个演示，重点介绍了几个可以使用API构建的简单自治应用程序：

-   `demo_security.py` - A simple security robot application, showing how to have a robot follow a security route using Navigate Through Poses to do a patrol route, indefinitely.
-   `demo_picking.py` - A simple item picking application, showing how to have a robot drive to a specific shelf in a warehouse to either pick an item or have a person place an item into a basket and deliver it to a destination for shipping using Navigate To Pose.
-   `demo_inspection.py` - A simple shelf inspection application, showing how to use the Waypoint Follower and task executors to take pictures, RFID scans, etc of shelves to analyze the current shelf statuses and locate items in the warehouse.
