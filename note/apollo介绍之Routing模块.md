# apollo介绍之Routing模块

- **Routing** - 主要关注起点到终点的长期路径，根据起点到终点之间的道路，选择一条最优路径。
- **Planning** - 主要关注几秒钟之内汽车的行驶路径，根据当前行驶过程中的交通规则，车辆行人等信息，规划一条短期路径。

# Routing模块分析



分析Routing模块之前，我们只需要能够解决以下几个问题，就算是把routing模块掌握清楚了。

1. 如何从A点到B点？
2. 如何规避某些点？ - 查找的时候发现是黑名单里的节点，则选择跳过
3. 如何途径某些点？- 采用分段的形式，逐段导航（改进版的算法是不给定点的顺序，自动规划最优的线路）
4. 如何设置固定线路，而且不会变？最后routing输出的结果是什么？

# protobuf（.proto）的使用



protobuf 是Google提供一个具有高效的协议数据交换格式工具库，已被广泛应用。
通俗点说，就是将要发送的消息，通过protobuf格式发出去，自动进行封装。另一方对proto格式解析，直接得到原数据。
注：适用linux

使用方式
通过编写一个要发送的消息格式文件（.proto文件），然后通过自动化生成工具，生成对应的处理类及其头文件。

# **建图**



routing需要的是一个拓扑结构的图，要想做routing，第一步就是要把原始的地图转换成包含拓扑结构的图。

**base_map就是高精度地图**

![image-20211028224410615](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20211028224410615.png)



**routing_map则是导航地图**，routing_map的结构为一个有向图，routing_map.txt中只包含一个节点(Node)，没有边(Edge)信息

![image-20211028224010619](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20211028224010619.png)



apollo建图的实现在"routing/topo_creator"中

自动驾驶中的道路是车道线级别的，原来的这种定义点和边的方式就不适用了（定义不了车道），所以apollo中引用的新的概念，apollo中的点就是一条车道，而边则是车道和车道之间的连接，点对应具体的车道，而边则是一个虚拟的概念，表示车道之间的关系



![image-20211028225155387](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20211028225155387.png)

其中节点和边的结构在protobuf中定义，在文件"modules/routing/proto/topo_graph.proto"中，其中：

- **NODE** - 包括车道唯一id，长度，左边出口，右边出口（这里的出口对应车道虚线的部分，或者自己定义的一段允许变道的路段），路段代价（限速或者拐弯的路段会增加成本，代价系数在routing_config.pb.txt中定义)，中心线（虚拟的，用于生成参考线），是否可见，车道所属的道路id。

- **EDGE** - 则包括起始车道id，到达车道id，切换代价，方向（向前，向左，向右）。

  ## **主流程**

  ```text
  ├── BUILD
  ├── edge_creator.cc            // 建边 
  ├── edge_creator.h
  ├── graph_creator.cc           // 建图
  ├── graph_creator.h
  ├── graph_creator_test.cc
  ├── node_creator.cc            // 建节点
  ├── node_creator.h
  └── topo_creator.cc           // main函数
  ```

![image-20211028225408975](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20211028225408975.png)

routing_map中就是grap_ protobuf格式的固化，后面routing模块会读取创建好的routing_map通过astar算法来进行路径规划。

## **创建节点**

```text
 // 1. 初始化节点信息
 // 1.1根据lane的边界，添加能够变道的路段
 
 // 2. 初始化节点代价
 // 2.1. 根据道路长度和速度限制来计算代价
 // 2.2. 掉头代价 > 左转代价 > 右转的代价
 
 
```

## **创建节点的边**

```text
// 设置起始，终止车道和类型
// 默认代价为0，即直接向前开的代价
```



到这里制作routing_map的流程就结束了，建图的主要目的是把base结构的map转换为graph（图）结构的map，从而利用图结构来查找最佳路径，下面会分析如何通过routing_map得到规划好的路线。

# Routing主流程

![img](https://pic3.zhimg.com/80/v2-b6f90c87c99bf5eb1b36bbda5f794842_720w.jpg)

把一些主要的流程摘要如下：

1. 在cyber中注册component，接收request请求，响应请求结果response
2. 读取routing_map并且建图graph
3. 获取request中的routing请求节点
4. 根据black_map生成子图sub_graph
5. 通过astar算法查找最短路径
6. 合并请求结果并且返回

<u>下面在结合具体的流程进行分析，这里主要要弄清楚2点：1.为什么要生成子图？ 2.如何通过astar算法查找最优路径？</u>

## **component**



首先我们从"routing_component.h"和"[http://routing_component.cc](https://link.zhihu.com/?target=http%3A//routing_component.cc)"开始，apollo的功能被划分为各个模块，启动时候由cyber框架根据模块间的依赖顺序加载(每个模块的dag文件定义了依赖顺序)，**每次开始查看一个模块时，都是从component文件开始**。

![image-20211028232017628](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20211028232017628.png)

routing模块都按照cyber的模块申明和注册，cyber框架负责调用Init进行初始化，并且收到消息时候触发Proc执行。

我们先看下"Init"函数:

```text
  // 设置消息qos，控制流量，创建消息发布response_writer_
  // 历史消息发布，和response_writer_类似 
  // 创建定时器
  // 执行Routing类
```

接下来当routing模块收到routing_request时，会触发"Proc()"，返回routing_response:

![image-20211028232431498](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20211028232431498.png)

从上面的分析可以看出，"**RoutingComponent**"模块实现的**主要功能**:

1. 实现"Init"和"Proc"函数
2. 接收"RoutingRequest"消息，输出"RoutingResponse"响应。

接下来我们来看routing的具体实现。

## **Routing类**

看代码之前先看下头文件是个很好的习惯。通过头文件，我们可以知道当前模块的依赖项，从而搞清楚各个模块之间的依赖关系。可以看到"Routing"模块是一个相对比较独立的模块，只依赖于地图

![image-20211028232736777](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20211028232736777.png)



上述的过程总结一下就是，首先读取routing_map并初始化Navigator类，接着遍历routing_request，因为routing_request请求为一个个的点，所以先查看routing_request的点是否在路上，不在路上则找到最近的路，并且补充信息（不在路上的点则过不去），最后调用"navigator_ptr_->SearchRoute"返回routing响应。



![image-20211028233013398](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20211028233013398.png)



## **Navigator类**





# 小结

1. 何从A点到B点？-采用的是A* 算法

2. 如何规避某些点？ - 查找的时候发现是黑名单里的节点，则选择跳过

3. 如何途径某些点？- 采用分段的形式，逐段导航（改进版的算法是不给定点的顺序，自动规划最优的线路）

4. 如何设置固定线路，而且不会变？最后routing输出的结果是RoutingResponse****

   `RoutingResponse`中的属性说明如下：

   | 名称            | 说明                         |
   | :-------------- | :--------------------------- |
   | header          | 消息头                       |
   | road            | 具体的路径信息，最重要的数据 |
   | measurement     | 距离                         |
   | routing_request | 原始请求                     |
   | map_version     | 地图版本                     |
   | status          | 状态位                       |



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

| 类型名称   | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| CurvePoint | 曲线上的一个点。                                             |
| CurveRange | 曲线上的一段。                                               |
| Node       | 车道上的一个节点，包含了所属车道，道路，长度，曲线起止点，中心线等信息。 |
| Edge       | 连接车道之间的边，包含了起止车道id，代价和方向等信息。       |
| Graph      | 完整地图的Topo结构，这其中包含了多个Node和Edge。             |

| 类名             | 描述                                                         |
| :--------------- | :----------------------------------------------------------- |
| TopoNode         | Topo地图中的一个节点。包含了所属Lane和Road等信息。 很显然，这是Topo地图中的核心数据结构。 |
| TopoEdge         | 连接TopoNode之间的边，该结构中包含了起止TopoNode等信息。     |
| NodeSRange       | 描述节点的某一段范围。一个TopoNode可以分为若干个NodeSRange。 |
| NodeWithRange    | 描述节点及其范围，该类是NodeSRange的子类。                   |
| TopoRangeManager | NodeSRange的管理器。可以进行查找，添加，排序和合并操作。     |
| SubTopoGraph     | Topo子图，由搜索算法所用（目前是A*搜索算法）。               |
| TopoGraph        | 对应了整个Topo地图。其构造函数需要一个Proto结构导出的地图文件， 它将从地图文件中读取完整的Topo结构。 |



# 参考

https://paul.pub/apollo-routing/

https://zhuanlan.zhihu.com/p/65533164