# 快速启动手册(外部PC控制)

## 前期准备

1. 参考《CasterMoma用户手册》，启动机器人

2. 参考《CasterMoma用户手册》的接口图，连接HDMI和键鼠

3. 使用如下指令，获取机器人的名称

   ```bash
   # 获取机器人名称
   hostname
   
   #返回值举例
   caster-01
   ```

4. 准备一台装有ROS Melodic的Ubuntu电脑，下文简称**外部PC**

5. 将网线连接到底盘的网口上，并参考[网络配置信息-外部设备设置](101-network_info.md#外部设备设置)，设置外部PC的IP地址

6. 连接网线后，在外部电脑中开启一个终端，使用PING指令，测试和Caster-PC的网络通信情况

   ```bash
   # 使用IP地址测试
   ping 192.168.33.5
   
   # 或者使用第3步获取的机器人名称，例如ping caster-01.local
   ping (机器人名称).local
   ```

7. 上述指令可以成功执行后，使用终端设置外部PC的**~/.bashrc**文件，让外部PC使用Caster-PC的ROS Master

   ```bash
   # 获取外部PC的主机名
   hostname
   
   # 设置本机名称，用上指令的返回值替换(hostname)，例如remote-pc.local
   echo 'export ROS_HOSTNAME="(hostname).local"' >> ~/.bashrc
   
   # 设置ROS Master地址，用第3步的返回值替换(机器人名称)，例如caster-01.local
   echo 'export ROS_MASTER_URI="http://(机器人名称).local:11311"' >> ~/.bashrc
   ```

8. 完成上述操作后，重新启动外部PC的Terminal，执行如下指令，安装CasterMoma的Desktop包

   ```bash
   # 安装必要的软件
   sudo apt install python-wstool python-catkin-tools
   
   # 创建Caster Moma的ROS工作空间
   mkdir -p ~/caster_moma_ws/src
   cd ~/caster_moma_ws/src
   wstool init https://raw.githubusercontent.com/CasterLab/caster_moma_rosinstall/master/caster_moma_desktop.rosinstall
   wstool update	# 此步骤可能会因为网络政策的原因失败，请多试几次
   
   # 安装ROS的包依赖，并编译
   rosdep install --from-paths . --ignore-src -r -y
   cd ~/caster_moma_ws
   catkin build
   
   # 添加环境依赖
   source ~/caster_moma_ws/devel/setup.bash
   echo 'source ~/caster_moma_ws/devel/setup.bash' >> ~/.bashrc
   ```

## ROS功能启动

1. 从外部PC使用SSH登录到Caster的PC中，IP地址参考[网络配置信息](101-network_info.md)

   ```bash
   # 如果机器人的名称为caster-01
   ssh caster@caster-01.local
   
   # 按照提示，输入密码（参考《网络配置信息》）
   ```

1. 运行如下代码，启动CasterMoma的控制程序，此launch文件会启动CasterMoma中所有的执行器和传感器

   ```bash
   # Kinova Gen3手臂
   roslaunch caster_moma_bringup caster_moma_gen3_bringup.launch
   
   # Kinova j2s6s300 手臂
   roslaunch caster_moma_bringup caster_moma_j2s6s200_bringup.launch
   
   # Kinova j2n6s300 手臂
   roslaunch caster_moma_bringup caster_moma_j2n6s200_bringup.launch
   ```

2. 此时可以在外部PC中，使用Rviz来查看Caster传感器的信息

   ```bash
   roslaunch caster_moma_viz display.launch
   ```

3. Rviz视图

   <img src="../img/display.png" alt="Rviz" style="zoom:50%;" />

4. 机器人诊断信息视图

   <img src="../img/rqt_runtime_monitor.png" alt="rqt_runtime_monitor" style="zoom:60%;" />

## 手柄控制

Caster可以使用手柄进行遥控，具体按键功能参考[手柄按键说明图](102-joystick_description.md)

1. 参考[ROS功能启动](02-quick_start_remote.md#ROS功能启动)，启动Caster的ROS功能

4. 根据[手柄按键说明图](102-joystick_description.md)，操控Caster进行移动

5. 对于移动操作，只有在按下`安全键`的时候，信号才会被Caster接收到。

## 云台控制

1. 参考[ROS功能启动](02-quick_start_remote.md#ROS功能启动)，启动Caster的ROS功能

2. 使用如下指令可以控制云台的俯仰，此指令既可以在外部PC上使用，也可以在机器人上使用

   ```bash
   # 归零
   rostopic pub /pan_tilt_driver_node/pan_tilt_cmd pan_tilt_msgs/PanTiltCmd "{yaw:0.0, pitch:0.0, speed:20.0}"
   
   # 旋转和俯仰同时动作
   rostopic pub /paiitilt_driver_node/pan_tilt_cmd pan_tilt_msgs/PanTiltCmd "{yaw:40.0, pitch:40.0, speed:20.0}"
   ```

## 躯干控制

1. 参考[ROS功能启动](02-quick_start_remote.md#ROS功能启动)，启动Caster的ROS功能

2. 使用如下指令可以控制躯干的升降，此指令既可以在外部PC上使用，也可以在机器人上使用

   ```bash
   # 躯干降到最低位
   rostopic pub /caster/body_controller/command std_msgs/Float64 "data: 0.0"
   
   # 躯干升到最高位
   rostopic pub /caster/body_controller/command std_msgs/Float64 "data: 0.4"
   ```

## 创建地图

1. 参考[ROS功能启动](02-quick_start_remote.md#ROS功能启动)，启动Caster的ROS功能

2. 使用SSH登录Caster-PC，并执行如下指令

   ```bash
   roslaunch caster_navigation gmapping.launch
   ```

3. 在外部PC中，运行如下指令，启动建图界面

   ```bash
   roslaunch caster_moma_viz display.launch type:=gmapping
   ```

4. 参考[手柄控制](02-quick_start_remote.md#手柄控制)，启动手柄遥控，操控Caster完成地图建立

5. 使用SSH登录Caster-PC，并执行如下指令保存地图

   ```bash
   # 使用map_saver保存地图, 扫描后的地图推荐存放至caster_navigation的map
   rosrun map_server map_saver -f $(rospack find caster_navigation)/map/[地图名称]
   ```

**注意：在建立地图时，请勿将充电桩放置于地面上，地图建立完成后再放置充电桩**

## 定位导航

1. 参考[ROS功能启动](02-quick_start_remote.md#ROS功能启动)，启动Caster的ROS功能

2. 参考[创建地图](02-quick_start_remote.md#创建地图)，完成地图创建

3. 使用SSH登录Caster-PC，并执行如下指令，启动导航功能

   ```bash
   # 使用map_file参数指定要加载的地图，例如加载home地图
   roslaunch caster_navigation navigation.launch map_file:=$(rospack find caster_navigation)/map/home.yaml
   ```

4. 在外部PC中，运行Rviz，用来监控机器人状态

   ```bash
   roslaunch caster_moma_viz display.launch type:=navigation
   ```

5. 在Rviz中，设定机器人的初始位置

6. 根据需求设定机器人目标点，进行运动

## 使用机械臂

1. 参考[ROS功能启动](02-quick_start_remote.md#ROS功能启动)，启动Caster的ROS功能

2. 根据机械臂型号和需求，选择如下指令启动机械臂的MoveIt控制功能，或仿真（仿真不需要启动Caster的ROS功能）。

   ```bash
   # 6自由度-2指-球关节 机械臂仿真
   roslaunch caster_moma_j2s6s200_config demo.launch
   # 6自由度-2指-球关节 机械臂真机
   roslaunch caster_moma_j2s6s200_config execute.launch

   # 6自由度-2指-非球关节 机械臂仿真
   roslaunch caster_moma_j2n6s200_config demo.launch
   # 6自由度-2指-非球关节 机械臂真机
   roslaunch caster_momn_j2n6s200_config execute.launch
   
   # Gen3 7自由度-2指 机械臂仿真
   roslaunch caster_moma_gen3_7d_2f85_moveit_config demo.launch
   # Gen3 7自由度-2指 机械臂真机
   roslaunch caster_moma_gen3_7d_2f85_moveit_config execute.launch
   ```

3. Moveit视图

   <img src="../img/moveit.png" alt="moveit" style="zoom:50%;" />

## 自动充电功能

1. 参考[定位导航](02-quick_start_remote.md#定位导航)，启动Caster的定位导航功能，并正确设置Caster在地图中的位置

2. 参考[自动充电原理和配置](03-auto_charge_description.md#参数配置)，完成对充电桩位置的设定

3. 使用SSH登录Caster-PC，运行如下指令，启动自动充电功能

   ```bash
   roslaunch caster_auto_charge auto_charge.launch
   ```

4. 按下手柄的`START`按键，Caster开始执行自动充电程序

5. 充电完成后，按下手柄的`BACK`按键，Caster从充电桩中推出

**注意：在Caster位于充电桩内时，勿使用导航功能，以免损坏Caster的受电模块。要使用导航功能，请确保Caster正确从充电桩中推出**