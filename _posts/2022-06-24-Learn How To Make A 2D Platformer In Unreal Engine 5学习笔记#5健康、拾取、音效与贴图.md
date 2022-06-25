---
title: Learn How To Make A 2D Platformer In Unreal Engine 5学习笔记#5生命、检查点、拾取、音效与贴图
date: 2022-06-24 00:36:00 +0800
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

## demo演示地址：[Learn How To Make A 2D Platformer In Unreal Engine 5学习笔记#5生命、检查点、拾取、音效与贴图](https://www.bilibili.com/video/BV1AU4y197Ys)

## 5.1How to create a moving platform

1. 创建一个actor蓝图，命名moving platform

2. 添加paper sprite，设置为灰色块

3. 调整形状

4. add interpToMovemrnt组件，设置behaviour type 为Ping Pong

5. 添加两个 control points

6. 创建两个变量，Location1（vector），Location2（vector），都设置为可编辑模式，并勾选show 3d widget（这样就可以直接在场景中修改位置了）

7. 创建一个变量Duration（float）

8. 前往construction script，设置控制点和时间，蓝图：
   
    [![j9MdQx.jpg](https://s1.ax1x.com/2022/06/22/j9MdQx.jpg)](https://imgtu.com/i/j9MdQx)

## 5.2How to create a Coin System

1. 拖入coin资源，制作flipbook

2. 创建Coin的actor

3. 添加paperFlipbook，调整大小

4. 添加sphere碰撞盒

5. 在角色蓝图中添加变量NoCoins（integer）

6. 去PlayerHud_WB中添加一个horizontal box，在里面加一个text，一个image

7. text绑定角色的NoCoins

8. 去coin的蓝图中添加begin overlap事件，让角色的NoCoins变量增加

9. 将金币放在移动平面的子物体中，可以跟随平板移动
   
   [![j9Nv8O.jpg](https://s1.ax1x.com/2022/06/22/j9Nv8O.jpg)](https://imgtu.com/i/j9Nv8O)

10. 之前一直没碰撞是因为金币的y坐标不是0，所以触发不了碰撞事件

## 5.3How to create a Checkpoint System

1. 创建变量Respawn Location（vector）

2. 创建自定义事件：RespawnPlayer，指向setactorLocation，设置为Respawn Location

3. 创建actor，命名checkPoint，添加一个box collison（解除hide）和一个scene

4. 为box collison添加begin overlap事件

5. 当碰撞时，设置respawn location为这个acor的scene的world location

6. 到蓝图的更新生命值板块，如果生命值<=0了，就指向RespawnPlayer节点
   [![j90lXF.jpg](https://s1.ax1x.com/2022/06/22/j90lXF.jpg)](https://imgtu.com/i/j90lXF)

7. 创建变量maxHealth为最大生命值

8. 先在beginPlay后设置maxHealth为Health
   [![j903m4.jpg](https://s1.ax1x.com/2022/06/22/j903m4.jpg)](https://imgtu.com/i/j903m4)

9. 在respawn player节点后再重新设置health为maxHealth

10. set health节点指向remove from parent，将playerhud从view port删除，接着指向create widget节点，选择playerHud_WB，再将return value设置为playerhud变量，最后再指向add to viewport节点
    [![j9080J.jpg](https://s1.ax1x.com/2022/06/22/j9080J.jpg)](https://imgtu.com/i/j9080J)

11. 在beginPlay节点后设置respawn location为角色位置（z+10），避免没有碰到检查点就死掉后重生在奇怪位置的bug

12. 拖入checkPoint资源（campFire），给检查点添加动画

13. 测试的时候，给一个按键加个apply damage功能即可

14. 做的时候有个小bug，生命条不显示，是因为忘加add to viewport节点了

## 5.4How to create a Life System

1. 添加变量lives

2. 每当生命值小于0时，就减少一条命（如果lives>0），如果没命了，就将下面创建的isDead置为true

3. 添加变量isDead

4. 更新死亡动画，如果死亡状态就是do once，设置死亡动画smoke（下面创建），动画播放结束就set sprite visibility不可见，角色的movement也set active（不勾选）
   [![j9I6sJ.jpg](https://s1.ax1x.com/2022/06/22/j9I6sJ.jpg)](https://imgtu.com/i/j9I6sJ)

5. 拖入deathSmoke资源，创建flipbook

6. 在playerhud_wb中添加vertical box，将之前的信息都放到里面，再添加一个horizontal box记录命的条数

7. 此时，出现了一个问题，当我往text绑定角色类的lives时，发现在下拉列表中找不到lives这个float变量，然后发现有的float变量能找到，有的float变量找不到，然后上虚幻论坛查了查受到点启发，去create bind，然后通过蓝图来绑定，就可以，其中自动把float给转换成double了，很怪。
   
   [![j9IcL9.jpg](https://s1.ax1x.com/2022/06/22/j9IcL9.jpg)](https://imgtu.com/i/j9IcL9)

## 5.5How to create a Spike System

1. 拖入spike资源（地刺）

2. 创建actor，命名spikes，添加paper sprite，box collision，去掉paper sprite的collision

3. 添加begin overlap事件，对玩家造成伤害，蓝图：
   
   [![jP1GFO.jpg](https://s1.ax1x.com/2022/06/23/jP1GFO.jpg)](https://imgtu.com/i/jP1GFO)

## 5.6Setting Up A Background For Are Level

1. 拖入tileset文件夹，其中BG1要单独拖入，不然创建sprite就会变成绿色（因为拖入tileset文件夹就会按normal map导入），全部创建精灵（创之前要记得apply 2dtexture）

2. 拖入BG1，设置坐标，y=-100，修改大小（测试而已）

3. 右键BG1、BG2、BG3（创建精灵前的），sprites action -> create tile set

4. 右键创建好的tile set创建tile map

5. 进入tile map，设置setup中的宽高10*6

6. 重命名这个图层为skybackground

7. 在上面创建一个图层，放置云，clouds

8. 再在上面创建一个图层，grassLayer，save后即可使用tilemap
   
   [![jP8oz4.jpg](https://s1.ax1x.com/2022/06/23/jP8oz4.jpg)](https://imgtu.com/i/jP8oz4)

## 5.7Tilesets

1. 将资源中的tile set设置成tile set（双击进去，将tile size改为16*16）

2. 点击上方的add box可以给tile set添加碰撞盒，add polygon可以给贴图添加奇形怪状的碰撞盒，add circle可以加原型碰撞盒

3. 点击colliding tiles可以显示加了碰撞的贴图
   
   [![jPt3TI.jpg](https://s1.ax1x.com/2022/06/23/jPt3TI.jpg)](https://imgtu.com/i/jPt3TI)

4. 创建个tilemap，20*20大小的，做个小地形放进去

5. 中间不知道为啥突然crash了，去论坛查了查，有人重新放了下素材就好了，我试了试，也可以了，虽然不知道为啥crash.....

## 5.8Setting Up Custom Projectiles

1. 进入rangeEnemy蓝图的攻击板块，右键SpawnActor的class节点，生成一个变量EnemyProjectile，设置为可以编辑的，后面是通过在编辑器的修改来改变弹药种类（其实可以加个branch）

2. 复制MushroomProjectile的actor，命名Bomb Projectile

3. 更换精灵图，修改伤害值

## 5.9Enemy Fixes

1. 当小怪成为尸体后，创建一个金币，spawnActor coin

2. 复制coin创建一个enemycoin，创建一个变量random coin Num，使得碰撞后+random coin Num个金币（记得勾选instance editable和expose on spawn，这样才能在节点显示）

3. 回到小怪蓝图的spawnActor coin节点，class设置为刚建的enemy coin

4. random coin Num指向random float in range节点，指向random float in range节点需要再取整，加个round节点（如果是int就不用了）

5. 将transform拆分，location为尸体的位置

6. 将capsule component拖入蓝图，指向set collision profile name节点，并左右连接set dead body和spawnactor节点

7. set collision profile name节点的name改为overlap only pawn
   
   [![jPBJbD.jpg](https://s1.ax1x.com/2022/06/23/jPBJbD.jpg)](https://imgtu.com/i/jPBJbD)

8. 在处理动作板块加个判断死亡，就不再执行动作

9. 将产生金币复制到melee enemy蓝图中

10. 在小怪攻击动画结束后如果死掉，就不再对玩家造成伤害，加个branch

11. 远程攻击的话，是spawn actor之前判断下是否死亡

## 5.10Creating A Health Pick Up

1. 给心脏贴图创建个精灵（之前竟然一直没建

2. 创建一个actor命名heartPickUp，add paper sprite（去掉碰撞）和box collision

3. 添加begin overlap事件，类似捡手斧，蓝图：
   
   [![jP6Kcn.jpg](https://s1.ax1x.com/2022/06/23/jP6Kcn.jpg)](https://imgtu.com/i/jP6Kcn)

4. 前往playerHud_WB，复制Heart Ref变量为RemovedHearts，拖入蓝图，指向add节点，for loop的array element指向add的select asset，最后左右分别连接remove heart节点和remove节点

5. 前往heart_WB，新建自定义事件节点showheart，跟remove heart反着来就行，显示心脏

6. 前往playerHud_WB，创建自定义事件showHeart，指向for loop，for loop的first index设置为1，last index连向showHeart节点

7. 前往heartPickUp，在set后指向showHeart节点（需要先有playerHud_wb的引用噢，就是获取2dCharacter的playerHud变量）
   
   [![jP6MXq.jpg](https://s1.ax1x.com/2022/06/23/jP6MXq.jpg)](https://imgtu.com/i/jP6MXq)

8. last index为这个heartPickUp的health，最后再destroy

9. 回到playerHud_WB，拖入removedHearts，指向get（a copy）节点，其中get的index指向为removedHearts的长度-1，这样copy的就是失去的最后一个心（也就是刚失去的）

10. get后指向 show up（也就是让一颗心visible），然后再add heart ref，remove removedhearts：
    
    [![jP6u1s.jpg](https://s1.ax1x.com/2022/06/23/jP6u1s.jpg)](https://imgtu.com/i/jP6u1s)

11. 由于满血时再吃心，会报错，前往playerHud_WB，在get节点后加个判断is valid节点，因为，removed hearts有可能长度为0，再-1成负数了，显然不可能

## 5.11Adding Sound Effects

1. 拖入音效资源

2. 前往Axe_pickup蓝图，在destory前，加入Play sound2d节点

3. 前往角色蓝图，生成斧子节点后播放投手斧的声音

4. 前往coin_sound文件夹，右键->sounds->sound cue，命名coinsoundcue

5. 将所有coinsound拖入cue的蓝图中，所有节点指向random
   
   [![jPRHZq.jpg](https://s1.ax1x.com/2022/06/24/jPRHZq.jpg)](https://imgtu.com/i/jPRHZq)

6. 前往coin的蓝图，在destroy前，加入play sound 2d节点，选择coinsoundcue

7. enemycoin的actor也加入play sound 2d节点，选择coinsoundcue

8. 前往heart pick up蓝图，加入play sound 2d节点选择heartpickup

9. 为近战攻击也创建个soundcue

10. 前往角色蓝图，在攻击后的检测中，如果hit actor is valid，就播放近战攻击声音，在delay前播放swoosh声音

11. 在set is attacking节点前加一个do once

12. sequence节点加一个pin，then2指向do once的reset

13. 前往enemy的蓝图，在设置小鬼受伤后面（set is damaged节点后）播放hurt动画

## 5.12Creating Double Jump

1. 直接去角色蓝图中，点击component中的2dcharacter（self），在details中搜索jump，修改jump max count即可（so easy）
