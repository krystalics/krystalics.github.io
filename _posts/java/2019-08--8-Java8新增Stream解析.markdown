[Java 8中的Streams API详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html) 

##### 为什么需要Stream？

需要注意的是这里的Stream与Java.io包中的InputStream和OutputStream是完全不同的两个概念。他并不是用来处理数据的，而是对集合(Collection)对象功能的增强，专注于对集合对象进行各种遍历，高效的聚合操作(aggregate operation)，或者大批量数据操作。

Stream API借助于Lambda表达式，极大的提高编程效率和可读性。同时它提供串行与并行两种模式进聚合操作，并发模式能够充分的利用多核处理器的优势，采用fork/join并发方式来拆分任务和加速处理过程。

##### 什么是聚合操作

这其实是数据库中的概念，比如SUM，AVERAGE等等内嵌函数就是一种聚合，这类的操作放在数据库中执行是可以的，但是很多时候我们进行操作的时候把它放进数据库再取出来速度比较慢，也有些场景是脱离DBMS的。Java在之前并没有提供很多这种辅助类的方法，更多的时候就是采用Iterator来迭代数据，自己写方法来计算

例子：需要对type为grocery的所有交易，进行降序排列。伪代码如下



```java
List<Transaction> groceryTransaction =new ArrayList<>();
//先将所有Tyep==grocery的加进列表
for(Transaction t: transactions){
    if(t.getType()==Transaction.GROCERY){
        groceryTransaction.add(t);
    }
}
// Transaction没有继承Comparatable时，需要手动提供Comparator
Collections.sort(groceryTransaction,new Comparator(){
    public int compare(Transaction t1,Transaction t2){
        return t2.getValue().compareTo(t1.getValue());
    }
});

List<Integer> transactionIds=new ArrayList<>();
for(Transaction t: transactions){
    transactionIds.add(t.getId());
}
```

在Java8中的排序和取值都可以很方便：

```java
List<Integer> transactionIds = transactions.parallelStream().
	filter(t-t.getType()==Transaction.GROCERY).
	sorted(comparing(Transaction::getValue)).reversed()).
	map(Transaction::getId).
	collect(toList());
```







