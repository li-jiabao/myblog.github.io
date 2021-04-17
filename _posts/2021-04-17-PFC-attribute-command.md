---
title: PFC模型——球创建的命令
author: lijiabao
date: 2021-04-17 22:00
categories: ["PFC-球单元","PFC笔记", "数值模拟"]
tags: ["PFC-球单元","PFC笔记", "数值模拟"]
---

介绍PFC中球的属性设置命令

## PFC的属性（attribute）和特性（property）

### PFC-球单元的ball attribute命令

`ball attribute keyword ... <range>`

设置ball的属性值。这个命令和`ball initialize`命令类似。使用`ball list`命令可以查看不同所有球的属性值

> <font color='red'>注意：</font>
>
> - 球的几何尺寸发生改变时，如果不能完全落在domain区域内，会失败并报错
> - 球的attribute是球的位置和大小之类的特征，这和球的property不一样（`ball property`命令），球的property是球的面属性，用来填充接触模型的property的

#### ball attribute关键字

1. `appliedforce v`：应用到球体的应力
2. `appliedmoment fx fy fz`：应用到球体的位移，分别是x，y，z方向的移动
3. `contactforce v`：在前面一个应力-位移更新后的球的接触应力的叠加，这个值在下一个应力-位移更新之后会被修改
4. `contactmoment v`：在前面一个应力-位移更新后的球的接触移动距离的叠加，这个值在下一个应力-位移更新之后会被修改
5. `damp f`：球的局部阻尼系数，f大于等于0，默认为0
6. `density f`：球的密度，大于等于0，默认球创建之后的密度设置为0。为了准确计算块体特性并整合球的运动方程，需要设置非零的密度。当cycling开始的时候有零密度的球存在会抛出异常
7. `displacement v`：作为cycling结果的累计球的位移向量
8. `euler v`（只在3D里面有）：当前Euler角的方向。方向只会在开启了方向追踪的时候才会被更新（`set orientation`命令）
9. `fragment i`：fragment（通过力学接触连接在一起的一系列实体）的ID
10. `position v`：球质心的位置，这个命令之后的模型应该使用清除命令（`clean`），为了确保和周边片（piece）的潜在接触被更新。
11. `radius f`：球半径，f>0。创建球的时候半径必须是正值。但是随后球的半径可以使用这个命令参数来进行修改，这个命令之后的模型应该使用清除命令（`clean`），为了确保和周边片（piece）的潜在接触被更新。
12. `rotation f`（只是2D的）：当前球的方向，方向只会在开启了方向追踪的时候才会被更新（`set orientation`命令）
13. `spin fx fy fz`：球的角速度
14. `velocity v`：球的平动速度向量
15. 在上述的所有命令前面加一个x、y或者z，分别表示这三个方向的内容，如`xappliedforce、xappliedmoment`等等

#### 命令示例

1. 示例1：设置球体密度

```
; 球体密度设置示例

; 1. 创建计算区域
domain extent -10 10

; 2. 选择默认的接触模型和他的特性
cmat default model linear property kn 1e6

; 3. 生成随机种子
set random 10001
; 4. 墙和球的生成
wall generate box -5 5 onewall
ball generate id 1 500 box -4.5 4.5 radius 0.5

; 5. 为所有的球体设置密度和局部阻尼系数
ball attribute density 1000.0 damp 0.7

; 6. 设置重力并求解
set gravity 10.0
solve

```

2. 设置初始条件：

```
; 设置初始条件

; 1. 设置计算区域
domain extent -10 10 condition destroy
; 2. 选择默认的接触模型和它的一些特性
cmat default model linear property kn 1e6 dp_nratio 0.2
; 3. 设置随机种子并在box中生成随机球
set random 10001
wall generate box -5 5 onewall
ball generate id 1 100 box -4.5 4.5 0.0 0.0 radius 0.5 cubic
; 4. 设置球的密度和阻尼系数
ball attribute density 1000.0 damp 0.7
; 5. 为指定range的球设置速度
ball attribute velocity (0.0, 0.0, -1.0) range x -4.5 0.0
; 对于不在上一步操作所处区域的球体设置施加appliedforce
ball attribute appliedforce (0.0, 0.0, -1.0) range x -4.5 0.0 not
; 6. 执行cycle
cycle 10000
; 为所有球设置速度并且为指定范围固定x方向的速度
ball attribute velocity (0.5，0.0, 0.0)
ball fix x range x -4.5 0.0
```

3. 修改系统几何形状

```
; 修改球的x的位置
ball attribute position add (0.0, 0.0, 0.0) xgradient (0.0, 0.0, 0.5)
; 执行clean更新位置和接触信息
clean
```

### PFC-球单元的ball property命令

`ball property s a<s1 a21...> <range>`

指定球面特性，值a指向名为s的面属性，可以指定任何数量的name-value对。



### property和attribute的区别



**property**：是专门用在piece（片）上的特性，是一系列的用户指定的name-value对，默认情况片是没有属性的。这些name-value对旨在用来表达片的surface condition，可能会被用来决定片之间如何相互作用

**attribute**：是模型组成成分的内在特征：位置、速度、大小等等，这一些列的属性是不可更改的。实体（ball、wall、clump）、piece（balls、pebble、facet）和接触都有attribute。使用`ball list attribute`、`wall list attribute`可以查看他们的属性attribute。



#### 错误的property赋值

如果把attribute的name给到property，会出现错误，比如：

```
ball attribute xvelocity 1.0 range z -10.0 0.0
ball property xvelocity 1.0 range z 0.0 10.0
```

