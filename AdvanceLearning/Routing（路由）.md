## Routing（路由）

### 模块介绍

Routing模块根据请求生成导航信息。

模块输入：

- 地图数据
- 请求，包括：开始和结束位置

模块输出：

- 路由导航信息

### 模块解析

Routing模块的内部结构如下图所示：

![img](https://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2018-07-07-baidu_apollo/Routing_module.png)

Routing模块的输入是地图数据和导航请求，因此其`Init`函数就是围绕这个逻辑的：

```
apollo::common::Status Routing::Init() {
  const auto routing_map_file = apollo::hdmap::RoutingMapFile();
  AINFO << "Use routing topology graph path: " << routing_map_file;
  navigator_ptr_.reset(new Navigator(routing_map_file));
  CHECK(common::util::GetProtoFromFile(FLAGS_routing_conf_file, &routing_conf_))
      << "Unable to load routing conf file: " + FLAGS_routing_conf_file;

  AINFO << "Conf file: " << FLAGS_routing_conf_file << " is loaded.";

  hdmap_ = apollo::hdmap::HDMapUtil::BaseMapPtr();
  CHECK(hdmap_) << "Failed to load map file:" << apollo::hdmap::BaseMapFile();

  AdapterManager::Init(FLAGS_routing_adapter_config_filename);
  AdapterManager::AddRoutingRequestCallback(&Routing::OnRoutingRequest, this);
  return apollo::common::Status::OK();
}
```

这段代码的重点是下面三个地方：

- `apollo::hdmap::RoutingMapFile()`包含了HD地图数据。
- `Navigator`负责导航，我们很容易想到这个类应当是该模块的核心。
- `Routing::OnRoutingRequest`是接收导航请求的回调函数。

在`Routing::OnRoutingRequest`中，最主要的就是通过`Navigator::SearchRoute`来搜索导航路径。

```
void Routing::OnRoutingRequest(const RoutingRequest& routing_request) {
  AINFO << "Get new routing request:" << routing_request.DebugString();
  RoutingResponse routing_response;
  apollo::common::monitor::MonitorLogBuffer buffer(&monitor_logger_);
  const auto& fixed_request = FillLaneInfoIfMissing(routing_request);
  if (!navigator_ptr_->SearchRoute(fixed_request, &routing_response)) {
    AERROR << "Failed to search route with navigator.";

    buffer.WARN("Routing failed! " + routing_response.status().msg());
    return;
  }
  buffer.INFO("Routing success!");
  AdapterManager::PublishRoutingResponse(routing_response);
  return;
}
```

目前，Apollo 2.5版本中的导航基于[A*算法](https://en.wikipedia.org/wiki/A*_search_algorithm)。这是一种在图形平面上，有多个节点的路径，求出最低通过成本的算法。该算法综合了Best-First Search和Dijkstra算法的优点：在进行启发式搜索提高算法效率的同时，可以保证找到一条最优路径（基于评估函数）。

在A*算法计算的过程中，会尝试多条路径。一旦遇到障碍物，便将该路径上的点标记为不需要继续探索（图中的实心点）。继续以剩下的空心点为基础探索。最终求得最优路径。

下图动态描述了[A*算法](https://en.wikipedia.org/wiki/A*_search_algorithm)查找目标路径的算法过程。

![img](https://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2018-07-07-baidu_apollo/Astar_progress_animation.gif)



| 类型名称        | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| LaneWaypoint    | 道路上的路径点，包含了id，长度和位置点信息。                 |
| LaneSegment     | 道路的一段，包含了id和起止点信息。                           |
| RoutingRequest  | 描述了路由请求的信息，一次路由请求可以包含多个路径点。详细结构见下文。 |
| Measurement     | 描述测量的距离。                                             |
| ChangeLaneType  | 道路的类型，有FORWARD，LEFT，RIGHT三种取值。                 |
| Passage         | 一段通路，其中可以包含多个LaneSegment，以及ChangeLaneType。  |
| RoadSegment     | 道路的一段，拥有一个id，并可以包含多个Passage。              |
| RoutingResponse | 路由请求的响应结果，可以包含多个RoadSegment，距离等信息。    |

输出RoadSegment，Passage，LaneSegment



# Routing模块结构一览

文末，我们通过一幅图来描述Routing模块中的主要组件以及它们的交互关系。

![img](https://qiangbo-workspace.oss-cn-shanghai.aliyuncs.com/2019-02-05-baidu-apollo-routing/overview.png)



参考

https://paul.pub/apollo-routing/