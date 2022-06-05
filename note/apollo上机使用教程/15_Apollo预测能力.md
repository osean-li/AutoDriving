# Apollo预测能力介绍

更新时间：2021-11-16

本文为开发者介绍 Apollo 预测模块的基本知识，包括预测模块在自动驾驶系统中的主要作用，预测模块的基本组成以及预测模块的输入输出。

## Apollo 预测能力介绍

预测模块通过障碍物的历史状态信息，来预测障碍物的未来轨迹。感知模块作为预测模块的上游，提供障碍物的位置、朝向、速度、加速度等信息，预测模块根据这些信息，给出障碍物未来的预测轨迹，供下游规划模块进行自车轨迹的规划。

### 预测模块的原理

预测模块主要有四个子模块组成，分别是：信息容器（container）、场景选择（scenario）、评估器（evaluator）和预测器（predictor）。

- 信息容器：储存上游信息，为之后的轨迹预测提供输入，当期储存的主要有：感知信息、定位信息以及自车轨迹规划信息。
- 场景选择：预测模块针对不同的场景采用不同的预测方法（如巡航、路口等场景），便于后续扩展，提高算法的泛化能力。
- 评估器：评估器基于障碍物的状态信息，结合预测模型，给出障碍物预测轨迹的概率或短预测时域的轨迹信息。
- 预测器：预测器直接或结合评估器的结果给出障碍物的完整预测时域的预测轨迹。

![prediction.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/prediction_bd8aa82.png)

### 预测模块的输入输出

| channel名称                  | 输入输出 | channel说明                                    |
| ---------------------------- | -------- | ---------------------------------------------- |
| /apollo/perception/obstacles | 输入     | 感知信息，包含障碍物的位置、朝向、速度、加速度 |
| /apollo/localization/pose    | 输入     | 定位信息，自车的位置、速度信息                 |
| /apollo/planning             | 输入     | 规划信息，自车规划的轨迹信息                   |
| /apollo/prediction           | 输出     | 预测轨迹，包含障碍物在预测时域内的未来轨迹信息 |

![prediction_infoflow.jpg](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/prediction_infoflow_2fd0e09.jpg)

# Apollo预测模块实践

更新时间：2021-11-11

## 一、概览

本文主要介绍调试预测模块的基本方法，帮助开发者快速上手，修改调试符合自己使用需求的预测模块

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
2. 在Dreamview上打开prediction按钮

## 三、在Dreamview上本地调试预测模块

当使用Apollo时遇到预测模块的问题需要调试代码时，可以基于本地环境进行初步验证，通常的做法是本地播放record，屏蔽record中原有的预测消息，观察调试后带来的效果，具体的步骤如下：

- 播放数据包（`docs/demo_guide/demo_3.5.record`为record路径，通过`-k /apollo/prediction`屏蔽原有的prediction消息）：

```text
cyber_recorder play -f docs/demo_guide/demo_3.5.record -k /apollo/prediction
```

- 打开prediction按钮，观察可视化结果：图中主车为图像中间的蓝色车辆，主车左侧的绿色几何体为障碍物，障碍物中绿色圆点连接而成的曲线即障碍物的预测轨迹，调试时可根据预测轨迹的形态，定位预测模块问题；如果默认视角看不清可视化结果，可以在Layer Menu中切换视角至near，方便近距离观察

![截屏2021-03-12 下午12.34.42.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/%E6%88%AA%E5%B1%8F2021-03-12%20%E4%B8%8B%E5%8D%8812.34.42_575cc00.png)