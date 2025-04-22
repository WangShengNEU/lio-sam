# 下载源码

    > # cd /LIO-SAM/catkin_ws/src
    > git clone https://github.com/TixiaoShan/LIO-SAM.git

# 工程参数修改及问题解决

## 修改 /LIO-SAM/CMakeList.txt

### pcl问题

为了让LIO-SAM支持C++14，需要修改下LIO-SAM文件夹下的CMakeList.txt文件：修改如下：

    > set(CMAKE_CXX_FLAGS "-std=c++14") #将c++11改为c++14

## /usr/bin/ld问题

同样在刚刚打开的CMakeLists文件中（最后）加入下述代码：

    > find_package(Boost REQUIRED COMPONENTS timer thread serialization chrono)

## 修改 /LIO-SAM/include/utility.h

### OpenCV 版本问题

找到/LIO-SAM/include/utility.h中的：#include<opencv/cv.h>，修改为

    > #include <opencv2/imgproc.hpp>

并将其置于
  
    > #include <pcl/kdtree/kdtree_flann.h>

之后

## 修改 config/params.yaml **(每次修改完 params.yaml 都需要重新编译)**

### 地图保存

如果想要保存地图，需要对config/params.yaml文件进行如下修改 

    > # 保存地图设置为true
    > savePCD: true         
    > # 设置地图保存路径
    > savePCDDirectory: "自己设置的路径"   

更改了配置文件后，还需更改一下_TIMEOUT_SIGINT参数，否则可能造成地图保存失败(这是由于ros会在_TIMEOUT_SIGINT秒后关闭ros节点，但是地图过大时，保存地图会花费一些时间，如果_TIMEOUT_SIGINT太小，很可能造成地图还未保存，节点就已经关闭了，所以需要适当调高_TIMEOUT_SIGINT值)，具体命令：

    > sudo gedit /opt/ros/noetic/lib/python3/dist-packages/roslaunch/nodeprocess.py

### 需要配置参数的数据集

    (1) 旋转数据集–Rotation dataset
    (2) 校园数据集(large)–Campus dataset (large)
    (3) 校园数据集(small)–Campus dataset (small)

在这些数据集中，点云主题是"points_raw"。 IMU 主题是"imu_correct"，它给出了 ROS REP105 （ros的坐标系参考标准）标准中的 IMU 数据。由于此数据集不需要 IMU 转换，因此需要更改以下配置才能成功运行此数据集：
  
==>>“config/params.yaml"中的"imuTopic"参数需要设置为"imu_correct”。
  
  > imuTopic: "imu_raw" # IMU data

"config/params.yaml"中的"extrinsicRot"和"extrinsicRPY"需要设置为单位矩阵(identity matrices)

  > extrinsicRot: [1, 0, 0,
                  0, 1, 0,
                  0, 0, 1]
  > extrinsicRPY: [1, 0, 0,
                  0, 1, 0,
                  0, 0, 1]

# 编译

    > # cd ..
    > catkin_make

# 启动程序运行示例数据

    > source /opt/ros/noetic/setup.bash

    > source devel/setup.bash

# 启动lio-sam功能包

    > roslaunch lio_sam run.launch

# 播放数据集，数据路径下打开新终端

    > rosbag play outdoor.bag

# 倍速播放，如果内容不足，不要倍速播放

    > rosbag play casual_walk.bag -r 3

[参考](https://blog.csdn.net/qq_42938987/article/details/108434290)

# 保存地图

rosservice call [service] [resolution] [destination]
  
    > # 打开新终端
    > source devel/setup.bash
    > rosservice call /lio_sam/save_map 0.2 "~/LIO-SAM/catkin_ws/src/output_maps"
