# 使用CyberRecorder播放数据包

更新时间：2021-11-25

`cyber_recorder` 是 Apollo Cyber RT 提供的播放工具，可以用来播放 record 文件。Record 是 Apollo 记录数据的一种数据格式。以 `.record` 为后缀的文件就是我们说的 record 数据包。本文以传感器原始数据包 `demo_sensor_data.tar.xz` 为例介绍如何使用 `cyber_recorder` 播放数据包。

## 步骤一：安装环境

`cyber_recorder` 需要运行在 Apollo Docker 环境内，且依赖于 Cyber RT 软件库。使用前，您需要通过如下方式来配置 Cyber RT 工具的运行环境。

```text
username@computername:~$: source /apollo/cyber/setup.bash
```

## 步骤二：进入Docker环境

在安装的根路经下输入命令，进入 Apollo 的 Docker 环境。

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

1. 解压数据包。

```text
tar -xvf demo_sensor_data.tar.xz
```

由于文件较大，请您耐心等待解压。解压后的文件为 `sensor_rgb.record`。

## 步骤四：播放数据包

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