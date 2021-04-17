---
title: PFC简介
author: lijiabao
date: 2021-04-14 20:30
categories: ["PFC笔记", "数值模拟"]
tags: ["PFC笔记", "数值模拟"]
---

简要介绍PFC（Particle flow code）颗粒流方法，以及PFC中的一些重要术语和概念

## PFC简介

>  PFC（颗粒流方法）：基于通用离散元模型（DEM）框架，由计算引擎和图形用户界面构成的细观分析软件。

PFC主要用于模拟有限尺寸颗粒的运动和相互作用，而颗粒是带质量的刚形体，可以平移和转动。颗粒内部通过内部惯性力、力矩，以成对接触力方式产生相互作用，接触力通过更新内力、力矩产生相互作用.

## PFC模型基本组成

>  PFC模型中的每一个颗粒被表示为实体，它不是一个点质量，而是一个带有有限质量和定义了表面的刚形体。

PFC模型主要由三部分组成：

- 实体（body），有三种类型：每个实体由一个片或者多个片组成，球只有一个片。<font color='red'>所有的实体都只能出现在domain区域，不能超出这个区域</font>
  1. 球（ball）：只有一个片。球在三维里表现为球体，二维里表现为单位厚度的圆盘
  2. 簇（clump）：簇的片叫做pebble，簇是pebble的集合。三维里是许多球体，二维里是许多单位厚度的圆盘。组成簇的pebble可以相互重叠，簇的实体内部没有接触，但是接触可以在不同实体间的片产生
  3. 墙（wall）：墙的片叫做facet，墙是facet（面）的集合。facets在二维表现为线段，三维表现为三角形，由facet可以构成任意复杂的空间多边形。
- 片（piece）：pebble和facet
- 接触（contact）：颗粒的相互接触接触相互作用定律，通过软接触方法实现的。所有的变形都只能产生于刚性实体接触

球和簇的运动遵守牛顿运动定律，但是墙的运动是用户指定的。因此只有球和簇有质量特性（质量、中心位置和惯性张量）和加载条件（接触的力和力矩，重力，外部作用力和力矩）

### PFC的常用术语

熟悉并且区分以下术语，可以更快的了解PFC方法：

1. domain（范围）
   domain表示的是一个区域，用来进行接触检测判断。PFC5.0中，所有的对象都在给定的domain区域中。
   domain的四种边界条件类型：

   1. stop：当颗粒、wall、clump等碰到边界之后，停止运动
   2. destroy：碰到边界直接删除
   3. reflect：碰到边界速度方向，往回弹
   4. periodic：从domain相对面（另一面）重新出现，用于均匀化方法。按照维度设置x，y，z三个方向的范围，只设置一个的话，其他的默认和设置的一样。

2. bodies（实体）和pieces（片）
   PFC实体有三种（ball，wall，clump），每个实体由一个或者多个piece构成，piece是用来进行接触检测判断的。每个body上所有的piece的计算数据都存储于该body上，用来进行系统运动方程的积分求解。

   **piece接触类型**：ball-ball、ball-facet、ball-pebble、pebble-pebble、pebble-facet
   接触类型顺序依次是：ball、pebble、facet

3. wall和facet
   wall是一系列的facet构成，每个facet具有2个或者3个端点，这些端点统称为wall的顶点（vertex），使用wall.vertex.list遍历查询，flac3D 6.0和PFC3D 5.0之间的耦合也是基于这个进行的

4. clump和pebble
   构成刚性簇的球称之为pebble，接触类型中的pebble-pebble指的是不同刚性簇之间的接触，不包括同一簇内部球体之间的接触

5. cluster和clump
   cluster指一组ball通过特定的设置利用接触相互黏结、密实，表现出簇的特性，但是之间的接触有限，在外力足够下颗粒又可破碎，称之为柔性簇
   clump指一系列球叠加在一起，无论什么条件，各pebble之间无相对变形，从而呈现出刚性颗粒运动的簇，称之为刚性簇。

6. DFN和fracture
   DFN（discrete fracture network）离散裂隙网络，fracture指的是单一裂隙，DFN是fracture的集合

7. damping
   阻尼（damping）是用来消耗系统内部能量。可以通过三种方式消耗内部能量：

   1. 摩擦
   2. 接触中的黏壶（dashpot）部分
   3. 在运动方程中设置局部阻尼（local damping）：静态求解，设置较大的局部阻尼，加快计算平衡，动力求解时，需要设置合理的局部阻尼。设置阻尼：ball attribute和clump attribute加关键字damp设置局部阻尼大小`ball attribute damp 0.7`类似的命令。PFC4.0默认0.7局部阻尼，PFC5.0默认设置为0
   
8. 属性（attribute）和接触特性（property）
   attributies是模型实体的内在特性，如：位置、速度和大小等。实体、片和接触都有attribute。
   properties仅仅应用到到piece（片）上，是用户指定的字符串和值的列表，这些字符串和值代表了piece的表面条件，properties可以决定piece之间如何相互作用，接触模型可以使用piece properties来决定结果的相互作用。properties通常用于属性继承，作为<font color='red'>接触参数设置的方法之一</font>

9. 时间步（timestep）
   采用的是自动时间步测定算法来评估时间步，确保运动方程的稳定求解。PFC5.0中是基于片（piece）速度进行
   PFC固定时间步（使用`set timestep fix`），根据固定时间步和指定最大时间步值（使用`set timestep maximum`命令）

10. 接触模型分配表（CMAT）
    PFC5.2中不假定接触服从线性接触行为，默认插入一个null接触模型，直到指定其接触模型。PFC5.0中有一个接触模型分配表（CMAT），和domain一样，每个数据文件将会以`cmat default `命令开始指定默认的力学行为，对于复杂模型创建更方便。

