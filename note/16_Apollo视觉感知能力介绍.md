

# Apollo视觉感知能力介绍

更新时间：2021-11-11

本文档介绍 Apollo 视觉感知模块。通过对本文档的学习，您可以初步了解到 Apollo 视觉感知模块整体结构以及各个模块的主要组成部分，并且可以对视觉感知模块的输入输出有比较清晰的认识。

Apollo 视觉感知模块输入输出如下图所示：

![图片名称](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_f599009.png)



## 视觉感知模块简介

摄像头作为无人车系统中最重要的传感器之一，因为其信息丰富、观测距离远等特点，在障碍物检测和红绿灯检测等方向发挥着不可替代的作用，是对激光雷达感知结果的重要补充。

但摄像头有着容易受环境影响、缺乏深度信息等缺点，给无人驾驶系统中的视觉感知算法带来了巨大的挑战。因此，如何建立一套高精确率和高稳定性的视觉感知算法，是无人车感知模块的核心问题。视觉感知算法在 Apollo 平台上主要有 3 个应用场景，分别是红绿灯检测、车道线检测、基于摄像头的障碍物检测。

![图片名称](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_d0b989b.png)

每个模块又可以分为 3 部分，分别是图像的预处理、神经网络模型以及后处理。

- **预处理**：对上游信息做一些处理和整合，以方便把信息直接输入到模型中做预测。
- **神经网络模型**：主要涉及一些深度学习算法，包括目标检测、语义分割、图像分类等。
- **后处理**：为了优化模型效果，我们会利用一些传统的算法进一步优化网络模型的预测，让我们的算法可以在实车上跑得更加流畅。

## 视觉感知模块的原理

### 红绿灯检测模块原理

红绿灯检测模块的主要作用是检测当前路况下在摄像头的视觉范围内的红绿灯的状态，这是一个典型的目标检测任务。如下图所示，红绿灯检测模块一共包含 4 个部分，分别是数据预处理、红绿灯位置检测、红绿灯识别和矫正。

1. 通常红绿灯在图像中所占比例比较低，为了能够有效的检测出红绿灯，我们会借助高精地图给出的的信息预先从相机图像中将包含红绿灯区域的大致位置取出来。
2. 红绿灯位置检测部分采用了一种常规的基于 CNN 的目标检测算法，模型接收的就是刚刚选取的区域，输出是红绿灯的边框以及红绿灯的类别。
3. 检测完成后，需要对红绿灯的颜色做识别。我们训练了 3 个轻量级的卷积神经网络做分类，这 3 个网络分别对应检测结果的 3 个类别，竖着的、横着的和方形的。
4. 最后，矫正器主要针对识别置信度不高的情况，通过查询前面几帧的检测状态对当前的红绿灯颜色进行矫正。

![图片名称](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_6479733.png)

### 车道线检测模块原理

我们把车道线检测算法当作分割问题，在图像中寻求对于语义上车道线存在位置的检测，即使它是模糊的、被光照影响的、甚至是完全被遮挡的，我们都希望能将它检测出来。

![图片名称](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_92ed759.png)

车道线检测的网络结构如上图所示，主干是一个编码-解码结构的分割网络，用来输出车道线分割结果。在车道编码器的末端附加一个网络分支来检测消失点。

该分支由卷积层和全连通层组成，其中卷积层为消失点任务转换车道特征，全连通层对整个图像进行全局汇总，输出消失点位置。

车道线检测模型在遮挡情况下和转弯时都可以对车道线进行有效的检测。

### 障碍物检测模块原理

障碍物检测部分采用的是基于单目视觉的障碍物检测算法，根据相机获取的图像得到障碍的类别和位置信息。

这里使用 7 个变量来表示 3D 边框，分别是物体的长宽高，物体的位置 x，y，z 以及物体的旋转角度 θ。

![图片名称](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_a96269d.png)

上图是 3D 障碍物检测模块的模型结构图，输入的是单张的图像信息，经过神经网络提取特征，最后接上检测模块（Apollo 有两个检测模型，分别是基于 YOLO 的 one-stage 检测方法和基于中心点检测的检测方法）。

最终，通过模型可以得到障碍物的 3D 信息（长宽高、角度等），可以根据这些信息求出障碍物的 3D 边框。

## 视觉感知模块的输入输出

下表列出了视觉感知模块各个 channel 的输入输出：

| channel 名称                      | 输入输出 | channel 说明                             |
| --------------------------------- | -------- | ---------------------------------------- |
| /apollo/sensor/camera/front_6mm   | 输入     | 前置 6mm 摄像头                          |
| /apollo/sensor/camera/front_12mm  | 输入     | 前置 12mm 摄像头                         |
| /apollo/localization/pose         | 输入     | 主车速度和角速度                         |
| /tf                               | 输入     | 定位信息                                 |
| /perception/obstacles             | 输出     | 具有航向、速度和分类信息的三维障碍物轨迹 |
| /perception/inner/PrefusedObjects | 输出     | 输出给融合模块的障碍物信息               |
| /apollo/perception/traffic_light  | 输出     | 交通信号灯检测与识别的输出               |

下表列出了视觉感知模块的相关参数以及修改建议：

| 文件名称                                 | 主要参数                      | 修改建议                                                     | 作用                      |
| ---------------------------------------- | ----------------------------- | ------------------------------------------------------------ | ------------------------- |
| fusion_camera_detection_component.pb.txt | camera_names                  | 根据实际相机名称修改                                         | 相机名称                  |
| fusion_camera_detection_component.pb.txt | input_camera_channel_names    | 根据实际相机 topic 名称修改                                  | 相机 topic 名称           |
| fusion_camera_detection_component.pb.txt | output_obstacles_channel_name | 如果只使用 camera 感知，则修改为 `/apollo/perception/obstacles`，如果使用融合感知，则修改为`/perception/obstacles` | 相机感知输出的 topic 名称 |
| fusion_camera_detection_component.pb.txt | camera_names                  | 根据实际相机名称修改                                         | 相机名称                  |
| apollo/modules/perception/data/params/   | -                             | 根据实际传感器标注结果确定                                   | 相机标定内外参            |

下面分别介绍每个模块的输入输出。

### 红绿灯检测模块

红绿灯检测模块输入输入如下表所示：

| channel 名称                     | 输入输出 | channel 说明               |
| -------------------------------- | -------- | -------------------------- |
| /apollo/sensor/camera/front_6mm  | 输入     | 前置 6mm 摄像头            |
| /apollo/sensor/camera/front_12mm | 输入     | 前置 12mm 摄像头           |
| /tf                              | 输入     | 定位信息                   |
| /apollo/perception/traffic_light | 输出     | 交通信号灯检测与识别的输出 |

### 障碍物检测模块

障碍物检测模块输入输入如下表所示：

| channel 名称                      | 输入输出 | channel 说明                             |
| --------------------------------- | -------- | ---------------------------------------- |
| /apollo/sensor/camera/front_6mm   | 输入     | 前置 6mm 摄像头                          |
| /apollo/sensor/camera/front_12mm  | 输入     | 前置 12mm 摄像头                         |
| /apollo/localization/pose         | 输入     | 主车速度和角速度                         |
| /perception/obstacles             | 输出     | 具有航向、速度和分类信息的三维障碍物轨迹 |
| /perception/inner/PrefusedObjects | 输出     | 输出给融合模块的障碍物信息               |

![图片名称](https://bce.bdstatic.com/doc/Apollo-Homepage-Document/Apollo_Doc_CN_6_0/image_3b10113.png)