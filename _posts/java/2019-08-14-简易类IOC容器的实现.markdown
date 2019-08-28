实习的时候有幸接触到了公司自写的一个小的BeanFactory容器，它相当于Spring中的一小部分（在类上注解@Component，然后容器会把所有带有@Component的类都初始化成对象）。

之前的版本是只有这一部分，我和另一个实习生的任务是加上@Autowired注解，这本来是一个人的任务，可能是考虑到这个问题有点棘手就分成了两个部分，而我就是负责@Autowired之后的循环依赖问题。

想要了解什么是循环依赖？

```java
public class A {
    B b=new B();

    public static void main(String[] args) {
       new A();
    }
}
public class B{
    A a=new A();
}
```

上面的例子：A中有B，B中有A二者看似非常融洽，实际上会一直循环的创建，最终导致 `java.lang.StackOverflowError` 栈溢出。我们在使用容器的时候，可以使用**依赖注入**来避免这种情况，也就是说即使容器中发生了循环依赖也可以正常工作。换句话说，容器解决了循环依赖的问题（单例的情况，如果是多例的话情况会不一样，所以下文中讨论的也都是单例的情况）。

这里引用一个例子，讲下什么是依赖注入  [例子原文链接](https://blog.csdn.net/a1181191837/article/details/60468422) 

```java
public class  Test{
    public static void main(String[] args) {
        A testa = new A();
        B testb = new B();
        //就是不在具体的事务类中初始化，放到第三方来初始化，并注入进去
        testa.b = testb; 
        testb.a = testa;
        
        testa.printB();
        testb.printA();
        testa.print();
        testb.print();
    }
}
 
class A{
    public B b;
    public void printB(){
        b.printB();
    }
     
    public void printA(){
        System.out.println("AA");
    }
     
    public void print(){
        b.printA();
    }
}
class B{
    public A a;
    public void printB(){      
        System.out.println("BB");
    }
     
    public void printA(){
        a.printA();
    }
    public void print(){
        a.printB();
    }
}
```

IOC容器比这复杂一些，调用链会形成一棵树，如果存在循环问题则需要另外做处理，而且是通过反射拿到有注解的那些类，把构造的bean都放在一个容器中（通常都是一个HashMap），在外部使用容器中的对象通过getBean获取，注意通过new出来的对象是会报错的，因为new出来的对象和容器中并不是一个对象，其中用@Autowired注解的在new出来的对象中为null，而在容器中是已经构造好的，在容器中是可以自由的通过@Autowired注解使用对象。

spring是做了隐藏，并没有让人感觉到在容器外部是通过getBean来获取对象。说了这么多，IOC就这么一点好处吗？并不是， [Spring IoC有什么好处呢?](https://www.zhihu.com/question/23277575) 

文中第一个回答先讲了**依赖倒置**：本来是由高层的类依赖底层类的实现，因为高层的方法都是在调用底层的方法，一般没有设计的系统都是这么干的。而依赖倒置顾名思义就是让底层依赖高层，这里并不是说底层调用高层的方法，**而是由高层决定需求，底层具体实现**。这里听起来就有点懵，这两个东西不都是一样的吗？文中举了个由高层变动需求，层次修改到底层的例子来证明依赖反转是非常有必要的。

依赖倒置只是一个概念性的东西，具体的实现还是由代码来体现。而IOC控制反转就是实现依赖倒置的一种方式，将依赖关系交给第三方容器去做，就像上面那个例子一样。**A看似依赖B，实际上是依赖初始化B的Test，B也是一样**

容器还可以省略类的构造步骤，比如A依赖B，B依赖C，我们要初始化A的时候就必须先初始化B，而这又会转到C，所以代码如下：

```java
C c=new C();
B b=new B(c);
A a=new A(b);
```

这样其实也可以接受，但是如果调用链很长，有几十个甚至上百个那么长，我相信是个人都受不了。更不用说错综复杂的依赖关系了，能不能把对象构造出来都是一个问题。而容器会在程序初始化的时候从该类的依赖底层开始构造，一直到把该类构造完存储到容器中，用的时候直接getBean就可以了。

还有统一管理等诸多好处就不细说了。



##### 从JVM加载Class文件说起

为什么会有容器这种技术呢？这就要追溯到JVM加载class文件了，这里细节请看[老大难的 Java ClassLoader 再不理解就老了](https://juejin.im/post/5c04892351882516e70dcc9b) 

JVM运行并不是一次性加载所需要的全部类，而是按需加载（**这里的加载就是将二进制的class文件通过ClassLoader()或者Class.forName()将class文件加载进jvm，变成java.lang.Class的一个对象**），即延迟加载。程序在运行过程中会遇到新的class会先判断是否已经加载过了，如果没有才会调用ClassLoader来加载class文件、加载完成的对象会存在ClassLoader里，下次就不需要重新加载了。

而我们可以通过代码调用反射中的ClassLoader或者Class.forName，来提前加载我们需要的类，进行初始化。还可以提前判断循环引用等。



##### 好了说了这么多，开始进入正题吧：

如何构造一个可用的IoC容器呢？

- 首先需要扫描当前程序的包，获得所有包含@Component类的Class
- 然后根据Class中的@Autowired的调用链构造一棵依赖树，如果遇到循环，就将该节点标记，下次遇到时就跳过该节点（这样就不会一直循环构造下去了）
- 后续遍历这棵树，从依赖底层开始创建对象一直到根节点。
- 构建容器与外部的接口
- 外部使用容器

其中最难的步骤在于扫描当前的所有包（包括Jar包），获取所有class文件中有注解@Component的Class。因为class文件时二进制流，有自己特定的格式，如何读取和写入都是一个问题。

参考 [JVM之用java解析class文件](https://juejin.im/post/589834a20ce4630056097a56#comment)  [Java扫描并加载包路径下的class文件](https://hacpai.com/article/1503198996878)

认真阅读上面的两篇文章就会发现一个问题，扫描到了所有的class和解析class文件二者好像并没有什么关联，我直接就可以根据扫描得到的class来进行之后的操作，那了解JVM解析class文件有什么用？

问了公司的前辈，它的回答虽然我不怎么认同，但姑且听之吧：

> 如果不解析class文件，我们会扫描和加载一个包内所有的类，比较消耗内存；还会执行里面的static赋值以及方法等，如果里面有static A a=Context.getBean()；从容器中获取对象，这个A就会被初始化为null，使用它的类就会报出NullPointerException。

消耗内存这个我是认同的，毕竟一个大一点的项目会有很多个类，稍微扫描大些的包大几千个类都会被加载进jvm（这里我们要知道，JVM加载class时按需加载），让后再根据类信息过滤掉没有@Component的类，这样初始化速度会变得很慢，而且内存消耗确实比较大。

但是第二点我就心存疑问，就去搜了一下发现只有用Class.forName()加载类的时候会执行static，而使用ClassLoader就不会执行static的内容。这里参考 [java反射中，Class.forName和classloader的区别](https://blog.csdn.net/qq_27093465/article/details/52262340) 

不过，就程序初始化慢以及内存消耗这块已经足够理由让我们去解析class文件了。直接解析class而不使用Class.forName()或者ClassLoader()加载它，可以让我们提前过滤掉非容器的类，节省了一大块内存（和加载非必要类的时间）。

现在的问题就虽然能解析class文件了，但是我们怎么 利用这个解析来工作呢？**在我们这里就是在不加载class文件的情况下怎么分析出class文件的注解**，所以首先要了解**注解在class文件的位置——常量池**。在常量池中获取注解名进行判断，是不是我们要的注解

首先这里还要了解一个知识点：运行时注解在常量池中以字符串 RuntimeVisibleAnnotations 作为标注来对应注解中的RUNTIME标记。我们要做的就是遍历常量池，找到该标记，获得该类的注解，然后做判断。

```java
public static void read(String classPath) {
        File file = new File(classPath);
        try {
            FileInputStream in = new FileInputStream(file); //读取路径中class的输入流
            ClassFile classFile = new ClassFile(); //存储该class的信息

            classFile.magic = U4.read(in);
            classFile.minorVersion = U2.read(in);
            classFile.majorVersion = U2.read(in);

            //解析常量池，类名以及各个方法名，描述符，annotation，public protected...字符串 final都存在常量池中
            classFile.constantPoolCount = U2.read(in);
            ConstantPool constantPool = new ConstantPool(classFile.constantPoolCount);
            constantPool.read(in);
            
           
            int index=-1;
            for(int i=1;i<classFile.constantPoolCount;i++){
                ConstantInfo constantInfo=constantPool.cpInfo[i];
                if(constantInfo instanceof ConstantUtf8){
                    String name=((ConstantUtf8)constantInfo).value();
                    //下面的字符串对应 RUNTIME，是运行时作用的注解的标志
                    if("RuntimeVisibleAnnotations".equals(name)){
                        index=i;
                        break;
                    }
                }
            }
            
            List<String> annotations=new ArrayList<>();
            for(int i=index+1;i<classFile.constantPoolCount;i++){
                ConstantInfo constantInfo=constantPool.cpInfo[i];
               if (constantInfo instanceof ConstantUtf8) {
                    try {
                        annotations.add(Type.getType(((ConstantUtf8) constantInfo).getValue()).getClassName());
                    } catch (Exception ignore) {
                    }
               }else{
//实际上一连串的UTF8不只是注解，还有类里定义的各个变量的全名，顺序不定
//包括方法里定义的也在，但是我们只需要知道这段UTF8过后 注解已经扫完了就行，之后的判断不需要管
                   break; 
               }
               
            }
            
            //此时，annotations 就装满了 该类的注解，我们需要对注解进行解析
            
        }catch (IOException e){
            
        }
}
```

之后经过过滤（通过完全类名做判断），获得该包中所有打了@Component的类了。

拿到具体的类之后，就需要根据其间的依赖关系开始实例化了。

具体见下一篇文章。























