# 文件结构分析

Apollo/modules/perception
`perception`
`├── base    // 一些基础类`
`├── BUILD`
`├── camera  //相机检测`
`├── common  // 公用的类`
`├── data    // 一些相机的内外参`
`├── fusion  // 障碍物融合`
`├── inference //一些接口`
`├── lib`
`├── lidar   // 激光雷达检测`
`├── map     // 高精度地图`
`├── onboard // 组件文件结构和组件类的实现`
`├── Perception_README_3_5.md`
`├── production  // 组件配置文件和启动文件`
`├── proto       // 一些protobuf的消息结构`
`├── radar       // 毫米波雷达`
`├── README.md`
`└── tool        // 一些工具方法`

base——整个感知模块公用的一些基础类型定义，比如表示感知目标的基础类类型；
camera——相机处理子模块；
common——整个感知模块公用的一些基础操作定义，如点云预处理、图像预处理等；
data——感知模块用到的数据，目前只有相机的标定参数yaml文件；
fusion——融合处理子模块；
interface——整个感知模块的一些公用接口定义，目前里面好像都是视觉感知与显卡的一些的接口；
lib——整个感知模块公用的一些算法定义，如config_manager参数配置操作；
datlidara——激光雷达处理子模块；
map——高精度地图的一些操作，主要在感知模块中用于提取感兴趣区域；
onboard——上车运行程序，其中component文件夹可以认为是感知模块的入口；
production——感知模块所有配置参数定义；
proto——感知模块protobuf文件定义；