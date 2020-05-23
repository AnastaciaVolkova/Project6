# Udacity Project6 "Home Service Robot"

Course Project from Robotics Software Engineer - Path planning and navigation.

localization system for a robot should be installed:
sudo apt-get install ros-melodic-navigation

## Description
### Scripts
- test_slam.sh
- test_navigation.sh
- pick_objects.sh
- add_marker.sh
- home_service.sh

#### <a name="test_slam"></a>test_slam.sh
Script does the following:

- Launches _world.launch_ (from package _my_robot_). <p>With the help of this launch-file an environment starts, which I create with gazebo. This is a model of my apartment (_wohnung.world_). Moreover my robot is loaded. I create the robot in  Unified Robotic Description Format (URDF) (_my_robot.xacro_). Nodes, which publish state of robot and joint states are also launched. Finally rviz, a 3D visualization tool for ROS, starts.</p>
- Launches _slam_gmapping_pr2.launch_ (from package _slam_gmapping_). <p> With the help of launch configuration file node _slam_gmapping_ is run. The node uses OpenSlam and provides laser-based SLAM. With usage of _slam_gmapping_, I  create a 2-D occupancy grid map. </p>
- Launches _teleop_twist_keyboard.py_ (from package  _teleop_twist_keyboard_). <p> With this node I drive a robot in my environment, in order to gather information about obstacles (mostly walls) in my apartment. </p>

**This script is used to create an environment map, which I will use further in navigation.**  I create the bitmap representation of map with the help of _map_server_.
The information, which was gathered by _slam_gmapping_, is provided by topic _map (nav_msgs/OccupancyGrid)_ to _map_server_. Node map_server creates a bitmap representation and yaml file, which describes parameters of map.
Map (wohnung.pgm and wohnung.yaml) is saved in ./my_robot/map/

#### test_navigation.sh
Script does the following:

- Launches _world.launch_ (from package _my_robot_). <p>See description from test_slam.sh</p>
- Launches _amcl.launch_. <p>The file starts _map_server_ node, _amcl_ from package _navigation_  node, _move_base_ from _navigation_ package node. Node _amcl_ realizes probabilistic localization of a robot in 2D. To track the pose of a robot against a known map, it uses Adaptive (or KLD-sampling) Monte Carlo algorithm. _move_base_ is a node which controls the robot movements, so that it can reaches the goal (so the goal should be provided). It uses components global_planner, global_costmap, local_planner, local_costmap, recovery_behaviours for creating paths.</p>
- Runs _map_server_ node, which provides a map to _move_base_.

**The script is used to manually set goal and navigate to it.**

#### pick_objects.sh
Script does the same as test_navigation.sh. In addition, it runs node _pick_objects_node_ from package _pick_objects_ with command line argument 0 (argument 0 means don't use _add_markers_node_). 
Node _pick_objects_node_ constructs an object of SimpleActionClient, with help of which the node can communicate with robot's action. To this object the node sends a goal. After reaching the first goal, the node waits for 5 seconds. Afterwords, the node sends the second goal to SimpleActionClient object and waits for the robot reaches the goal.

#### add_marker.sh
Script does the same as _test_navigation.sh_. In addition, I  run node _add_markers_node_ from package _add_markers_ with command line _-9.0 -5.0 -4.0 3.0_. Through command line I pass positions of two goals (x1, y1, x2, y2). The node analyzes command line arguments, set marker properties and publishes topic _visualization_marker_  to _rviz_.

#### home_service.sh
Script does the same as _test_navigation.sh_. In addition, I run the nodes, which I create: _pick_objects_node_ and _add_markers_node_.  They are started without arguments. It means that they communicate with each other. Node _pick_objects_node_ sends two navigation goals to _move_base_ node via SimpleActionClient. After the first goal is sent to _move_base_ node, node _pick_objects_node_ sends request to _add_markers_node_ node. The request asks _add_markers_node_ to show the marker on the place of first goal. Node _pick_objects_node_ waits until the robot reaches the first goal. Afterwards _pick_objects_node_ sends request to delete marker (node _add_markers_node_ handles this request) at the place of first node. Node _pick_objects_node_ makes the robot to wait 5 seconds. Afterwards the second goal is sent to _move_base_ by node _pick_objects_node_. When second goal is reached, _pick_objects_node_ sends request to _add_markers_node_ to show marker at the place of second goal.
### Significant nodes
#### slam_gmapping
This node implements OpenSlam's Gmapping with usage of Rao-Blackwellized particle filter. I use the node to build the map, with help of laser and pose data of the robot (the node subscribes to _tf_ and _scan_ topics). The 2-D occupancy grid map I retrieve with help of _map_server_ node and _map_ topic.
#### amcl
_amcl_ node provides probabilistic localization in 2D. To track the robot's pose inside known map, which I get with a help of _gmapping_ node, the _amcl_ node implements the adaptive (or KLD-sampling) Monte Carlo localization approach. Inputs for the node are a laser-based map, laser scans, and transform messages. Output is pose estimation. 
