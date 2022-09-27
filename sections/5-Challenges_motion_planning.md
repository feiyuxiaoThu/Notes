# 自动驾驶规划中的挑战

## 轨迹生成

motion planning算法是从机器人领域发展起来的，逐渐发展出适用于自动驾驶领域的各种算法。自动驾驶的轨迹生成主要有下面几种：

+ 基于采样搜索方法：Dijkstra、RRT、A*、hybird A*和Lattice等
+ 基于曲线插值：RS曲线、Dubins曲线、多项式曲线、贝塞尔曲线和样条曲线等
+ 基于最优化方法：Apollo的piece wise jerk等

上述方法一般都是相互结合一起使用的，比如多项式曲线需要对终端状态进行采样，贝塞尔曲线对控制点采样，hybird A*中使用到了RS曲线或者Dubins

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/11.png)

下图是各种轨迹或者路径生成方法的优缺点，可见没有哪种方法是完美的，因此不同的工况下需要使用不同的算法。目前行业内主要应用的是多项式插值(高速)和最优化的方法。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/12.png)

## motion planning问题与挑战

上述Motion Planning的方法，基本是解决轨迹生成问题，不同的方法适用于不同的场景。目前行业内的轨迹生成方法已经不是主要瓶颈，并且主流是采用最优化的方法。但是在Motion Planning领域内仍然存在许多挑战需要去攻克。主要分几个方面介绍。

- 最优性问题；
- 认知推理问题；
- 不确定性问题(Uncertainty/Probability)；
- Sigle Agent;
- Multiple Agent;
- 工程化问题；

### 最优性问题

全局最优是NP-hard问题，为了实时性，行业内多数采用横纵向解耦的规划方法，但是这么做会牺牲最优性，在一些工况下不能得到良好的车辆行为，比如在超车、对向来车、向心加速度约束处理、横向规划需要考虑纵向规划能力等。例如，当Ego Vehicle前方有一个减速行驶车辆时，横纵向解耦的方法一般只有当前方车辆车速降低到一定值时才会超车行驶，Ego Vehicle的行为表现就是会减速甚至停车，然后再绕障行驶，显然不是最优的行驶策略。如果采用时空一体化规划方法，则可以避免减速或者停车行为。下图中左图是解耦方法的示例，在前方有减速停车车辆是，ego vehicle会进行减速，右图是时空规划的示例，在前方车辆减速时会进行超车。
![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/21.png)

### 认知推理问题

#### 地图拓扑推理

以Apollo为例，PNCMap模块从HDMap提取数据形成参考线，并且通过HDMap API查询道路元素，但同时Planning模块也忽略了一些道路的拓扑关系，例如汇入汇出路口，而特殊的道路拓扑会影响到车辆的行为。此外，在没有HD Map而依靠视觉车道线的情况下，此时感知车道线会发生异常。在汇入汇出道路和十字路口道路中，其道路拓扑问题尤为凸显。
![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/221.png)

#### 障碍物统一建模

交通场景的参与者有车辆、摩托车、自行车、行人、锥桶等，广义上来讲还包括人行横道、红绿灯、道路限速等地图静态元素，Motion Planning需要针对不同的元素做出不同的决策。障碍物统一建模可以简化问题，并且提升计算效率。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/222.png)

+ Aopllo将所有交通参与者抽象为Static Obstacle，Dynamic Obstacle和Virtual Obstacle，obstacle就是box， Static Obstacle和Dynamic Obstacle为车辆、行人等， Virtual Obstacle为人行横道、禁停区等。路径规划时不考虑Virtual Obstacle。

+ 使用能量场相关的方法，将交通参与者使用能量函数表示。上图中间图就是清华

  [^1]: 基于动态行车安全场的智能网联汽车决策规划方法研究

  提出的行车安全场，由静止物体的势能场、运动物体的动能场和驾驶员的行为场构成。最优轨迹就是寻找一条能量和最小的轨迹。

+ 论文

  [^2]: W. Ding, L. Zhang, J. Chen and S. Shen, “Safe Trajectory Generation for Complex Urban Environments Using Spatio-Temporal Semantic Corridor,”

  将交通参与者分为obstacle-like和constraint-like， obstacle-like是动静态车辆、红灯等，将其映射到slt的3D栅格中， constraint-like是限速、停车标志等作为semantic boundary。根据决策序列动作在slt配置空间内生成若干cube边界供轨迹生成使用。

#### 场景认知推理

由于现实中环境的复杂性，一种决策策略或者规划方法难以处理不同的工况，因此对行驶环境进行分类，在不同的场景下选择不同的策略可以提升motion planning的性能。那么怎么进行场景分类和场景识别，在不同的场景motion planning又该有哪些不同？这些问题都是需要解决的。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/223.png)

+ Apollo中场景分类为LANE_FOLLOW、SIDE_PASS、STOP_SIGN_UNPROTECTED等，有两种场景识别方式，一是通过规则的方法，一是通过机器学习的方法。不同的scenario有不同的stage，stage中以此执行task，即使是相同的task在不同scenario中参数配置也可能不同。
+ 毫末将城市场景路口多、拥堵多和变道多的特点，将行驶场景分为十类，显然是和Apollo中的scenario分类是不同的，然而在毫末的场景识别方法却不得而知。此外还提出了行驶环境熵的概念来描述行驶环境的拥堵状态。

### 不确定性

#### 定位不确定性

在多数的motion planning中都是认为定位是准确，但是实际中由于遮挡、多径干涉等问题，定位往往是不准确的，以论文

[^3]: Artunedo, Antonio, et al. “Motion planning approach considering localization uncertainty.” IEEE Transactions on Vehicular Technology 69.6 (2020): 5983-5994.

中的左下图所示，由于定位误差导致从HDMap查询到的道路边界产生误差，从而使规划和车辆行驶轨迹在道路边界上。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/231.png)

论文将定位不确定性假设为高斯分布，并且定位模块可以计算出概率分布的期望与方差。论文将车辆坐标系转换到了UTM坐标系下，根据定位的高速分布情况和坐标变换公式，就可以计算出车辆周围环境在定位影响下的不确定性，如上右图所示，其中颜色越深表示不确定性越大，其不确定性计算公式主要由下式得到。可以发现距离Ego Vehicle越远其不确定越高，随着车辆的前进，其不确定性会被更新。路径规划方法采用了Lattice（五次多项式曲线）的方法，在cost计算时，增加了两个项目。一个是硬约束：规划路径上点的最大不确定性不能大于某一个阈值；二是在cost function中增加了不确定性的权重和。
![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/231_.png)

#### 感知不确定性

由于传感器噪声、车辆震动、行驶环境和不完善的算法，感知得到结果具有不确定性，甚至是错误的。由于感知的不确定性会造成motion planning结果的不安全性。一种简单的处理方式是加buffer，但是粗暴的处理方式会减小motion planning的可行域，可能造成过于激进或者过于保守的行驶策略。
论文

[^4]: Lee, Seongjin, Wonteak Lim, and Myoungho Sunwoo. “Robust parking path planning with error-adaptive sampling under perception uncertainty.” Sensors 20.12 (2020): 3560.

以装备了Around View Monitoring(AVM)的泊车应用为例，由于感知误差会使路径规划在实际超车停车位置，可能会发生碰撞。如下左图所示。论文将感知的不确定性建模为高斯分布，感知效果距离ego vehicle越远不确定性越高，如下右图所示。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/232.png)

以论文中整体架构如下左图所示：

+ Parking space sampling：对距离ego vehicle最近的两个角点进行采样，将采样点看作是正态分布的，根据采样角点和设定的停空间的长度，计算ego vehicle后轴中心的停车点；
+ Path candidate generation：采用ocp理论对每个采样点进行路径规划，其中将时域问题转化为Ferent坐标系下，并使用SQP求解非线性问题；
+ Optimal Path Selection：使用utility theory进行最优路径的选择。Utility function为$EU(S) = P(s) \times U_{ideal}(s) + (1 - P(s)) \times U_{real}(s)$, 其中 $P(s)$ 为路径对应采样点的概率， $U_{ideal} $ 为路径到目标点（当前时刻感知检测到的，并非采样得到的)的偏差效用函数值，$U_{real}$ 为路径到 ego vehicle 当前位置的效用函数值。
+ 为路径上到ego vehicle当前位置的效用函数值。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/232_.png)

#### 预测不确定性

预测是实现L4以上高级别自动驾驶的重要环节，然而目前为止，预测对整个行业都是一个非常难的问题，因此预测的准确性很差，在不确定性预测结果下做motion planning是非常重要的。
论文

[^5]: W. Xu, J. Pan, J. Wei and J. M. Dolan, “Motion planning under uncertainty for on-road autonomous driving,” 2014 IEEE International Conference on Robotics and Automation (ICRA), 2014, pp. 2507-2512, doi: 10.1109/ICRA.2014.6907209.

提出了一个基于高斯分布的规划架构，处理预测和控制不确定性带来的规划轨迹不安全的问题。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/233.png)

+ 候选轨迹生成：通过多阶段横纵向采样生成。可以理解为Aopllo Lattice方法。

+ 预测轨迹生成：对于某一个车辆的轨迹进行预测(进行规划)时，认为其他车辆是匀速行驶的，并且其状态都是确定的，则通过对候选轨迹的cost计算，得到最优的预测轨迹。之后通过卡尔曼滤波计算预测轨迹的概率分布，并假设其遵从正态分布。

+ Ego vehicle轨迹生成：此时需要考虑其他交通参与者的预测的不确定性。针对每一条候选轨迹，通过LQR算法计算出控制误差，然后再通过卡尔曼滤波计算出轨迹的概率分布，在轨迹评价进行cos计算时，碰撞检测是基于预测和ego vehicle规划轨迹的概率分布的，即在所有概率分布内都不能发生碰撞。

  作者认为此方法相当于给box加上一个自适应的buffer，而常规的固定大小的buffer会导致保守或者激进的驾驶行为。

  论文

  [^6]: Pek, Markus Koschi Christian and Matthias Althoff. “An Online Verification Framework for Motion Planning of Self-driving Vehicles with Safety Guarantees.” (2019).

  提出了一种可以嵌入现有motion planning框架的fail-safe机制，分为三部分：

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/233_.png)

+ Set-based prediction：根据制定的交通参与者的驾驶策略和车辆运动学模型，将原有的交通参与者单一的预测轨迹，改为多预测轨迹；

+ Fail-safe trajectory：根据预测的结果，计算原planning trajectory有碰撞风险的第一个轨迹点，然后再根据最优化理论生成轨迹；

+ Online verification：将ego vehicle在第二步生成的轨迹上进行投影，判断其是否和第一步的预测车辆轨迹是否有碰撞。

  感觉此方法是又重新做了一遍motion planning，由于论文中没有描述fail-safe trajectory是否考虑decision的结果，可能会造成safe trajectory不满足decision结果，并且此论文只是仿真，并没有实际应用。
  

#### Partially Observable Environments

由于传感器自身的感知范围受限和感知结果的不确定性，在不良光照或者恶劣天气中会进一步放大。而在城市工况中，建筑物的遮挡会造成不完全感知，如下图所示。此外，大型车辆也会造成感知遮挡问题，而多数的motion planning都是以完全感知进行处理的，规划结果具有很大的不安全性。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/234.png)

论文

[^7]: Ö. Ş. Taş and C. Stiller, “Limited Visibility and Uncertainty Aware Motion Planning for Automated Driving,” 2018 IEEE Intelligent Vehicles Symposium (IV), 2018, pp. 1171-1178, doi: 10.1109/IVS.2018.8500369.

提出了一种处理不完全感知的安全的motion planning，使规划轨迹在最危险情况下可以在车辆最大制动能力下安全停车而不发生碰撞。分为两种情况：一是在直道上行驶考虑感知的不确定性和感知距离范围，二是在城市十字路口考虑不完全感知情况。并且容易嵌入其他的motion planning架构中，作者在其之前提出基于最优化方法的轨迹规划中进行了仿真验证(综述中的图(b))。
作者为其理论设计了几个假设：

+ 定位的纵向位置和速度信息遵从高斯分布；

+ 感知的有效范围是已知的，并且感知的结果遵从高斯分布；

+ 地图信息中包含建筑物位置，且为凸多边形；

+ 使用Intelligent Driver Model(IDM)进行车辆加速度预测；

  由于论文分直道和十字路口两种情况处理，因此需要进行场景识别，论文采用了基于规则的方式进行场景识别。

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/234_.png)

+ 上左图：红色虚线是感知观测到的环境的时间，黑色虚线是进行planning的时间，可见planning使用的感知信息是$t_p$ 时刻前的。此外由于 planning 需要保证连续性，在计算周期 $t_{plan}$ 中的规划轨迹要保证一致。更重要的是由于执行器的延迟，在 $t_{safe}$ 时间内要保证轨迹的安全性，论文中设定 $t_{safe} = 2t_{pin}$
+ 上中图：在直道行驶分为感知范围内没有车辆或者感知范围内有车辆两种情况：一是感知范围内没有车辆，假设驾驶感知范围外有一个静止车辆，将其设为虚拟静止障碍物，通过其高斯分布特性可以计算得到$t_{safe}$ 时间内，满足以最大制动能力刹车的纵向位移和速度约束；二是感知范围内有车辆，考虑感知不确定性情况下的最危险情况，即前车以最大制动能力刹车，通过其高斯分布特性可以计算得到$t_{safe}$ 时刻内，满足以最大制动能力刹车的纵向位移和速度约束；
+ 上右图：在十字路口行驶，根据IDM模型计算ego vehicle是需要让行还是有路权需要明确的表明自己优先通过的意图。最后转换为直道行驶的两种类型的约束。

### Single Agent

Single Agent认为是单智能体问题，即ego vehicle会对周围环境做出决策，而不考虑ego vehicle决策对其他交通参与者的影响，显然这种假设是不对的，但是却简化了motion planning问题。
决策是影响自动驾驶发展另一个重要方面，随着自动驾驶的等级越高，决策的重要性越高。决策的难点是体现无人驾驶车辆的智能性，如何使无人驾驶车辆可以像人类驾驶员一样处理高维度、多约束的复杂场景，甚至要比人类驾驶员的表现更好。目前多数方法使基于规则的方法，其能力有限。以基于规则方法的决策设计来说，在下匝道工况，一般会设计一个距离匝道口的距离阈值，当ADC到匝道口的距离在阈值内时，就开始向最右侧车道变道。假设这个阈值是300m，如果ADC在匝道口前350m处中间车道行驶，由于前车速度较低，自动决策变道一般会向左侧车道变道(左侧车道限速高，超车遵从左侧超车，从小鹏NGP等可以看出也是左侧车道优先)，但是由于匝道原因，变道后需要向最右侧车道变道，会进行三次变道，会显得不够智能。再比如在匝道前300m最右侧车道行驶，前方由于施工或者事故不能行驶，此时只能由驾驶员接管。由此可见，由于现实工况的复杂性，基于规则的方式很难做到良好的驾乘体验。

港科大关于POMDP

[^8]: L. Zhang, W. Ding, J. Chen and S. Shen, “Efficient Uncertainty-aware Decision-making for Automated Driving Using Guided Branching,” 2020 IEEE International Conference on Robotics and Automation (ICRA), 2020, pp. 3291-3297, doi: 10.1109/ICRA40945.2020.9197302.

的决策工作，相比于基于规则的方法，性能有了一定提升，其对主车和其他交通参与者的行为进行了剪枝，降低了OPMDP的耗时。但是其考虑了其他交通参与者会对ego vehicle的行为进行规避等，可以看出是一个Multiple Agent问题来处理。

### Multiple Agent

上述的Single Agent中认为交通参与者不会对主车的行为做出相应的决策，但实际中，当主车做出决策后，其行为会影响到其他交通参与者的行为，而使原有的预测结果的可信性降低，尤其是有些简单基于规则的prediction不依赖于planning结果，或者使用上一帧planning的结果(Apollo)。
当主车L沿着trajectory1行驶时，A2可能会减速避让，当主车沿着trajectory2行驶时，A2可能会加速通过路口。但是当主车沿着trajectory2行驶时，预测A2可能会加速通过路口，但是A2可能会理解错主车L的意图进行减速，会造成两辆车锁死。因此主车怎么理解其他交通参与者的意图和怎么让其他交通车理解主车的意图至关重要

[^9]: Autonomous vehicles’ intended cooperative motion planning for unprotected turning at intersections

![](https://github.com/feiyuxiaoThu/Notes/blob/master/media/5-pics/25.png)

### 工程化问题

在planning中还面临一些工程化问题：

+ 实时性：在第一个问题中提高了最优性问题，如果要处理最优性，由于在三维空间搜索计算的复杂性，其实时性很能保证，这也是限制时空联合规划应用的一个原因。此外最优化算法中的“大规模约束”和非线性也面临实时性的挑战；
+ 完备性：插值、Lattice等算法是概率完备的，尤其在复杂多障碍物环境中，有限的采样很难获得无碰撞的轨迹。而最优化方法由于数值求解，也不能达到完备性，常用的osqp求解器甚至会给出一个错误的解；
+ 难量化性：planning中的评价指标多是主观性的，比如舒适性和通过性等，很难量化评价。不同工程师调参得到体感不同，又与乘客的主管感受不同。因此提出了机器学习的方法来学习planning中的参数或者变道策略。

### 行业解决问题

针对上述问题，行业内公司也在探索并提出了自己的方案。

+ 轻舟智航采用了时空联合规划解决最优性问题，提高规划性能，并且自研了非线性规划器高效求解
+ 图森新一代框架，感知在提供障碍物位置、速度等信息时，同时提供不确定性或者概率信息，以保证决策规划可以提前做出安全舒适的决策
+ 特斯拉将planner用于交通参与者的其他车辆，与其他车辆交互时，不能只为主车规划，而是要为所有交通参与者共同规划，针对整体场景的交通流进行优化。为了做到这一点，会为场景中的每个参与对象都运行autopilot规划器。除此之外，针对停车场景，采用$A^*$搜索算法和神经网络结合策略，大大减少了节点探索
+ 小鹏和特斯拉针对车道线缺失，道路拓扑变化问题做了优化
+ Waymo提出了ChauffeurNet提升决策性能，Apollo借鉴ChauffeurNet提出了自己的强化学习架构。
