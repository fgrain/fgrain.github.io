---
title: 五子棋AI
date: 2021-03-10 16:10:21
tags:
	- 算法
---

**需要用到的知识**
博弈树
极大极小值搜索算法 
评估函数
Alpha-Beta剪枝算法 

<!-- more-->

### 博弈树

博弈树是指由于动态博弈参与者的行动有先后次序，因此可以依次将参与者的行动展开成一个树状图形，顾名思义，博弈树就对博弈双方以相同的次数轮流进行决策行为的图形描述。对于任何一种博弈竞赛，我们都可以将其构建成一颗博弈树，其节点对应某一种局面，其分支表示进行一步决策，博弈树的根表示开始位置，叶子节点表示博弈到此结束。

![v2-10ffcd50714f92016883fdfff851d10a_r](%E4%BA%94%E5%AD%90%E6%A3%8BAI/v2-10ffcd50714f92016883fdfff851d10a_r.jpg)

### 极大极小值搜索算法 

我们需要对博弈树进行深度搜索，对此采用极大极小值搜索算法。

假设在博弈树中，从根节点为0开始，奇数层表示电脑可能的走法，偶数层表示玩家可能的走法。我们规定对电脑越有利，分数越大，对玩家越有利，分数越小，分数的起点是0。

电脑走棋的层我们称为 MAX层，这一层电脑要保证自己利益最大化，那么就需要选分最高的节点。

玩家走棋的层我们称为MIN层，这一层玩家要保证自己的利益最大化，那么就会选分最低的节点。

![v2-09b098668cdfd678a5d4e5ca2894f40a_b](%E4%BA%94%E5%AD%90%E6%A3%8BAI/v2-09b098668cdfd678a5d4e5ca2894f40a_b.png)

此图中甲是电脑，乙是玩家，那么在甲层的时候，总是选其中值最大的节点，乙层的时候，总是选其中最小的节点。

也就是说在电脑落子前会遍历所有可走的位置，模拟玩家落子，预判多步以后的局面，找出最有利于自己的一步棋。

伪代码：

```c
function minimax(node, depth, maximizingPlayer)
     if depth = 0 or node is a terminal node
         return the heuristic value of node
 
     if maximizingPlayer
         bestValue := ?∞
         for each child of node
             v := minimax(child, depth ? 1, FALSE)
             bestValue := max(bestValue, v)
         return bestValue
 
     else    (* minimizing player *)
         bestValue := +∞
         for each child of node
             v := minimax(child, depth ? 1, TRUE)
             bestValue := min(bestValue, v)
         return bestValue
```



### 评估函数

每一种棋形的收益不同，这里建立一个评分表来对各种可能会出现的棋形做出评估，五子连珠时分值最大

```c#
	private Dictionary<string, int> scoreTable = new Dictionary<string, int>();
		scoreTable.Add("aa___", 10);                      //眠二
        scoreTable.Add("a_a__", 10);
        scoreTable.Add("___aa", 10);
        scoreTable.Add("__a_a", 10);
        scoreTable.Add("a__a_", 10);
        scoreTable.Add("_a__a", 10);
        scoreTable.Add("a___a", 10);

        scoreTable.Add("__aa__", 600);
        scoreTable.Add("_aa__", 600);
        scoreTable.Add("__aa_", 600);                   //活二 "_aa___"
        scoreTable.Add("_a_a_", 500);
        scoreTable.Add("_a__a_", 500);

        scoreTable.Add("a_a_a", 1000);
        scoreTable.Add("aa__a", 1000);
        scoreTable.Add("_aa_a", 1000);
        scoreTable.Add("a_aa_", 1000);
        scoreTable.Add("_a_aa", 1000);
        scoreTable.Add("aa_a_", 1000);
        scoreTable.Add("aaa__", 1000);                     //眠三

        scoreTable.Add("_aa_a_", 5000);                    //跳活三
        scoreTable.Add("_a_aa_", 5000);

        scoreTable.Add("_aaa_", 9000);                    //活三

        scoreTable.Add("_aaaa", 15000);
        scoreTable.Add("aaaa_", 15000);
        scoreTable.Add("a_aaa", 15000);                    //冲四
        scoreTable.Add("aaa_a", 15000);
        scoreTable.Add("aa_aa", 15000);

        scoreTable.Add("_aaaa_", 100000);                 //活四

        scoreTable.Add("aaaaa", int.MaxValue);                
```

### Alpha-Beta剪枝算法 

如果要搜索所有节点无疑会花费大量的时间，Alpha-Beta剪枝算法用于搜索树时可以裁剪没有意义不需要搜索的分支，以提高运算速度。

基本思想
借助极大极小搜索的特点，MAX节点对应A值，不会下降，MIN节点对应的B值不会上升。这里的A-B值分别指倒推时对应节点的值。

对于MAX来说，其某个后继节点MIN的B值小于该MAX或者更早的先辈节点的A值时，停止对该MIN节点的搜索。同理，当某个MAX节点的A值大于等于其先辈MIN节点的B值时，停止对该节点的搜索。由此，我们可以得到，A-B必须要求搜索树某一部分达到最深计算出一部分MAX的A值或者MIN的B值，随着搜索，不断改进A-B值。

* 剪枝规则总结

1.    A剪枝：任何MIN的B小于任何他的父节点MAX的A时，停止该MIN以下的所有搜索，这个MIN的最终倒推值为当前的B值。（该B可能与真正的不同，但是没关系，不影响最终结果）

2.    B剪枝：任何MAX的A大于或者等与他的父节点MIN的B时，停止该MAX节点以下的所有搜索，这个MAX的最终倒推值为当前的A值。
![v2-34e744db9d47deca9846f3e978c9e35c_b](%E4%BA%94%E5%AD%90%E6%A3%8BAI/v2-34e744db9d47deca9846f3e978c9e35c_b.png)