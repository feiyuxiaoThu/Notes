# 城市NGP感知融合算法

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/1.jpeg)

## 小鹏自动驾驶发展历程

+ 2015-2018, prototype: APA （Automatic Parking Assistance, 自动停车辅助)
+ 2018-2019， G3-XPILOT 2.5 : APA/ACC/LCC/ALC, first mass production APA. vision+radar fusion solution。（高速L2-L3）
+ 2020-2021， P7-XPILOT 3.0 : VPA（Valet Parking Assist， 停车场记忆泊车）/ Highway NGP（高速领航）. Only full stack self-research autonomous driving software China OEM.2021-2022， P5-XPILOT 3.5. City NGP (城区)， end-to-end autonomous driving. (高速NGP->城市NGP)
+ 2022-2023, next model-XPILOT 4.0. L4 Autonomous Driving. Partial scenes no driver control.

说明：

+ NGP （Navigation Guided Pilot, 高速自主导航驾驶）: 基于用户设置的导航路线，实现从A点到B点的自动导航辅助。最开始高速上实现，车辆自主实现超车变道，上下匝道，切换高速等功能。
+ CNGP (City Navigation Guided Pilot): 第一个装备了LiDAR的量产城市City NGP。基于驾驶员在城市驾驶场景中设置的导航路线，辅助驾驶员对车辆从点A到点B的导航。

## City NGP 技术栈

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/2.png)

+ 静态环境（定位）

  基于Camera， IMU, GPS, Wheel Odo硬件模块，和Localization, Camera/Lidar Perception, Map Fusion (HDMAP)软件模块， 实现基于高精度地图的**厘米级高精度定位**，获取自车在世界中的位置。

  ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/3.png)

+ 动态环境（感知）

  融合多个传感器的感知，实现对周围360度环境稳定、可靠、一致的描述。（对agents实现厘米级定位）

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/4.png)

+ 交互（对人类司机的预测和本车的决策规划与控制）

  基于精确预测的类人驾驶行为

  ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/5.png)

  ## CNGP 主要传感器 (P5)

  + 2 **Lidar units**, 左右前，150度视角

    ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/21.png)

  + 12 **ultrasonic sensors**, 泊车探测不明物体

    ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/22.png)

  + 5 **millimeter-wave radar** (前方，四个角落， 360度)

    ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/23.png)

  + 5 **monocular vision cameras**， 用于行车的摄像头 （两侧+后向）

    ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/24.png)

  + 4 **surround view cameras** （环视）

    ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/25.png)

  + 1 **trinocular front-view camera** (前向三目摄像头)

    ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/26.png)

  + 1 **driver monitor camera**, 司机检测

    ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/27.png)

  + 360度 **dual-perception fusion**

    ![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/28.png)

## 感知融合

### Camera-Camera Fusion

将不同相机的检测bbox进行关联，得到目标更准确的3D表示。

左前，前视，右前 左后，俯视， 右后

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/31.png)

### Radar-Camera fusion

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/32.png)

+ 匹配用的度量：2D空间距离差异（雷达点投影）+ 3D空间距离差异
+ 左侧窗口中，很多点表示雷达探测点，有箭头的表示前向雷达探测目标（箭头表示速度。）
+ 右侧窗口中黄色圆圈表示雷达目标投影到图像，其中数字是雷达目标的id。
+ radar target measurement中的obstacle probability不太可靠，只供参考。rel表示relative。
+ radar有很多虚假目标，需要区分。
  

### LiDAR-Camera fusion

从P5开始使用Lidar, Lidar点云目标检测生成3D bbox, 与视觉检测关联。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/33.png)

> 以上介绍的是传感器单帧观测的融合，然后需要将融合观测与追踪的目标进行数据关联，然后更新追踪目标的运动状态，实现稳定连续的追踪效果。

## 状态估计

### 坐标变换和运动补偿

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/41.png)

+ 左上：不同传感器观测融合需要先将观测转换到统一坐标系。坐标系变换将目标从一个坐标系转换到另一个坐标系，需要旋转+平移。（左上角）
+ 左下：毫米波雷达测量的目标相对位置X和相对速度V（in radar frame）。V是X的导数。
+ 右：恢复目标在世界坐标系的位置和速度。位置Position使用坐标转换，从radar frame转换到world frame。线速度 linear velocity等于对位置求导。
  

### 数据关联

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/42.png)

+ 两个层级
+ track to measurement association ： 跟踪过程中
+ measurement to measurement association: track初始化时，关联多个传感器观测，生成观测完整的track; 或同一个传感器不同时间观测的关联
+ 经典算法：匈牙利算法（Hungarian method）

## 扩展卡尔曼滤波

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/43.png)

## Case study: 静态目标检测 —— 为什么静态目标检测非常重要且充满挑战？

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/51.png)

+ 大多数ADAS Radars没有足够的垂直分辨率。（容易分不清车辆和上方的立交桥/路牌等）

+ Radar目标可能是静态道路用户（stationary road users）和静态环境(static environment)。（很难区分静态障碍物和静态环境）

+ 单帧视觉对深度、尺度的感知很困难。（深度学习，corner case）

+ 高速行驶（120km/h）检测到静止目标，为了稳定可靠停在目标后面，需要检测距离在140m以上，才能更安全舒适地停车。

  > 对于上述问题，如何解决**融合时，探索多传感器的时空相关性？**

### 对Radar目标进行视觉校验

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/4-pics/52.png)

+ 只有当camera和radar有若干次成功关联后（confirmed），才发布这个radar目标到下游模块。
+ 为了防止发布的radar目标是特殊情况下产生的误检（比如快速驶过路牌的车辆），在目标可视时，进行周期性地视觉关联。即要求视觉目标与radar目标有个持续一致的关联。
+ 使用视觉的box检测序列和滤波器（根据运动变化）估计深度和速度。只有当状态收敛，才会发布目标。当视觉出现测距偏差（常出现于外表类似，大小不同的车辆，比如工程车），用已经追踪上的radar目标进行深度纠正。

## 问答

1. **传感器延时对融合的影响？不同传感器的延时是多少？**

   camera,lidar 几十毫秒左右。radar的delay比较大，10hz左右。感知+融合一共在100ms内完成。

2. **大加速度下，融合如何稳定跟踪？**

   当汽车做急减速影响大，特别是毫米波雷达内部对信号进行滤波处理，一般存在相位上的delay。建议在急减速时，适当提高做完运动补偿的加速度的协方差，因为radar相位差导致运动补偿后的加速度不准。

3. **有没有使用双目测距？**

   取决于其他传感器的配置，有了radar和lidar，双目优势不大。多用三目，不同视角+景深。

4. **判断融合相比单一传感器的效果提升？**

   融合结果的概率分布的方差更小，消除不确定性。需要平衡性能和成本。

5. **远处纯视觉的目标如何保证速度的准确性，不造成误刹车？**

   远处目标比较难。对于太远目标，纯视觉产生目标时限制在低速范围内，利用追踪的多帧历史结果对初步速度范围进行估计（不是单帧的），这个估计不用太准，因为对于快速的自车速度，目标0.1m/s和2m/s差别不大，可以发布出来。对于远处快速运动目标，lidar捕获生成目标，然后视觉进一步修正。

6. **高精地图在sensor fusion中主要解决哪些问题？**

   之前，滤除雷达杂波，路段信息验证目标合理性。现在城市场景用的少。

7. **小鹏 运动模型使用交互多模型IMM吗？后融合中，数据关联采用概率数据关联吗？有采用RFS(Random Finite Set)方法吗？**

   感知融合没使用IMM。预测规划有IMM。数据关联不使用概率模型，概率模型平均性能比较好，但是对于量产车不够。主要用自己定义的metric，distance。