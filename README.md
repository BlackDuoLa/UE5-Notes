# UE5C++-Notes



## 目录

- **基础操作**

- **GAS技能系统**

  > Ability System Component (ASC)
  >
  > Attribute Set (AS)

- **网络复制**



## 基础操作

日志打印

UE_LOG(LogTemp,Error,TEXT("okk"));
UE_LOG(LogTemp,Warning,TEXT("okk"));
UE_LOG(LogTemp,Display,TEXT("okk"));

屏幕打印

GEngine->AddOnScreenDebugMessage(-1, 5. 0f, FColor: :Red, TEXT("My Name is ok”))



#### 代理/委托
**单播代理**
描述: 只能绑定一个函数的委托。在某些场景下，你只希望一个函数来响应某个事件，此时可以使用单播委托

| 单播代理 | 返回参数 |
| ------------------ | ---------- |
| DECLARE_DELEGATE(NoParamDelegate) | 无返回参数 |
| DECLARE_DELEGATE_OneParam(OneParamDelegate,int) | 带有一个参数 |
| DECLARE_DELEGATE_TwoParam(TwoParamDelegate,int,int) | 带有两个参数 |
|  DECLARE_DELEGATE_ThreeParam(ThreeParamDelegate,int,int,int) | 带有三个参数 |
| DECLARE_DELEGATE_ReVal(int,ReValParamDelegate) | 带有返回参值 |

```C++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "DuoActor.generated.h"
//无返回参数
DECLARE_DELEGATE(NoParamDelegate);
//带有一个参数
DECLARE_DELEGATE_OneParam(OneParamDelegate,int)
//带有两个参数
DECLARE_DELEGATE_OneParam(OneParamDelegate,int，int)


UCLASS()
class TEXT_API ADuoActor : public AActor
{
	GENERATED_BODY()
public:	
	ADuoActor();
    // 声明一个委托实例(无返回参数)
	NoParamDelegate MyDelegate1;
    //创建委托的函数(无返回参数)
	void MyDelegate();
protected:
	virtual void BeginPlay() override;

};

CPP:
ADuoActor::ADuoActor()
{
    // 绑定上面的MyDelegate1函数(无返回参数)
	MyDelegate1.BindUObject(this,&ADuoActor::MyDelegate);
}
void ADuoActor::BeginPlay()
{
	Super::BeginPlay();
    // 触发委托(无返回参数)
	MyDelegate1.ExecuteIfBound();
}

void ADuoActor::MyDelegate()
{
	GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::Red,TEXT("Add Dele"));
}
```
**多播代理**

**描述**: 可以绑定多个函数的委托。当事件发生时，所有绑定的函数都会被调用。这在需要多个对象响应同一事件时非常有用。

| 多播代理 | 返回参数 |
| ------------------ | ---------- |
| DECLARE_DELEGATE_DELEGATE(NoParamDelegate) | 无返回参数 |
| DECLARE_MULTICAST_DELEGATE_OneParam(OneParamDelegate,int) | 带有一个参数 |
| DECLARE_MULTICAST_DELEGATE_TwoParam(TwoParamDelegate,int,int) | 带有两个参数 |
| DECLARE_MULTICAST_DELEGATE_ThreeParam(ThreeParamDelegate,int,int,int) | 带有三个参数 |
| DECLARE_MULTICAST_DELEGATE_ReVal(int,ReValParamDelegate) | 带有返回参值 |

``` c++
DECLARE_MULTICAST_DELEGATE_OneParam(OneParamDelegate,float)

// 声明一个动态委托实例
OneParamDelegate MyDelegate;

//创建委托的函数
UFUNCTION()
void MyDelegate1(float NewHealth);
UFUNCTION()
void MyDelegate2(float NewMana);
UFUNCTION()
void MyDelegate3(float NewVigor);

// 触发委托，所有绑定的函数都会被调用
OneParamDelegate.AddUObject(this, ADuoActor::MyDelegate1);
OneParamDelegate.AddUObject(this, ADuoActor::MyDelegate2);
OneParamDelegate.AddUObject(this, ADuoActor::MyDelegate3);

//执行多播代理
OneParamDelegate.Broadcast("OneParamDelegate")

//函数里面写对应需要执行的操作
void ADuoActor::MyDelegate1(float NewHealth)
{
	//里面写需要执行的操作
    float Health = 50.f; 
    GEngine->AddOnScreenDebugMessage(-1, 5. 0f, FColor: :Red, TEXT("已执行函数1”))
}
void ADuoActor::MyDelegate2(float NewMana)
{
	//里面写需要执行的操作
    float Mana = 20.f; 
    GEngine->AddOnScreenDebugMessage(-1, 5. 0f, FColor: :Red, TEXT("已执行函数2”))
}
void ADuoActor::MyDelegate3(float NewVigor)
{
	//里面写需要执行的操作
    float Vigor = 30.f; 
    GEngine->AddOnScreenDebugMessage(-1, 5. 0f, FColor: :Red, TEXT("已执行函数2”))
}

```

**动态多播代理**
描述: 支持蓝图绑定的委托，可以在C++和蓝图之间互操作。动态委托主要用于绑定到UFUNCTION函数，特别是在需要序列化的情况下。
| 动态多播代理 | 返回参数 |
| ------------------ | ---------- |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOneParamDelegate,int) | 带有一个参数 |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParam(FTwoParamDelegate,int,int) | 带有两个参数 |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParam(FThreeParamDelegate,int,int,int) | 带有三个参数 |
``` C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOneParamDelegate,float);

// 声明一个委托实例
FOneParamDelegate MyDelegate;

//创建委托的函数
UPROPERTY(BlueprintAssignable)
void MyDelegate();

// 执行动态多播代理，绑定在蓝图中进行
FOneParamDelegate.Broadcast("FOneParamDelegate");

```

## GAS技能系统

### Ability System Component

​	它负责管理和执行Gameplay Abilities（游戏能力）、Gameplay Effects（游戏效果）以及Gameplay Attributes（游戏属性）。

#### 核心功能

​	**管理Gameplay Abilities**：

- `AbilitySystemComponent`可以为角色赋予一组`Gameplay Abilities`，这些能力可以被激活、执行并影响角色的行为。例如，一个角色可能具有跳跃、攻击、治疗等能力，这些能力都由ASC来管理。

​	**应用Gameplay Effects**：

- ASC负责应用和管理Gameplay Effects（GE）。这些效果通常会改变角色的属性，如增加或减少健康值、施加状态效果（如减速、眩晕等）。ASC确保这些效果能够正确地应用到角色上，并处理它们的生命周期。

​	**管理Gameplay Attributes**：

- Gameplay Attributes是ASC管理的属性值，通常与角色的状态相关，例如健康值、法力值、耐力等。ASC可以通过Gameplay Effects来改变这些属性，并触发相应的事件。

​	**处理Gameplay Tags**：

- ASC还可以管理`Gameplay Tags`，这些标签用于表示角色的状态或条件（例如，是否处于眩晕状态，是否拥有某种Buff）。这些标签在能力和效果的应用过程中扮演重要角色。

​	**网络同步**：

- ASC提供了内建的网络同步支持，确保在多人游戏中，能力和效果的应用可以在服务器和客户端之间保持一致。







## Attribute Set(AS)

用于创建的各种属性（生命值、法力值、体力值、攻击力、防御力、饥饿值、饥渴值等）

**属性创建方式**：

```C++
// .h

//创建时需带上的头文件
#include "AbilitySystemComponent.h"

#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)

UCLASS()
class GAME_DUO_API UDuoAttributeSet : public UAttributeSet
{
	GENERATED_BODY()
        
     UDuoAttributeSet();

    //此函数用于对属性的网络同步
    virtual void GetLifetimeReplicatedProps(TArray < FLifetimeProperty>& OutLifetimeProps) const override;
    
	//创建一个名为Health的属性值(生命值)、关联网络同步函数
	UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_Health, Category = "Vital Attributes ")
	FGameplayAttributeData Health;
	//ATTRIBUTE_ACCESSORS的作用就是基于 Health 属性的声明，自动生成该属性的访问器代码。
	ATTRIBUTE_ACCESSORS(UAuraAttributeSet, Health);

	//创建网络同步函数
	UFUNCTION()
	void OnRep_Health(const FGameplayAttributeData& OldHealth) const;

    
// .CPP

    UDuoAttributeSet::UDuoAttributeSet()
{
    //对属性赋值有三种方法：
    //1.利用函数赋值(就是这里)、2.利用数据表赋值、3.利用GE赋值
 
    //对创建的属性进行初始化赋值
    InitHealth(50.f);
}
    
    
    void UDuoAttributeSet::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
       //指定属性如何进行网络同步、Health：同步的类型、COND_None：无条件同步、REPNOTIFY_Always：无论值是否变化，只要进行同步就触发回调
   DOREPLIFETIME_CONDITION_NOTIFY(ULikeAttributeSet, Health, COND_None, REPNOTIFY_Always);
        
}
    
	//当一个属性（如 Health）在服务器上发生变化并同步到客户端时，你通常希望在客户端上执行一些额外的逻辑，比如更新UI、触发动画或其他游戏逻辑。
   //GAMEPLAYATTRIBUTE_REPNOTIFY 宏用于自动处理这些通知逻辑，确保属性值变化时能够正确触发相关回调。
    void ULikeAttributeSet::OnRep_Health(const FGameplayAttributeData& OldHealth) const
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(ULikeAttributeSet, Health, OldHealth);
}
    


```

| 宏 | 描述   |
| ----- | ---- |
| .h   |   |
| #defineATTRIBUTE_ACCESSORS(ClassName, PropertyName)        | 这是一个宏定义，它用于自动生成与游戏属性（Gameplay Attributes）相关的访问器代码。 |
| GAMEPLAYATTRIBUTE_PROPERTY_GETTER(Class Name, Property Name) | 这个宏定义了一个getter函数，用于获取`PropertyName`属性的元信息（UProperty）。 |
| GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName)               | 这个宏定义了一个getter函数，用于获取`PropertyName`属性的值。这是一个简单的访问器，用于读取属性的当前值 |
| GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName)               | 这个宏定义了一个setter函数，用于设置`PropertyName`属性的值。这个函数通常会用来更新属性的数值，并可能触发相应的事件或回调。 |
| GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)              | 这个宏定义了一个初始化函数，用于初始化`PropertyName`属性的值。这通常在对象创建时被调用，以确保属性有一个默认的初始值。 |
| .CPP                                                       |                                                              |
| DOREPLIFETIME_CONDITION_NOTIFY                             | 用于指定类的属性如何进行网络同步。`DOREPLIFETIME`：用于将类的属性添加到需要同步的属性列表中。`CONDITION`：这是同步的条件(Replication Condition)。`NOTIFY`：这是一个通知条件，指定在同步时如何触发回调通知函数 |
| `COND_OwnerOnly`                                           | 仅向拥有该对象的客户端进行同步                               |
| `COND_SkipOwner`                                           | 向所有客户端同步，但跳过拥有该对象的客户端                   |
| `COND_InitialOnly`                                         | 仅在对象首次被创建时进行同步                                 |
| `COND_None`                                                | 无条件同步                                                   |
| `REPNOTIFY_OnChanged` | 仅当属性的值发生变化时才触发通知回调 |
| `REPNOTIFY_Always` | 无论值是否变化，只要进行同步就触发回调 |

## Gameplay Effects(GE)

### GE中Source和Target的定义

​	游戏中任何能使属性发生改变的Actor(玩家、敌人、陷阱），用`Source`和`Target`来判断，改变的是自身的属性还是对方的属性

**举例：**玩家有个技能，可以改变防御属性，设置Source就是改变自身的防御数值，设置Target就是改变敌人的防御数值



#### Source（源）

- **定义**：Source是指触发或应用游戏效果的实体。它可以是玩家、NPC、项目（如武器）、环境元素等任何能够产生效果的对象。
- **用途**：Source用于确定效果的来源，这可能会影响效果的属性（如强度、类型、持续时间等）。例如，不同来源的伤害效果可能有不同的伤害值或效果类型。
- **例子**：假设在一个射击游戏中，玩家（作为Source）使用一把步枪射击敌人。步枪作为玩家持有的武器，也可以被视为Source的一部分，因为它决定了射击的某些特性（如子弹类型、伤害值等）。当子弹击中敌人时，它触发了一个伤害效果，这个效果的Source就是玩家（或更准确地说，是玩家手中的步枪）。

#### Target（目标）

- **定义**：Target是指游戏效果被应用到的实体。它通常是Source的行为或动作的直接接受者。
- **用途**：Target定义了效果影响的具体对象，这决定了哪些属性会受到影响（如生命值、速度、防御力等）。
- **例子**：继续上面的射击游戏例子，当玩家的子弹击中敌人时，敌人就是伤害效果的Target。这个效果将减少敌人的生命值，并可能产生其他效果（如击退、流血等）。

举例：

#### Source（源）的设置

- **定义**：在伤害事件中，Source是指发起攻击或触发效果的实体，即造成伤害的法师玩家。

- 实现方式：

  - 在游戏中，每个玩家（包括法师）都有一个唯一的标识符（如玩家ID）。
  - 当法师玩家使用技能或攻击其他玩家时，游戏引擎会记录这个动作的发起者（即Source），并将其与相应的玩家ID关联起来。
  - 伤害计算和其他效果触发逻辑会根据这个Source来执行。

#### Target（目标）的设置

- **定义**：在伤害事件中，Target是指接收攻击或效果的实体，即被伤害的法师玩家。

- 实现方式：

  - 法师玩家在释放技能或进行攻击时，通常会指定一个或多个目标。
  - 这些目标可以是游戏中的其他玩家、NPC或特定的游戏对象。
  - 游戏引擎会根据玩家的输入（如鼠标点击、键盘按键等）来确定目标（即Target），并将其与相应的玩家ID或对象ID关联起来。
  - 一旦目标被确定，伤害计算和其他效果就会应用到这个Target上。

#### 两个法师玩家相互伤害的场景

- 在这个场景中，每个法师玩家既是Source也是潜在的Target。
- 当法师A攻击法师B时，法师A是Source，法师B是Target。
- 同时，如果法师B也对法师A进行了反击，那么法师B就变成了Source，而法师A则变成了Target。
- 游戏引擎会同时处理这两个方向的伤害事件，并根据每个法师的攻击力、防御力、抗性等属性来计算最终的伤害值。





## 网络复制

### Replicated

`Replicated` 标志用于指示该属性需要在网络游戏中被复制。当属性被标记为 `Replicated` 时，UE4 的网络复制系统会在需要时自动将该属性的值从服务器发送到所有客户端，或者从客户端发送到服务器（取决于复制的方向和设置）。这通常用于同步所有玩家都可以看到或影响的游戏状态，如玩家的位置、生命值、分数等。

### ReplicatedUsing

`ReplicatedUsing` 标志用于指定一个自定义的回调函数，该函数将在属性被网络复制时调用。这允许开发者在属性值改变并且被网络发送之前或之后执行自定义的逻辑。这对于需要基于属性值变化执行额外操作（如播放动画、更新UI等）的情况非常有用。

**注意：**`ReplicatedUsing` 并不改变属性复制的基本行为；它只是在复制过程中提供了一个额外的钩子（hook）来执行自定义代码。如果你只需要简单的属性复制而不需要额外的逻辑，那么只使用 `Replicated` 标志就足够了。

### 网络属性复制相关的宏

| 宏名称                         | 作用                           | 特点                                               |
| ------------------------------ | ------------------------------ | -------------------------------------------------- |
| DOREPLIFETIME                  | 将属性标记为需要网络复制       | 基本复制功能，无额外条件或通知                     |
| DOREPLIFETIME_CONDITION        | 在基本复制的基础上增加条件判断 | 根据条件决定是否复制属性                           |
| DOREPLIFETIME_CONDITION_NOTIFY | 结合条件判断和通知机制         | 根据条件决定是否复制属性，并在复制时执行自定义代码 |



