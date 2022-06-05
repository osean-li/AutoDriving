# 使用CyberVisualizer查看原始感知数据实践

更新时间：2021-11-25

`cyber_visualizer` 是一款用来显示 channel 数据的可视化工具，本文以传感器原始数据包 `demo_sensor_data.tar.xz` 为例介绍如何使用 `cyber_visualizer` 查看点云和相机数据。

## 步骤一：安装环境

`cyber_visualizer` 需要运行在 Apollo Docker 环境内，且依赖于 Cyber RT 软件库。使用前，您需要通过如下方式来配置 Cyber RT 工具的运行环境。



```bash
username@computername:~$: source /apollo/cyber/setup.bash
```

## 步骤二：进入Docker环境

在安装的根路经下输入以下命令，进入 Apollo 的 Docker 环境。

```text
./apollo.sh
```

如果您的终端如下面所示，说明您成功进入了 Apollo 的运行容器：

```text
[user@in-runtime-docker:/apollo]$ 
```

## 步骤三：下载并解压数据包

1. 在 Apollo Docker 环境中下载数据包。

   ```text
   wget https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz
   ```

2. 解压数据包。

   ```text
   tar -xvf demo_sensor_data.tar.xz
   ```

## 步骤四：播放数据包

解压后，在 Apollo Docker 环境中执行如下命令运行 `cyber_recorder`。

```text
cyber_recorder play -f sensor_rgb.record -loop
```

## 步骤五：运行Cyber_visualizer

重新打开一个终端，在 Apollo Docker 环境中执行如下命令运行 `cyber_visualizer`：



```bash
username@computername:~$: cyber_visualizer
```

启动 `cyber_visualizer` 后，您可以看到如下界面：

![cyber_visualizer1.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/cyber_visualizer1_a3106da.png)

## 步骤六：查看原始感知数据

播放数据包后，`cyber_visualizer` 会收到所有活动 channel 的信息（来自回放数据）并显示出来。当有数据流经 channel 时，在 `ChannelNames` 下会显示 channel 列表。您可以查看点云和相机数据。

![cyber_visualizer2.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/cyber_visualizer2_ea6fafb.png)

单击工具栏中的选项，您可以开启参考网格、显示点云、添加图像或同时显示多个相机数据的功能。如果启用了 **显示网格** 选项，则可以双击 **ChannelName** 下方的 **Grid** 列表中的 **Color** 项来设置网格的颜色。默认颜色为灰色。

您还可以编辑 **CellCount** 的值来调整网格中的单元格数量。对于点云或图像，可以通过 **ChannelName** 和 **Action** 选项选择源 channel 来播放或停止来自相应 channel 的数据。

- 下图是实际车辆在道路上运行时的 `cyber_visualizer` 画面，下方的三个摄像机 channel 数据和顶部的一个点云 channel 数据同时显示。

  ![cyber_visualizer3.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/cyber_visualizer3_c280326.png)

- 要在 3D 点云场景中调整虚拟摄像机，可以在点云显示部分右键单击，将弹出一个对话框，如下图所示：

  ![cyber_visualizer4.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/cyber_visualizer4_4cea268.png)

- 点云场景支持两种类型的虚拟相机：自由型和目标型（从上面的弹出对话框中选择类型）。

  - **自由型相机**：对于点云场景中的这种类型的相机，您可以按住鼠标左键或右键并移动来改变相机的姿态。要更改相机的间距，您可以滚动鼠标滚轮。
  - **目标类型相机**：对于点云场景中的这类相机，要改变相机的视角，可以按住鼠标左键再移动。要更改相机到观察点的距离（默认观察点为坐标系原点 (0,0,0)），可以滚动鼠标滚轮。 也可以直接在对话框中修改相机信息，改变相机在点云场景中的观察状态。**Step** 是对话框中的步长值。

- 将鼠标放在摄像机 channel 的图像上，可以双击左键在左侧菜单栏突出显示对应的channel。右击图像，可以弹出删除摄像机通道的菜单。

- 播放和暂停：单击 **Play** 时，将显示所有频道。当单击 **Stop** 时，所有 channel 信息将停止显示。