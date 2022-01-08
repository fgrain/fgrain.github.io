---
title: UnityEvent和事件系统
date: 2021-03-14 23:27:29
tags:
	- Unity
	- UnityEvent
---

本文参考：
* https://blog.csdn.net/qq_28849871/article/details/78366236
* https://blog.csdn.net/soul900524/article/details/79173354

## UnityEvent和C# Event

UnityEvent是基于Unity引擎对C# Event特性进行了一些更适合开发游戏的改变。了解Event特性要从Delegate(委托)说起

### Delegate（委托）

C#中提供了一种可以存放方法的容器，也就是委托。一般来说，变量是程序在内存中为数据开辟的一块空间，变量可以存放某一具体数值，或某个对象的引用。C#则让Delegate可以存放函数。

<!-- more-->

使用Delegate一般分为三步：

1. 定义一种委托类型
2. 声明一个该类型的委托变量
3. 通过声明的委托调用函数执行相关操作

下面是在Unity中使用Delegate的一个实例

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DelegateExp : MonoBehaviour
{
    //1.为的delegate定义一种函数原型
    public delegate void MyDelegate(int args);
    //2.声明一个delegate变量
    public MyDelegate myDelegate;

    private void Start()
    {
        //3.引用delegate变量实现的函数
        myDelegate = PrintNum;
        myDelegate(5);

        myDelegate = PrintDoubleNun;
        myDelegate(5);
    }

    public void PrintNum(int num)
    {
        print("Print number:" + num);
    }
    
    public void PrintDoubleNun(int num)
    {
        print("Print double number:" + num * 2);
    }
}
```

将脚本挂到一个游戏物体上运行Unity

![image-20210315000332464](UnityEvent%E5%92%8C%E4%BA%8B%E4%BB%B6%E7%B3%BB%E7%BB%9F/image-20210315000332464.png)

通过delegate的变量我们执行的两种函数。delegate定义了一个用于存放相同函数原型（相同参数类型，相同返回值）的容器。因为他们的函数原型相同，所以向delegate传递一次参数就可以让所有添加到delegate上的函数正常执行。

在上述例子中在上述例子中，我们第二次向myDelegate赋值时覆盖掉了第一次赋值的函数，所以第二次引用myDelegate只会调用PrintDoubleNum(int num)函数。实际上，delegate作为函数容器，并不仅仅只能容纳一个函数，而是可以同时被委任多个函数。例如，当你把上述代码中的第二次赋值改为 myDelegate += PrintDoubleNum; 就可以实现同时打印两条语句的效果。这种delegate一般被称为multicast delegate（多播委托）。

### 基于delegate实现的Event

两者区别：

1. event是特殊的delegate，用event实现的功能用delegate同样可以实现。
2. event较之delegate具有继承方面的安全性。
3. 用event，别的类只能订阅/取消订阅，如果用一个 public delegate成员变量，别的类可以调用或者覆盖我们的delegate变量。
4. 一般来说，如果你要创建一个包含多个类的动态体系，使用event而不是delegate。

Event在multicast delegate（多播委托）的基础上演变而来，关于Event，比较形象的比喻就是广播者和订阅者。event执行时会广播给订阅他函数，告诉每个函数该运行了，但不管函数的实现细节。

> **Idol.cs**

```c#
using UnityEngine;

public class Idol : MonoBehaviour {
    public delegate void IdolBehaviour(string behaviour);
    public static event IdolBehaviour IdolDoSomethingHandler;

    private void Start()
    {
        //Idol 决定搞事了, 如果他还有粉丝的话, 就必须全部都通知到
        if (IdolDoSomethingHandler != null)
        {
            IdolDoSomethingHandler("Idol give up writing.");
        }
    }
}
```

> **SubscriberA.cs**

```c#
using UnityEngine;

public class SubscriberA : MonoBehaviour {
    /// <summary>
    /// OnEnable在该脚本被启用时调用,你可以把它看做路转粉的开端
    /// </summary>
    private void OnEnable()
    {
        //粉丝通过订阅偶像来获取偶像的咨询, 并在得到讯息后执行相应的动作
        Idol.IdolDoSomethingHandler += LikeIdol;
    }

    /// <summary>
    /// OnEnable在该脚本被禁用时调用,你可以把它看做粉转路的开端
    /// </summary>
    private void OnDisable()
    {
        Idol.IdolDoSomethingHandler -= LikeIdol;
    }

    /// <summary>
    /// 粉丝A是一个脑残粉
    /// </summary>
    /// <param name="idolAction"></param>
    public void LikeIdol(string idolAction)
    {
        print(idolAction + " I will support you forever!");
    }
}
```

> **SubscriberB.cs**

```c#
using UnityEngine;

public class SubscriberB : MonoBehaviour {
    /// <summary>
    /// OnEnable在该脚本被启用时调用,你可以把它看做路转粉的开端
    /// </summary>
    private void OnEnable()
    {
        //粉丝通过订阅偶像来获取偶像的咨询, 并在得到讯息后执行相应的动作
        Idol.IdolDoSomethingHandler += HateIdol;
    }

    /// <summary>
    /// OnEnable在该脚本被禁用时调用,你可以把它看做粉转路的开端
    /// </summary>
    private void OnDisable()
    {
        Idol.IdolDoSomethingHandler -= HateIdol;
    }

    /// <summary>
    /// 粉丝B是一个无脑黑
    /// </summary>
    /// <param name="idolAction"></param>
    public void HateIdol(string idolAction)
    {
        print(idolAction + " I will hate you forever!");
    }
}
```

将三个脚本分别绑定在一个GameObject上，并运行游戏

![image-20210315153009185](UnityEvent%E5%92%8C%E4%BA%8B%E4%BB%B6%E7%B3%BB%E7%BB%9F/image-20210315153009185.png)

1. 粉丝必须订阅idol才能收到通知
2. 本例中两个粉丝均采用了调用静态字段IdolDoSomethingHandler的方法实现订阅，实际上你也可以为每个粉丝添加一个public Idol idol;然后在Editor上直接绑定。（这点是Unity特有的）

3. idol广播时不会知道粉丝的反应，也不会对粉丝的反应做出回应
4. 我们在声明event delegate时并没有给它分配内存，使用时直接赋值或添加即可。



### UnityEvent

Unity在C#Event特性的基础上进行了改良，Event只能用于纯代码编程，而UnityEvent可以和UnityEditor配合使用提高效率。

上一节中，你必须在每个粉丝对象中访问Idol的IdolDoSomethingHandler，然后把自己将采取的行动添加上去。这样有两个坏处——其一就是你必须时刻提防订阅的时机，假如不小心在Idol发动态之后才订阅，那你就永远收不到那条动态了。其二就是不方便管理，想要查看订阅偶像的所有粉丝，我们就得查找项目中所有IdolDoSomethingHandler的引用，然后再把每个粉丝的文件打开，可以说是非常麻烦了。

为了避免上述的缺点，UnityEvent使用Serializable让用户可以在Editor中直接绑定所有粉丝的调用，即一目了然又不用担心把握不准订阅的时机。

> **Idol.cs**

```c#
using UnityEngine;
using UnityEngine.Events;

//使用Serializable序列化IdolEvent, 否则无法在Editor中显示
[System.Serializable]
public class IdolEvent : UnityEvent<string> {

}

public class Idol : MonoBehaviour {
    //public delegate void IdolBehaviour(string behaviour);
    //public event IdolBehaviour IdolDoSomethingHandler;
    public IdolEvent idolEvent;

    private void Start()
    {
        //Idol 决定搞事了, 如果他还有粉丝的话, 就必须全部都通知到
        if (idolEvent == null)
        {
            idolEvent = new IdolEvent();
        }
        idolEvent.Invoke("Idol give up writing.");
    }
}
```

> **SubscriberA.cs**

```c#
using UnityEngine;

public class SubscriberA : MonoBehaviour {
    /// <summary>
    /// 粉丝A是一个脑残粉
    /// </summary>
    /// <param name="idolAction"></param>
    public void LikeIdol(string idolAction)
    {
        print(idolAction + " I will support you forever!");
    }
}
```

> **SubscriberB.cs**

```c#
using UnityEngine;

public class SubscriberB : MonoBehaviour {
    /// <summary>
    /// 粉丝B是一个无脑黑
    /// </summary>
    /// <param name="idolAction"></param>
    public void HateIdol(string idolAction)
    {
        print(idolAction + " I will hate you forever!");
    }
}
```

将三个脚本绑定在三个GameObject上，此时两个粉丝还未实现订阅。和Event不同，UnityEvent在序列化后可以在Editor上显示，并且可以在Editor上设置好需要执行的函数

![image-20210315164014120](UnityEvent%E5%92%8C%E4%BA%8B%E4%BB%B6%E7%B3%BB%E7%BB%9F/image-20210315164014120.png)

运行

![image-20210315160434052](UnityEvent%E5%92%8C%E4%BA%8B%E4%BB%B6%E7%B3%BB%E7%BB%9F/image-20210315160434052.png)

除此之外，UnityEvent依然提供和C# Event 类似的运行时绑定的功能，不过不同的是，UnityEvent是一个对象，向其绑定函数是通过AddListener()方法实现的。以SubscriberB为例，我们可以在代码中实现同等效果的绑定：

> **SubscriberB.cs**

```c#
using UnityEngine;

public class SubscriberB : MonoBehaviour {
    public Idol myIdol;

    /// <summary>
    /// OnEnable在该脚本被启用时调用,你可以把它看做路转粉的开端
    /// </summary>
    private void OnEnable()
    {
        //粉丝通过订阅偶像来获取偶像的咨询, 并在得到讯息后执行相应的动作
        myIdol.idolEvent.AddListener(HateIdol);
    }

    /// <summary>
    /// OnEnable在该脚本被禁用时调用,你可以把它看做粉转路的开端
    /// </summary>
    private void OnDisable()
    {
        myIdol.idolEvent.RemoveListener(HateIdol);
    }

    /// <summary>
    /// 粉丝B是一个无脑黑
    /// </summary>
    /// <param name="idolAction"></param>
    public void HateIdol(string idolAction)
    {
        print(idolAction + " I will hate you forever!");
    }
}
```

> 由于UnityEvent是一个对象，所以自然可以允许我们通过继承实现自己的Event，实际上Unity中包括Button在内的许多UI组件的点击事件都是通过继承自UnityEvent来复写的。 
> 可访问性(public/private)决定了UnityEvent的默认值，当可访问性为public时，默认会为其分配空间(new UnityEvent())；当可访问性为private时，默认UnityEvent为null，需要在Start()中为其分配内存。



## EventSystem

EventSystem在Unity中是一个看起来像是专门服务于UGUI系统的组件，每当在场景里创建UGUI对象时，Unity编辑器都会自动产生一个EventSystem对象放在场景中，与之相对应的也有一个Canvas对象，这两个对象就组成了UGUI系统的基础，所有开发人员能看到和能用到的UGUI功能都依附于这两个对象。

### UGUI中的EventSystem

使用UGUI制作游戏界面时，EventSystem的作用就像是一个专为UGUI设计好的消息中心，它管理着所有能参与消息处理的UGUI组件，包括但不仅限于Panel，Image，Button等。 
  如果在Unity创建好EventSystem之后观察该对象上附带的组件可以看到，至少有两个组件会被自动添加，一个是EventSystem组件，也就是消息机制的核心；另一个是StandaloneInputModule，这个是负责产生输入的组件。 
  StandaloneInputModule本身是个继承自BaseInputModule的实现类，而类似的实现类Unity中还有另外几个，甚至用户也能自定义一个实现类用于事件处理。 
  看起来这个系统似乎缺少一个部分，就是怎么确定某个事件是发给谁的。在UGUI中，EventSystem主要相应的是用户在界面上的操作，也就是点击（触摸），拖动，长按之类的鼠标动作，那么EventSystem怎么知道这些操作针对的是谁呢？举个简单例子，界面上有好几个Button，用户到底点击了哪个，EventSystem怎么确定这个目标对象的？ 
  对人类来说，看一眼就知道点击的哪个了，可Unity不行，它不能“看”，只能通过数据的运算来感知这些动作，因此为了确定操作对象究竟是哪个，一个必不可少的步骤就是检测。 
  在GUI之外的游戏场景编辑中，要感知当前鼠标对准的物体是哪个，最常用的方法就是射线检测了，从摄像机对着鼠标指向的方向发出射线，通过碰撞来检测目标。这个方案简单实用，可以说在游戏中随处可见，而UGUI所使用的机制也就是这一套射线检测，只不过射线的发射和碰撞处理都被隐藏在了组件之中。 
  所以，缺失的部分就是射线检测模块，这个模块不在EventSystem上，而是在Canvas上挂着；这很好理解，Canvas是所有UGUI组件的根对象，所以由他来负责射线处理是相当正常的解决方案，至于射线到底碰到了谁，UGUI组件自然有射线接收反馈来确定。 
  Canvas上挂载的组件叫做GraphicRaycaster，它实际上是BaseRaycaster的实现类，专门负责Canvas之下的图形对象的射线检测与计算问题。 
  至此，EventSystem在UGUI中的情况就比较清晰了，一个EventSystem对象负责管理所有事件相关对象，该对象下挂载了EventSystem组件和StandaloneInputModule组件，前者为管理脚本，后者为输入模块。Canvas对象下挂载了GraphicRaycaster负责处理射线相关运算，用户的操作都会通过射线检测来映射到UGUI组件上，InputModule将用户的操作转化为射线检测，Raycaster则找到目标对象并通知EventSystem，最后EventSystem发送事件让目标对象进行响应。

### 事件响应

UGUI的事件响应处理有多种方式，最简单而直接的一种应该就是通过实现特定接口的方法来处理事件响应了。 
  由于Canvas挂载了GraphicRaycaster组件，因此在Canvas对象之下的所有GUI对象都可以通过挂载脚本并且实现一些和事件相关的接口来处理事件，比如常见的IPointerClickHandler接口就是用于处理点击事件的接口。 
  可以实现的接口列表大概如下所示

- IPointerEnterHandler - OnPointerEnter - 当光标进入对象时调用
- IPointerExitHandler - OnPointerExit - 当光标退出对象时调用
- IPointerDownHandler - OnPointerDown - 在对象上按下指针时调用
- IPointerUpHandler - OnPointerUp - 松开鼠标时调用（在指针正在点击的游戏对象上调用）
- IPointerClickHandler - OnPointerClick - 在同一对象上按下再松开指针时调用
- IInitializePotentialDragHandler - OnInitializePotentialDrag - 在找到拖动目标时调用，可用于初始化值
- IBeginDragHandler - OnBeginDrag - 即将开始拖动时在拖动对象上调用
- IDragHandler - OnDrag - 发生拖动时在拖动对象上调用
- IEndDragHandler - OnEndDrag - 拖动完成时在拖动对象上调用
- IDropHandler - OnDrop - 在拖动目标对象上调用
- IScrollHandler - OnScroll - 当鼠标滚轮滚动时调用
- IUpdateSelectedHandler - OnUpdateSelected - 每次勾选时在选定对象上调用
- ISelectHandler - OnSelect - 当对象成为选定对象时调用
- IDeselectHandler - OnDeselect - 取消选择选定对象时调用
- IMoveHandler - OnMove - 发生移动事件（上、下、左、右等）时调用
- ISubmitHandler - OnSubmit - 按下 Submit 按钮时调用
- ICancelHandler - OnCancel - 按下 Cancel 按钮时调用

  只要在挂载的脚本中实现所需要的接口，对应的事件回调也就可以执行了。

```c#
public class EventTest : MonoBehaviour, IPointerClickHandler, IDragHandler, IPointerDownHandler, IPointerUpHandler {

    public void OnDrag(PointerEventData eventData) {
        // Execute every update when dragging
    }

    public void OnPointerClick(PointerEventData eventData) {
        // quick down and up will perform click
    }

    public void OnPointerDown(PointerEventData eventData) {
        // pointer down
    }

    public void OnPointerUp(PointerEventData eventData) {
        // pointer up
    }

    // Use this for initialization
    void Start () {
        //
    }

    // Update is called once per frame
    void Update () {
        //
    }
}
```

第二种方式相对要复杂一些，主要是利用了EventTrigger组件。 
EventTrigger组件是一个通用的事件触发器，它可以用来管理单个组件上的所有可能触发的事件，其使用方法有编辑器设定和动态设置两种。 
编辑器设定方法是在指定组件上添加EventTrigger组件，然后为它添加触发事件类型，再为指定类型添加回调方法。这种做法的操作很简单，而且灵活性也相当高，想要跨脚本调用方法只需要鼠标拖一拖点一点就好。 
但是这样在编辑器中设定事件回调会在项目变大时造成比较严重的管理障碍，尤其是当绑定了EventTrigger以及回调指向的物体有修改或者删除情况时，所造成的引用缺失需要花费更多的时间进行处理。 
所以，想要更好地管理大量的事件触发和回调处理，可以尝试采用动态设置的方案。 
所谓动态设置其实就是在代码中设置EventTrigger来处理事件回调，方法也很简单

```c#
protected void setupEventTrigger(GameObject target, UnityAction<BaseEventData> listener, EventTriggerType type) {
    if(target != null) {
        EventTrigger trigger = target.GetComponent<EventTrigger>() as EventTrigger;
        if(trigger == null) {
            trigger = target.AddComponent<EventTrigger>();
        }
        trigger.triggers = new List<EventTrigger.Entry>();
        EventTrigger.Entry entry = new EventTrigger.Entry();
        entry.eventID = type;
        entry.callback = new EventTrigger.TriggerEvent();
        entry.callback.AddListener(listener);
        trigger.triggers.Add(entry);
    }
}
```

将这个方法放到需要的类中，然后针对某个GameObject调用即可。 
当然还可以把这个方法变为拓展方法，更方便调用。

```c#
public static void setupEventTrigger(this GameObject obj, UnityAction<BaseEventData> listener, EventTriggerType type) {
    EventTrigger trigger = obj.GetComponent<EventTrigger>() as EventTrigger;
    if (trigger == null) {
        trigger = obj.AddComponent<EventTrigger>();
    }
    trigger.triggers = new List<EventTrigger.Entry>();
    EventTrigger.Entry entry = new EventTrigger.Entry();
    entry.eventID = type;
    entry.callback = new EventTrigger.TriggerEvent();
    entry.callback.AddListener(listener);
    trigger.triggers.Add(entry);
}
```

参数中的UnityAction类型是Unity自带的一个事件委托封装，其用法和C#的Action类很相似，只要将一个参数为BaseEventData的方法作为参数传入即可。而EventTriggerType是一个枚举类，其中包含了所有可能用到的事件类型，在这里用于说明当前绑定的回调是响应哪个事件的。 
  需要注意的是，EventTrigger绑定的回调会消化掉事件，因此如果还有事件透传的需求要手动解决；比如在ScrollView中的组件，如果使用添加EventTrigger的方法来实现了点击事件回调则ScrollView的滑动事件就被消化掉了，换言之此时如果点击到组件上并拖动，ScrollView并不会滑动，因为事件已经被实现了点击事件回调的组件拦截下来了。 
  除了使用透传，针对点击事件和拖拽事件的冲突问题还有另外一种解决方案，那就是Button组件。 
  UGUI的Button组件上挂载了一个名为Button的脚本，它的主要作用就是实现和Button相关的一系列功能，包括回调，鼠标悬浮，点击变色等等。而这个脚本里有一个预设的点击事件onClick，这个字段本身是个ButtonClickedEvent对象，而这个类继承自UnityEvent，它的实现里有关于回调的两个方法AddListener和RemoveListener，用起来就像是传统的监听器那样。 
  如果不使用EventTrigger来设置PointerClick事件回调，而是为组件添加Button脚本并通过向onClick增加监听器来实现点击事件回调的话，那么事件的冲突问题就得到了解决，Button脚本只会响应并消化跟点击相关的事件，其它的事件都会自动透传。 
  Button的使用方法如下

```c#
protected void setupClickListener(Button btn, UnityAction listener, bool isAppend = false) {
    if(btn != null) {
        if(!isAppend) {
            btn.onClick.RemoveAllListeners();
        }
        btn.onClick.AddListener(listener);
    }
}
```

注意在调用方法前要通过GameObject.AddComponent方法来为指定对象添加Button组件，使用返回值作为参数调用设置方法。 
当然也可以将它改为拓展方法

```c#
public static void setupOnclickListener(this GameObject obj, UnityAction listener, bool isAppend = false) {
    Button btn = obj.GetComponent<Button>();
    if(btn == null) {
        btn = obj.AddComponent<Button>();
    }
    if(!isAppend) {
        btn.onClick.RemoveAllListeners();
    }
    btn.onClick.AddListener(listener);
}
```

### 场景中的EventSystem

EventSystem虽然多见于UGUI使用中，但其实它也能在一般的场景中使用，如果没有实现自己的事件系统而又需要一些回调处理的方案的话，可以试着直接将EventSystem应用到一般的游戏场景中。 
  要这样使用EventSystem的话，核心在于前文提到过的事件系统三大部分，分别是EventSystem，InputModule和Raycaster。通过考察三者各自的作用可知，EventSystem和InputModule都和EventSystem对象紧密结合，而唯有Raycaster是孤零零地在Canvas对象上处理所有Canvas内部的射线检测。 
  那么想要借助EventSystem的能力来处理场景中的事件传递，肯定不能去动EventSystem对象，毕竟这是建立事件系统时自动创建的对象，不用说一定是要用到的。那么就只剩下Raycaster了，这个组件在Canvas上挂载，用于处理射线检测，那么如果想要在场景里进行射线检测，应该把组件挂到哪里呢？ 
  一般而言，摄像机是一个不错的选择，因为通常来说游戏大部分时候都只有一个摄像机，而且基本上可以操作的界面也只隶属于一个摄像机，因此将Raycaster挂载到游戏的主摄像机上就是个很自然的考虑了。 
​		而Unity编辑器提供的Raycaster一共有三种

- GraphicRaycaster 界面射线处理器，用于Canvas
- Physics2DRaycaster 2D场景射线处理器，用于2D场景
- PhysicsRaycaster 3D场景射线处理器，用于3D场景

  因此要用到的就是后两种了，根据当前场景的特点选择相应的Raycaster并挂载到主摄像机上即可，剩下的就和UGUI中很像了。不过需要注意的是，在UGUI中想要让组件可以响应事件必须将组件的RaycasterTarget属性勾选上，而场景中则要在需要响应事件的对象上挂载碰撞器，满足需求的任何碰撞器都可以。 
  然后就和前文讲的一样，实现对应接口或者添加EventTrigger组件来实现各种事件回调。 
  一个需要重视的地方在于，使用这样的方案实现的回调，其传递的数据PointerEventData中包含的位置参数还是屏幕位置，而且跟像素相关，以屏幕左下角为原点的坐标。如果希望获取触发事件时的世界坐标，则需要用到PointerEventData类中的pointerCurrentRaycast成员，该成员表示了射线检测的结果，因此其中包含碰撞点的世界坐标。 

