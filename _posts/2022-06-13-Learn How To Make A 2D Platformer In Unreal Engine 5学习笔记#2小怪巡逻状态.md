---
title: Learn How To Make A 2D Platformer In Unreal Engine 5学习笔记#2小怪巡逻状态
date: 2022-06-13 23:43:00 +0800
categories: [Unreal5]
tags: [学习笔记, 蓝图]

pin: false
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true


---

## demo视频地址：[# Learn How To Make A 2D Platformer In Unreal Engine 5学习记录#2小怪巡逻状态](https://www.bilibili.com/video/BV1mF411F7gE/)

## 2.1How we are going to set up are enemy characters

1. 在sprite文件夹中创建Enemies文件夹并将资源的Enemies文件夹内容拖入

2. 在Enemies文件夹右键（在Create advanced asset下）blueprints -> enumeration，命名ListOfEnemies，双击打开，点击new（我这里是add enumerator）新建三个enumerators，命名分别为：MushroomEnemy，GoblinEnemy，FlyingEnemy，（我已经受不了一会大写一会小写了，教程中的老师MushroomEnemy的e小写了，蚌埠住了），点击save保存
   
   [![X5X7Sf.jpg](https://s1.ax1x.com/2022/06/14/X5X7Sf.jpg)](https://imgtu.com/i/X5X7Sf)

## 2.2Setting Up Structures

1. 在Enemies文件夹右键（在Create advanced asset下）blueprints -> structure，命名EnemyStructure，双击进入

2. 创建一个New Variable，变量类型为之前创建的ListOfEnemies，命名为EnemyType

3. 修改default values至GoblinEnemy（其实选啥都行应该）

4. 创建7个New Variable，类型：paperFlipbook -> objects reference,命名为：Idle、Moving、Attacking、Ranged Attack、Damaged、Die、Dead Body
   
   [![X5XHl8.jpg](https://s1.ax1x.com/2022/06/14/X5XHl8.jpg)](https://imgtu.com/i/X5XHl8)

## 2.3Setting Up The Data Table

1. 在Enemies文件夹右键Miscellaneous->data table,命名EnemyDataTable。

2. EnemyDataTable中点击add，创建一个mushroomEnemy的数据表

3. 前往Enemies文件夹中的mushroomEnemy文件夹 ，创建Attack, Idle, Death, Run, RangeAttack, Projectile(炮弹), Damaged文件夹

4. 将怪的精灵图申请Paper2d后再Extract Sprite

5. Sprite Extract选Grid

6. Cell Width:150(被分成8份),点击Extract即可提取

7. 创建flipbook——MushroomAttack

8. 重复以上工作，制作damaged、idle等flipbook

9. 奖Death_Sprite_3制作成flipbook：MushroomDeadBody

10. 回到EnemyDataTable为蘑菇怪添加动画
    
    [![X5Xb6S.jpg](https://s1.ax1x.com/2022/06/14/X5Xb6S.jpg)](https://imgtu.com/i/X5Xb6S)

## 2.4Modifying The Data Table

1. 在Enemies文件夹创建Enemy_BP文件夹

2. 在里面创建蓝图->paperCharacter，命名Enemy

3. Enemy蓝图的Sprite设置为EnemyIdle

4. Scale放大为3.5，降低capsule的height至64

5. 调整精灵位置

6. 拖入场景中

7. 进入蓝图，创建变量，类型为ListofEnemy，命名enemyType并设置

8. 进入事件图中，删掉Event Tick和Event ActorBeginOverlap节点

9. Event BeginPlay节点指向Get Data Table Row names节点，选择Enemy Data Table

10. Get Data Table Row name节点指向For Each Loop节点(Out Row Names 和 Array连接)

11. For Each Loop节点再指向一个新的Get Data Table Row 节点（Array element和Row name连接），右键out row，split

12. 拖入Get EnemyType节点 指向Equal(Enum)，Get Data Table Row 节点的Out RowEnemy Type 和Equal(Enum)节点相连

13. Equal(Enum)节点作为新建的branch节点的condition

14. Get Data Table Row 节点的Row Found指向Branch节点

15. 为Get Data Table Row 节点的每一个动作out添加变量，并拖入set类型的节点，branch的true连接Idle的节点，然后一次连接其余动作节点

16. 以上节点意思就是通过遍历data table，匹配目标，利用其预设的动画，为paperCharacter的flipbook类型的变量赋值
    
    [![X5XOmQ.jpg](https://s1.ax1x.com/2022/06/14/X5XOmQ.jpg)](https://imgtu.com/i/X5XOmQ)

## 2.5Creating The Enemy Pt 1

1. 进入蓝图，新建Event Tick节点（看到这蚌埠住了，之前为啥要删掉哈哈哈哈）

2. 新建自定义事件节点：Handle Animation

3. 新建自定义事件节点：Handle Movement

4. Event Tick节点指向新的Handle Animation节点，新的Handle Animation节点再指向新的Handle Movement节点（其实就是再update函数里写俩脚本运行）

5. 接下来创建一套节点，判断当有速度时选择设置flip book：
   
   [![X5XqOg.jpg](https://s1.ax1x.com/2022/06/14/X5XqOg.jpg)](https://imgtu.com/i/X5XqOg)

6. 添加一个float变量enemyDirection默认值是1

7. Handle Movement节点指向sequence节点

8. 拖入Get Movement Directon节点，当大于0时，作为branch的条件，sequence节点的then 0指向branch节点

9. branch的true和false指向SetActorRoatation节点，其中false的SetActorRoatation节点的z要设置为180

10. sequence节点的then 1指向add movement input节点，其x设置为1，scale连接get enemy direction节点
    
    [![X5XXwj.jpg](https://s1.ax1x.com/2022/06/14/X5XXwj.jpg)](https://imgtu.com/i/X5XXwj)

## 2.6Creating The Enemy Pt 2

1. 承接上回，add movement input节点指向Line trace by channel节点

2. 新建GetActorLocation节点，指向Line trace by channel节点的start参数

3. 新建Get actor forward vector节点指向multiply，x设置10（但我听他说的是 a hundred fifty，麻了），再加上刚才的location作为Line trace by channel节点的end。x就是150，他说对了，但写错了哈哈

4. Line trace by channel节点的return value作为一个branch的condition

5. true指向do once节点，completed指向character movement的set max walk speed节点

6. set max walk speed节点指向delay节点，将duration生成一个变量waitTime，设置为2

7. delay节点再指向character movement的set max walk speed节点，为walk speed创建一个变量enemyWalkSpeed，设置为400（将设置enemyWalkSpeed的这三个节点拖到设置怪物动画板块）

8. set max walk speed节点后再设置下速度，将速度*-1即可

9. 将set enemy direction节点再指向 do once 节点的reset（双击连接线可以增加控制点调整曲线）
   
   [![X5XjTs.jpg](https://s1.ax1x.com/2022/06/14/X5XjTs.jpg)](https://imgtu.com/i/X5XjTs)

10. 点击运行按钮那一行里后面的三个点按钮，选择simulate可以直接在编辑视图运行

11. 进入蓝图，选择enemieDirection 在details中将instance editable和expose on spawn勾选就可以在编辑模式下修改这个值了

12. 为了让怪物判断悬崖，添加以下节点来检测前方是否有地板

13. 在add movement input节点后添加sequence节点，0还是指向的检查墙壁板块的节点，1指向检查地板，节点和检查墙壁类似，粘贴过来复用

14. 将local location节点的z坐标-100用于检测地面是否有碰撞

15. 如过没有碰撞就掉头了，这的branch就可以连接上面写的掉头do once了
    
    [![X5Xxkn.jpg](https://s1.ax1x.com/2022/06/14/X5Xxkn.jpg)](https://imgtu.com/i/X5Xxkn)

16. 效果：
    
    [![X5XzYq.jpg](https://s1.ax1x.com/2022/06/14/X5XzYq.jpg)](https://imgtu.com/i/X5XzYq)
