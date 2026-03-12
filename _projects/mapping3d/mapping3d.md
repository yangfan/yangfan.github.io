---
layout: page
title: 3D Point Cloud Mapping
description: |
  This project designs and implements a complete pipline of the 3D point cloud mapping system. The system is comprised of two parts: online frontend module and offline backend module. The frontend is responsible for collecting sensor data (3d lidar scan, imu data, gnss data), creating keyframes and saving point cloud data. The backend is then executed to reduce accumulated error and improve the consistency of the map. The main job of the backend is to detect and evaluate the loop closure, eliminate sensor data outliers and optimize the keyframe poses and partition the map.

img: /assets/img/projects/mapping3d/local_view.gif
importance: 1
category: 3D Mapping
github: https://github.com/yangfan/offline_mapping
---

This project designs and implements a complete pipline of the 3D point cloud mapping system. The system is comprised of two parts: online frontend module and offline backend module. The frontend is responsible for collecting sensor data (3d lidar scan, imu data, gnss data), creating keyframes and saving point cloud data. The lidar-inertial odometry created in previous project is used in the frontend. The backend is then executed to reduce accumulated error and improve the consistency of the map. The main job of the backend is to detect and evaluate the loop closure, eliminate sensor data outliers and optimize the keyframe poses and partition the map. The map is partitioned into a number of smaller submaps to facilitate the dynamic map loading in localization module which saves memory usage and accelerates localization.

### Components

1. Front End
2. Loop Closure
3. Optimization
4. Map Partition

- result: local view

  <div class="row justify-content-center">
      <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/mapping3d/local_view.png' | relative_url }}" alt=""/>
        </div>
      </div>
  </div>
  <div class="caption">
    3D Point Cloud Map 
  </div>

  <div class="row justify-content-center">
      <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/mapping3d/local_view.gif' | relative_url }}" alt=""/>
        </div>
      </div>
  </div>
  <div class="caption">
    3D Point Cloud Map (animated) 
  </div>

### Front End

Goal: Use IESKF Lidar IMU Odometry to create Keyframes with timestamp lio pose, scan, gnss pose (not used in lio), etc.

#### Procedure

1.  GNSS data processing:
    1. ros message `sensor_msgs::NavSatFix` to `GNSS`
    2. compute UTM coordinates (x, y, z) from lon, lat, alt
    3. use the position of first valid GNSS data (status >= STATUS_FIX) as origin
    4. store GNSS data with timestamp in array
2.  Run ieskf lidar imu odometry
    1. add point cloud data and imu data to lio
    2. if new keyframe was created, (a) store pointcloud as pcd file , (b) match pointcloud data with gnss data, store gnss position/pose in keyframe if it's valid
    3. store all keyframe data in txt file

#### Code

- [source code](https://github.com/yangfan/offline_mapping/tree/master/src/mapping/FrontEnd.cpp)

- [executable](https://github.com/yangfan/offline_mapping/tree/master/src/app/main_frontend.cpp)

  - run frontend: `./bin/main_frontend`

- [create global map](https://github.com/yangfan/offline_mapping/tree/master/src/app/merge_kfs.cpp)

  1.  merge keyframes: `./bin/merge_kfs --pose_type=lio`
  2.  show map: `pcl_viewer ./data/output/keyframes/pcd/map.pcd`

  <div class="row justify-content-center">
      <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/mapping3d/frontend.png' | relative_url }}" alt=""/>
        </div>
      </div>
  </div>
  <div class="caption">
    Frontend Output Map
  </div>

### Loop Closure

Goal: find keyframes that are spatially close but created at different times

#### Procedure

1. Find Candidate
   1. go through all keyframe pairs
   2. skip keyframes that were temporally close
   3. skip keyframes that are close to previous detected loop keyframe pairs
   4. candidate detected if the distance is small enough
2. evaluate candidates: scan (query keyframe) to map (submap of target keyframe)
   - build submap of keyframes temporally close to target keframe
   - get pointcloud of query keyframe from pcd file
   - match scan to submap with ascending resolutions
   - compute score of NDT alignment
3. remove outliers with low score

#### Code

- [source code](https://github.com/yangfan/offline_mapping/tree/master/src/mapping/LoopClosure.cpp)

- [executable](https://github.com/yangfan/offline_mapping/tree/master/src/app/main_loopclosure.cpp)

  - run loop closure detection: `./bin/main_loop`

- pose graph structure:

  <div class="row justify-content-center">
      <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/mapping3d/pose_graph.png' | relative_url }}" alt=""/>
        </div>
      </div>
  </div>
  <div class="caption">
    Pose Graph Structure
  </div>

### Optimization

Goal: Refine keyframe pose by pose graph optimization

#### Procedure

1.  ICP alignment between gnss and lio trajectory
2.  Build Pose graph
    - create optimizer
    - create vertices: keyframes
    - create edges: gnss position, relative motion between two keyframes based on lio, loop closure constraint between two keyframes that are physcially close.
3.  Solve optimization
4.  Remove outliers (disable outlier edges), i.e., chi2 > robust kernel delta, optimization again.
5.  Solve optimization again
6.  save results: asign optimization poses to keyframes, save keyframes info to txt

#### Code

- [source code](https://github.com/yangfan/offline_mapping/tree/master/src/mapping/Optimization.cpp)

- [executable](https://github.com/yangfan/offline_mapping/tree/master/src/app/main_optimization.cpp)

- optimization 1: `./bin/main_opt --opt_stage=1`

  <div class="row justify-content-center">
      <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/mapping3d/opt1.png' | relative_url }}" alt=""/>
        </div>
      </div>
  </div>
  <div class="caption">
    3D Point Cloud Map after first stage optimization
  </div>

- optimization 2: `./bin/main_opt --opt_stage=2`

  <div class="row justify-content-center">
      <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/mapping3d/opt2.png' | relative_url }}" alt=""/>
        </div>
      </div>
  </div>
  <div class="caption">
    3D Point Cloud Map after second stage optimization
  </div>

- show trajectory: `python3 ./scripts/trajectory.py ./data/output/keyframes/kf_info.txt`

    <div class="row justify-content-sm-center">
        <div class="col-sm-6 mt-3 mt-md-0">
            <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/mapping3d/traj2d.png'| relative_url }}" alt="" title="2d trajectory"/>
        </div>
        <div class="col-sm-6 mt-3 mt-md-0">
            <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/projects/mapping3d/traj3d.png'| relative_url }}" alt="" title="3d trajectory"/>
        </div>
    </div>
    <div class="caption">
        Mapping Trajectory
    </div>
    <br/>

### Map Partition

Goal: Split map into a grid of submaps

#### Procedure

1. Iterate each keyframe
   1. iterate each lidar point: compute submap id, i.e. `id = int ((pos - origin.pos) * resolution)`
   2. create new submap if it hasn't been created yet, otherwise insert to the exsiting submap
2. Save each submap including pointcloud and submap id.

#### Code

- [source code](https://github.com/yangfan/offline_mapping/tree/master/src/app/partition_map.cpp)

- command: `./bin/partition --info_file=kf_info.txt`

  <div class="row justify-content-center">
      <div class="col">
        <div class="w-50 mx-auto" style="background-color: white;">
            <img class="img-fluid" src="{{ '/assets/img/projects/mapping3d/partitions.png' | relative_url }}" alt=""/>
        </div>
      </div>
  </div>
  <div class="caption">
    Map Partition
  </div>

### Commands

#### Frontend

1. create keyframes scan (pcd files) and keyframes info (txt file): `./bin/main_frontend`
2. (optional) merge all keyframe scan based on lio pose: `./bin/merge_kfs --info_file=kf_info.txt --pose_type=lio`
3. (optional) visualize keyframes scan: `pcl_viewer ./data/output/keyframes/pcd/map.pcd`
4. generate files: `kf_info.txt`, `id.pcd` pointcloud.

#### Optimization stage 1

1. optimize stage 1 with gnss edges and lio edges: `./bin/main_opt --opt_stage=1`
2. (optional) visualize trajectory of lio pose and optimized pose: `python3 ./script/trajectory.py  ./data/output/keyframes/kf_info.txt`
3. (optional) merge all keyframe scan based on lio pose: `./bin/merge_kfs --info_file=kf_info.txt --pose_type=opt1`
4. (optional) visualize keyframes scan: `pcl_viewer ./data/output/keyframes/pcd/map.pcd`
5. (optional) visualize pose graph structure `g2o_viewer ./data/output/keyframes/pose_graph.g2o`
6. update opt1 pose in `kf_info.txt`

#### Loop closure

1. detect loop closure: `./bin/main_loop `
2. generate loop closure file: `loop.txt`

#### Optimization stage 2

1. optimize stage 1 with gnss edges and lio edges: `./bin/main_opt --opt_stage=2`
2. (optional) visualize trajectory of lio pose and optimized pose: `python3 ./script/trajectory.py  ./data/output/keyframes/kf_info.txt`
3. (optional) merge all keyframe scan based on lio pose: `./bin/merge_kfs --info_file=kf_info.txt --pose_type=opt2`
4. (optional) visualize keyframes scan: `pcl_viewer ./data/output/keyframes/pcd/map.pcd`
5. (optional) visualize pose graph structure `g2o_viewer ./data/output/keyframes/pose_graph.g2o`
6. update opt2 pose in `kf_info.txt`

## Localization

see more details in [repo](https://github.com/yangfan/localization).

### Components

1. Imu lidar data Sync
2. eskf
3. Initialization
4. map management
5. NDT alignment

### Initialization

Goal: get IMU initial bias, gravity, covariance of accelerometer and gyroscope noise, initial pose

#### Procedure

1. initialize IMU
2. initialize pose by GNSS data (eskf state)
   - choose submaps based on GNSS position
   - create candidates based on GNSS position and variouse rotations ranging from 0 to 360 deg
   - align current pointcloud with submaps using ascending levels of NDT
   - valid initial pose should have matching score greater than the threshold
3. set eskf initial state, i.e., position, orientation, velocity (should be zero)

### Map Management

Goal: load and unload submaps based on current position

#### Procedure

1. Load submap info, i.e., Id number, from txt file
2. Compute submap id from current position, and load it if it hasn't been loaded yet.
3. Unload map that are far away from current position (determined by submap id)
4. Reset NDT target if map has been modified (load or unload).

### Repository

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.liquid repository="yangfan/offline_mapping" %}
</div>
