---
title: PFC命令简要介绍
author: lijiabao
date: 2021-04-21 22:30
categories: ["PFC笔记", "数值模拟"]
tags: ["PFC笔记", "数值模拟"]
---

介绍PFC的命令，以及命令的功能

## PFC命令指南

### ball(球）相关命令

- `ball attribute`：设置球体的固有属性，如位置、大小和速度等
- `ball create`：使用指定的属性创建单个ball
- `ball delete`：删除球体命令
- `ball distribute`：可重叠的分布球体，可指定按照指定的distribution(分布)来分布球体位置
- `ball extra`：设置球体的额外变量
- `ball fix`：固定指定球体的速度
- `ball free`：释放指定球体的速度，球体的运动就反应了作用在球体上面的force
- `ball generate`：无重叠的分布生成球体
- `ball group`：指定球体ball的组名
- `ball history`：添加一个球体history。history是就是在模型过程中定期对模型属性进行取样而指定的浮点值表
- `ball initialize`：修改球体的属性，和`ball attribute`命令类似
- `ball list`：列出球体相关的信息
- `ball property`：设置球体面特性，主要是用在piece上的，用来决定piece之间怎么相互作用
- `ball result`：修改ball  result logic的使用，允许在不同时间实例下将模型状态state保存到内存或者输出为2进制文件。这个ball result可以用来重新创建模型，还可作于有效后处理
- `ball tolerance`：设置接触关系探测的容许范围
- `ball trace`：添加一个球颗粒的踪迹信息。球的位置和速度在模型运行的时候可以被取样和存储，并且通过空间的路径可以被看到。
- `list ball`：列出球的信息
- `trace ball`：添加球粒的踪迹信息，这个命令是和`ball trace`相同的命令
- `ball ccfd list`：列出ccfd球体的信息。ccfd是：组合了DEM计算和可计算的流体动力的一个组件
- `ball thermal attributes`：设置球体温度参数的值
- `ball thermal extra`：设置球体温度的额外参数
- `ball thermal fix`：固定球体的温度
- `ball thermal free`：释放球体的温度
- `ball thermal group`：指定特定温度的球体的分组名
- `ball thermal history`：添加一个球体的温度历史
- `ball thermal initialize`：修改球体温度的属性
- `ball thermal list`：列出温度球体的信息
- `ball thermal property`：修改温度球体的面属性
- `history ball thermal`：修改球体的温度历史
- `list ball thermal`：列出温度球体的信息



### clump（簇）相关命令

- `clump attribute`：设置簇的属性值
- `clump create`：使用指定的属性创建一个球簇
- `clump delete`：删除球簇或者pebble
- `clump distribute`：有重叠的分布球簇
- `clump extra`：设置球簇或者pebble的额外变量信息
- `clump fix`：固定指定的球簇的苏苏
- `clump free`：释放指定球簇的速度
- `clump generate`：生成非重叠的球簇或者pebble
- `clump group`：将指定球簇或者pebble的组名
- `clump history`：添加球簇的历史信息
- `clump initialize`：修改球簇的属性值，可attribute类似
- `clump list`：列出球簇的信息
- `clump order`：设置旋转EOM的顺序
- `clump property`：指定簇pebble面特性（surface property）
- `clump replicate`：从模板中创建一个簇
- `clump result`：修改簇计算结果的使用
- `clump rotate`：旋转簇
- `clump scale`：缩放簇
- `clump template`：在单个簇模板上进行操作
- `clump tolerance`：设置接触探测的容忍度容许范围
- `clump trace`：添加一个簇或者pebble颗粒的踪迹trace
- `history clump`：添加一个簇历史信息
- `list clump`：列出clump的信息
- `trace clump`：添加一个簇或者pebble颗粒的踪迹信息trace
- `clump ccfd list`：列出ccfd簇的信息。CCFD：整合了力学DEM计算和计算流体力学的代码而成的组件
- `clump ccfd attribute`：设置ccfd簇的属性
- `clump ccfd extra`：设置ccfd簇的额外变量信息
- `clump ccfd history`：添加一个ccfd簇的历史信息
- `clump ccfd group`：指定一个ccfd簇的组名
- `clump ccfd initialize`：修改ccfd簇的属性
- `clump thermal attribute`：设置温度簇的属性
- `clump thermal extra`：设置温度簇的额外变量信息
- `clump thermal group`：指定温度簇的组名
- `clump thermal fix`：固定指定温度簇的速度
- `clump thermal free`：释放指定温度簇的速度
- `clump thermal history`：添加一条温度簇的历史信息
- `clump thermal initialize`：设置温度簇的属性
- `clump thermal list`：列出温度簇的信息
- `clump thermal property`：设置温度簇的面特性（surface property）
- `cmat add`：为CMAT（接触模型属性分配表）添加一个新的连接接入口（entry）
- `cmat apply`：应用接触模型分配表到现村的接触上
- `cmat default`：设置CMAT的默认插槽（slot）
- `cmat list`：列出接触模型属性分配表的信息
- `cmat modify`：修改CMAT中的已存的entry
- `cmat remove`：从CMAT中移除entry

### wall（墙）相关命令

- `wall activeside`：指定哪一面的facet是激活的
- `wall addfacet`：添加一个facet（wall中组成成分）
- `wall conveyor`：为facet指定一个旋转传送速度
- `wall create`：根据顶点信息创建墙体
- `wall delete`：删除墙体或者facet
- `wall extra`：设置墙或者facet的额外变量信息
- `wall generate`：根据指定形状创建墙体
- `wall group`：指定墙或者facet的组名
- `wall history`：添加一个墙体历史
- 

### contact（接触）相关命令

- `contact activate`：修改接触激活的标志位flag
- `contact extra`：设置接触的额外变量信息
- `contact group`：指定特定接触的组名
- `contact groupbehavior`：接触和片piece都有组指定信息
- `contact history`：添加一个接触历史
- `contact inhibit`：阻碍限制在指定范围的接触
- `contact list`：列出接触信息
- `contact method`：调用一个接触method
- `contact model`：指定接触模型
- `contact persist`：设置接触持续性标志位，是否持续接触
- `contact property`：指定接触特性property
- `list contact`：列出接触信息

### dfn（离散裂隙网络）相关命令

- `dfn addfracture`：添加一个确定的裂隙fracture
- `dfn aperture`：为裂隙指定孔径aperture。aperture可以用来在接触模型指定的时候以及DFN范围内元素之间，用于连通性距离计算
- `dfn attribute`：设置裂隙的属性值
- `dfn autoupadate`：设置交集中裂隙自动更新
- `dfn cluster`：计算集群并且指定相应的组
- `dfn conbine`：DFN中的简化方法，住主要是将满足一定要求的裂隙整合分类到一起去进行计算处理
- `dfn connectivity`：计算和指定裂隙之间的网络连通性
- `dfn copy`：赋值裂隙到另外一个裂隙网络当中
- `dfn delete`：删除DFN或者裂隙
- `dfn export`：导出裂隙到文件中
- `dfn extra`：设置dfn额外的变量信息
- `dfn generate`：从数据描述中创建生成裂隙
- `dfn gimport`：从指定的几何geometry集合中导入裂隙
- `dfn group`：指定裂隙的组名
- `dfn import`：从文件中导入裂隙
- `dfn information`：获取DGN裂隙信息
- `dfn initialize`：设置DFN或者裂隙的属性
- `dfn intersection`：划分或者删除指定的交集
- `dfn list`：列出DFN或者fracture属性
- `dfn model`：通过裂隙指定接触模型
- `dfn property`：指定裂隙的面特性
- `dfn prune`：修剪去除孤单的裂隙，单独的裂隙
- `dfn template`：DFN模板实体
- `dfn trace`：创建和scanline映射相关的交集

### geometry（几何体）相关命令

- `geometry copy`：复制几何数据
- `geometry delete`：删除几何对象
- `geometry edge`：创建一个edge边缘
- `geometry explode`：把edge连接的多边形explode分割成集合
- `geometry export`：导出几何数据
- `geometry generate`：根据指定形状创建几何对象
- `geometry group`：指定几何体组名
- `geometry import`：导入几何数据
- `geometry list`：列出几何体信息
- `geometry node`：在指定位置创建一个结点
- `geometry polygon`：创建一个多边形
- `geometry totate`：选装结点
- `geometry set`：指定当前几何体集
- `geometry tessellate`：镶嵌结点
- `geometry translate`：平移结点



