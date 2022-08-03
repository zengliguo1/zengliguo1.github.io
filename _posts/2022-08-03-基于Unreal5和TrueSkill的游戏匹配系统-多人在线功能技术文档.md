---
title: 基于Unreal5和TrueSkill的游戏匹配系统-多人在线功能技术文档
date: 2022-08-03 11:45:00 +0800
categories: [Unreal5, 多人在线游戏匹配系统]
tags: [多人在线, OnlineSubsystem]

pin: true
author: 
    name: CALL1CE
    link: https://space.bilibili.com/9330604
toc: true
comments: true
math: true
mermaid: true

---

## 一、Creating a Multiplayer Plugin

### 1. Multiplayer Concepts

1. Peer-to-peer连接
   
   [![j3T7Cjjpg](https://s1.ax1x.com/2022/07/03/j3T7Cj.jpg)](https://imgtu.com/i/j3T7Cj)

2. client-server模式：
   
   - listen-server：玩家中一台机器做服务器
   
   - dedicated-server：专门的一台服务器

[![j3To5Qjpg](https://s1.ax1x.com/2022/07/03/j3To5Q.jpg)](https://imgtu.com/i/j3To5Q)

3. unreal engine multiplayer: authoritative client - server 模式：
   
   可以根据配置来选择用哪种模式（监听服务器还是专用服务器模式）

## 2. Testing Multiplayer

#### 2.1 Testing in editor

1. play按钮右侧的三个点中，可以改变number of player

2. Net Mode：
   
   - play standalone 单机模式
   
   - play as listen server，编辑器作为监听服务器
   
   - play as client 会创建一个专用服务器

#### 2.2 setting up lan connection局域网

1. 创建一个新的level：lobby（大厅），让其他玩家通过局域网连接进入（file->new level, save current level）

2. 进入Third Person Character蓝图，创建蓝图，按1键打开新level，option填listen，作为监听服务器，2键执行command命令，输入Open 172.22.26.21

3. 获取自己电脑的ip，打开cmd，输入ipconfig得到ipv4地址：IPv4 地址 . . . . . . . . . . . . : 172.22.26.21

4. 打包出package project，放到新建的Build文件夹中

5. 使用两台机器测试，连接同一个网络

### 3. LAN Connection

#### 3.1 C++创建局域网连接

1. Source -> MultiplyShooter -> MultiplyShooter.h（打错了，不过没关系，这个项目是用来测试多人玩家的）创建几个函数
   
   ```cpp
     //可以在蓝图中创建节点
     UFUNCTION(BlueprintCallable)
     void OpenLobby();
     UFUNCTION(BlueprintCallable)
     void CallOpenLevel(const FString& Address);
     UFUNCTION(BlueprintCallable)
     void CallClientTravel(const FString& Address);
   ```

2. 在cpp文件实现
   
   在官方文档查openlevel函数，加入官方文档中提到的头文件`#include "Kismet/GameplayStatics.h"`
   
   这里面的CallOpenLevel()和CallClientTravel()函数的功能是一样的
   
   ```cpp
   void AMutiplyShooterCharacter::OpenLobby()
   {
     UWorld* World = GetWorld();
     if (World)
     {
         //右键level->get file path : E:/Ue5/MutiplyShooter/Content/ThirdPerson/Maps/Lobby.umap
         World->ServerTravel("/Game/ThirdPerson/Maps/Lobby?listen");
     }
   }
   
   void AMutiplyShooterCharacter::CallOpenLevel(const FString& Address)
   {
     //Address是FString类型，加*就成了C风格的字符串，这个可以隐式地创建FName对象
     UGameplayStatics::OpenLevel(this, *Address);
   }
   //让角色传送到ip地址为address的房间去
   void AMutiplyShooterCharacter::CallClientTravel(const FString& Address)
   {
     //获取本地角色控制器
     APlayerController* PlayerController = GetGameInstance()->GetFirstLocalPlayerController();
     if (PlayerController)
     {
         PlayerController->ClientTravel(Address, ETravelType::TRAVEL_Absolute);
     }
   }
   ```

3. 记得build后再回到编辑器（在vs里如果编译不成功，比如报错Unable to build while Live Coding is active. Exit the editor and game, or press Ctrl+Alt+F11 if iterating on code in the editor or game MutiplyShooter ，就在编辑器里按**CTRL**+**ALT**+**F11**来手动编译，可以编译成功）（或者在Edit -> editor preference中搜索live coding，关掉，就能在vs编译了，而且这样就能打包了，不然用live coding没发打包）

4. 进入角色控制器蓝图
   
   [![j8KFSgjpg](https://s1.ax1x.com/2022/07/03/j8KFSg.jpg)](https://imgtu.com/i/j8KFSg)

5. 打包测试（测试失败可能是ip地址错误的问题）

### 4. Online Subsystem

    为了不输入对方的IP地址就同世界各地的人们一起游戏，需要使用一个服务器服务，比如steam，又为了避免使用不同服务器代码库不同带来的影响，虚幻引擎抽象了一层Online Subsystem，以至于只用写一遍代码，打包成一个插件就可以在配置后连接各种类型服务器的服务，因为虚幻引擎的在线子系统处理了其中的细节。

### 5. Online Sessions

#### 5.1 online subsystem在线子系统

    在线子系统提供了一种访问在线平台服务功能的方式。在线平台，是指像Steam和Xbox Lives等这样的东西。这些平台中的每一个都有自己的一套服务支持，如朋友、成就。设置匹配会话，等等。在线子系统包含一组接口，旨在处理每个平台的这些不同服务。因此无论我们选择哪种服务，都可以用在线子系统来处理我们对这些接口的使用。我们所要做的就是为一个特定的平台配置我们的项目。

#### 5.2 session interface会话接口

    会话接口处理创建、管理和销毁游戏会话。它还处理搜索会话和其他匹配功能。一个会话可以被认为是游戏的一个实例，在服务器上运行，有一系列的属性，并且一个会话可以被公布，以便其他玩家可以找到这个会话并加入进来或者是私人的，所以只有被邀请的人才能加入游戏。

    一个典型的游戏会话的基本寿命是这样的。

1. 首先，你用一组所需的设置来创建会话。

2. 然后你等待其他玩家的加入，并在他们进来的时候注册每个人。

3. 一旦有足够的玩家加入，你就开始会话。

4. 然后每个玩家都在同一个会话中玩游戏。

5. 一旦比赛结束，你就可以结束会话并取消玩家的注册。

6. 然后你可以更新会话，改变比赛的设置。

   会话接口函数：CreateSession(), FindSession(), JoinSession(), StartSession(), DestroySession()

#### 5.3 game plan

    我们的目标是能够在我们游戏的菜单上点击一个按钮。现在，我们先假设我们有两个菜单按钮，Host和Join。

    当点击host时，我们的代码将配置会话设置，然后调用会话接口函数创建会话。一旦完成这些，我们就可以打开大厅关卡，等待其他玩家加入。

    然后当别人开始游戏并点击加入，我们将配置一些搜索设置。组属性将有助于过滤掉我们没有兴趣加入的任何游戏会话。然后我们将调用接口函数查找会话。这将返回一些搜索结果，我们将遍历这些结果并挑选一个有效的会话。一旦我们完成了这一工作，我们就能得到适当的P地址，我们可以用ClientTravel()函数来使用。好了，然后用这个函数去旅行到其他玩家那里，收听服务器，并与他们一起加入大厅关卡。

    现在要把所有这些功能放到它自己的整洁的小类中，用来处理这些会话相关的功能。但我们还不想因为创建所有这些新的类而被淹没。现在，我们将简单地从角色类中访问在线子系统。并在那里调用这些函数，只是为了看看一切是如何运作的。一旦找们对如何便用在线子系统以及它的会话接口功能有了了解。然后创建我们自己的类来处理这些事情，我们将把它设计成可以用于任何我们希望使用它的游戏中。

### 6. Configure For Steam

#### 6.1 creating a new project

1. 创建Third Person项目（C++，starter content），MenuSystem

2. 配置steam

3. edit -> plugins, 搜索online subsystem steam，勾选enable，然后restart

4. 实际启用steam模块，进入vs，Source -> MenuSystem -> MenuSystem.Build.cs，在`PublicDependencyModuleNames`中添加`OnlineSubsystemSteam`和`OnlineSubsystem`
   
   ```csharp
   PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HeadMountedDisplay", "OnlineSubsystemSteam", "OnlineSubsystem" });
   ```

#### 6.2 configure the project for steam

1. 打开项目文件夹->config->defalutEngine.ini，添加内容

2. 去文档中找到 Online Subsystem Steam，里面有需要粘贴的内容
   
   ```ini
   [/Script/Engine.GameEngine]
   +NetDriverDefinitions=(DefName="GameNetDriver",DriverClassName="OnlineSubsystemSteam.SteamNetDriver",DriverClassNameFallback="OnlineSubsystemUtils.IpNetDriver")
   
   [OnlineSubsystem]
   DefaultPlatformService=Steam
   
   [OnlineSubsystemSteam]
   bEnabled=true
   SteamDevAppId=480
   
   ; If using Sessions
   ; bInitServerOnClient=true
   
   [/Script/OnlineSubsystemSteam.SteamNetDriver]
   NetConnectionClassName="OnlineSubsystemSteam.SteamNetConnection"
   ```

3. 关闭vs和项目，到项目文件夹，删除Saved、Intermediate和Binaries文件夹

4. 右键MenuSystem.uproject，generate visual studio project files

5. 双击MenuSystem.uproject，会提示丢失文件，点击yes就会重新生成

6. 接下来回到项目即可

### 7. Accessing the Online Subsystem

1. 进入MenuSystemCharacter.h，在末尾添加关于会话的public部分
   
   ```cpp
   public:
     // 在线对话接口的指针
     // IOnlineSessionPtr OnlineSessionInterface;
     // IOnlineSessionPtr是TSharedPtr<class IOnlineSession, ESPMode::ThreadSafe>的别名
     TSharedPtr<class IOnlineSession, ESPMode::ThreadSafe> OnlineSessionInterface;
   ```

2. 进入MenuSystemCharacter.cpp，到构造函数的底部，添加代码访问在线子系统
   
   ```cpp
   IOnlineSubsystem* OnlineSubsystem = IOnlineSubsystem::Get();
     if (OnlineSubsystem)
     {
         //获取在线会话的接口
         OnlineSessionInterface = OnlineSubsystem->GetSessionInterface();
         //检测是否找到在线子系统
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,//不会清除掉之前打印的内容
                 15.f,//持续15s
                 FColor::Blue,
                 FString::Printf(TEXT("Found subsystem %s"), *OnlineSubsystem->GetSubsystemName().ToString())//GetSubsystemName()得到FName，ToString得到FString，再加个*得到c风格字符串
             );
         }
     }
   ```

3. 前往文档中的IOnlineSubsystem，去找到需要添加的头文件，并加在cpp文件中`#include "OnlineSubsystem.h"`

4. 到文档中搜索 IOnlineSession，将需要的头文件添加在cpp文件中`#include "Interfaces/OnlineSessionInterface.h"`

5. 如果收到报错Unable to delete hot-reload file（但我直接编译成功了，没报错），需要关闭vs和引擎，
   
   - 删除Saved、Intermediate和Binaries文件夹
   
   - 右键MenuSystem.uproject，generate visual studio project files
   
   - 双击MenuSystem.uproject，会提示丢失文件，点击yes就会重新生成

6. 登录steam

7. 在编辑器运行会连接不上，打包后运行就可以连接上

### 8. Creating a Session

#### 8.1 Delegate and callback

1. 在MenuSystemCharacter.h中添加函数（protected）
   
   ```cpp
   protected:
     UFUNCTION(BlueprintCallable)
     void CreateGameSession();
   ```

2. 在MenuSystemCharacter.h中创建一个委托变量(private)
   
   ```cpp
   private:
     FOnCreateSessionCompleteDelegate CreateSessionCompleteDelegate;
   ```
   
   此时会报错：“CreateSessionCompleteDelegate”: 未知重写说明符 MenuSystem
   
   因为FOnCreateSessionCompleteDelegate是一个typedef的名字，这时可以像之前IOnlineSessionPtr一样写它的原本名字，也可以直接把cpp文件中的`#include "Interfaces/OnlineSessionInterface.h"`剪切到头文件去。要注意：这个头文件要放在`#include "MenuSystemCharacter.generated.h"`的上方，这时也可以把`TSharedPtr<class IOnlineSession, ESPMode::ThreadSafe>`换回`IOnlineSessionPtr`了

3. 在MenuSystemCharacter.h的protected下创建回调函数来和刚才创建的委托变量绑定
   
   ```cpp
   void OnCreateSessionComplete(FName SessionName, bool bWasSuccessful);
   ```

4. 薛定谔的编译，刚才编译报错，只是往`#include "Interfaces/OnlineSessionInterface.h"`头文件下加了一个回车，就编译成功了。

#### 8.2 Bind the callback

1. 在构造函数处为委托变量初始化

`AMenuSystemCharacter::AMenuSystemCharacter(): CreateSessionCompleteDelegate(FOnCreateSessionCompleteDelegate::CreateUObject(this, &ThisClass::OnCreateSessionComplete))`

    其中，ThisClass就是这个类名的typedef

#### 8.3 Create a game session

1. 去cpp文件中构建CreateGameSession函数
   
   ```cpp
   void AMenuSystemCharacter::CreateGameSession()
   {
     //当按1时执行回调
     if (!OnlineSessionInterface.IsValid())
     {
         return;
     }
     //检测当前是否有对话
     auto ExitingSession = OnlineSessionInterface->GetNamedSession(NAME_GameSession);
     //如果已经有会话
     if (ExitingSession != nullptr)
     {
         OnlineSessionInterface->DestroySession(NAME_GameSession);
     }
     //将委托加入委托列表
     OnlineSessionInterface->AddOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegate);
     //使用MakeShareable将指针创建为Tshared类型的指针
     TSharedPtr<FOnlineSessionSettings> SessionSettings = MakeShareable(new FOnlineSessionSettings());
      //设置会话设置
      SessionSettings->bIsLANMatch = false;//不是lan连接
      SessionSettings->NumPublicConnections = 4;//可以连接的最大人数
      SessionSettings->bAllowJoinInProgress = true;//允许中途加入
      SessionSettings->bAllowJoinViaPresence = true;//允许不同地区的人加入
      SessionSettings->bShouldAdvertise = true;//steam会显示房间供玩家加入
      SessionSettings->bUsesPresence = true;//显示地区
     //获取角色控制器的指针
     const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
     OnlineSessionInterface->CreateSession(*LocalPlayer->GetPreferredUniqueNetId(), NAME_GameSession, *SessionSettings);
   
   }
   ```
   
   去文档查找FOnlineSessionSettings函数的头文件，并加入cpp文件中。

#### 8.4 Print the session name

1. 去构建回调函数OnCreateSessionComplete()
   
   ```cpp
   void AMenuSystemCharacter::OnCreateSessionComplete(FName SessionName, bool bWasSuccessful)
   {
     if (bWasSuccessful)//如果成功创建会话
     {
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,//不清除之前的debug
                 15.f,
                 FColor::Blue,
                 FString::Printf(TEXT("Created ssesion : %s"), *SessionName.ToString())
             );
         }
     }
     else//如果没有成功创建会话
     {
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,//不清除之前的debug
                 15.f,
                 FColor::Red,
                 FString::Printf(TEXT("Failed to create session!"))
             );
         }
     }
   }
   ```

2. 前往角色蓝图，按1键执行create game session节点

3. falied create原因：UE5.0.2有些许不同，需要把配置文件中的部分改一下：
   
   ```ini
      [OnlineSubsystemSteam]
       bEnabled=true
       SteamDevAppId=480
   
       ; If using Sessions
       ; bInitServerOnClient=true
   ```
   
   改为：
   
   ```ini
   [OnlineSubsystemSteam]
   bEnabled=true
   SteamDevAppId=480
   bInitServerOnClient=true
   ```

### 9. Setup for Joining Game Sessions

#### 9.1 JoinGameSession()

1. 在MenuSystem.h文件中protected下创建一个蓝图节点函数
   
   ```cpp
     UFUNCTION(BlueprintCallable)
     void JoinGameSession();
   ```

2. 去cpp文件中实现函数（会在之后实现）

#### 9.2 Delegate and callback

1. 接下来会像创建会话时一样创建委托和回调，在MenuSystem.h文件中的private下创建join会话的委托变量。
   
   ```cpp
   FOnFindSessionsCompleteDelegate FindSessionsCompleteDelagate;
   ```

2. 在protected下创建回调函数
   
   ```cpp
   void OnFindSessionsComplete(bool bWasSuccessful);
   ```

3. 去cpp文件实现回调

#### 9.3 Binding the callback

1. 在cpp的构造函数处为委托绑定回调
   
   `FindSessionsCompleteDelegate(FOnFindSessionsCompleteDelegate::CreateUObject(this, &ThisClass::OnFindSessionsComplete))`

#### 9.4 Session search settings

1. 实现JoinGameSession函数，（其中SessionSearch在头文件中的private下创建变量`TSharedPtr<FOnlineSessionSearch> SessionSearch`,因为要在回调函数中使用这个变量来获取会话结果数组）
   
   ```cpp
   void AMenuSystemCharacter::JoinGameSession()
   {
     //寻找会话
     //先判断在线会话接口的有效性
     if (!OnlineSessionInterface.IsValid())
     {
         return;
     }
     //将委托变量加入委托列表
     OnlineSessionInterface->AddOnFindSessionsCompleteDelegate_Handle(FindSessionsCompleteDelegate);
     //创建会话设置的智能指针,将类的指针makesharable
     SessionSearch = MakeShareable(new FOnlineSessionSearch());
     SessionSearch->MaxSearchResults = 1000;//最大搜索个数
     SessionSearch->bIsLanQuery = false;//禁止局域网的搜索
     //获取角色控制器
     const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
     //执行在线会话接口的寻找会话的函数
     OnlineSessionInterface->FindSessions(*LocalPlayer->GetPreferredUniqueNetId(), SessionSearch.ToSharedRef());//将ptr转为ref
   
   }
   ```

2. 实现回调函数
   
   ```cpp
   void AMenuSystemCharacter::OnFindSessionsComplete(bool bWasSuccessful)
   {
     //遍历寻找到的会话数组
     for (auto Result : SessionSearch->SearchResults)
     {
         FString Id = Result.GetSessionIdStr();//会话Id
         FString User = Result.Session.OwningUserName;//会话创建者的名字
         //打印出来
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,
                 15.f,
                 FColor::Cyan,
                 FString::Printf(TEXT("Id : %s, User : %s"), *Id, *User)
             );
         }
   
     }
   
   }
   ```

3. 到JoinGameSession函数中，在找之前添加一个SessionSearch的设置
   
   ```cpp
   //确保搜索的 会话询问 设置为 现在存在
     SessionSearch->QuerySettings.Set(SEARCH_PRESENCE, true, EOnlineComparisonOp::Equals);
   ```

4. 去蓝图中添加节点，按2时执行JoinGameSession

5. 由于5.0.2版本问题导致出错，需要在CreateSession函数中添加一条SessionSettings：`SessionSettings->bUseLobbiesIfAvailable = true;`而且要regenerate（可能就是没有重新生成，所以失败了）

6. 打包，用两台机器测试

7. 在joingamesession的函数里，`MakeShareable(new FOnlineSessionSearch());`这处代码的new FOnlineSessionSearch的后面一开始没有加()，也没报错，不知道是不是这里的原因导致找不到会话

### 10. Steam Regions

    之前一直搜索不到人，一直以为是代码哪里出问题了，原来不是代码出问题了，是steam区域的问题，这个搜索只能搜索到同一区域的人，要到steamm的设置中去更改

[![jaMmkQjpg](https://s1.ax1x.com/2022/07/06/jaMmkQ.jpg)](https://imgtu.com/i/jaMmkQ)

### 11. Joining the Session

#### 11.1 Create a lobby level

1. 在Maps文件夹中file->new level，将地板scale放大点，保存为Lobby

2. 在OnCreateSessionComplete函数中添加跳转代码：
   
   ```cpp
     if (bWasSuccessful)//如果成功创建会话
     {
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,//不清除之前的debug
                 15.f,
                 FColor::Blue,
                 FString::Printf(TEXT("Created ssesion : %s"), *SessionName.ToString())
             );
         }
         UWorld* World = GetWorld();
         if (World)
         {
             //E:/Ue5/MenuSystem/Content/ThirdPerson/Maps/Lobby.umap
             World->ServerTravel(FString("/Game/ThirdPerson/Maps/Lobby?listen"));//作为监听服务器
         }
     }
   ```

#### 11.2 Specify a "match type"

    在CreateGameSession()函数中添加一条设置

```cpp
//设置游戏模式
    SessionSettings->Set(FName("MatchType"), FString("FreeForAll"), EOnlineDataAdvertisementType::ViaOnlineServiceAndPing);
```

#### 11.3 Check the match type

    在OnFindSessionsComplete函数的遍历中，获取matchtype，并检查

```cpp
for (auto Result : SessionSearch->SearchResults)
    {
        FString Id = Result.GetSessionIdStr();//会话Id
        FString User = Result.Session.OwningUserName;//会话创建者的名字
        FString MatchType;//比赛类型
        Result.Session.SessionSettings.Get(FName("MatchType"), MatchType);
        //打印出来
        if (GEngine)
        {
            GEngine->AddOnScreenDebugMessage(
                -1,
                15.f,
                FColor::Cyan,
                FString::Printf(TEXT("Id : %s, User : %s"), *Id, *User)
            );
        }
        if (MatchType == FString("FreeForAll"))
        {
            if (GEngine)
            {
                GEngine->AddOnScreenDebugMessage(
                    -1,
                    15.f,
                    FColor::Cyan,
                    FString::Printf(TEXT("Joing Match Type : %s"), *MatchType)
                );
            }
        }
    }
```

#### 11.4 get the ip address

1. 在头文件的private下，创建加入会话完成的委托`FOnJoinSessionCompleteDelegate JoinSessionCompleteDelegate;//加入会话完成的委托`

2. 在头文件的protected下，加入会话完成创建回调函数`void OnJoinSessionComplete(FName SessionName, EOnJoinSessionCompleteResult::Type Result);//加入会话完成的回调函数`

3. 绑定委托`JoinSessionCompleteDelegate(FOnJoinSessionCompleteDelegate::CreateUObject(this, &ThisClass::OnJoinSessionComplete))`

4. 修改OnFindSessionsComplete函数，现在开头加上
   
   ```cpp
     if (!OnlineSessionInterface.IsValid())
     {
         return;
     }
   ```
   
   在模式对的情况下，为接口加入委托，并执行join
   
   ```cpp
   //加到委托列表中去
   OnlineSessionInterface->AddOnJoinSessionCompleteDelegate_Handle(JoinSessionCompleteDelegate);
   //获取角色控制器
   const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
   //加入会话
   OnlineSessionInterface->JoinSession(*LocalPlayer->GetPreferredUniqueNetId(), NAME_GameSession, Result);
   ```

```
5. 实现join的回调函数

```cpp
void AMenuSystemCharacter::OnJoinSessionComplete(FName SessionName, EOnJoinSessionCompleteResult::Type Result)
{
    if (!OnlineSessionInterface.IsValid())
    {
        return;
    }
    FString Address;
    if (OnlineSessionInterface->GetResolvedConnectString(NAME_GameSession, Address))
    {
        if (GEngine)
        {
            GEngine->AddOnScreenDebugMessage(
                -1,
                15.f,
                FColor::Yellow,
                FString::Printf(TEXT("Connect String : %s"), *Address)
            );
        }
    }
}
```

#### 11.5 join the session

    在join的回调函数中添加跳转场景

```cpp
void AMenuSystemCharacter::OnJoinSessionComplete(FName SessionName, EOnJoinSessionCompleteResult::Type Result)
{
    if (!OnlineSessionInterface.IsValid())
    {
        return;
    }
    FString Address;
    if (OnlineSessionInterface->GetResolvedConnectString(NAME_GameSession, Address))
    {
        if (GEngine)
        {
            GEngine->AddOnScreenDebugMessage(
                -1,
                15.f,
                FColor::Yellow,
                FString::Printf(TEXT("Connect String : %s"), *Address)
            );
        }
        //获得角色控制器指针
        APlayerController* PlayerController = GetGameInstance()->GetFirstLocalPlayerController();
        //跳转场景
        if (PlayerController)
        {
            PlayerController->ClientTravel(Address, ETravelType::TRAVEL_Absolute);
        }
    }
}
```

### 12. Creating a Plugin

1. plugins插件和modules模块
   
   [![jagWXqjpg](https://s1.ax1x.com/2022/07/06/jagWXq.jpg)](https://imgtu.com/i/jagWXq)

2. 我们的项目文件uproject其实就是一个module

3. 依赖关系，下层只能依赖同层和上层，比game module能依赖engine module，因为先有engine module才有game module
   
   [![jaRwo8jpg](https://s1.ax1x.com/2022/07/06/jaRwo8.jpg)](https://imgtu.com/i/jaRwo8)

#### 12.1 Create our plugin

1. Edit->Plugins，点击add，选择blank命名MultiplayerSessions，填写descriptor data:A plugin for handling online mutiplayer sessions，最后创建插件

2. 如果不显示插件，可以点content drawer的settings勾选显示插件

3. 编译文件

#### 12.2 Add dependency

1. 找到刚创建的MultiplayerSessions.uplugin文件，添加依赖：
   
   ```
     "Plugins": [
         {
             "Name": "OnlineSubsystem",
             "Enabled": true
         },
         {
             "Name": "OnlineSubsystemSteam",
             "Enabled": true
         }
     ]
   ```

2. 打开MultiplayerSession.Build.cs，添加public依赖模块：
   
   ```csharp
         PublicDependencyModuleNames.AddRange(
             new string[]
             {
                 "Core",
                 "OnlineSubsystem",
                 "OnlineSubsystemSteam"
                 // ... add other public dependencies that you statically link with here ...
             }
             );
   ```

3. 编译文件

### 13. Creating our Own Subsystem

1. 父类的选择：Game instance
   
   [![jwK9qfjpg](https://s1.ax1x.com/2022/07/07/jwK9qf.jpg)](https://imgtu.com/i/jwK9qf)

#### 13.1 Create our own subsystem

1. 创建一个新的c++类，选择ugameinstancesubsystem，选择multiplayerSessions 模块，命名MultiplayerSessionsSubsystem

2. 如果有红色波浪线报错的话，就重新generate一下uproject，要记得将plugins中MultiPlayerSessions文件夹中的二进制文件夹和intermediate文件夹删掉

3. 在MultiplayerSubsystem.h中添加一些头文件和构造函数声明、变量声明
   
   ```cpp
   #pragma once
   
   #include "CoreMinimal.h"
   #include "Subsystems/GameInstanceSubsystem.h"
   #include "Interfaces/OnlineSessionInterface.h"
   
   #include "MultiplayerSubsystem.generated.h"
   
   /**
   * 
   */
   UCLASS()
   class MULTIPLAYERSESSIONS_API UMultiplayerSubsystem : public UGameInstanceSubsystem
   {
     GENERATED_BODY()
   public:
     UMutiplayerSubsystem();
   
   protected:
   
   private:
     IOnlineSessionPtr SessionInterface;
   };
   ```

4. 在MultiplayerSubsystem.cpp中实现构造函数
   
   ```cpp
   #include "MultiplayerSubsystem.h"
   #include "OnlineSubsystem.h"
   UMultiplayerSubsystem::UMultiplayerSubsystem()
   {
     IOnlineSubsystem* Subsystem = IOnlineSubsystem::Get();
     if (Subsystem)
     {
         SessionInterface = Subsystem->GetSessionInterface();
     }
   }
   ```

#### 14. Session Interface Delegates

1. 需要实现的方法：
   
   [![jw8ny9jpg](https://s1.ax1x.com/2022/07/07/jw8ny9.jpg)](https://imgtu.com/i/jw8ny9)

2. delegate handle：当委托完成后，clear掉
   
   [![jw8Zz4jpg](https://s1.ax1x.com/2022/07/07/jw8Zz4.jpg)](https://imgtu.com/i/jw8Zz4)

#### 14.1 Functions

1. 在头文件声明函数
   
   ```cpp
   public:
     UMultiplayerSubsystem();
     //
     //处理会话功能，菜单类将会调用
     //
     void CreateSession(int32 NumPublicConnections, FString MatchType);
     void FindSessions(int32 MaxSearchResults);
     void JoinSession(const FOnlineSessionSearchResult& SessionResult);
     void DestroySession();
     void StartSession();
   ```

#### 14.2 Delegates

1. 在头文件添加委托变量
   
   ```cpp
   private:
     IOnlineSessionPtr SessionInterface;
   
     //
     //要添加在线会话接口的委托列表中的委托
     //将绑定MultiplayerSubsystem内部回调函数到这些委托
     //
     FOnCreateSessionCompleteDelegate CreateSessionCompleteDelegate;
     FOnFindSessionsCompleteDelegate FindSessionsCompleteDelegate;
     FOnJoinSessionCompleteDelegate JoinSessionCompleteDelegate;
     FOnDestroySessionCompleteDelegate DestroySessionCompleteDelegate;
     FOnStartSessionCompleteDelegate StartSessionCompleteDelegate;
   ```

#### 14.3 Callbacks

1. 在头文件中声明回调函数（protected）
   
   ```cpp
   protected:
     //
     //将要添加在 在线会话接口委托列表中委托的 内部回调函数
     //因为不需要在这个类外调用，所以写在protected里
     void OnCreateSessionComplete(FName SessionName, bool bWasSuccessful);
     void OnFindSessionsComplete(bool bWasSuccessful);
     void OnJoinSessionComplete(FName SessionName, EOnJoinSessionCompleteResult::Type Result);
     void OnDestroySessionComplete(FName SessionName, bool bWasSuccessful);
     void OnStartSessionComplete(FName SessionName, bool bWasSuccessful);
   ssful);
   ```

#### 14.4 Bind the callbacks

1. 在cpp文件的构造函数下初始化，绑定委托和回调函数
   
   ```cpp
   UMultiplayerSubsystem::UMultiplayerSubsystem():
   CreateSessionCompleteDelegate(FOnCreateSessionCompleteDelegate::CreateUObject(this, &ThisClass::OnCreateSessionComplete)),
   FindSessionsCompleteDelegate(FOnFindSessionsCompleteDelegate::CreateUObject(this, &ThisClass::OnFindSessionsComplete)),
   JoinSessionCompleteDelegate(FOnJoinSessionCompleteDelegate::CreateUObject(this, &ThisClass::OnJoinSessionComplete)),
   DestroySessionCompleteDelegate(FOnDestroySessionCompleteDelegate::CreateUObject(this, &ThisClass::OnDestroySessionComplete)),
   StartSessionCompleteDelegate(FOnStartSessionCompleteDelegate::CreateUObject(this, &ThisClass::OnStartSessionComplete))
   ```

#### 14.5 Dekegate handles

1. 在头文件中为每一个委托创建delegate handle
   
   ```cpp
     FOnCreateSessionCompleteDelegate CreateSessionCompleteDelegate;
     FDelegateHandle CreateSessionCompleteDelegateHandle;
     FOnFindSessionsCompleteDelegate FindSessionsCompleteDelegate;
     FDelegateHandle FindSessionsCompleteDelegateHandle;
     FOnJoinSessionCompleteDelegate JoinSessionCompleteDelegate;
     FDelegateHandle JoinSessionCompleteDelegateHandle;
     FOnDestroySessionCompleteDelegate DestroySessionCompleteDelegate;
     FDelegateHandle DestroySessionCompleteDelegateHandle;
     FOnStartSessionCompleteDelegate StartSessionCompleteDelegate;
     FDelegateHandle StartSessionCompleteDelegateHandle;
   ```

2. 编译文件

### 15. The Menu Class

#### 15.1 Create a menu class

1. 在MultiplayerSessionsC++Class->MultiplySessions->Public文件夹中创建新的c++类，搜索uuserwidget，选择UserWidget，下一步选择MultiplayerSessions，命名Menu

2. 此时重新加载后，编译无法通过，尝试重新generate(千万不要！！！会打不开项目)

3. 打开MultiplayerSessions.Build.cs文件，在PublicDependencyModuleNames.AddRange中添加引用依赖，解决了报错，再编译一下，就能打开项目了
   
   ```cpp
         PublicDependencyModuleNames.AddRange(
             new string[]
             {
                 "Core",
                 "OnlineSubsystem",
                 "OnlineSubsystemSteam",
                 "UMG",
                 "Slate",
                 "SlateCore"
                 // ... add other public dependencies that you statically link with here ...
             }
             );
   ```

4. 在Menu .h添加函数声明
   
   ```cpp
   public:
     UFUNCTION(BlueprintCallable)
     void MenuSetup();
   ```

5. 在cpp中实现
   
   ```cpp
   void UMenu::MenuSetup()
   {
     AddToViewport();//添加到窗口
     SetVisibility(ESlateVisibility::Visible);//设置为可见
     bIsFocusable = true;
   
     UWorld* World = GetWorld();
     if (World)
     {
         APlayerController* PlayerController = World->GetFirstPlayerController();
         if (PlayerController)
         {
             //设置input的模式为UI模式，并设置其中的属性
             FInputModeUIOnly InputModeData;
             InputModeData.SetWidgetToFocus(TakeWidget());//聚焦这个界面
             InputModeData.SetLockMouseToViewportBehavior(EMouseLockMode::DoNotLock);//设置鼠标不锁定模式
             PlayerController->SetInputMode(InputModeData);//设置输入模式
             PlayerController->SetShowMouseCursor(true);//鼠标光标可见
         }
     }
   }
   ```

#### 15.2 Create a menu widget

1. 在MultiplayerSessions Content文件夹中创建user interface->widget，命名UBP_Menu

2. 进入UBP_Menu的设计模式，创建两个按钮，一个HostButton，一个JoinButton，将锚点设置成底部中心，调整按钮位置和大小

3. 进入蓝图模式，点击class settings，将parent class设置为Menu

4. 进入level blueprint，创建beginPlay节点，指向create widget节点，选择UBP_Menu，指向MenuSetup节点，return value连接target（如果没有menu set节点的话就尝试重启项目）
   
   [![j0CjSOjpg](https://s1.ax1x.com/2022/07/07/j0CjSO.jpg)](https://imgtu.com/i/j0CjSO)

### 16. Accessing our Subsystem

#### 16.1 Button callbacks

1. 到Menu.h文件中为按钮创建变量
   
   ```cpp
   private:
   
     UPROPERTY(meta = (BindWidget))//要和界面的按钮控件绑定，就要加这个，而且变量名要和界面中创建的一致
     class UButton* HostButton;
   
     UPROPERTY(meta = (BindWidget))
     UButton* JoinButton;
   
     UFUNCTION()//因为要使用引擎的click事件或者委托，所以加上这个
     void HostButtonClicked();
   
     UFUNCTION()
     void JoinButtonClicked();
   ```

2. 实现按钮事件（其实就是debug一下先）
   
   ```cpp
   void UMenu::HostButtonClicked()
   {
     if (GEngine)
     {
         GEngine->AddOnScreenDebugMessage(
             -1,
             15.f,
             FColor::Yellow,
             FString(TEXT("Host Button clicked"))
             );
     }
   }
   
   void UMenu::JoinButtonClicked()
   {
     if (GEngine)
     {
         GEngine->AddOnScreenDebugMessage(
             -1,
             15.f,
             FColor::Yellow,
             FString(TEXT("Join Button clicked"))
         );
     }
   }
   ```

3. 接下来将事件绑定到按钮上，要先在头文件重写一下初始化函数
   
   ```cpp
   protected:
   
     virtual bool Initialize() override;
   ```

4. 在cpp文件中添加头文件`#include "Components/Button.h"`

5. 实现initialize函数
   
   ```cpp
   bool UMenu::Initialize()
   {
     if (!Super::Initialize())
     {
         return false;
     }
   
     if (HostButton)
     {
         HostButton->OnClicked.AddDynamic(this, &ThisClass::HostButtonClicked);
     }
     if (JoinButton)
     {
         JoinButton->OnClicked.AddDynamic(this, &ThisClass::JoinButtonClicked);
     }
   
     return true;
   }
   ```

6. 编译并测试(如果没反应，可以尝试下重启项目，再不行就重新generate一下)

#### 16.2 Access our subsystem

1. 在menu.h的private中声明变量，我的类名叫UMultiplayerSubsyste，是因为创建类的时候少打了session，删除类也有点麻烦，所以也就没改名了
   
   ```cpp
   //管理所有在线会话的功能
     class UMultiplayerSubsystem* MultiplayerSessionsSubsystem;
   ```

2. 在cpp文件中，丰富menusetup函数，并添加MutiplayerSubsystem的头文件
   
   ```cpp
     UGameInstance* GameInstance = GetGameInstance();
     if (GameInstance)
     {
         //为之前创建的变量赋值
         MultiplayerSessionsSubsystem = GameInstance->GetSubsystem<UMultiplayerSubsystem>();
   
     }
   ```

3. 修改button的响应函数(具体实现在下一节)
   
   ```cpp
   void UMenu::HostButtonClicked()
   {
     if (GEngine)
     {
         GEngine->AddOnScreenDebugMessage(
             -1,
             15.f,
             FColor::Yellow,
             FString(TEXT("Host Button clicked"))
             );
     }
   
     if (MultiplayerSessionsSubsystem)
     {
         MultiplayerSessionsSubsystem->CreateSession(4, FString("FreeForAll"));
     }
   }
   ```

### 17. Create Session

#### 17.1 Implement CreateSession()

1. 实现MultiplayerSubsystem.cpp的CreateSession函数
   
   ```cpp
   void UMultiplayerSubsystem::CreateSession(int32 NumPublicConnections, FString MatchType)
   {
     if (!SessionInterface.IsValid())
     {
         return;
     }
     //判断是否存在NAME_GameSession的会话
     auto ExistingSession = SessionInterface->GetNamedSession(NAME_GameSession);
     if (ExistingSession != nullptr)
     {
         SessionInterface->DestroySession(NAME_GameSession);
     }
     //将委托加入委托列表，并保存至handle中以便之后消除
     CreateSessionCompleteDelegateHandle = SessionInterface->AddOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegate);
   
     LastSessionSettings = MakeShareable(new FOnlineSessionSettings());
     //如果没连接steam那就是局域网连接，否则不是
     LastSessionSettings->bIsLANMatch = IOnlineSubsystem::Get()->GetSubsystemName() == "NULL" ? true : false;
     LastSessionSettings->NumPublicConnections = NumPublicConnections;
     LastSessionSettings->bAllowJoinInProgress = true;
     LastSessionSettings->bAllowJoinViaPresence = true;
     LastSessionSettings->bShouldAdvertise = true;
     LastSessionSettings->bUsesPresence = true;
     LastSessionSettings->Set(FName("MatchType"), MatchType, EOnlineDataAdvertisementType::ViaOnlineServiceAndPing);//这个应该是指显示出在线服务和ping值
   
     const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
     //如果创建会话失败，那么就消除委托列表中的创建委托
     if (!SessionInterface->CreateSession(*LocalPlayer->GetPreferredUniqueNetId(), NAME_GameSession, *LastSessionSettings))
     {
         SessionInterface->ClearOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegateHandle);
     }
   
   }
   ```

2. 在头文件创建私有变量：`TSharedPtr<FOnlineSessionSettings> LastSessionSettings;`

3. 在cpp文件包含头文件：`#include "OnlineSessionSettings.h"`

#### 17.2 Travel to the lobby

1. 在menu.cpp文件中丰富HostButtonClicked()

```cpp
    if (MultiplayerSessionsSubsystem)
    {
        MultiplayerSessionsSubsystem->CreateSession(4, FString("FreeForAll"));
        UWorld* World = GetWorld();
        if (World)
        {
            World->ServerTravel("/Game/ThirdPerson/Maps/Lobby?listen");
        }
    }
```

2. 编译并测试，不需要打包的测试方法：右键uproject->lauch game

3. 测试失败，没有跳转场景，查了下，发现是ServerTravel里路径参数写错了，还得是去copy path最保险

4. 在menu.h中声明一个私有函数MenuTearDown，用来解除inputmodeUI
   
   ```cpp
   void UMenu::MenuTearDown()
   {
     RemoveFromParent();
   
     UWorld* World = GetWorld();
     if (World)
     {
         APlayerController* PlayerController = World->GetFirstPlayerController();
         if (PlayerController)
         {
             FInputModeGameOnly InputModeData;
             PlayerController->SetInputMode(InputModeData);
             PlayerController->SetShowMouseCursor(false);
         }
     }
   }
   ```

5. 重写Menu类的虚函数OnLevelRemovedFromWorld() (protected)
   
   ```cpp
   void UMenu::OnLevelRemovedFromWorld(ULevel* InLevel, UWorld* InWorld)
   {
     MenuTearDown();
     //调用父级的函数
     Super::OnLevelRemovedFromWorld(InLevel, InWorld);
   }
   ```

#### 17.3 Add Input to MenuSetup

1. 在头文件中添加私有变量并初始化
   
   ```cpp
   int32 NumPublicConnection{ 4 };
   FString MatchType{ TEXT("FreeForAll") };
   ```

2. 修改menuSetup函数（参数也要修改,头文件中要设置默认值），主要就是给变量赋初值
   
   `void MenuSetup(int32 NumberOfPublicConnections = 4 , FString TypeOfMatch = FString(TEXT("FreeForAll")));`
   
   ```cpp
   void UMenu::MenuSetup(int32 NumberOfPublicConnections , FString TypeOfMatch )
   {
     NumPublicConnection = NumberOfPublicConnections;
     MatchType = TypeOfMatch;
   ```

3. 修改HostButtonClicked()函数中，createSession的参数，改成变量
   
   `MultiplayerSessionsSubsystem->CreateSession(NumPublicConnection, MatchType);`

4. 此时打开level bliprint可能会看到menu setup节点没有添加变量，重启下项目即可

### 18. Callbacks to our Subsystem Functions

    使用自定义的委托，在执行完会话创建的委托回调函数后，创建自定义委托来执行menu类的回调函数，以此来使multiplayer session subsystem可以与menu类实现互通

[![j04JRfjpg](https://s1.ax1x.com/2022/07/08/j04JRf.jpg)](https://imgtu.com/i/j04JRf)

#### 18.1 Declare new delegate

1. 在MultiplayerSubsystem.h中声明自定义委托（动态多播委托：多个类的回调函数都可以绑定这个委托，DYNAMIC意味着被序列化，而且可以在蓝图中加载和保存），并创建public变量，声明在类外(可能会报错有红色波浪线，没关系，做完这一节，最后重启项目和vs再编译一遍即可)
   
   ```cpp
   DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMultiplayerOnCreateSessionComplete, bool, bWasSuccessful);
   ```
   
   ```cpp
     //
     //自定义委托变量（用于和menu类的回调函数绑定）
     //
     FMultiplayerOnCreateSessionComplete MultiplayerOnCreateSessionComplete;
   ```

#### 18.2 Create callbacks on the menu

1. 在menu.h中声明回调函数（protected）
   
   ```cpp
     //
     //回调函数（用于MultiplayerSystem的自定义委托）
     //
     void OnCreateSession(bool bWasSuccessful);
   ```

#### 18.3 Bind the callbacks

1. 在MenuSetup函数的底部添加代码
   
   ```cpp
     if (MultiplayerSessionsSubsystem)
     {
         MultiplayerSessionsSubsystem->MultiplayerOnCreateSessionComplete.AddDynamic(this, &ThisClass::OnCreateSession);
     }
   ```

2. 在MultiplayerSubsystem.cpp中的createsession函数中，如果创建函数失败了，那么就广播这个自定义委托（false）
   
   ```cpp
     if (MultiplayerSessionsSubsystem)
     {
         MultiplayerSessionsSubsystem->MultiplayerOnCreateSessionComplete.AddDynamic(this, &ThisClass::OnCreateSession);
     }
   ```

3. 在执行CreateSessionCompleteDelegate委托的回调函数时，自定义委托广播true
   
   ```cpp
   void UMultiplayerSubsystem::OnCreateSessionComplete(FName SessionName, bool bWasSuccessful)
   {
     if (SessionInterface)
     {
         SessionInterface->ClearOnCreateSessionCompleteDelegate_Handle(CreateSessionCompleteDelegateHandle);
     }
   
     MultiplayerOnCreateSessionComplete.Broadcast(bWasSuccessful);
   }
   ```

4. 去实现自定义委托的回调函数
   
   ```cpp
   void UMenu::OnCreateSession(bool bWasSuccessful)
   {
     if (bWasSuccessful)
     {
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,
                 15.f,
                 FColor::Yellow,
                 FString(TEXT("Session created successfully!"))
             );
         }
     }
   }
   ```

5. 此时运行可能没反应，是因为要将menu中的那个回调函数设置成UFUNCTION,才能绑定成功（因为用的时dynamic委托）:
   
   ```cpp
     //
     //回调函数（用于MultiplayerSystem的自定义委托）
     //
     UFUNCTION()
     void OnCreateSession(bool bWasSuccessful);
   ```

6. 将之前的跳转代码从按钮事件剪切到自定义委托的回调函数内
   
   ```cpp
   void UMenu::OnCreateSession(bool bWasSuccessful)
   {
     if (bWasSuccessful)
     {
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,
                 15.f,
                 FColor::Yellow,
                 FString(TEXT("Session created successfully!"))
             );
         }
         //成功创建后跳转场景
         UWorld* World = GetWorld();
         if (World)
         {
             //E:/Ue5/MenuSystem/Content/ThirdPerson/Maps/Lobby.umap
             World->ServerTravel("/Game/ThirdPerson/Maps/Lobby?listen");
         }
     }
     else
     {
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,
                 15.f,
                 FColor::Red,
                 FString(TEXT("Failed to create a session!"))
             );
         }
     }
   }
   ```

7. 编译测试

### 19. More Subsystem Delegates

#### 19.1 More delegates

1. MultiplayerSubsystem.h文件中声明委托，其中有dynamic的意味着可以使用蓝图节点，没有的就不能使用（更深层次的原理不太清除）
   
   ```CPP
   DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMultiplayerOnCreateSessionComplete, bool, bWasSuccessful);
   DECLARE_MULTICAST_DELEGATE_TwoParam(FMultiplayerOnFindSessionsComplete, const TArray<FOnlineSessionSearchResult>& SessionResult, bool bWasSuccessful);
   DECLARE_MULTICAST_DELEGATE_OneParam(FMultiplayerOnJoinSessionComplete, EOnJoinSessionCompleteResult::Type Result);
   DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMultiplayerOnDestroySessionComplete, bool, bWasSuccessful);
   DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMultiplayerOnStartSessionComplete, bool, bWasSuccessful);
   ```

2. 创建委托变量
   
   ```cpp
     //
     //自定义委托变量（用于和menu类的回调函数绑定）
     //
     FMultiplayerOnCreateSessionComplete MultiplayerOnCreateSessionComplete;
     FMultiplayerOnFindSessionsComplete MultiplayerOnFindSessionsComplete;
     FMultiplayerOnJoinSessionComplete MultiplayerOnJoinSessionComplete;
     FMultiplayerOnDestroySessionComplete MultiplayerOnDestroySessionComplete;
     FMultiplayerOnStartSessionComplete MultiplayerOnStartSessionComplete;
   ```

#### 19.2 Menu callbacks

1. 在menu.h声明回调函数
   
   ```cpp
     //
     //回调函数（用于MultiplayerSystem的自定义委托）
     //
     UFUNCTION()
     void OnCreateSession(bool bWasSuccessful);
     void OnFindSessions(const TArray<FOnlineSessionSearchResult>& SessionResult, bool bWasSuccessful);
     void OnJoinSession(EOnJoinSessionCompleteResult::Type Result);
     UFUNCTION()
     void OnDestroySession(bool bWasSuccessful);
     UFUNCTION()
     void OnStartSession(bool bWasSuccessful);
   ```

2. 在menu.cpp中menuSetup函数底部绑定回调
   
   ```cpp
     if (MultiplayerSessionsSubsystem)
     {
         MultiplayerSessionsSubsystem->MultiplayerOnCreateSessionComplete.AddDynamic(this, &ThisClass::OnCreateSession);
         MultiplayerSessionsSubsystem->MultiplayerOnFindSessionsComplete.AddUObject(this, &ThisClass::OnFindSessions);
         MultiplayerSessionsSubsystem->MultiplayerOnJoinSessionComplete.AddUObject(this, &ThisClass::OnJoinSession);
         MultiplayerSessionsSubsystem->MultiplayerOnDestroySessionComplete.AddDynamic(this, &ThisClass::OnDestroySession);
         MultiplayerSessionsSubsystem->MultiplayerOnStartSessionComplete.AddDynamic(this, &ThisClass::OnStartSession);
     }
   ```

3. OnJoinSession函数会报错，是因为缺少头文件，在头文件文件中添加`#include "Interfaces/OnlineSessionInterface.h""`

4. 在cpp文件添加头文件`#include "OnlineSessionSettings.h"`

5. 之后如果突然出现了一些不明不白的红色波浪线，那么重启下项目和vs即可解决

### 20. Join sessions from the menu

#### 20.1 Join session

1. 修改menu.cpp的joinButtonClicked
   
   ```cpp
   void UMenu::JoinButtonClicked()
   {
     if(MultiplayerSessionsSubsystem)
     {
         MultiplayerSessionsSubsystem->FindSessions(10000);
     }
   
   }
   ```

2. 实现multiplayerSubsystem的findsession函数，在头文件创建私有变量``
   
   ```cpp
   void UMultiplayerSubsystem::FindSessions(int32 MaxSearchResults)
   {
     if (!SessionInterface.IsValid())
     {
         return;
     }
     //将委托变量加入委托列表
     FindSessionsCompleteDelegateHandle = SessionInterface->AddOnFindSessionsCompleteDelegate_Handle(FindSessionsCompleteDelegate);
     //创建会话设置的智能指针,将类的指针makesharable
     LastSessionSearch = MakeShareable(new FOnlineSessionSearch());
     LastSessionSearch->MaxSearchResults = MaxSearchResults;//最大搜索个数
     LastSessionSearch->bIsLanQuery = IOnlineSubsystem::Get()->GetSubsystemName() == "NULL" ? true : false;
   
     //确保搜索的 会话询问 设置为 现在存在的
     LastSessionSearch->QuerySettings.Set(SEARCH_PRESENCE, true, EOnlineComparisonOp::Equals);
     //获取角色控制器
     const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
   
     //执行在线会话接口的寻找会话的函数
     if (!SessionInterface->FindSessions(*LocalPlayer->GetPreferredUniqueNetId(), LastSessionSearch.ToSharedRef()))//将ptr转为ref
     {
         SessionInterface->ClearOnCancelFindSessionsCompleteDelegate_Handle(FindSessionsCompleteDelegateHandle);
         //搜索失败，那么广播时传的参数就是空的TArray
         MultiplayerOnFindSessionsComplete.Broadcast(TArray<FOnlineSessionSearchResult>(), false)''
     }
   }
   ```

3. 实现回调`OnFindSessionsComplete(bool bWasSuccessful)`
   
   ```cpp
   void UMultiplayerSubsystem::OnFindSessionsComplete(bool bWasSuccessful)
   {
     if (SessionInterface)
     {
         SessionInterface->ClearOnCreateSessionCompleteDelegate_Handle(FindSessionsCompleteDelegateHandle);
     }
   
     if (LastSessionSearch->SearchResults.Num() <= 0)//没有找到会话，也广播false给menu
     {
         MultiplayerOnFindSessionsComplete.Broadcast(TArray<FOnlineSessionSearchResult>(), false);
         return;
     }
   
     MultiplayerOnFindSessionsComplete.Broadcast(LastSessionSearch->SearchResults , bWasSuccessful);
   }
   ```

#### 20.2 Joing a session

1. 实现menu的回调函数OnJoinSession
   
   ```cpp
   void UMenu::OnFindSessions(const TArray<FOnlineSessionSearchResult>& SessionResult, bool bWasSuccessful)
   {
     //遍历搜索到的所有会话
     for (auto Result : SessionResult)
     {
         FString SettingsValue;
         Result.Session.SessionSettings.Get(FName("MatchType"), SettingsValue);//将会话中设置的比赛类型存到SettingsVale中
         if (SettingsValue == MatchType)//如果搜索到的，和我menu设置的menuType一致
         {
             MultiplayerSessionsSubsystem->JoinSession(Result);
             return;
         }
     }
   }
   ```

2. 去实现MultiplayerSubsystem的JoinSession函数
   
   ```cpp
   void UMultiplayerSubsystem::JoinSession(const FOnlineSessionSearchResult& SessionResult)
   {
     if (!SessionInterface.IsValid())
     {
         MultiplayerOnCreateSessionComplete.Broadcast(EOnJoinSessionCompleteResult::UnknownError);
         return;
     }
   
     JoinSessionCompleteDelegateHandle = SessionInterface->AddOnJoinSessionCompleteDelegate_Handle(JoinSessionCompleteDelegate);
   
     //获取角色控制器
     const ULocalPlayer* LocalPlayer = GetWorld()->GetFirstLocalPlayerFromController();
     if (!SessionInterface->JoinSession(*LocalPlayer->GetPreferredUniqueNetId(), NAME_GameSession, SessionResult))
     {
         SessionInterface->ClearOnJoinSessionCompleteDelegate_Handle(JoinSessionCompleteDelegateHandle);
   
         MultiplayerOnJoinSessionComplete.Broadcast(EOnJoinSessionCompleteResult::UnknownError);
     }
   }
   ```

3. 实现回调函数
   
   ```cpp
   void UMultiplayerSubsystem::OnJoinSessionComplete(FName SessionName, EOnJoinSessionCompleteResult::Type Result)
   {
     if (SessionInterface)
     {
         SessionInterface->ClearOnJoinSessionCompleteDelegate_Handle(JoinSessionCompleteDelegateHandle);
     }
     MultiplayerOnJoinSessionComplete.Broadcast(Result);
   }
   ```

4. 在menu.cpp实现join的回调,并添加头文件`#include "OnlineSubsystem.h"`
   
   ```cpp
   void UMenu::OnJoinSession(EOnJoinSessionCompleteResult::Type Result)
   {
     IOnlineSubsystem* Subsystem = IOnlineSubsystem::Get();
     if (Subsystem)
     {
         IOnlineSessionPtr SessionInterface = Subsystem->GetSessionInterface();
         if (SessionInterface.IsValid())
         {
             FString Address;//保存要join的地址
             SessionInterface->GetResolvedConnectString(NAME_GameSession, Address);
             //获得角色控制器指针
             APlayerController* PlayerController = GetGameInstance()->GetFirstLocalPlayerController();
             //跳转场景
             if (PlayerController)
             {
                 PlayerController->ClientTravel(Address, ETravelType::TRAVEL_Absolute);
             }
         }
     }
   }
   ```

5. 由于找不到Session，应该是在createSession函数里忘了加`LastSessionSettings->bUseLobbiesIfAvailable = true;`这个了

### 21. Tracking Incoming Players

    游戏模式：设定规则，处理角色进入游戏、离开游戏。

    游戏状态：每一个客户端都会监视游戏状态信息，游戏状态是一个类，保存着每个玩家的数据数组

[![jDilVJjpg](https://s1.ax1x.com/2022/07/08/jDilVJ.jpg)](https://imgtu.com/i/jDilVJ)

#### 21.1 Create a game mode

1. 把game mode创建在MenuSystem文件夹，而不是放在插件里，新建C++类，选择GAME MODE BASE,命名LobbyGameMode

#### 21.2 Track incoming players

1. 在头文件的public下重写函数声明
   
   ```cpp
   public:
     virtual void PostLogin(APlayerController* NewPlayer) override;
     virtual void Logout(ACpntroller* Exiting) override;
   ```

2. 实现两个函数，添加头文件`#include "GameFrameWork/GameStateBase.h"`不然会报错
   
   ```cpp
   void ALobbyGameMode::PostLogin(APlayerController* NewPlayer)
   {
     //执行父类的函数
     Super::PostLogin(NewPlayer);
   
     if (GameState)
     {
         int32 NumberOfPlayers = GameState.Get()->PlayerArray.Num();
         if (GEngine)
         {
             //打印玩家数量
             GEngine->AddOnScreenDebugMessage(
                 1,
                 60.f,
                 FColor::Yellow,
                 FString::Printf(TEXT("Players in game: %d"), NumberOfPlayers)
             );
             //打印进入游戏的玩家名字
             APlayerState* PlayerState = NewPlayer->GetPlayerState<APlayerState>();
             if (PlayerState)
             {
                 FString PlayerName = PlayerState->GetPlayerName();//保存玩家名字
                 GEngine->AddOnScreenDebugMessage(
                     -1,
                     60.f,
                     FColor::Cyan,
                     FString::Printf(TEXT("%s has joined the game!"), *PlayerName)
                 );
             }
         }
     }
   }
   
   void ALobbyGameMode::Logout(AController* Exiting)
   {
     Super::Logout(Exiting);
   
     //打印进入游戏的玩家名字
     APlayerState* PlayerState = Exiting->GetPlayerState<APlayerState>();
     if (PlayerState)
     {
         int32 NumberOfPlayers = GameState.Get()->PlayerArray.Num();
         //打印玩家数量
         GEngine->AddOnScreenDebugMessage(
             1,
             60.f,
             FColor::Yellow,
             FString::Printf(TEXT("Players in game: %d"), NumberOfPlayers - 1)
         );
         //打印退出的玩家名字
         FString PlayerName = PlayerState->GetPlayerName();//保存玩家名字
         GEngine->AddOnScreenDebugMessage(
             -1,
             60.f,
             FColor::Cyan,
             FString::Printf(TEXT("%s has exited the game!"), *PlayerName)
         );
     }
   }
   ```

3. 在multiplayerSubsystem类的createSession()中添加一行设置
   
   ```cpp
   LastSessionSettings->BuildUniqueId = 1;//如果不设置的话，就搜索不到其他人创建的房间了
   ```

4. 设置项目最大人数，打开config文件夹中的DefaultGame.ini，添加：
   
   ```ini
   [/Script/Engine.GameSession]
   MaxPlayers=100
   ```

5. 在ThirdPerson文件夹创建一个蓝图，搜索LobbyGameMode，命名BP_LobbyGameMode

6. 打开蓝图，将Default Pawn Class改为BP_ThirdPersonCharacter,编译并保存

7. 进入Lobby场景，将World Settings的GameMode Override改为BP_LobbyGameMode，Default Pawn Class改为BP_ThirdPersonCharacter，只有这样玩家在场景中移动才能传到服务器

### 22. Start Session

### 23. Path to Lobby

1. 在menu头文件创建私有变量`FString PathToLpbby{ TEXT("") };`

2. 给setup函数添加一个形参`void MenuSetup(int32 NumberOfPublicConnections = 4 , FString TypeOfMatch = FString(TEXT("FreeForAll")), FString LobbyPath = FString(TEXT("/Game/ThirdPerson/Maps/Lobby")));`

3. 在setup函数顶部添加代码：
   
   ```CPP
     //给PathToLobby赋值
     PathToLobby = FString::Printf(TEXT("%s?listen"), *LobbyPath);
   ```

4. 修改createsession函数
   
   ```cpp
   World->ServerTravel(PathToLobby);
   ```

5. 进入初始房间level blueprint，可以修改节点的参数了

### 24. Polishing the Menu Subsystem

#### 24.1 Create Session on destroy

    当加入他人会话的玩家退回到主界面时，点击host会显示创建失败，但再次点击就会成功，是因为第一次点击destroy了上一个会话就立马create，但实际上还没来得及destroy完，所以需要重新设计下这个删除功能

1. 在MultiplayerSubsystem.h中声明一些私有变量
   
   ```cpp
     bool bCreateSessionOnDestroy{ false };
     int32 LastNumPublicConnection;
     FString LastMatchType;e;
   ```

2. 修改createSession中销毁会话部分代码
   
   ```cpp
     //如果有会话存在，那么保存此次创建会话的信息，并在之后创建
     if (ExistingSession != nullptr)
     {
         bCreateSessionOnDestroy = true;
         LastNumPublicConnection = NumPublicConnections;
         LastMatchType = MatchType;
   
         DestroySession();
     }
   ```

3. 实现Destroy
   
   ```cpp
   void UMultiplayerSubsystem::DestroySession()
   {
     if (!SessionInterface.IsValid())
     {
         //给menu的回调发送false
         MultiplayerOnDestroySessionComplete.Broadcast(false);
         return;
     }
   
     DestroySessionCompleteDelegateHandle = SessionInterface->AddOnDestroySessionCompleteDelegate_Handle(DestroySessionCompleteDelegate);
   
     if (!SessionInterface->DestroySession(NAME_GameSession))//如果销毁失败
     {
         SessionInterface->ClearOnDestroySessionCompleteDelegate_Handle(DestroySessionCompleteDelegateHandle);//清除句柄
         MultiplayerOnDestroySessionComplete.Broadcast(false);
     }
   }
   ```

4. 实现destroy的回调函数
   
   ```cpp
   void UMultiplayerSubsystem::OnDestroySessionComplete(FName SessionName, bool bWasSuccessful)
   {
     if (SessionInterface)
     {
         SessionInterface->ClearOnDestroySessionCompleteDelegate_Handle(DestroySessionCompleteDelegateHandle);//清除句柄
     }
     if (bWasSuccessful && bCreateSessionOnDestroy)//如果接口的destroy成功且是通过删除再创建新会话
     {
         bCreateSessionOnDestroy = false;//重置标记
         CreateSession(LastNumPublicConnection, LastMatchType);//创建会话，这样其实会多创一次，但这样保证了一定会创建成功一个
     }
     MultiplayerOnDestroySessionComplete.Broadcast(bWasSuccessful);//广播道菜单去
   }
   ```

#### 24.2 Quit Game button

    在右上角创建QuitButton即可，直接给按钮添加点击事件退出游戏即可

#### 24.3 Disable menu button

1. 修改menu.cpp的按钮事件即可，加上这行：`HostButton->SetIsEnabled(false);`

2. 如果创建加入会话失败，那么只需要在回调函数中添加`HostButton->SetIsEnabled(true);`
   
   在find回调中添加：
   
   ```cpp
   if (SessionResult.Num() == 0 || !bWasSuccessful)
     {
         if (GEngine)
         {
             GEngine->AddOnScreenDebugMessage(
                 -1,
                 15.f,
                 FColor::Red,
                 FString(TEXT("Sessions Num : 0! or find error!"))
             );
         }
         JoinButton->SetIsEnabled(true);
         return;
     }
   ```
   
   在join回调底部，如果join失败也`JoinButton->SetIsEnabled(true);`
