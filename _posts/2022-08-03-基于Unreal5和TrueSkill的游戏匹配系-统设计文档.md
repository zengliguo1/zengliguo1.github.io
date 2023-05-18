---
title: 基于Unreal5和TrueSkill的游戏匹配系统-设计文档
date: 2022-08-03 11:26:00 +0800
categories: [Unreal5, 多人在线游戏匹配系统]
tags: [多人在线, TrueSkill]

pin: true
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: true
mermaid: true

---

   





<html>
<h2>
    <center>
摘 要
    </center>
</h2>
</html>

    在竞技游戏对局中，往往势均力敌的对决才能带来精彩的比赛。游戏内的匹配系统就是根据玩家既往的战绩，对玩家的竞技水平进行评估，将水平接近的玩家匹配到同一房间内进行游戏比赛。本项目使用Unreal5的Subsystem与Steam服务器搭建多人在线系统，并使用TrueSkill算法中的部分算法对正在匹配的玩家进行分组，组成4V4对战，由于时间的限制与TrueSkill算法的难度，仅利用游戏质量公式实现了匹配功能，并未实现排位功能。

    **关键词**：匹配系统；Unreal5；TrueSkill；SubSystem；Steam。

<html>
<h2>
    <center>
Abstract
    </center>
</h2>
</html>

    In a competitive game match, it is often a close match that brings about a wonderful game. The matching system in the game is to evaluate the player's competitive level based on the player's previous record, and match the players with similar levels to the same room for game competition. This project uses the Subsystem of Unreal5 and the Steam server to build a multiplayer online system, and uses part of the TrueSkill algorithm to group the players being matched to form a 4V4 battle. Due to time constraints and the difficulty of the TrueSkill algorithm, only the game quality formula is used. The matching function is implemented, but the ranking function is not implemented.

    **Keywords**: matchmaker; Unreal5; TrueSkill; SubSystem; Steam.

<html>
<h2>
    <center>
第1章 绪论
    </center>
</h2>
</html>

    基于Unreal5和TrueSkill的游戏匹配系统的完成经历了多个阶段，接下来的设计文档将从评分系统调研、实现过程以及结论展望方面介绍该系统。

### 1.1 引言

    评分系统计算选手竞技水平的评分并排名，在竞技项目中有着重要的应用，例如体育和游戏.首先它对选手竞技水平的提升和下降有统一的度量，帮助评估选手实力和激发选手动力；其次排名能为赛事组织者安排种子选手提供依据，使得比赛安排更加合理；最后，体育爱好者通过排名直观了解选手，增强比赛的观赏性.目前，广泛应用的评分系统有等级分制、Elo评分和TrueSkill评分系统等[1]。

    竞技游戏和体育项目中的技能评分系统具有三个主要功能。第一，它们允许玩家与其他技能水平相似的玩家进行匹配，从而产生有趣、平衡的比赛；第二，可以将评分提供给玩家和感兴趣的观众，从而激发兴趣；第三，评分可以作为比赛资格的标准。随着在线游戏的出现，评分系统的热度急剧增加，因为每天数百万玩家的在线体验质量都会受到评分系统的影响[2]。

    本系统采用的算法源自TrueSkill，完整的TrueSkill包含了排名系统，这一部分涉及到的数学公式数量较多，在短时间内无法完全掌握并实现，因此，此系统仅仅基于TrueSkill算法的部分原理，实现了匹配系统，利用均值和标准差来描述玩家的技能水平，通过计算玩家与房主的比赛质量系数来判断是否加入比赛，在这一步已经完成了初步的匹配，匹配到的都是与房主水平相近的玩家，当房间人数达到8人时，利用随机取样算法，分出两组进行对战，也许这并不是期望平局率最高的方案，但这也许是个能让比赛有趣的方案。

<html>
<h2>
    <center>
第2章 评分系统
    </center>
</h2>
</html>

### 2.1 ELO评分系统

    Elo评分是根据选手过去成绩来计算选手目前相对水平的一种评分系统。Elo评分系统 由美国物理学教授ArpadElo于1959年提出，最初是用于计算国际象棋比赛中的棋手的相对水平。El0评分系统假设棋手赢得比赛的期望结果是选手当前评分差距的函数，选手的Elo评分的更新与实际结果和期望结果的偏差成比例[1]。

    选手战胜选手的概率 ，即预期结果为:

$$
E_A = \frac{\gamma_A}{\gamma_A+\gamma_B}
=\frac{10^{R_A/100}}{10^{R_A/100}+10^{R_B/100}}
=\frac{1}{1+10^{-(R_A-R_B)/400}}
\tag1
$$

$$
E_B=1-E_A\tag2
$$

    式中：RA、RB分别为棋手A和B的Elo评分.当RA=RB时，EA=EB=O.5,即比赛的期望结果为平局.当棋手A领先B的Elo等级分越多时，A战胜B的概率越大，反之亦然。

    比赛结束后，棋手A的Eo评分应更新为:

$$
R_A^\prime=R_A+K(S_A-E_A)\tag3
$$

    其中$S_A$为比赛的实际结果，在双人游戏中，$S_A$规定如下：

    当棋手A赢B时，$S_A$ =1；当比赛平局，$S_A$=0.5；其他情况$S_A$ =0。

    K为调节参数，控制评分更新的幅度.通常，对于新选手采用较高的K值，随比赛增加而逐渐减小K[1]。

### 2.2 Glicko评分系统

    Elo系统的问题在于无法确定选手评分的可信度，而Glicko系统正是针对此进行改进。因此Glicko系统扩展了Elo，将不再是仅计算选手评分（可以视为选手实力的“最佳猜测”），还加入了“评分误差”（RD，ratings deviation），从统计术语的概念来说，RD用于衡量一个评分的不确定度（RD值越高，评分越不可信）。高RD值意味着选手并不频繁地进行对战，或者该选手仅进行了很少次数的对战，而低RD值说明选手会很经常地进行对抗比赛[3]。

### 2.3 TrueSkill评分系统

    Elo评价系统计算简单易用，但主要用于1对1型的对抗结果。如果对战形式是多人多队，或者更复杂的对战形式，Elo评价系统就显得非常局限。另一方面，Elo评价系统涉及多个随机变量（个人能力由某个分布描述和随机对战结果），而随机变量见又存在着明显的依赖关系。显然，这非常适合以贝叶斯概率图模型的观点进行组织建模，在给定了对战结果后给出评价对象能力的后验推断，TrueSkill就提供这样一种解决方案。相较Elo评价系统，TrueSkill的优势在于：

1. 适用于复杂的组队形式，更具一般性。

2. 置于更完善的建模体系，容易扩展。

3. 继承了贝叶斯建模的好的特点，如模型选择等[4]。

    本系统采用了TrueSkill算法中的MatchQuality算法来计算比赛质量（平局系数），范围为(0,1)，系数越接近1，比赛的质量越高，以下是公式：

$$
q_{draw}(\beta^2,\mu_i,\mu_j,\sigma_i,\sigma_j)
:=\sqrt{\frac{2\beta^2}{2\beta^2+\sigma_i^2+\sigma_j^2}}
\cdot exp\left(
-\frac{(\mu_i-\mu_j)^2}{2(2\beta^2+\sigma_i^2+\sigma_j^2)}
\right)
\tag4
$$

<html>
<h2>
    <center>
第3章 系统实现过程
    </center>
</h2>
</html>

### 3.1 多人在线系统

#### 3.1.1 client-server 模式

    client-server模式有两种模式，分别是listen-server和dedicated-server模式，在本系统中，使用了listen-server模式，让参与游戏的一名玩家作为监听服务器，其他7名参与的玩家为客户端。

#### 3.1.2 虚幻引擎的Online Subsystem 在线子系统

    为了不输入对方的IP地址就同世界各地的人们一起游戏，需要使用一个服务器服务，比如steam，又为了避免使用不同服务器代码库不同带来的影响，虚幻引擎抽象了一层Online Subsystem，以至于只用写一遍代码，打包成一个插件就可以在配置后连接各种类型服务器的服务，因为虚幻引擎的在线子系统处理了其中的细节。

    在线子系统提供了一种访问在线平台服务功能的方式。在线平台，是指像Steam和Xbox Lives等这样的东西。这些平台中的每一个都有自己的一套服务支持，如朋友、成就。设置匹配会话，等等。在线子系统包含一组接口，旨在处理每个平台的这些不同服务。因此无论我们选择哪种服务，都可以用在线子系统来处理我们对这些接口的使用。我们所要做的就是为一个特定的平台配置我们的项目。

#### 3.1.3 session interface 会话接口

    会话接口处理创建、管理和销毁游戏会话。它还处理搜索会话和其他匹配功能。一个会话可以被认为是游戏的一个实例，在服务器上运行，有一系列的属性，并且一个会话可以被公布，以便其他玩家可以找到这个会话并加入进来或者是私人的，所以只有被邀请的人才能加入游戏。

    一个典型的游戏会话的基本寿命是这样的。

1. 首先，用一组所需的设置来创建会话。
2. 然后你等待其他玩家的加入，并在他们进来的时候注册每个人。
3. 一旦有足够的玩家加入，就开始会话。
4. 然后每个玩家都在同一个会话中玩游戏。
5. 一旦比赛结束，就可以结束会话并取消玩家的注册。
6. 然后可以更新会话，改变比赛的设置。

    会话接口函数：CreateSession(), FindSession(), JoinSession(), StartSession(), DestroySession()

[![vVR7xe.jpg](https://s1.ax1x.com/2022/08/03/vVR7xe.jpg)](https://imgtu.com/i/vVR7xe)

#### 3.1.4 session interface functions 会话接口函数

    通过使用接口提供的方法，创建委托并绑定回调函数来实现多人联机。

[![vVRqrd.jpg](https://s1.ax1x.com/2022/08/03/vVRqrd.jpg)](https://imgtu.com/i/vVRqrd)

    构造委托： `FOnCreateSessionCompleteDelegate CreateSessionCompleteDelegate;`
创建回调函数：`void OnCreateSessionComplete(FName SessionName, bool bWasSuccessful);`
    绑定委托与回调函数：`CreateSessionCompleteDelegate(FOnCreateSessionCompleteDelegate::CreateUObject(this, &ThisClass::OnCreateSessionComplete))`
    将委托加入委托列表：`CreateSessionCompleteDelegateHandle = SessionInterface->AddOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegate);`
    从委托列表中清除使用后的委托：`SessionInterface->ClearOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegateHandle);`

#### 3.1.5 custom delegate 自定义委托

[![vVRbKH.jpg](https://s1.ax1x.com/2022/08/03/vVRbKH.jpg)](https://imgtu.com/i/vVRbKH)

    使用自定义的委托，在执行完会话创建的委托回调函数后，创建自定义委托来执行menu类的回调函数，以此来使multiplayer session subsystem可以与menu类实现互通。

### 3.2 匹配系统

#### 3.2.1 根据比赛质量算法进行匹配

    公式中有两个变量$\mu$、$\sigma$和一个常量$\beta$参与运算。

    $\mu$代表着均值，也是分布的期望值。$\mu$代表着玩家的技能水平高低。

    $\sigma$代表概率分布的标准差，意味着样本分散距离。$\sigma$代表玩家的不稳定性，$\sigma$越高，玩家的水平就越不稳定。

    $\beta$代表着技能等级间距，也就是当玩家A的$\mu$大于玩家B的$\mu$一个$\beta$的值时，玩家A就有近似80%的胜率，也就意味着玩家A比玩家B高一个等级。

    代码实现：

```cpp
double UMenu::CheckQuality(double Mu1, double Sigma1, double Mu2, double Sigma2)
{
    double BETA = 25 / 6;
    double c = 2 * std::pow(BETA, 2) + std::pow(Sigma1, 2) + std::pow(Sigma2, 2);
    double d = 2 * std::pow(BETA, 2) / c;
    double e = -1 * std::pow(Mu1 - Mu2, 2) / (2 * c);
    return exp(e) * sqrt(d);
}
```

#### 3.2.2 随机取样算法分组

    每当有一个玩家进入大厅房间后，便将玩家加入数组中，当玩家数量到达8时，通过随机取样算法来从8人中随机抽取4人组成TeamA，其余组成TeamB，组完队后传送到游戏房间。

    随机取样算法代码实现：

```cpp
int m = NumberOfPlayers / 2;
srand((unsigned int)time(0));
for (int i = 0; i < NumberOfPlayers; i++)
{
    if (rand() % (NumberOfPlayers - i) < m)//rand()%(n-i)的取值范围是[0, n-i)
    {
        GEngine->AddOnScreenDebugMessage(
            -1,
            60.f,
            FColor::Blue,
            FString::Printf(TEXT("%s in TeamA"), *PlayersName[i])
        );
        m--;
    }
    else
    {
        GEngine->AddOnScreenDebugMessage(
            -1,
            60.f,
            FColor::Red,
            FString::Printf(TEXT("%s in TeamB"), *PlayersName[i])
        );
    }
}
```

<html>
<h2>
    <center>
第4章 系统界面
    </center>
</h2>
</html>

### 4.1 初始关卡

[![vVRXVI.jpg](https://s1.ax1x.com/2022/08/03/vVRXVI.jpg)](https://imgtu.com/i/vVRXVI)

    初始关卡，两个TextBox可以通过键盘输入来修改玩家的$\mu$和$\sigma$用于测试，点击OK即可更新玩家的$\mu$和$\sigma$。点击Host按钮，玩家会作为监听服务器创建一个会话，并跳转到大厅关卡。其他玩家点击Join按钮，便会开始匹配，将搜索到的结果遍历，当自己与房主的比赛质量系数大于预设时，便加入会话并跳转到大厅关卡。

### 4.2 大厅关卡

[![vVRLqA.jpg](https://s1.ax1x.com/2022/08/03/vVRLqA.jpg)](https://imgtu.com/i/vVRLqA)

    此时当有玩家进入大厅时，会在屏幕左侧显示，当玩家数量到达8时，便会通过随机取样算法分组，并跳转到游戏关卡。

### 4.3 游戏关卡

[![vVRjat.jpg](https://s1.ax1x.com/2022/08/03/vVRjat.jpg)](https://imgtu.com/i/vVRjat)

    当玩家为8人时，开启游戏，跳转到游戏关卡，左侧日志中打印了8人分别所属的队伍。

<html>
<h2>
    <center>
第5章 总结与展望
    </center>
</h2>
</html>

### 5.1 总结

    在实现本系统的过程中了解了很多未知领域的知识，过去一年里只做过单机游戏，这次夏令营第一次接触了网络游戏的实现，并且是首次使用Unreal5的C++实现游戏，比Unity和GameMaker Studio2都要更难一些，但Unreal的蓝图对新手来讲还是十分友好的。

    通过这次夏令营，也开阔了视野，TrueSkill算法出乎意料得富有挑战性，并且也很有趣，想要实现完整的排位系统对目前的我来讲还是有一定难度的，但对于明年的我来讲，也许不会再是个令我费解的难题。

### 5.2不足之处及未来展望

    这个系统不足之处实在是太多了，不只是时间短的缘故，Unreal引擎是从五月底才开始接触的，还不算熟悉，这也是进度不快的原因之一。在匹配系统的制作上，仅仅使用了质量系数，没有使用更多的数据来使得匹配更有说服性。

    对匹配系统的未来展望：

    能够充分利用选手的数据，不仅仅是TrueSkill算法中使用的均值与标准差，还应有对局信息，甚至所使用英雄、枪支的数据信息。

    在分组的过程中，使用了随机采样算法，并不能保证最高的平局系数，在对战人数少的情况下还可以考虑暴力遍历，但当人数上升后，可以考虑使用群智能算法来分组，比如遗传算法、免疫算法等。

    尝试解决炸鱼问题、高手匹配问题等。

    将改良版的匹配系统应用到毕业设计中。

<html>
<h2>
    <center>
参考文献
    </center>
</h2>
</html>

[1]吴霖,陈磊,邓超,袁梅宇,江虹.基于TrueSkill模型的围棋棋手排名方法及评估[J].昆明理工大学学报(自然科学版),2013,38(03):47-55.

[2]Herbrich R,Minka T,Graepel T.TrueSkill(TM):A Bayesian skill rating system[C]//In Advances in Neural Information Pro-cessing Systems 20.Scholkopf B,Platt J,Hoffman T.Cambridge:MIT Press,2007:569-576.

[3]GLICKMAN M E.The Glicko system[EB/OL].http://www.glicko.net/glicko.html.

[4]谢白羽. 游戏思考系列03：游戏匹配机制(MMR、ELO、trueskill2、皇家战争、Glicko等，详细讲ELO，其他的简略)[EB/OL].2021[2022-7-12]. 
https://blog.csdn.net/weixin_43679037/article/details/121855591.

[5]Jeff Moser. Computing Your Skill[EB/OL]. 2010[2022-7-12]. http://www.moserware.com/2010/03/computing-your-skill.html.

[6]白婷. TrueSkill 原理及实现[EB/OL]. 2016[2022-7-12]. https://www.cnblogs.com/baiting/p/5591614.html.
