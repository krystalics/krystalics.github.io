集合类库知识

---

##### **1.队列(queue)**

可以在尾部添加元素，头部删除元素，并且可以查找队列中元素的个数。它最简形式可能是这样的：并没有涉及到队列是如何实现的》》

```java
public interface Queue<E>{
    void add(E element);
    E remove();
    int size();
}
```

可以通过1. **循环数组** 和 2. **链表** 两种方式实现 。在JDK1.8中 它继承了Collection类

```java
public interface Queue<E> extends Collection<E>{
    
}
```



##### **2.Collection接口**

在Java类库中，集合类的基本接口是Collection接口，在jdk中它继承了Iterable

```java
public interface Collection<E> extends Iterable<E> {
    
}
```

它里面有两个最基本的方法

```java
public interface Collection<E>{
    //向集合中添加元素，如果改变了集合就返回 true ，如果集合没有变化就返回 false
    //如果在添加元素的时候，集合中已经有了该元素，就会返回false
    boolean add(E element); 
    Iterator<E> iterator(); 
}
```



##### 3.Iterator接口与Iterable接口

```java
public interface Iterator<E>{
//通过反复调用next方法，可以逐个访问集合中的每个元素。到末尾会抛出 NoSuchElementException
    E next(); 
//所以需要 hasNext()方法来确认。
    boolean hasNext();
//Iterator的remove方法很不一样，它必须在next()方法之后使用，删除掉刚刚next()的元素。
//如果没有接着next()方法，直接使用remove()就抛出 IllegalStateException异常
    void remove(); 
//提供forEach，lambda表达式
    default void forEachRemaining(Consumer<? super E> action);
}
```

如果想要查看集合中的所有元素，就请求一个迭代器。并在`hasNext()`返回true时反复调用`next()`方法。

```java
Collection<String> c=...;
Iterator<String> iter=c.iterator();
while(iter.hasNext()){
    String element=iter.next();
    // ...
}

//简单来写，还可以有另一个for each写法
for(String element:c){
    //
}

// Iterator 本身也提供一个更加现代的方法 lambda表达式
iter.forEachRemaining(element->do something with element);
```

`for each` 循环可以与任何实现了`Iterable`接口的对象一起工作，而`Collection`刚好实现了`Iterable`接口。jdk实现入下：

```java
public interface Iterable<T> {
    
    Iterator<T> iterator();  
    
// 提供forEach方法
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }

    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```



**元素访问的顺序取决于集合类型**，对于ArrayList 进行迭代，迭代器将从索引0开始，每迭代1次，索引值加1。HashSet中的元素则按照某种随机的次序出现。



##### 4.泛型实用方法 与 AbstractCollection的诞生

Collection和Iterator都是泛型接口，可以编写操作任何集合类型的实用方法。这里实用方法是指开发中经常会使用的方法，比如Collection中的部分方法

```java
int size();
boolean isEmpty();
boolean contains(Object obj);
boolean containsAll(Collection<?> c);
boolean equals(Object other);
boolean remove(Object obj);
void clear();
...
```

如果让每一个实现Collection接口都要提供如此之多的方法是做重复的工作，所以为了能够让实现者更容易地实现这个接口——Java类库提供了AbstractCollection , 它将很多实用方法实现了，而其他想要实现Collection接口的都可以直接`extends AbstractCollection` 。



##### 5.集合框架中的接口 结构图

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/11.png?raw=true">

集合中有两个基本接口，Collection和Map。两个体系的一些不同：

Collection中添加元素 是  add(E element) 。

不过由于Map是键值对，所以用 put(K key,V value)来添加元素  V get(K key)获取元素

注意游离于几个体系之外的是 RandomAccess  ，它的作用是测试某个集合是否支持高效的随机访问，就像数组一样可以用下标任意顺序访问元素。里面没有任何方法

```java
if(c instanceof RandomAccess){
    user random access algorithm
}else{
    
}
```



List接口就提供了可以随机访问的方法

```java
void add(int index,E element);
void remove(int index);
E get(int index);
E set(int index,E element);
```



坦率的讲集合框架中的访问设置的并不是很好。其中有两种有序集合 List(由数组支持，所以可以提供随机访问) 和 LinkedList 是链表有序，但是随机访问很慢，最好使用迭代器来访问。



##### 6.Set

基本等同于 Collection接口，但是它定义的行为更加严谨。

add()方法不允许增加重复元素，而且重新定义了equals()方法，认为两个集合只要包含同样的元素就认为是相同的，不要求顺序相同：与equals()匹配的是hashCode()方法要定义保证包含相同元素的两个集合会得到相同的散列码



##### 7.具体集合接口

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/12.png?raw=true">

现在具体的介绍几个常用的集合类型：

ArrayList：一种可以动态增长和缩减的索引序列

LinkedList： 一种可以在任何位置进行高效地插入和删除操作的**有序**序列

HashSet： 一种没有重复元素的**无序集**合

TreeSet：一种**有序集**



HashMap：一种存储键/值关联的数据类型

TreeMap：一种键值有序的排列的映射表



##### 7.1	LinkedList

链表与泛型集合之间有个重要区别。链表是一个有序集合，每个对象的位置非常重要。 LinkedList.add() 将元素添加到链表末尾，但是我们常常需要将元素添加到链表中间，所以需要依赖迭代器来添加；如果不需要有序像是set那样，就不用借助迭代器了。

链表提供 ListIterator 对象，可以双向遍历自身。它的

`add()`方法是将元素添加到"光标"(**Java的迭代器是位于两个元素之间的，类似于光标：C++就是指向元素**)处，

`remove() `是删除在它之前光标经过的元素，可能是`next() `也可能是 `previous() `。

`set()`方法用一个新元素取代 `next()|previous()`方法返回的元素

```java
List<String> list=new LinkedList<>();
ListIterator<String> iter=list.listIterator();

String oldValue=iter.next();
iter.set(newValue); //set方法用newValue取代在链表中的oldValue
```



重点：如果某个迭代器修改集合时，另一个迭代器对其进行遍历，就会出现混乱的状态：

```java
// 获得两个迭代器
ListIterator<String> iter1=list.listIterator();
ListIterator<String> iter2=list.listIterator(); 

iter1.next();
iter1.remove();  //第一个迭代器把第一个元素给删了，由于iter2检测出该链表从外部被修改了
iter2.next(); //调用next()时抛出  ConcurrentModificationException异常
```

为了避免发生并发修改的异常，我们可以简单遵循一些编程规范：**一个容器可能会有很多迭代器，但是这些迭代器只能读取列表；另外专门提供一个能够读写的迭代器。**

还有一种简单的方法可以检测并发修改的问题：**集合可以跟踪改写（只有add，remove：set不算）操作的次数**，每个迭代器都维护一个独立的计数值，在每个迭代器方法的开始检测自己的改写操作的计数值是否与**集合**的改写操作计数值相同。如果不一致就抛出`ConcurrentModificationException`异常。**即如果有其他迭代器修改集合，集合的计数值会升高导致与本迭代器数值不一致。**

由于链表的数据结构限制，所以不能够随机访问链表中的某一个元素，只能通过迭代器遍历比对。尽管LinkedList有提供一个get(index)方法获取对应位置的元素，那也只是包装了迭代器（**这个方法也做了微小的优化，如果Index>list.size()/2 就从列表尾部开始搜索元素**），效率也不高。如果发现自己正在使用这个方法，很可能要解决的问题使用了错误的数据结构（即，不应该使用链表来存储数据）

有了上一段的知识，可以预想到下面一段的代码效率有多低了：

```java
//相当于 二重循环。
for(int i=0;i<linkedlist.size();i++){
    int value=linkedlist.get(i);
}
```



##### 7.2	ArrayList

ArrayList封装了一个动态再分配的对象数组。

>对于一个经验丰富的Java程序员来说，需要动态数组的时候可能会使用 Vector 类。为什么要使用ArrayList呢？因为Vector类的所有方法都是同步的，即被synchronized修饰。可以由两个线程安全地访问一个Vector对象。但是如果由一个线程访问Vector，代码要在同步操作上耗费大量时间。ArrayList不是同步的，因此在不需要同步时，使用ArrayList



##### 7.3	散列集 

hash table是一种能够快速查找所需要对象的一种数据结构。原因是：散列表为每个对象计算一个整数，称之为散列码(hash code) 。散列码是根据对象的属性产生的一个整数。比如String类的hashCode计算方式如下：

```java
private int hash;//默认值是0

public int hashCode() {
    int var1 = this.hash; 
    if (var1 == 0 && this.value.length > 0) {
      char[] var2 = this.value; 
// 字符串里面的每一个元素 都参与运算，最终得出 hash值
      for(int var3 = 0; var3 < this.value.length; ++var3) {
        var1 = 31 * var1 + var2[var3];
      }

      this.hash = var1;
    }

    return var1;
  }

//定义了 hashCode 就必须定义配套的 equals
public boolean equals(Object var1) {
    if (this == var1) { //如果是和自己比
      return true;
    } else { 
      if (var1 instanceof String) {// 如果var1是字符串这一类的
        String var2 = (String)var1;
        int var3 = this.value.length;
        if (var3 == var2.value.length) { //先比较长度
          char[] var4 = this.value;
          char[] var5 = var2.value;
//  对比每一个元素是否相同
          for(int var6 = 0; var3-- != 0; ++var6) {
            if (var4[var6] != var5[var6]) {
              return false;
            }
          }

          return true;
        }
      }

      return false;
    }
  }
```



在Java中，散列表用链表数组实现：就是传统的拉链法解决冲突。每一个列表都被称之为 桶(bucket)，插入对象的时候先计算出该对象的散列码然后对桶数取余。例如某个对象的散列码是 76268，并且有128个桶。对象应该保存在第108号桶中，如果桶中没有其他元素直接插入就可以。

有时候桶被占满了，这种情况被称之为散列冲突(hash collision) 。这时，需要用新对象与桶中的所有对象进行比较，查看这个对象是否存在。(在java SE 8中，桶满时会从链表变为平衡二叉树。如果选择的散列函数不当，可能会产生很多冲突)

如果我们想要优化散列表的性能，可以指定一个初始的桶数。如果大致知道会有多少个元素要插入散列表中，可以将**桶数设置为 元素个数的 75%~150%** 。简单来说，如果大约100个元素，那么桶数就在 75~150之间。**如果散列表太满，就需要再散列(rehashed)**。如果要对散列表再散列，就需要创建一个桶数更多的表并将所有元素插入到这个新表中。

**装填因子**(load factor)默认值为**0.75**，表示当**表中超过75%的位置被占用时**，这个表就会用**双倍的桶数自动进行再散列。**HashSet就是基于散列表的集



##### 7.4	TreeSet树集

TreeSet类与散列集十分类似，不过有所优化。树集是一个有序集合，可以以任意顺序将元素插入到集合中，会自动进行排序。**数据结构是红黑树，每次插入时都被放在正确的位置。**

```java
SortedSet<Integer> sortedSet=new TreeSet<>();
sortedSet.add(1);
sortedSet.add(5);
sortedSet.add(2);
for(int i:sortedSet){
    System.out.println(i);
}
```

要使用树集，该对象必须能够进行比较，这些对象必须实现 `Comparable`接口(声明这些类是可比较的)，或者构造时必须提供一个`Comparator`(即提供一个比较的机制)

```java
class Item implements Comparable<Item>{
  private int id;
  private String description;

  Item(String description,int id){
    this.description=description;
    this.id=id;
  }

  public String getDescription() {
    return description;
  }

  public String toString(){  //print时会自动调用 toString方法
    return "[description="+description+",id="+id+"]";
  }

  @Override  // 默认按照 id 来比较  
  public int compareTo(Item antherItem) {
    return Integer.compare(this.id, antherItem.id);
  }
}
```

构造Item，用于两个比较机制的对照：

```java
SortedSet<Item> parts=new TreeSet<>();  //默认顺序是根据 id
parts.add(new Item("Toster",1234));
parts.add(new Item("Widget",2421));
parts.add(new Item("Modem",5353));

//我们自己定义一个比较器,写法比较特别，用到了 Item:: 和一个方法
// NavigableSet 包含一些 遍历和搜索的有序集的方法，本来应该是包含在SortedSet中的，但是没有
NavigableSet<Item> sortByDescription=new TreeSet<>(Comparator.comparing(Item::getDescription)); 

sortByDescription.addAll(parts);//添加一样的元素

System.out.println(parts);
System.out.println(sortByDescription);
```





















