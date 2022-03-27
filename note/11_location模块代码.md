## 定位模块: Localization

定位模块从不同的信息源，如GPS(经纬度)，LiDAR(距离)和IMU(加速度等)来评估车辆当前的信息。常用的定位技术有两类：

- 基于RTK(Real-Time Kinematic, 实时动态载波相位差分技术)定位。由OnTimer函数以一定频率触发定位。
- 多传感器融合(Multiple Sensor Fusion, MSF)定位。由一系列传感器触发函数触发。

(1) 定位模块输入数据

- RTK-base method.
  - GPS - Global Position System/全球定位系统
  - IMU - Interial Measurement Unit/惯性测量单元
- MSF method.
  - GPS - Global Position System/全球定位系统
  - IMU - Interial Measurement Unit/惯性测量单元
  - LiDAR - Light Detection And Ranging Sensor/光检测和测距传感器，激光雷达

## **Localization模块简介**

localization模块主要实现了以下2个功能：

```text
1.输出车辆的位置信息（planning模块使用）
2.输出车辆的姿态，速度信息（control模块使用）
```

其中apollo代码中分别实现了3种定位方法：

```text
1. GNSS + IMU定位
2. NDT定位（点云定位）
3. MSF（上面2种方式的融合定位）
```

> MSF方法并没有开发源码，是以动态库的方式提供的。实现的思路可以参考论文"Robust and Precise Vehicle Localization Based on Multi-Sensor Fusion in Diverse City Scenes"

## **目录**

下面是localization的目录结构，看之前最好看下该模块的readme文件

```text
├── common          // 声明配置(flags)，从conf目录中读取相应的值
├── conf            // 配置文件存放目录
├── dag             // cyber DAG流
├── launch          // cyber的配置文件，依赖DAG图（这2个和cyber有关的后面再分析）
├── msf             // 融合定位（gnss,点云,IMU融合定位）
│   ├── common
│   │   ├── io
│   │   ├── test_data
│   │   └── util
│   ├── local_integ
│   ├── local_map
│   │   ├── base_map
│   │   ├── lossless_map
│   │   ├── lossy_map
│   │   ├── ndt_map
│   │   └── test_data
│   ├── local_tool
│   │   ├── data_extraction
│   │   ├── local_visualization
│   │   └── map_creation
│   └── params
│       ├── gnss_params
│       ├── vehicle_params
│       └── velodyne_params
├── ndt                         // ndt定位
│   ├── map_creation
│   ├── ndt_locator
│   └── test_data
│       ├── ndt_map
│       └── pcds
├── proto                   // 消息格式
├── rtk                     // rtk定位
└── testdata                // imu和gps的测试数据
```

可以看到，主要是rtk，ndt，msf这3个目录分别代表了不同的定位方法，而proto是消息的格式定义，common和conf主要是存放一些配置和消息TOPIC。下面我们逐个分析各个模块。



## **RTK**

可以看到rtk模块相对比较简单，**目录结构如下：**

```text
├── BUILD                               // bazel编译文件
├── rtk_localization.cc                 // rtk定位功能实现模块
├── rtk_localization_component.cc       // rtk消息发布模块
├── rtk_localization_component.h
├── rtk_localization.h
└── rtk_localization_test.cc            // 测试
```

**主要流程如下：**

![image-20220327170308749](D:\自动驾驶\AutoDriving\note\image-20220327170308749.png)

在[http://rtk_localization_component.cc](https://link.zhihu.com/?target=http%3A//rtk_localization_component.cc)中可以看到

```text
RTKLocalizationComponent::RTKLocalizationComponent()
    : localization_(new RTKLocalization()) {}
```

即class RTKLocalization()实际上是RTKLocalizationComponent()中的一个属性localization_，我们可以发现apollo模块的架构大部分都是这样，**一个模块负责发布接收消息，一个模块负责实现具体的功能**，后面的模块在前面的模块中注册为一个**属性**。

其中"rtk_localization_component.cc"注册为标准的cyber模块，RTK定位模块在"Init"中初始化，每当接收到"localization::Gps"消息就触发执行"Proc"函数。

```
class RTKLocalizationComponent final
    : public cyber::Component<localization::Gps> {
 public:
  RTKLocalizationComponent();
  ~RTKLocalizationComponent() = default;

  bool Init() override;

  bool Proc(const std::shared_ptr<localization::Gps> &gps_msg) override;
```

其中"rtk_localization_component.cc"注册为标准的cyber模块，RTK定位模块在"Init"中初始化，每当接收到"localization::Gps"消息就触发执行"Proc"函数。

```
class RTKLocalizationComponent final
    : public cyber::Component<localization::Gps> {
 public:
  RTKLocalizationComponent();
  ~RTKLocalizationComponent() = default;

  bool Init() override;

  bool Proc(const std::shared_ptr<localization::Gps> &gps_msg) override;
```

下面我们分别查看这2个函数。

1. Init函数
   Init函数实现比较简单，一是初始化配置信息，二是初始化IO。初始化配置信息主要是读取一些配置，例如一些topic信息等。下面主要看下初始化IO。

   ![image-20220327180616690](D:\自动驾驶\AutoDriving\note\picture\image-20220327180616690.png)
   
2. Proc
   在每次接收到"localization::Gps"消息后，触发执行"Proc"函数。这里注意如果需要接收多个消息，这里是3个消息，则选择最慢的消息作为触发，否则，如果选择比较快的消息作为触发，这样会导致作为触发的消息刷新了，而其它的消息还没有刷新。所以这里采用的是GPS消息作为触发消息（5hz），IMU的消息刷新快（100hz）。下面我们看具体的实现。



主要的执行过程在"GpsCallback"中，然后通过"GetLocalization"和"GetLocalizationStatus"获取结果，最后发布对应的位置信息、位置转换信息和位置状态信息。

#### 获取定位信息

**PrepareLocalizationMsg**

功能如下

> ```
> 1.寻找最匹配的IMU信息  
> 2.根据GPS和IMU信息，给位置信息赋值
> 3.查找最近的GPS状态信息
> 4.根据GPS状态信息，给位置状态信息赋值
> ```

#### FindMatchingIMU

```
// 1.寻找最匹配的IMU信息 
```

实现方法

1. 遍历imu_list，获取imu时间（最新的IMU时间）和gps时间最近的`header().timestamp_sec() - gps_timestamp_sec`

2. 根据最新的IMU消息和它之前的消息做插值InterpolateIMU

3. InterpolateIMU根据上述函数得到2个IMU消息分别对角速度、线性加速度、欧拉角进行插值。原则是根据比例，反比例进行插值。
   1. InterpolateXYZ：**Interpolate插值函数**根据距离插值，反比例，即frac1越小，则越靠近p1，frac1越大，则越靠近p2；
   2. 分别对角速度、线性加速度、欧拉角进行插值

**ComposeLocalizationMsg**

FindMatchingIMU 获取的imu_msg作为其输入之一

apollo::localization::Pose 的数据结构

- position 获取位置

- orientation 获取方向

- linear_velocity 获取线性速度

- linear_acceleration 线性加速度

- angular_velocity 角速度，也需要根据航向转换

- linear_acceleration_vrf

- angular_velocity_vrf

- euler_angles 欧拉角

- heading_

  



![image-20220327194332285](D:\自动驾驶\AutoDriving\note\picture\image-20220327194332285.png)

```
// 2.根据GPS和IMU信息，给位置信息赋值
```

填充位置信息，这里实际上涉及到姿态解算，具体是根据GPS和IMU消息对位置信息进行赋值。需要注意需要根据航向对IMU的信息进行转换。

1. 获取位置 // world frame -> map frame，map_offset_从配置文件里读取
2.  获取方向
3.  获取速度
4. 获取imu的线性加速度
5. 设置角速度，也需要根据航向转换
6. 设置欧拉角

**FindNearestGpsStatus**

```
// 3.查找最近的GPS状态信息
```

获取最近的Gps状态信息，这里实现的算法是遍历查找离"gps_time_stamp"最近的状态

**FillLocalizationStatusMsg**

```
// 4.根据GPS状态信息，给位置状态信息赋值
```

获取位置状态，一共有3种状态：稳定状态(INS_RTKFIXED)、浮动状态(INS_RTKFLOAT)、错误状态(ERROR)

ins_status_ = 3? pos_type=56 ?



#### 发布消息

最后通过以下几个函数发布消息。

1. PublishPoseBroadcastTopic // 发布位置信息
2. PublishPoseBroadcastTF // 发布位置转换transform信息
3. PublishLocalizationStatus // 发布位置状态信息

# 参考

https://github.com/daohu527/dig-into-apollo/tree/main/modules/localization#findmatchingimu

