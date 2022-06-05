## 引言

本篇文章基于lgsvl仿真工具和apollo框架进行安装和配置。

## 1.安装环境

```text
主机系统：18.04.5 LTS（Bionic Beaver）
运行内存：32G
显卡：NVIDIA GTX 2060 12G
Apollo 7.0
lgsvl：svlsimulator-linux64-2021.3
```

## 2.安装步骤

### 2.1 apollo7.0 安装编译

apollo7.0 安装可以参考以前的一篇文章。不同的是使用apollo.sh编译工程时候，要带上gpu,使用如下命令完成。

```text
cd apollo/
bash docker/scripts/dev_start.sh
bash docker/scripts/dev_into.sh
sudo bash apollo.sh build
```

参考以前的文档

### 2.2 apollo端启动

```text
bash ./scripts/bootstrap.sh start
bash ./scripts/bootstrap_lgsvl.sh #启动dreamview的监视器
bash ./scripts/bridge.sh#桥接，websocket,端口默认9090
```

![image-20220327110829082](D:\自动驾驶\AutoDriving\note\image-20220327110829082.png)

## 3. lgsvl端安装配置

### 3.1 下载lgsvl压缩包

下载lgsvl的安装包（[下载地址](https://link.zhihu.com/?target=https%3A//www.svlsimulator.com/)），lgsvl安装很简单，就是压缩包，解压缩之后就完成了安装。在解压缩目录下，终端输入后，显示界面如下。已经使用过和注册过了

```text
./simulator
```



![image-20220327110352028](D:\自动驾驶\AutoDriving\note\image-20220327110352028.png)

### 3.2 lgsvl 用于 apollo调试的配置

![image-20220327110522097](D:\自动驾驶\AutoDriving\note\image-20220327110522097.png)



如下述图片，你的配置就成功了，下一步就是run simulation就可以了

![image-20220327110936887](D:\自动驾驶\AutoDriving\note\image-20220327110936887.png)

与仿真器对应，可以通过键盘上的方向键，控制车的行动。

![image-20220327114206347](D:\自动驾驶\AutoDriving\note\image-20220327114206347.png)

**效果演示**



![1648353003952](D:\自动驾驶\AutoDriving\note\1648353003952.gif)

## 4. 总结

完成上述步骤后，你也就完成了lgsvl仿真器和apollo的连接了，在后续调试中，可能需要在仿真器中设置相关的硬件部署，可以参考官方文档！祝大家用的开心！

# 参考

https://zhuanlan.zhihu.com/p/438701298?utm_source=wechat_session&utm_medium=social&utm_oi=1066664251602550784&utm_campaign=shareopn

