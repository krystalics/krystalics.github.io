### 这是算法导论第3版的第一章的练习题：

---



##### 1.1-1 给出一个需要排序或需要计算凸包(convex hull)的真实示例

​	凸包(Convex Hull)是一个计算几何（图形学）中的概念。用不严谨的话来讲，给定二维平面上的点集，凸包就是将最外层的点连接起来构成的凸多边型，它能包含点集中所有点的。严谨的定义和相关概念参见[维基百科：凸包](http://zh.wikipedia.org/zh-cn/%E5%87%B8%E5%8C%85)。

​	**答案：** 考试成绩出来了，需要按照高到低的顺序排序。至于凸包，不是很了解



##### 1.1-2 除了速度之外，在现实环境中可以使用哪些其他效率指标？	

​	**答案：** 空间复杂度，



##### 1.1-3 选择您之前看到的数据结构，并讨论其优势和局限性。

**​	答案：** 比如栈和数组吧：栈 先进后出，只有一个出口的数据结构，而数组可以任意读写其中每个单元。

​	栈：提供后进先出的存取方式，缺点是存取其他项很慢

​	数组：插入块，如果知道下标存取也非常快。缺点是查找慢，删除慢，大小固定。



##### 1.1-4上面给出的最短路径和旅行商问题有哪些相似之处？他们有什么不同？

​	**答案：** 二者都是需要抽象成图的思想，

​	不同：最短路径是要求起点到终点之间最短的一条路径。旅行商是要走遍图中的每一个点，又不走重复，回到原点。



##### 1.1-5 想出一个现实问题，只有最好的解决方案才能解决。然后提出一个“近似”最好的解决方案就足够了

​	问题1：找出100个点中，距离最短的两个点。

​	问题2：考哪所大学呢？清华吧，近似最优解了。



---

##### 1.2-1 在应用层需要算法内容应用的一个实例，并讨论涉及的算法和功能

​	**答案：** 在使用高德地图的时候，我想知道去某个地点坐公交花时间最少的路径。涉及到了路径的产生算法，还有简单的加法。



##### 1.2-2 假设我们正在比较插入排序与归并排序在相通机器上的实现，对于规模为n的输入，插入排序运行8n^2步，而归并排序运行64nlgn步。求出两者速度相同时的n大小

​	**答案：** 直接列一个方程，8n^2=64nlgn ，求出方程的解即可。具体过程就不赘述了



##### 1.2-3 n最小为何值时运行时间 100n^2  的一个算法快于相同机器下运行的2^n 运行时间的另一个算法

​	**答案：** 直接列一个方程，100n^2=2^n ，求出方程的解即可。具体过程就不赘述了



---

#### 思考题

1-1 (运行时间比较) 假设求解问题的算法需要f(n) 毫秒，对下表中的f(n)和时间t，确定可以在时间t内解决的问题的最大规模 n:

​	**分析过程：** f(n)=t，以秒为例  f(n)=1000 ，代入f(n)求出n即可

|        | 1 秒钟  | 1分钟 | 1小时 |
| ------ | ------- | ----- | ----- |
| lg n   | 2^1000  |       |       |
| 根号 n | 1000000 |       |       |
| n      | 1000    |       |       |
| n^2    | 33.3... |       |       |
| n^3    | 10      |       |       |
| 2^n    | lg 1000 |       |       |
| n!     |         |       |       |

​	