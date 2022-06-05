# 使用Dreamview调试Apollo激光雷达感知实践

更新时间：2022-03-24

本文档旨在为开发者提供本地环境下，如何利用 Apollo 的仿真界面 Dreamview 运行 Apollo 激光雷达感知模块的说明。通过本文档，您可以学习到：

- 如何通过 Apollo 开发环境和 Dreamview 运行激光雷达感知模块。
- 如何进行点云和障碍物信息可视化。

## 步骤一：进入 Apollo 环境并启动 Dreamview

1. 进入 Apollo Docker 环境。

```text
./apollo.sh
```

1. 在 Docker 环境里启动 Dreamview。

```text
bash scripts/bootstrap.sh
```

如果需要关闭 Dreamview，请您执行以下命令：

```text
./scripts/bootstrap.sh stop
```

## 步骤二：选择车型和地图

1. 在浏览器中输入网址 `http://localhost:8888`，打开 Dreamview。

2. 选择相应的模式、车辆和地图信息。

   ![2021-07-13 16-50-59屏幕截图.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/2021-07-13%2016-50-59%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE_7d5988d.png)

## 步骤三：启动所需模块

1. 修改 channel 信息。

- 修改 `/apollo/modules/perception/production/dag` 目录下`dag_streaming_perception.dag` 文件。

  ```text
  vim /apollo/modules/perception/production/dag/dag_streaming_perception.dag
  ```

  用文本编辑工具 vim 将 `channel: "/apollo/sensor/lidar128/compensator/PointCloud2"` 修改为 `channel: "/apollo/sensor/velodyne64/compensator/PointCloud2"`。

![2021-07-08 14-20-17屏幕截图.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/2021-07-08%2014-20-17%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE_9e65374.png)

- 修改 `/apollo/modules/perception/production/conf/perception/lidar`目录下`velodyne128_segmentation_conf.pb.txt` 文件。

  用文本编辑工具 vim 将 `lidar2novatel_tf2_child_frame_id: "velodyne128"` 修改为 `channel: lidar2novatel_tf2_child_frame_id: "velodyne64"`。

  ```text
  vim /apollo/modules/perception/production/conf/perception/lidar/velodyne128_segmentation_conf.pb.txt
  ```

![2021-07-13 16-58-02屏幕截图.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/2021-07-13%2016-58-02%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE_9868f5c.png)

1. 在 **Module Controller** 标签页启动 **Perception** 模块。

   ![2021-07-13 17-02-39屏幕截图.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/2021-07-13%2017-02-39%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE_7c89fdd.png)

   **Perception** 模块启动一般需要 2 分钟左右时间。如需确定感知模块是否已启动成功，您可以在 Docker 环境下，命令行输入：

   ```text
   tail -f /apollo/data/log/mainboard.INFO
   ```

   该命令表示实时显示 cyber 日志的尾部。当屏幕中该日志显示如下信息时，则可以确定感知模块已启动完毕。

   ![Screenshot from 2021-06-25 17-02-23_bed4020.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/Screenshot%20from%202021-06-25%2017-02-23_bed4020_bed4020.png)

## 步骤四：下载并使用 Cyber 播放数据包

1. 下载数据包。

   在您本地电脑上下载 Apollo 的传感器原始数据包 [demo_sensor_data.tar.xz](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz)。

2. 解压下载好的数据包。

   ```text
   tar -xvf demo_sensor_data.tar.xz
   ```

   解压后的 record 的大小为 7.6 GB。

3. 将数据包复制到 `/apollo/data/bag` 目录下。

   为了在 Docker 中可以访问 record，将下载的数据包复制到 `/apollo/data/bag` 目录下`。

4. 进入 docker 环境后，播放数据包。

   ```text
   cyber_recorder play -f /path/to/your_record
   ```

5. 查看 channel 信息。

   在 docker 环境下，用 cyber_monitor 命令，查看激光雷达相关 channel 是否正常输出，并使用上下方向键选择 channel，使用右方向键查看 channel 详细数据：

   ```text
   cyber_monitor
   ```

   启动之后，请检查如下 channel 是否正常输出，绿色代表正常输出：

   ![2021-07-09 19-06-23屏幕截图.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/2021-07-09%2019-06-23%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE_2ca9de4.png)

   如果激光雷达感知模块启动成功，检测的障碍物信息会输出在 `/apollo/perception/obstacles` 通道里面，并在 Dreamview 上面可视化障碍物信息：

   ![2021-06-17 20-24-42屏幕截图.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/2021-06-17%2020-24-42%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE_8b4c842.png)

   ![2021-06-17 20-25-20屏幕截图.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/2021-06-17%2020-25-20%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE_b42870c.png)

   如果以上步骤都正常执行，则可以在 Dreamview 中看见下面的画面。

   ![Screenshot from 2021-07-14 12-42-23.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/Screenshot%20from%202021-07-14%2012-42-23_6c87dd5.png)

## 步骤五：点云可视化

如果您想对点云进行可视化的查看，可以利用 CyberVisualizer 工具。CyberVisualizer 是基于 Cyber 开发的传感器原始数据可视化工具。

1. 在进入 Docker 环境后，在 terminal 输入如下命令启动 CyberVisualizer。

   ```text
   cyber_visualizer
   ```

2. 点击左上角的 **Show PointCloud**。

3. 在左侧 **Title** 中，双击 **PointCloud2**，出现 **ChannelName** 和 **Action**。

4. 点击 **Action** 后的 **Play**。

   点击 **Play** 后，可以看见以下画面。

   ![Screenshot from 2021-07-13 21-18-12.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/Screenshot%20from%202021-07-13%2021-18-12_dccaee5.png)

   您可以利用鼠标对点云的视角进行放大、缩小等操作，来仔细查看点云的变化。激光雷达的感知就是对这些点云进行处理，最终实现目标识别。