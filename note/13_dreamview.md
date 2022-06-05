# Dreamview简介



## 功能简介

Dreamview 是一个 web 应用程序，提供如下的功能：

- 可视化显示当前自动驾驶车辆模块的输出信息。例如：规划路径、车辆定位、车架信息等。
- 为使用者提供人机交互接口以监测车辆硬件状态，对模块进行开关操作，启动自动驾驶车辆等。
- 提供调试工具。例如：PnC 监视器可以高效的跟踪模块输出的问题。

不过，Dreamview 的大部分的功能，主要是为了在实际上车调试中使用的。

在快速入门中，只使用 Dreamview 的可视化功能将 record 数据包的数据可视化。

## 界面布局和特性

该应用程序的界面分为多个区域：标题、侧边栏、主视图和工具视图。

## 标题

标题包含 5 个下拉列表，可以如图所示进行操作：

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_438cb4d.png)

## 侧边栏和工具视图

侧边栏控制着显示在工具视图中的模块。

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_1b1477c.png)

### Tasks

在 DreamView 中，您可以操作的 tasks 有：

- **Quick Start**：当前选择的模式支持的指令。
  - **Setup**：开启所有模块。
  - **Reset All**：关闭所有模块。
  - **Start Auto**：开始车辆的自动驾驶。在实车中，**Start Auto** 就是让车辆进入自动驾驶模式的按钮。
- **Others**：工具经常使用的开关和按钮。
- **Module Delay**：从模块中输出的两次事件的时间延迟。
- **Console**：从 Apollo 平台输出的监视器信息。

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_b214e86.png)

### Module Controller

监视硬件状态和对模块进行开关操作。启动和停止这些模块功能，不仅可以用 Dreamview 上的这些可视化按钮启动，也可以通过命令行来启动。在后续的各个模块的实践中，您可以看到用命令行启动各个模块的说明。

> 注意：在本文档中，只是播放数据包，所以这些模块的开与关都不影响车辆的运行。

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_25f94cc.png)

### Layer Menu

显式控制各个元素是否显示的开关。

**Layer Menu** 中的功能，在本 Demo 演示中也是可以使用的。

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_6897a8e.png)

您可以在主视图中展示多种从 **Layer Menu** 选择的视图模式。

- 默认视图

  ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_280d2b9.png)

- 近距离视图

  ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_52b2844.png)

- 俯瞰视图

  ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_7f2fd85.png)

- 地图视图

  ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_a95100b.png)

  - 放大/缩小：滚动鼠标滚轮或使用两根手指滑动。
  - 移动：按下右键并拖拽或或使用三根手指滑动。

您可以通过在下面红框里的开关，开启和关闭对应的模块的可视化功能。

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_3920673.png)

### Route Editing

在向 **Routing** 模块发送寻路信息请求前，可以编辑路径信息的可视化工具。

如果您希望我们提供 Route Editing 相关的实践文档，请在我们的 [反馈意见收集问答页面](https://apollo.auto/developer/questiondata_cn.html?target=1354) 留言反馈。

### 主视图

主视图在 web 页面中以动画的方式展示 3D 计算机图形。

![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_72937a2.png)

下表列举了主视图中涉及到的各个元素。部分元素没有出现在本 Demo 中，仅供开发者参考。

| Visual Element                                               | Depiction Explanation                     |
| ------------------------------------------------------------ | ----------------------------------------- |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_04703c0.png) | 自动驾驶车辆。                            |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_ecd921a.png) | 车轮转动的比率。左右转向灯的状态。        |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_1dd14b2.png) | 交通信号灯状态。                          |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_5eda381.png) | 驾驶状态：AUTO，DISENGAGED，MANUAL 等。   |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_814318f.png) | 行驶速度 km/h。加速速率/刹车速率。        |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_f778b9b.png) | 红色粗线条表示建议的寻路路径。            |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_ff8f38a.png) | 轻微移动物体决策—橙色表示应该避开的区域。 |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_7f979cc.png) | 绿色的粗曲线条带表示规划的轨迹。          |



### 障碍物

| Visual Element                                               | Depiction Explanation                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_05b9d13.png) | 车辆障碍物。                                                 |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/e6993aa0909ff3aaa2e6ec4b9_e6993aa.png) | 行人障碍物。                                                 |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_fdf501f.png) | 自行车障碍物。                                               |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_c104fff.png) | 未知障碍物。                                                 |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_2fb0eec.png) | 速度方向显示了移动物体的方向，长度随速度按照比率变化。       |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_c01e6ee.png) | 白色箭头显示了障碍物的移动方向。                             |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_63e56a2.png) | 黄色文字表示：障碍物的跟踪 ID，自动驾驶车辆和障碍物的距离及障碍物速度。 |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_6d51572.png) | 线条显示了障碍物的预测移动轨迹，线条标记为和障碍物同一个颜色。 |



### Planning决策-决策栅栏区

决策栅栏区显示了 Planning 模块对车辆障碍物做出的决策。每种类型的决策会表示为不同的颜色和图标，如下图所示：

| Visual Element                                               | Depiction Explanation                         |
| ------------------------------------------------------------ | --------------------------------------------- |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_70480cb.png) | **停止**：表示物体主要的停止原因。            |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_cda3179.png) | **停止**：表示物体的停止原因。                |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_385f951.png) | **跟车**：物体。                              |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_7966735.png) | **让行**：物体决策—点状的线条连接了各个物体。 |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_ea789bb.png) | **超车**：物体决策—点状的线条连接了各个物体。 |

线路变更是一个特殊的决策，因此不显示决策栅栏区，而是将路线变更的图标显示在车辆上。

| Visual Element                                               | Depiction Explanation |
| ------------------------------------------------------------ | --------------------- |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_7e8bcf6.png) | 变更到左车道。        |
| ![img](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_7793b25.png) | 变更到右车道。        |

在优先通行的规则下，当在交叉路口的停车标志处做出让行决策时，被让行的物体在头顶会显示让行图标。

| Visual Element                                               | Depiction Explanation  |
| ------------------------------------------------------------ | ---------------------- |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_5df5f8e.png) | 停止标志处的让行物体。 |



### Planning决策-停止原因

如果显示了停止决策栅栏区，则停止原因展示在停止图标的右侧。可能的停止原因和对应的图标为：

| Visual Element                                               | Depiction Explanation  |
| ------------------------------------------------------------ | ---------------------- |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_1047bad.png) | 前方道路侧边区域。     |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_3e5da29.png) | 前方人行道。           |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_bf9b22a.png) | 到达目的地。           |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_8fe4288.png) | 紧急停车。             |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_c2bbe61.png) | 自动驾驶模式未准备好。 |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_b4fed97.png) | 障碍物阻塞道路。       |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_041dfc8.png) | 前方行人穿越。         |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_c58b185.png) | 黄/红信号灯。          |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_df8b018.png) | 前方有车辆。           |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_eab05c8.png) | 前方停止标志。         |
| ![image.png](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_88aa74f.png) | 前方让行标志。         |

# 参考

[Apollo](https://apollo.auto/document_cn.html?target=/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/)

