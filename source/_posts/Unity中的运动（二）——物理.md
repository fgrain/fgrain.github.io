---
title: Unity中的运动（二）——物理
date: 2021-03-05 20:32:14
tags:
	- Unity
---

**本文内容**
* 控制刚体小球的速度
* 通过跳跃支持垂直运动
* 检查地面及其角度
* 使用ProBuilder创建测试场景
* 沿着斜坡移动

<!-- more -->

## 1.刚体

为了使球体在复杂的3D环境中移动，我们必须支持与任意几何图形的交互。我们将使用Unity现有的物理引擎，而不是自己实现。

与物理引擎结合可以使用两种通用方法来控制角色。首先是刚体方法，即通过施加力或改变其速度，使角色的行为像常规物理对象一样，而间接控制它。第二种是运动学方法，即在仅查询物理引擎以执行自定义碰撞检测的同时进行直接控制。

### 1.1刚体组件

用第一种方式控制球体首先要向其添加一个刚体组件。可以使用刚体的默认配置。

添加刚体之后在代码中去掉位置约束，移动小球，小球可以经过平面边缘，但会掉下去

![image-20210305211826449](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8%EF%BC%88%E4%BA%8C%EF%BC%89%E2%80%94%E2%80%94%E7%89%A9%E7%90%86/image-20210305211826449.png)

为防止小球掉下去我们可以围绕着平面造一堵墙，当小球视图移动到一个角落时，由于物理引擎和代码为定位球产生冲突，球会变得抖动。我们将其移入墙壁，PhysX会将其后推来解决碰撞，如果停止推动PhysX将使球体因为动量保持运动。

![image-20210305214133898](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8%EF%BC%88%E4%BA%8C%EF%BC%89%E2%80%94%E2%80%94%E7%89%A9%E7%90%86/image-20210305214133898.png)

### 1.2控制刚体速率

直接调整球的位置可以有效的使球移动，但有些情况我们需要对球施加力或调整球的速度来间接控制球。

我们需要访问球体的刚体组件， 可以在Awake中初始化字段。然后给刚体设置速度

```c#
	private Rigidbody body;
    private void Awake()
    {
        body = GetComponent<Rigidbody>();
    }
```

小球碰撞到墙时会影响速度，所以我们要先获取小球的速度，再对小球的速度进行调整

```c#
 		velocity = body.velocity;

        Vector3 desiredSpeed = new Vector3(playerInput.x, 0, playerInput.y) * maxSpeed;
        float maxSpeedChange = maxAcceleration * Time.deltaTime;

        velocity.x = Mathf.MoveTowards(velocity.x, desiredSpeed.x, maxSpeedChange);
        velocity.z = Mathf.MoveTowards(velocity.z, desiredSpeed.z, maxSpeedChange);

        body.velocity = velocity;
```

### 1.3无摩擦运动

通过调整球体的速度，我们让小球看上去像之前仅调整位移的运动，但速度慢了许多，这是因为PhysX会产生摩擦，虽然这更真实，但会使配置球体变得困难。我们可以创建一个新的物理材质取消摩擦和反弹，把所有值设为0，并将“combine”设为Minimum。

![image-20210306143701902](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8%EF%BC%88%E4%BA%8C%EF%BC%89%E2%80%94%E2%80%94%E7%89%A9%E7%90%86/image-20210306143701902.png)

然后把材质分配给球体的碰撞器（collider）

![image-20210306143732725](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8%EF%BC%88%E4%BA%8C%EF%BC%89%E2%80%94%E2%80%94%E7%89%A9%E7%90%86/image-20210306143732725.png)

现在小球不受摩擦和碰撞，运动丝滑了许多。

但球体在碰撞的时候仍会反弹一点，这是因为PhysX不会防止碰撞，而是会在碰撞发生后检测它们，然后移动刚体以使它们不再相交。如果运动特别块墙体有很薄，会发生小球穿墙而过的情况。可以通过更改“Rigidbody”的“Collision Detcetion”模式来防止这种情况，仅在运动非常快的时候这样设置。

若我们想让小球滑动而不是滚动，通过“Rigidbody”组件的“Constraints ”冻结旋转来实现

![image-20210306145009927](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8%EF%BC%88%E4%BA%8C%EF%BC%89%E2%80%94%E2%80%94%E7%89%A9%E7%90%86/image-20210306145009927.png)

### 1.4 Fixed Update

物理引擎使用固定的时间步长，而不管帧速率如何。Upadta为每一帧调用但每一帧的时间不固定，而FixedUpdate则不受帧率影响，它使以固定时间间隔（0.02s）被调用。所以一些物理熟悉的更新操作应放在FixedUpdate中进行操作，这样物体的物理表现更加平滑。

我们将检查输入并设置速度的部分放在Update中，调整速度的部分放在FixedUpdate中。

```c#
    void Update()
    {
        playerInput.x = Input.GetAxis("Horizontal");
        playerInput.y = Input.GetAxis("Vertical");
        playerInput = Vector2.ClampMagnitude(playerInput, 1f);
        desiredSpeed = new Vector3(playerInput.x, 0, playerInput.y) * maxSpeed;
    }
	private void FixedUpdate()
    {
        velocity = body.velocity;     
        float maxSpeedChange = maxAcceleration * Time.deltaTime;
        velocity.x = Mathf.MoveTowards(velocity.x, desiredSpeed.x, maxSpeedChange);
        velocity.z = Mathf.MoveTowards(velocity.z, desiredSpeed.z, maxSpeedChange);
        body.velocity = velocity;
    }
```

> 根据你的帧速率，FixedUpdate每次调用时候，Update可以被调用零次，一次或多次。每个帧都会发生一系列FixedUpdate调用，然后调用Update，然后渲染该帧。当物理时间步长相对于帧时间太大时，这会使物理仿真的离散性质变得明显。可以通过减少固定时间步长或启用刚体的插值模式来解决此问题。
>
> *Interpolate* 插值，如果发现刚体移动有卡顿，可以尝试选择此选项。 
>
> - *None* 不使用插值
> - *Interpolate* 根据上一帧的Transform进行平滑
> - *Extrapolate* 根据估算的下一帧的Transform进行平滑



## 2.跳跃

### 2.1跳跃指令

使用Input.GetButtonDown("Jump")来检测玩家是否在该帧按下跳跃按钮。默认空格为跳跃键，可以在Editor->ProjectSetting->InputManager中更改按键。我们在Update中执行这个操作，用desiredJump字段跟踪是否需要跳跃，在FixedUpdate中执行跳跃操作。

```C#
	private bool desiredJump;
	......
	void Update()
    {
        ......
        desiredJump = Input.GetButtonDown("Jump");
	}
```

但我们可能不在下一帧调用FixedUpdate，这种情况下desiredJump被设为false，跳转指令将被遗忘。我们可以通过或运算防止这种情况，直到我们显示的将其设置回false

```c#
desiredJump |= Input.GetButtonDown("Jump");
```

在调整速度后并且在fixedUpdate调用之前检测是否跳跃，若有跳跃请求，调用Jump方法，该方法初始化为在速度y的分量上加5，模拟突然向上的加速度。

![image-20210307211148536](Unity%E4%B8%AD%E7%9A%84%E8%BF%90%E5%8A%A8%EF%BC%88%E4%BA%8C%EF%BC%89%E2%80%94%E2%80%94%E7%89%A9%E7%90%86/image-20210307211148536.png)

### 2.2跳跃高度

我们将跳跃的高度设为可配置的。可以通过直接控制跳跃速度来实现这一点，但这并不直观。直接控制跳跃高度更方便。

```c#
[SerializeField, Range(0f, 10f)] float jumpHeight = 2f; 
```

跳跃需克服重力，具体来说
$$
v_y=\sqrt{-2gh}
$$

```c#
velocity.y = Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);
```

由于物理模拟是离散性的，我们很可能无法达到所需的高度。在时间步长之间的某个中间值时达到最大值。

### 2.3 在地面上跳跃

现在球可以在空中起跳，但通常来说球只能从地面起跳。我们不能直接询问刚体是否正在接触地面，但当它与物体相撞时我们可以得到通知。如果脚本有一个onCollisionEnter方法，那么PhysX检测到一个新的碰撞后就会调用它。在此之后将调用onCollisionExit方法，将onGround设为false。

```c#
	private bool onGround;
	private void OnCollisionEnter(Collision collision)
    {
        onGround = true;
    }
    private void OnCollisionExit(Collision collision)
    {
        onGround = false;
    }
```

现在我们只能在地面上跳跃，如果我们没有接触到任何物体，将忽略跳跃指令。

```c#
	if (onGround) 
            velocity.y = Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);
```

当球体只接触地面时跳跃指令会起作用，但如果球体短暂的接触了墙壁然后分开，跳跃指令就失效了，因为在和墙壁分开时调用了OnCollisionExit方法，onGround被设为false。解决方法是通过OnCollisionStay更新状态，只要保持碰撞就会调用该方法。

```c#
    private void OnCollisionStay(Collision collision)
    {
        onGround = true;
    }
```

每一个物理步骤开始时都会先调用FixedUpdate方法，最后调用collision方法。所以当FixedUpdate方法被调用时，如果有碰撞发生，onGround将在最后一步被设为true，如果要保证onGround有效，应当在FixedUpdate结束时将其设置回false。

### 2.4 不应该有墙跳跃

现在接触任何物体都可以跳跃，在半空中接触墙也可以接着跳，我们需要区分墙壁和地面。将地面定义为主要水平面，可以通过检测接触点的法线向量来检查我们所碰撞的对象是否满足条件。

涉及到网格碰撞器时一次碰撞可能有多个接触组成，但在平面球体碰撞中只有一个接触点，我们在Oncollision中传递一个碰撞参数，用EvaluateCollision方法处理碰撞信息。

通过collision中的contactCount属性找到接触点的数量，通过GetContact方法遍历所有点，然后获得法线的属性

```c#
    private void EvaluateCollision(Collision collision)
    {
        for(int i = 0; i < collision.contactCount; i++)
        {
            Vector3 normal = collision.GetContact(i).normal;
        }
    }
```

从地面起跳时法线指向正上方，即y分量为1，因从当y分量为1时小球在地面上

```c#
			Vector3 normal = collision.GetContact(i).normal;
            onGround |= normal.y == 1;
```

 ### 2.5 空中起跳

有些情况下游戏会支持多段跳，我们可以设置最多允许多少次空中起跳。

```c#
    [SerializeField, Range(0f, 5f)] float maxAirJumps = 2;
    private float jumpPhase = 0;
```

现在在FixedUpdate中检查小球状态，若小球在地面jumpPhase = 0，在达到最大跳跃次数前都可以起跳，将状态更新的语句单独放在一个方法中方面后续更新。

```c#
	private void UpdateState()
    {
        velocity = body.velocity;
        if (onGround) jumpPhase = 0;
    }
    private void Jump()
    {
        if (onGround || jumpPhase < maxAirJumps)
        {
            jumpPhase++;
            velocity.y = Mathf.Sqrt(-2f * Physics.gravity.y * jumpHeight);
        }
    }
```

