---
title: PFC模型——球创建的命令
author: lijiabao
date: 2021-04-17 16:00
categories: ["PFC-球单元","PFC笔记", "数值模拟"]
tags: ["PFC-球单元","PFC笔记", "数值模拟"]

---

介绍PFC中球的创建命令以及一些属性设置命令

## PFC创建模型之创建球体命令

### 创建ball的命令

主要有三种创建球的命令：

#### ball create命令

`ball create keyword ...`使用指定的属性来创建单个球（ball）

> <font color='red'>注意：</font>
>
> - 创建ball和wall之前，需要指定domain计算区域
> - 

##### **该命令的keyword：**

1. `group s <slot i > ...`：将新建的ball设置为s组，slot设置为i，如果slot关键字没有指定，就会指定slot为1
2. `id i`：球的id，如果没有指定，会使用下一个可用的ID，当指定的id已经存在时，会报错
3. `position v`：球的位置，默认为原点
4. `radius f`：球的半径，如果没有指定，<font color='red'>会以domain范围最小长度的1/40的大小为半径创建，大于或等于最小domain范围的1/4大小不允许</font>
5. `x f`：x方向的位置坐标，默认为0
6. `y f`：y方向的位置坐标，默认为0
7. `z f`：z方向的位置坐标，3D才有，默认为0

```
; 使用示例：
domain create -10.0 10.0
ball create x -5.0
ball create x 5.0 radius 4.0
ball create id 10 x 0.0 radius 1.0 group middle
```

#### ball generate命令

`ball generate keyword ... <range>`生成不重叠球

generate命令生成不重叠的球体。当指定数量的球体生成好或者多次尝试没有重叠的放置球体完成时，生成过程就会停止。

默认球的位置和半径会遍布整个模型区域来创建，因此生成的球的集合会受到随机数生成器（`set random`命令)的影响。球的半径还可以使用gauss关键字来生成高斯分布的球体半径

**常用的generate生成命令示例：**

```
; generate 示例
domain extent -10.0 10.0
ball generate box -5.0 5.0 -5.0 5.0 radius 1.0 1.2 number 30 group miedum
ball generate box 5.0 10.0 -5.0 5.0 radius 1.3 1.4 number 10 group big
```

##### **该命令的keyword关键字：**

1. `box fxmin fxmax <fymin fymax <fzmin fzmax>>`：球体将会被限制在这个box内部，不会和box面和边缘重叠。默认box范围就是domain范围，如果指定xyz这些参数，就会根据指定的范围来生成box区域，如果省略了yz范围，会直接使用最大指定范围
2. `cubic`：指定球位置从而生成cubic packing并完整的充填指定区域。就是生成立方体排列的球体分布，完整充填整个区域。
3. `hexagonal`：指定球位置从而生成（2D下六边形，3D下位face-centred-cubic）的packing并完整充填这整个区域。2D平面看是六面体形状，3D下是面位中心的立方体。
4. `radius fradllow <fradhi>`：指定生成的球体的半径范围，如果没有指定fradhi，fradhi和fradlow相等，默认二者为1.0.不能使用fishdistribution关键字指定
5. `number i`：指定生成的球的数量，这个并不会影响cubic和hexagonal的状态。
6. `fishdistribution s a1...an`：球的半径通过FISH函数s代表的分布情况来指定。fish函数的参数可以指定为不需要参数。这个函数参数的值必须返回的是浮点型数值，才能作为球的半径
7. `gauss <f>`：通过（平均值为`(fradlow+fradhi)/2`和标准差`(fradhi-fradlow)/2`）的高斯分布指定球半径。可选的参数f（默认0.1）
8. `id idlow <idhi>`：第一个生成的球的ID设置为idlow。如果指定了idhi，是用来计算生成的球的数量的，球的ID是按照顺序来选择的。
9. `group s <slot i > ...`：将新建的ball设置为s组，slot设置为i，如果slot关键字没有指定，就会指定slot为1
10. `tries i`：往指定区域填充指定数量的球体的尝试次数，默认的i是20000次。

**示例：**

```
; generate命令示例
; 生成一个球体半径为5，指定中心点为-10.0 0.0 0.0
ball generate id 1 200      ...
              radius 0.5 0.6 ...
              box -15.0 -5.0 -5.0 5.0  ...
              range sphere center -10.0 0.0 0.0 radius 5.0

;生成一个半径为5的球体，并采用高斯分布来分布球
ball generate id 201 400      ...
              radius 0.5 0.6 ...
              gauss          ...
              box  5.0 15.0 -5.0 5.0 ...
              range sphere center 10.0 0.0 0.0 radius 5.0
```

![](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/generate%E5%88%9B%E5%BB%BA%E7%90%83%E4%BD%93%E5%88%86%E5%B8%83.png)

**使用fishdistribution来指定自己设定的分布法则分布球体：**

```
set random 10001
domain extent -25.0 25.0

; 定义自己的分布函数来计算球体分布位置和半径
define mydist(rmin,rmax)
  p = math.random.uniform
  a = 12.0/(rmax-rmin)^3
  b = 0.5*(rmin+rmax)
  temp = (3.0*p/a) - (b-rmin)^3
  if temp >= 0.0 then
    mydist = b + temp^(1.0/3.0)
  else
    mydist = b - (-1.0*temp)^(1.0/3.0)
  endif  
end

ball generate id 1 10000      ...
              fishdistribution @mydist 0.5 1.0
              
measure create id 1                 ...
               x  0.0 y 0.0 z 0.0  ...
               radius 25.0           ...
               bin 100 1.0 2.0
                
measure dump id 1 table size_dist

```

![](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/generate%E5%88%9B%E5%BB%BA%E5%AE%9A%E5%88%B6%E5%B0%BA%E5%AF%B8%E7%9A%84%E5%88%86%E5%B8%83.png)

#### ball distribute命令

`ball distribute keyword ... <range>`

有重叠的分布球体位置并生成球体。这个生成过程当指定的孔隙度（porosity）达到时停止。默认情况下，球体位置和半径在模型区域内部均匀分布。球体生成结果也会受到随机数生成器（`set random`）的影响。可以通过gauss关键字来指定高斯分布生成球体半径。可以指定带有半径范围（range）和体积片段的球体分布。和generate命令（带有半径的球体从单个分布选取）不一样，这个命令可以生成重要的球体重叠这一类操作。

##### **该命令的keyword关键字：**

1. `bin i keyword ...`：指定bin为i的分布特性，可以指定任何数量的半径范围、体积片段和分布类型。但是体积片段的总和需要等于1。下面是这个bin关键字下的关键字：
   1. `radius fradllow <fradhi>`：指定生成的球体的半径范围，如果没有指定fradhi，fradhi和fradlow相等，默认二者为1.0.不能使用fishdistribution关键字指定
   2. `fishdistribution s a1...an`：球的半径通过FISH函数s代表的分布情况来指定。fish函数的参数可以指定为不需要参数。这个函数参数的值必须返回的是浮点型数值，才能作为球的半径
   3. `gauss <f>`：通过（平均值为`(fradlow+fradhi)/2`和标准差`(fradhi-fradlow)/2`）的高斯分布指定球半径。可选的参数f（默认0.1）
   4. `group s <slot i > ...`：将新建的ball设置为s组，slot设置为i，如果slot关键字没有指定，就会指定slot为1
   5. `volumefraction f`：在这个分布下的球的体积片段，体积片段总和必须等于1
2. `box fxmin fxmax <fymin fymax <fzmin fzmax>>`：球体将会被限制在这个box内部，不会和box面和边缘重叠。默认box范围就是domain范围，如果指定xyz这些参数，就会根据指定的范围来生成box区域，如果省略了yz范围，会直接使用最大指定范围
3. `numbin i`：用来生成这个集合体所用的分布数量
4. `porosity f`：最终的孔隙度设置为f，默认2D是0.16，3D是0.359.
5. `resolution f`：可选的精度，用来作为每个球生成半径的相乘的系数。默认f是1.0

**简单示例：**

```
; 生成半径范围1.0 1.6，孔隙度为0.30的分布
ball distribute radius 1.0 1.6 porosity 0.30
```

**实际用例：**

```
; 生成一个bimodal 颗粒尺寸大小的分布
; 下面的这行代码生成了一个含有25%小球（实际半径范围在0.2到0.32）和75%大球（实际半径范围0.4到0.6）的集合体

; 1. 清空之前的内容和操作
new 
; 2. 定义计算区域的范围
domain extent -10 10
; 3. 设置接触模型分配表
cmat default model linear method deformability emod 1e6 kratio 1.25 property fric 0.25

; 4. 生成一个为球体的几何体
geometry set sphere
geometry generate sphere radius 5.0
; 5. 使用生成的几何体来生成墙wall
wall import geometry sphere

; 6. 使用distribute命令来生成颗粒
; 使用的孔隙度是0.36，并采用精度0.2来生成半径（半径乘以0.2）
ball distribute porosity 0.36                 ...
                resolution 0.2                ...
                numbins 2                     ...
                bin 1                         ...
                  radius 1.0 1.6              ...
                  volumefraction 0.25         ...
                  group small                 ...       
                bin 2                         ...
                  radius 2.0 3.0              ...
                  volumefraction 0.75         ...
                  group large                 ...
                range geometry sphere count 1

; 7. 设置球体的密度和阻尼系数
ball attribute density 1000.0 damp 0.7
cycle 1000 calm 100
ball delete range geometry sphere count 1 not
; 8. 求解问题
set timestep scale
solve arat 1e-3
```

![](https://cdn.jsdelivr.net/gh/li-jiabao/NoteImg@main/img/distribute%E5%91%BD%E4%BB%A4%E7%94%9F%E6%88%90%E5%AE%9E%E4%BD%93%E9%A2%97%E7%B2%92.png)

