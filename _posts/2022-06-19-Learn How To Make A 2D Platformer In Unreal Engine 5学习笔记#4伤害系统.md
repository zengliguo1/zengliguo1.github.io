---
title: Learn How To Make A 2D Platformer In Unreal Engine 5学习笔记#4伤害系统
date: 2022-06-19 20:30:00 +0800
categories: [Unreal5, Learn How To Make A 2D Platformer]
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

## demo演示地址：[Learn How To Make A 2D Platformer In Unreal Engine 5学习记录#4伤害系统](https://www.bilibili.com/video/BV1FB4y1q7as)

## 4.1Damaging The Enemy

1. 把enemies文件夹托在content里

2. 进入enemy蓝图

3. 新建event anydamage节点

4. 进入enemy structure，新建变量EnemyHealth（float）

5. 进入enemy datatable，为蘑菇怪的生命值设置处置5

6. 返回蓝图，前往怪物数据表初始化板块（原设置怪物动画），初始化生命值（新建变量 ）

7. 回到event anydamage节点，减少生命值，设置后退效果
   
   [![XOb3tg.jpg](https://s1.ax1x.com/2022/06/18/XOb3tg.jpg)](https://imgtu.com/i/XOb3tg)

8. 前往角色蓝图，在apply damage节点的damage cause设置为self

## 4.2Updating The Enemy Damage Flipbook

1. 在enemy蓝图中，受到伤害后更改动画，创建一个变量作为状态机isDamaged？（bool），如果生命值小于等于0的话就死亡，否则就设置为isDamaged状态，并指向retriggerable delay节点，时间为damaged动画的时间，最后再设置为非isDamaged状态
   
   [![XOLwYF.jpg](https://s1.ax1x.com/2022/06/18/XOLwYF.jpg)](https://imgtu.com/i/XOLwYF)

2. 前往处理动画板块，在开始处，新建branch（快捷键：按住B后左键），判断是否在isDamaged状态，是的话就做受伤动画

3. damage动画太快，可以选择多复制其中的一帧

4. 注意：设置了5滴血，打5次后就不会再出现受伤动画了嗷

## 4.3Giving The Axe Projectile Damage

1. 前往axe蓝图，添加球形碰撞，调整大小

2. 将paperSprite的collision presets设置为nocollision

3. 球形碰撞添加on component begin overlap事件

4. 对enemy造成伤害，并设置如果标签不是axe就造成伤害
   
   [![XXSKnH.jpg](https://s1.ax1x.com/2022/06/18/XXSKnH.jpg)](https://imgtu.com/i/XXSKnH)

5. 去给角色和斧子都设置为axe标签（在角色蓝图中component的details中可以搜索到）

6. 给角色蓝图中的spawnactor axe节点的owner设置为self

## 4.4Creating An Enemy Death System

1. 新建EnemyDead变量，在怪生命值为0时，设置为true

2. 在handleAnimation节点后再创建一个branch，如果没有dead跟以前一样，死了就：

3. 创建变量EnemyDeadBody?并作为select的index，如果有尸体（true），那就是get enemy dead body节点，否则就是enemy die节点，最后指向set flipbook来设置动画
   
   [![XX8B6S.jpg](https://s1.ax1x.com/2022/06/19/XX8B6S.jpg)](https://imgtu.com/i/XX8B6S)

4. 回到damage板块，当设置怪物的isEnemyDead后，die的动画结束后，设置EnemyDeadBody?为true，并在设置死亡前的branch后加上do once节点

5. 在处理动作板块，在sequence节点的then1处添加branch，死亡后不再移动

6. 考虑到死亡时有跳帧，那么有可能是动画结束时间的问题，把时间-0.2再delay会好一点
   
   [![XX846U.jpg](https://s1.ax1x.com/2022/06/19/XX846U.jpg)](https://imgtu.com/i/XX846U)

## 4.5Creating A Jump Attack System

1. 终于要修跳怪头上后就无限下落的问题了，进入角色蓝图

2. 选择component的character movement，在details中搜索pla，勾选constrain to plane，坐标轴设置为Y，意味着角色的y坐标不会改变，同样，enemy蓝图中也做此设置

3. 给enemy蓝图添加box collison，调整大小和位置，放在头部

4. 点击这个，可以更改对齐精度和是否对齐
   
   [![XXJAa9.jpg](https://s1.ax1x.com/2022/06/19/XXJAa9.jpg)](https://imgtu.com/i/XXJAa9)

5. 添加begin overlap事件，判断当角色在跳跃状态时，就触发一个向上的速度

6. 在damage板块添加自定义事件，JumpDamage，指向do once之前的branch

7. 先到 角色蓝图中添加变量jumpdamage，作为跳跃伤害

8. 回到enemy蓝图，在碰撞板块，添加扣血逻辑并最后指向之前创建的JumpDamage节点，最后连接launch character
   
   [![XXtHKK.jpg](https://s1.ax1x.com/2022/06/19/XXtHKK.jpg)](https://imgtu.com/i/XXtHKK)

9. 当enemy死亡后，destroy component ，来销毁box碰撞
   
   [![XXtTv6.jpg](https://s1.ax1x.com/2022/06/19/XXtTv6.jpg)](https://imgtu.com/i/XXtTv6)

10. bug：角色起跳时就发生碰撞，那么将box改小一点即可

## 4.6Making The Enemy Damage The Player Character

1. 给enemy添加box collision，命名attack box，调整位置和大小，添加碰撞事件，begin overlap事件和end overlap事件

2. begin overlap节点投射到2dcharacter后，指向set timer by event节点，event事件指向一个新建的自定义事件，attackPlayer，return value 设置一个新变量为AttackEvent，Time设置为0.1，勾选Looping

3. attack player事件：如果enemy未死亡，do once指向设置isAttacking为true
   
   [![XjMFG8.jpg](https://s1.ax1x.com/2022/06/19/XjMFG8.jpg)](https://imgtu.com/i/XjMFG8)

4. 在处理动画板块，在isdamaged的branch后，再添加一个branch，根据isAttacking状态设置攻击动画

5. 回到攻击板块，在进入攻击状态后，delay节点后执行sequence节点，then0指向set isAttacking为false，then1指向line trace for objects
   
   [![XjMAxg.jpg](https://s1.ax1x.com/2022/06/19/XjMAxg.jpg)](https://imgtu.com/i/XjMAxg)

6. line trace for objects节点的线由怪物自身向前发射，长度100，objects types指向make array节点，选择pawn，actors ignore指向make array，再指向self，勾选ignore self

7. out hit 指向break hit result，当hit actor == get Player character，如果true，apply damage，hit actor指向apply damage

8. 新建变量enemyDamage（float）（1）作为base damage，并勾选变量的instance editable和expose on spawn

9. 前往角色蓝图的event anydamage节点，其中的damage指向减节点

10. 回到enemy蓝图，apply damage节点指向delay节点，新建变量coolDown（float）（3），结束后，指向之前do once的reset
    
    [![XjMkRS.jpg](https://s1.ax1x.com/2022/06/19/XjMkRS.jpg)](https://imgtu.com/i/XjMkRS)

11. end overlap节点要刷新attack event并且指向do once的reset，这是因为当玩家没有被打到，离开碰撞盒时，就要重置do once，可以再次攻击，否则do once没打到玩家的话就一直不重置，就不能再攻击了
    
    [![XjMiPf.jpg](https://s1.ax1x.com/2022/06/19/XjMiPf.jpg)](https://imgtu.com/i/XjMiPf)

12. 前面有点小bug没有播放动画，后来又是只攻击了一次就停下来了，查了下，是有两处节点没有连接起来，所以部分节点没有运行......
    
    [![Xj3bcT.jpg](https://s1.ax1x.com/2022/06/19/Xj3bcT.jpg)](https://imgtu.com/i/Xj3bcT)
    [![Xj3H3V.jpg](https://s1.ax1x.com/2022/06/19/Xj3H3V.jpg)](https://imgtu.com/i/Xj3H3V)

13. 由于怪物打玩家总是只掉一滴血，所以去修这个bug

14. 由于update health每次只减少一滴血，所以要加个循环
    
    [![Xjd9Xt.jpg](https://s1.ax1x.com/2022/06/19/Xjd9Xt.jpg)](https://imgtu.com/i/Xjd9Xt)

15. enemy在攻击时也不能移动，加上这个条件
    
    [![Xjdp6I.jpg](https://s1.ax1x.com/2022/06/19/Xjdp6I.jpg)](https://imgtu.com/i/Xjdp6I)
    
    ## 4.7Creating A Ranged Enemy Pt 1

    由于第5节内容较少，所以和第3节合并

1. 复制enemy 蓝图，命名rangeEnemy

2. 进入rangeEnemy蓝图，删掉处理动作板块，将handle movement改为handle rotation（因为要小怪面向玩家），蓝图如下：（可以从伤害板块找到部分节点）
   [![jSBdER.jpg](https://s1.ax1x.com/2022/06/21/jSBdER.jpg)](https://imgtu.com/i/jSBdER)

3. 修改attack的box

4. 修改 overlap begin事件（删掉end事件和begin doonce之后的节点）

5. 将isAttack改为isRangeAttack，并修改之前的动画

6. 动画播放完毕就生成子弹，蓝图：
   
   [![jSBUb9.jpg](https://s1.ax1x.com/2022/06/21/jSBUb9.jpg)](https://imgtu.com/i/jSBUb9)

7. 将子弹精灵图片提取出来，将第一帧命名为mushroom projectile

8. 将axe的actor复制一份拖到enemy文件夹，命名mushroom projectile

9. 修改精灵图，将Projectilemovement的速度修改为700

10. 将actor has tag节点的tag修改为enemy

11. 为enemy添加enemy  tag

12. 回到mushroom projectile蓝图，add scene，作为发射点

13. 最后一个delay完成后要指向do once的reset。蓝图：
    
    [![jSvC4A.jpg](https://s1.ax1x.com/2022/06/21/jSvC4A.jpg)](https://imgtu.com/i/jSvC4A)

## 4.8Creating A Ranged Enemy Pt 2

1. 由于投掷的斧子触碰到怪物的攻击碰撞盒也会触发，所以要去修改怪物的攻击碰撞盒，collision中的presets修改为custom，ignore world static和worldDynamic

2. Blueprint Runtime Error: "Attempted to access Axe_C_0 via property K2Node_ComponentBoundEvent_OtherActor, but Axe_C_0 is not valid (pending kill or garbage)". Node:  Branch Graph:  EventGraph Function:  Execute Ubergraph Mushroom Projectile Blueprint:  MushroomProjectile
   
   [![jSvNE4.jpg](https://s1.ax1x.com/2022/06/21/jSvNE4.jpg)](https://imgtu.com/i/jSvNE4)

3. 解决这个问题，意思就是飞斧打到怪物的子弹，想要造成伤害，但子弹没有受到伤害节点，所以就报错了

4. 前往axe的蓝图，在begin overlap后加一个is valid节点，other actor指向其 input object

5. 在mushroomProjectile蓝图中做相同的事情

## 4.9How To Add New Enemy Sprites

1. 处理goblin精灵图（创建动作文件夹Attack、Death、Idle、Projectile、run、damaged、rangeattack），提取精灵图、创建flipbookf

2. 前往enemydata table，添加哥布林

3. 可以通过修改enmey蓝图的type直接更改enemy

4. 将蓝图中，初始化精灵图的节点复制到construction script中去，可以在开始游戏前就看到更换的效果
   
   [![jSx2LT.jpg](https://s1.ax1x.com/2022/06/21/jSx2LT.jpg)](https://imgtu.com/i/jSx2LT)
