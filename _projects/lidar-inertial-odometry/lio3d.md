---
layout: page
title: ESKF Lidar Inertial Odometry
description: |
  This project implements 3D lidar inertial odometry based on error state kalman filter. Given 3D lidar scan and IMU sensor data, the goal is to create 3D point cloud map of the environment. The key elements include ESKF for state estimation, point cloud registration estimating transformation between two point clouds, and lidar-inertial odometry creating keyframes which form the 3D point cloud map. Two types of LIO is implemented, namely loosely coupled LIO which only takes point cloud alignment as the observation in ESKF and tightly coupled LIO which takes residuals of point cloud alignment as part of the ESKF observation model.
img: /assets/img/projects/lidar-inertial-odometry/lio3d/ndt_inc_lo.gif
importance: 1
category: 3D Lidar Inertial Odometry
github: https://github.com/yangfan/lio3d
---

This project implements 3D lidar inertial odometry based on error state kalman filter. Given 3D lidar scan and IMU sensor data, the goal is to create 3D point cloud map of the environment. The key elements include ESKF for state estimation, point cloud registration estimating transformation between two point clouds, and lio that creating keyframes which form the 3D point cloud map. Two types of LIO is implemented, namely loosely coupled LIO which only takes point cloud alignment as the observation in ESKF and tightly coupled LIO which takes residuals of point cloud alignment as a part of ESKF observation model.

### Components

- Point Cloud Registration:
  - Normal Distribution Transform (NDT)
  - Incremental NDT
  - ICP
- Lidar odometry based on NDT alignment
- Loosely coupled Lidar-Inertial Odometry based on Error State Kalman Filter (ESKF)
- Tightly coupled Lidar-Inertial Odometry based on Iterative ESKF

### Point Cloud Registration

Probelm statement: given target and source point cloud, compute pose where source cloud is observed relative to pose where target cloud is observed.

#### NDT

1. Given target point cloud $p_{ti}$, build the grid which discretize space containing the cloud. Compute the mean and covariance of position of points for each grid cells say $\mu_i$, $\Sigma_i$.
2. Given source point cloud $p_{si}$, formulate least squared problem: min $\sum_i \left \lVert R * p_{si} + t - p_{ti} \right \rVert_{\Sigma_i}$
3. Solve least square problem by Gauss-Newton method:
   - Jacobian: $J_i = [ -R \hat{p}*{si}, \ I_3 ] \in \mathbb{R}^{3 \times 6}$
   - Error: $e_i = R  p_{si} + t - p_{ti}$
   - solve iteratively: $J_i^T \Sigma_i ^{-1} J_i \cdot \Delta x = -J_i^{T} \Sigma_i^{-1} e_i$
   - update: $x = x + \Delta x$
4. Solution is the optimized pose: $(R, t)$.
5. C++ parallel algorithm is used to accelerate optimization.

- code

  - [source code](https://github.com/yangfan/lio3d/tree/master/src/matching/NDT.cpp)
  - [unit test](https://github.com/yangfan/lio3d/tree/master/test/ndt_test.cpp): `./test/ndt_test`

    <div class="row justify-content-center">
        <div class="col">
            <div class="w-50 mx-auto" style="background-color: white;">
                <img class="img-fluid" src="{{ '/assets/img/projects/lidar-inertial-odometry/lio3d/ndt_alignment.png' | relative_url }}" alt=""/>
            </div>
        </div>
    </div>
    <div class="caption">
        NDT Alignment, blue: target cloud, green source cloud, red: aligned cloud
    </div>

#### Incremental NDT

Goal: Build target grid incrementally, limit number of voxel cells, accelerate alignment.

1. Update mean and variance incrementally, i.e.,
   - mean: $\mu_{n} = \frac{m * \mu_{h} + n * \mu_{c}}{m + n}$
   - covariance: $\Sigma_{n} = \frac{m *(\Sigma_h + (\mu_h - \mu_n) * (\mu_h - \mu_n)^T) + n * (\Sigma_c + (\mu_{c} + (\mu_c - \mu_n)(\mu_c - \mu_n)^T) )}{m + n}$
2. Limit number of grid cells and delete cells that are not updated for a long time. A LRU cache is implemented for this purpose.
3. To have better numerical stability, the inverse of covariance matrix, i.e., information matrix is computed by SVD.

- code
  - [source code](https://github.com/yangfan/lio3d/tree/master/src/matching/NDT_INC.cpp)

#### ICP 3D

- kd-tree is implemented to find the nearest neighbors.

- point to point

  - for each source point, find the nearest neighbor from target cloud with kd-tree. Denote $p_{si}$ the query point in source cloud, $p_{ti}$ the closest point in target cloud.
  - solve least square problem: min $\sum_i \left \lVert R * p_{si} + t - p_{ti} \right \rVert_2$
  - Gauss-Newton iteration with same Jacobian and residual defined in NDT.
  - After each iteration, find again the nearest neighbor for each source point.

- point to line

  - for each source point, find the nearest k neighbors, and fitting a line from them, i.e., $p = s_0 + \lambda d$.
  - solve least square problem: min $\sum_i \left \lVert \hat{d} (R p_i + t - s_0) \right \rVert_2$. In other words, minimize the distance between query point and the line.
  - Jacobian: $[-\hat{d} R \hat{p}_i, \ \hat{d}] \in 3 \times 6$
  - Go back to the first step until converge

- point to plane

  - for each source point, find the nearest k neighbors and the corresponding fitting plane, i.e., $n^T p_i  + d = 0$
  - solve least square problem: $\sum_i \left \lVert \sum_i n^T (R p_i + t) + d \right \rVert_2$
  - Jacobian: $[-n^T R \hat{p}_i, \ n^T] \in 1\times 6$
  - go back to the first step until converge

- [source code](https://github.com/yangfan/lio3d/tree/master/src/matching/ICP3D.cpp)
- [unit test](https://github.com/yangfan/lio3d/tree/master/test/icp_test.cpp): `./test/icp_test`

### Lidar Odometry

Goal: given pointcloud, estimate global pose at each timestamp.

#### Procedure

1. Set origin of global frame as the pose of first keyframe.
2. At each time stamp, build NDT grid by the source pointcloud.
3. Compute initial guess of current pose assuming the last relative motion remains the same. Solve NDT alignment. The solution is the global pose of current frame.
4. Create keyframe if the motion between current frame and last keyframe is greater than a threshold. Update target cloud of NDT.
5. Compute the relative motion from last to current frame. Store the estimated pose for the next iteration.

#### Code

- NDT LO

  - [source code](https://github.com/yangfan/lio3d/tree/master/src/lio/ndt_lo.cpp)

  - [unit test](https://github.com/yangfan/lio3d/tree/master/test/ndt_lo_test.cpp): `./test/ndt_lo_test`

    <div class="row justify-content-center">
        <div class="col">
            <div class="w-50 mx-auto" style="background-color: white;">
                <img class="img-fluid" src="{{ '/assets/img/projects/lidar-inertial-odometry/lio3d/ndt_lo.gif' | relative_url }}" alt=""/>
            </div>
        </div>
    </div>
    <div class="caption">
        NDT Lidar Odometry
    </div>

- Incremental NDT LO

  - [source code](https://github.com/yangfan/lio3d/tree/master/src/lio/ndt_inc_lo.cpp)

  - [unit test](https://github.com/yangfan/lio3d/tree/master/test/ndt_inc_lo_test.cpp): `./test/ndt_inc_lo_test`

    <div class="row justify-content-center">
        <div class="col">
            <div class="w-50 mx-auto" style="background-color: white;">
                <img class="img-fluid" src="{{ '/assets/img/projects/lidar-inertial-odometry/lio3d/ndt_inc_lo.gif' | relative_url }}" alt=""/>
            </div>
        </div>
    </div>
    <div class="caption">
        Incremental NDT Lidar Odometry
    </div>

    <div class="row justify-content-center">
        <div class="col">
            <div class="w-50 mx-auto" style="background-color: white;">
                <img class="img-fluid" src="{{ '/assets/img/projects/lidar-inertial-odometry/lio3d/ndt_lo.png' | relative_url }}" alt=""/>
            </div>
        </div>
    </div>
    <div class="caption">
        3D Point Cloud Map for ULHK dataset
    </div>

### ESKF Lidar-Inertial Odometry

Goal: Estimate the state of robot at each timestamp by loosely fusing lidar scan and IMU data.

#### Procedure

1. Initialize IMU by integrating gyroscope and accelerometer readings while the robot keeps static. Compute bias and covariance of gyroscope and accelerometer, gravity, which are used in ESKF.
2. Predict nominal state by IMU integration. Nominal state includes: position, velocity, rotation, bias of gyroscope, bias of accelerometer, gravity (dim: 18 by 1). Error state remains zero. Covariance matrix also gets updated (dim: 18 by 18).
3. Undistort lidar scan influenced by motion. Due the motion during the lidar scanning, the pose of lidar at which the lidar point gets observed is different. The goal of undistortion is to transform the lidar points to the same observation pose, i.e., motion compensation. The undistortion computes the global pose of lidar sensor when scan point is observed ($T_{WLi}$) by interoplating imu poses, and then transform scan point to the pose of last imu reading ($p_{Le}$). More precisely $p_{Le} = T_{LI} * T_{IeW} * T_{WIi} * T_{IL} * p_{Li}$
4. Correct state of ESKF from observation. The observation is the difference between global pose estimated from NDT alignment and the predicted nominal pose by ESKF. ESKF state is updated from Kalman gain and innovation. Then the error state is reset to 0 and covariance get updated.
5. Create keyframe if the relative motion to last keyframe is over the threshold. The target cloud of NDT is also updated.

#### Code

- [source code](https://github.com/yangfan/lio3d/tree/master/src/eskf/LioEskf.cpp)
- [unit test](https://github.com/yangfan/lio3d/tree/master/test/eskf_test.cpp): `./test/eskf_test`

    <div class="row justify-content-center">
        <div class="col">
            <div class="w-50 mx-auto" style="background-color: white;">
                <img class="img-fluid" src="{{ '/assets/img/projects/lidar-inertial-odometry/lio3d/eskf.gif' | relative_url }}" alt=""/>
            </div>
        </div>
    </div>
    <div class="caption">
        LIO based on Error State Kalman Filter
    </div>

    <div class="row justify-content-center">
        <div class="col">
            <div class="w-50 mx-auto" style="background-color: white;">
                <img class="img-fluid" src="{{ '/assets/img/projects/lidar-inertial-odometry/lio3d/NCLT.png' | relative_url }}" alt=""/>
            </div>
        </div>
    </div>
    <div class="caption">
        3D Point Cloud Map of NCLT dataset
    </div>

### Iterative ESKF Lidar-Inertial Odometry

Goal: Estimate the state of robot at each timestamp by tightly fusing lidar scan and IMU data. In loosely LIO, the NDT alignment may fail in degenerate environment, which leads to divergence of eskf estimation. In tightly couple LIO, the NDT algnment residual is used in the observation model of iterative ESKF. NDT alignment is constrained by IMU readings.

#### Procedure

1. Initialize IMU by integrating gyroscope and accelerometer readings while the robot keeps static. Compute bias and covariance of gyroscope and accelerometer, gravity, which are used in ESKF.
2. Predict nominal state by IMU integration. Nominal state includes: position, velocity, rotation, bias of gyroscope, bias of accelerometer, gravity (dim: 18 by 1). Error state remains zero. Covariance matrix also gets updated (dim: 18 by 18).
3. Undistort lidar scan influenced by motion. The undistortion computes the global pose of lidar sensor when scan point is observed ($T_{WLi}$) by interoplating imu poses, and then transform scan point to the last imu reading ($p_{Le}$). More precisely $p_{Le} = T_{LI} * T_{IeW} * T_{WIi} * T_{IL} * p_{Li}$
4. Correct state of iESKF from observation iteratively. At each iteration, compute the residual and Jacobian of NDT alignment based on current state estimation. The Kalman gain and innovation are computed from them. The nominal state and covariance are then updated. Say at j iteration:
   - Let $H_j = \sum_i J_i^T \Sigma_i^{-1}J_i \ \in \mathbb{R}^{18 \times 18}$
   - Let $b_j = \sum_i -J_i^T \Sigma_i^{-1} r_i \ \in \mathbb{R}^{18}$, where $r_i = R * p_{si} + t - p_{ti}$
   - error state $\delta x_{j + 1} = (P_j^{-1} + H_j)^{-1} b_j$
5. Create keyframe if the relative motion to last keyframe is over the threshold. The target cloud of NDT is incrementally updated.

#### Code

- [source code](https://github.com/yangfan/lio3d/tree/master/src/eskf/LioIeskf.cpp)
- [unit test](https://github.com/yangfan/lio3d/tree/master/test/ieskf_test.cpp): `./test/ieskf_test`

    <div class="row justify-content-center">
        <div class="col">
            <div class="w-50 mx-auto" style="background-color: white;">
                <img class="img-fluid" src="{{ '/assets/img/projects/lidar-inertial-odometry/lio3d/ieskf.gif' | relative_url }}" alt=""/>
            </div>
        </div>
    </div>
    <div class="caption">
        LIO based on Iterative Error State Kalman Filter
    </div>

### Dataset

1. ULHK:

- no time info
- time unit: microseconds
- issue: the scan time is estimated by angular velocity which is not accurate enough. Therefore the start time of a scan is tens of (around 20 ) ms later than the end time of previous scan.
- cloud topic name: /velodyne_points_0
- imu topic name: /imu/data

2. NCLT:

- has time info
- time unit: seconds
- cloud topic name: points_raw
- imu topic name: imu_raw

3. [ICP pointcloud dataset](https://lgg.epfl.ch/statues_dataset.php)

### Repository

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="yangfan/lio3d" %}
</div>
