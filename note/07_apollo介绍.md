

## 简介

apollo是百度的自动驾驶开源框架，根据自动驾驶的功能划分为不同的模块，下面会根据目录结构和功能模块分别介绍和学习"apollo模块"。下面简单介绍下各个模块的作用:

- **定位** 

- **感知** 

- **规划**

- **控制** 

  

# 目录结构

```html
|-cyber 消息中间件，替换ros作为消息层
|-docker 容器相关
|-docs 文档相关
|-modules 自动驾驶模块，主要的定位，预测，感知，规划都在这里
    |-calibration 校准，主要用于传感器坐标的校准，用于感知模块做传感器融合
    |-canbus 通讯总线，工业领域的标准总线，鉴于工业界的保守，我估计后面会有新的总线来取代
    |-common
    |-contrib
    |-control 控制模块，根据planning生成的路径对车辆轨迹进行控制，再底层就是发送命令到can总线，实现车辆的控制。
    |-data 地图等生成好的数据放在这里（其他数据待补充）
    |-dreamview 仿真，能够对自动驾驶过程中的数据进行回放，其他厂家也有推出一些仿真平台，后面有机会再介绍下
    |-drivers 雷达，lidar，GPS, canbus，camera等驱动
    |-guardian 监护程序？
    |-localization 定位，获取汽车的当前位置
    |-map 地图模块
    |-monitor 监控模块，主要是监控汽车状态，并且记录，用于故障定位，健康检查等
    |-perception 感知，获取汽车当前的环境，行人，车辆，红绿灯等，给planning模块规划线路
    |-planning 规划，针对感知到的情况，对路径做规划，短期规划，只规划100-200M的距离，生成好的路径给control模块
    |-prediction 预测，属于感知模块，对运动物体的轨迹做预测
    |-routing 导航线路，就是百度地图上查询2点之间的线路，生成的线路短期规划还是planning模块
    |-third_party_perception 第三方感知模块
    |-tools 工具，这里面的工具倒是很多，后面再详细介绍下
    |-transform 转换，主要是？
    |-v2x 顾名思义就vehicle-to-everything，其希望实现车辆与一切可能影响车辆的实体实现信息交互，
          目的是减少事故发生，减缓交通拥堵，降低环境污染以及提供其他信息服务.
|-scripts 脚本
|-third_party 第三方库
|-tools 工具目录，基本就是个空目录
```

## 编译

apollo采用的是bazel来进行编译

# 自动驾驶技术栈

![自动驾驶技术栈](D:\自动驾驶\AutoDriving\note\picture\自动驾驶技术栈.png)