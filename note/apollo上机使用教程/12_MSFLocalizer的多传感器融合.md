# 使用MSFLocalizer的多传感器融合定位实践

更新时间：2022-01-26

本文档介绍 Apollo 多传感器融合定位（MSF Localization）的使用方法以及效果演示。

## MSF Localizer 概览

MSF Localizer 订阅和发布的主要 Channel 如下表所示：

| Channel                                        | 输入/输出 | 说明             |
| ---------------------------------------------- | --------- | ---------------- |
| /apollo/sensor/hesai40/compensator/PointCloud2 | 输入      | 融合后的点云数据 |
| /apollo/sensor/gnss/imu                        | 输入      | IMU 消息         |
| /apollo/sensor/gnss/odometry                   | 输入      | 里程计消息       |
| /apollo/sensor/gnss/best_pose                  | 输入      | GPS 位置         |
| /apollo/sensor/gnss/heading                    | 输入      | GPS 航向角       |
| /apollo/localization/msf_gnss                  | 输出      | GNSS 定位结果    |
| /apollo/localization/msf_lidar                 | 输出      | Lidar 定位结果   |
| /apollo/localization/pose                      | 输出      | 融合定位结果     |
| /apollo/localization/msf_status                | 输出      | 融合定位状态     |



## 步骤一：启动 MSF Localizer

### 1. 配置环境

请您按照以下步骤配置环境：

1. 安装必备软件，参见 [必备软件安装指南](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/安装说明/必备软件安装指南)。
2. 使用安装包安装 Apollo，参见 [使用安装包安装Apollo](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/安装说明/使用安装包安装Apollo)。
3. 准备定位测试数据。有两种方式：
   - 使用自有硬件平台录制 recorder 数据，需包含的传感器有雷达、IMU、GNSS，
   - 使用 Apollo 提供的开源数据，按照下面第 3 小节下载定位 demo 数据。

### 2. 启动 Apollo Docker 环境

1. 启动终端，进入 Apollo 安装路径：

   ```text
   cd apollo
   ```

2. 执行脚本进入 Apollo Docker 环境：

   ```text
   ./apollo.sh
   ```

   出现以下提示，证明您已经进入 docker 环境：

   ```text
   [user@in-runtime-docker:/apollo]$ 
   ```

### 3. 下载并解压数据包

下载定位 demo 数据 [demo-localization-data-3.5.tar.gz](https://apollo-system.cdn.bcebos.com/dataset/localization/demo-localization-data-3.5.tar.gz)。

执行以下命令下载数据：

```text
wget https://apollo-system.cdn.bcebos.com/dataset/localization/demo-localization-data-3.5.tar.gz
```

执行以下命令解压数据：

```text
tar -xvf demo-localization-data-apollo-3.5.tar.gz
```

解压后得到以下文件：

```text
- data
 |- local_map           # MSF Localization 定位地图
 |- ndt_map             # NDT Localization 定位地图
 |- params              # 车辆参数
 |- records             # bag 数据
```

### 4. 将数据包复制到指定目录

为了在 Docker 中可以访问下载的数据包，将下载的数据包复制到 `/apollo/data/bag` 目录下。

### 5. 配置参数

请您按照以下步骤配置参数：

1. 配置传感器外参。

   运行以下命令，拷贝车辆参数到定位模块目录下：

   ```text
   # DATA_PATH 请修改为第 3 小节 中下载的 demo 数据的路径
   cp -r DATA_PATH/params/* /apollo/modules/localization/msf/params/
   ```

   params 文件夹下包含以下文件：

   ```text
   - gnss_params
   |- ant_imu_leverarm.yaml:               #杆臂值
   - velodyne_params
   |- velodyne128_novatel_extrinsics.yaml: #IMU -> LiDAR 坐标变换
   |- velodyne128_height.yaml:             #LiDAR 相对于地面的高度
   ```

2. 配置地图参数。

   打开 `/apollo/modules/localization/conf/localization.conf` 文件并修改 `--map_dir` 字段：

   ```text
   # 使用 vim 打开 localization.conf
   vim /apollo/modules/localization/conf/localization.conf
   # 移动到第五行，修改 --map_dir 字段
   --map_dir=DATA_PATH
   ```

### 6. 启动 MSF Localizer

```text
cyber_launch start /apollo/modules/localization/launch/msf_localization.launch
```

### 7. 播放数据包

```text
 # DATA_PATH 请修改为 3.下载定位 demo 数据的路径
cd DATA_PATH/records
cyber_recorder play -f record.* 
```

## 步骤二：定位结果查看

### 1. 使用 Cyber Monitor 查看 Channel

在 Apollo docker 环境中运行以下命令：

```text
cyber_monitor
```

如果 recorder 数据包正常，可以看到定位发出的 topic 消息。 当看到以下 topic 显示绿色时，定位模块运行正常：

```text
/apollo/localization/msf_lidar
/apollo/localization/msf_status
/apollo/localization/pose
```

![855d4fbc0b1501f8bdfcebbcf05f09a4.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/855d4fbc0b1501f8bdfcebbcf05f09a4_855d4fb.png)



### 2. 使用 DreamView 查看定位状态

请按照以下步骤使用 DreamView 查看定位状态：

1. 启动 DreamView。

   运行以下命令启动 Dreamview 后台，如果已经启动可跳过这一步：

   ```text
   ./scripts/bootstrap.sh
   ```

2. 在 Dreamview 中选择车辆模型和地图。

   在浏览器中打开 `http://localhost:8888`，在左边栏选择 Module Controller，在右上角的下拉框中选择对应的车辆模型和地图：

   - 车辆模型选择 Mkz Example，
   - 地图选择 Sunnyvale Big Loop。

3. （可选）在 Dreamview 中配置视角。

   在浏览器中打开 `http://localhost:8888`，在左边栏选择 **Layer Menu**，

   - 在 **Point of View** 中选择 **Overhead**（鸟瞰），
   - 在 **Position** 中同时勾选 **Localization** 和 **GPS**。

   如果 recorder 数据包和地图正常，可以看到以下图像：

   ![833db996c97416551bfbb78cc7bddf0a.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/833db996c97416551bfbb78cc7bddf0a_833db99.png)

   网页正中是地图和汽车，其中黑白的背景是地图，颜色表示环境中反射值：颜色越亮表示反射值越高，颜色越暗表示反射值越低，纯黑色表示没有被激光雷达扫描到的区域。

   白色车辆模型表示经过 MSF 融合算法得到的位置坐标，蓝色方框表示 GPS 位置，可以在视频中看到蓝色方框在不停跳动，但车辆位置比较稳定，说明融合定位算法提供了更好的平滑性。