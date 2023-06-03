---
tip: translate by openai@2023-06-02 11:35:21
title: Behavior Server
---

Source code on [Github](https://github.com/ros-planning/navigation2/tree/main/nav2_behaviors).

The Behavior Server implements the server for handling recovery behavior requests and hosting a vector of plugins implementing various C++ behaviors. It is also possible to implement independent behavior servers for each custom behavior, but this server will allow multiple behaviors to share resources such as costmaps and TF buffers to lower incremental costs for new behaviors.

> **行为服务器(Behavior Server)**实现了**处理恢复行为请求和托管一组实现**各种 C++行为的插件的服务器。也可以为每个自定义行为实现独立的行为服务器，但是**该服务器将允许多个行为共享资源**，如成本图和 TF 缓冲区，以降低新行为的增量成本。

> [!NOTE]
> 这里给出的 “Behavior Server” 还真是可以！

# Behavior Server Parameters

local_costmap_topic

:

> Type Default
>
> ---
>
> string \"local_costmap/costmap_raw\"
>
> Description
>
> : Raw costmap topic for collision checking on the local costmap.

global_costmap_topic

:

> Type Default
>
> ---
>
> string \"global_costmap/costmap_raw\"
>
> Description
>
> : Raw costmap topic for collision checking on the global costmap.

local_footprint_topic

:

> Type Default
>
> ---
>
> string \"local_costmap/published_footprint\"
>
> Description
>
> : Topic for footprint in the local costmap frame.

global_footprint_topic

:

> Type Default
>
> ---
>
> string \"global_costmap/published_footprint\"
>
> Description
>
> : Topic for footprint in the global costmap frame.

cycle_frequency

:

> Type Default
>
> ---
>
> double 10.0
>
> Description
>
> : Frequency to run behavior plugins.

transform_tolerance

:

> Type Default
>
> ---
>
> double 0.1
>
> Description
>
> : TF transform tolerance.

local_frame

:

> Type Default
>
> ---
>
> string \"odom\"
>
> Description
>
> : Local reference frame.

global_frame

:

> Type Default
>
> ---
>
> string \"map\"
>
> Description
>
> : Global reference frame.

robot_base_frame

:

> Type Default
>
> ---
>
> string \"base_link\"
>
> Description
>
> : Robot base frame.

behavior_plugins

:

> Type Default
>
> ---
>
> vector\<string\> {\"spin\", \"back_up\", \"drive_on_heading\", \"wait\"}
>
> Description
>
> : List of plugin names to use, also matches action server names.
>
> Note
>
> : Each plugin namespace defined in this list needs to have a `plugin` parameter defining the type of plugin to be loaded in the namespace.
>
>     Example:
>
>     ``` yaml
>     behavior_server:
>       ros__parameters:
>         behavior_plugins: ["spin", "backup", "drive_on_heading", "wait"]
>         spin:
>           plugin: "nav2_behaviors/Spin"
>         backup:
>           plugin: "nav2_behaviors/BackUp"
>         drive_on_heading:
>           plugin: "nav2_behaviors/DriveOnHeading"
>         wait:
>           plugin: "nav2_behaviors/Wait"
>     ```

# Default Plugins

When the `behavior_plugins` parameter is not overridden, the following default plugins are loaded:

> Namespace Plugin
>
> ---
>
> \"spin\" \"nav2_behaviors/Spin\" > \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
> \"backup\" \"nav2_behaviors/BackUp\" > \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
> \"drive_on_heading\" \"nav2_behaviors/DriveOnHeading\" > \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-- \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--
> \"wait\" \"nav2_behaviors/Wait\"

# Spin Behavior Parameters

Spin distance is given from the action request

simulate_ahead_time

:

> Type Default
>
> ---
>
> double 2.0
>
> Description
>
> : Time to look ahead for collisions (s).

max_rotational_vel

:

> Type Default
>
> ---
>
> double 1.0
>
> Description
>
> : Maximum rotational velocity (rad/s).

min_rotational_vel

:

> Type Default
>
> ---
>
> double 0.4
>
> Description
>
> : Minimum rotational velocity (rad/s).

rotational_acc_lim

:

> Type Default
>
> ---
>
> double 3.2
>
> Description
>
> : maximum rotational acceleration (rad/s\^2).

# BackUp Behavior Parameters

Backup distance, speed and time_allowance is given from the action request.

simulate_ahead_time

:

> Type Default
>
> ---
>
> double 2.0
>
> Description
>
> : Time to look ahead for collisions (s).

# DriveOnHeading Behavior Parameters

DriveOnHeading distance, speed and time_allowance is given from the action request.

simulate_ahead_time

:

> Type Default
>
> ---
>
> double 2.0
>
> Description
>
> : Time to look ahead for collisions (s).

# AssistedTeleop Behavior Parameters

AssistedTeleop time_allowance is given in the action request

projection_time

:

> Type Default
>
> ---
>
> double 1.0
>
> Description
>
> : Time to look ahead for collisions (s).

simulation_time_step

:

> Type Default
>
> ---
>
> double 0.1
>
> Description
>
> : Time step for projections (s).

cmd_vel_teleop

:

> Type Default
>
> ---
>
> string cmd_vel_teleop
>
> Description
>
> : Topic to listen for teleop messages.

# Example

```yaml
behavior_server:
  ros__parameters:
    local_costmap_topic: local_costmap/costmap_raw
    local_footprint_topic: local_costmap/published_footprint
    global_costmap_topic: global_costmap/costmap_raw
    global_footprint_topic: global_costmap/published_footprint
    cycle_frequency: 10.0
    behavior_plugins:
      ["spin", "backup", "drive_on_heading", "wait", "assisted_teleop"]
    spin:
      plugin: "nav2_behaviors/Spin"
    backup:
      plugin: "nav2_behaviors/BackUp"
    drive_on_heading:
      plugin: "nav2_behaviors/DriveOnHeading"
    wait:
      plugin: "nav2_behaviors/Wait"
    assisted_teleop:
      plugin: "nav2_behaviors/AssistedTeleop"
    local_frame: odom
    global_frame: map
    robot_base_frame: base_link
    transform_timeout: 0.1
    simulate_ahead_time: 2.0
    max_rotational_vel: 1.0
    min_rotational_vel: 0.4
    rotational_acc_lim: 3.2
```
