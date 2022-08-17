# planning/global_velocity_planner

/src/global_velocity_planner_node.cpp:81:

```
m_map_client = this->create_client<HADMapService>("HAD_Map_Service");
```

/test/test_global_velocity_planner_node.cpp:119:

```
m_fake_node->create_service<HADMapService>(...
```

 

```
<depend>rclcpp</depend>
<depend>rclcpp_action</depend>
<depend>rclcpp_components</depend>

<depend>autoware_auto_common</depend>
<depend>autoware_auto_control_msgs</depend>
<depend>autoware_auto_planning_msgs</depend>
<depend>autoware_auto_vehicle_msgs</depend>
<depend>autoware_auto_geometry</depend>
<depend>had_map_utils</depend>
<depend>lanelet2_core</depend>
<depend>lanelet2_traffic_rules</depend>
<depend>autoware_auto_tf2</depend>
<depend>motion_common</depend>
<depend>geometry_msgs</depend>
<depend>visualization_msgs</depend>
<depend>tf2</depend>
<depend>tf2_ros</depend>

<test_depend>ament_cmake_gtest</test_depend>
<test_depend>ament_index_python</test_depend>
<test_depend>ament_lint_auto</test_depend>
<test_depend>ament_lint_common</test_depend>
<test_depend>autoware_testing</test_depend>
<test_depend>ros_testing</test_depend>
```





# planning/object_collision_estimator_nodes



