---
title: Learn How To Make A 2D Platformer In Unreal Engine 5学习笔记#3Hud和投掷斧子
date: 2022-06-16 22:00:00 +0800
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

## demo演示视频：[Learn How To Make A 2D Platformer In Unreal Engine 5学习记录#3Hud和攻击](https://www.bilibili.com/video/BV1d3411g7iR/?vd_source=f4a853b19ac511f1de91664a40bf16e9)

## 3.1Creating a HUD For The Players Health

1. 添加hearts图片资源

2. 打开2dCharacter文件夹，右键->user interface->widget blueprint,命名Heart_WB,进入

3. 先搜索panel，先拖进去一个canvas panel，教程里自带panel，但我这个版本没有，所以得自己加上

4. 在palette搜索image，拖入，并在detail中的brush的image改为heart，调整size和position，命名FullHeart

5. 复制FullHeart，命名NoHeart，修改image
   
   [![XTx8gI.jpg](https://s1.ax1x.com/2022/06/15/XTx8gI.jpg)](https://imgtu.com/i/XTx8gI)

6. 前往角色的蓝图，创建Health变量（float），设置为5

7. 回到编辑器，创建PlayerHud_WB，进入

8. 拖进一个Wrap Box，将锚点（anchors）设置为top middle

9. 在details搜索wra，找到explicit wrap size，勾选，并设置size为1500，进入graph，右上角

10. 只留下 event construct节点，其余删掉

11. event construct节点指向cast to 2dCharacter节点，其中的object指向Get Player Character节点

12. 右键cast to 2dCharacter节点的As 2d character，生成变量2DCharacter

13. 进入角色蓝图中的construction script（就是构造函数）蓝图中，创建变量HeartContainers（int），拖入set HeartContainers

14. 拖入get health，并连接set HeartContainers节点

15. construction script节点执行set HeartContainers节点

16. 前往PlayerHud_WB的蓝图，set 2DCharacter节点（刚才生成变量时自动生成的）指向for loop节点，first设置为1

17. set 2DCharacter节点指向get heart container节点，get heart container节点再指向for loop的last index

18. for loop指向create widget节点，并选择Heart_WB，

19. 进入Designer mode（右上角）

20. 选中之前创建的wrap box，再details中，勾选 is variable

21. 回到Graph mode，可以看到左侧变量中有了wrap box_0，拖入get wrap box_0节点

22. get wrap box_0节点指向add child to wrap Box

23. create widget节点的return value 指向add child to wrap Box的content

24. 创建变量HeartRef（Heart WB（Array））

25. 拖入get HeartRef，指向add节点

26. create widget节点的return value 指向add节点

27. create widget节点执行add节点，add节点再指向add child to wrap Box节点
    
    [![X7S1Xt.jpg](https://s1.ax1x.com/2022/06/15/X7S1Xt.jpg)](https://imgtu.com/i/X7S1Xt)

28. 前往角色蓝图，去event beginplay节点板块，set节点指向create widget ，选择playerHud_WB

29. 将create widget节点的return value 生成变量Playerhud，set Playerhud节点指向 add viewport节点
    
    [![X7pI2j.jpg](https://s1.ax1x.com/2022/06/15/X7pI2j.jpg)](https://imgtu.com/i/X7pI2j)
    [![X7p5GQ.jpg](https://s1.ax1x.com/2022/06/15/X7p5GQ.jpg)](https://imgtu.com/i/X7p5GQ)

## 3.2Setting Up What Happens When The Player Is Damaged

1. 前往heart_wb的蓝图，创建自定义事件RemoveHeart

2. 拖入get fullheart，指向set visibility，hidden
   
   [![X7H736.jpg](https://s1.ax1x.com/2022/06/16/X7H736.jpg)](https://imgtu.com/i/X7H736)

3. 进入2d character蓝图，创建event anydamage节点

4. 拖入get health节点，并-1后set heath，event anydamage节点执行set节点

5. 前往player heart蓝图，创建自定义事件updatehealth，拖入get heart ref节点，指向for eachloop的array，updatehealth执行这个loop

6. 拖入get heart ref，计算length（length节点），再减1

7. array index ==  步骤6的输出，作为一个branch的条件

8. loop执行这个branch

9. loop的 array element 指向 remove heart

10. branch的true指向这个removeheart节点

11. 最后再remove掉heart ref里面的这个元素：拖入heart ref节点，指向remove节点的array，removeheart节点执行remove节点，最后for each loop的array element指向remove的item
    
    [![X7HHgK.jpg](https://s1.ax1x.com/2022/06/16/X7HHgK.jpg)](https://imgtu.com/i/X7HHgK)

12. 回到2d character蓝图，在刚才的板块最后拖入PlayerHud节点，并执行updateHealth节点

13. 为了测试，加一个节点，搜索1，按下1时执行set health
    
    [![X7HT9x.jpg](https://s1.ax1x.com/2022/06/16/X7HT9x.jpg)](https://imgtu.com/i/X7HT9x)

## 3.3Creating A Screen Shake Effect

1. 新建蓝图，搜索shake，创建MatineCameraShake，命名damageCameraShake

2. 进入蓝图，设置以下参数：
   
   [![X7OkDO.jpg](https://s1.ax1x.com/2022/06/16/X7OkDO.jpg)](https://imgtu.com/i/X7OkDO)

3. 进入2d character蓝图

4. 在updateHealth节点后执行Play world camera shake节点，shake选择刚创建的shake

5. 创建get actor location节点，连接epiccenter

6. orient shake towards epiccenter勾选

7. outer radius 500
   
   [![X7OK2t.jpg](https://s1.ax1x.com/2022/06/16/X7OK2t.jpg)](https://imgtu.com/i/X7OK2t)

## 3.4Setting Up Melee Attack Pt 1

1. 删掉10、11、12attack精灵图后创建flipbook

2. 添加action mapping，meeeAttack（左键）和projectileAttack（x）

3. 进入角色蓝图

4. 新建MeleeAttack节点，如果不在falling和wallslide就攻击

5. 创建变量isAttackinng（bool）

6. 前往动画处理板块

7. 在检测不在falling状态下，来检查是否在isAttacking状态，不在就执行以前的节点，在的话就设置我i攻击动画（为动画创建变量AttackingAnimation）

8. 回到按键触发攻击板块，获取攻击动画的total duration（运行后查看是0.6）

9. 添加delay节点，0.6s后停止动画，设置isAttacking为false
   
   [![XHmfNd.jpg](https://s1.ax1x.com/2022/06/16/XHmfNd.jpg)](https://imgtu.com/i/XHmfNd)

## 3.5Setting Up Melee Attack Pt 2

1. 在delay节点后创建sequence

2. then1指向线追踪检测节点（lineTrace for objects），start：actor位置，end为朝向的100距离，objects type指向make array，选择pawn，action to ignore指向make array，选择self，out hit指向break hit result节点

3. hit actor指向apply damage节点，base damage设置1

4. 线追踪检测节点执行apply damage节点
   
   [![XHMwqK.jpg](https://s1.ax1x.com/2022/06/16/XHMwqK.jpg)](https://imgtu.com/i/XHMwqK)

## 3.6Setting Up Ranged Attack

1. 进入objects文件夹，拖入axe资源

2. 创建actor蓝图类，命名Axe

3. add paper sprite，设置为刚添加的axe，scale设置为2

4. add projectile movement，initial 和max speed分别设置为200、500

5. 设置gravity = 1

6. 点击simulation可以演示

7. add rotating movement，rotation rate的Y设置为-180

8. 前往2dCharacter蓝图，新建projectileAttack节点，指向spawnActor Axe节点，并split transform

9. 进入viewport视图，Add scene，命名AxeSpawn，放到合适的位置（斧子生成位置）

10. 返回蓝图视图，拖入AxeSpawn，指向get world location节点，并连接spawnActor Axe节点的transform

11. 提譬如 ismovingright，指向select节点，false的话z设置为180

12. 去设置axe，scale5，速度设置为1000

13. 创建变量：NumOfAxe：10

14. 再扔斧子之前检查是否有斧子，有才能扔，并且扔了就要减1
    
    [![XH7uNR.jpg](https://s1.ax1x.com/2022/06/16/XH7uNR.jpg)](https://imgtu.com/i/XH7uNR)

## 3.7Displaying Projectiles On The HUD

1. 前往PlayerHud_WB，添加horizontal box，并在其下添加3个text和1个img（img也可以外挂）

2. 点击一个text，在detail的content下，点击bind，绑定2dcharacter的numofaxe变量，第二个text为“/”，第三个为MaxAxe，img为斧子精灵图

3. 修改text的padding，right统统设置10，size可以设置为fill
   
   [![XHLQde.jpg](https://s1.ax1x.com/2022/06/16/XHLQde.jpg)](https://imgtu.com/i/XHLQde)

## 3.8Adding Projectile Pick Up

1. 创建actor，命名AxePick

2. add paper sprite，设置为axe，调整scale5，在details中的collision中的collision presets设置为nocollision

3. add box collision，调整box extent

4. 找到details的events为其添加on component begin overlap（重叠）事件

5. other actor指向cast 2d character节点，接下来设置增加斧子数量，需要注意的是：
   
   * 创建Noofaxe变量作为斧子能够增加的数量
   
   * 使用clamp节点控制斧子数量不能超过maxAxe

6. set完后消失，指向destrory节点
   
   [![XHX7qK.jpg](https://s1.ax1x.com/2022/06/16/XHX7qK.jpg)](https://imgtu.com/i/XHX7qK)

效果：

[![XbSSfg.jpg](https://s1.ax1x.com/2022/06/16/XbSSfg.jpg)](https://imgtu.com/i/XbSSfg)
