# apollo规划能力介绍

更新时间：2022-01-07

规划是自动驾驶中重要的模块。规划模块基于预测模块输入的障碍物信息、地图定位信息、路径导航信息对车辆的未来的运动轨迹进行规划，保证自动驾驶车辆高效、安全、舒适的行至目标位置。

本文档主要介绍规划模块的基本原理以及输入输出，帮助开发者快速了解规划模块的运作机制，方便开发者开展后续的模块开发任务

## Apollo 规划能力介绍

Apollo 规划模块的主要作用是结合障碍物、地图定位以及导航信息为自动驾驶车辆规划一条运动轨迹，这条轨迹由若干轨迹点组成，每个轨迹点均包含了位置坐标、速度、加速度、加加速度、相对时间等信息，这些信息为自动驾驶车辆的运动提供依据，参照规划的轨迹，自动驾驶车辆可以高效、安全、舒适的驶向目的地。

## Apollo 规划模块原理

Apollo 规划模块功能的实现是基于场景（scenario-based）实现的，针对不同的场景，规划模块通过一系列独立的 **任务（task）** 组合来完成轨迹的规划。开发者可以根据自己的使用需求，调整 `apollo/modules/planning/conf/scenario/` 下的配置文件，调配任务组合实现自己的规划要求。

![architecture-planning.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/architecture-planning_33f0ce0.png)

Apollo 规划架构示意图如上，其中部分重要模块如下：

- 状态机（Apollo FSM（Finite State Machine））：一个有限状态机，结合导航、环境等信息确定自动驾驶车辆的驾驶场景
- 规划分发器（Planning Dispatcher）：根据状态机与车辆相关信息，调用合适当前场景的规划器
- 规划器（Planner）：结合上游模块信息，通过一系列的任务组合，完成自动驾驶车辆的轨迹规划
- 决策器 & 优化器（Deciders & Optimizers）：一组实现决策和优化任务的 task 集合。优化器用于优化车辆的轨迹和速度。决策器则基于规则，确定自动驾驶车辆何时换车道、何时停车、何时蠕行（慢速行进）或蠕行何时完成等驾驶行为。

## 规划模块的输入输出

### 规划模块的输入

| channel 名称                     | 输入 | channel 说明      |
| -------------------------------- | ---- | ----------------- |
| /apollo/canbus/chassis           | 输入 | 车身底盘反馈信息  |
| /apollo/localization/pose        | 输入 | 车辆定位信息      |
| /apollo/perception/traffic_light | 输入 | 感知红绿灯信息    |
| /apollo/prediction               | 输入 | 预测障碍物信息    |
| /apollo/relative_map             | 输入 | 局部地图信息      |
| /apollo/routing_response         | 输入 | 导航 routing 信息 |



### 规划模块的输出

| channel 名称     | 输出 | channel 说明           |
| ---------------- | ---- | ---------------------- |
| /apollo/planning | 输出 | 自动驾驶车辆的轨迹信息 |

# apollo规划模块实践

更新时间：2021-11-11

## 一、概览

本文主要介绍调试规划模块的基本方法，帮助开发者快速上手，修改调试符合自己使用需求的规划模块

## 二、进入 docker 环境并启动Dreamview

1. 首先进入 apollo docker 环境，并启动Dreamview平台。

进入 apollo docker 环境

```text
./apollo.sh
```

接着在 docker 环境里，启动 Dreamview

```text
./scripts/bootstrap.sh
```

注：如果需要关闭Dreamview则执行命令

```text
./scripts/bootstrap.sh stop
```

1. 在浏览器输入网址，打开Dreamview

```text
http://localhost:8888/
```

1. 在Dreamview的右上角的 --vehicle--选择对应的车型（教程使用mkz_example）, --map--选择对应的地图（教程使用sunnyvale loop）。车辆选择后，程序会把会把calibration/data目录下对应的车型参数和传感器内外参文件拷贝到作用目录下。

## 三、在Dreamview上本地调试规划模块

当使用Apollo时遇到规划模块的问题需要调试代码时，可以基于本地环境进行初步验证，通常的做法是本地播放record，屏蔽record中原有的规划消息，观察调试后带来的效果，具体的步骤如下：

- 播放数据包（`docs/demo_guide/demo_3.5.record`为record路径，通过`-k /apollo/planning`屏蔽原有的planning消息）：

```text
cyber_recorder play -f docs/demo_guide/demo_3.5.record -k /apollo/planning
```

- 打开planning按钮，观察可视化结果：图中主车为图像中间的蓝色车辆，主车左侧的绿色几何体为障碍物，穿过主车的浅蓝色曲线即自动驾驶车辆的规划轨迹，可以观察轨迹的状态定性的分析规划问题；如果默认视角看不清可视化结果，可以在Layer Menu中切换视角至near，方便近距离观察
- 进一步精细化分析规划问题，可以打开PNC monitor，界面右侧会显示路径规划和速度规划的相关定量图线，供开发者做进行的分析和问题定位

![123323.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/123323_2226332.png)