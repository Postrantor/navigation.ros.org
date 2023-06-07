---
title: Smac Planner
---

Source code and `README` with design, explanations, and metrics can be found on [Github](https://github.com/ros-planning/navigation2/tree/main/nav2_smac_planner). A brief explanation can be found below, but the `README` contains the most detailed overview of the framework and planner implementations.

The Smac Planner plugin implements three A\* based planning algorithms: 2D A\*, Hybrid-A\*, and State Lattice path planners. It is important to know that in June 2021 and December 2021, the package received a several **major** updates both improving path quality and run-times by 2-3x.

# Provided Plugins

> The plugins listed below are inside the `nav2_smac_planner` package. See the pages for individual configuration information.

::: {.toctree maxdepth="1"}
smac/configuring-smac-2d.rst smac/configuring-smac-hybrid.rst smac/configuring-smac-lattice.rst
:::

# Description

The `nav2_smac_planner` package contains an optimized templated A\* search algorithm used to create multiple A\*-based planners for multiple types of robot platforms. It uses template node types to develop different search-based planners.

We support circular differential-drive and omni-directional drive robots using the `SmacPlanner2D` planner which implements a cost-aware A\* planner. We support car-like (ackermann) and legged vehicles using the `SmacPlannerHybrid` plugin which implements a Hybrid-A\* planner. We support non-circular, arbitrary shaped, any model vehicles using the `SmacPlannerLattice` plugin which implements a State Lattice planner (e.g. omni, diff, ackermann, legged, custom). It contains control sets and generators for ackermann, legged, differential drive and omnidirectional vehicles, but you may provide your own for another robot type or to have different planning behaviors.

The last two plugins are both **kinematically feasible** and support **reversing**. They have performance similar to its 2D counter parts like 2D-A\* and NavFn via highly optimized heuristic functions and efficient programming. An example of the 3 planners can be seen below, planning a roughly 75 m path.

-   Hybrid-A\* computed the path in 144ms
-   State Lattice computed the path in 113ms
-   2D A\* computed the path in 243ms
-   For reference: NavFn compute the path in 146ms, including some nasty path discontinuity artifacts

Usual planning times are below 100ms for some environments, occasionally approaching up to 200ms. The performance of all 3 planners is roughly comparable with naive 2D search algorithms that have long been mainstays of the ROS Navigation ecosystem, but also achieving kinematic feasibility, support for reversing, and using modern state of the art techniques.

![2D A\* (Panel 1), Hybrid-A\* (Panel 2), State Lattice (Panel 3)](smac/3planners.png){.align-center}