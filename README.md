# UE5C++-Notes



## 目录

- **基础操作**

- **GameplayAbilities(GAS)技能系统**

  > Ability System Component (ASC)
  >
  > Attribute Set (AS)
  > GameplayEffect(GE)

- **网络复制**



## 基础操作

### 打印

#### 日志打印

**Log**: 用于普通信息输出。

**Warning**: 用于非致命性问题的警告（用于提示非致命性的问题或潜在的错误，程序仍能继续运行）

**Error**: 用于严重错误的报告（用于记录导致程序无法正常运行的错误，通常需要开发人员立即处理）

**Display**: 用于在终端或控制台显示的重要信息。

**Verbose**和**VeryVerbose**: 用于非常详细和极其详细的信息输出，通常用于深度调试。

````c++
方法一：
int32 Health = 42; 
float Mana = 50.f;
bool MyBool = true;
FString MyText = TEXT("Hello, Unreal!");


//打印自定义文本
UE_LOG(LogTemp,Log,TEXT("ok"));
// 打印整数
UE_LOG(LogTemp, Warning, TEXT("The value of Health is: %d"), Health);
// 打印浮点数
UE_LOG(LogTemp, Error, TEXT("The value of Mana is: %f"), Mana);
// 打印布尔
UE_LOG(LogTemp, Display, TEXT("The value of MyBool is: %s"), MyBool ? TEXT("True") : TEXT("False"));
// 打印文本型
UE_LOG(LogTemp, Verbose, TEXT("The value of MyText is: %d"), *MyText);
````

**整数**使用`%d`格式化符号。

**浮点数**使用`%f`格式化符号

**布尔值**使用`%s`格式化符号，并通过三元运算符将布尔值转换为`"True"`或`"False"`字符串

**文本**使用`%s`格式化符号，并将`FString`转换为C风格字符串（通过`*`操作符）

#### 屏幕打印

````c++
	int32 Health = 42; 
	float Mana = 50.f;
	bool MyBool = true;
	FString MyText = TEXT("Hello, Unreal!");

	//屏幕打印自定义文本
   	GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Blue,TEXT("YES"));
	//屏幕打印整数
   	GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Blue,FString::Printf(TEXT("The value of MyFloat is: %d"), MyFloat));
	//屏幕打印浮点数
	GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Blue,FString::Printf(TEXT("The value of MyFloat is: %f"), MyFloat));
    //屏幕打印布尔
	GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Red, FString::Printf(TEXT("The value of MyBool is: %s"), MyBool ? TEXT("True") : TEXT("False")));
    //屏幕文本型
	GEngine->AddOnScreenDebugMessage(-1,5.0f,FColor::Yellow,FString::Printf(TEXT("The value of MyText is: %s"), *MyText));


方法二：
    //把相应类型，都转换为FString类型
    
   int32 Health = 42;
    // 将整数类型，转换为FString类型
    FString THealth = FString::FromInt(Health);
	//打印转换成FString类型的Health
	GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Blue,THealth);

````

### UPRPERTY(宏) 

`PROPERTY` 是一个宏，用于标记类中的成员变量，以便让这些变量在引擎的编辑器、序列化、复制等机制中得以使用。它主要用在 C++ 类中，使变量能够与引擎的各种系统交互。具体作用包括：

1. **在编辑器中可见**：标记为 `UPROPERTY` 的变量可以在蓝图编辑器中被查看和修改。常用于自定义类的可调属性，使设计师能够在编辑器中配置。
2. **序列化**：标记为 `UPROPERTY` 的变量可以在保存和加载时被序列化，确保对象状态的持久性。
3. **复制**：用于网络同步的变量可以通过 `UPROPERTY` 的标记来支持复制（Replication），使客户端和服务器之间的状态保持一致。
4. **垃圾回收**：UE5 有自己的垃圾回收机制，使用 `UPROPERTY` 标记的对象引用会自动被引擎追踪，从而避免内存泄漏。
5. **反射系统**：`UPROPERTY` 是 Unreal 的反射系统的一部分，它允许代码在运行时检查和操作对象的属性。这在蓝图与 C++ 之间进行数据传递时非常有用。

#### 常见修饰符分类

`UPROPERTY` 在 Unreal Engine 5 中有许多修饰符，可以控制属性的编辑权限、可见性、复制行为等。这里是 `UPROPERTY` 的完整语法及常见的修饰符分类和作用：

1. **编辑器和蓝图可见性**
   - **EditAnywhere**：属性可以在编辑器的任何地方进行编辑。
   - **EditDefaultsOnly**：只能在类默认值中编辑，不能在实例化对象中编辑。
   - **EditInstanceOnly**：只能在实例化对象中编辑，不能在类默认值中编辑。
   - **VisibleAnywhere**：属性在编辑器中可见，但不可编辑。
   - **VisibleDefaultsOnly**：在类默认值中可见，但不可编辑。
   - **VisibleInstanceOnly**：在实例化对象中可见，但不可编辑。
   - **BlueprintReadOnly**：该属性在蓝图中可读，但不可写。
   - **BlueprintReadWrite**：该属性在蓝图中可读可写。
2. **复制与网络相关**
   - **Replicated**：该属性将会在网络上被同步（复制）。
   - **ReplicatedUsing = FunctionName**：定义一个回调函数，当属性被复制时执行。
   - **NotReplicated**：显式声明该属性不进行复制（默认行为）。
   - **RepSkip**：标记该属性在服务器之间的复制时跳过。
3. **序列化控制**
   - **SaveGame**：标记该属性将在存档时被序列化。
   - **Transient**：该属性不会被序列化（不会保存到磁盘或被复制）。
4. **高级属性**
   - **Config**：属性可以从配置文件（INI 文件）中加载。
   - **GlobalConfig**：从全局配置文件加载。
   - **Localize**：将该属性标记为可本地化。
   - **Interp**：该属性可以通过动画系统插值。
   - **DuplicateTransient**：对象在被复制时，该属性不会被复制。
   - **NonPIEDuplicateTransient**：只在游戏模式下复制，而不是在编辑器模式下。
   - **TextExportTransient**：导出时不包含该属性。
   - **Export**：该属性可以通过导出/导入机制进行外部引用。
   - **Instanced**：属性类型为对象，并且实例化时在运行时创建。
   - **NoClear**：防止该属性在编辑器中被清空。
   - **AssetRegistrySearchable**：使该属性可被资产注册表搜索。
5. **内存管理**
   - **Reference**：该属性是一个引用类型。
   - **Pointer**：该属性是一个指针。
   - **WeakObjectPtr**：表示对其他对象的弱引用，防止引用循环。

### UFUNCTION(宏) 

​	`UFUNCTION` 是一个用于声明类成员函数的宏，它的主要作用是使函数能够与引擎的反射系统、蓝图系统、网络同步等机制进行交互。通过使用 `UFUNCTION`，你可以让 C++ 函数在蓝图中调用、实现网络同步、控制编辑器行为等。

1. **蓝图可调用**： 使用 `UFUNCTION` 可以使 C++ 函数在蓝图中调用。这对于将复杂的 C++ 逻辑暴露给设计师或非程序员团队成员来说非常重要。例如：

上面的函数可以在蓝图中被调用，这样你就能在蓝图节点中直接使用它。

2. **网络复制和同步**： `UFUNCTION` 还可以用于网络游戏中，在服务器和客户端之间同步函数调用。例如，服务器端处理某些事件，而客户端同步这些事件：

这个函数只能在服务器上执行，并且通过网络可靠地调用。

3. **蓝图事件**： `UFUNCTION` 允许在蓝图中实现事件或覆盖函数。这对于让蓝图能够扩展和自定义 C++ 类非常有用：

这个函数没有 C++ 实现，而是留给蓝图来实现。

4. **控制台命令**： 通过 `UFUNCTION`，你可以将函数暴露为控制台命令，方便在游戏运行时通过控制台进行调试或调整：

5. **延迟调用**： `UFUNCTION` 还可以用于声明延迟执行的函数，例如需要等待某个时间或条件的函数：

6. **编辑器相关功能**： 你可以使用 `UFUNCTION` 来定义可以在编辑器中运行的函数，这对于工具开发和编辑器自定义非常有用：

#### 常见修饰符分类

 1. **蓝图相关**

- **BlueprintCallable**：该函数可以在蓝图中被调用。
- **BlueprintPure**：表示该函数是纯函数（不会修改对象状态），并且可以在蓝图中被调用。纯函数在调用时不需要执行节点。
- **BlueprintAuthorityOnly**：该函数只能在服务器端（拥有权）蓝图中调用。
- **BlueprintCosmetic**：该函数只能在客户端（无拥有权）蓝图中调用。
- **BlueprintImplementableEvent**：表示该函数没有在 C++ 中实现，而是供蓝图实现。当蓝图实现时，蓝图会覆盖该函数。
- **BlueprintNativeEvent**：该函数可以在蓝图中被覆盖，但 C++ 中也可以提供默认实现。

2. **网络相关**

- **Server**：表示该函数只能在服务器上调用，通常需要配合 `Reliable` 或 `Unreliable` 使用。
- **Client**：表示该函数只能在客户端上调用。
- **NetMulticast**：该函数可以在服务器调用，并广播给所有客户端。
- **Reliable**：该函数调用通过网络时将被可靠传输（确保到达，但可能有延迟）。
- **Unreliable**：该函数调用通过网络时将不可靠传输（不保证到达，但更快）。
- **WithValidation**：用于 `Server` 和 `Client` 函数，表示函数需要验证。你需要实现一个 `_Validate` 函数来验证参数合法性。

 3. **执行与权限**

- **Exec**：允许该函数通过控制台命令调用。函数的第一个参数通常是 `const FString& Cmd`，即输入的命令字符串。
- **AuthorityOnly**：仅能在服务器端调用该函数。
- **Cosmetic**：仅能在客户端调用该函数。

 4. **线程与性能相关**

- **CallInEditor**：该函数可以在编辑器中运行（无需运行游戏）。
- **Latent**：标记该函数为延迟函数（即函数可能不会立即完成，比如等待时间、等待某些条件等）。
- **Category = "CategoryName"**：将函数归类到指定的蓝图节点类别中，方便在蓝图编辑器中找到。

 5. **函数重载控制**

- **Override**：标记该函数覆盖了父类的虚函数。
- **Final**：防止该函数被子类覆盖。
- **Virtual**：声明该函数为虚函数，可以被子类覆盖。

 6. **安全与内存管理**

- **Static**：标记该函数为静态函数（不依赖类实例）。
- **Const**：表示该函数不会修改类的状态（只能调用 `const` 成员函数或访问 `const` 成员变量）。

### 代理/委托

#### 单播代理
描述: 只能绑定一个函数的委托。在某些场景下，你只希望一个函数来响应某个事件，此时可以使用单播委托

| 单播代理 | 返回参数 |
| ------------------ | ---------- |
| DECLARE_DELEGATE(NoParamDelegate) | 无返回参数 |
| DECLARE_DELEGATE_OneParam(OneParamDelegate,int) | 带有一个参数 |
| DECLARE_DELEGATE_TwoParams(TwoParamDelegate,int,float) | 带有两个参数 |
| DECLARE_DELEGATE_ThreeParams(ThreeParamDelegate,int,int,int) | 带有三个参数 |
| DECLARE_DELEGATE_ReVal(int,ReValParamDelegate) | 带有返回参值 |

```C++
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "DuoActor.generated.h"

//无返回参数
DECLARE_DELEGATE(NoParamDelegate);
//带有一个参数
DECLARE_DELEGATE_OneParam(OneParamDelegate, int)
//带有两个参数
DECLARE_DELEGATE_TwoParams(TwoParamDelegate, int,float)


UCLASS()
class TEXT_API ADuoActor : public AActor
{
	GENERATED_BODY()
public:	
	ADuoActor();
    virtual void BeginPlay() override;
    
    
    //声明无返回值单播委托实例
	NoParamDelegate NoMyDelegate;
    // 声明绑定到委托的函数
	void NoFunction1();
    
	//声明一个返回值单播委托实例
	OneParamDelegate OneMyDelegate;
    // 声明绑定到委托的函数
	void OneFunction(int32 Value);
    
	//  声明两个返回值单播委托实例
	TwoParamDelegate TwoMyDelegate;
    // 声明绑定到委托的函数
	void TwoFunction(int32 Value,float Value1);



};

CPP:

void ADuoActor::BeginPlay()
{
    
  // 绑定无返回值的函数NoFunction1到委托
  NoMyDelegate.BindUObject(this, &ADuoActor::NoFunction1);
  // 调用委托
  NoMyDelegate.Execute();  

  // 绑定一个返回值的函数OneFunction到委托
  OneMyDelegate.BindUObject(this, &ADuoActor::OneFunction);
  // 调用委托
  OneMyDelegate.Execute(42);  // 传递参数42
    
  // 绑定两个返回值的函数TwoFunction到委托
  TwoMyDelegate.BindUObject(this, &ADuoActor::TwoFunction);
  // 调用委托
  TwoMyDelegate.Execute(42,61.f);  // 传递参数整数：42，浮点数：61
    
}



//构造函数
void ADuoActor::NoFunction1()
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Blue, TEXT("The No"));
}

void ADuoActor::OneFunction(int32 Value)
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Cyan, FString::Printf(TEXT("The value of MyFloat is: %d"), Value));
}

void ADuoActor::TwoFunction(int32 Value, float Value1)
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Red, FString::Printf(TEXT("The value is: %d,The Value1 is %f"), Value, Value1));
}
```
#### 多播代理

**描述**: 可以绑定多个函数的委托。当事件发生时，所有绑定的函数都会被调用。这在需要多个对象响应同一事件时非常有用。

| 多播代理 | 返回参数 |
| ------------------ | ---------- |
| DECLARE_DELEGATE_DELEGATE(NoParamDelegate) | 无返回参数 |
| DECLARE_MULTICAST_DELEGATE_OneParam(OneParamDelegate,int) | 带有一个参数 |
| DECLARE_MULTICAST_DELEGATE_TwoParams(TwoParamDelegate,int,int) | 带有两个参数 |
| DECLARE_MULTICAST_DELEGATE_ThreeParams(ThreeParamDelegate,int,int,int) | 带有三个参数 |
| DECLARE_MULTICAST_DELEGATE_ReVal(int,ReValParamDelegate) | 带有返回参值 |

``` c++
DECLARE_MULTICAST_DELEGATE(NoParamDelegate);//无返回值
DECLARE_MULTICAST_DELEGATE_OneParam(OneParamDelegate, int32);//一个返回参数
DECLARE_MULTICAST_DELEGATE_TwoParams(TwoParamDelegate, int32, float);//两个返回参数



UCLASS()
class TEXT_API ADuoActor : public AActor
{
	GENERATED_BODY()
public:	
	ADuoActor();
    virtual void BeginPlay() override;
    
    //声明无返回值多播委托实例
	NoParamDelegate NoMulticastDelegate;
    
	// 声明绑定到委托的函数
	void NoFunction1();
	void NoFunction2();


	 //声明无返回值多播委托实例
	OneParamDelegate OneMulticastDelegate;

	// 声明绑定到委托的函数
	void OneFunction1(int32 Value);
	void OneFunction2(int32 Value);
    

 	//声明无返回值多播委托实例
	TwoParamDelegate TwoMulticastDelegate;
    
	// 声明绑定到委托的函数
	void TwoFunction1(int32 Value,float Value1);
	void TwoFunction2(int32 Value,float Value1);

};


CPP:

void ADuoActor::BeginPlay()
{
    // 绑定函数到无返回值的多播委托
    NoMulticastDelegate.AddUObject(this, &ADuoActor::NoFunction1);
    NoMulticastDelegate.AddUObject(this, &ADuoActor::NoFunction2);
    // 调用多播委托
    NoMulticastDelegate.Broadcast();


    // 绑定函数到一个返回值的多播委托
    OneMulticastDelegate.AddUObject(this, &ADuoActor::OneFunction1);
    OneMulticastDelegate.AddUObject(this, &ADuoActor::OneFunction2);
    // 调用多播委托
    OneMulticastDelegate.Broadcast(54);  // 传递参数

    
	// 绑定函数到两个返回值的多播委托
    TwoMulticastDelegate.AddUObject(this, &ADuoActor::TwoFunction1);
    TwoMulticastDelegate.AddUObject(this, &ADuoActor::TwoFunction2);
    TwoMulticastDelegate.Broadcast(10,15.f);// 传递参数
    
}

//构造函数

void ADuoActor::NoFunction1()
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Cyan,TEXT("NoFunction1"));
}

void ADuoActor::NoFunction2()
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Green, TEXT("NoFunction2"));
}

void ADuoActor::OneFunction1(int32 Value)
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Cyan, FString::Printf(TEXT("The OneFunction1 is: %d"), Value));
}

void ADuoActor::OneFunction2(int32 Value)
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Red, FString::Printf(TEXT("The OneFunction2 is: %d"), Value));
}

void ADuoActor::TwoFunction1(int32 Value, float Value1)
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Cyan, FString::Printf(TEXT("The value1 is: %d,The float is:%f"), Value,Value1));
}

void ADuoActor::TwoFunction2(int32 Value, float Value1)
{
    GEngine->AddOnScreenDebugMessage(-1, 5.0f, FColor::Cyan, FString::Printf(TEXT("The value2 is: %d,The float is:%f"), Value,Value1));
}


```

#### 动态多播代理
描述: 支持蓝图绑定的委托，可以在C++和蓝图之间互操作。动态委托主要用于绑定到UFUNCTION函数，特别是在需要序列化的情况下。
| 动态多播代理 | 返回参数 |
| ------------------ | ---------- |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE(FNoParamDelegate); |不带参数 |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOneParamDelegate,int,Value); | 带有一个参数 |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FTwoParamDelegate,int,Value,float,Value1); | 带有两个参数 |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE_ThreeParams(FThreeParamDelegate,int,Value,float,Value1,bool,Value2); | 带有三个参数 |
``` C++
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FNoParamDelegate);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOneParamDelegate, int32, Value);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FTwoParamDelegates, int32, Value, float, Value1);

UCLASS()
class TEXT_API ADuoActor : public AActor
{
	GENERATED_BODY()
public:	
	ADuoActor();
    virtual void BeginPlay() override;


	// 声明不带参数的动态委托实例
	UPROPERTY(BlueprintAssignable)
	FNoParamDelegate NoMulticastDelegate;

	// 声明带一个参数的动态委托实例
	UPROPERTY(BlueprintAssignable)
	FOneParamDelegate OneMulticastDelegate;

	// 声明带两个参数的动态委托实例
	UPROPERTY(BlueprintAssignable)
	FTwoParamDelegates TwoMulticastDelegate;
    
};

CPP

void AMyActor2::BeginPlay()
{
	Super::BeginPlay();

    // 调用多播委托
	NoMulticastDelegate.Broadcast();
	OneMulticastDelegate.Broadcast(54);// 传递参数
	TwoMulticastDelegate.Broadcast(22, 33.f);// 传递参数
    //在蓝图中进行绑定，输出事件
    
}

```

## GAS技能系统

### Ability System Component

​	它负责管理和执行Gameplay Abilities（游戏能力）、Gameplay Effects（游戏效果）以及Gameplay Attributes（游戏属性）。

#### 核心功能

##### 1. **能力（Ability）管理**

- **激活能力** (`ActivateAbility`)：
  - 允许角色激活一个能力（Ability），这通常是一个技能或特殊动作，比如施放法术或使用特殊武器攻击。
- **取消能力** (`CancelAbility`)：
  - 取消一个正在激活的能力，可以在特定条件下或响应某些事件时取消正在进行的能力。
- **确认或否定能力激活** (`ConfirmAbility`) / (`DenyAbility`)：
  - 用于在能力激活的过程中，根据某些条件确认或否定能力的激活。
- **获取能力** (`GetAbility`)：
  - 返回角色已拥有的能力（通常是一个 `GameplayAbility` 的子类），可以通过索引或标签等方式获取。
- **给角色授予能力** (`GiveAbility`)：
  - 给角色授予一个新的能力，可以是永久性的，也可以是临时的。
- **移除能力** (`ClearAbility`)：
  - 从角色身上移除一个能力，通常用于清除临时能力或当角色失去某些条件时。

##### 2. **效果（Gameplay Effect）管理**

- **应用效果** (`ApplyGameplayEffectToTarget`)：
  - 将一个 `GameplayEffect` 应用到指定目标上，这可以是正面效果（如增益Buff）或负面效果（如减益Debuff）。
- **移除效果** (`RemoveActiveGameplayEffect`)：
  - 移除一个已经应用的 `GameplayEffect`，通常在效果到期或条件不再满足时移除。
- **获取当前应用的效果** (`GetActiveGameplayEffect`)：
  - 获取并查询角色当前正在受影响的所有效果，返回一个效果列表或某个特定的效果实例。

##### 3. **属性（Attribute）管理**

- **获取属性值** (`GetNumericAttribute`)：
  - 获取角色当前某个属性的数值（例如生命值、魔法值、耐力值等）。
- **设置属性值** (`SetNumericAttributeBase`)：
  - 设置角色某个属性的基础值，这个操作通常会触发属性的重新计算并更新相关效果。
- **修改属性值** (`AdjustAttributeForMaxChange`)：
  - 根据最大值的变化调整属性值，常用于处理属性上限变化时自动调整当前值。

##### 4. **事件（Event）处理**

- **绑定事件** (`BindToGameplayEvent`)：
  - 将一个事件绑定到能力系统组件上，例如触发某个能力时通知其他系统。
- **触发事件** (`TriggerGameplayEvent`)：
  - 在特定条件下或能力操作中触发一个自定义事件，通常用于在不同系统间通信或协调能力行为。

##### 5. **冷却时间管理**

- **设置冷却时间** (`SetCooldown`)：
  - 设置一个能力的冷却时间，防止其在冷却时间内再次使用。
- **查询冷却时间** (`GetCooldownTimeRemaining`)：
  - 查询当前冷却时间的剩余时间，通常用于UI显示或判断是否可以再次激活能力。

##### 6. **目标数据处理**

- **应用目标数据** (`ApplyGameplayEffectToTarget`)：
  - 使用目标数据（Target Data）将一个效果应用到多个目标上。目标数据是指在游戏中被选择或指定的多个目标，可以是范围内的敌人等。
- **获取目标数据** (`GetTargetData`)：
  - 获取能力或效果的目标数据，用于进一步处理或应用其他效果。

##### 7. **复制和网络同步**

- **网络同步** (`ReplicateAbility`)：
  - 在网络环境下，确保能力的激活、效果应用等操作能够在客户端和服务器之间同步。
- **预测和回滚** (`AbilityPrediction`)：
  - 对能力的执行进行预测和回滚处理，以减少网络延迟对游戏体验的影响。

##### 8. **查询系统**

- **查询能力** (`HasAbility`)：
  - 查询角色是否拥有特定能力。
- **查询效果** (`HasMatchingGameplayTag`)：
  - 查询角色是否受特定的 `GameplayTag` 影响，可以用于判断某种状态（如角色是否在隐身状态）。
- **查询属性** (`GetAttributeSet`)：
  - 获取角色的属性集，通常是一个包含多个属性的集合，用于统一管理角色属性。





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

`FGameplayEffectContextHandle`是作为一个容器或引用，指向或包含与游戏效果应用相关的上下文信息



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



