# Win10和Ubuntu双系统搭建



# ubuntu镜像的选择：i386还是amd64？

只取决于电脑CPU位数

i386是32位CPU指令集架构，命名来由是基于intel 80386

amd64是64位CPU指令集架构，命名来由是amd发明了此架构

命名中的i和amd只是和来由有关，不是限制了厂商平台

**因此，虽然我的电脑是Intel的cpu，但是由于是64位的，因此选择amd64**



1. 双系统搭建

2. https://blog.csdn.net/qq_37674858/article/details/80678927

    [EasyBCD.zip](E:\easyBcd\EasyBCD.zip)  （推荐使用）

   分区引导使用EasyBCD

   

   https://www.jb51.net/article/173277.htm

   ![image-20210921114655198](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20210921114655198.png)

![image-20210921114717217](C:\Users\HW\AppData\Roaming\Typora\typora-user-images\image-20210921114717217.png)



# 问题

 Ubuntu安装日常踩坑——Ubuntu安装过程中分区时出现空闲空间不可用的情况



       我是从windows7下的硬盘里面（非C盘）压缩出了一个新加卷，此时我Windoes下的的硬盘情况时是C、D、F和压出的一个未分配。然后在安装Ubuntu过程中进行分区时，我首先选中200G的空闲分区，点击’+‘进行编辑分区，分完400M的/boot（主分区）后，剩下的空间就变成了不可用了。


原因分析:

linux系统最多只能有4个主分区，说明此时你电脑里的硬盘已经有4个主分区了，或者是ubuntu认为你有4个主分区了。


解决方案：
1.更改分区顺序，先设置逻辑分区，后设置主分区



# 修复Ubuntu18.04与Windows 10双系统丢失grub引导界面

https://blog.csdn.net/weixin_39212776/article/details/81239805
