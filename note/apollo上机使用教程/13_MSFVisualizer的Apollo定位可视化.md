# 基于MSFVisualizer的Apollo定位可视化实践

更新时间：2022-01-26

本文档介绍如何利用 Apollo MSF 可视化工具，对定位结果进行可视化。

## 前提条件

- 完成必备软件安装
- 使用安装包安装Apollo

## 步骤一：在本地安装 X-Server

### 1. 启动 Apollo Docker 环境

进入 Apollo 安装路径，启动终端，执行脚本，进入 Apollo Docker 环境。



```shell
cd apollo
./apollo.sh
```

### 2. 为 docker 配置 X-Server



```shell
sudo apt-get update
sudo apt install x11-xserver-utils
```

## 步骤二：启动定位模块

可视化工具受消息触发后才会运行，启动可视化工具前，请您先启动定位模块。参见 [使用MSFLocalizer的多传感器融合定位实践步骤一](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo定位能力/使用MSFLocalizer的多传感器融合定位实践#步骤一)。

## 步骤三：可视化工具使用

执行以下命令启动可视化工具：



```shell
cyber_launch start /apollo/modules/localization/launch/msf_visualizer.launch
```

第一次运行时，可视化工具需要生成可视化需要的图像，等待较久的时间是正常的。

当弹出如下图所示的窗口时，说明可视化工具运行正常，其中灰色背景表示地图，绿色细线表示原始点云数据，红色粗线圆圈表示 LiDAR 定位的置信度，绿色粗线圆圈表示 MSF 定位的置信度。

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_2fc50ce.png)