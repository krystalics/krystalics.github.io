来自：[Java常量理解与总结 ](https://www.jianshu.com/p/c7f47de2ee80)  [深入解析String#intern](https://www.jianshu.com/p/c7f47de2ee80)

 

在Java语言中有8种基本类型和一种比较特殊的类型String。这些类型为了使它们在运行过程中速度更快，更节省内存，都提供了一种常量池的概念，常量池就类似于一个Java系统级别提供的缓存。

8种基本类型的常量池都是系统协调的，`String`类型的常量池比较特殊。它的主要使用方法有两种：

- 直接使用双引号声明出来的`String`对象会直接存储在常量池中。
- 如果不是用双引号声明的`String`对象，可以使用`String`提供的`intern`方法。intern 方法会从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中





##### 什么是常量呢？

用final 修饰的成员变量表示常量，值一旦给定就无法改变！

final 修饰的变量有三种：静态变量，实例变量和局部变量，分别表示三种类型的常量。

##### Class文件中的常量池

在Class文件结构中，最开始的4个字节用于存储 魔数Magic Number，用于确定一个文件是否能够被JVM接受，再接着4个字节用于存储版本号，前两个字节存储次版本号，后两个 存储主版本号，再接着是用于存放常量的常量池，由于常量的数量是不固定的，所以 常量池的入口放置一个U2类型的数据(const_pool_count)存储常量池容量计数值。

常量池主要用于存放两大常量：**字面量(Literal)** **和符号引用量(Symbolic References)**

字面量相当于Java语言层面常量的概念，如文本符串，声明为final的常量值等

符号引用则属于编译方面的概念，包括如下三种类型的常量：

- 类和接口的全限定名
- 字段名称和描述符
- 方法名称和描述符

运行时常量池相对于CLass文件常量池的另外一个重要特征是**具备动态性**，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入CLass文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用比较多的就是**String类的intern()**方法。



##### 常量池的好处

常量池是为了避免频繁的创建和销毁对象而影响系统性能，其实现了对象的共享。例如字符串常量池，在编译阶段就把所有的字符串文字放到一个常量池。

1. 节省内存空间：常量池中所有相同的字符串常量被合并，只占用一个空间
2. 节省运行时间：比较字符串时，==比equals快。对于两个引用变量，只用==判断引用是否相等，也就可以判断实际值是否相等。



理论讲完了，是时候来点实际的了：

java中的基本类型的包装类大部分都实现了常量池技术：Byte  Short Integer  Long  Character  Boolean;  而且它们包装的数值在 [-128,127]之间 超出了这个范围任然会创建新对象。

```java
 	Integer i1=127;
    Integer i2=127;
    System.out.println(i1==i2);  //true
    i1=-128;
    i2=-128;
    System.out.println(i1==i2);  //true
    i1=128;
    i2=128;
    System.out.println(i1==i2);  //false
    i1=-129;
    i2=-129;
    System.out.println(i1==i2); //false

    // Double 不支持常量池技术
    Double d1=1.4;
    Double d2=1.4;
    System.out.println("double  "+(d1==d2));  //double  false

    String s1="s";
    String s2="s";
    System.out.println("string  "+(s1==s2));  //string true

```

如果情况复杂一点：**加上 intern()方法** 和String

```java
String s1="ab";
String s2=new String("ab");
System.out.println(s1==s2); //false
```

这两种不同的创建方法是有差别的，第一种方式是在常量池中拿对象，第二种方式是直接在堆内存空间创建一个新的对象。
 **只要使用new方法，便需要创建新的对象。**

 **连接表达式 +**
 （1）只有使用引号包含文本的方式创建的String对象之间使用“+”连接产生的新对象才会被加入字符串池中。
 （2）对于所有包含new方式新建对象（包括null）的“+”连接表达式，它所产生的新对象都不会被加入字符串池中。

```java
	String hello ="hello";
    String helloWorld ="hello world";  //这两个字符串都被加入到常量池中
    System.out.println(helloWorld =="hello"+" world"); //用加法创建的新对象加入到常量池中时，发现已经有该值了，于是返回的引用与之前的一致。
    System.out.println(helloWorld == hello +" world"); // 这个有变量加入，创建的对象不能够在运行时加入到常量池中，所以引用不一样
// 而下列代码使用了 intern方法 可以动态加载到常量池中
    System.out.println(helloWorld.intern()==(hello +" world").intern());
```

**intern**：如果常量池中存在当前字符串, 就会直接返回当前字符串. 如果常量池中没有此字符串, 会将此字符串放入常量池中后, 再返回











































