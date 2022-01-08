---
title: 事件数据-EventData
date: 2021-03-16 21:36:54
tags:
	- Unity
	- UnityEvent
---

**BaseEventData**：基础事件数据——事件数据的基类，和EventSystem配合使用

**PointerEventData**：指针事件数据——鼠标与触摸事件的相关数据（点击、抬起、拖动等），UGUI中大部分事件数据类型都是PointerEventData类型

<!-- more-->

- button：该属性有3个取值。分别是Left（鼠标左键）、Right（鼠标右键）、Middle（鼠标中键）
- clickCount：连续点击的次数
- clickTime：发送点击事件的时间
- delta：当前帧与上一帧的位置差值
- dragging：是否为拖拽状态
- position：指针当前位置
- pressPosition：指针按下时的位置
- scrollDelta：当前帧与上一帧的滚动量差值
- useDragThreshold：是否使用拖动阈值（DragThreshold）
- worldNormal：射线检测到的第一个物体的法线
- worldPosition：射线检测到的第一个物体的世界坐标
- lastPress：最后按下的游戏物体
- pointerDrag：拖拽的游戏物体
- pointerEnter：指针进入的游戏物体
- pointerPress：指针按下时的游戏物体
- rawPointerPress：不管处理不处理按下事件，都会保存指针按下时的游戏物体
- pointerId：指针ID（TouchID，鼠标按键ID）
- pointerCurrentRaycast：指针当前的检测射线
- pointerPressRaycast：指针按下时的检测射线
- enterEventCamera：指针进入时的事件相机
- pressEventCamera：指针按下时的事件相机

**AxisEventData**：轴向事件数据——手柄和键盘中控制轴向相关的事件数据（参考InputManager设置）

- moveVector：原始的输入向量值。即键盘和手柄输入的轴向值（参考InputManager设置）

各分量取值为-1、0、1三个，没有小数部分（因为使用的是Input.GetAxisRaw()函数）

参考StandaloneInputModule.GetRawMoveVector函数的实现和调用位置

- moveDir：移动的方向。公有五个值：Left、Up、Right、Down、None