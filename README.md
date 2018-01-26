# Spatio-Temporal Voxel Layer

This is a drop in replacement for the voxel_grid voxel representation of the environment. This package does a number of things to improve on the voxel grid package and extend the capabilities offered to the users, under a LGPL v2.1 license. Developed and maintained by [Steven Macenski](https://www.linkedin.com/in/steven-macenski-41a985101/) at [Simbe Robotics](http://www.simberobotics.com/).

This package sits on top of [OpenVDB](http://www.openvdb.org/), an open-source C++ library built by Dreamworks "comprising a novel hierarchical data structure and a suite of tools for the efficient storage and manipulation of sparse volumetric data discretized on three-dimensional grids. It is developed and maintained by DreamWorks Animation for use in volumetric applications typically encountered in feature film production."

Leveraging OpenVDB, we have the ability to efficiently maintain a 3 dimensional voxel-representative world space. We wrap this with ROS tools and interfaces to the [navigation stack](http://wiki.ros.org/navigation) to allow for use of this layer in standard ROS configurations. It is certainly possible to utilize this package without ROS/Navigation and I invite other competing methodologies to develop here and create interfaces. 

An example video can be seen [here](https://www.youtube.com/watch?v=8YIFPiI1vrg&feature=youtu.be).

## **Spatio**-
The Spatio in this package is the representation of the environment in a configurable `voxel_size` voxel grid stored and searched by OpenVDB. 

## -**Temporal**
The Temporal in this package is the novel concept of `voxel_decay` whereas we have configurable functions that expire voxels and their occupation over time. Infrasture was created to store times in each voxel after which the voxel will disappear from the map. This is combined with checking inclusion of voxels in current measurement frustums to accelerate the decay of those voxels that do not have measurements but should if still in the scene and remain marked. This is done rather than simply clearing them naively or via costly raytracing. The time it takes to clear depends on the configured functions and acceleration factors.

Future extensions will also to query a static map and determine which connected components belong to the map, not in the map, or moving. Each of these three classes of blobs will have configurable models to control the time they persist, and if these things are reported to the user.    

## Local Costmap
This package utilizes all of the information coming in from the robot before the decay time for the local costmap. Rather than having a defined, discrete spatial barrier for the local planner to operate in, it instead relies on the user configuration of the layer to have a short decay time of voxels (1-30 seconds) so that you only plan in relavent space. This was a conscious design requirement since frequently the local planner should operate with more information than other times when the speed is greater or slower. This natively implements dynamic costmap scaling for speed.

It is the user's responsibility to chose a decay time that makes sense for your robot's local planner. 5-15 seconds I have found to be nominally good for most open-sourced local planner plugins.

## Global Costmap
Similar to the local costmap, the amount of information you want to store due to entropy in your scenes depend on your use-case. It is certainly possible to **not** decay the voxels in the global map at all. However, in practical application, I find a time 1-10 minutes to be a good balance due to things moving in the scene (i.e. store, warehouse, construction zone, office, etc).

## Installation
Required dependencies ROS Kinetic, navigation, OpenVDB, TBB.

### ROS

[See install instructions here.](http://wiki.ros.org/kinetic/Installation)

### Navigation

`sudo apt-get install ros-kinetic-navigation`

### OpenVDB

`sudo apt-get install libopenvdb3.1 libopenvdb-doc libopenvdb-dev libopenvdb-tools`

### TBB

`sudo apt-get install libtbb-dev libtbb2 libtbb-doc`

## Configuration and Running

### costmap_common_params.yaml

An example fully-described configuration is shown below.

```

rgbd_obstacle_layer:
  enabled:               true
  voxel_decay:           20 #seconds
  voxel_decay_static:    1.
  decay_model:           0
  voxel_size:            0.05
  track_unknown_space:   true
  observation_persistence: 0.0
  max_obstacle_height:   2.0
  unknown_threshold:     15
  mark_threshold:        0
  combination_method:    1
  obstacle_range:        3.0
  origin_z:              0.0
  publish_voxel_map:     true
  observation_sources: rgbd1_clear rgbd1_mark
  rgbd1_mark:
    data_type: PointCloud2
    topic: camera1/depth/points
    marking: true
    clearing: false
    min_obstacle_height: 0.3
    max_obstacle_height: 2.0
  rgbd1_clear:
    data_type: PointCloud2
    topic: camera1/depth/points
    marking: false
    clearing: true
    min_z: 0.1
    max_z: 7.0
    vertical_fov_angle: 0.7
    horizontal_fov_angle: 1.04
    decay_acceleration: 1.
```

### local/global_costmap_params.yaml

Add this plugin to your costmap params file. 

`- {name: rgbd_obstacle_layer,     type: "spatio_temporal_voxel_layer/SpatioTemporalVoxelLayer"}`


### Running

`roslaunch [navigation_pkg] move_base.launch`

**This is a highly experimental package under heavy development.**