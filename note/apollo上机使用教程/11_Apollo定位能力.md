# Apollo定位能力介绍

更新时间：2021-11-18

本文档介绍 Apollo 定位模块以及 Apollo MSF（Multi-Sensor Fusion）的原理。

## Apollo 定位模块

高精度、高鲁棒性的定位系统是自动驾驶系统不可或缺的基础模块。Apollo 定位模块针对不同的应用需求提供了三种不同实现方式的定位模块。

### RTK（Real Time Kinematic）定位模块

利用 GPS + IMU 实现的全局定位导航系统，在 GPS 信号良好的情况下可以实现厘米级定位精度。

### MSF（Multi-Sensor Fusion）定位模块

结合 GPS + IMU + Lidar 实现的多传感器融合全局定位导航系统，利用多传感器优缺点的互补，实现高精度、高鲁棒性的定位能力。对于 GPS 失效或者 Lidar 地图环境变更场景具备一定的冗余处理能力。本模块可提供城市道路、高速、部分隧道等场景下的定位能力。

### NDT（Normal Distribution Transform）定位模块

结合 GPS + IMU + Lidar 实现的简单融合定位系统，其中 lidar 定位采用开源 NDT 算法实现。本系统依赖 GPS 信号质量，具备简单城市场景下的高精度，鲁棒的定位能力。

## MSF 定位模块原理

这里重点介绍 （Multi-Sensor Fusion）定位模块实现原理，系统框图如下图所示：

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_fc494fd.png)

MSF 定位系统以多种传感器数据和离线制作的高精度 Lidar 定位地图为输入，其中 GNSS Localization 模块以车端 GPS 信号和基站数据为输入，输出高精度 RTK 定位结果。LiDAR Localization 模块以在线 lidar 扫描数据和高精度 Lidar 定位地图为输入，提供高精度 lidar 定位结果。SINS 模块利用IMU数据进行惯性导航。后端采用 error-state Kalman filter 融合多种传感器量测信息。输出高精度的车辆位置和姿态。

详情参见 [Robust and Precise Vehicle Localization Based on Multi-Sensor Fusion in Diverse City Scenes](https://ieeexplore.ieee.org/document/8461224)。