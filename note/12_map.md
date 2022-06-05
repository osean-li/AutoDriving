## Map模块简介

其实我们只需要知道map模块的主要功能是“加载openstreet格式的地图，并且提供一系列的API给其他模块使用”。然后再根据具体的场景来了解地图各个部分的作用，就算是对map模块比较了解了。

## Map目录结构

本章主要介绍下apollo代码的map模块，map的代码目录结构如下:

```
├── data           // 生成好的地图
│   └── demo
├── hdmap          // 高精度地图
│   ├── adapter    // 从xml文件读取地图(openstreet保存格式为xml)
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
map模块没有实现的功能是高精度地图的制作，简略的制图过程将在下面章节介绍。

