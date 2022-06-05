# 0 概览



上机实践教程主要介绍 Apollo Cyber RT 的使用和上机实践 Apollo 各个模块操作。

## Apollo Cyber RT

Apollo Cyber RT 是专为自动驾驶场景而设计的高性能运行时框架，它提供了以下开发工具来辅助日常开发，您可以使用我们提供的数据包进行实践。

| 开发工具         | 说明                                                    | 相关文档                                                     | 测试数据包地址                                               |
| ---------------- | ------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| cyber_recorder   | 命令行工具，用来播放数据包。                            | [使用CyberRecorder播放数据包](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/实时通信框架CyberRT的使用/使用CyberRecorder播放数据包) | [demo_sensor_data.tar.xz](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz) |
| cyber_monitor    | 命令行工具，可以在终端中实时显示 channel 信息列表。     | [使用CyberMonitor查看Channel数据](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/实时通信框架CyberRT的使用/使用CyberMonitor查看Channel数据实践) | [demo_sensor_data.tar.xz](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz) |
| cyber_visualizer | 显示 channel 数据的可视化工具，可以查看点云和相机数据。 | [使用CyberVisualizer查看原始感知数据](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/实时通信框架CyberRT的使用/使用CyberVisualizer查看原始感知数据实践) | [demo_sensor_data.tar.xz](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz) |

## 上机实践教程

上机实践教程介绍 Apollo 视觉感知、激光雷达感知、感知融合、定位等模块的实践。

| 上机实践教程                     | 相关文档                                                     | 测试数据包地址                                               |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 上机实践 Apollo 视觉感知能力     | [Apollo视觉感知模块实践](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo视觉感知能力/Apollo视觉感知模块实践) | [demo_sensor_data_for_vision.tar.xz](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data_for_vision.tar.xz) |
| 上机实践 Apollo 激光雷达感知能力 | [使用Dreamview调试Apollo激光雷达感知实践](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo激光雷达感知能力/使用Dreamview调试Apollo激光雷达感知实践) | [demo_sensor_data.tar.xz](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz) |
| 上机实践 Apollo 感知融合能力     | [Apollo感知融合模块实践](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo感知融合能力/Apollo感知融合模块实践) | [demo_sensor_data_for_vision.tar.xz](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data_for_vision.tar.xz) |
| 上机实践 Apollo 定位能力         | [使用MSFLocalizer的多传感器融合定位实践](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo定位能力/使用MSFLocalizer的多传感器融合定位实践) [基于MSFVisualizer的Apollo定位可视化实践](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo定位能力/基于MSFVisualizer的Apollo定位可视化实践) | [demo-localization-data-3.5.tar.gz](https://apollo-system.cdn.bcebos.com/dataset/localization/demo-localization-data-3.5.tar.gz) |
| 上机实践 Apollo 规划能力         | [Apollo规划模块实践](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo规划能力/apollo规划模块实践) | [demo_3.5.record](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_3.5.record) |
| 上机实践 Apollo 预测能力         | [Apollo预测模块实践](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo预测能力/Apollo预测模块实践) | [demo_3.5.record](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_3.5.record) |
| 上机实践 Apollo 控制能力         | [基于Dreamland调试控制能力实践](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/上机实践Apollo控制能力/基于Dreamland调试控制能力实践) | [demo_3.5.record](https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_3.5.record) |

# 1 实时通信框架CyberRT的使用

## 1.1 CyberRT介绍

更新时间：2021-11-23

Apollo Cyber RT 是一个开源、高性能的运行时框架，专为自动驾驶场景而设计。针对自动驾驶的高并发、低延迟、高吞吐量进行了大幅优化。本文介绍 Cyber RT 优势，开发工具以及常用术语。

## 优势

Apollo Cyber RT 具有以下优势：

### 加速开发

- 具有数据融合功能的定义明确的任务接口，
- 一系列开发工具，
- 大量传感器驱动程序。

### 简化部署

- 高效自适应的消息通信，
- 具有资源意识的可配置用户级调度程序，
- 可移植，依赖更少。

### 为自动驾驶赋能

- 默认的开源运行时框架，
- 为自动驾驶搭建专用模块。

## 开发工具

Cyber RT 框架同时也提供了以下工具用来辅助日常开发，包括：

- 可视化工具 `cyber_visualizer`。详情参见 [使用 CyberVisualizer 查看原始感知数据](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/实时通信框架CyberRT的使用/使用CyberVisualizer查看原始感知数据实践)。
- 命令行工具 `cyber_monitor` 和 `cyber_recorder`。详情参见 [使用 CyberMonitor 查看 Channel 数据](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/实时通信框架CyberRT的使用/使用CyberMonitor查看Channel数据实践) 和 [使用CyberRecorder播放数据包](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/实时通信框架CyberRT的使用/使用CyberRecorder播放数据包)。

这些工具需要运行在 Apollo Docker 环境内，且依赖于 Cyber RT 软件库。使用这些工具前，您需要通过如下方式来配置 Cyber RT 工具的运行环境：



```bash
username@computername:~$: source /apollo/cyber/setup.bash
```

## 常用术语

### Component

在自动驾驶系统中，模块（如感知、定位、控制系统等）在 Cyber RT 下以 Component 的形式存在。不同 Component 之间通过 Channel 进行通信。Component 概念不仅解耦了模块，还为将模块拆分为多个子模块提供了灵活性。

### Channel

Channel 用于管理 Cyber RT 中的数据通信。用户可以发布/订阅同一个 Channel，实现 P2P 通信。

### Task

Task 是 Cyber RT 中异步计算任务的抽象描述。

### Node

Node 是 Cyber RT 的基本组成部分。每个模块都包含一个 Node 并通过 Node 进行通信。通过在节点中定义 Reader/Writer 或 Service/Client，模块可以具有不同类型的通信形式。

### Reader/Writer

Reader/Writer 通常在 Node 内创建，作为 Cyber RT 中的主要消息传输接口。

### Service/Client

除 Reader/Writer 外，Cyber RT 还提供了用于模块通信的 Service/Client 模式。它支持节点之间的双向通信。当对服务发出请求时，客户端节点将收到响应。

### Parameter

参数服务在 Cyber RT 中提供了全局参数访问接口。它是基于 Service/Client 模式构建的。

### 服务发现

作为一个去中心化的框架，Cyber RT 没有用于服务注册的主/中心节点。所有节点都被平等对待，可以通过“服务发现”找到其他服务节点。使用 `UDP` 用来服务发现。

### CRoutine

参考协程（Coroutine）的概念，Cyber RT 实现了 Coroutine 来优化线程使用和系统资源分配。

### Scheduler

为了更好地支持自动驾驶场景，Cyber RT 提供了多种资源调度算法供开发者选择。

### Message

Message 是 Cyber RT 中用于模块之间数据传输的数据单元。

### Dag 文件

Dag 文件是模块拓扑关系的配置文件。您可以在 dag 文件中定义使用的 Component 和上游/下游通道。

### Launch 文件

Launch 文件提供了一种启动模块的简单方法。通过在 launch 文件中定义一个或多个 dag 文件，可以同时启动多个模块。

### Record 文件

Record 文件用于记录从 Cyber RT 中的 Channel 发送/接收的消息。回放 Record 文件可以帮助重现 Cyber RT 之前操作的行为。

## 1.2 使用CyberRecorder播放数据包

更新时间：2021-11-25

`cyber_recorder` 是 Apollo Cyber RT 提供的播放工具，可以用来播放 record 文件。Record 是 Apollo 记录数据的一种数据格式。以 `.record` 为后缀的文件就是我们说的 record 数据包。本文以传感器原始数据包 `demo_sensor_data.tar.xz` 为例介绍如何使用 `cyber_recorder` 播放数据包。

### 步骤一：安装环境

`cyber_recorder` 需要运行在 Apollo Docker 环境内，且依赖于 Cyber RT 软件库。使用前，您需要通过如下方式来配置 Cyber RT 工具的运行环境。

```text
username@computername:~$: source /apollo/cyber/setup.bash
```

### 步骤二：进入Docker环境

在安装的根路经下输入命令，进入 Apollo 的 Docker 环境。

```text
./apollo.sh
```

如果您的终端如下面所示，说明您成功进入了 Apollo 的运行容器：

```text
[user@in-runtime-docker:/apollo]$ 
```

### 步骤三：下载并解压数据包

1. 在 Apollo Docker 环境中下载数据包。

```text
wget https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz
```

1. 解压数据包。

```text
tar -xvf demo_sensor_data.tar.xz
```

由于文件较大，请您耐心等待解压。解压后的文件为 `sensor_rgb.record`。

### 步骤四：播放数据包

解压后，在 Apollo Docker 环境中执行如下命令运行 `cyber_recorder`。

```text
cyber_recorder play -f /path/to/your_record
```

本文中为：

```text
cyber_recorder play -f sensor_rgb.record -loop
```

选项 --loop 用于设置循环回放模式。

播放数据包后，您可以使用 `cyber_monitor` 工具查看 channel 信息，参见 [使用 CyberMonitor 查看 Channel 数据](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/实时通信框架CyberRT的使用/使用CyberMonitor查看Channel数据实践) 。

您还可以使用 `cyber_visualizer` 查看播放效果，参见 [使用 CyberVisualizer 查看原始感知数据](https://apollo.auto/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/上机使用教程/实时通信框架CyberRT的使用/使用CyberVisualizer查看原始感知数据实践)。

## 1.3 使用CyberMonitor查看Channel数据实践

更新时间：2021-11-25

命令行工具 `cyber_monitor` 提供了终端中实时显示 channel 信息列表的功能。

本文以传感器原始数据包 `demo_sensor_data.tar.xz` 为例介绍如何使用 `cyber_monitor` 来查看消息。

### 步骤一：安装环境

`cyber_monitor` 需要运行在 Apollo Docker 环境内，且依赖于 Cyber RT 软件库。使用前，您需要通过如下方式来配置 Cyber RT 工具的运行环境。



```bash
username@computername:~$: source /apollo/cyber/setup.bash
```

### 步骤二：进入Docker环境

在安装的根路经下输入命令，进入 Apollo 的 Docker 环境。



```bash
./apollo.sh
```

如果您的终端如下面所示，说明您成功进入了 Apollo 的运行容器：

```text
[user@in-runtime-docker:/apollo]$ 
```

### 步骤三：下载并解压数据包

1. 在 Apollo Docker 环境中下载数据包。

```text
wget https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz
```

1. 解压数据包。

```text
tar -xvf demo_sensor_data.tar.xz
```

### 步骤四：播放数据包

解压后，在 Apollo Docker 环境中执行如下命令运行 `cyber_recorder`。

```text
cyber_recorder play -f sensor_rgb.record -loop
```

### 步骤五：查看 channel 数据

打开新的终端，在 Apollo Docker 环境中输入以下命令，启动 `cyber_monitor`。

```text
username@computername:~$: cyber_monitor
```

`cyber_monitor` 启动之后，会在终端中显示一个交互界面。`cyber_monitor` 自动从拓扑中收集所有 channel 信息并分两列显示：分别为 **Channels**（channel 名称）和 **FrameRatio**（数据频率）。

channel 信息默认显示为红色，当有数据流经 channel 时，对应的行就会显示成绿色。

![cyber_monitor.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/cyber_monitor_4365395.png)

### 命令行参数

- 使用 `-h` 选项，获取帮助信息：

  

  ```bash
  username@computername:~$: cyber_monitor -h
  ```

- 使用 `-c` 选项，可以让 `cyber_monitor` 监测一个指定的 channel 信息：

  

  ```bash
  username@computername:~$: cyber_monitor -c ChannelName
  ```

- 其它常用命令介绍：

  ```text
  ESC | q key ---- 退出
  Backspace ---- 后退
  h | H ---- 显示帮助页
  PageDown | Ctrl+d ---- 上一页
  PageUp | Ctrl+u ---- 下一页
  Up, down or w, s keys ---- 上下移动当前的高亮行
  Right arrow or d key ---- 进入高亮行, 显示高亮行数据的详细信息
  Left arrow or a key ---- 从当前界面返回上一层界面
  Enter key ---- 与 d 键相同
  f | F ---- 显示数据帧频率
  t | T ---- 显示 channel 消息类型
  Space ---- 关闭|开启 channel (仅在 channel 有数据到达时有效；channel 关闭后会变成黄色)
  i | I ---- 显示 channel 的 Reader 和 Writer 信息
  b | B ---- 显示 channel 消息内容
  n | N ---- 显示消息中 RepeatedField 的下一条数据
  m | M ---- 显示消息中 RepeatedField 的上一条数据
  ```

## 1.4使用CyberVisualizer查看原始感知数据实践



`cyber_visualizer` 是一款用来显示 channel 数据的可视化工具，本文以传感器原始数据包 `demo_sensor_data.tar.xz` 为例介绍如何使用 `cyber_visualizer` 查看点云和相机数据。

### 步骤一：安装环境

`cyber_visualizer` 需要运行在 Apollo Docker 环境内，且依赖于 Cyber RT 软件库。使用前，您需要通过如下方式来配置 Cyber RT 工具的运行环境。



```bash
username@computername:~$: source /apollo/cyber/setup.bash
```

### 步骤二：进入Docker环境

在安装的根路经下输入以下命令，进入 Apollo 的 Docker 环境。

```text
./apollo.sh
```

如果您的终端如下面所示，说明您成功进入了 Apollo 的运行容器：

```text
[user@in-runtime-docker:/apollo]$ 
```

### 步骤三：下载并解压数据包

1. 在 Apollo Docker 环境中下载数据包。

   ```text
   wget https://apollo-system.cdn.bcebos.com/dataset/6.0_edu/demo_sensor_data.tar.xz
   ```

2. 解压数据包。

   ```text
   tar -xvf demo_sensor_data.tar.xz
   ```

### 步骤四：播放数据包

解压后，在 Apollo Docker 环境中执行如下命令运行 `cyber_recorder`。

```text
cyber_recorder play -f sensor_rgb.record -loop
```

### 步骤五：运行Cyber_visualizer

重新打开一个终端，在 Apollo Docker 环境中执行如下命令运行 `cyber_visualizer`：



```bash
username@computername:~$: cyber_visualizer
```

启动 `cyber_visualizer` 后，您可以看到如下界面：

![cyber_visualizer1.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/cyber_visualizer1_a3106da.png)

### 步骤六：查看原始感知数据

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

