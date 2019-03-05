#### 基础算法理解：

##### <a href="#find">查找算法</a>  参考自http://www.cnblogs.com/maybe2030/p/4715035.html ，经过整理后写下

##### <a href="#sort">排序算法</a> 参考自 https://www.cnblogs.com/maybe2030/p/4715042.html 



---

<a id="find">查找算法</a>

​	查找算法分为**静态查找**和**动态查找**，都是针对查找表而言的。动态表指查找表中有删除和插入操作的表。

​	也分为无序查找和有序查找：顾名思义。

​	平均查找长度（Average Search Length，ASL）: 需和指定key进行比较的关键字的个数的期望值，称为查找算法在查找成功时的平均查找长度。对于含有n个元素的查找表，查找成功的平均查找长度为ASL=Pi*Ci 的和 

Pi=查找表中第i个数据元素的概率  

Ci=找到第i个元素时已经比较过的次数。



**顺序查找，**算法简单就不讲了。



**二分查找**：**必须是有序的**，用给定值 k 先与中间节点的关键字比较，中间节点把线形表分成两个字表，若相等则查找成功；不相等，再根据k与该中间节点关键字的比较结果确定一下查找哪个子表，这样递归进行，直到查找结束。

​	**复杂度分析**：最坏情况下，关键词比较次数为log2(n+1)，且期望时间复杂度为O(log2n)

代码入下：

```java

public class Half {

  private int a[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}; //事先排序好的数组

  private int binarySearch(int value) {
    int low = 0, high = a.length - 1, mid = 0;
    while (low <= high) {
      mid = (low + high) / 2;
      if (a[mid] == value) {
        return mid; //要么就 return mid
      } else if (a[mid] > value) {
        high = mid - 1;
      } else {
        low = mid + 1;
      }
    }
    return -1; //没找到就返回 -1
  }

  private int BinarySearch_recursive(int low, int high, int value) {
    int mid = (low + high) / 2;
    if (a[mid] == value) {
      return mid;
    } else if (a[mid] > value) {
      return BinarySearch_recursive(low, mid - 1, value);
    } else {
      return BinarySearch_recursive(mid + 1, high, value);
    }
  }

  public static void main(String[] args) {
    Half h = new Half();
    System.out.println(h.binarySearch(5));
    System.out.println(h.BinarySearch_recursive(0,h.a.length-1,5));
  }

}

```



##### 插值查找：

​	如果说二分查找是只会按照固定步伐行走的机器人，那么插值查找就是能够自由调节步伐的程序。因为无论需要查找什么，二分查找都是从中间开始，然后往复。而**插值查找就是基于二分查找**进行了改善，修改了mid的算法：从原来的1/2 变成了根据参数大小向value靠拢的比例：
$$
二分查找：mid=\frac{low+hith}{2}=low+\frac{1}{2}(high-low) \\
插值查找：mid=\frac{low+high}{某种比例}=low+\frac{value-a[low]}{a[high]-a[low]}(high-low)
$$
这种比例的调整是 **目标值和左界之间的长度** 与 **整个区间的长度** 的比例，**这种插值查找适用于表较长关键字分布比较均匀的查找表**。 我们也可以调整这种比例，根据数据的具体情况改变比例算法，加快查找速度。

复杂度分析：无论成功失败，时间复杂度均为O$(log2(log2n)$ 

代码同上类似，只需要更改mid的算法即可。



##### 斐波那契查找：

​	斐波那契数列：1,1,2,3,5,8,13,21,34,55,89.......（从第三个数开始，后面每个数都是前面两个数的和），我们发现随着数列递增，前后两个数的比值越来越接近0.618。黄金比例在查找技术中仍然充满魅力。

​	基本思想：**还是基于二分查找的一种改进**，通过运用黄金比例的概念在数列中选择查找点（mid）进行查找，提高查找效率。它要求开始**表中记录的个数**比**某个斐波那契数小1** 即  **n=F(k)-1** 开始将k值与第F(k-1)位置的记录比较 即 mid=low+F(k-1)-1 ,比较结果分为三种：

- = ：mid位置即为所求
- &gt; ：low=mid+1，k-=2 
- &lt; ：high=mid-1，k-=1

​       感觉这种算法应用起来复杂，而且时间复杂度也没有比二分查找要好多少，就不具体深究了。知道一下这个概念就好了。



##### 树表查找：

##### 二叉树查找算法：

​	基本思想是：先将要查找的数据 **抽象成二叉树的形态**，**确保树的左分支的值小于右分支的值**，然后再和每个节点的父节点比较大小，查找最适合的范围。这种算法效率很高，但是麻烦的是要先创建一棵树。

​	**二叉查找树：**也叫二叉搜索树，或者是一棵空树又或者具有以下性质：

- 若任意节点的左子树不空，则左子树上所有节点的值均小于它根节点的值

- ​                       右                       右                                       大于    其余部分与上一行类似

- 任意左，右子树也分别为二叉查找树

  而且对它进行中序遍历即可得到有序数列。


​       复杂度分析：和二分查找一样，插入和查找的时间复杂度均为 $O(logn)$，但是在最坏的情况下仍然会有O(n) 的时间复杂度。原因在于**插入和删除元素的时候没有保持二叉查找树的性质**，这就是为什么下面要介绍 平衡二叉树的原因。 （盗图）下图就是一棵二叉查找树，和关于查找（插入与查找差不多），删除时(两个节点的情况)

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/Introduction_to_Algorithm/img/3.png?raw=true">

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/Introduction_to_Algorithm/img/5.png?raw=true">

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/Introduction_to_Algorithm/img/6.png?raw=true">



```java
class Node {  //根据节点 形成一棵树

  private Node parent; //父节点
  private Node left; // 左节点
  private Node right; //右节点
  private int value; // 本身的值

  public Node getLeft() {
    return left;
  }

  public void setLeft(Node left) {
    this.left = left;
  }

  public Node getRight() {
    return right;
  }

  public void setRight(Node right) {
    this.right = right;
  }

  public int getValue() {
    return value;
  }

  public void setValue(int value) {
    this.value = value;
  }

  public Node getParent() {
    return parent;
  }

  public void setParent(Node parent) {
    this.parent = parent;
  }
}


```



```java
import java.util.Arrays;

public class BinaryTreeSearch {
  private Node root; //树的根节点
  private int a[]; //原始数据

  // 构建二叉树，尽量保证根节点的两边 节点一样多。
  public BinaryTreeSearch(int[] array) {
    a = array; // java中数组可以直接赋值引用
    root = new Node(); //初始化 根节点
    int a2[] = a.clone();
    Arrays.sort(a2); //获得排序后的数据，然后选择 中间值做为根节点
    int index = a2.length / 2; //中间值的位置大概是长度的一半
    int mid = a2[index];
    root.setValue(mid);

    generateTree(); //初始化的时候，就生成这颗二叉树
  }

  private void generateTree() {
    // 是通过下面的插入算法 ，从根节点开始，不断将原始数据插入到树的结构中，完成整个树的构建
    for (int value : a) {
      insert(value); //不断将a中的数据插入到 二叉树中
    }
  }

  private void insert(int value) {
    // 建立一个循环，就像查找一个数一样，但是这是插入这个值而不是查找
    Node node = root; // 将其当做循环的节点
    while (node != null) {
      Node parent = node; // 作为父节点，如果向下是空，那么就用parent.setRight(node)或者 parent.setLeft(node)建立联系
      if (value == node.getValue()) {
        return; // 这里暂时处理是，不插入相等的数据，先简单点
      }
      if (value > node.getValue()) {
        node = node.getRight(); //值大于右边节点的 插在右边 节点向右走
        if (node == null) { // 如果右节点为空,就建立一个节点，并赋值
          node = new Node();
          parent.setRight(node); //利用父节点 建立两个节点之间的联系
          node.setParent(parent); //设置父节点
          node.setValue(value);
          return; //插入目的达到，返回
        } // 如果不为空，就继续循环
      } else { // 如果值小于根节点，
        node = node.getLeft(); // 节点向左走
        if (node == null) {
          node = new Node();
          parent.setLeft(node); // 使用父节点，建立左子节点
          node.setParent(parent); //设置父节点
          node.setValue(value);
          return;
        }
      }
    }
  }

  private Node find(int value) {
    Node node = root; // 将其当做循环的节点
    while (node != null) {
      int temp = node.getValue();
      if (value == temp) {
        return node;
      }
      if (value > temp) {
        node = node.getRight();
      } else { // 如果值小于根节点，
        node = node.getLeft();
      }
    }
    return null;
  }

```

```java
// 需要删除某个数据时，情况会变得复杂，
  //   由二叉树的性质可知，最小值即最左节点，最大值为最右节点
  private void delete(int value) {
    //获得引用 ，还需要得到它的父节点。
    Node target = find(value);
    // 如果目标不存在，则返回
    if (target == null) {
      return;
    }
    // 判断目标节点是其父节点的左节点还是右节点，默认为右节点
    boolean right = true;
    if (target.getValue() < target.getParent().getValue()) {
      right = false;  //如果小于父节点的值说明是左节点
    }
    deleteNode(target, target.getParent(), right);
  }

  private void deleteNode(Node target, Node parent, boolean right) {
    // 1. 先判断目标节点  无子树
    if (target.getRight() == null && target.getLeft() == null) { //无子树
      target = null; //直接把这个节点删掉
      return;
    }

    // 2.如果目标节点只有一棵子树
    if (right) { //且如果它是父节点的右子树
      if (target.getRight() == null || target.getLeft() == null) { //只有一棵子树
        if (target.getRight() == null) {
          Node leftNode = target.getLeft(); // 右子树是空的，就将左子树顶上去
          parent.setRight(leftNode);
          target.setLeft(null); // 将原有的连接切断，target就游离在树之外了，最后再赋值null
          target = null;
        } else {
          Node rightNode = target.getRight();
          parent.setRight(rightNode);
          target.setRight(null);
          target = null;
        }
        return;
      }
    } else { //如果目标节点是父节点的左子树，大致流程与上述过程一致
      if (target.getRight() == null || target.getLeft() == null) {
        if (target.getRight() == null) {
          Node leftNode = target.getLeft(); 
          parent.setLeft(leftNode);
          target.setLeft(null); 
          target = null;
        } else {
          Node rightNode = target.getRight();
          parent.setLeft(rightNode);
          target.setRight(null);
          target = null;
        }
        return;
      }
    }
    //两棵子树时
    // 需要找到 目标节点右子树中最小节点。  1.先处理 min节点上的关系
    if(target.getRight()!=null&&target.getLeft()!=null){
      Node min = getRightTreeMin(target);
      if (min.getRight() != null) {
        min.getParent().setLeft(min.getRight()); //因为 min 值一定是在左边,将右边的子树绑定到自己父节点上
        min.getRight().setParent(min.getParent());
      }
      // 2. 开始转移target节点的关系，将 target 节点上的关系全都转到 min上
      min.setRight(target.getRight());
      min.setLeft(target.getLeft());
      target.getRight().setParent(min);
      target.getLeft().setParent(min);
      parent.setRight(min);
      min.setParent(parent);

      target = null; //删除完毕

    }
  }

  private Node getRightTreeMin(Node node) { //获得右子树中最小节点。在树的最左侧
    node = node.getRight(); //先移至右子树根节点上
    Node min = null;
    while (node != null) {
      min = node;
      node = node.getLeft(); //一直往左走
    }
    return min;
  }


  public static void main(String[] args) {
    int a[] = {1, 3, 2, 9, 6, 8, 4, 7, 5, 0};
    BinaryTreeSearch binaryTreeSearch = new BinaryTreeSearch(a);
    System.out.println(binaryTreeSearch.find(3));
    binaryTreeSearch.delete(3);
    System.out.println(binaryTreeSearch.find(3));
    binaryTreeSearch.insert(3);
    System.out.println(binaryTreeSearch.find(3));

  }
}
```



更难的平衡二叉树，红黑树什么的暂时不会。就不写了。



##### 哈希查找

什么是哈希函数？

　　哈希函数的规则是：通过某种转换关系，使关键字适度的分散到指定大小的的顺序结构中，越分散，则以后查找的时间复杂度越小，空间复杂度越高。

​	简单来说就是构建了一个索引序列，让所有与索引相似的组成一行，如下图。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/Introduction_to_Algorithm/img/7.png?raw=true">

​	重点是冲突的解决方式。上图就是简单一点的 线性探测法：先到达表头，然后从那一行开始找，做个循环就可

```java
package com.company.chapter2.find;


class HashNode{
  int value;
  HashNode next;
}

// 这里建立一个以 0~9为索引的hash表，表示数据的个位数
public class Hash {

  private int a[];
  private HashNode head[]=new HashNode[10]; //总共10个索引

  public Hash(int []a){
    this.a=a;
    for(int i=0;i<10;i++){
      head[i]=new HashNode();
      head[i].value=i;
    }
    generateHashTable();
  }

  private void generateHashTable(){
    for (int i:a){
      insert(i);
    } //将数组a中的数据都插入哈希表中
  }

  private void insert(int value){
    HashNode node=new HashNode();
    node.value=value;
    int index=value%10;
    // 将 node 插到head后面，
    node.next=head[index].next;
    head[index].next=node; //指向 该节点
  }

  private HashNode find(int value){
    HashNode head_node=head[value%10];
    while(head_node.next!=null){
      if(head_node.next.value==value){
        return head_node.next;
      }
      head_node=head_node.next;
    }
    return null;
  }

  private void delete(int value){
    int index=value%10;
    HashNode head_node=head[index];
    while(head_node.next!=null){
      if(head_node.next.value==value){
        head_node.next=head_node.next.next;
        return;
      }
      head_node=head_node.next;
    }

  }

  public static void main(String []args){
    int a[]={243,52,64,75,643,86,20,9,78,54};
    Hash hash=new Hash(a);

    System.out.println(hash.find(243));
    hash.delete(243);
    System.out.println(hash.find(243));
    hash.insert(243);
    System.out.println(hash.find(243));

  }

}
```





---

##### <a id="sort">排序算法-基础</a>

​	排序有分为内部排序和外部排序，内部排序是数据记录在内存中进行排序，外部排序因为数据量过大，所以需要访问硬盘。一般来说，我们现在接触到的都是内部排序。

​	按照常见的分类算法还可以将内部排序分为两个大类：比较排序和非比较排序。非比较排序（如计数排序，基数排序，桶排序）时间复杂度很低，为线性复杂度$O(n)$ ，但是这样的排序方法受到的限制比较多。不是通用的，所以主要讲通用的排序算法（插入排序，快速排序，归并排序...）



**选择排序**的思想是最简单的，就是不断的将现有数组中最大的值往末端移，直到循环结束。就不在这里赘述了。

**插入排序**：有很多的优化和改进。先简单介绍一下思想：将数组分为两个部分，前半段有序后半段无序。默认数组中第一个元素是有序的，直接从第二个元素开始，与前面有序集合比较，如果比它们都小，就插入到第一个位置。接着第三个与前面两个比较，这样不断插入直到结束

也很简单，但是很多细节可以优化。比如

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/Introduction_to_Algorithm/img/8.png?raw=true">

```java
private  void sort_insert(int[] A) {
  
    for (int i = 1; i < A.length; i++) { 
      int key = A[i]; //当前元素的值
      int j = i - 1;
      while (j >= 0 && A[j] > key) { 
        A[j + 1] = A[j]; //相当于 前面一个元素的值向后面挪一步，
        j--;
      }
      A[j + 1] = key; 
    // 要么j=-1 跳出循环，证明前面没有比key小的元素，所以第一个元素的值就应该是key
   // 要么是执行到一半有A[j]<=key ，自然 A[j+1]=key
    }
  }

// 上面一段有点麻烦，下面简化程序来自参考文章：
public void InsertionSort(int A[]) {
    for(int i=1; i<A.length; i++)
        for(int j=i-1; j>=0 && a[j]>a[j+1]; j--)
            Swap(a[j], a[j+1]); //用交换函数来代替赋值，也不用临时变量了
}
```



**希尔排序** Shells Sort：

​	是基于插入排序的改进，又叫做缩小增量排序。基本思想是：先将整个待排序的记录序列分割成为若干个子序列分别进行直接插入排序，待整个序列中的记录基本有序时，再对全体进行插入排序。

**算法流程：**

1. 选择一个增量序列t1,t2,.ti,tj...,tk  其中ti>tj，tk=1；
2. 按照增量序列个数k，对序列进行k趟排序
3. **每趟排序根据对应增量ti**，将待排序序列分割成若干个长度为m的子序列。分别对各子表进行插入排序。仅增量因子为1时，整个序列作为一个表来处理。表长度为序列长度。



<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/Introduction_to_Algorithm/img/9.png?raw=true">



**时间复杂度：**$O(n^{1+e})   0<e<1$ ，在**元素基本有序的情况下，效率很高**，不稳定的排序算法

> 相等元素的前后顺序没有改变，就叫做排序是稳定的

这里希尔排序会造成原序列中相等的元素在新序列中顺序与之前不一致，所以是不稳定的排序算法

```java
public class ShellsSort {

  public void shells(int[] A) {
    //1.选择一个增量序列，这里选择 1,2,4...n等比序列，从大的增量n开始,到1结束
    //定义增量为 increment
    for (int increment = A.length / 2; increment > 0; increment /= 2) {
      //对每个增量 进行一次 插入排序，
      for(int i=increment;i<A.length;i++){
        //从第increment开始往前滚，交换后面的值比对应前面小的
        for(int j=i;j-increment>=0&&A[j]<A[j-increment];j-=increment){
          // 交换两者的值
          int temp=A[j-increment];
          A[j-increment]=A[j];
          A[j]=temp;
        }
      }
    }// 增量的循环，
  }

  public static void main(String[] args) {
    int[] A = {5, 2, 4, 6, 1, 3}; //初始的数组
    ShellsSort shellsSort=new ShellsSort();
    shellsSort.shells(A);
    for(int i:A){
      System.out.println(i);
    }
  }
```



**堆排序** Heap Sort

​	堆排序是一种树形选择排序，是对选择排序的改进。图片来自：[一篇博客](https://www.cnblogs.com/chengxiao/p/6129630.html)

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/Introduction_to_Algorithm/img/10.png?raw=true">

​	堆排序的基本思想是：**将待排序序列构造成一个大顶堆**，此时，整个序列的最大值就是堆顶的根节点。**将其与末尾元素进行交换，此时末尾就为最大值。**然后将剩余n-1个元素重新构造成一个堆，重复上述过程。最终得到一个有序序列

具体的详细内容见：[图解排序算法之堆排序](https://www.cnblogs.com/chengxiao/p/6129630.html)



**归并排序**：Merge Sort

[归并排序](https://krystalics.github.io/2019/01/25/Getting-Started/#%E5%BD%92%E5%B9%B6%E6%8E%92%E5%BA%8F) 整理了算法导论和网上的一篇文章(抱歉，已经忘了哪篇)



**快速排序**：Quick Sort

基本思想：**分而治之**

- 先从数列中取出一个数作为**基准数**
- **根据基准数将数列进行分区**，小于基准数的放在左边，大于在右边
- 重复上述操作，直到各区间只有一个数为止

时间复杂度$O(nlogn)$ ，但是初始数列基本有序时，快速排序反而会变成冒泡排序。快速排序图解过程如下：

[图解快速排序算法](https://blog.csdn.net/sfsk_sa/article/details/79544469)  我做了注解：

```java
public void quickSort(int left, int right, int a[]) {
  // 一开始先对整个序列进行初始排序，所以left=0, right=a.length-1。
  // 这里直接就以第一个数为基准数了
  if (left < right) { //这里的意思是，如果该区间只有1个元素就停止排序
    int start = left, end = right, temp = a[start];
    while (start < end) {
      // 从右向左找小于基准值a[start]的数
      while (start < end && a[end] > temp) {
        end--;
      }
      // 将start与end比较是为了得知，右侧如果有比temp小的数就 将其赋值到a[start]中，应为已经有临时变量，所以不用交换
      if (start < end) {
        a[start++] = a[end];
      }//将右侧小的数交换到左侧，下标+1 即往后走一步

      // 右侧找完了之后再从左侧开始找，直到找到左侧比temp大或者没有
      while (start < end && a[start] < temp) {
        start++;
      }
      if (start < end) {
        a[end--] = a[start];
      } //将左侧大的数交换到右侧 然后指标-1，即往前走

    }
    a[start] = temp; //最后将临时变量还给a[start],尽管start已经改变位置了，可是那才是它在这个序列中的位置
    quickSort(left, start - 1, a); //之后 开始对 start的左侧
    quickSort(start + 1, right, a); //右侧 开始进行快速排序
  }
}
```

















































