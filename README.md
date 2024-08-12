# UE5C++-Notes



## 目录

- **基础操作**

- **GAS技能系统**

  > Ability System Component (ASC)
  >
  > Attribute Set (AS)





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
DECLARE_DELEGATE(NoParamDelegate);

// 声明一个委托实例
NoParamDelegate MyDelegate;

//创建委托的函数
void MyDelegate();

// 绑定上面的MyDelegate函数
MyDelegate.BindUObject(this, ADuoActor::MyDelegate);

// 触发委托
MyDelegate.ExecuteIfBound();

void ADuoActor::MyDelegate()
{
//里面写需要执行的操作
    GEngine->AddOnScreenDebugMessage(-1, 5. 0f, FColor: :Red, TEXT("已执行函数”))
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

