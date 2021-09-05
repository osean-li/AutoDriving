## Planing（计划）

### 模块介绍

Planing模块根据定位信息，车辆状态（位置，速度，加速度，底盘），地图，路线，感知和预测，计算出安全和舒适的形式线路让控制器执行。

目前的系统实现中包含了四种计划器：

- RTKReplayPlanner（自Apollo 1.0以来）：RTK重放计划器首先在初始化时加载记录的轨迹，并根据当前系统时间和车辆位置发送适当的轨迹段。
- EMPlanner（自Apollo1.5以来。EM是Expectation Maximization的缩写）：EM计划器，会根据地图，路线和障碍物计算驾驶决策和线路。基于动态规划（Dynamic programming，简称DP）的方法首先用于确定原始路径和速度曲线，然后使用基于二次规划（Quadratic programming，简称QP）的方法来进一步优化路径和速度曲线以获得平滑的轨迹。
- LatticePlanner：网格计划器
- NaviPlanner：这是一个基于实时相对地图的计划器。它使用车辆的FLU（Front-Left-Up）坐标系来完成巡航，跟随，超车，接近，变道和停车任务。

模块输入：

- RTK重放计划器：
  - Localization
  - 记录的RTK轨迹
- EM计划器：
  - Localization
  - Perception
  - Prediction
  - HD Map
  - Routing

模块输出：

- 无碰撞和舒适的轨迹让控制模块的执行

### 模块解析

Planing模块在初始化的`Init`函数中，这里面会注册所有的计划器，然后根据配置文件中的配置确定当前所使用的计划器。目前，配置文件中配置的是EM计划器。

Planing模块在`Start`函数中设定了一个Timer用来完成定时任务：

```
Status Planning::Start() {
  timer_ = AdapterManager::CreateTimer(
      ros::Duration(1.0 / FLAGS_planning_loop_rate), &Planning::OnTimer, this);
...
```

`Planning::OnTimer`最主要的就是调用`RunOnce()`，而后者包含了Planing模块的核心逻辑。目前，`FLAGS_planning_loop_rate`值是10。也就是说，Planing模块运行的频度是每秒钟10次。

在Planning::Plan函数中，更通过配置的计划器进行路线的计算，然后将结果对外发布。关键代码如下：

```
Status Planning::Plan(const double current_time_stamp,
                      const std::vector<TrajectoryPoint>& stitching_trajectory,
                      ADCTrajectory* trajectory_pb) {
  auto* ptr_debug = trajectory_pb->mutable_debug();
  if (FLAGS_enable_record_debug) {
    ptr_debug->mutable_planning_data()->mutable_init_point()->CopyFrom(
        stitching_trajectory.back());
  }

  auto status = planner_->Plan(stitching_trajectory.back(), frame_.get());

  ExportReferenceLineDebug(ptr_debug);

  const auto* best_ref_info = frame_->FindDriveReferenceLineInfo();
  if (!best_ref_info) {
    std::string msg("planner failed to make a driving plan");
    AERROR << msg;
    if (last_publishable_trajectory_) {
      last_publishable_trajectory_->Clear();
    }
    return Status(ErrorCode::PLANNING_ERROR, msg);
  }
  ptr_debug->MergeFrom(best_ref_info->debug());
  trajectory_pb->mutable_latency_stats()->MergeFrom(
      best_ref_info->latency_stats());
  // set right of way status
  trajectory_pb->set_right_of_way_status(best_ref_info->GetRightOfWayStatus());
  for (const auto& id : best_ref_info->TargetLaneId()) {
    trajectory_pb->add_lane_id()->CopyFrom(id);
  }

  best_ref_info->ExportDecision(trajectory_pb->mutable_decision());

  ...

  last_publishable_trajectory_->PrependTrajectoryPoints(
      stitching_trajectory.begin(), stitching_trajectory.end() - 1);

  for (size_t i = 0; i < last_publishable_trajectory_->NumOfPoints(); ++i) {
    if (last_publishable_trajectory_->TrajectoryPointAt(i).relative_time() >
        FLAGS_trajectory_time_high_density_period) {
      break;
    }
    ADEBUG << last_publishable_trajectory_->TrajectoryPointAt(i)
                  .ShortDebugString();
  }

  last_publishable_trajectory_->PopulateTrajectoryProtobuf(trajectory_pb);

  best_ref_info->ExportEngageAdvice(trajectory_pb->mutable_engage_advice());

  return status;
}
```



## 解析百度Apollo之决策规划模块

Apollo系统中的**Planning**模块实际上是整合了**决策和规划**两个功能，该模块是自动驾驶系统中最核心的模块之一（另外三个核心模块是：**定位**，**感知**和**控制**）。

下面这幅图描述了这其中的模块和它们之间的交互关系。

这其中的黄线代表了数据流，黑线代表了控制流。

![img](https://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2019-03-12-apollo-planning/module_interation.png)

Planning模块负责整个车辆的驾驶决策，而驾驶决策需要根据当前所处的地理位置，周边道路和交通情况来决定。Planning不直接控制车辆硬件，而是借助于控制模块来完成。

从这幅图中可以看出，对于Planning模块来说：

- 它的上游模块是：定位，地图

# 模块概述

决策规划模块的主要责任是：**根据导航信息以及车辆的当前状态，在有限的时间范围内，计算出一条合适的轨迹供车辆行驶**。



# 模块概述

决策规划模块的主要责任是：**根据导航信息以及车辆的当前状态，在有限的时间范围内，计算出一条合适的轨迹供车辆行驶**。

这里有好几个地方值得注意：

1. 车辆的行驶路线通常由Routing模块提供，Routing模块会根据目的地以及地图搜索出一条代价尽可能小的路线。关于Routing模块我们已经在另外一篇文章中详细讲解过（[《解析百度Apollo之Routing模块》](https://paul.pub/apollo-routing/)），这里不再赘述。
2. 车辆的当前状态包含了很多因素，例如：车辆自身的状态（包括姿态，速度，角速度等等），当前所处的位置，周边物理世界的静态环境以及交通状态等等。
3. Planning模块的响应速度必须是稳定可靠的（当然，其他模块也是一样）。正常人类的反应速度是300ms，而自动驾驶车辆想要做到安全可靠，其反应时间必须短于100ms。所以，Planning模块通常以10Hz的频率运行着。如果其中某一个算法的时间耗费了太长时间，就可能造成其他模块的处理延迟，最终可能造成严重的后果。例如：没有即时刹车，或者转弯。
4. ”合适的轨迹“有多个层次的含义。首先，”轨迹“不同于“路径”，“轨迹”不仅仅包含了行驶路线，还要包含每个时刻的车辆的速度，加速度，方向转向等信息。其次，这条轨迹必须是底层控制可以执行的。因为车辆在运动过程中，具有一定的惯性，车辆的转弯角度也是有限的。在计算行驶轨迹时，这些因素都要考虑。最后，从人类的体验上来说，猛加速，急刹车或者急转弯都会造成非常不好的乘坐体验，因此这些也需要考虑。这就是为什么决定规划模块需要花很多的精力来优化轨迹，Apollo系统中的实现自然也不例外。



# 整体Pipeline

有相关专业知识的人都会知道，决策规划模块的主体实现通常都是较为复杂的。

Apollo系统中的实现自然也不例外，这里先通过一幅图说明其整体的Pipeline。然后在下文中我们再逐步熟悉每个细节。

![img](https://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2019-03-12-apollo-planning/pipeline.png)

这里有三个主要部分需要说明：

- `PncMap`：全称是Planning and Control Map。这个部分的实现并不在Planning内部，而是位于`/modules/map/pnc_map/`目录下。但是由于该实现与Planning模块紧密相关，因此这里放在一起讨论。该模块的主要作用是：根据Routing提供的数据，生成Planning模块需要的路径信息。
- `Frame`：Frame中包含了Planning[一次计算循环](https://paul.pub/apollo-planning/#id-planningcycle)中需要的所有数据。例如：地图，车辆状态，参考线，障碍物信息等等。`ReferenceLine`是车辆行驶的参考线，`TrafficDecider`与交通规则相关，这两个都是Planning中比较重要的子模块，因此会在下文中专门讲解。
- `EM Planner`：下文中我们会看到，Apollo系统中内置了好几个Planner，但目前默认使用的是EM Planner，这也是专门为开放道路设计的。该模块的实现可以说是整个Planning模块的灵魂所在。因此其算法值得专门用另外一篇文章来讲解。读者也可以阅读其官方论文来了解：[Baidu Apollo EM Motion Planner](https://arxiv.org/abs/1807.08048)。







**roll**，**pitch**和**yaw**分布描述了车辆在纵向，侧向和垂直方向上的角度，具体如下图所示：

![img](https://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2019-04-30-apollo-reference-line/vehicle-coord.png)

## RouteSegments

我们回顾一下，Routing的搜索结果`RoutingResponse`中包含了下面三个层次的结构：

- `RoadSegment`：描述道路，一条道路可能包含了并行的几条通路（`Passage`）。
- `Passage`：描述通路，通路是直连不含变道的可行驶区域。一个通路可能包含了前后连接的多个车道。
- `LaneSegment`：描述车道，车道是道路中的一段，自动驾驶车辆会尽可能沿着车道的中心线行驶。

而pnc_map模块中的`RouteSegments`对应了上面的`Passage`结构，它其中会包含若干个车道信息。这个类继承自`std::vector<LaneSegment>`。





参控

https://paul.pub/apollo-reference-line/#id-referenceline%E7%BB%93%E6%9E%84