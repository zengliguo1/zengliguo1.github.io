---
title: 复刻《PokemonGo》部分功能的学习记录
date: 2022-06-02 10:47:00 +0800
categories: [Unity3D]
tags: [虚拟现实, 学习笔记]

pin: true
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: false
mermaid: true

---

## 一、实验摘要

    《宝可梦GO》是一款能对现实世界中出现的宝可梦进行探索捕捉、战斗以及交换的游戏。玩家可以通过智能手机在现实世界里发现宝可梦，进行抓捕和战斗。玩家作为宝可梦训练师抓到的宝可梦越多会变得越强大，从而有机会抓到更强大更稀有的宝可梦。

    本次实验复刻了宝可梦GO的LBS地图系统、AR捕捉系统以及仓库系统。

## 二、实验过程

### 0.环境介绍

- Unity2017.3.1

- Andriod SDK 、NDK、OpenJDK均来自于Unity2020.3.14版本安卓模块

### 1.游戏内容分析

#### 传统部分

1. 选择阵营，参与争夺

2. 宠物捕捉

3. 宠物升级进化

4. 宠物PK

5. 道具系统：收集，购买

#### 创新内容

1. LBS：
   
   - 增加与现实世界互动
   
   - 增加玩家代入感
   
   - 增加社交互动

2. AR
   
   - 增加现实世界互动
   
   - 形式新奇

### 2.项目设计

- app功能
  
  1. 用户登录
  
  2. LBS基于地理的服务功能
  
  3. AR捉宠功能
  
  4. 宠物仓库功能
  
  5. 数据更新与保存

- 制作流程及标准
  
  1. 以模块的形式做成独立的项目
  
  2. LBS使用GoMap插件
  
  3. AR使用Vuforia
  
  4. 宠物仓库
  
  5. UGUI制作登陆界面

- 资源
  
  1. 角色模型
  
  2. 精灵球、食物模型
  
  3. 卡通宠物模型
  
  4. UI美术
  
  5. 粒子特效及音效

### 3.unity实现LBS服务（Location Based Service）

1. 先配置环境
- unity2017.3.1

- 安卓SDK（我用的是unity2021版本自带的sdk）
2. 导入GoMap插件包

3. 在demo文件夹拷贝一个POI demo的场景

4. build出一个安卓apk进行测试（注意在导出之前要更改player setting中的package name）

### 4.获取地图运营商密钥

1. 在mapbox官网注册账号
2. 点击token创建新的token
3. 选择默认选项即可
4. 注意，一开始申请注册账号的时候需要visa信用卡，就放弃了注册，但后来给我发送了确认邮件，我验证后就注册成功了

### 5.GoMap的图块机制

![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206021039909.png)

1. tile buffer
   
   图块缓冲越大，一次申请访问的地图就越大

2. Coordinates / units Ratio
   
   单位取真实世界的米

3. Zoom level
   
   值越大，地图比例越小。放大和缩小不会更改比例尺，意味着放再大单位还是取真实世界的米

### 6.GoMap的坐标机制

    开始游戏时，世界中心坐标将会和场景中心坐标(0,0,0)匹配，并且不会变动，人物的行走会改变人物的位置坐标，这与普通地图SDK有所不同（普通地图SDK人物不产生位移，地图画面移动）

### 7.设置地图玩家角色

1. 替换角色模型
   
   - 修改角色比例与方向，再拖到Avatar下，将角色物体拖到Avtar物体的Move Avtar脚本的Avatar Figure上

2. 让模型根据移动速度产生动画的变化
   
   - 创建Animator文件夹（在Avatar_01下），创建Animator controller作为角色模型的控制器
   
   - 将animation文件夹中的Idle、run、walk拖入控制器
   
   - 将any state连接（Transition）三个动画
   
   - 在any state中添加参数：三个trigger（idle、run、walk）
   
   - 在三个连接中添加对应的条件（condition）
   
   - 将控制器拖到模型Boy的Animator组件的controller中

3. 写脚本控制动画
   
   - 创建文件夹，创建脚本BoyControl.cs
   
   ```csharp
   using System.Collections;
   using System.Collections.Generic;
   using UnityEngine;
   
   public class BoyControl : MonoBehaviour {
   //储存动画控制器
   private Animator ator;
   //储存角色控制器
   private MoveAvatar moveAvatar;
   // Use this for initialization
   void Start () {
       //获取动画控制器
       ator = gameObject.GetComponent<Animator>();
       //获取父物体的角色控制器
       moveAvatar = transform.parent.GetComponent<MoveAvatar>();
   }
   
   // Update is called once per frame
   void Update () {
       //如果角色控制器当前的状态是 定义的待机状态
       if(moveAvatar.animationState == MoveAvatar.AvatarAnimationState.Idle)
       {
           //而且当前动画不是待机动画，就将动画控制器的动画设置为待机
           if(!ator.GetCurrentAnimatorStateInfo(0).IsName("Idle"))ator.SetTrigger("Idle");
       }
       else if(moveAvatar.animationState == MoveAvatar.AvatarAnimationState.Run)
       {
           if (!ator.GetCurrentAnimatorStateInfo(0).IsName("Run")) ator.SetTrigger("Run");
       }
       else if(moveAvatar.animationState == MoveAvatar.AvatarAnimationState.Walk)
       {
           if (!ator.GetCurrentAnimatorStateInfo(0).IsName("Walk")) ator.SetTrigger("Walk");
       }
   }
   }
   ```
   
   4. 将main camera的target设置为boy模型，保证视角的正确

### 8.范围内生成随机点

1. 指定预制体作为事件点的载体

2. 获取角色坐标信息

3. 定义随机点与角色距离范围

4. 在距离范围内取一个随机坐标

5. 在随机坐标的位置生成随机点预制体
- 新建一个cube改名为EventPoint，拖到创建的prefab文件夹

- 新建脚本InsPoint.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class InsPoint : MonoBehaviour {
      //储存地图角色
      public GameObject Ava;
      //储存事件点预制体
      public GameObject PrePoint;
      //储存距离范围最小点
      public float MinDis = 3f;
      //储存距离范围最大点
      public float MaxDis = 50f;
      //储存当前角色当前位置
      private Vector3 v3Ava;
      // Use this for initialization
      void Start () {
  
      }
  
      // Update is called once per frame
      void Update () {
  
      }
  
      public void InsPointFuc()
      {
          //获取角色坐标位置
          v3Ava = Ava.transform.position;
          //从距离范围内取一个随机距离值
          float _dis = Random.Range(MinDis, MaxDis);
          //从原点为（0，0）随机获取一个方向的向量
          Vector2 _pOri = Random.insideUnitCircle;
          //将该方向向量标准化
          Vector2 _pNor = _pOri.normalized;
          //计算生成随机点的位置向量             随机点x坐标              随机点y坐标
          Vector3 _v3Point = new Vector3(v3Ava.x + _pNor.x * _dis, 0, v3Ava.z + _pNor.y * _dis);
          //生成随机点的预制体
          GameObject poiMark = Instantiate(PrePoint, _v3Point, transform.rotation);
  
      }
  }
  ```

- 新建一个空物体作为manager，删掉之前创建的eventpoint物体

- 将随机点生成脚本挂载manager上

- 将avtar、预制体prePoint拖到manager的脚本上

- 创建canvas，创建按钮（锚点右上角），添加manager中的随机点函数到按钮中

### 9.在地图上生成随机事件

**随机事件**：

1. 生成小精灵

2. 生成精灵球

3. 生成食物

**实现手段：**

1. 制作三个预制体

2. 在随机点的脚本中随机选取事件

3. 生成对应的预制体

**具体流程**

- 导入资源包食物和精灵球，放到package文件夹下

- 将资源包中prefab的蛋糕和精灵球拖到hierarchy修改比例后放到工程的prefab文件夹作为预制体（将精灵球刚体组件删掉）

- 创建一个圆柱暂时作为宠物

- 新建脚本PointEvent.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class PointEvent : MonoBehaviour {
  
      //分别储存精灵、精灵球、食物的预制体
      public GameObject Pet;
      public GameObject Ball;
      public GameObject Food;
      //储存偏移量
      private Vector3 offset;
      // Use this for initialization
      void Start () {
          int _randomEvent = Random.Range(0, 3);//随机选 0 1 2
          if (_randomEvent == 0) insPet();
          else if (_randomEvent == 1) insBall();
          else if (_randomEvent == 2) insFood();
  
          offset = new Vector3(0, 5f, 0);
      }
  
      // Update is called once per frame
      void Update () {
  
      }
      private void insPet()
      {
          Instantiate(Pet, transform.position + offset, transform.rotation);
      }    
      private void insFood()
      {
          Instantiate(Food, transform.position + offset, transform.rotation);
      }    
      private void insBall()
      {
          GameObject _ball = Instantiate(Ball, transform.position + offset, transform.rotation);
          //设置精灵球的角度
          _ball.transform.localEulerAngles = new Vector3(-30f, 0, 0);
      }
  }
  ```

- 将脚本挂载到eventPoint预制体上，并将蛋糕、精灵球和精灵（现在是圆柱）挂载在脚本上

- 删掉eventPoint预制体上的meshrender和cube组件（不用显示）

- 新建脚本MoveEffect.cs控制随机生成的物体的动态效果
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class MoveEffect : MonoBehaviour {
  
      //设置起始弧度参数
      private float radian = 0f;
      //弧度变化值
      private float perRad = 0.03f;
      //位移偏移量
      private float add = 0f;
      //储存物体生成时的真实坐标
      private Vector3 posOri;
      // Use this for initialization
      void Start () {
          //获取初始位置的坐标信息，否则就是0，0，0
          posOri = transform.position;
      }
  
      // Update is called once per frame
      void Update () {
  
          //弧度值变化
          radian += perRad;
          //计算偏移量
          add = Mathf.Sin(radian);
          //更改物体位置
          transform.position = posOri + new Vector3(0, add, 0);
  
          //控制物体旋转（我试了下，无论是space.self还是world都是绕自己y轴旋转 怪）
          transform.Rotate(0, Time.deltaTime * 25f, 0, Space.World);
  
      }
  }
  ```

### 10.生成精灵

**实现方式：**

1. 制作精灵预制体

2. 使用resource加载预制体

3. 编写随机生成事件

**制作流程**

- 导入精灵包

- 将精灵包文件夹中prefab->low poly -> naked中的预制体拖到hierarchy中，修改大小，改名字为01

- 在精灵文件夹中新进animator文件夹，并创建animator controller

- 将Animation文件夹中的idle basic拖到刚创建的controller中

- 将controller添加到01上

- 在根目录创建Resource文件夹，并在里面创建Pets文件夹，将01拖到pets文件夹中形成预制体

- 将场景中的01删掉

- 以相同的方式添加多个精灵

- 创建脚本InsPets.cs生成宠物
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class InsPets : MonoBehaviour {
      //创建数组 存储精灵预制体
      private GameObject[] pets;
  
      void Awake()
      {
          pets = Resources.LoadAll<GameObject>("Pets");
      }
  
      // Use this for initialization
      void Start () {
          InsPet();
      }
  
      // Update is called once per frame
      void Update () {
  
      }
  
      private void InsPet()//注意不要和脚本同名
      {
          //随机生成一个精灵序号
          int _petIndex = Random.Range(0, pets.Length);
          //生成精灵
          Instantiate(pets[_petIndex], transform.position, transform.rotation);
      }
  }
  ```

- 将脚本拖到之前创建的精灵预制体（圆柱），删掉圆柱的mesh

### 11.随机事件与角色互动

**互动事件：**

1. 角色碰撞到食物或者精灵球时，对应UI数字增加

2. 角色碰到精灵时，出现面板询问是否捕捉

**实现方式：**

1. 给事件添加碰撞器，使用触发碰撞器

2. 给角色增加"Avatar“标签，并增加碰撞器

3. 判断进入事件点碰撞器的物体标签是否为Avatar

**具体流程：**

- 搭建UI，在canvas新建image至左上角，命名为im_Ball,锚点设置在左上角

- 在im_Ball上添加text_Mark 作为×号和text_num作为数字，将锚点放在右侧（因为符号和数字在父物体img的右侧）

- 复制im_Ball做成food

- 创建Pics文件夹再在里面创建UI文件夹，将食物和精灵球图片拖入，将图片类型改为sprite

- 新建脚本UI_Mgr_02.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  using UnityEngine.UI;
  public class UI_Mgr_02 : MonoBehaviour {
  
      //储存精灵球和食物的数量
      public Text Tx_BallNum;
      public Text Tx_FoodNum;
      //申请静态类变量,便于其他脚本直接调用该类的方法
      public static UI_Mgr_02 Instance;
  
      void Awake()
      {
          Instance = this;
      }
  
      public void AddBallNum()
      {
          int _num = int.Parse(Tx_BallNum.text);
          _num++;
          Tx_BallNum.text = _num.ToString();
      }
      public void AddFoodNum()
      {
          int _num = int.Parse(Tx_FoodNum.text);
          _num++;
          Tx_FoodNum.text = _num.ToString();
      }
  
  }
  ```

- 创建碰撞脚本Ball_Find.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class Ball_Find : MonoBehaviour {
  
      //碰撞函数
      void OnTriggerEnter(Collider other)
      {
          if(other.tag == "Avatar")//与标签为“Avatar”的物体碰撞
          {
              UI_Mgr_02.Instance.AddBallNum();
              //销毁物体
              Destroy(gameObject);
          }
      }
  
  }
  ```

- 创建空物体UI_mgr并挂载上UI_Mgr_02.cs，将Im_Ball和Im_Food的数字text挂载到UI_Mgr_02.cs变量上（不要挂maker，不然会报错嗷）

- 更改ball的预制体，添加刚体组件并把限制constraint中的freeze全打上勾，使其不受物理的原因而移动变化，挂上ballFind脚本，将碰撞器的is trigger勾上

- 为boy物体添加”Avatar“标签和box_collider

- 食物同理

### 12.触发精灵捕捉面板

**面板内容：**

1. 提示信息：精灵种类

2. 开始捕捉

3. 放弃捕捉

**流程：**

- 在canvas上添加img_catch

- 在img_catch添加三个text分别是恭喜、发现宠物和宠物名字，再添加两个按钮，分别为捕捉和放弃

- 扩充UI_Mgr_02.cs
  
  ```csharp
      //储存捕捉面板
      public GameObject Im_Catch
  
          //设置捕捉面板的激活状态
      public void SetIm_Catch(bool bl)
      {
          Im_Catch.SetActive(bl);
      };
  ```

- 将捕捉面板物体挂载到ui管理物体的脚本上

- 创建脚本Pet_Find.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class Pet_Find : MonoBehaviour {
        void Start()
      {
          //精灵初始朝向角色
          transform.LookAt(GameObject.FindGameObjectWithTag("Avatar").transform);
      }
      void OnTriggerEnter(Collider other)
      {
          if (other.tag == "Avatar")//与标签为“Avatar”的物体碰撞
          {
              UI_Mgr_02.Instance.SetIm_Catch(true);
            //销毁物体
            Destroy(gameObject);
          }
      }
  
  }
  ```

- 为所有精灵添加碰撞体和刚体，像食物和精灵球那样调整，挂上PetFind.cs

- 为面板的放弃捕捉按钮添加im-mgr物体，并选上SetIm_Catch方法

### 12.LBS部分重构

- 在Resources文件夹中新建文件夹ball和foods

- 将prefab文件夹中的对应物体拖入刚新建的文件夹

- 重构PointEvent.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class PointEvent : MonoBehaviour {
  
      //分别储存精灵、精灵球、食物的预制体
      private GameObject[] Pets;
      private GameObject[] Balls;
      private GameObject[] Foods;
      //储存偏移量,让生成的物体高一点
      private Vector3 offset;
      // Use this for initialization
      void Awake()
      {
          Balls = Resources.LoadAll<GameObject>("Balls");
          Foods = Resources.LoadAll<GameObject>("Foods");
          Pets = Resources.LoadAll<GameObject>("Pets");
      }
      void Start () {
          offset = new Vector3(0, 5f, 0);
          int _randomEvent = Random.Range(0, 3);//随机选 0 1 2
          if (_randomEvent == 0) insPet();
          else if (_randomEvent == 1) insBall();
          else if (_randomEvent == 2) insFood();
      }
  
      // Update is called once per frame
      void Update () {
  
      }
      private void insPet()
      {
          // 随机生成一个精灵序号
          int _petIndex = Random.Range(0, Pets.Length);
          Instantiate(Pets[_petIndex], transform.position + offset, transform.rotation);
      }    
      private void insFood()
      {
          int _foodIndex = Random.Range(0, Foods.Length);
          Instantiate(Foods[_foodIndex], transform.position + offset, transform.rotation);
      }    
      private void insBall()
      {
          int _ballIndex = Random.Range(0, Foods.Length);
          GameObject _ball = Instantiate(Balls[_ballIndex], transform.position + offset, transform.rotation);
          //设置精灵球的角度
          _ball.transform.localEulerAngles = new Vector3(-30f, 0, 0);
      }
  
  }
  ```

- 删掉InsPet.cs和Pet预制体

- 使用代码为预制体增加组件，继续重构PointEvent.cs，修改脚本中的insBall和insFood函数即可
  
  ```csharp
    private void insFood()
      {
          int _foodIndex = Random.Range(0, Foods.Length);
          GameObject _food = Instantiate(Foods[_foodIndex], transform.position + offset, transform.rotation);
  
          //添加碰撞体组件
          _food.AddComponent<BoxCollider>();
          //勾选isTrigger
          _food.GetComponent<BoxCollider>().isTrigger = true;
          //设置碰撞体
          _food.GetComponent<BoxCollider>().center = new Vector3(0, 0, 0);
          _food.GetComponent<BoxCollider>().size = new Vector3(0.33f, 0.3f, 0.33f);
          //添加刚体组件
          _food.AddComponent<Rigidbody>();
          //更改刚体组件的限制
          _food.GetComponent<Rigidbody>().constraints = RigidbodyConstraints.FreezeAll;
          //添加moveeffect组件和find组件
          _food.AddComponent<MoveEffect>();
          _food.AddComponent<Food_Find>();
      }    
      private void insBall()
      {
          int _ballIndex = Random.Range(0, Foods.Length);
          GameObject _ball = Instantiate(Balls[_ballIndex], transform.position + offset, transform.rotation);
          //设置精灵球的角度
          _ball.transform.localEulerAngles = new Vector3(-30f, 0, 0);
          //添加球形碰撞体组件
          _ball.AddComponent<SphereCollider>();
          //勾选isTrigger
          _ball.GetComponent<SphereCollider>().isTrigger = true;
          //设置碰撞体大小
          _ball.GetComponent<SphereCollider>().radius = 0.011f;
          //添加刚体组件
          _ball.AddComponent<Rigidbody>();
          //更改刚体组件的限制
          _ball.GetComponent<Rigidbody>().constraints = RigidbodyConstraints.FreezeAll;
          //添加moveeffect组件和find组件
          _ball.AddComponent<MoveEffect>();
          _ball.AddComponent<Ball_Find>();
      }
  ```

### 13.配置Vuforia

1. 在周围随机出现精灵

2. 发射精灵球
- 新建一个项目pokemonGo_AR

- 创建ARCamera时导入Vuforia的包

- 切换至安卓平台

- 更改playersetting中的XR settings 勾选Auforia

- 为ARcamera添加密钥

- 更改ARcamera下VuforiaConfiguraton的Device Tracker，勾选Track Device Pose

- 导出apk测试（导之前更改playersetting 里的名称还有other setting里的Android TV要取消掉）

### 14.规划AR模块的制作（生成精灵球）

**功能流程**

1. 读取数据

2. 配置精灵球

3. 配置精灵

4. 可以发射精灵球

5. 触发精灵捕捉成功

**流程：**

- 创建cube作为ballpos，调整位置与大小并拖给ARcamera作为子物体

- 创建一个球体作为精灵球，调整大小位置，删除碰撞器组件（因为要在代码中动态添加）

- 在Resources文件夹中创建Balls文件夹，并将球体拖进去作为预制体（在合并项目时，同名文件夹也会合并）

- 创建scripts文件夹再创建AR文件夹，在里面创建ARBallCtrl.cs脚本
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class ARBallCtrl : MonoBehaviour {
      //储存精灵球预制体
      private GameObject[] balls;
      //精灵球生成位置
      public Transform PosInsBall;
      // Use this for initialization
      void Start () {
          //加载路径在“Resources/Balls”的预制体
          balls = Resources.LoadAll<GameObject>("Balls");
          insBall();
      }
  
      // Update is called once per frame
      void Update () {
  
      }
      private void insBall()
      {
          Instantiate(balls[0], PosInsBall.position, PosInsBall.rotation);
          //设置精灵球的父物体 为了保证发射前始终在屏幕的固定位置
        _ball.transform.SetParent(PosInsBall);
        //为精灵球添加球体碰撞器组件
        _ball.gameObject.AddComponent<SphereCollider>();
  
      }
  }
  ```

- 新建脚本管理器，添加ARBallCtrl.cs脚本，将BallPos拖到脚本中

### 15.发射精灵球

**发射效果**

1. 用手指滑动发射

2. 发射后精灵球运动的过程类似于真实物体抛射

3. 可以碰到精灵

4. 受重力影响下落

5. 落在地面上

**制作思路**

1. 有碰撞器

2. 有刚体

3. 计算手指滑动方向

4. 计算手指滑动距离

5. 计算手指滑动的速度

**注意事项**

1. 有的滑动方向不能触发精灵球发射

2. 滑动距离不够不能触发精灵球发射

3. AR交互对算法的影响

**流程**

- 创建脚本ARShootBall.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class ARShootBall : MonoBehaviour {
  
      //设置给精灵球的力度
      public float FwdForce = 200f;
      //设置夹角的参照数值
      public Vector3 StanTra = new Vector3(0, 1f, 0);
  
      //判断手指是否碰到精灵球位置
      private bool blTouched = false;
      //判断精灵球是否发射
      private bool blShooted = false;
      //手指滑动起始点
      private Vector3 startPosition;
      //手指滑动的终点
      private Vector3 endPosition;
      //手指滑动的距离
      private float disFlick;
      //手指滑动的速度
      private float speedFlick;
      //手指滑动的时间(帧数)
      private int timeFlick = 0;
      //手指滑动的偏移向量
      private Vector3 offset;
      //记录主摄像机
      private Camera cameraMain;
  
      // Use this for initialization
  
      void Start () {
          //赋值为主摄像机
          cameraMain = Camera.main;
      }
  
      // Update is called once per frame
      void Update () {
          if(blTouched)//如果按在小球上，允许检测手指的滑动
          {
              slip();
  
          }
      }
      //重置参数
      private void resetVari()
      {
          //起始位置为手指按下的位置
          startPosition = Input.mousePosition;
      }
      //当鼠标/手指 按下/触摸到 脚本挂载的物体
      void OnMouseDown()
      {
          if(blShooted == false)//如果没发射
          {
              blTouched = true;//允许检测手指滑动
          }
      }
      //计算手指的滑动
      private void slip()
      {
          timeFlick += 25;//时间每帧增加25
          if(Input.GetMouseButtonDown(0))//当手指按下的时候
          {
              resetVari();//重置参数
          }
          if(Input.GetMouseButton(0))//当手指一直按下的时候
          {
              //将终点位置坐标刷新为手指位置
              endPosition = Input.mousePosition;
              //获取手指在世界坐标上的偏移量
              offset = cameraMain.transform.rotation * (endPosition - startPosition);
              //计算手指滑动的距离
              disFlick = Vector3.Distance(endPosition, startPosition);
          }
          if(Input.GetMouseButtonUp(0))//当手指抬起来
          {
              //计算速度
              speedFlick = disFlick / timeFlick;
              //手指触摸设置为false
              blTouched = false;
              //时间重置
              timeFlick = 0;
              //如果手指滑动距离超过20而且是向上滑动
              if(disFlick > 20 && endPosition.y- startPosition.y > 0)
              {
                  //允许发射
                  shootBall();
  
              }
          }
      }
  
      //发射精灵球
      private void shootBall()
      {
          //给精灵球添加刚体组件
          transform.gameObject.AddComponent<Rigidbody>();
          //使用局部变量储存刚体组件，便于后续操作
          Rigidbody _rigBall = transform.GetComponent<Rigidbody>();
          //给精灵球一个初始速度 
          _rigBall.velocity = offset.normalized * speedFlick;
          //为精灵球添加力度,方向为朝着屏幕前方（即为摄像机的前方）
          _rigBall.AddForce(cameraMain.transform.forward * FwdForce);
          //让精灵球旋转
          _rigBall.AddTorque(transform.right);
          //设置精灵球阻力 
          _rigBall.drag = 0.5f;
          //设置为发射状态
          blShooted = true;
          //让发射出去的球脱离父物体
          transform.parent = null;
          //执行协程函数
          StartCoroutine(LateInsBall());
      }
  
      //协程函数
      IEnumerator LateInsBall()
      {
          yield return new WaitForSeconds(0.2f);
          //延迟两秒执行
          //生成新的精灵球
          ARBallCtrl.Instance.InsNewBall();
      }
  }
  ```

- 删掉BallPos的碰撞器和渲染插件，只需要为精灵球提供位置

- 将ARBallCtrl.cs的脚本生成新精灵球函数insBall改为公有函数方便调用，并修改：
  
  ```csharp
    //申请脚本静态类对象
    public static ARBallCtrl Instance;
    public void InsNewBall()
  ```

- 创建材质文件夹，新建一个带颜色的材质赋给cube方便区分球和cube

### 16.AR场景中生成精灵

**流程**

- 升高ARcamera的y坐标至1.6

- 在Resources文件夹新建Pets文件夹，将cube拖入作为预制体

- 在ARcamera的周围创建多个生成点cube

- 删除cube的碰撞器和网格（因为只需要提供位置）

- 新建空物体PosMgr统一管理生成点（作为生成点的父物体）

- 新建脚本ARInsPets.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class ARInsPet : MonoBehaviour {
  
      //储存生成精灵位置的物体
      public Transform[] TraPos;
      //储存精灵
      private GameObject[] pets;
      // Use this for initialization
      void Start () {
          pets = Resources.LoadAll<GameObject>("Pets");
      }
  
      // Update is called once per frame
      void Update () {
  
      }
  
      public void InsPet()
      {
          //随机选一个生成的位置点
          int _index = Random.Range(0, TraPos.Length);
          //获取该位置的transform
          Transform _tra = TraPos[_index];
          //在该位置生成精灵
          Instantiate(pets[0], _tra.position, _tra.rotation);
  
      }
  }
  ```

- 将脚本挂载到PosMgr上，并将五个位置点cube放在脚本上

### 17.地图模块与AR模块的整合

**目标**

1. 将两个项目文件合并

2. 测试调整整合后的项目场景

3. 场景的跳转

**流程**

- 打开pokemonGo_map项目文件，导出包

- 复制pokemonGo_AR项目文件改名为pokemonGo_main，作为主项目文件

- 导入包

- 切换场景到map

- 测试，为防止鼠标移动到按钮时相机发生变化，新建一个ARcamera并关掉所有组件

- 切换场景到AR

- 由于在文件合并时，相同文件夹的内容也会合并，同名的预制体会覆盖之前的预制体，所以要调整Ball

- 制作捕捉精灵时的跳转函数，修改UI_Mgr脚本
  
  ```csharp
  using UnityEngine.SceneManagement;
  
  public void Btn_GoARScn()
    {
        SceneManager.LoadScene("AR_Scn");
    }
  ```

- 要将场景添加到build setting中

- 切换至AR_Scn,创建一个脚本ARUI_Mgr.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  using UnityEngine.SceneManagement;
  public class ARUI_Mgr : MonoBehaviour {
  
      public void Btn_GoMapScn()
      {
          SceneManager.LoadScene("Map_Scn");
      }
  }
  ```

### 18.Resource中的预制体标准

**标准**

1. 同一文件夹中的预制体类型要相同

2. 同一批预制体的配置信息保持一致（坐标轴位置、旋转角度、模型朝向、导出前的操作）

3. 外部导入的资源在进入unity前就要统一标准

**流程**

- 修改ARInsPet脚本，使精灵大小合适，在生成精灵函数最后进行修改
  
  ```csharp
  //在该位置生成精灵
          GameObject _pet = Instantiate(pets[0], _tra.position, _tra.rotation);
          //调整缩放比例
          _pet.transform.localScale = new Vector3(0.5f, 0.5f, 0.5f) ;
  ```

- 将BallPos 的scale全改为1

- 修改ARBallCtrl.cs脚本中生成精灵球函数最后加上
  
  ```csharp
  //设置精灵球的大小为25倍
          _ball.transform.localScale = new Vector3(25f, 25f, 25f);
  ```

- 用代码来调整碰撞器的缩放比，并添加发射精灵球脚本
  
  ```csharp
          //为精灵球添加发射精灵球脚本
          _ball.gameObject.AddComponent<ARShootBall>();
          //设置精灵球的大小为25倍
          _ball.transform.localScale = new Vector3(25f, 25f, 25f);
          //获取碰撞器并更改半径
          _ball.GetComponent<SphereCollider>().radius = 0.01f;
  ```

### 19.数据的全局管理

**流程**

- 切换至map场景，将img_Ball拖入Prefeb文件夹形成预制体

- 回到AR场景，将预制体拖到场景中

- 在脚本文件夹，创建Map文件夹，把之前的脚本拖到Map中

- 再新建一个Static文件夹，用来存放全局脚本

- 新建脚本StaticData.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public static class StaticData {
      //精灵球初始数量
      public static int BallNum = 5;
  }
  ```

- 修改UI_Mgr脚本
  
  ```csharp
      //精灵球数量
    public Text Tx_BallNum;
    //单例
    public static ARUI_Mgr Instance;
    void Awake()
    {
        Instance = this;
    }
    public void UpdateUIBallNum()
    {
        Tx_BallNum.text = StaticData.BallNum.ToString();
    }
  ```

- 修改ARBallCtrl脚本
  
  ```csharp
      void Start () {
        //加载路径在“Resources/Balls”的预制体
        balls = Resources.LoadAll<GameObject>("Balls");
        //刷新精灵球数量
        ARUI_Mgr.Instance.UpdateUIBallNum();
        InsNewBall();
    }
    public void InsNewBall()
    {
        if(StaticData.BallNum > 0)
        {
            GameObject _ball = Instantiate(balls[0], PosInsBall.position, PosInsBall.rotation);
            //设置精灵球的父物体 为了保证发射前始终在屏幕的固定位置
            _ball.transform.SetParent(PosInsBall);
            //为精灵球添加球体碰撞器组件
            _ball.gameObject.AddComponent<SphereCollider>();
            //为精灵球添加发射精灵球脚本
            _ball.gameObject.AddComponent<ARShootBall>();
            //设置精灵球的大小为25倍
            _ball.transform.localScale = new Vector3(25f, 25f, 25f);
            //获取碰撞器并更改半径
            _ball.GetComponent<SphereCollider>().radius = 0.01f;
  
        }
    }
  ```

- 修改精灵球发射脚本ARShootBall中的发射函数，在发射结束后添加以下两行代码
  
  ```csharp
          //减少精灵球数量
          StaticData.BallNum--;
          //更新UI
          ARUI_Mgr.Instance.UpdateUIBallNum();
  ```

- 回到地图场景

- 修改UI_Mgr_02.cs脚本，增加start函数，初始化场景精灵球数量，修改增加精灵球数量
  
  ```csharp
      void Start()
      {
          Tx_BallNum.text = StaticData.BallNum.ToString();
      }
      public void AddBallNum()
      {
          StaticData.BallNum++;
          Tx_BallNum.text = StaticData.BallNum.ToString();
      }
  ```

- 保存精灵的全局变量：修改StaticData.cs脚本
  
  ```csharp
      //当前正要捕捉的精灵序号
      public static int CatchingPetIndex;
  ```

- 修改Pet_Find脚本，修改void OnTriggerEnter(Collider other)函数
  
  ```csharp
  void OnTriggerEnter(Collider other)
      {
          if (other.tag == "Avatar")//与标签为“Avatar”的物体碰撞
          {
              //显示捕捉面板
              UI_Mgr_02.Instance.SetIm_Catch(true);
              //当与角色碰撞时，更改全局变量：要捕捉的精灵序号
              StaticData.CatchingPetIndex = PetIndex;
              //销毁物体
              Destroy(gameObject);
          }
      }
  ```

- 修改PointEvent脚本的生成精灵函数
  
  ```csharp
  private void insPet()
      {
          // 随机生成一个精灵序号
          int _petIndex = Random.Range(0, Pets.Length);
          //储存随机生成的精灵
          GameObject _pet = Instantiate(Pets[_petIndex], transform.position + offset, transform.rotation);
          //更新精灵的序号
          _pet.GetComponent<Pet_Find>().PetIndex = _petIndex;
      }    
  ```

- 回到AR场景，修改ARInsPet脚本中的InsPet函数。并修改start函数
  
  ```csharp
      //在该位置生成要捕捉的精灵
      GameObject _pet = Instantiate(pets[StaticData.CatchingPetIndex], _tra.position, _tra.rotation);
  
  void Start () {
      pets = Resources.LoadAll<GameObject>("Pets");
      //生成精灵
      InsPet();
  }
  ```

- 删掉之前生成精灵的按钮

### 20.精灵捕捉的处理

**功能效果**

1. 精灵播放被捕捉到的动画

2. 精灵消失

3. 显示捕捉成功界面

**流程**

- 打开PetsAnimator

- 打开Animation_FBX文件夹，将Hurt_die动画和jump_basic动画拖到Animator中

- 连接：Any State -> hurt -> jump

- 添加一个Trigger参数Catched

- 将Any State -> hurt的连线condition添加参数Catched

- 回到AR场景，在canvas上创建一个Panel组件Pn_Catched，锚点在四角

- 在panel上添加输入框

- 新建两个按钮，一个是确认，一个是放生

- 修改ARUI.Mgr脚本，添加以下内容
  
  ```csharp
      //储存面板
      public GameObject PnCatched;
        public void Show_PnCatched()
    {
        PnCatched.SetActive(true);
    }
  ```

- 给精灵球添加Ball标签

- 修改PetFind脚本
  
  ```csharp
  //延迟两秒显示捕捉面板并销毁精灵
    IEnumerator ShowCatchedPn()
    {
        yield return new WaitForSeconds(2f);
        //显示捕捉面板
        ARUI_Mgr.Instance.Show_PnCatched();
        //销毁精灵物体
        Destroy(transform.gameObject);
    }
    //播放被捕捉的动画
    private void playCatched()
    {
        transform.GetComponent<Animator>().SetTrigger("Catched");
    }
    void OnTriggerEnter(Collider other)
    {
        if (other.tag == "Avatar")//与标签为“Avatar”的物体碰撞
        {
            //显示捕捉面板
            UI_Mgr_02.Instance.SetIm_Catch(true);
            //当与角色碰撞时，更改全局变量：要捕捉的精灵序号
            StaticData.CatchingPetIndex = PetIndex;
            //销毁物体
            Destroy(gameObject);
        }
        if(other.tag == "Ball")
        {
            //播放动画
            playCatched();
            //显示面板
            ShowCatchedPn();
  
        }
    }
  ```

### 21.精灵命名

**流程**

- 在脚本文件夹中创建Pet文件夹，新建PetSave脚本
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class PetSave{
      //记录精灵名字
      private string strName = "未命名宠物";
      //记录精灵模型的序号
      private int petIndex = 0;
      //精灵名字属性
      public string PetName
      {
          get { return strName; }
          set { strName = value; }
      }
      //精灵序号属性
      public int PetIndex
      {
          get { return petIndex; }
          set { petIndex = value; }
      }
  
      //构造函数
      public PetSave(string name, int index)
      {
          PetName = name;
          PetIndex = index;
      }
  }
  ```

- 修改全局数据脚本StaticData
  
  ```csharp
      //申请列表储存捕捉到的精灵
      public static List<PetSave> PetList = new List<PetSave>();
      /// <summary>
      /// 向全局数据的精灵列表中添加精灵
      /// </summary>
      /// <param name="petSave">精灵的属性类 </param>
      public static void AddPet(PetSave petSave)
      {
          PetList.Add(petSave);
      }
  ```

- 修改ARUI.Mgr脚本，为捕捉面板的按钮添加函数
  
  ```csharp
     //储存精灵名字
    public Text InputPetName;
      //捕捉面板的确认按钮
      public void Btn_Yes()
      {
          //从输入框中获取精灵名字
          string _name = InputPetName.text;
          //从全局脚本中获取精灵序号
          int _index = StaticData.CatchingPetIndex;
          //向全局脚本的精灵列表中添加一条精灵属性类数据
          StaticData.AddPet(new PetSave( _name, _index));
      }
  ```

### 22.精灵仓库的制作

**流程**

- 新建Store.Scn场景

- 创建Text，修改canvas的canves Scaler中mode为Scale with the screen Size并修改分辨率为1080*1920

- 调整text大小尺寸，命名为Tx_Title并将锚点放在左上角

- 新建按钮放在右上角，返回

- 新建画布Cav_world，修改canves Scaler中mode为Scale with the screen Size并修改分辨率为1080*1920

- 在Cav_world上新建panel调整大小

- 在panel上新建text组件保存信息：宠物名称和宠物类型

- 添加image用来储存精灵图

- 再复制两个panel

- 将摄像机模式改为正交模式并调整相机大小

- 将Cav_World的canvas组件中Render Mode改为Camera，并将main camera拖到Render camera上

- 创建三个空物体储存精灵的位置

- 新建空物体Pos_Mgr管理这三个位置

- 在脚本文件夹中新建文件夹Store，创建新脚本StoreInsPets.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class StoreInsPet : MonoBehaviour {
      //储存展示的位置
      public Transform[] Pos;
      //储存所有精灵预制体
      private GameObject[] pets;
  
      //储存展示的精灵
      private GameObject[] petsShow = new GameObject[3];
  
      void Awake()
      {
          pets = Resources.LoadAll<GameObject>("Pets");
      }
      // Use this for initialization
      void Start () {
          InsPet();
      }
  
      // Update is called once per frame
      void Update () {
  
      }
  
      public void InsPet()
      {
          // 获取精灵数量
          int _petNum = StaticData.PetList.Count;
          //如果仓库中有精灵
          if(_petNum > 0)
          {
              for(int i = 0; i < 3; i++)
              {
                  //如果i大于了最后一个精灵的序号
                  if (i > (_petNum - 1)) return;
                  //储存精灵信息
                  PetSave _petInfo = StaticData.PetList[i];
                  //生成精灵
                  Instantiate(pets[_petInfo.PetIndex], Pos[i].position, Pos[i].rotation);
  
                  //获取精灵的名字
                  string _petNm = _petInfo.PetName;
                  //更新精灵的名字
                  StoreUIMgr.Instance.UpdatePetNm(i, _petNm);
              }
          }
  
      }
  }
  ```

- 新建脚本StoreUIMgr.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  using UnityEngine.UI;
  public class StoreUIMgr : MonoBehaviour {
      //显示精灵名字的text
      public Text[] PetName;
      //显示精灵类型的text
      public Text[] PetType;
      //单例
      public static StoreUIMgr Instance;
  
      void Awake()
      {
          Instance = this;
      }
  
      //更新精灵的名字函数
      public void UpdatePetNm(int index, string strName)
      {
          PetName[index].text = strName;
      }
  }
  ```

- 新建一个空物体Mgr，并将刚才创建的两个脚本挂载上去

- 将三个位置物体挂载到Mgr的StoreInsPets下，三个名字和类型text挂载到Mgr的StoreUIMgr

- 切换到AR场景，在ARUI_Mgr脚本的Btn_Yes()函数最后加上跳转场景的代码
  
  ```csharp
          //跳转到仓库界面
          SceneManager.LoadScene("Store_Scn");
  ```

- 给确定按钮绑定上Mgr物体上的ARUI_Mgr脚本的Btn_Yes()函数

- 由于精灵仓库场景中ARcamera的影响使得精灵显示不正常，新建一个ARcamera，关掉组件

### 23.完善

**场景切换设计：**

1. 地图场景：
   
   - 可随时跳转到精灵仓库
   
   - 碰到精灵时可跳转到AR场景

2. AR场景：
   
   - 随时跳转到精灵仓库
   
   - 捕捉成功跳转到精灵仓库
   
   - 随时跳转到地图场景
   
   - 放弃捕捉时跳转到地图场景

3. 仓库场景
   
   - 随时跳转到地图场景

**流程**

- 修改StoreUIMgr脚本，添加场景切换函数
  
  ```csharp
  using UnityEngine.SceneManagement;
      //跳转到地图场景
      public void Btn_ToMap()
    {
        SceneManager.LoadScene("Map_Scn");
    }
  ```

- 给返回按钮添加Mgr物体，选择StoreUIMgr脚本的Btn_ToMap()函数

- 返回Map场景，复制扫描按钮，命名为精灵仓库

- 修改UI_Mgr_02脚本，添加跳转场景函数

- 回到AR场景，复制回到地图按钮，命名精灵仓库

- 修改ARUI_Mgr脚本，添加跳转到精灵仓库和放生的函数

**完善**

1. AR场景

2. 仓库场景

**流程**

- 修改ARInsPet脚本，将ARcamera挂载到脚本上
  
  ```csharp
  //储存相机位置
  public Transform CameraTra;
  
   InsPet()中最后添加：
  //让生成的精灵面向我们
  _pet.transform.LookAt(new Vector3(CameraTra.position.x, _pet.transform.position.y, CameraTra.position.z));
  ```

- 切换到精灵仓库场景，添加宠物类型显示功能，修改StoreUIMgr脚本，添加刷新精灵种类的函数
  
  ```csharp
      /// <summary>
      /// 刷新精灵类型函数
      /// </summary>
      /// <param name="index">精灵序号</param>
      /// <param name="strType">精灵类型名字</param>
      public void UpadatePetType(int index, string strType)
      {
          Tx_PetType[index].text = strType;
      }
  ```

- 再StaticData脚本中定义一个静态函数，用来根据序号返回精灵的种类
  
  ```csharp
  public static string GetPetName(int index)
      {
          if(index == 0)
          {
              return "不灭狂雷";
          }
          else if(index == 1)
          {
              return "牛头酋长";
          }
          else if (index == 2)
          {
              return "赤小兔";
          }
          else if (index == 3)
          {
              return "沙漠皇帝";
          }
          else if (index == 4)
          {
              return "乌迪尔";
          }
          else if (index == 5)
          {
              return "傲之追猎者";
          }
          else if (index == 6)
          {
              return "麻花藤";
          }
          else if (index == 7)
          {
              return "沙漠死神";
          }
          else
          {
              return "不屈之枪";
          }
  
      }
  ```

- 修改StoreInsPet脚本，在InsPet函数最后添加刷新类型的函数
  
  ```csharp
      /// <summary>
      /// 刷新精灵类型函数
      /// </summary>
      /// <param name="index">精灵序号</param>
      /// <param name="strType">精灵类型名字</param>
      public void UpadatePetType(int index, string strType)
      {
          Tx_PetType[index].text = strType;
      }
  ```

### 24.美术及音效

- 新建一个登录场景Login_Scn

- 为防止Vuforia的影响，先创建一个ARcamera，关掉所有组件

- 在Pic文件夹中创建Icons、Btn、InputF、BG、Words文件夹，把之前的图标图片放到Icons中，导入美术资源，设置为sprite，记得apply

- 创建canvas，再创建一个panel作为背景

- 将导入的背景图片设置为panel的源图像，并将不透明度调为100%

- 在背景Panel上新建image，并选择导入的美术资源，设置锚点（就近原则）

- 新建inputfile，一个输入用户名，一个输入密码

- 新建一个登录按钮

- 新建几个image装饰一下

- 为登录添加跳转功能，在脚本文件夹新建Login文件夹，创建Login_UI脚本
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  using UnityEngine.SceneManagement;
  public class Login_UI : MonoBehaviour {
  
      public void Login()
      {
          SceneManager.LoadScene("Map_Scn");
      }
  }
  ```

- 新建管理器物体，将脚本挂载到管理器上，为按钮添加跳转功能

- 新建一个文件夹Audios，存放音乐与音效，导入音乐包

- 创建一个空物体管理声音AuMgr（用来添加按钮的音源）,在Mgr添加audio source组件，勾选loop，选择一个bgm导入

- 创建一个脚本Login_Au
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class Login_Au : MonoBehaviour {
  
      private AudioSource Au;
      //静态单例
      public static Login_Au Instance;
  
      void Awake()
      {
          Instance = this;
      }
      // Use this for initialization
      void Start () {
          Au = GetComponent<AudioSource>();
      }
  
      public void Btn_Play()
      {
          Au.Play();
      }
  }
  ```

- 切换到Map场景，换掉场景中所有按钮图片

- 为Mgr添加音源

- 新建脚本Map_Au
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public class Map_Au : MonoBehaviour {
  
      private AudioSource Au;
      //静态单例
      public static Map_Au Instance;
  
      void Awake()
      {
          Instance = this;
      }
      // Use this for initialization
      void Start()
      {
          Au = GetComponent<AudioSource>();
      }
  
      public void Btn_Play()
      {
          Au.Play();
      }
  }
  ```

- 新建空物体AuMgr，挂载声音控制脚本

- 给每个按钮的响应函数内添加一条：
  
  ```csharp
          //播放按钮声音
          Map_Au.Instance.Btn_Play();
  ```

- 切换到AR界面

- 更改UI按钮、添加BGM、按钮音效、AR场景的AU管理脚本

- 切换到精灵仓库界面，重复上面的操作

### 25.游戏保存功能

- 导入Easy Save包

- 切换到仓库场景，复制一个按钮为保存

- 在静态脚本文件夹创建脚本SaveAndLoad.cs
  
  ```csharp
  using System.Collections;
  using System.Collections.Generic;
  using UnityEngine;
  
  public static class SaveAndLoad{
      public static void Save()
      {
          //保存精灵球数量
          ES3.Save<int>("BallNum", StaticData.BallNum);
          //保存已捕捉的精灵的数量
          ES3.Save<int>("PetNum", StaticData.PetList.Count);
          //保存每一个已捕捉到的精灵信息
          for(int i = 0; i < StaticData.PetList.Count; i++)
          {
              //储存每一个精灵的名字
              ES3.Save<string>("PetName" + i.ToString(), StaticData.PetList[i].PetName);
              //储存每一个精灵的序号
              ES3.Save<int>("PetIndex" + i.ToString(), StaticData.PetList[i].PetIndex);
          }
      }
      //加载函数
      public static void Load()
      {
          if(ES3.KeyExists("BallNum") && ES3.KeyExists("PetNum"))
          {
              //读取精灵球数量
              StaticData.BallNum = ES3.Load<int>("BallNum");
              //读取精灵数量
              int _petNum = ES3.Load<int>("PetNum");
              //读取每个精灵的信息
              for(int i = 0; i < _petNum; i++)
              {
                  string _petName = ES3.Load<string>("PetName" + i.ToString());
                  int _petIndex = ES3.Load<int>("PetIndex" + i.ToString());
                  StaticData.AddPet(new PetSave(_petName, _petIndex));
              }
          }
      }
  
  }
  ```

- 修改StoreUI_Mgr脚本，添加保存按钮的函数，并给按钮加上函数
  
  ```csharp
      public void Btn_Save()
      {
          Store_Au.Instance.Btn_Play();
          SaveAndLoad.Save();
      }
  ```

- 将加载函数加载登录的按钮函数内（在LoginUI脚本中），登录即读取
  
  ```csharp
      public void Login()
      {
          Login_Au.Instance.Btn_Play();
          SaveAndLoad.Load();
          SceneManager.LoadScene("Map_Scn");
      }
  ```

## 三、项目界面展示

| 界面                                                                               | 介绍                                                                                                                           |
| -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206021039903.jpg) | 登录界面：该界面用户名密码其实并没有效果， 点击登录，会自动读取上一次的记录并跳转到地图界面                                                                               |
| ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206021039904.jpg) | 地图界面：打开app后允许gps功能，玩家在现实世界移动，游戏世界的角色也会跟着移动，地图为学校。左上方显示精灵球和食物数量，通过点击右上角的扫描区域，可以在周围随机刷新精灵球、食物和精灵。玩家碰撞到精灵球、食物，数量会增加，碰撞到精灵弹出捕捉面板 |
| ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206021039907.jpg) | AR战斗界面：玩家手持手机环顾四周，寻找精灵，找到精灵后，通过滑动手指的动作来投掷精灵球，碰撞到精灵便捕捉成功，捕捉成功后显示成功面板，起名确认后跳转到仓库界面，否则跳转到地图界面                                   |
| ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206021039902.jpg) | 仓库界面：玩家捕捉的精灵将显示在该界面，点击保存将会保存当前精灵，下次登录时会自动读取                                                                                  |

## 四、实验中遇到的问题

1. 环境配置问题，如下
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206021039908.jpg)

    在上一次实验时，我就想导出安卓包，但出现了SDK的各种问题，先是找不到SDK，然后是版本低，不能够支持最新版本的Vuforia，尝试了很多办法。最开始解决找不到SDK时，把Unity卸载了重装，但安装又出现了问题，又看博客的单独安装，就是把各种需要的模块、包、SDK等等单独下载，再整合一起，找到SDK了，但版本又不支持，于是又重新下载了最新的Unity，但最新的Unity的安卓sdk版本依旧不支持最新版本的Auforia，于是乎，便排查出不是Unity的问题，而是安卓包和Auforia版本不匹配的问题，最后又下回了Unity2020的稳定版本，导入Vuforia后，降低Vuforia的版本后能够成功导出安卓包了。但此次实验的环境是Unity2017，因为Unity2017自带Vuforia，更方便，根本不用担心安卓包和Auforia版本不匹配的问题。

2. 实验开始阶段MapBox的密钥申请问题

    MapBox以往和Vuforia是一样的，只要注册登录就可以申请密钥，使用其LBS插件的功能，但今年有所变化，注册账号必须要visa等国际银行卡，但这种银行卡不是随随便便轻而易举就能办下来的，所以正在我一筹莫展之时，MapBox一小时后还是给我发来了验证邮件，没有银行卡还是注册账号成功了。

3. 中间有时会发生一些小bug，不过无足轻重很快就能调整好，比较麻烦的问题是最后的canvas布局问题
   
   ![](https://raw.githubusercontent.com/CALL1CE/ImgStage/main/202206021039906.jpg)

    导到手机上测试时，发现仓库界面的精灵信息面板发生重叠，发现是分辨率的不同导致的，电脑上设置的分辨率是1080*1920，而我的手机估计是10：21的比例，所以有所不同，于是查阅博客对canvas scaler的描述，更改canvas scaler的UI Scale Mode为 Scale With Screen Size，并根据情况更改Match即可，像上面这个，宽度其实不用缩放，只需要根据手机的高度来缩放UI的高度即可，因此Match拉到1。

## 五、个人收获

    本项目的技术难点不多，大多学习到的是项目的管理和游戏基本功能在Unity中的实现

1. 项目管理方式，为不同模块创建不同的文件夹存放脚本、预制体、资源等内容。在场景中使用不同的Manager空物体来管理场景中的脚本事件，比如Au_Mgr管理音频的脚本事件，Mgr管理场景主要事件的脚本，UI_Mgr管理界面的事件脚本。

2. 项目的合并，相同文件夹会自动合并，在开发不同模块的功能时，如AR场景和地图场景可以分开开发，开发完毕后再合并项目。

3. 使用Unity的Resources文件夹加载预制体，不用再一个一个手动拖拽添加了，节省了不少人力。

4. 强力插件的运用，本项目就使用了GoMap插件和Easy Save插件，使用强力的插件能够节约开发时间

5. 静态脚本以及单例的运用。使用静态脚本储存全局变量，可以保存场景切换时的数据，这一点与GameMaker Studio2的global变量如出一辙，并且要利用静态脚本实现保存和读取的功能，本项目使用了插件，只用实现保存和读取功能能即可，在GameMaker Studio2中还需要多两步，就是保存为json文件和读取json文件，整体流程很相似。单例可以直接使用类.单例名来调用该类的函数，功能和静态脚本很相似。
