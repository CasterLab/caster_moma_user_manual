# 开发说明

本文对开发中常用到的接口进行了说明

[TOC]

## ROS参考资料

1. ROS Wiki Tutorials：[http://wiki.ros.org/ROS/Tutorials](http://wiki.ros.org/ROS/Tutorials)
2. 古月居：[https://www.guyuehome.com](https://www.guyuehome.com)

## 源码库

1. Caster：[https://github.com/CasterLab/caster_robot](https://github.com/CasterLab/caster_robot)
2. CasterMoma：[https://github.com/CasterLab/caster_moma_robot](https://github.com/CasterLab/caster_moma_robot)
3. kinova_ros（用于Kinova Gen2系列机械臂）：[https://github.com/I-Quotient-Robotics/kinova_ros](https://github.com/I-Quotient-Robotics/kinova_ros)
4. ros_kortex（用于Kinova Gen3系列机械臂）：[https://github.com/Kinovarobotics/ros_kortex](https://github.com/Kinovarobotics/ros_kortex)

## 传感器信息

`hongfu_bms_status_node/hongfu_bms` ([hongfu_bms_msg/HongfuStatus](https://github.com/I-Quotient-Robotics/hongfu_bms/blob/master/hongfu_bms_msg/msg/HongfuStatus.msg))

电池BMS信息，包含电池状态，剩余电量，以及电池的充放电信息

`imu_data` ([sensor_msgs/Imu](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Imu.html))

IMU的信息，包含Caster的三轴加速度/角加速度，以及姿态信息

`joy` ([sensor_msgs/Joy](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Joy.html))

手柄按键信息

`odom` ([nav_msgs/Odometry](http://docs.ros.org/melodic/api/nav_msgs/html/msg/Odometry.html))

里程计信息

`scan` ([sensor_msgs/LaserScan](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/LaserScan.html))

底层激光雷达数据（Hokuyo UST-20LX）

`scan_2` ([sensor_msgs/LaserScan](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/LaserScan.html))

中层激光雷达数据（Rplidar）

`dauxi_ks106_node/ultrasonic_front_left` ([sensor_msgs/Range](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Range.html))

超声波数据（左前）

`dauxi_ks106_node/ultrasonic_front_right` ([sensor_msgs/Range](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Range.html))

超声波数据（右前）

`dauxi_ks106_node/ultrasonic_rear_left` ([sensor_msgs/Range](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Range.html))

超声波数据（左后）

`dauxi_ks106_node/ultrasonic_rear_right` ([sensor_msgs/Range](http://docs.ros.org/melodic/api/sensor_msgs/html/msg/Range.html))

超声波数据（右后）

##  运动控制

Caster使用[yocs_cmd_vel_mux](http://wiki.ros.org/yocs_cmd_vel_mux)对多路控制信号进行优先级判断，具体参数请查看[caster_control/config/cmd_vel_mux.yaml](https://github.com/CasterLab/caster_robot/blob/master/caster_control/config/cmd_vel_mux.yaml)。用户可以根据控制指令的分类，通过对应的Topic，控制Caster的运动。用户也可以根据情况增加更多的输入通路。

### 多路控制器状态

`yocs_cmd_vel_mux/active` ([std_msgs/String](http://docs.ros.org/api/std_msgs/html/msg/String.html))

当前有效的控制信号

### 输入信号

`yocs_cmd_vel_mux/input/safety_cmd` ([geometry_msgs/Twist](http://docs.ros.org/api/geometry_msgs/html/msg/Twist.html))

优先级：5；安全控制信号，建议用户将安全模块的控制信号接入此Topic

`yocs_cmd_vel_mux/input/joy_cmd` ([geometry_msgs/Twist](http://docs.ros.org/api/geometry_msgs/html/msg/Twist.html))

优先级：4；手柄控制信号，默认状态下，手柄的控制信号通过此Topic控制Caster的运动，仅次于安全控制信号的优先级，可以保证用户能够随时通过手柄抢断其他路控制，确保Caster的安全
`yocs_cmd_vel_mux/input/navigation_cmd` ([geometry_msgs/Twist](http://docs.ros.org/api/geometry_msgs/html/msg/Twist.html))

优先级：3; 导航控制信号，默认状态下，move_base的控制信号会通过此Topic控制Caster的运动

### 输出信号

`yocs_cmd_vel_mux/output/cmd_vel`([geometry_msgs/Twist](http://docs.ros.org/api/geometry_msgs/html/msg/Twist.html))

实际控制Caster的信号，此信号根据优先级判断应该采用哪一路的输入信号。默认状态下，此Topic已经被remap到cmd_vel。

**注意：不建议直接通过cmd_vel来控制Caster，使用yocs_cmd_vel_mux/input/来代替，以确保使用者可以随时通过手柄，夺取Caster的控制优先权，确保Caster的安全**！！

## 云台控制

`diagnostics` ([diagnostic_msgs/DiagnosticArray](http://docs.ros.org/melodic/api/diagnostic_msgs/html/msg/DiagnosticArray.html))

用于控制Kinect2云台

数值范围：速度0~40，角度范围：-60~60（水平，单位：度），-60~60（俯仰，单位：度）

## 升降控制

caster_robot/body_controller/goal ([std_msgs/Float64](http://docs.ros.org/api/std_msgs/html/msg/Float64.html))

控制躯体的升降，数值范围0~0.4，单位M

## 诊断信息

`diagnostics` ([diagnostic_msgs/DiagnosticArray](http://docs.ros.org/melodic/api/diagnostic_msgs/html/msg/DiagnosticArray.html))

设备诊断信息，包含驱动器和电池的相关状态，用户可以通过此Topic了解Caster的运行状态

## 定位导航信息

关于定位导航的信息，具体使用方式，请参阅[move_base](http://wiki.ros.org/move_base)以及[amcl](http://wiki.ros.org/amcl)

`map` ([nav_msgs/OccupancyGrid](http://docs.ros.org/melodic/api/nav_msgs/html/msg/OccupancyGrid.html))

地图数据

`move_base/global_costmap/costmap` ([nav_msgs/OccupancyGrid](http://docs.ros.org/melodic/api/nav_msgs/html/msg/OccupancyGrid.html))

全局花费地图

`move_base/local_costmap/costmap` ([nav_msgs/OccupancyGrid](http://docs.ros.org/melodic/api/nav_msgs/html/msg/OccupancyGrid.html))

局部路径花费地图

`amcl_pose` ([geometry_msgs/PoseWithCovarianceStamped](http://docs.ros.org/melodic/api/geometry_msgs/html/msg/PoseWithCovarianceStamped.html))

定位数据

`move_base`(actionlib api)

导航指令，使用actionlib进行控制，具体使用方式请参考[http://wiki.ros.org/move_base](http://wiki.ros.org/move_base)

## 自动充电

`dock_pose` ([geometry_msgs/PoseStamped](http://docs.ros.org/melodic/api/geometry_msgs/html/msg/PoseStamped.html))

充电桩位置

**注意：此坐标是在laser_link坐标系下，用户需要根据情况，使用TF转换到其它的坐标系下使用**

`dock_action` (actionlib api)

使用actionlib进行控制，具体使用方式请参考[http://wiki.ros.org/move_base](http://wiki.ros.org/move_base)

自动充电指令，具体使用方式，请参考[自动充电原理与配置](03-auto_charge_description.md)