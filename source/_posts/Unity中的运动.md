---
title: Unity中的运动（一）——滑动小球
date: 2021-02-19 14:35:48
tags:
	- Unity
---

## 1.控制位置

### 1.1设置场景

​	创建一个平面和一个球体，将摄像机放到平面上方，使用正交视角，关闭球体的投射阴影

![image-20210219155957137](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219155957137.png)

分别为球体和平面创建材质，并为球体的移动轨迹创建材质，同时为球体创建一个移动脚本

<!-- more -->

![image-20210219160153824](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219160153824.png)

将MovingSphere和Trail Renderer组件挂到球体上，Trail Renderer可以标记出物体运动的轨迹

![image-20210219160350231](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219160350231.png)

将轨迹材质附加到Trail Renderer组件的材质上面，设置轨迹宽度和颜色，在预览里拖动

![image-20210219161201789](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219161201789.png)

### 1.2 读取玩家输入

由于是平面移动所以用vector2储存玩家输入，将x，和y初始化为0，使他们将球体固定在XZ平面上，y的输入相当于z

```c#
    Vector2 playerInput;
    private void Start()
    {
        playerInput.x = 0;
        playerInput.y = 0;
    }

    void Update()
    {        
        playerInput.x = Input.GetAxis("Horizontal");
        playerInput.y = Input.GetAxis("Vertical");       
        transform.localPosition = new Vector3(playerInput.x, 0, playerInput.y);
    }
```

调用方法Input.GetAxis作为输入。我们将水平值用于X，将垂直值用于Y。默认通过WASD控制输入，演示如下

![image-20210219162419691](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219162419691.png)

### 1.3 归一化输入向量

轴在静止时返回零，而在极限时返回-1或1。当我们使用输入来设置球体的位置时，它被约束为具有相同范围的矩形。至少，按键输入就是这种情况，因为每次按键是独立的。如果是摇杆，则是连续的，通常我们在任何方向上都被限制为距原点的最大距离为1，从而将位置限制在一个圆内。

控制器输入的优点是，无论方向如何，输入向量的最大长度始终为1。因此，各个方向的移动速度都可以一样快。按键不是这种情况，单个按键的最大值为1，而同时按下两个按键的最大值为√2，这意味着对角线移动最快。

通过将输入矢量除以其大小，可以确保矢量的长度永远不会超过1。结果始终为单位长度矢量，除非其初始长度为零，但在这种情况下，结果是未知的。此过程称为归一化向量。我们可以通过对向量调用Normalize来实现此目的，该向量会自行缩放并在结果不确定时变为零向量。

![image-20210219170546677](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219170546677.png)

### 1.4 约束输入向量

限定球运动的范围可通过约束向量的大小实现，一种方便的方法是调用静态Vector2.ClampMagnitude方法而不是Normalize，并使用向量（最大值为1）作为参数。结果是一个相同或缩小到所提供最大值的向量。

```c#
playerInput = Vector2.ClampMagnitude(playerInput, 0.5f);
```

调整向量大小使运动范围变小

![image-20210219171652618](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219171652618.png)



## 2.控制速度

目前位置球体位置的变换的输入的向量是同一个值，这不适合运动，我们要不断更新球体的位置，通过输入向量加上球体上一刻的位置来计算

### 2.1 相对运动

```C#
Vector3 displacement= new Vector3(playerInput.x, 0, playerInput.y);
transform.localPosition += displacement; 
```



![image-20210219175445608](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219175445608.png)

### 2.2 速率

由于update每帧都在更新球体位置，所以球体速度不受控制，帧率越高速度越快。想要获得稳定的速度可以通过增量时间Time.deltaTime实现。增量时间是实时变动的，而且每一帧都在变动
1秒30帧，那增量时间就是 1/30 秒
1秒60帧，那增量时间就是 1/60 秒

```c#
Vector3 displacement = new Vector3(playerInput.x, 0, playerInput.y) * Time.deltaTime;
```

![image-20210219180953286](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219180953286.png)

### 2.3 速度

最大输入向量为一，在unity里的速度就是1m/s，可以通过缩放向量增大速度，设置一个maxSpeed字段并带有SerializeField属性，提供Range属性.

SerializeField意为序列化域，可以将private属性的字段现实在检查器面板里，在面板里修改后再次启动unity它是有值的，不需要再次赋值

```c#
[SerializeField, Range(0f, 100f)] float maxSpeed = 10f;
```
```c#
Vector3 velocity = new Vector3(playerInput.x, 0, playerInput.y) * maxSpeed;
Vector3 displacement = velocity * Time.deltaTime;
```
![image-20210219222322265](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219222322265.png)



### 2.4 加速度

由于代码能直接控制速度，因此可以立即变更速度，但实际上速度不能被立即更改，变更速度需要加速度，引入加速度后，速度这个变量要时时更新

```c#
private Vector3 velocity;
```

```c#
Vector3 acceleration= new Vector3(playerInput.x, 0, playerInput.y) * maxSpeed;
velocity += acceleration * Time.deltaTime;
Vector3 displacement = velocity * Time.deltaTime;
transform.localPosition += displacement;
```



### 2.5 所需速度

控制加速度而不是速度会产生更平滑的运动，但同时会消弱我们对球体的控制。但在大多数游戏中需要对速度进行更直接的控制。

我们可以通过直接控制目标速度并将加速度应用于实际速度，直到它与期望的速度相匹配。然后，我们可以通过调整球的最大加速度来调整球的响应速度。为此添加一个可序列化的字段。

```c#
[SerializeField, Range(0f, 100f)] float maxAcceleration = 10f;
```

我们设置一个最大速度和最大加速度

```c#
Vector3 desiredSpeed = new Vector3(playerInput.x, 0, playerInput.y) * maxSpeed;
float maxSpeedChange = maxAcceleration * Time.deltaTime;
```

![image-20210219231746961](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210219231746961.png)

当速度不达标时做出改变

```c#
if (velocity.x < desiredSpeed.x)
{
   //velocity.x += maxSpeedChange;//可能导致速度溢出
   velocity.x = Mathf.Min(velocity.x + maxSpeedChange, desiredSpeed.x);
}
   else if (velocity.x > desiredSpeed.x)
{
   velocity.x = Mathf.Max(velocity.x - maxSpeedChange, desiredSpeed.x);
}
```

也可以用Mathf.MoveTowards函数实现上述功能，参数为当前值，目标值，每次变化的量

```c#
velocity.x = Mathf.MoveTowards(velocity.x, desiredSpeed.x, maxSpeedChange);
velocity.z = Mathf.MoveTowards(velocity.z, desiredSpeed.z, maxSpeedChange);
```



## 3.约束位置

除了控制角色速度外，游戏很多地方还需要限制角色的移动范围

### 3.1 留在正方形内

为了限定小球移动范围，我们使用Rect结构，此结构共有四个参数，前两个代表左下角的坐标值，后两个代表矩形的长和宽。

```c#
[SerializeField] Rect allowedArea=new Rect(-5,-5,10,10);
```

将新位置赋值给transform.localPosition之前，我们先判断它是否在允许的范围内，利用Rect结构的Contains方法，此时注意若参数直接为newPosition则会检测其XY坐标，但我们需要检查的是XZ坐标，所有将其传递给新的Vector2

```c#
Vector3 newPosition = transform.localPosition + displacement;
if(!allowedArea.Contains(new Vector2(newPosition.x, newPosition.z)))
{
    newPosition = transform.localPosition;
}
transform.localPosition = newPosition;
```

![image-20210220022600893](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8/image-20210220022600893.png)



### 3.2 确切的位置

上述代码中只要小球下一刻的坐标不在范围内不会动，这样导致一个问题，当小球运动到边缘时若想让他转向，此时小球有两个方向的速度，合成后的走向依然不在范围内，要等到一方的速度归零才能做出转向。

我们可以使用Mathf.Clamp函数来讲新的位置始终限制在约束范围内，此函数有三个参数，第一个是value，第二个是最小值，第三个为最大值，value的范围被限制在他们当中。这样小球运动到边缘转向时就不会有卡顿的情况。

```c#
newPosition.x = Mathf.Clamp(newPosition.x, allowedArea.xMin, allowedArea.xMax);
newPosition.z = Mathf.Clamp(newPosition.z, allowedArea.yMin, allowedArea.yMax);
```



### 3.3 消除速度

此时小球运动到边缘时速度并没有立马归零，而是根据加速度的大小逐渐降低，但事实上此时物体的速度是0。所有在小球碰到边缘是我们要消除指向该边缘方向的速度分量。

```c#
		if (newPosition.x < allowedArea.xMin)
        {
            newPosition.x = allowedArea.xMin;
            velocity.x = 0;
        }
        else if(newPosition.x>allowedArea.xMax)
        {
            newPosition.x = allowedArea.xMax;
            velocity.x = 0;
        }
        if (newPosition.z < allowedArea.yMin)
        {
            newPosition.z = allowedArea.yMin;
            velocity.z = 0;
        }
        else if (newPosition.z > allowedArea.yMax)
        {
            newPosition.z = allowedArea.yMax;
            velocity.z = 0;
        }
```



### 3.4 反弹

在碰撞过程中并非总是消除速度，如果我们的球体是一个完美的弹跳球，它将反转运动方向。

```c#
		if (newPosition.x < allowedArea.xMin)
        {
            newPosition.x = allowedArea.xMin;
            velocity.x = -velocity.x;
        }
        else if(newPosition.x>allowedArea.xMax)
        {
            newPosition.x = allowedArea.xMax;
            velocity.x = -velocity.x;
        }
        if (newPosition.z < allowedArea.yMin)
        {
            newPosition.z = allowedArea.yMin;
            velocity.z = -velocity.z;
        }
        else if (newPosition.z > allowedArea.yMax)
        {
            newPosition.z = allowedArea.yMax;
            velocity.z = -velocity.z;
        }
```



### 3.5 减少反弹量

反转时不需要保留全部速度，通过添加一个反弹字段使其可配置，默认情况设为0.5，范围0-1。这事小球可以完全反弹或不反弹

```c#
[SerializeField, Range(0f, 1f)] float bounciness = 0.5f;
```

```c#
		if (newPosition.x < allowedArea.xMin)
        {
            newPosition.x = allowedArea.xMin;
            velocity.x = -velocity.x * bounciness;
        }
        else if(newPosition.x>allowedArea.xMax)
        {
            newPosition.x = allowedArea.xMax;
            velocity.x = -velocity.x * bounciness;
        }
        if (newPosition.z < allowedArea.yMin)
        {
            newPosition.z = allowedArea.yMin;
            velocity.z = -velocity.z * bounciness;
        }
        else if (newPosition.z > allowedArea.yMax)
        {
            newPosition.z = allowedArea.yMax;
            velocity.z = -velocity.z * bounciness;
        }
```

