---
title: Learn How To Make A 2D Platformer In Unreal Engine 5学习笔记#1角色动作和滑墙系统
date: 2022-06-11 23:30:00 +0800
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

## demo视频地址：[Learn How To Make A 2D Platformer In Unreal Engine 5学习记录#1](https://www.bilibili.com/video/BV1ZS4y1i74r?spm_id_from=333.999.0.0&vd_source=f4a853b19ac511f1de91664a40bf16e9)

## 1.1Introduction

1. 创建blank项目，platformer 2dproject（蓝图模式，默认选项，取消掉初学者内容包）

2. 更改自己喜欢的layout（这里就跟教程选择一样的ue4layout了），点击上方window->load layout

## 1.2Setting up input player input

1. 创建文件夹2DPlatfomerBlueprints

2. 创建蓝图，搜索paperCharacter，创建并命名2dCharacter

3. 设置输入：点击上方Edit->project settings->Engine->input

4. 添加Action Mapping：Jump：Space Bar和Up

5. 添加Axis Mapping：Moveright：Right（Scale 1）和Left（Scale -1）

6. 在初始目录新建文件夹Sprites，再里面新建PlayerCharacterSprites

7. 将HeroSprites中资源拖入文件夹

8. 精灵模糊解决：选中精灵图片->右键->Sprite Actions->Apply Paper2D Texture Settings

9. 为图片创建sprite：选中精灵图片->右键->Sprite Actions->create sprites

10. 为精灵创建flipbook，选中精灵序列，右键->create flipbook

11. 双击点开flipbook可以在右侧更改播放速度

12. 以同样的方式创建run、jump、full的flipbook

## 1.3Setting up the player character pt1

1. 双击之前创建的蓝图

2. 选择sprite并使用idle作为源sprite

3. 锁住scale并调整参数至5，调整精灵位置

4. 点击上方event graph

5. 右键创建节点，搜索jump，添加之前创建的action event：Jump

6. 点击Pressed连接Jump，点击released连接Stop Jumping

7. 右键创建节点，添加之前创建的Moveright，点击箭头（exec执行）创建新节点Add movement input，并将Moveright的Axis Value连接至Scale Value，将Add movement input中world Direction设置**x=1**
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111456290.jpg)

8. 在左侧components下点击add添加springArm并点击视图中上侧的旋转按钮，将方向旋转-90°

9. 在springArm添加camera并将投影模式设置为Orthographic（正交投影），Ortho Width设置为1000

10. 在sprites文件夹中创建新文件夹Objects，在里面创建GraySquare文件夹，在里面拖进资源graySquare，并用该图片创建sprite

11. 将之前创建的2dCharacter拖到场景中，在details处搜索pawn，将auto possess pawn设置为player0（游戏开始相机便会锁定角色）

12. 删掉地形（不能直接删父landscape，删不掉，得删完其下的所有，才能删），并将graySquare拖入

13. 更改人物和灰色地块的y为0，并将灰色地块拉长
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111432085.jpg)

## 1.4Setting up the player character pt2

1. 进入2dCharacter中的ViewPort，设置springArm的rotation type为world

2. 在左侧components搜索Character movement并拖到蓝图中，指向get velocity节点，右键split struct pin

3. 将event tick指向print string节点，并将刚才创建的getvelocity节点的x速度与print string的 in string连接便可以打印速度（试试就行了，可以删掉print string节点了）

4. 将get velocity的x指向compare float节点，将event kit指向compare float的exec（执行）

5. 创建get controler节点，return value指向is valid节点（带问号的）的input object

6. compare float节点的>指向isValid的exec

7. get controller节点的return value还要指向set control rotation节点target，isValid节点的isValid再指向set control rotation节点

8. set control rotation节点的new roatation指向make rotator

9. 复制一份get controler节点、is valid节点、set control rotation节点、make rotator节点
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111516020.jpg)

10. 将compare float的<指向复制的isValid的exec，并将复制的make rotator的z改为180

11. 以上是为了当x速度大于0时，人物朝右，小于0时，人物朝左
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111521286.jpg)

12. 我看接下来意思要搞一个状态机来判断朝左朝右

13. InputAxis Moveright节点指向branch，InputAxis Moveright节点的axis value指向greater节点并设置为0，greater节点指向branch的condition

14. 在my blueprint的variables中添加boolean：ismoveingright?并将这个变量拖入蓝图，使用set ismoveingright?节点

15. 将branch的true指向set ismoveingright?节点，并将ismoveingright?勾选

16. 最后将set ismoveingright?节点指向add movement input节点

17. branch的false节点指向一个新的branch节点（其实这里有点没搞懂）

18. InputAxis Moveright节点的axis value指向less节点设置为0，less节点再指向刚才新建的branch的condition，新branch的true指向一个新建的set ismoveingright?节点

19. set ismoveingright?节点指向add movement input节点

20. 新branch的false也指向add movement input节点（感觉这一条线会在松开小键盘右键时触发）
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111658915.jpg)

21. 前往更改朝向的蓝图区域，将compare float的==指向一个branch，branch的condition分支设置为Get ismovingright，如果true，就执行上方（朝右看）的is Valid，false就执行下方的is Valid
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111659978.jpg)

22. 点击左上角compile编译后，设置ismovingright的默认为true

23. 以上是为了当没有移动时，仅仅靠按左右键就能更改人物的朝向（比如现在跳跃时，之前按左右是不能改变朝向的，因为x方向的速度一直没有变的，但加了这部分蓝图后，只要按键就会改变）

## 1.5Giving The Player Character Animations

1. 选中下面这部分蓝图，按下C键，分组为处理输入事件
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111701153.jpg)

2. 新建节点custom event，命名Updateanimation

3. event kit指向一个新建的Updateanimation节点

4. 创建get velocity节点，指向vector length节点再指向greater节点

5. 将左侧component的sprite拖进蓝图，并指向新的节点set flipbook

6. greater指向新建select节点（三个分支），select节点和set flipbook的new flipbook连接

7. select节点true设置为run，false设置为idle

8. greater节点连接select节点

9. updateanimation节点连接set flipbook节点

10. 右键select节点的true，点击promote variable生成run的状态，同理生成idle状态

11. 点击左侧 my bluegraph中的variables，为run和idle创建种类文件夹（category）Animation

12. updateanimation节点指向一个branch

13. 将左侧components中的character movement拖到蓝图中，并指向is falling 节点，is falling 节点指向branch的conditon

14. branch的false连接set flipbook，true

15. 复制select和set flipbook节点（用于创建降落时的动画）

16. 创建get velocity节点，并split struct pin，如果z坐标less（新节点）0，连接select节点

17. 设置select节点true为fall动画（z的速度<0说明在下落），false为jump动画

18. 最后将branch的true连接set filpbook

19. 同上，为jump和fall创建variable（Jumping、Falling）（我不知道为啥这里首字母要大写，之前的run和idle不大写）
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111740964.jpg)

## 1.6Changing The Player Characters Speed

1. 点击左侧component中的character movement 在右侧details搜索speed可以看到速度

2. 将character movement拖入蓝图，指向新建的set max walk speed节点

3. 将event beginplay节点（本来就有）连接set max walk speed节点

4. 为max walk speed新建variable：Playerspeed

5. 编译后，点击playerspeed，然后就可以设置速度啦
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111802611.jpg)

## 1.7Adding Sprinting

1. 先利用graysquare创建个台阶

2. edit -> project settings -> input添加action mappings：sprint，按键为left shift

3. 进入蓝图的处理输入板块

4. 新建sprint节点

5. 拖进来character movement节点指向set max walk speed节点，为其创建variable：SprintingSpeed（我很费解为啥这里的s又大写了，感觉这老师就是随便写的哈哈哈，没太严格的规范）

6. sprint节点的pressed连接set max walk speed节点

7. 设置SprintingSpeed的数值

8. 复制冲刺的节点，改成冲刺完的节点，需要把SprintingSpeed换成之前创建的Playerspeed
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206111802612.jpg)

## 1.8Creating Dash Ability

1. 打开蓝图，修改camera的width至2000

2. 编辑下地图
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206112010940.jpg)

3. 创建dash的fipbook

4. 添加Action Mapping：Dash，按键为Z

5. 前往蓝图中的输入事件板块，创建Dash节点，指向新建的launch character节点（要split struct pin 嗷）

6. 创建get ismovingright节点，指向select节点，select节点的return value 连接launch character节点的launch velocity x

7. select节点的false和true都 设置为2500

8. 勾选launch character节点的XYOverride（试了下，勾没勾没啥区别啊）

9. 由于跳跃时冲刺会冲很远，所以要修改下摩擦力，在蓝图的component中找到character movement，在details中找到jumping/falling，修改falling lateral friction 为2.0

10. 在蓝图的变量中，复制jumping，改为Dash， 将details中的默认值改为dashflipbook（需要先编译一下）

11. 创建一个Boolean变量： isdashing?

12. 将isdashing拖入蓝图（set），使launch character节点指向isdashing?节点，并勾选（true）

13. 将Dash变量拖入蓝图，指向GetTotal Duration，获取动画时间

14. isdashing?节点指向retriggerable delay节点，并将GetTotal Duration的return value连接retriggerable delay节点的duration

15. 最后retriggerable delay节点再指向set isdashing?节点（这次就不勾了false）
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206112056585.jpg)

16. 前往处理动画板块

17. 在Update animation节点后添加branch节点，拖入isdashing节点并连接该branch节点，false连接之前的branch

18. 拖入sprite，指向set flipbook节点

19. 将之前创建的dash变量（其实就是dash的flipbook）拖入，连接set flipbook节点的new flipbook

20. 修改下dash的距离：3300，并修改falling lateral friction 为2.5
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206112104587.jpg)

## 1.9Creating Wall Slide Mechanic

1. 制作wallSlide的flipbook

2. 进入蓝图，创建custom event节点，命名WallSlidemechanic（不是很懂为什么最后一个单词首字母不大写了）

3. 向上找event tick节点，在Updateanimation节点后指向WallSlidemechanic节点

4. 回到WallSlidemechanic节点处，拖入character movement节点，指向is falling节点

5. is falling节点指向一个branch的condition

6. WallSlidemechanic节点执行branch

7. branch true指向line trace channel 节点

8. 新建节点：get actor location，其return value 指向line trace by channel 节点的start

9. 新建节点：get actor forward vector，其return value 指向multiply（就是乘个数呗）（split），x设置为50

10. get actor location节点添加add节点，连接multiply节点和add节点，最后将add节点连接至line trace by channel 节点的end

11. line trace by channel 节点的out hit指向break hit result节点

12. break hit result节点的Blocking hit指向 set Wallslide?节点（新建的变量），ine trace by channel 节点也要执行set Wallslide?节点

13. 在最开始的branch节点的false指向 set Wallslide?节点（不勾选意味着不是falling状态就不能是wallslide状态）
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206112251538.jpg)

14. 回到处理动画板块，在判断是否在空中的branch的true后新建branch，用于判断 is Wallslide状态，false就跟以前一样

15. 拖进sprite并指向set flipbook

16. 添加wallslide动画（flipbook）变量
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206112251539.jpg)

17. 向上找到event tick后面的 wall Slide Mechanic节点，指向一个branch，根据wallslide状态判断，如果false跟以前一样
    
    ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206112251540.jpg)

## 1.10Creating Wall Jump

1. 前往处理输入事件，在Jump的pressed后添加branch，condition就是Wallslide?，false就跟以前一样，true指向launch character（velocity拆分）

2. 新建get actor forward节点（split（拆分） return value） ，x指向mutiply 节点，mutiply 节点设置为-1500（反向跳跃），再指向launch character的节点中的x

3. 为了跳跃时向上一点，将launch character的节点中的z设置为650

4. 勾选上launch character的节点的XYZ override，这样计算的就是绝对数值而不是相对数值

5. 角色在空中时很难控制，所以在蓝图的components中找到character movement，details中搜索air，将air control设置为1

6. 修改角色的碰撞胶囊在蓝图的components中找到capsule component，拖入动画处理板块中，指向set capsule radius节点，radius设置为1

7. 当is falling的branch是true时，连接set capsule radius节点（记得set capsule radius节点要连上之后的branch节点）

8. 复制capsule component节点和set capsule radius节点，当不再空中时，胶囊半径恢复
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206112328749.jpg)

9. 将滑墙系统中mutiply节点的x从50改为5（这个距离就是碰撞检测的距离）
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206112329185.jpg)
