---
title: PFC命令流基本编制顺序
author: lijiabao
date: 2021-04-14 21:30
categories: ["PFC笔记", "数值模拟"]
tags: ["PFC笔记", "数值模拟"]
---

介绍PFC命令流编制的时候的一个大致的顺序和框架

## PFC命令流编制顺序

对于PFC命令流编制，需要按照一定的顺序，分别实现不同的功能才能进行最终的计算：接触的定义必须在球和墙生成之后，domain的定义必须在球和墙生成之前

**<font color='red'>下面介绍一个简单的命令流编制的实例：</font>**

1. <font color='red'>释放当前内存</font>，开始新的任务分析：`new`这一步是进行计算必须的步骤
2. 设置日志文件，可设置可不设置。日志主要是为了代码出错时更好的找出问题
   `set logfile filename.log`设置日志文件保存位置和文件名
   `set log on append开启日志记录，两种：append（在已有文件中续写），truncate（覆盖原有文件内容）
3. 设置模型名称，用于图像显示所用，实际计算并无影响
   `title ‘a example'`
4. <font color='red'>设置计算区域</font>（必要条件，创建ball和wall等实体前的步骤）
   `domain extent -100 100 condition periodic`
5. 设置随机种子（不指定，种子随机，每次生成的模型可能都不一样，试样不重复）
   `set random 10005`用来保证计算过程中随机数相同，保证结果重复
6. <font color='red'>生成模型的边界wall</font>（必要条件）也可以使用一组ball来作为边界
   `wall generate box -50.0 50.0`生成一个矩形wall
7. <font color='red'>创建颗粒体系</font>（ball、clump、cluster等）并分组用来对后面的属性赋值
   `ball generate radius 1.2 1.5 box -5.0 5.0 number 500`
   `ball group small_balls range radius 1.2 1.35`
   `ball group big_balls range radius 1.35 1.4`
8. <font color='red'>设定球的实体属性</font>（必要条件）如：密度、速度和阻尼等
   `ball attribute density 100.0` 设置密度
   `ball fix zvelocity range group big_balls` 固定某些颗粒流的速度，选择上面一部设置的分组来指定
   `ball attribute radius multiply 1.2`对颗粒的属性进行操作
   `ball attribute damp 0.7` 设置阻尼
9. <font color='red'>指定接触模型</font>（必要条件）可以使用contact方法、cmat方法或者属性继承方法实现
   `cmat default model linear property kn 1.0e8 fri 1.0` cmat方法实现
10. 设置球的表面属性（接触属性）
    `ball property kn 2e8 ks 1e8 fric 1.2` 属性继承
11. 添加外力（重力场和外界施加的作用力等）
    `set gravity 10.0`  设置重力
12. 设定时间步长（若不指定，取默认值）
    `set timestep auto`设置自动步长
    `set timestep 5e-3`设置指定步长
13. 记录数据（针对ball、wall、clump、measure和contact等对象）
    `wall history id 1 zcontactforce id 1`查看id为1的wall的z方向的接触应力
14. <font color='red'>计算求解</font>（必要条件）
    `; step 1000`
    `; cycle 2002`
    ` solve time 10.0 ;  solve fishhalt @stop_me`
15. 输出数据，分析
    `history write 1 file wzcforce000` 输出id为1的wall的z接触应力数据到文件wzcforce000中，默认csv后缀
16. 保存模型以及模型调用
    `save example1`保存模型为example1
    `ball attribute displacement multiply 0.0` 根据情况将唯一速度和接触力进行归零设置
    在上述的步骤完成之后，就可以改变荷载参数、物理力学变化、外部环境和力学参数变化下的系统的力学响应
    `set log off` 打开了日志文件的话需要关闭日志文件
    `return`返回操作界面

上述的步骤就是基本的PFC计算模拟所需要的一些步骤，只是不同情况、不同的模型复杂度下，模型的建立、接触定义和计算求解复杂度也不一样，但是按照上述的步骤进行编制，就可以进行简单模型的创建和计算