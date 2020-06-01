## 前期准备

1. 一台装有ROS Melodic的Ubuntu电脑，下文简称**外部PC**

2. 将网线连接到底盘的网口上，并参考[网络信息-外部设备设置](network_info.md#外部设备设置)，设置外部PC的IP地址

3. 使用PING指令，测试和Caster-PC的网络通信情况

   ```bash
   # 使用IP地址测试
   ping 192.168.33.5
   
   # 或者使用域名测试
   ping caster-pc.local
   ```

4. 设置外部PC的**~/.bashrc**文件，让外部PC使用Caster-PC的ROS Master

   ```bash
   echo 'export ROS_MASTER_URI="http://caster-pc.local:11311"' >> ~/.bashrc
   ```

5. 重新启动外部PC的Terminal

## ROS功能启动

1. 使用SSH登录到Caster的PC中，IP地址参考[网络信息](network_info.md)

2. 运行如下代码，启动底盘控制程序

   ```bash
   roslaunch caster_man_bringup bringup.launch
   ```

3. 也可以在外部PC中，使用Rviz来查看Caster传感器的信息

   ```bash
   roslaunch caster_man_viz display.launch
   ```

## 手柄控制

Caster可以使用手柄进行遥控，具体按键功能参考[手柄功能定义](joystick_description.md)

1. 参考[ROS功能启动](quick_start.md#ROS功能启动)，启动Caster的ROS功能

4. 根据[手柄功能定义](joystick_description.md)，操控Caster进行移动

5. 对于移动操作，只有在按下`安全键`的时候，信号才会被Caster接收到。

## 创建地图

1. 参考[ROS功能启动](quick_start.md#ROS功能启动)，启动Caster的ROS功能

2. 使用SSH登录Caster-PC，并执行如下指令

   ```bash
   roslaunch caster_navigation gmapping.launch
   ```

3. 在外部PC中，运行如下指令，启动建图界面

   ```bash
   roslaunch caster_man_viz display.launch type:=gmapping
   ```

4. 参考[手柄控制](quick_start.md#手柄控制)，启动手柄遥控，操控Caster完成地图建立

5. 使用SSH登录Caster-PC，并执行如下指令保存地图

   ```bash
   # 使用map_saver保存地图, 扫描后的地图推荐存放至caster_navigation的map
   rosrun map_server map_saver -f $(rospack find caster_navigation)/map/[地图名称]
   ```

**注意：在建立地图时，请勿将充电桩放置于地面上，地图建立完成后再放置充电桩**

## 定位导航

1. 参考[ROS功能启动](quick_start.md#ROS功能启动)，启动Caster的ROS功能

2. 参考[创建地图](quick_start.md#创建地图)，完成地图创建

3. 使用SSH登录Caster-PC，并执行如下指令，启动导航功能

   ```bash
   # 使用map_file参数指定要加载的地图，例如加载home地图
   roslaunch caster_navigation navigation.launch map_file:=$(rospack find caster_navigation)/map/home.yaml
   ```

4. 在外部PC中，运行Rviz，用来监控机器人状态

   ```bash
   roslaunch caster_man_viz display.launch type:=navigation
   ```

5. 在Rviz中，设定机器人的初始坐标

## 使用机械臂

1. 参考[ROS功能启动](quick_start.md#ROS功能启动)，启动Caster的ROS功能

2. 根据机械臂型号和需求，选择如下指令启动机械臂的MoveIt控制功能，或仿真

   ```bash
   # 6自由度-2指机械臂仿真
   roslaunch caster_man_j2s6s200_config demo.launch
   # 6自由度-2指机械臂真机
   roslaunch caster_man_j2s6s200_config execute.launch

   # Gen3 7自由度-2指机械臂仿真
   roslaunch caster_gen3_moveit_config demo.launch
   # Gen3 7自由度-2指机械臂真机
   roslaunch caster_gen3_moveit_config execute.launch
   ```

## 自动充电功能

1. 参考[定位导航](quick_start.md#定位导航)，启动Caster的定位导航功能，并正确设置Caster在地图中的位置

2. 参考[自动充电原理和配置](auto_charge_description.md#参数配置)，完成对充电桩位置的设定

3. 使用SSH登录Caster-PC，运行如下指令，启动自动充电功能

   ```bash
   roslaunch caster_auto_charge auto_charge.launch
   ```

4. 按下手柄的`START`按键，Caster开始执行自动充电程序

5. 充电完成后，按下手柄的`BACK`按键，Caster从充电桩中推出

**注意：在Caster位于充电桩内时，勿使用导航功能，以免损坏Caster的受电模块。要使用导航功能，请确保Caster正确从充电桩中推出**