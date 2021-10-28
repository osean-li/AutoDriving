# apollo介绍之map模块（二）

本章主要介绍下apollo代码的map模块，map的代码目录结构如下

```text
├── data           // 生成好的地图
│   └── demo
├── hdmap          // 高精度地图
│   ├── adapter    // 从xml文件读取地图(opendrive保存格式为xml)
│   │   └── xml_parser
│   └── test-data
├── pnc_map        // 给规划控制模块用的地图
│   └── testdata
├── proto          // 地图各元素的消息格式(人行横道，车道线等)
├── relative_map   // 相对地图
│   ├── common
│   ├── conf
│   ├── dag
│   ├── launch
│   ├── proto
│   ├── testdata
│   │   └── multi_lane_map
│   └── tools
├── testdata       // 测试数据？
│   └── navigation_dummy
└── tools          // 工具
```



apollo的高精度地图采用了opendrive格式，opendrive是一个统一的地图标准，这样保证了地图的通用性。其中map模块主要提供的功能是读取高精度地图，并且转换成apollo程序中的Map对象。直白一点就是说把xml格式的opendrive高精度地图，读取为程序能够识别的格式。

map模块没有实现的功能是高精度地图的制作，简略的制图过程可以[参考]([王方浩：高精度地图制作](https://zhuanlan.zhihu.com/p/57958859))。

由于opendrive格式是一个标准，可以它的参考官方网站。下面主要介绍下apollo是如何读取xml地图，并且使用的。

地图的读取在adapter中，其中xml_parser目录提供解析xml的能力。而[http://opendrive_adapter.cc](https://link.zhihu.com/?target=http%3A//opendrive_adapter.cc)则实现了地图的加载，转换为程序中的Map对象。然后地图在[http://hdmap_impl.cc](https://link.zhihu.com/?target=http%3A//hdmap_impl.cc)中提供一系列api接口给其他模块使用。

下面先介绍下地图消息格式，主要在proto目录

map.proto 分为地图头部信息和结构体，头部信息主要介绍了地图的基本信息“版本，时间，投影方法，地图大小，厂家等”。结构体主要是道路的不同组成部分，包括“人行横道，路口区域，车道，停车观察，信号灯，让路标志，重叠区域，禁止停车，减速带，道路，停车区域，路边的小路，或者行人走的路”。

首先是地图的基本信息

```text
message Header {
  optional bytes version = 1;   //地图版本
  optional bytes date = 2;      //地图时间
  optional Projection projection = 3; //投影方法
  optional bytes district = 4;        //区
  optional bytes generation = 5;      //
  optional bytes rev_major = 6;       //
  optional bytes rev_minor = 7;       //
  optional double left = 8;           //左
  optional double top = 9;            //上
  optional double right = 10;         //右
  optional double bottom = 11;        //底
  optional bytes vendor = 12;         //供应商
}
```

下面是地图的道路信息，其中有2个标志(StopSign，YieldSign)是美国才有的，后来查看了下知乎发现对应到国内是(停，让)，具体的含义都是一样，停车的意思是到路口先停止，看下有没有车，然后再开始启动，让车就是先让行，比如交汇路口，理应让直行的车辆先通过，然后再汇入道路。[参考]([国内为什么不能采用美国的stop sign制度在很多路口设置呢？](https://www.zhihu.com/question/20512694))

下面在介绍下overlap，overlap在注释里的解释是“任何一对在地图上重合的东西，包括（车道，路口，人行横道）”，比如路口的人行横道和道路是重叠的，还有一些交通标志和道路也是重叠的，这是创造的一个逻辑概念。（不知道这样理解是否正确）

```text
message Map {
  optional Header header = 1;        //上面所说的地图基本信息

  repeated Crosswalk crosswalk = 2;  //人行横道
  repeated Junction junction = 3;    //交叉路口
  repeated Lane lane = 4;           //车道
  repeated StopSign stop_sign = 5;  //停车标志
  repeated Signal signal = 6;       //信号灯
  repeated YieldSign yield = 7;     //让车标志
  repeated Overlap overlap = 8;     //重叠区域
  repeated ClearArea clear_area = 9;  //禁止停车区域
  repeated SpeedBump speed_bump = 10;  //减速带
  repeated Road road = 11;             //道路
  repeated ParkingSpace parking_space = 12; //停车区域
  repeated Sidewalk sidewalk = 13;  //路边的小路，或者行人走的路，现在的版本已经去掉？但是其他模块有些还有sidewalk
}
```



map_crosswalk.proto 人行横道(google图片搜索出了彩虹人行横道和三维人行横道，就问深度学习该怎么办？)

```text
message Crosswalk {
  optional Id id = 1;        //编号

  optional Polygon polygon = 2;  //多边形

  repeated Id overlap_id = 3;   //重叠ID
}
```

![img](https://pic3.zhimg.com/80/v2-17bdce51918bfdd5b14c84e94022a172_720w.jpg)

map_junction.proto 路口，道路汇聚点

```text
message Junction {
  optional Id id = 1;    //编号

  optional Polygon polygon = 2;     //多边形

  repeated Id overlap_id = 3;    //重叠id
}
```

![img](https://pic1.zhimg.com/80/v2-f6fc2e24f6cfbfff0c22155a8eeeb7bc_720w.jpg)

map.lane.proto 车道线，介绍的比较复杂

```text
// A lane is part of a roadway, that is designated for use by a single line of vehicles.
// Most public roads (include highways) have more than two lanes.
message Lane {
  optional Id id = 1;         //编号

  // Central lane as reference trajectory, not necessary to be the geometry central.
  optional Curve central_curve = 2;     //中心曲线

  // Lane boundary curve.
  optional LaneBoundary left_boundary = 3;          //左边界
  optional LaneBoundary right_boundary = 4;         //右边界

  // in meters.
  optional double length = 5;                       //长度

  // Speed limit of the lane, in meters per second.
  optional double speed_limit = 6;           //速度限制

  repeated Id overlap_id = 7;                //重叠区域id

  // All lanes can be driving into (or from).
  repeated Id predecessor_id = 8;           //前任id
  repeated Id successor_id = 9;             //继任者id

  // Neighbor lanes on the same direction.
  repeated Id left_neighbor_forward_lane_id = 10;    //左边相邻前方车道id
  repeated Id right_neighbor_forward_lane_id = 11;   //右边相邻前方车道id

  enum LaneType {               //车道类型
    NONE = 1;                  //无
    CITY_DRIVING = 2;           //城市道路
    BIKING = 3;                 //自行车
    SIDEWALK = 4;               //人行道
    PARKING = 5;                //停车
  };
  optional LaneType type = 12;         //车道类型

  enum LaneTurn {
    NO_TURN = 1;        //直行
    LEFT_TURN = 2;      //左转弯
    RIGHT_TURN = 3;     //右转弯
    U_TURN = 4;         //掉头
  };
  optional LaneTurn turn = 13;          //转弯类型

  repeated Id left_neighbor_reverse_lane_id = 14;       //左边相邻反方向车道id
  repeated Id right_neighbor_reverse_lane_id = 15;      //右边相邻反方向车道id

  optional Id junction_id = 16;

  // Association between central point to closest boundary.
  repeated LaneSampleAssociation left_sample = 17;      //中心点与最近左边界之间的关联
  repeated LaneSampleAssociation right_sample = 18;     //中心点与最近右边界之间的关联

  enum LaneDirection {
    FORWARD = 1;     //前
    BACKWARD = 2;    //后，潮汐车道借用的情况？
    BIDIRECTION = 3;  //双向
  }
  optional LaneDirection direction = 19;   //车道方向

  // Association between central point to closest road boundary.
  repeated LaneSampleAssociation left_road_sample = 20;    //中心点与最近左路边界之间的关联
  repeated LaneSampleAssociation right_road_sample = 21;    //中心点与最近右路边界之间的关联
}
```

![img](https://pic4.zhimg.com/80/v2-4f3c86f1ed99263d5e6dc263c94ed5d3_720w.jpg)

map_stop_sign.proto 停止信号

```text
message StopSign {

  optional Id id = 1;         //编号

  repeated Curve stop_line = 2;    //停止线，Curve曲线应该是基础类型

  repeated Id overlap_id = 3;     //重叠id

  enum StopType {
    UNKNOWN = 0;        //未知
    ONE_WAY = 1;        //只有一车道可以停
    TWO_WAY = 2;
    THREE_WAY = 3;
    FOUR_WAY = 4;
    ALL_WAY = 5;
  };
  optional StopType type = 4;
}
```

![img](https://pic3.zhimg.com/80/v2-aa9ea13b3f9aec8614167272d525a4f6_720w.jpg)

map_signal.proto 交通信号标志

```text
message Subsignal {
  enum Type {
    UNKNOWN = 1;    //未知
    CIRCLE = 2;     //圈???
    ARROW_LEFT = 3;  //左边
    ARROW_FORWARD = 4;  //前面
    ARROW_RIGHT = 5;    //右边
    ARROW_LEFT_AND_FORWARD = 6;   //左前
    ARROW_RIGHT_AND_FORWARD = 7;  //右前
    ARROW_U_TURN = 8;   //掉头
  };

  optional Id id = 1;
  optional Type type = 2;

  // Location of the center of the bulb. now no data support.
  optional apollo.common.PointENU location = 3;    //也是基础类型？
}

message Signal {
  enum Type {
    UNKNOWN = 1;
    MIX_2_HORIZONTAL = 2;
    MIX_2_VERTICAL = 3;
    MIX_3_HORIZONTAL = 4;
    MIX_3_VERTICAL = 5;
    SINGLE = 6;
  };

  optional Id id = 1;
  optional Polygon boundary = 2;    //多边形
  repeated Subsignal subsignal = 3;   //子信号
  // TODO: add orientation. now no data support.
  repeated Id overlap_id = 4;   //重叠id
  optional Type type = 5;      //这里的类型是主要指交通标识的个数及位置？？
  // stop line
  repeated Curve stop_line = 6;     //在哪里结束？
}
```

![img](https://pic3.zhimg.com/80/v2-36d51e20ce7a4b81057278e8b5b2671a_720w.jpg)

map_yield_sign.proto 让行标志（美国才有）

```text
message YieldSign {
  optional Id id = 1;       //编号

  repeated Curve stop_line = 2;    //在哪里结束

  repeated Id overlap_id = 3;     //重叠id
}
```

![img](https://pic4.zhimg.com/80/v2-9744d057e65777bfe1ad92db42a71c8b_720w.jpg)

map_overlap.proto 这里只介绍了LaneOverlapInfo，其他的还没有对应的格式

```text
message LaneOverlapInfo {
  optional double start_s = 1;  //position (s-coordinate)
  optional double end_s = 2;    //position (s-coordinate)
  optional bool is_merge = 3;
}
// Information about one object in the overlap.
message ObjectOverlapInfo {
  optional Id id = 1;

  oneof overlap_info {
    LaneOverlapInfo lane_overlap_info = 3;
    SignalOverlapInfo signal_overlap_info = 4;
    StopSignOverlapInfo stop_sign_overlap_info = 5;
    CrosswalkOverlapInfo crosswalk_overlap_info = 6;
    JunctionOverlapInfo junction_overlap_info = 7;
    YieldOverlapInfo yield_sign_overlap_info = 8;
    ClearAreaOverlapInfo clear_area_overlap_info = 9;
    SpeedBumpOverlapInfo speed_bump_overlap_info = 10;
    ParkingSpaceOverlapInfo parking_space_overlap_info = 11;
    SidewalkOverlapInfo sidewalk_overlap_info = 12;
  }
}

// Here, the "overlap" includes any pair of objects on the map
// (e.g. lanes, junctions, and crosswalks).
message Overlap {
  optional Id id = 1;

  // Information about one overlap, include all overlapped objects.
  repeated ObjectOverlapInfo object = 2;
}
```

> 逻辑概念，没有具体的规则显示这个区域

map_clear_area.proto 禁止停车

```text
// A clear area means in which stopping car is prohibited

message ClearArea {
  optional Id id = 1;            //编号
  repeated Id overlap_id = 2;    //重叠id
  optional Polygon polygon = 3;  //多边形
}
```

![img](https://pic4.zhimg.com/80/v2-0accabfd5e326ccbda0184bc0771f4a3_720w.jpg)

map_speed_bump.proto 减速带

```text
message SpeedBump {
    optional Id id = 1;          //编号
    repeated Id overlap_id = 2;   //重叠区域
    repeated Curve position = 3;  //曲线位置
}
```

![img](https://pic2.zhimg.com/80/v2-47ba4be15f7259ac87f0592758f41e71_720w.jpg)

map_road.proto 道路的信息，是由一些RoadSection组成

```text
// road section defines a road cross-section, At least one section must be defined in order to
// use a road, If multiple road sections are defined, they must be listed in order along the road
message RoadSection {
  optional Id id = 1;
  // lanes contained in this section
  repeated Id lane_id = 2;
  // boundary of section
  optional RoadBoundary boundary = 3;
}

// The road is a collection of traffic elements, such as lanes, road boundary etc.
// It provides general information about the road.
message Road {
  optional Id id = 1;
  repeated RoadSection section = 2;

  // if lane road not in the junction, junction id is null.
  optional Id junction_id = 3;
}
```

![img](https://pic4.zhimg.com/80/v2-48341468d0fe5462cca5a53d06b18e53_720w.jpg)

map_parking.proto 停车区域

```text
// ParkingSpace is a place designated to park a car.
message ParkingSpace {
  optional Id id = 1;

  optional Polygon polygon = 2;

  repeated Id overlap_id = 3;

  optional double heading = 4;
}
```

![img](https://pic2.zhimg.com/80/v2-41a3876a49fd226204368519c7310e89_720w.jpg)

map_sidewalk.proto 路边的小路，或者行人走的路

```text
// A sidewalk (American English) or pavement (British English), also known as a footpath or footway, is a path along the side of a road.

message Sidewalk {
  optional Id id = 1;
  repeated Id overlap_id = 2;
  optional Polygon polygon = 3;
}
```

![img](https://pic2.zhimg.com/80/v2-caa32a3d7116be5b201a0c7740f35dd9_720w.jpg)



其中还剩下的4个没有介绍

map_id.proto

这里的map_id是基础id?

```text
message Id {
  optional string id = 1;     //id，字符类型
}
```



map_speed_control.proto 限制速度

```text
message SpeedControl {
  optional string name = 1;
  optional apollo.hdmap.Polygon polygon = 2;
  optional double speed_limit = 3;
}
```



map_geometry.proto 地图的几何形状？

```text
// Polygon, not necessary convex.
message Polygon {
  repeated apollo.common.PointENU point = 1;
}

// Straight line segment.
message LineSegment {
  repeated apollo.common.PointENU point = 1;
}

// Generalization of a line.
message CurveSegment {
  oneof curve_type {
    LineSegment line_segment = 1;
  }
  optional double s = 6;  // start position (s-coordinate)
  optional apollo.common.PointENU start_position = 7;
  optional double heading = 8;  // start orientation
  optional double length = 9;
}

// An object similar to a line but that need not be straight.
message Curve {
  repeated CurveSegment segment = 1;
}
```



map_pnc_junction.proto PNC路口（具体的场景是什么？？）

```text
message PNCJunction {
  optional Id id = 1;

  optional Polygon polygon = 2;

  repeated Id overlap_id = 3;
}
```



上面只是简单的介绍了下地图的数据格式，具体的应用场景，还需要结合planning模块进一步学习。



我们再回过头来看adapter模块，其中xml_parser就是针对道路的不同元素部分做的解析。

```text
├── adapter
│   ├── BUILD
│   ├── coordinate_convert_tool.cc    // 坐标转换工具
│   ├── coordinate_convert_tool.h
│   ├── opendrive_adapter.cc          // 加载opendrive格式地图
│   ├── opendrive_adapter.h
│   ├── proto_organizer.cc            // 
│   ├── proto_organizer.h
│   └── xml_parser          // xml_parser针对道路的不同元素做相应解析
│       ├── common_define.h
│       ├── header_xml_parser.cc
│       ├── header_xml_parser.h
│       ├── junctions_xml_parser.cc
│       ├── junctions_xml_parser.h
│       ├── lanes_xml_parser.cc
│       ├── lanes_xml_parser.h
│       ├── objects_xml_parser.cc
│       ├── objects_xml_parser.h
│       ├── roads_xml_parser.cc
│       ├── roads_xml_parser.h
│       ├── signals_xml_parser.cc
│       ├── signals_xml_parser.h
│       ├── status.h
│       ├── util_xml_parser.cc
│       └── util_xml_parser.h
```



最后在看下hdmap_impl，主要实现了一系列的api来查找道路中的元素。由于实现的接口太多，后面有时间了看是否能够整理下api文档。

## **总结**

其实我们只需要知道map模块的主要功能是“加载opendrive格式的地图，并且提供一系列的API给其他模块使用”。然后再根据具体的场景来了解地图各个部分的作用，就算是对map模块比较了解了。