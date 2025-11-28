---
layout: page
title: Graph-based SLAM
description: |
  This project implements GraphSLAM algorithm which solves the full SLAM problem. Given a graph in which each node refers to a robot pose or landmark position and the edge represents a spatial constraint between two poses, the algorithm update the node configuration such that the error introduced by the constraints is minimized.
img: /assets/img/projects/SLAM/Graph-based-slam/graph-slam-intel.gif
importance: 4
category: SLAM
github: https://github.com/yangfan/probabilitics-robotics/tree/master/Graph_SLAM
images:
  - path:
    column:
    text:
not_empty: true
---

The GraphSLAM is a full SLAM algorithm. It defines a sparse graph in which each node corresponds to a pose of the robot or landmark and each edge corresponds to a spatial constraint between two nodes. The posterior of the full SLAM problem is therefore represented by this graph. There are two components in Graph-based SLAM: (1) Graph construction (front-end); (2) Graph optimization (back-end).

The methods for the front-end problem include dense scan-matching, feature-based matching and descriptor-based matching. In this project, we implement GraphSLAM for solving the back-end problem. More specifically, given the graph structure, we need to compute the node configuration such that the error introduced by the constraints is minimized.

## GraphSLAM Overview

Given the raw data, i.e., measurement $$z_{1:t}$$ and the controls $$u_{1:t}$$, GraphSLAM first converts the data into a graph. The nodes in the graphs are the robot poses and features in the map. The edge between two nodes indicates the spatial constraint between them.

More specifically, an edge exists when (1) the robot moves from one node to another; (2) the robot observes a map feasture; (3) the robot observes map feature from two nodes.

Therefore the error functions are:

1. error between robot poses:
   $$e_{ij}(x_i, x_j) = \mathrm{t2v}(Z_{ij}^{-1}(X_i^{-1} X_j))$$,
   where function $$\mathrm{t2v}$$ converts transfromation matrix to the vector of pose. $$X_i, X_j$$ are the transformation of node i and node j. $$Z_{ij}$$ is the observation of node j from node i.

2. error between robot pose i and landmark pose j:
   $$e_{ij}(x_i,x_j) = R^T_i(x_j - t_i) - z_{ij}$$ where $$R_i$$ denotes the rotation matrix of node i, $$X_j, t_i$$ denote the translation of landmark and robot respectively, $$z_{ij}$$ is observation of landmark j with respective to robot pose i.

The goal of the GraphSLAM is:

$$
x^* = \mathrm{argmin}_x \sum_{ij} e_{ij}^T \Omega e_{ij}
$$

We use the Gauss–Newton algorithm to minimize the error. The basic steps of the algorithm is following:

1. Linearize error function around the current poses $$x$$ and compute error for each edge.

   $$e_{ij}(x+\Delta x) \approxeq e_{ij} + J_{ij}\Delta x$$

2. Compute the terms for the linear system.

   $$
   \begin{equation*}
   \begin{split}
   b^T &= \sum_{ij} e_{ij}^T \Omega_{ij} J_{ij} \\
   H &= \sum_{ij} J_{ij}^T \Omega_{ij} J_{ij}
   \end{split}
   \end{equation*}
   $$

3. Solve the linear system ($$H$$ in this case is a sparse matrix).

   $$\Delta x^* = -H^{-1} b$$

4. Update the poses.

   $$x \leftarrow x + \Delta x^*$$

5. repeat procedure until the convergence.

## Code Explanation

1. `Fx = compute_global_error(g)`: compute the error of the current graph `g`.

2. `[e, A, B] = linearize_pose_pose_constraint(x1, x2, z)`: compute the error `e` and Jacobian `A`, `B` of a pose-pose constraint based on the current robot poses `x1`, `x2` and observation `z`.

3. `[e, A, B] = linearize_pose_landmark_constraint(x, l, z)`: compute the error `e` and Jacobian `A`, `B` of a pose-landmark constraint based on the robot pose `x`, landmark position `l` and the observation `z`.

4. `dx = linearize_and_solve(g)`: perform one iteration of Gauss-Newton algorithm, construct and solve the linear approximation.

## Results

The GraphSLAM algorithm computes solution on four data set<sup>[1]</sup>. The robot poses are represented by the blue crosses. The landmark positions are represented by the red dots. The edges are the green lines.

The figure below shows the results for the graph with pose-pose constraints only.

<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/SLAM/Graph-based-slam/graph-slam-pose-pose.gif'| relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col-sm-6 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/SLAM/Graph-based-slam/graph-slam-intel.gif'| relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
  Left: data set: simulation-pose-pose.dat, initial error: 138862234.08, Final error: 8269.42;
  
   Right: data set: intel.dat, Initial error: 1795138.99, Final error: 359.99.
</div>
<br/>

The following figures show the results for graph with both pose-pose constraints and pose-landmark constraints.

<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/SLAM/Graph-based-slam/graph-slam-pose-landmark.gif'| relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col-sm-6 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/SLAM/Graph-based-slam/graph-slam-dlr.gif'| relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
  Left: data set: simulation-pose-landmark.dat, initial error: 3030.31, Final error: 474.10;
  
   Right: data set: dlr.dat, Initial error: 369655335.57, Final error: 56860.35.
</div>
<br/>

### Repository

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="yangfan/probabilitics-robotics/tree/master/Graph_SLAM" %}
</div>

### References

1. **Robot Mapping**  
   University of Freiburg, WS 2013/14, [http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/)
