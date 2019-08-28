## I. 序言
自从 BM 将 DPOS 引入 EOS 以来，如何更好的选出超级节点就成了社区争论的焦点，一票多投，一票多投可弃权制，一票一投等等，BOS 沿用了 EOS 的投票机制即一票多投（一票30投，可弃权）机制，然而由于帕累托法则的存在，导致票权会集中在少数人手中，再加上一票多投的机制允许持票人间可以进行换票，从而最终导致超级节点被控制在少数持票人手中，进而加剧筹码的不平衡，进一步导致贫富差距增大。

为此在投票方案设计中我们尝试着引入改进的“波达计数法”[^1]投票机制来缓解这一问题。“波达计数法”曾是罗马议会采用的投票制度，BET 将采用基于 DPOS 改进的“波达计数法”来进行选举，并以期找到一个更为合理的投票方案，从而在保证 DPOS 系统安全和公平上能做出积极的改进。

## II. 关于股权证明机制和投票选举

关于股权证明机制文章有很多这里不再进行原理性的重复论述。本文仅就筹码集中与安全性方面进行讨论。为便于理解，我们将对上述进行简化：
 
1. 假设所有投票所用票权皆来自投票者本人
2. 将简化原模型为概率发现模型
   
接下来我们假设： 

* 某个持有人持股占比为 Q
* 作弊隐秘度函数 H
* 作弊被发现后损失函数 F(Q）
* 维护系统稳定情形下一般收益函数为 X(Q）
* 作弊收益 W 

*注意：这里假设的函数只用来做定性分析* 

于是我们可以简单的得到该持股人的策略关系即： 

	根据 W+(H-1)*F(Q) 与 X(Q) 的大小关系进而决定相应的策略

因为 F 是 Q 的正相关函数，X 也是 Q 的正相关函数，而 W 是带有投机性的随机参数，因此可以断定随着 Q 的增大其作弊收益会显著降低。在理性自然人假设下其选择维持公正是优势策略，因此从这个角度来看 POS 机制是稳定的模型。  
事实上更进一步的分析时当我们加入道德参数，并将可监督性量化可以的得到股权结构与股东收益之间的如下关系和结论：

A) 股权绝对集中结构  

	股权绝对集中于经营者，则监督困难，经营者道德值减小乃至变为负值。
	经营者责任心(→min)=经营者道德(→-)×{1+经营者资产所有权(max)}
	股东收益(→min)=经营者责任心(→min）×经营者能力×经营条件

B) 股权相对集中结构

     股权相对集中，可以激励经营者，稳定骨干队伍，产生制衡监督。
     经营者责任心(→max)=经营者道德(→max)×{1+经营者资产所有权(max)}
     股东收益(→max)=经营者责任心(→max）×经营者能力×经营条件

C) 股权分散结构

	股权过于分散，监督动力不足，经营者道德值也会减小乃至变为负值。
	经营者责任心(→min)=经营者道德(→-)×{1+经营者资产所有权(min)}
	股东收益(→min)=经营者责任心(→min）×经营者能力×经营条件
   
所以，股权相对集中是优先选择的股权结构（详细的知识请参见股权结构相关书籍和论文）。  

因此我们需要在系统主体稳定的前提下尽量避免大户集权，那么有没有一种选票方案是理想的解决之法呢？ 

### 投票与选举
在详细讨论这个问题之前我们先来尝试着定义一下问题：  

	假设有 N 个人参加选举，共有 K 个人进行投票，且每个投票者心中对于候选人的满意程度存在着一个排列，  
	目的是制定一个选举方案，最终可以得到多数“公正”的投票结果。
 
那么什么是公正的呢？这里我们采用 Arrow 的五条规则[^2]：

1. 每一个选民的影响力都一样，更进一步说，就是非独裁的
2. 选民除了不能不理性之外，没有其他任何关于其心目中排序的限制
3. 如果所有选民都认为 A>B，则选举结果也要显示 A>B
4. 选举结果关于 A 和 B 的排序，应该只由卷宗内 A 和 B 的相对顺序决定，与任何第三者无关
5. 如果所有选民都是理性地将选择对象排序，而且诚实投票，则选举的结果也要显示理性的排序

但不幸的是有多种证明方法[^3][^4][^5]可以证明这样的“公正”是不存在的，这也是 Arrow 不可能定理的内容。因为我们的方案会在“波达计数法”的方案基础上进行修改，这里我们对其情形下的定理给出证明： 


设给定的 k 个整数 m1、m2、...mk，令 σ(1,2,...,n) 为 (1,2,...,n) 的置换群，定义如下映射：
   
![math01](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math01.png)  

对于任意一组数 t1>t2>...>tn，用上述 k 个置换做一个 n×k 矩阵：  

![math02](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math02.png)  

依次定义n个数： 

![math03](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math03.png)  

于是有： 

![math04](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math04.png) 	

我们知道对于上述等式可以找到n组数： 

![math05](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math05.png)   

使得对应的：

![math06](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math06.png)   

满足：

![math07](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math07.png)  


从上面的推导可以看出存在多个解，也就是说“波达计数法”同样是不稳定的。 
那么问题出在哪里呢？事实上是因为我们的假设条件制约了这样的公正性。

学者 Saari[^6] 在他的论文中对上述问题进行了详细的论述。Saari 的论点之一就是 Arrow 的第4、5条的条件会互相「抵销」，因此其对上述第四条规则有进行了修正，如下： 

	4'.选举结果关于 A 和 B 的排序，应该只由卷宗内 A 和 B 的相对顺序和距离决定，与任何第三者无关。
	
那么在这样的假设下 Saari 证明了：
	
	在多于两个候选人前提下“波达计数法”选举程序是『公正』的 
	
具体的技术细节请参考 Saari 的相关论文。也就是说从这个意义上来说“波达计数法”选举程序是更优的。因此我们尝试着在该结论下结合 DPOS 方案进行 BET 投票的适配。

## III. BET 投票方案
接下来我们先给出投票方案的整体描述：

设参加选举人数为N，K个人进行投票。我们的投票规则如下：

1. 对于任意的投票人要求其给出一个序列 Ln，该序列的顺序即为其心目中候选人适合与否的排列顺序
2. 投票人投票权重为其所持有并抵押的 Token 数量记为 Pi
3. 给定一个函数 F(i)，该函数返回对于序列不同序号的打分
4. 对于任意的候选人其记分规则如下：

	Vk = ∑ F(i,j)*Pi 
			
5. 排序所有参加选举候选人，得出最终当选序列。

BET 将采用如下 F 函数：

	F(i) = (N+1-i)/N

证明：

> 假设任何投票人其手中所持 Token 数⌊Pi⌋即是票数，且每次只能投1票， <br>
> 如此可将**每个投票人**看作是一个具有相同投票意愿投票者的集合。 <br>
> 在此假设下，进一步来看这就变成了 ∑⌊Pi⌋ 人进行“波达计数法”投票的模型， <br>
> 因此其具有与“波达计数法”方案相同的理论特性。

进一步来看对于任意的持票人 Pi 由于他需要对所有的参选人都进行投票（排序），也就是说其对于任意一个参选人都做了贡献，只不过贡献的程度是按照函数 F 来进行的，也就是说设 Pi>Pj 且对于任意的 k>l 有 F(k)<F(l)，于是有:
	
	Pi/Pj > (∑F(i) * Pi)/(∑F(i) * Pj)

其中 i，j 分别排在在对方的序列中劣势位置，因此对比一票多投的方案明显的削弱了其票权的作用。  

下面是不同 F 函数下与一票多投控制力进行的对比。

> 线下面积为一票多投制投票者的投票最大值 

![math08](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math08.png)  


> 线下面积为 F 为“波达计数法”时的投票最大值 

![math09](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math09.png)  


> 线下面积为 F 为 X^(-σ) 方案时的投票最大值

![math10](https://raw.githubusercontent.com/boscore/Documentation/master/imgs/dops_borad_count_math10.png)  


虽然并不严谨但可以直观的感受到它们之间的不同之处。接下来我们通过几个简单的例子来说明该方案的特性与优势。

### 案例一 
假设存在三个投票人 A，B，C，为了讨论方便我们假设候选人也是同样的三人，并假设其优先投票给自己，我们的投票准则是在三人中选择两人胜出。  

我们假设 A，B，C 分别持有 Token 数量为 at，bt，ct，假设在 A 与 B 进行串谋，如果采用 EOS 现行的投票方案，只要 `at+bt>ct` 则 C 将丧失了最终的权利，但在 BET 的方案下需要 `at+2bt>2ct`（这里假设 at>bt ），才能得到相应的结果。在各个候选人相差不悬殊的情形下，可以实现 A，C 当选。

### 案例二
假设有一个拥有2000万 Token 的大户 A，另外的一些候选人的支持者只在100万到600万之间，如果采用 EOS 的策略，那么在其他候选人无法达成一致的前提下，A几乎可以控制整个当选人顺序。由于 A 的强大控制力导致许多候选人会选择和 A 进行交易，最终导致 A 的控制力进一步加强。 

但对于 BET 方案来说，情况将变得不一样了，例如我们采用`(n+1-i)/n`作为 F 函数，n 这里假设为25，那么对大户 A 来说其对于某第 i 个支持账户所能提供的贡献值为`(n+1-i)/n*2*10^7` ，例如当 i 为20时其贡献值为480万；也就是说当有一个投票者手中持有的 Token 超过该值时，那么选择和这个 Token 持有者合作是比与 A 合作更优的策略。

因此这能在一定程度上减弱 A 的控制力同时又能保证尽可能多的利益相关者的利益，同时由于该方案是**必需投满**票方案，在理性人假设下投票者为了利益最大化除了投给自己之外一定会将剩余的票权投给对系统稳定性最大的候选人，进而保证系统的安全和稳定。

## IV. 总结
该方案是对现有 EOS 投票方案的进一步探索，借助 Saari 等人的研究成果，以期能使 BET 投票方案更加完善，该方案相较于现行的 EOS 方案有更强的抗大票权者控制能力，同时又保证委托制的安全性。但正如 Arrow 不可能定理说的那样，当限定在某些条件下时完美的投票方案是不存在的。因此我们希望在合理的假设的前提下能够变得更好。


[^1]: [波达计数法，Borda Count](https://en.wikipedia.org/wiki/Borda_count)

[^2]: Kenneth J. Arrow: A Difficulty in the concept of social welfare. In Journal of Political Economy
Vol. 58, No. 4 (Aug., 1950), pp. 328-346

[^3]: B steven j Paradoxes in Politics: An Introduction to the Nonobvious in Political Science

[^4]: Michel A. Habib and Alexander Ljungqvist  Firm Value and Managerial Incentives: A Stochastic Frontier Approach

[^5]: Charles R. Hadlock. • ”Discrete Mathematical Models: With Applications to Social, Biological, and Environmental. Problems” byFred Roberts. ...... Prentice-Hall, Englewood Cliffs, N.J., 1976.

[^6]: D.G.Saari  choice electration  Mathematics and Voting


