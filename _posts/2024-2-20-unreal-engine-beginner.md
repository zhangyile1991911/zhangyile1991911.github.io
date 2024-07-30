---
title: UnrealEngine勉強日記
author: zhangyile
date: 2024-7-20 11:33:00 +0800
categories: [Unreal Engine]
tags: [Unreal Engine,Development]
comments: false
img_path: /assets/img/
image:
  path: UnrealEngine/unrealenginetitle.jpg
  lqip: data:image/webp;base64,UklGRpoAAABXRUJQVlA4WAoAAAAQAAAADwAABwAAQUxQSDIAAAARL0AmbZurmr57yyIiqE8oiG0bejIYEQTgqiDA9vqnsUSI6H+oAERp2HZ65qP/VIAWAFZQOCBCAAAA8AEAnQEqEAAIAAVAfCWkAALp8sF8rgRgAP7o9FDvMCkMde9PK7euH5M1m6VWoDXf2FkP3BqV0ZYbO6NA/VFIAAAA
  alt: Responsive rendering of Chirpy theme on multiple devices.
---

### 02-24
1 今天看了Unreal Engine的youtube官方入门教程
Unreal Engine 5 Beginner Tutorial - UE5 Starter Course
https://www.youtube.com/watch?v=k-zMkzmduqI&t=6412s
看到190分钟

### 02-25
Unreal Engine 5 Beginner Tutorial - UE5 Starter Course
https://www.youtube.com/watch?v=k-zMkzmduqI&t=6412s
看到240分钟 后面基本都是场景搭建相关，和我现在需要掌握的知识相关性不大

### 02-26
接下来准备看
https://www.youtube.com/watch?v=dS5AUaYFcdw
Getting into C++ with Unreal Engine
目标看完part1-3

### 02-27
https://www.youtube.com/watch?v=wJo5tusarvY&t=1627s
看到Part5
学习了
如果提供给蓝图的函数
BlueprintPure 有返回值 引き出しがある
BlueprintCallable 没返回值　引き出しがない
DECLARE_DYNAMIC_MULTICAST_DELEGATE_XXXParams
提供监听

### 02-28
向屏幕上打印调试信息
```
GEngine->AddOnScreenDebugMessage(-1, 0.49f, FColor::Silver,*(FString::Printf(
TEXT("Movement - IsCrouched:%d | IsSprinting:%d"), bIsCrouched, bIsRunning)));
DECLARE_DYNAMIC_MULTICAST_DELEGATE_
```
通过宏生成监听class
内部通过TArray管理监听函数，所以按照先进先调用顺序
继承顺序
UOBJECT->AActor->APawn->ACharacter->ACharacterBB
启动游戏时候顺序　
スタートの流れ
```
UWorld::SpawnPlayActor()
|
AGameModeBase::HandleStartingNewPlayer()
|
AGameModeBase::RestartPlayer()
|
AGameModeBase::RestartPlayerAtPlayerStart()
|
AGameModeBase::FinishRestartPlayer()
|
AController::Possess()
```

### 02-29
https://www.youtube.com/watch?v=ErVcuQXTm_c
今天继续看UI初级实现
UE_LOGFMT

转换enum，重载++和--操作符
```
UENUM(BlueprintType)
enum class EHudViewMode: uint8
{
    CleanAndPristine UMETA(Tooltip="Get that mess outta my face!"),
    Minimal UMETA(Tooltip="Just the facts, maam."),
    Moderate UMETA(Tooltip="Keep me well informed"),
    SensoryOverload UMETA(Tooltip="My other UI is a derivatives trading screen")
};

inline EHudViewMode& operator++(EHudViewMode& ViewMode)
{   
    if (ViewMode == EHudViewMode::SensoryOverload)
       ViewMode = EHudViewMode::CleanAndPristine;
    else
       ViewMode = static_cast<EHudViewMode>(static_cast<int>(ViewMode) + 1);

    return ViewMode;
}

inline EHudViewMode& operator--(EHudViewMode& ViewMode)
{
    if (ViewMode == EHudViewMode::CleanAndPristine)
       ViewMode = EHudViewMode::SensoryOverload;
    else
       ViewMode = static_cast<EHudViewMode>(static_cast<int>(ViewMode) - 1);
    return ViewMode;
}
```

### 03-05
使用Interp将变量暴露给Animations中使用

```
UPROPERTY(Interp,Category="Animations")
bool isMoving = false;
```


### 03-06
C++层面 提供 抽象基类
然后基于这个抽象基类 生成blueprint
UIを作る前にC＋＋で親クラスを作ろう
この親クラスを継承してblueprintでUIクラスを生成します
一旦直接blueprintでUIクラス生成したら親クラスを調整できなくなります


### 03-09

```
//绑定Animation需要将属性标记为 Transient (临时性),因为这个属性没有必要存到硬盘上,在runtime的时候绑定
UPROPERTY(Transient,meta = (BindWidgetAnim))
TObjectPtr<UWidgetAnimation> SliderIn;


打印FName类型的log
UE_LOG(LogTemp,Log,TEXT("UInventoryWidget::AddItem %s"),*item.PickupText.ToString());
```

### 03-22

UStaticMeshComponent用于在场景中显示单个静态网格。这是最基本的组件之一，用于添加单一的、不重复的网格到场景中，如游戏中的一个独特物体或角色。
- 单一实体：每个UStaticMeshComponent代表场景中的一个实体，有自己的变换（位置、旋转和缩放）。
- 独立渲染：每个组件独立渲染，与场景中的其他UStaticMeshComponent无关。
- 灵活性：非常灵活，适用于需要独立移动或具有不同物理属性（如碰撞、材质等）的对象。

UHierarchicalInstancedStaticMeshComponent（HISMC）是用于高效渲染大量相同网格的组件。通过实例化技术，可以大幅减少绘制大量重复物体时的性能开销。
- 高效的批处理：可以在单个渲染调用中绘制许多实例，这在渲染大量相同对象时非常高效，如森林、草地或城市景观中的建筑物。
- 共享网格和材质：所有实例共享相同的网格和材质数据，但每个实例可以有自己的变换（位置、旋转和缩放）。
- 层次化实例：使用一种层次化的方法来管理实例，这可以进一步优化大量实例的渲染，特别是当某些实例不在摄像机视野内时。



### 03-28
打印debug堆栈
```
ENGINE_API virtual int32 GetFunctionCallspace(UFunction* Function, FFrame* Stack) override;
```

### 04-03
Border组件可以通过 Content Color and Opacity来控制内部的所有字体颜色

![Desktop View](UnrealEngine/border.jpg)


### 04-08
如果需要基于cpp的struct建立DataTable的话
struct必须继承于 FTableRowBase
类型必须是BlueprintType

```
USTRUCT(BlueprintType)
struct FLevelUpData : public FTableRowBase
{
GENERATED_USTRUCT_BODY()

public:

FLevelUpData()
: XPtoLvl(0)
, AdditionalHP(0)
{}

/** The 'Name' column is the same as the XP Level */

/** XP to get to the given level from the previous level */
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=LevelUp)
int32 XPtoLvl;

/** Extra HitPoints gained at this level */
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=LevelUp)
int32 AdditionalHP;

/** 使用
TSoftObjectPtr或
TAssetPtr时，资产是按需加载的，这意味着你可能需要显式加载它，特别是在运行时 */
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category=LevelUp)
TSoftObjectPtr<UTexture> AchievementIcon;
};
```


### 04-16
在cpp OnConstruct == blueprint Construct Script
每次在Editor中改变参数就会调用

### 04-18

```
MoveTemp()
/*
在 Unreal Engine 和 C++ 中，
MoveTemp 是一个用于强制将对象转换为右值引用的模板函数。它是对 
std::move 的封装，用于支持移动语义，允许资源从一个对象“转移”到另一个对象而不进行复制。
MoveTemp 是 Unreal Engine 的 C++ 标准库的一部分，用于优化性能，特别是当处理大型数据结构时。*/
```


### 04-20

在 Unreal Editor 中操作时，构造函数可能会因多种编辑器操作而被调用，例如：
- 场景保存和加载时。
- 在编辑器中进行复制和粘贴操作。
- 调整相关属性或在关卡编辑器中移动 Actor。


### 04-21
UInstancedStaticMeshComponent 底层是连续的数组
連続配列でインスタンスを格納する



### 04-22
在UI的Editor界面修改值之后的回调函数

```
virtual void PostEditChangeProperty(FPropertyChangedEvent& PropertyChangedEvent) override;

void UMyButtonAction::PostEditChangeProperty(FPropertyChangedEvent& PropertyChangedEvent)
{
    Super::PostEditChangeProperty(PropertyChangedEvent);

    if(PropertyChangedEvent.GetPropertyName() == GET_MEMBER_NAME_CHECKED(UMyButtonAction,ButtonText))
    {
       if(myText)
       {
          myText->SetText(ButtonText);
       }
    }
}
```

### 05-04
使用AnimBlueprint设置好相关动画逻辑，有点类似于Unity中的Animator和Animation然后在cpp层中引用

```
AMyCharacter::AMyCharacter()
{
    // 注意替换路径为你的动画蓝图实际路径
    static ConstructorHelpers::FClassFinder<UAnimInstance> AnimBP(TEXT("/Game/Path/To/MyCharacterAnimBP.MyCharacterAnimBP_C"));
    if (AnimBP.Succeeded())
    {
        AnimBPClass = AnimBP.Class;
    }
}

// 确保你的角色有一个骨架网格组件
Mesh->SetAnimInstanceClass(AnimBPClass);

UAnimInstance* AnimInstance = Mesh->GetAnimInstance();
if (AnimInstance)
{
    AnimInstance->SetBoolParameter(FName("IsRunning"), true);
    AnimInstance->SetFloatParameter(FName("Speed"), 600.0f);
}
```


### 05-05

```
//带Component是一种资源类型
//Componentをつけてるタイプはアセットです。
//不带Component后缀的是一种组件,可以附加在AActor上
//Componentをつけていないタイプはコンポーネントです。
USkeletalMeshComponent
USkeletalMesh
UStaticMeshComponent
UStaticMesh
```


### 05-07
UClass继承自UStruct。需要注意的是，
UStruct只包含成员变量的反射信息，其支持高速的构造和析构；
而UClass则更加重量级，其需要对成员函数进行反射。
在UClass中有成员变量FuncMap，其用于存储当前UClass包含的成员函数信息


### 05-18
Async
- TaskGraph: 适用于需要与其他任务协调和并行执行的任务。
- Thread: 用于需要完全独立运行的任务，适合长时间运行的独立任务。
- ThreadPool: 适用于大量短时间任务的并行执行，通过使用线程池提高性能。
- LargeThreadPool: 适用于需要更多线程资源的大量并行任务执行。
- BackgroundTask: 用于对性能影响较小的任务，避免阻塞主线程。


### 05-22

//创建子Actor可以传一个回调函数进去,因为创建子Actor是非同步的
```
ChildActorComponent->CreateChildActor([](AActor* Acotr)->{
    
}});
```

### 05-30

cppの考え方を踏襲するな
在UE中 接口使用 不是靠Cast转换！！！！

```
UINTERFACE(MinimalAPI)
class UIMyUnitAnimation : public UInterface
{
    GENERATED_BODY()
};

/**
 * 
 */
class TBS20140401_API IIMyUnitAnimation
{
    GENERATED_BODY()

    // Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
    UFUNCTION(BlueprintImplementableEvent,BlueprintCallable)
    void SetUnitAnimationState(EUnitAnimation NewState);
};

//我在blueprint中实现了IIMyUnitAnimation接口,在cpp中调用
//以下是错误示范
auto inst = MySkeletalMeshComponent->GetAnimInstance();
auto myinter = Cast<IIMyUnitAnimation>(inst);//这个转换结果肯定是nullptr
myinter->Execute_SetUnitAnimationState(inst,EUnitAnimation::Walk);    

//以下是正确示范
MyAnimInstance = MySkeletalMeshComponent->GetAnimInstance();
UObject* AnimInstanceObject = Cast<UObject>(MyAnimInstance);
if (AnimInstanceObject->GetClass()->ImplementsInterface(UIMyUnitAnimation::StaticClass()))
{
   UE_LOG(LogTemp,Log,TEXT("ImplementsInterface true"))
   IIMyUnitAnimation::Execute_SetUnitAnimationState(MyAnimInstance,EUnitAnimation::Walk);
}

```

### 06-15

在 ActorComponent蓝图中 无法使用timeline组件
因为Timeline其实也是个ActorComponent
所以在Actor中可以使用。


### 7-11
没法准备控制Beginplay和Tick的调用
如果要保证前后执行顺序的话 最好自己去生成Actor，而不是直接把Actor放在Scene中
BeginplayとTickの実行順番を制御できませんから、
もしこの2つ関数の実行順番を固めたいなら　自分がコードで生成しといきます。


### 7-15
GameUserSetting配置 限制窗口大小

```
[/Script/Engine.GameUserSettings]
bUseVSync=False
ResolutionSizeX=800
ResolutionSizeY=600
LastUserConfirmedResolutionSizeX=800
LastUserConfirmedResolutionSizeY=600
WindowPosX=-1
WindowPosY=-1
FullscreenMode=2
LastConfirmedFullscreenMode=2
Version=5
```


### 7-16

首先在编辑器下cook content，然后使用Visual Studio，以Development或者Debug Game模式启动，（注意不是Development Editor /Debug Game Editor)

简单地说就是ue打包之后的游戏也是可以调试的。但是首先游戏的资产要先cook结束，才能被打包后的游戏在调试状态下正确加载。
在那之后，你只需要把你的工程以debug game形式（无代码优化）或者development形式（代码有优化）在vs里面启动，就会自动进入你看到的那个状态。

### 7-17
Beginplay() 呼び出しタイミングは固まってないから　他の仕組みで制御したほうがいいと思うんですけど。
例えばBeginplayを放棄して　自分がmodule生成順番を管理します今はプロジェクトが終わる段階だから


### 7-19
遇到问题 输入回车键 没有正确跳转到场景 开始游戏
调查思路
1. 确认 回车键 对应代码是否有被正确执行 -> 通过log确认
2. 重新打包完成 确定有log输出 说明按键有被正确回调
3. 查看 加载地图信息
4. 地图文件 没有被正确打包 加载错误
5. 发现json文件没有被正确打包
6. google 搜索 unreal engine how to package json file

#### 障害が出てきた時の考え方
- 障害一覧
エンターキーを押すと正しく画面が遷移しなかった。
- 調査順番
1. Packageの設置の問題ですか　キーボード正しく捕獲出来ますか
2. input処理部分のコードにLogを追加
3. Logが正しく出力した、エンターキーが対応するコードを実行、この部分問題なし
4. 地図のデータを読み込むことを確認
5. Logを通じてデータを読み込まなかったことを確認
6. データのファイルが存在ことを確認
7. パッケージ際このデータのファイルが含まれてない
8. googleで英語を用いて調査「unreal engine how to package json file」  
9. パッケージの項目を設置する
![Desktop View](UnrealEngine/packagesetting.jpg)