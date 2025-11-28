---
layout: page
title: Simulation of Multiple Robots in Warehouse
description: |
  Control and simulate multiple robots working in the open warehouse environment where other uncontrolled agents are working around. The robots are expected to visit the assigned tasks and avoid not only the obstacles that are represented by the occupancy map (e.g. shelf, wall) but also  the uncontrolled static and moving agents that are not represented by the occupancy map.

img: /assets/img/projects/multi-robot-system/fullview.gif
importance: 1
category: Multi-robot System
github: https://github.com/yangfan/warehouseRobot
---

This project designs a multi-robot system working in the warehouse facility or distribution center. The robots are moving around the facility to pick up packages from the loading stations and delivering them to unloading stations for storing or processing as shown in the figure below. It is assumed that the occupancy grid map that represents the free and occupied space is available. However there could be other uncontrolled moving agents working in the same environment and those agents cannot be included in the occupancy map. In this project, the robots working in the open environment are able to avoid both the obstacles represented in the occupancy grid map and the uncontrolled agents.

<div class="row justify-content-center">
    <div class="col-9">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/multi-robot-system/warehouse.png' | relative_url }}" alt=""/>
    </div>
</div>  
<br />
## Model Overview

The multi-robot delivery system in this project is modeled in Simulink with the Stateflow charts and Robotics System Toolbox in matlab. The model is comprised of three parts: central scheduler, robot controller, plant.

In particular, the central scheduler sends the dilivery command to each robot. For example, when a robot has arrived at the assigned loading station and picked up the books, the central scheduler will send the message that asks the robot to visit the specific unloading station for the books. When robot finishes the delivery, it will ask robot to go back to the charging station or pick up other packages depending on the schedule designed in the Stateflow charts.

The robot controller will plan the path based on the delivery commands sent by the central scheduler. Then it computes the velocity commands for robot to follow the planned path. If robot have to avoid the uncontrolled agents, the robot controller should further adjust the velocity command based on the observation of the surrounding environment obtained from the sensors.

The plant module contains differential-drive robot model which simulates the execution of the velocity commands and return the ground-truth poses of the robot.

<div class="row justify-content-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/multi-robot-system/model.png' | relative_url }}" alt=""/>
    </div>
</div>
<br />

## Collision Avoidance

In order to deliver the packages successfully, robots have to avoid any obstacle during the motion. We need to deal with three problems: (1) How to plan the collision-free path for robot to pick up package and deliver them to unloading station? (2) How to compute the velocity command such that the robot can follow the planned path? (3) How to avoid the unexpected obstacles that may appear when robot is moving? Three components are used in robot controller module to solve each of the three problems: `mobileRobotPRM` roadmap path planner, `Pure Pursuit` path tracking algorithm and `Vector Field Histogram` (VFH) block.

<div class="row justify-content-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/multi-robot-system/robot_controller.png' | relative_url }}" alt=""/>
    </div>
</div>
<br/>
 It is assumed that we have a certain knowledge of the environment in the warehouse so that the occupancy grid map can be computed. The `mobileRobotPRM` function takes the occupancy grid map as the argument and computes a probabilistic roadmap (PRM) based on the free and occupied spaces. The PRM is a network graph of possible path. Given the start and goal location, PRM uses the network of connected nodes to find an obstacle-free path.

Ideally, robot can reach the goal position by simply following the planned path. However in realistic environment, robot's motion is inherently uncertain. For example, the robot could drift off the path on the uneven road. Therefore it is necessary to use a path tracking algorithm, e.g., pure pursuit. The `pure pursuit` computes the velocity commands for following a path using a set of waypoints and current pose. The algorithm tries to move the robot from its current position to reach some look-ahead point in front of the robot.

With `mobileRobotPRM` and `pure pursuit`, robot is able to work in the environment where the only moving objects are robots. The problem happens when robot are working in the open environment where there are uncontrolled agents. Since those agents are not represented in the occupancy map, they may appear in the planned obstacle-free path for the robots. The `VFH` block is used to deal with this issue. The VFH check if the direction provided by the pure pursuit is obstacle-free based on the range-bearing sensor readings and compute steering direction closest to the target direction if there are obstacles in front of the robots. In particular, VFH computes polar density histogram to identify obstacle location and proximity. These histograms are then converted to binary histogram to indicate valid steering direction for the robot.

For example, the binary occupancy map of an warehouse environment is shown in the figure below. Note that an uncontrolled obstacle located in [35, 30] is not included in the map used to compute the PRM. Given the dilvery command fed by the central schedular, the PRM computes the obstacle-free path to the goal location. The robot follows the planned path based on the velocity commands sent by the pure pursuit controller. When robot is close to the uncontrolled obstacle, VFH adjust the steering direction by the sensor readings.

<div class="row justify-content-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/multi-robot-system/single_robot.gif' | relative_url }}" alt=""/>
    </div>
</div>
<br/>

In addition to the static obstacle, the robot can also avoid the uncontrolled moving obstacle. As shown in the second example below, multiple robots are moving in the same environment and they don't know the position of other robots until the sensors observe them. Besides the static obstacle, each robots should also avoid other robots which are not considered in computing the path by PRM.

<div class="row justify-content-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/multi-robot-system/fullview.gif' | relative_url }}" alt=""/>
    </div>
</div>
<br/>

A local view on robot 2.

<div class="row justify-content-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/multi-robot-system/view2.gif' | relative_url }}" alt=""/>
    </div>
</div>
<br/>

## Discussion

In this project, we simulate a multi-robot system for the package delivery in the warehouse. Given the assigned packages, the robots visit loading stations and unloading stations with the collision avoidance. The collision avoidance scheme used in this project enable robots to avoid both the static and moving obstacles that are uncontrolled in the environment. Beside the indoor warehouse environment, the collision avoidance scheme may also applicable to the ourdoor application such as passenger delivery and last-mile delivery.

There are several remaining questions that are not discussed in this project. Although the robots are able to finish the delivery without the collision, if we care about the travel cost or the delivery time, the package assignment and path planning used in this project may not be the optimal especially in the high uncertain open environment. Therefore the task allocation and path planning under the uncertain are important in the multi-robot system. In fact, these are parts of the movitations of my research. I have published two ICRA papers that proposed algorithms to simultaneously compute the task allocation and path planning and the solution provide a probablistic guarantee on robot's performance in each execution of the task. They are available to read on my publication page.

### Repository

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="yangfan/warehouseRobot" %}
</div>
