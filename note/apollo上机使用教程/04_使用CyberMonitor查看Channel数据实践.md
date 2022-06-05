# 使用CyberMonitor查看Channel数据实践

更新时间：2021-11-25

命令行工具 `cyber_monitor` 提供了终端中实时显示 channel 信息列表的功能。

本文以传感器原始数据包 `demo_sensor_data.tar.xz` 为例介绍如何使用 `cyber_monitor` 来查看消息。

## 步骤一：安装环境

`cyber_monitor` 需要运行在 Apollo Docker 环境内，且依赖于 Cyber RT 软件库。使用前，您需要通过如下方式来配置 Cyber RT 工具的运行环境。



```bash
username@computername:~$: source /apollo/cyber/setup.bash
```

## 步骤二：进入Docker环境

在安装的根路经下输入命令，进入 Apollo 的 Docker 环境。



```bash
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

## 步骤四：播放数据包

解压后，在 Apollo Docker 环境中执行如下命令运行 `cyber_recorder`。

```text
cyber_recorder play -f sensor_rgb.record -loop
```

## 步骤五：查看 channel 数据

打开新的终端，在 Apollo Docker 环境中输入以下命令，启动 `cyber_monitor`。

```text
username@computername:~$: cyber_monitor
```

`cyber_monitor` 启动之后，会在终端中显示一个交互界面。`cyber_monitor` 自动从拓扑中收集所有 channel 信息并分两列显示：分别为 **Channels**（channel 名称）和 **FrameRatio**（数据频率）。

channel 信息默认显示为红色，当有数据流经 channel 时，对应的行就会显示成绿色。

![cyber_monitor.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/cyber_monitor_4365395.png)

## 命令行参数

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