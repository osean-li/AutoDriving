# Apollo视觉感知模块实践

更新时间：2022-02-21

本文档主要介绍 Apollo 视觉感知模块实践相关内容。通过本文档，您可以学到：

- 如何在本机环境中启动 Apollo 视觉感知模块。
- 播放对应的 recorder 查看感知的输出结果。

## 启动视觉感知模块

### 步骤一：进入 Docker 环境并启动 Dreamview

1. 进入 Apollo Docker 环境。

   

   ```bash
   ./apollo.sh
   ```

2. 在 Docker 环境里，启动 Dreamview。

   

   ```bash
   ./scripts/bootstrap.sh
   ```

   如果需要关闭 Dreamview，请您执行以下命令：

   ```text
   ./scripts/bootstrap.sh stop
   ```

### 步骤二：选择车型和地图

1. 在浏览器输入网址 `http://localhost:8888/`，打开 Dreamview。

2. 选择车型和地图。

   - 在 Dreamview 右上角的 **--vehicle--** 中，选择 **mkz_example**，
   - 在 Dreamview 右上角的 **--map--** 中，选择 **Sunnyvale Big Loop**。

   车辆选择后，程序会把会把 `calibration/data` 目录下对应的车型参数和传感器内外参文件拷贝到作用目录下。地图选择后在 Dreamview 下便可以正常显示当前地图。

### 步骤三：启动所需模块

1. 在 Dreamview 上打开 **Transform** 按钮。

   您也可以在终端执行如下命令：

   ```text
   cyber_launch start /apollo/modules/transform/launch/static_transform.launch
   ```

   **Transform** 模块为了发布传感器各坐标系变换供各模块调用。

2. 启动 **image decompression** 模块。

   ```text
   cyber_launch start modules/drivers/tools/image_decompress/launch/image_decompress.launch
   ```

   视觉模块依赖的输入是相机传感器采集到的图像数据，**image decompression** 模块可以将压缩的图像数据解压缩。

### 步骤四：启动视觉感知模块

**启动红绿灯检测模块**

```text
cyber_launch start /apollo/modules/perception/production/launch/perception_trafficlight.launch
```

**启动视觉障碍物检测模块**

```text
mainboard -d modules/perception/production/dag/dag_streaming_perception_camera.dag
```

这个命令会同时开启视觉障碍物检测和车道线检测，在启动时终端可能出现一些红色的警告提示，只要进程不退出都是正常现象，模型可以正常启动。

视觉感知模块启动需要一些时间，您可以通过下面两种方法判断是否启动完毕：

- 在终端执行如下命令，查看 GPU 的占用情况：

  

  ```bash
    watch -n 0.1 nvidia-smi
  ```

  当 mainboard 进程显存占用不再变化，证明模型已经完成启动。

- 在 Docker 环境下，执行如下命令：

  

  ```bash
   tail -f /apollo/data/log/mainboard.INFO
  ```

  该命令表示实时显示 cyber 日志的尾部。当屏幕中该日志显示如下信息时，则可以确定感知模块已启动完毕。

  ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_9c32931.png)

**（可选）单独启动车道线检测模块**

```text
mainboard -d ./modules/perception/production/dag/dag_streaming_perception_lane.dag
```

## 在 Dreamview 上可视化检测结果

视觉障碍物主要检测远处（>50m）的障碍物。视觉障碍物感知结果不单独输出，而是跟激光雷达的检测结果融合后输出。为了让您直观地感受 Apollo 视觉感知模块的输出，本文档把视觉感知的结果单独输出。完成视觉感知实践操作后，要将 channel 的修改复原，以免影响雷达感知的输出。

在 Dreamview 上查看视觉感知的检测结果，请您按照以下步骤对配置文件进行修改：

### 步骤一：下载并解压数据包

数据包里存放了我们路测时采集的数据，包括定位、感知、预测规划等模块的信息。我们提供了测试数据包 [demo_sensor_data_for_vision.tar.xz](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data_for_vision.tar.xz) 供开发者进行测试。



```bash
   wget https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data_for_vision.tar.xz
```



```bash
   tar -xvf demo_sensor_data_for_vision.tar.xz
```

### 步骤二：将数据包复制指定目录

为了在 Docker 中可以访问 record，将下载的数据包复制到 `/apollo/data/bag` 目录下。

### 步骤三：进入 Docker 环境并启动 Dreamview

1. 进入 Apollo Docker 环境。

   ```text
   ./apollo.sh
   ```

2. 在 Docker 环境里，启动 Dreamview。

   ```text
   ./scripts/bootstrap.sh
   ```

   > 注意：再次启动 Dreamview 前，请您先关闭 Dreamview。

### 步骤四：选择车型和地图

1. 在浏览器输入网址 `http://localhost:8888/`，打开 Dreamview。

2. 选择车型和地图。

   - 在 Dreamview 右上角的 **--vehicle--** 中，选择 **mkz_example**，
   - 在 Dreamview 右上角的 **--map--** 中，选择 **Sunnyvale Big Loop**。

   车辆选择后，程序会把会把 `calibration/data` 目录下对应的车型参数和传感器内外参文件拷贝到作用目录下。地图选择后在 Dreamview 下便可以正常显示当前地图。

### 步骤五：启动所需模块

1. 在 Dreamview 上打开 **Transform** 按钮。

   您也可以在终端执行如下命令：

   ```text
   cyber_launch start /apollo/modules/transform/launch/static_transform.launch
   ```

   **Transform** 模块为了发布传感器各坐标系变换供各模块调用。

2. 启动 **image decompression** 模块。

   ```text
   cyber_launch start modules/drivers/tools/image_decompress/launch/image_decompress.launch
   ```

   视觉模块依赖的输入是相机传感器采集到的图像数据，**image decompression** 模块可以将压缩的图像数据解压缩。

### 步骤六：修改输出的 channel 名称

视觉障碍物检测默认的输出 channel 为 `/perception/obstacles`，不会在 Dreamview 上显示检测结果。如果想要在 Dreamview 上可视化结果，需要修改下面文件的一部分关键字。

1. 打开文件。

```text
 vim modules/perception/production/conf/perception/camera/fusion_camera_detection_component.pb.txt
```

1. 定位到第 10 行，修改为：

   ```text
   output_final_obstacles : true
   ```

2. 定位到第 11 行，修改为：

   ```text
   output_obstacles_channel_name : "/apollo/perception/obstacles"
   ```

   > 注意：请您修改完 channel 信息之后，再启动障碍物检测模块。否则会检测不到障碍物。

### 步骤七：启动视觉障碍物检测模块

```text
mainboard -d modules/perception/production/dag/dag_streaming_perception_camera.dag
```

> 注意：成功启动的判定标准是观察处理器的显存不再发生变化或者 cyber 日志尾部输出结束信息。

### 步骤八：播放数据包

等到感知模块成功启动后，执行如下命令：

```text
cyber_recorder play -f /apollo/data/bag/demo_sensor_data_for_vision.record -k /apollo/perception/obstacles /apollo/prediction
```

如果在 Dreamview 可以看到障碍物，说明模块启动成功，如下图所示。图中的绿色框便是我们的视觉感知算法检测出的障碍物。

完成视觉感知实践操作后，要将 channel 的修改复原，以免影响雷达感知的输出。

![image_6b83b30.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_6b83b30_6b83b30.png)