# Installation and Run Instructions

This guide contains information about required dependencies and how to run the MultiDLO ROS package.

## Minimum Requirements

Installation and execution of MultiDLO was verified with the below dependencies on an Ubuntu 20.04 system with ROS Noetic.

We provide Docker files for compatibility with other system configurations. Refer to the docker [README.md](https://github.com/RMDLO/multidlo/blob/master/docker/README.md) for more information on using docker, and see the docker [requirements.txt](https://github.com/RMDLO/multidlo/blob/master/docker/requirements.txt) file for a list of the tested versions of MultiDLO package dependencies.

## Other Requirements

We used an Intel RealSense d435 camera in all of the experiments performed in our paper. We used the [librealsense](https://github.com/IntelRealSense/librealsense) and [realsense-ros](https://github.com/IntelRealSense/realsense-ros/tree/ros1-legacy) packages for testing with the RealSense D435 camera and for obtaining the [camera configuration file](https://github.com/RMDLO/multidlo/blob/master/config/preset_decimation_4.0_depth_step_100.json).

## Installation

The repository is organized into the following directories:

```
  ├── src/        # contains MultiDLO ROS node and initialization code and data
  ├── launch/     # contains ROS launch files for the camera, tracker, and RViz
  ├── rviz/       # contains Rviz configurations for visualization
  ├── config/     # contains camera configurations
```

First, [create a ROS workspace](http://wiki.ros.org/catkin/Tutorials/create_a_workspace). Next, `cd YOUR_TRACKING_ROS_WORKSPACE/src`. Clone the MultiDLO repository into this workspace and build the package:

```bash
git clone https://github.com/RMDLO/multidlo.git
catkin build multidlo
source ../devel/setup.bash
```

All configurable parameters for the MultiDLO algorithm are in [`launch/multidlo.launch`](https://github.com/RMDLO/multidlo/blob/master/launch/multidlo.launch). Rebuilding the package is not required for any parameter modifications to take effect. However, `catkin build` is required after modifying any C++ files. Remember that `source <YOUR_TRACKING_ROS_WS>/devel/setup.bash` is required in every terminal running MultiDLO ROS nodes.

## Usage

To run MultiDLO, the three ROS topic names below should be modified in `launch/multidlo.launch` to match the ROS topic names published by the user's RGB-D camera:
* `camera_info_topic`: a CameraInfo topic that contains the camera's projection matrix
* `rgb_topic`: a RGB image stream topic
* `depth_topic`: a depth image stream topic

**Note: MultiDLO assumes the RGB image and its corresponding depth image are ALIGNED and SYNCHRONIZED. This means depth_image(i, j) should contain the depth value of pixel rgb_image(i, j) and the two images should be published with the same ROS timestamp.**

Other useful parameters:
* `num_of_nodes`: the number of nodes initialized for the DLO
* `result_frame_id`: the tf frame the tracking results (point cloud and marker array) will be published to
* `visualize_initialization_process`: if set to `true`, OpenCV windows will appear to visualize the results of each step in initialization. This is helpful for debugging in the event of initialization failures.

Once all parameters in `launch/multidlo.launch` are set to proper values, run MultiDLO with the following steps:
1. Launch the RGB-D camera node
2. Launch RViz to visualize all published topics: 
```bash
roslaunch multidlo visualize_output.launch
```
3. Launch the MultiDLO node to publish tracking results:
```bash
roslaunch multidlo multidlo.launch
```

The MultiDLO node outputs the following:
* `/multidlo/results_marker`: the tracking result with nodes visualized with spheres and edges visualized with cylinders in MarkerArray format, 
* `/multidlo/results_pc`: the tracking results in PointCloud2 format
* `/multidlo/results_img`: the tracking results projected onto the received input RGB image in RGB Image format


## Run MultiDLO with Recorded ROS Bag Data:
1. Download one of the provided rosbag experiment `.bag` files [here](https://doi.org/10.13012/B2IDB-6432640_V1). Note: the file sizes are large! Please first make sure there is enough local storage space on your machine.
2. Open a new terminal and run 
```bash
roslaunch multidlo visualize_output.launch bag:=True
```
3. Start the tracking node:
```bash
roslaunch multidlo multidlo.launch
```
4. Open a third ternimal and run the below command to replay the `.bag` file and publish its topics:
```bash
rosbag play --clock /path/to/filename.bag
```

## Data:

The ROS bag files used in our paper and the supplementary video can be found [here](https://doi.org/10.13012/B2IDB-6432640_V1). The data include five scenarios used for the qualitative evaluation shared in the original MultiDLO manuscript.