 参考[Understanding getBean() in Spring](https://www.baeldung.com/spring-getbean)

上篇文章中说到我们已经获取了某个包中带有@Component注解的所有class，我们的目标是将它们实例化，并且解决循环依赖问题

既然到了要实例化的阶段，就不得不考虑容器中的继承关系了，也就是说在容器外部使用容器对象可以实现多态，

```java
A a=ApplicationContext.getBean("A");
....
a=ApplicationContext.getBean("B"); //这里假设 BeanName为B的类继承A，
```

这一块关系比较模糊，需要一个具体的设计。我们暂时不死磕这里，看看Spring是怎么做的。先来个例子：下面创建了两个Bean，Tiger的scope是prototype意味着多例，Lion没有设置默认是单例。Tiger有两个BeanName，Lion只有一个。

```java
@Configuration
class AnnotationConfig{
    @Bean(name={"tiger","kitty"})
    @Scope(value="prototype")
    Tiger getTiger(String name){
        return new Tiger(name);
    }
    
    @Bean(name="lion")
    Lion getLion(){
        return new Lion("Hardcoded lion name");
    }
    
    interface Animals{}  //假设Lion和Tiger都继承了 Animals
}
```

Spring的BeanFactory提供了5个getBean()的API，

```java
Lion lion=(Lion)context.getBean("lion"); //容器中都是Object对象，所以需要转
//直接靠 BeanName获取，如果容器中没有该BeanName，就会抛NoSuchBeanDefinitionException的异常
```

```java
Lion lion=context.getBean("lion",Lion.class); //这里直接指定类型，内部帮忙转换，所以不用显示转
```

```java
Lion lion=context.getBean(Lion.class); //直接靠类型获取
Lion lion=context.getBean(Animals.class); //靠父类获取会出问题，因为不知道具体是哪个子类或者是自己，也就是说如果有另一个  SmallLion 继承了 Lion，第一行代码也会抛出异常
//所以这种getBean只适合没有继承关系的Class使用
```

```java
Tiger tiger=(Tiger) context.getBean("tiger","parameter"); //可以传入参数，作为构造函数的参数，因为tiger是prototype的，多例 每次从容器中getBean都是一个新的对象
```

```java
Tiger tiger=context.getBean(Tiger.class,"parameter"); //构造参数，还是只适用于多例
```

5个API介绍完了，是不是大概对怎么实例化有点想法了？主要通过BeanName和Class来得到容器中的对象。

Spring的BeanName是有一个默认值，即类名的小写单词（并未包含全路径，毕竟还要getBean呢），如果有两个类名相同的在容器中，就提示错误，修正就好了（还有其他情况，这里就不细说）。如果设置了显示的BeanName，就使用该BeanName。

将Class中的信息存储到BeanDefinition中，然后根据BeanDefinition构建实例，其中scope为单例的放进容器(其实就是一个HashMap)，多例的情况就是每次getBean的时候得到的都是不一样的对象所以它们的会放在另一个容器中，

难点是多例对象中包含的单例对象是单例还是多例？？？很明显单例还是单例的，那这么一来不就是两个多例对象共享几个单例对象，造成非线程安全的情况，而且多例的坑还挺多的所以我们暂时不做多例的情况。

拿到所有注有@Component的类的Class信息，之后还有一大波工作要做。需要根据这些信息抽象出更高级的信息，方便之后的构造实例。

最后程序通过@Application注解作为入口，调用注解了@Application类的init()方法，简单的封装一下入口，一个基本的ioc框架就大功告成啦。



---

##### Spring中是如何解决循环依赖的呢？

[Spring源码分析，Spring的循环依赖分析](https://segmentfault.com/a/1190000017995375)

首先我们要明确，在Spring中有三种循环依赖：

- 基于构造方法的循环依赖
- 基于setter构造的循环依赖 （也叫 field 的属性依赖：我的做法也是这样，也只做了这个）
- 基于prototype范围的依赖

而Spring也只是解决了第二种： 让我们了解一下，实例化Bean的过程：

- createBeanInstance：实例化，其实也就是调用对象的构造方法实例化对象
- populateBean：填充属性，这一步就是对Autowired 属性进行 依赖注入
- initializeBean：调用默认或自定义的init方法

能够解决第二种循环依赖，是因为提前将bean create出来（这个步骤叫提前曝光），然后再一个个属性填充(field.set())。而在第一种方式使用构造函数注入时，并没有这个**提前曝光的步骤**，Spring是将需要构造的Bean放入 当前创建Bean池(其实就是一个Set)**如果该池中已经有该Bean就说明循环依赖了**。然后将构造函数中的参数 Bean 也放入池中，这是一个递归的过程，直到最后。

如果没有循环依赖，就会把该对象的属性填充。也就是曝光，放到一个容器中。

##### 基于prototype范围的依赖

对于多例的循环依赖，Spring是无法解决的。因为它没有做缓存和提前曝光。后续就没有后续了，

























