---
title: Unity对象管理（一） ——持久化对象
date: 2021-02-23 16:11:51
tags:
	- Unity
---

## 1.按需创建对象

我们可以在游戏中任意创造物体，例如子弹发射，敌人，随机道具生成等，但当我们退出游戏再次进入时，Unity不会自动为我们记录过程当中的变化，需要我们自己去做。

本例中我们会创建一个非常简单的游戏，在按下一个键时随机生成一个立方体。只要我们能够跟踪不同游戏会话之间的立方体，就可以在此基础上增加游戏的复杂性。

<!-- more -->

### 1.1 准备工作

我们需要一个Game组件脚本控制生成立方体，因此它需要包含一个public字段来连接一个预置实例

```c#
	public Transform cubePrefab;
```

创建一个空物体将脚本挂在上面，在创建一个cube的预制体，给脚本一个它的引用

### 1.2 玩家输入

游戏应该根据玩家的输入来生成立方体，所以必须要检测玩家的输入。这里可以利用Unity的输入系统来检测按键，我们设置按下C建时生成立方体，通过在脚本中添加一个KeyCode字段实现

```c#
	public KeyCode creatKey;
	private void Awake()
    {
        creatKey = KeyCode.C;
    }
```

在Update方法中通过Input.GetKeyDown方法检测是否按下按键

该方法返回一个bool值，Input.GetKeyDown只在玩家按下按键的第一帧返回true，Input.GetKey每一帧都返回true，还有Input.GetKeyUp在玩家松开按键时返回true

```c#
    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.C))
        {
            Instantiate(cubePrefab);
        }
    }
```

### 1.3 随机立方体

上述方法生成立方体时只会在初始位置生成立方体，多个立方体叠加生成，我们希望随机化创建每一个立方体，大小位置旋转都不相同。

为了方便实例化多个对象，我们将初始化立方体单独写成一个方法

```c#
	private void Update()
    {
        if (Input.GetKeyDown(KeyCode.C))
        {
            CreatCube();
        }
    }

    private void CreatCube()
    {
        Transform t = Instantiate(cubePrefab);
    }
```

使用静态Random.insideUnitSphere属性获取随机点，意为在单位圆中随机生成坐标，这里将半径设为5

使用静态Random.rotation属性设置随机旋转

使用Random.Range获取随机数，并将其与Vector3.one相乘，得到随机大小

```c#
    private void CreatCube()
    {
        Transform t = Instantiate(cubePrefab);
        t.localPosition = Random.insideUnitSphere * 5;
        t.localRotation = Random.rotation;
        t.localScale = Vector3.one * Random.Range(0.1f, 1);
    }
```

![image-20210223172640314](Unity%E5%AF%B9%E8%B1%A1%E7%AE%A1%E7%90%86%EF%BC%88%E4%B8%80%EF%BC%89-%E2%80%94%E2%80%94%E6%8C%81%E4%B9%85%E5%8C%96%E5%AF%B9%E8%B1%A1/image-20210223172640314.png)

### 1.4 开始新游戏

开始新游戏我们需要退出游戏模式重新进入，但这仅在Unity编辑器里可行，玩家则需要退出应用，然后重新启动它才能开始新游戏。因此我们需要在保持游戏模式的同时开始新游戏。

我们可以通过重新加载场景开始新游戏，也可以通过销毁所有的立方体。游戏中我们通过按键检测，按下N键则开始新游戏，我们应该一次处理一个键，所有只有在C键没按下时才检测N键

```c#
	public KeyCode newGame;
	private void Awake()
    {
        creatKey = KeyCode.C;
        newGame = KeyCode.N;
    }
    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.C))
        {
            CreatCube();
        }
        else if (Input.GetKeyDown(KeyCode.N))
        {
            BeginNewGame();
        }
    }
```

### 1.5 持有物体的引用

现在游戏中可以生成随机数量的立方体，但要销毁立方体需要持有对立方体的引用，我们每生成一个立方体就将其加入List列表中，在Awake中初始化List

```c#
	private List<Transform> cubeList;
	private void Awake()
    {
        cubeList = new List<Transform>();
        creatKey = KeyCode.C;
        newGame = KeyCode.N;
    }
    private void CreatCube()
    {
        Transform t = Instantiate(cubePrefab);
        t.localPosition = Random.insideUnitSphere * 5;
        t.localRotation = Random.rotation;
        t.localScale = Vector3.one * Random.Range(0.1f, 1);
        cubeList.Add(t);
    }
```

### 1.6 清空列表

在BeginNewGame中遍历cubeList，逐个销毁游戏对象，然后清空列表。由于List保存的时Transform对象，销毁时要带上.gameObject

```c#
    private void BeginNewGame()
    {
        for(int i = 0; i < cubeList.Count; i++)
        {
            Destroy(cubeList[i].gameObject);
        }
        cubeList.Clear();
    }
```



## 2.保存和加载

如果只是在单个场景进行保存和加载，那么将一系列转换数据保存在内存中就够了。在保存时复制所有立方体的位置，旋转和缩放，并在加载时使用记住的数据重置游戏的生成立方体。最简单的方法是将数据保存在文件中。

### 2.1 保存路径

游戏文件的存储位置取决于文件系统。Unity会通过Application.persistentDataPath属性提供具体路径地址。此路径是文件夹的位置，完整的路径还需要包含文件名。

路径使用正斜杠反斜杠取决操作系统，这里使用Path.Combine处理细节

```c#
private string savePath;
//在Awake里初始化
savePath = Path.Combine(Application.streamingAssetsPath + "saveFile");
```

### 2.2 打开文件以便写入

为了保存数据首先要打开文件，通过File.Open方法完成，该方法需要2个参数，文件路径和打开方式。我们为了写入数据，需要创建新文件或替换已有文件，通过FileMode.Create作为第二个参数指定它。File.Open返回一个文件流。

```c#
public void save()
    {
		FileStream f = File.Open(savePath, FileMode.Create);
    }
```

我们使用二进制数据写入，需提供文件流作为参数

```c#
BinaryWriter writer = new BinaryWriter(f);
//也可以用var关键字
//var writer = new BinaryWriter(f);
```

### 2.3 关闭文件

我们可以用Close方法关闭文件，但这不安全。如果打开和关闭文件之间出现问题可能会引发异常，可以用try，finally捕获异常，也可以用Using语句的语法糖简化代码（没有实现IDisposable不能使用using语法糖）

```c#
        using (
            BinaryWriter writer = new BinaryWriter(File.Open(savePath, FileMode.Create))
        ){}
```

### 2.4写数据

游戏要知道我们何时保存，通过key控制此操作，默认按下S保存

```c#
public KeyCode saveGame;
//在Awake中初始化
saveGame = KeyCode.S;
```

通过调用Write方法将数据写入文件

```c#
        using (
            BinaryWriter writer = new BinaryWriter(File.Open(savePath, FileMode.Create)))
        {
            //先保存物体个数
            writer.Write(cubeList.Count);
            for(int i = 0; i < cubeList.Count; i++)
            {
                Transform t = cubeList[i];
                writer.Write(t.localPosition.x);
                writer.Write(t.localPosition.y);
                writer.Write(t.localPosition.z);
                writer.Write(t.localRotation.eulerAngles.x);
                writer.Write(t.localRotation.eulerAngles.y);
                writer.Write(t.localRotation.eulerAngles.z);
                writer.Write(t.localScale.x);
                writer.Write(t.localScale.y);
                writer.Write(t.localScale.z);
            }
        }
```

### 2.5 加载数据

加载数据要读取文件里的数据，创建一个新方法Load实现此功能

```c#
    private void Load()
    {
        using (
            var reader = new BinaryReader(File.Open(savePath, FileMode.Open)))
        {

        }
    }
```

由于写入的第一个数据是count属性，通过Reader的ReadInt32实现此操作。读取数据后实例化新的立方体并将其添加到列表中。Vector3向量使用float类型，通过ReadSingle读取。Read每读取一个字符都会提升当前流的位置。

```c#
            int count = reader.ReadInt32();
            for(int i = 0; i < count; i++)
            {
                Vector3 p;
                p.x = reader.ReadSingle();
                p.y = reader.ReadSingle();
                p.z = reader.ReadSingle();
                Vector3 r;
                r.x = reader.ReadSingle();
                r.y = reader.ReadSingle();
                r.z = reader.ReadSingle();
                Vector3 s;
                s.x = reader.ReadSingle();
                s.y = reader.ReadSingle();
                s.z = reader.ReadSingle();
                Transform t = Instantiate(cubePrefab);
                t.localPosition = p;
                t.localRotation = Quaternion.Euler(r);
                t.localScale = s;
                cubeList.Add(t);
            }
```

在加载保存的游戏前需要重置当前游戏，设置按下L键调用Load方法

这里需要注意的是，Unity中物体的旋转是由四元数控制的，保存的时候要保存欧拉角的三个值，读取的时候再转化成四元数。



## 3.抽象存储

以上方式虽能读取二进制数细节，但编写单个3D向量需要三个Write调用。保存和加载对象时，若能在更高层次上进行工作，只需一次方法调用就可以读写整个3D向量。数据以什么方式储存都没有关系。游戏不需要知道这些细节。

### 3.1 游戏数据的读取器和写入器

为了隐藏读取和写入数据的细节，我们将自己创建读取器和写入器

以写入器为例，GameDataWriter不需要继承MonoBehaviour，因为我们不会把他附加到游戏对象上，它将充当BinaryWriter的包装器。

```c#
public class GameDataWrite
{
    BinaryWriter writer;

    public GameDataWrite (BinaryWriter writer)
    {
        this.writer = writer;
    }
}
```

重载Write方法写入不同类型的数据。这里旋转选择利用四元数存储。

```c#
    public void Write(float value)
    {
        writer.Write(value);
    }
    public void Write(int value)
    {
        writer.Write(value);
    }
    public void Write(Quaternion value)
    {
        writer.Write(value.x);
        writer.Write(value.y);
        writer.Write(value.z);
        writer.Write(value.w);
    }
    public void Write(Vector3 value)
    {
        writer.Write(value.x);
        writer.Write(value.y);
        writer.Write(value.z);
    }
```

同样的完成一个读取器，充当BinaryReaderd的包装器

```c#
public class GameDataReader 
{
    BinaryReader reader;

    public GameDataReader(BinaryReader reader)
    {
        this.reader = reader;
    }
}
```

向量和四元数根据写入顺序读取数据

```c#
    public float ReadFloat()
    {
        return reader.ReadSingle();
    }
    public int ReadInt()
    {
        return reader.ReadInt32();
    }
    public Quaternion ReadQuaternion()
    {
        Quaternion value;
        value.x = reader.ReadSingle();
        value.y = reader.ReadSingle();
        value.z = reader.ReadSingle();
        value.w = reader.ReadSingle();
        return value;
    }
    public Vector3 ReadVector3()
    {
        Vector3 value;
        value.x = reader.ReadSingle();
        value.y = reader.ReadSingle();
        value.z = reader.ReadSingle();
        return value;
    }
```

### 3.2 持久化对象

在Game中写入立方体的transform数据要简单得多，但我们需要代码看起来更加简洁一些，如果可以简单地调用writer.Write（objects [i]）将非常方便，但这需要GameDataWriter自己区分transform的细节。

换个思路，Game不需要知道如何保存对象，这是对象自己的责任，Game可以使用对象[i] .Save（writer）保存数据。

我们创建一个PersistableObject组件脚本，该脚本知道如何保存和加载该数据，这个脚本将挂载到预制体上

```c#
public class PersistableObject : MonoBehaviour
{
    public void Save(GameDataWrite write)
    {
        write.Write(transform.localPosition);
        write.Write(transform.localRotation);
        write.Write(transform.localScale);
    }
    public void Load(GameDataReader reader)
    {
        transform.localPosition = reader.ReadVector3();
        transform.localRotation = reader.ReadQuaternion();
        transform.localScale = reader.ReadVector3();
    }
}
```

通过预制体创建的对象会附加一个PersistableObject组件。具有多个这样的组件是没有意义的。我们可以通过向类添加DisallowMultipleComponent属性来强制执行此操作。

```c#
[DisallowMultipleComponent]
public class PersistableObject : MonoBehaviour
{
    ......
}
```

### 3.3 持久化存储

现在我们有了一个持久化对象类型，接下来创建一个PersistentStorage类来保存这样的对象。它包含和Game相同的保存和加载逻辑，不同之处在于它仅保存和加载单个PersistableObject实例。将其设为MonoBehaviour，这样我们就可以将其附加到游戏对象上，并且可以初始化其保存路径。

```c#
public class PersistentStorage : MonoBehaviour
{
    private string savePath;

    private void Awake()
    {
        savePath = Path.Combine(Application.streamingAssetsPath + "saveFile");
    }

    public void Save(PersistableObject o)
    {
        using (
            BinaryWriter writer = new BinaryWriter(File.Open(savePath, FileMode.Create)))
        {
            o.Save(new GameDataWrite(writer));
        }
    }
    public void Load(PersistableObject o)
    {
        using(
            BinaryReader reader=new BinaryReader(File.Open(savePath, FileMode.Open)))
        {
            o.Load(new GameDataReader(reader));
        }
    }
}
```

附加此组件到场景中的一个空物体上，它代表了游戏的持久化存储。理论上将可以有多个这样的存储对象，用于存储不同的事物或提供对不同存储类型的访问。

### 3.4 可持久化游戏

为了利用新的可持久对象方法，我们必须重写Game。将预置的对象的内容类型更改为PersistableObject。调整CreateObject使其可以处理此类型更改。然后删除读取写入的方法。

```c#
    public PersistableObject cubePrefab;
    private List<PersistableObject> cubeList;
	......
    private void CreatCube()
    {
        PersistableObject o = Instantiate(cubePrefab);
        Transform t = o.transform;
        t.localPosition = Random.insideUnitSphere * 5;
        t.localRotation = Random.rotation;
        t.localScale = Vector3.one * Random.Range(0.1f, 1);
        cubeList.Add(o);
    }
```

让Game依赖于PersistentStorage实例来处理存储数据的细节。添加此类型的公共存储字段，以便我们可以为Game提供对存储对象的引用。为了再次保存和加载游戏状态，我们让Game本身继承了PersistableObject。然后，它可以使用存储加载并保存自身。

```c#
public class Game : PersistableObject{
    ......
	public PersistentStorage storage;
    ......
    private void Update()
    {
        ......
        else if (Input.GetKeyDown(KeyCode.S))
        {
            storage.Save(this);
        }
        else if (Input.GetKeyDown(KeyCode.L))
        {
            BeginNewGame();
            storage.Load(this);
        }
    }
}
```

![image-20210302170845044](Unity%E5%AF%B9%E8%B1%A1%E7%AE%A1%E7%90%86%EF%BC%88%E4%B8%80%EF%BC%89-%E2%80%94%E2%80%94%E6%8C%81%E4%B9%85%E5%8C%96%E5%AF%B9%E8%B1%A1/image-20210302170845044.png)

### 3.5 重新方法

现在我们保存和加载游戏的时候只是通过PersistentStorage调用了Game继承于PersistableObject的Save和Load方法，即保存自身的transform属性。所以我们需要在Game中重写这两个方法来保存对象列表。

```c#
    public override void Save(GameDataWrite write)
    {
        write.Write(cubeList.Count);
        for(int i = 0; i < cubeList.Count; i++)
        {
            cubeList[i].Save(write);
        }
    }

    public override void Load(GameDataReader reader)
    {
        int count = reader.ReadInt();
        for(int i = 0; i < count; i++)
        {
            PersistableObject o = Instantiate(cubePrefab);
            o.Load(reader);
            cubeList.Add(o);
        }
    }
```

在此之前要将PersistableObject中的Save和Load方法加上virtual关键字

```c#
    public virtual void Save(GameDataWrite write)
    {
        write.Write(transform.localPosition);
        write.Write(transform.localRotation);
        write.Write(transform.localScale);
    }
    public virtual void Load(GameDataReader reader)
    {
        transform.localPosition = reader.ReadVector3();
        transform.localRotation = reader.ReadQuaternion();
        transform.localScale = reader.ReadVector3();
    }
```

