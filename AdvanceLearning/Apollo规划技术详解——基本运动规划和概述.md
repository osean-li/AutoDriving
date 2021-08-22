# Apollo规划技术详解——基本运动规划和概述

# 什么是运动规划

**规划问题**本质上是一个搜索问题

从内容考虑，规划问题涉及三个领域，**机器人领域**，**控制领域**和**人工智能**。不同的领域对问题的理解不同。



> 1. robotic fields，规划就是如何产生轨迹以完成目标，涉及RRT， A*，D* lite等。
> 2. 控制领域，规划理解为到达目标状态的动态系统，涉及MPC， LQR等。
> 3. artificial intellegence（AI），是生成状态到动作的映射，涉及reinforcement  learning，end-to-end imitation learning等。

# 如何构建汽车的运动规划问题

从最简单问题出发，把运动规划抽象成一个path finding problem（路径查找问题），只关心无人车怎样走，周围环境是不变的。

如图想寻找最短路径，可通过广度优先的方法，也可以使用深度优先方法。但是<u>对于最短路径优化问题，深度优先方法的效率太低。</u>

![**图1 路径查找问题**](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20210822171154271.png)



另外，可以通过启发式方式对搜索问题进行优化。前面的广度优先方法没有利用起点和终点之间的信息，问题就在于不知道目标在哪里。

**A-star算法**是大概知道红点在右边，定义一个启发式函数，该函数猜测距离目标还有多远，通过这种方法先搜索一些比较近的点，然后从这个点出发逐渐扩大搜索圈。



![图片](https://mmbiz.qpic.cn/mmbiz_png/Lic3WIjno4gzO9sR5svZ3ncFXHwGZKUvbOBB5FHGSKicRCSVCfMwVDAPMvIAKXbicJwNC8IeGJDC99en8rbiaCgjdw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​																			**图2 A-star算法**

A-star花费时间比广度优先算法时间更短，因为它有信息支持，**现在的一些路径搜索算法本质上都是从A-star算法出发，需要知道目标函数的样子。**

目前，**A-star算法**还不能直接用在规划模块上，因为**A-star算法**本身要求对整个环境全知。而自动驾驶对周围环境是部分观察的

![image-20210822185943995](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20210822185943995.png)

对于部分观察我们可以使用**贪心算法**（总是**做出在当前看来是最好的选择**），其实就是一个增量搜索，就是在看见的情况下尽量走好

如图4所示，利用D-star算法对部分观察的数据进行**控制规划**。它利用当前能够看到的信息进行增量规划，D-star的特点是处理在看到的有限范围的条件下，如何到达预定地点的搜索问题方法。<u>这种增量搜索很难通过一步步的迭代达到全局最优解。</u>



![图片](https://mmbiz.qpic.cn/mmbiz_png/Lic3WIjno4gzO9sR5svZ3ncFXHwGZKUvbpNoflv1158mV7v0NJYaJ8QdvJYEpIzBHKickMJoXaLhRfoBRs4y4s4A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​																			**D-star算法**

在现实生活中，人类开车是很少做90度直角转弯的，这样的折线并没有考虑无人车运动过程中的运动模型和动力学模型。更进一步，可以通过平滑性曲线的方式来优化折线，换成一些较为平滑的曲线来完成，如图5所示。



![图片](https://mmbiz.qpic.cn/mmbiz_png/Lic3WIjno4gzO9sR5svZ3ncFXHwGZKUvbYbLibWibofpB74AaBd38iaVuKUJ994NbnROgjr0x1nvib2f2tm8M6Y8dzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

​																						**曲线平滑**

那么，上述介绍的搜索技术离自动驾驶运动规划还差多少？

- 首先是在部分观察空间的动态障碍物，规划模块怎么处理动态障碍物是关键并且是有难度的。
- 其次是自动驾驶汽车能否按照规划的行驶，可能需要一个**动态模型**。
- 第三还缺少了遵守交通规则，它是道路安全的基本保证，将交通规则融入规划也是一个难点。
- 第四是实时计算，目前来说百度要求规划模块运转周期是100毫秒。

**运动规划问题**就是让自动驾驶车辆能够安全平稳到达终点，本质是一个三维规划问题，即 XY 坐标加上时间维度，叫做 3D Trajectory Optimization Problem（***轨迹优化问题***）。

从车辆动力学模型来说，维度需要进一步上升，因为涉及到车头的方向，车的**转向角、加速度**等问题。



 **无人驾驶系统软件的组成**

无人驾驶系统软件包括**定位、感知、预测、运动规划**和**控制**等。

定位就是知道汽车在哪里。location

感知即是对周围环境信息的获取。 det

预测就是感知到的障碍物的行进意图，例如旁边有无人超车。 planning+navgation

运动规划是在知道这些信息后，告知汽车如何运动。planning+navgation

控制模块就是得到指令之后如何控制汽车。control

