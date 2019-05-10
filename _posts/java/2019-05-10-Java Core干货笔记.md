自昨日拿到Java核心技术，粗略翻了几十页。就已经发现了好多个之前不知道的点，所以就想着把这些点记录下来，就当做读书笔记了。

---

##### Tip1：关于JIT

​	Java虚拟机可以**将执行最频繁的字节码序列翻译成机器码**，这一过程被称为**即时翻译**(Just in time , JIT技术)。就像是C++中的inline函数，JVM还可以检测指令序列的行为，从而增强安全性。这是java为高性能做出的优化。关于JIT和这部分的内容还来自 [深入浅出JIT](https://www.ibm.com/developerworks/cn/java/j-lo-just-in-time/index.html) 

首先我们大家都知道，javac(java的编译器)将java程序编译成字节码，JVM通过解释器将字节码翻译成对应的机器指令，逐条读入，逐条解释翻译。很显然，经过解释执行，其执行熟读必然会比可执行的二进制机器码要慢很多。为了加快速度，所以引入了JIT。

在运行时JIT会把翻译过的机器码保存起来，以备下次使用（对应开头说的，将执行最频繁的字节码序列翻译成机器码）。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/1.png?raw=true">

*这篇文章讲的非常的好，虽然我看不太懂(尴尬了)，但是根据我博览各种各式的文章经验来看，这样的内容绝对是优秀的。*



##### Tip2：关于基本数据类型

​	在Java中int类型永远都是 4个字节(32位)，而学过C/C++的同学们都知道，它们的数据类型如int的字节数是取决于机器的CPU，所以在很多程序中会出现在一个机器上运行的好好地，换一个环境内存就溢出了。而Java的数据类型具有固定的大小，为它的可移植性助力。顺便说一句，java并没有C++中的unsigned 修饰符。

| 类型   | 存储需求 |
| ------ | -------- |
| int    | 4字节    |
| short  | 2字节    |
| long   | 8字节    |
| byte   | 1字节    |
| float  | 4字节    |
| double | 8字节    |

很意外是不是，long的长度竟然和double是一样的，我一直以为double比long长，(by the way: x=2.4 默认是double类型，x=2.3f 有f后缀才是float类型) ，长整型数值后面也会有L后缀，x=300000000333L。在很多时候我们并不会使用byte和short类型，它们应用于特殊的场景，例如，底层文件的处理或者需要控制占用大量的存储空间量的大数组。

java中还可以直接赋值 十六进制数，a=0xAB13 。八进制容易混淆所以不常用，也可以在数字中加入_ 表示数值更清晰，int x=1_000_000; 编译器会自动忽略_   。 十六进制中表示指数用 **p ,如 a=0x1.0p-3  指数用十进制表示**

Double.POSITiVE_INFINITY 正极值，Double.NEGATIVE_INFINITY 负极值。尽管double的精度已经很高了，但是在无法接受舍入误差的金融计算领域仍然是不够的，所以java提供了BigDecimal类，用于无误差计算

char原本用于表示单个字符，不过现在用来表示Unicode字符，有时Unicode需要两个char值来表示，像是中文，boolean布尔类型只有两个值：true和false ，不能用整型来赋值 如 boolean s=0; 是错的，C++是可以的。

对于浮点数的算术运算，实现这样的可移植性是很困难的。double类型使用64位存储一个数值，而有些处理器使用80位的浮点寄存器。这些寄存器增加了中间计算过程的计算精度，将结果存在**80位的寄存器中，最后将其截断为64位存进内存，这和64位机器上的计算结果不一致。**所以jvm规定所有的运算都必须进行截断，而有时不能进行精确运算的结果会溢出，截断操作耗时间所以效率也比较慢，于是规定使用 `strictfp` 关键字来说明该区域（类，方法）中的所有运算都必须使用严格的浮点运算。





##### Tip3：关于final

​	声明一个变量必须用赋值语句对变量显示初始化，如果没有编译不能通过。java不区分声明和定义。使用final来定义常量而不是const，**final如果修饰类**，那么这个类不可以被继承，**final如果修饰方法**，那么这个方法不可以被重载，**final如果修饰变量**，那么这个变量不能被改变。

在java中还有finally，finalize 两个关键字，它们和final的差别还是很大的。finally是try{}catch{}finally{} 捕获异常的代码块中最后无论有没有异常，都必须要执行的区域。finalize：是Java垃圾回收前，必须执行的代码块



##### Tip4：关于Math和运算

取模运算可以适配除了boolean类型的其他数据，还支持负数的取模。很多时候我们需要使用Math.里的方法，但是一般我们都是这么用的	`Math.pow(2,Math.sqrt(2))` 在这一行代码中就写了两个Math，比较麻烦，如何引入Math呢，	`import static java.lang.Math.*;`

Math类中的哪些静态方法都是为了追求速度，为了达到最快的性能，所有的方法都使用计算机浮点单元中的例程。如果需要的是一个完全可以预测的结果比速度更重要的话，可以使用**StrictMath** 类。

数值转换是我们经常遇到的问题，计算的时候java会自动帮助我们转换它们的类型，按照顺序进行，

1. if (操作数中有double) 其他操作数会转为double

2. else if (操作数中有float) 其他操作数会转换成float
3. else if (操作数中有long) 其他操作数会转为long
4. else if (操作数中有int)  其他操作数会转为int

很明显顺序是  double>float>long>int  

强制类型转换，例如 (int) 3.7  会造成精度损失，结果为3，如果想让其损失的小点，那么可以加上 (int) Math.round(3.7)  结果是4 ，因为round函数返回的是long所以还是要强制转换。**如果将其转换时超出了表示范围，如 (int) 294883109313904  ,就会造成溢出，实际值会发生变化。**



##### Tip5: 关于 位运算和逻辑运算

`&&`和`||`都是按照‘短路’方式来求值的。只要第一个表达式的值确定了，第二个就不用计算了。比如：`4>2||2<3`其实只需要判断第一个如果是正确的就不用继续，错误的话才会去判断第二个。&&计算也类似，就不赘述了。而单个符号 &和|都是需要判断完整的答案才可以。&&的优先级比||高

`&`和`|`还是位运算的符号，~ not  ^ 异或。<< 左移，>>右移用符号位填充高位，  >>>也是右移，不过它会用0来填充高位，而不是符号位。而且在进行位移运算时，如果是int类型的(比如 1<<35 , 实际上得到的值是 1<<3 ， 因为它实际上是溢出了，重新填充的，相当于35模32的运算)。



##### Tip6：关于String

像JS这样的动态语言一样，Java的String类型也有提供组合字符串数组的方法，不过不在对象中，而是String的静态方法，String.join("reg","",""....)  如下：

```java
String []s={"I","love","you"};
String ss=String.join("**",s);
// I**love**you
```

相信大家应该都知道，在Java中String是个不可修改(immutable)的对象，我们每次对它进行修改实际上是创建了一个新的String对象，这对需要频繁操作String的一些方法来说效率太低了。所以Java提供了两个新的可以修改的字符串对象：StringBuffer(很多方法都有synchronized，线程安全) ，StringBuilder(适用于单线程) 实际用法都差不多。

有时要检查一个字符串既不是null也不为空串，要先判断它不为null，因为判断空串需要在不是null的前提下：`if(str!=null&&str.length()!=0)` 。字符串实际上是char值序列(实现了CharSequence接口)，之前的tip2中有提到过**char是采用UTF-16编码表示Unicode码点的代码单元。**什么是码点呢？其实就相当于ASCII中 字符A对应数字 65一样，是字符在UTF-16编码中的编号。有时候特殊的字符需要有两个码点来表示，所以在细节上会有一些差别。

在String方法中提供了好几种有关于codePoint （码点）的方法 比如：`ss.codePointAt(0))` 就是**返回ss字符串中第一个字符的码点**，由于关于码点的方法不多，这里顺便就介绍了吧；

```java
String hello="hello";
int n=hello.length();  // n=5
//实际的长度，要看字符串中是否有需要两个码点表示的字符，
int npCount=hello.codePointCount(0,n); //从0到n字符之间的码点数
//要想得到第 i  个码点，
int index=hello.offsetByCodePoints(0,i); //hello.codePointCount(0,i) 类似 
int cp=hello.codePointAt(index); 

```

再介绍下StringBuilder的一般使用流程

```java
StringBuilder builder=new StringBuilder();
builder.append('char'); //增加单个字符
builder.append("string");  //增加字符串

String s=builder.toString(); //变为字符串。
```



##### Tip7：关于输入与输出

 对于Java的输入，一开始接触的时候我是拒绝的，后来接触多了才习惯。很多同学想问，这里能有什么知识点呢？不就是 `Scanner sc=new Scanner(System.in)` 然后sc各种next方法就可以完成从控制台输入的目的了吗？对我本人而言，有一个细节是之前不了解的，所以才想着记录一下。

`sc.nextLine()  ` 这个方法是可以读入空格的，而同样是读入字符串  `sc.next()` 是以空格为间隔，不能读入空格。 关于输出的问题请看下一段代码

```java
double x=10000.0/3;
System.out.print(x); //3333.333333333..输出的格式并没有一个规范，可能并不是我们想要的
// 所以java模仿c语言，出了一个方法
System.out.printf("%8.2f",x);  // 3333.33  八个字符宽度两个小数。
```



##### Tip8：关于控制流程

java中没有goto语句，相对应的break可以携带标签，直接实习从内层循环跳出的目的。如下：

```java
lable: //标志 必须在最外层循环的前面，而且要带冒号
for (int i = 0; i < 4; i++) {
  for (int j = 0; j < 4; j++) {
    System.out.println("break"); //只循环了一次，正常的break是会打印 4个break出来的
    break lable; 
  }
}
System.out.println("breakover");
//结果 
// break
// breakover
```

在循环中的判断条件如果是 检测两个浮点数是否相等，要格外注意。

```java
for(double x=0;x!=10;x+=0.1){
    // 由于舍入误差最终可能得不到精确值，例如在循环中 因为0.1无法精确的用二进制表示，所以x将从9.9999999998 变成 10.099999999998,导致死循环 （数据可能不准确，但是亲测会死循环）
}
```



##### Tip9：关于数组

数组的很多操作相信大家都是烂熟于心了，我也不说太多。java中允许数组长度为0，在编写一个结果为数组的方法时，如果碰巧结果为空，则这种语法形式就显得非常有用。此时就可以创建一个长度为0的数组。`new elementType[0]` **数组长度为0与null不同**。

java还提供复制业务，`int[] a=Arrays.copyOf(array,array.length())`;  如果a的长度小于原始数组的长度，则只拷贝最前面的数据元素。也提供输出方法，`System.out.println(Arrays.toString(a))`; 不用循环，而二维数组或者更多维的数组，也可以通过类似的方法： `System.out.println(Arrays.deepToString(a))`

到目前为止，我们所看到的数组和其他语言中差异并不大，但是实际上存在着细微差异，而这正是java的优势所在：java实际上并没有多维数组，只有一维数组。多维数组被解释为”数组的数组“。

```java
double balances[][]=new double[10][6];
//balances实际上只有10个元素，每个元素又是由 6个浮点数 组成的数组，
```

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/2.JPG?raw=true">



**所以在java中多维数组可以单独存取某一行，也可以让两行互换。**



##### Tip10：关于main方法的参数args

每个Java程序都有一个String args[]参数的main方法，这个方法将接收一个字符串数组，也就是行参数。这个在IDE中用不到。`java Classname -g hello world` `args[0]="g" args[1]="hello" args[2]="world" `



##### Tip11： 一个老生常谈的东西——封装

很多时候我们的封装都是系统定义的封装，什么意思呢，就是我们采用public protected private 三个权限访问控制来控制我们的封装系统。在看到书中这一段代码之前，我也一直是这样的状态。

```java
class Employee{
    private Date hireDay;
    ...
    public Date getHireDay(){
        return hireDay;
    }
}
```

这段代码的意思很明显，Employee提供一个获取雇佣时间的方法，很明显这个时间是不能够被修改的。

```java
Employee lin=new Employee(...);//参数省略
Date d=lin.getHireDay();
double theYearsInMilliSeconds=10*365.25*24*60*60*1000;
d.setTime(d.getTime()-(long)theYearsInMilliSeconds); 
```

我们的封装很明显出现了问题，因为Date类自己提供了更改器，上面的**对象 d 和 Employee.hireDay 引用同一个对象**，这就意味着只要通过修改 d 的值就可以从外部改变hireDay，与我们最初的目的是不符的。

所以如果需要返回一个**可变对象**的引用时，应该返回它的克隆。返回一个新对象的引用。

```java
class Employee{
    private Date hireDay;
    ...
    public Date getHireDay(){
        return (Date)hireDay.clone();
    }
}
```



##### Tip12：方法参数

方法参数总共有两种类型：1. 基本数据类型   2. 对象引用 

基本数据类型是值传递，所以**一个方法不可能修改一个基本数据类型的参数，因为传进去的都是拷贝值** ，基本数据类型有8种：byte,short,int,long,float,double,char,boolean。

重难点在于传进对象引用，**方法得到的所有参数值都是一份拷贝，包括对象引用**。所以我们可以**通过该拷贝修改它引用的内容，却不能修改它本身的引用**。这句话很拗口，我们通过一个例子来展示：

```java
package com.company;
public class Main {

    //由于方法中的参数都是一份拷贝，所以交换a，b两个引用并不能作用到 原来的引用本身
  public static void swap(Employee a,Employee b){
    Employee temp;
    temp=a;
    a=b;
    b=temp;
  }
  public static void main(String[] args) {
    Employee a = new Employee("Alice");
    Employee b=new Employee("bob");
    
    System.out.println(a.name); //Alice
    System.out.println(b.name); //bob
    swap(a,b);//经过交换之后
    System.out.println(a.name); //Alice
    System.out.println(b.name); //bob
    
    Employee temp;
    temp=a;
    a=b;
    b=temp;

    System.out.println(a.name); //bob
    System.out.println(b.name); //Alice
  }
}

class Employee {
  String name;
  Employee(String name) {
    this.name = name;
  }
}
```



##### Tip13：类的初始化顺序

按照顺序：1. static静态变量  2.初始块  3.构造器  ，按照这个顺序下来的构建一个类如下：

```java
class Employee{
    private static int nextId;
    private int id;
    static{  //静态块
        nextId=10;
    }
    {  //初始块
        id=nextId;
        nextId++;
    }

    Employee(){
        //构造器
        this("调用另一个构造器是用this");
        
    }
}
```

 

##### Tip14: 多态中的陷阱

一般的，我们知道**子类不能够赋值为父类**，这是很常见的规则。但是有一种情况会出现 父类和子类的引用同时指向一片空间，但是那片空间是由父类初始化的：**一般的转化是这样的 super=child ，而该空间是由子类初始化的** ，**由父类初始化的会报错  child=super**  

但是在对象数组中，可能出现第二种情况而不报错：

```java
class Manager extends Employee{
    private int bonus; //父类没有
    setBonus(double bonus){
        // 这个方法，父类没有
        this.bonus=bonus;
    }
}

Manager managers[]=new Manager[10];
Employee employee[]=managers; // 这里是子类转为父类，符合规则
//该数组空间有两组 引用 指向，一组是manager  另一组是 mains
employee[0]=new Employee(); //将第一个空间 由父类初始化，同时mains[0]也指向这里
mains[0].setBonus(13); //触发 ArrayStoreException异常，
//它会调用一个不存在的 属性 bonus,
//因为employee初始化的空间并不符合 manager对象的要求，所以会发出存储异常

```



##### Tip15: 继承——构造器

同时讲多态的时候，别忘了继承，里面有一点特别要注意。子类的构造器会调用父类的构造器，如果父类有一个有参数的构造器，而子类没有构造器，就会报错：提醒你要在子类初始化父类的构造器

```java
class Manager extends Employee{
    Manager(){
        super("父类有参数构造器，子类不能省略构造器");
        ...
    }
}
```

这里顺便提下 `super`这个概念，它和`this`不一样，**不是一个对象引用** ，所以不能将super赋给另一个对象变量，**它只是一个指示编译器调用父类方法的特殊关键字**

 











































































