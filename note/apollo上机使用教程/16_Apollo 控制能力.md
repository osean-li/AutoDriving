# Apollo 控制能力介绍

更新时间：2021-11-16

本文档为开发者介绍 Apollo 控制模块的基本知识、Apollo 控制能力、控制模块的组成及其输入输出。

## Apollo 控制能力介绍

控制模块是整个自动驾驶软件系统中的执行环节，控制模块的目标是基于规划模块输出的目标轨迹和定位模块输出的车辆状态生成方向盘、油门、刹车控制命令，并通过 canbus 模块给车辆底层执行器。简单而言，就是告诉车辆该打多大方向盘、多大的油门开度、多大的刹车制动力。

## Apollo 控制模块原理

控制模块由两个子模块组成：横向控制模块和纵向控制模块。

### 横向控制模块

横向控制根据规划模块的轨迹生成方向盘指令。

横向控制模块如下图所示：

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_8c0d1a7.png)

- 方向盘闭环：通过上游规划模块输出的目标方向盘角度和下游 canbus 模块返回的实际方向盘角度做差，作为控制器的方向盘闭环输出。
- 方向盘开环：通过上游规划模块输出的目标道路曲率经过转换，输出控制器方向盘开环输出。
- 车辆：是控制器的控制对象，通过方向盘开环、方向盘闭环生成的方向盘角度打方向盘。

### 纵向控制模块

纵向模块根据规划模块的轨迹生成油门、刹车指令。

纵向控制模块如下图所示：

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_e6b4874.png)

- 位置闭环：通过规划模块的目标车辆位置和定位模块给出的实际车辆位置做差，经过纵向位置控制器生成对应的速度偏差。
- 速度闭环：结合位置闭环的结果，规划模块的目标速度以及 canbus 模块返回的实际车速，通过纵向速度控制器生成对应的加速度偏差。
- 表定表：速度，加速度和油门、刹车对应关系表，通过输入速度、加速度可以查出对应的油门、刹车。

## Apollo 控制模块输入输出

控制模块的输入输出的 channel 如下图所示。 ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_7c17bb9.png)



### 控制模块输入 channel

控制模块有三个输入 channel：

| channel名称               | 输入输出 | channel说明                      |
| ------------------------- | -------- | -------------------------------- |
| /Apollo/planning          | 输入     | 规划信息，自车规划的轨迹信息     |
| /Apollo/localization/pose | 输入     | 定位信息，自车的位置             |
| /Apollo/canbus/chassis    | 输入     | 底盘信息，自车的方向盘、速度信息 |



### 控制模块输出 channel

控制模块有一个输出 channel：

| channel名称     | 输入输出 | channel说明                      |
| --------------- | -------- | -------------------------------- |
| /Apollo/control | 输出     | 控制信息，方向盘角度、油门、刹车 |



## 参考文献

1. "Vehicle dynamics and control." Rajamani, Rajesh. Springer Science & Business Media, 2011.
2. "Optimal Trajectory generation for dynamic street scenarios in a Frenet Frame", M. Werling et., ICRA2010