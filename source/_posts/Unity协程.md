---
title: Unity协程
date: 2021-03-22 15:10:42
tags:
	- Unity
	- 协程
---

本文参考：

* https://blog.csdn.net/huang9012/article/details/38492937

* https://blog.csdn.net/weixin_44079314/article/details/84997940

## 协程是什么

协程不是进程，也不是线程，它就是一个函数，一个特殊的函数——可以在某个地方挂起，并且可以重新在挂起处继续运行。

<!-- more-->

协程方法与普通方法的区别：

被调用时：普通方法被调用时，原来执行的部分保留现场，停止执行，然后去执行要调用的方法，并且，被调用的方法执行完之后才能返回到调用前的状态接着往下执行。协同方法的执行是不用等协同方法执行完再执行调用之前原来方法的代码。 而是两者异步执行。

## 协程的使用

Unity的协程系统是基于C#的一个简单而强大的接口 ，[IEnumerator](http://msdn.microsoft.com/en-us/library/system.collections.ienumerator.aspx)，它允许你为自己的集合类型编写枚举器。在Unity中只要继承MonoBehaviour就可以使用协程

```c#
IEnumerator DelayCoroutine() {
	// work before delay
	yield return new WaitForSeconds(<time value to delay>);
	// work after delay
}
StartCoroutine(DelayCoroutine());
```

在IEnumerator类型的方法中写入需要执行的操作，遇到yield后会暂时挂起，等到yield return后的条件满足才继续执行yield语句后面的内容。

通过StartCoroutine启动协程。StartCoroutine会判断yield返回的类型，如果他发现IEnumerator其实是一个WaitForSeconds类型的话，那么他就会进行特殊等待，一直等到WaitForSeconds延时结束了，才会结束挂起，而至于WWW或者WaitForFixedUpdate等类型，StartCoroutine也是同样的特殊处理

```c#
yield return null; // 下一帧再执行后续代码
yield return 0; //下一帧再执行后续代码
yield return 6;//(任意数字) 下一帧再执行后续代码
yield break; //直接结束该协程的后续操作
yield return asyncOperation;//等异步操作结束后再执行后续代码
yield return StartCoroution(/*某个协程*/);//等待某个协程执行完毕后再执行后续代码
yield return WWW();//等待WWW操作完成后再执行后续代码
yield return new WaitForEndOfFrame();//等待帧结束,等待直到所有的摄像机和GUI被渲染完成后，在该帧显示在屏幕之前执行
yield return new WaitForSeconds(0.3f);//等待0.3秒，一段指定的时间延迟之后继续执行，在所有的Update函数完成调用的那一帧之后（这里的时间会受到Time.timeScale的影响）;
yield return new WaitForSecondsRealtime(0.3f);//等待0.3秒，一段指定的时间延迟之后继续执行，在所有的Update函数完成调用的那一帧之后（这里的时间不受到Time.timeScale的影响）;
yield return WaitForFixedUpdate();//等待下一次FixedUpdate开始时再执行后续代码
yield return new WaitUntil()//将协同执行直到 当输入的参数（或者委托）为true的时候....如:yield return new WaitUntil(() => frame >= 10);
yield return new WaitWhile()//将协同执行直到 当输入的参数（或者委托）为false的时候.... 如:yield return new WaitWhile(() => frame < 10);
```

### 一些例子

多次输出“Hello”，一帧输出一次

```c#
	void Start()
    {
        StartCoroutine(SayHelloFiveTimes());
    }  
	IEnumerator SayHello5Times()
    {
        for (int i = 0; i < 5; i++)
        {
            Debug.Log("Hello");
            yield return 0;
        }
    }
```

每一帧输出“Hello”，无限循环

```c#
    IEnumerator SayHelloEveryFrame()
    {
        while (true)
        {
            Debug.Log("Hello");
            yield return 0;
        }
    }
```

计时

```c#
	IEnumerator CountSeconds()
    {
        int seconds = 0;
        while (true)
        {
            yield return new WaitForSeconds(1f);
            seconds++;
            Debug.Log(seconds + " seconds have passed since the Coroutine started.");
        }
    }
```

![image-20210322153957597](Unity%E5%8D%8F%E7%A8%8B/image-20210322153957597.png)

协程方法有个特点就是当协程挂起时，方法的状态被存储了，这使得方法中定义的这些变量都会保存它们的值，即使是在不同的帧中。

### 开始和终止协程

开始协程在上面的例子中用到了

```c#
StartCoroutine(CountSeconds());
```

如果我们想要终止所有的协程，可以通过[StopAllCoroutines()](http://docs.unity3d.com/Documentation/ScriptReference/MonoBehaviour.StopAllCoroutines.html)方法来实现。

如果想要终止特定的协程不能通过StopCoroutine(CountSeconds())来实现，我们要结束一个协程对象可以通过下面的方式

```c#
	Coroutine c;
    void Start()
    {
        c = StartCoroutine(CountSeconds());        
    }

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.J))
        {
            StopCoroutine(c);
        }
    }
```

或者在开始协程的时候将它的方法名作为字符串，这样结束时也可以通过字符串结束

```c#
StartCoroutine("CountSeconds");
StopCoroutine("CountSeconds");
```

### 协程的参数

协程作为方法可以传递参数，当协程有参数的时候不能用字符串启动。

```c#
	void Start()
    {
        //输出5次Hello，间隔1秒
        c = StartCoroutine(RepeatMessage(5, 1, "Hello"));
    }
	IEnumerator RepeatMessage(int count, float frequency, string message)
    {
        for (int i = 0; i < count; i++)
        {
            Debug.Log(message);
            yield return new WaitForSeconds(frequency);
        }
    }
```

### 嵌套的协程

协程最强大的一个功能就是它们可以通过使用yield语句来相互嵌套。

```c#
	IEnumerator SaySomeThings()
    {
        Debug.Log("The routine has started");
        yield return StartCoroutine(RepeatMessage(1, 1f, "Hello"));
        Debug.Log("1 second has passed since the last message");
        yield return StartCoroutine(RepeatMessage(1, 2.5f, "Hello"));
        Debug.Log("2.5 seconds have passed since the last message");
    }
```

![image-20210322161328013](Unity%E5%8D%8F%E7%A8%8B/image-20210322161328013.png)

### 控制对象行为的例子

* **运动到某一位置**

在Inspector面板中设置目标位置和运动速度，在游戏开始时将一个物体移动到目标位置

```c#
    public Vector3 targetPosition;
    public float moveSpeed=5;
    void Start()
    {
        c = StartCoroutine(MoveToPosition(targetPosition));
    }
	IEnumerator MoveToPosition(Vector3 target)
    {
        while (transform.position != target)
        {
            transform.position = Vector3.MoveTowards(transform.position, target, moveSpeed*Time.deltaTime);
            yield return 0;
        }
    }
```

* **按指定路径前进**

我们可以让运动到某一位置的程序做更多，不仅仅是一个指定位置，我们还可以通过数组来给它赋值更多的位置，通过MoveToPosition() ，我们可以让它在这些点之间持续运动。

```c#
	public List<Vector3> path;    
	IEnumerator MoveOnPath(bool loop)
    {
        do
        {
            foreach (var point in path)
                yield return StartCoroutine(MoveToPosition(point));
        }
        while (loop);
    }
```

## 协程的执行顺序

### 生命周期函数

![1](Unity%E5%8D%8F%E7%A8%8B/1.png)

![2](Unity%E5%8D%8F%E7%A8%8B/2.png)

![3](Unity%E5%8D%8F%E7%A8%8B/3.png)

### return null的协程

可以看到，在GameLogic部分对协程中挂起的条件进行了判断。
也就是说，协程顺序为：
（当前帧为第1帧）
第1帧在start中开启协程，执行协程（自上而下），遇到yield return null 将后面的内容挂 起。
这时继续执行第1帧剩下的东西直到第1帧Update执行结束，这时对挂起的协程进行判断 是否满足return条件， 
满足则在第2帧Update之后，在LateUpdate前执行协程中yield return 以后的代码；
不满足条件则继续执行第1帧的LateUpdate。
第2帧同第1帧相同。

```c#
using System.Collections;
using UnityEngine;

public class CorTest2 : MonoBehaviour
{
    int i = 0;//update中判断次数的变量
    private void Start()
    {
        Debug.Log("start 1");
        //开启协程1
        StartCoroutine(Test());
        Debug.Log("start 2");

    }
    private void Update()
    {
        Debug.Log("第" + ++i + "帧开始");
    }
    private void LateUpdate()
    {
        Debug.Log("第" + i + "帧结束");
    }
    IEnumerator Test()
    {
        while (true)
        {
            Debug.Log("协程1第一次");
            //挂起时机
            yield return null;
            Debug.Log("协程1第二次");
        }
    }
}
```

![image-20210322165718479](Unity%E5%8D%8F%E7%A8%8B/image-20210322165718479.png)

可以看到，协程运行到一半在第一帧被挂起，第二帧Update执行完后满足条件继续执行。

### yield return StartCoroutine()

```c#
	IEnumerator Test()
    {
        while (true)
        {
            Debug.Log("协程1第一次");
            //挂起时机
            yield return StartCoroutine(Test2());
            Debug.Log("协程1第二次");
        }
    }

    IEnumerator Test2()
    {
        Debug.Log("协程2第一次");
        yield return null;
        Debug.Log("协程2第二次");
    }
```

![image-20210322165932697](Unity%E5%8D%8F%E7%A8%8B/image-20210322165932697.png)

原理都是一样的，执行完yield return 后挂起（注意不是遇到就挂起，而是执行），在每一帧的update与lateupdate之间对挂起的内容进行判断，满足则继续执行被挂起的协程的剩余部分。