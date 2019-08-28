#### 《Head First 设计模式》——阅读笔记

---

#### 策略模式

书中生动的从模拟鸭子应用开始，不断的变更需求，改动代码引导我们走向设计的世界。

- 从一开始的继承，有个鸭子父类，子类分别是有各自特征的鸭子（比如外观特征，一个红毛一个绿毛）。
- 需求增加为，需要鸭子会飞，于是直接在鸭子父类中加一个fly()方法让所有子类鸭子可以继承它（相信这一步是绝大多数初学者都会干的事儿）
- 有些鸭子不会飞，比如塑料鸭子，这时怎么办呢？ 在塑料鸭子中覆盖fly()方法，里面什么都不干就好了
- 那假设又有一个需求，需要鸭子会叫。也按照fly()的方式覆盖吗？还有很多需求，可能会对应部分子类而另一些子类并没有这些特性，放在父类中必然需要做大量的覆盖工作，有没有什么简便的方法呢？

上面的步骤牵扯出一个**设计原则**：**找出应用中可能需要变化的地方，把它们独立出来** ，对应上面的例子，我们可以将**飞行和叫这两个行为给独立出来**，只有会飞的鸭子继承Flyable（新建一个接口），会叫的鸭子继承Quackable（新建一个接口）。这样就能很完美的解决各个子类群特性不一致的问题啦，

而具体飞行和叫喊的行为也各自不同，比如会飞和不会飞都可以继承FlyBehavior（新建的一个接口），叫声也有不同的声音，可以让不同叫声的实现继承QuackBehavior（新建的一个接口），具体的行为都封装在具体的类中。有人嫌接口麻烦，也可以使用抽象类，这都是可以共通的。

**再利用我们很熟悉的多态，就可以动态的解决鸭子在运行过程中可能发生的变化**（不如受伤之后就不会飞了，这个时候利用多态就可以更改鸭子的行为）这又牵扯出一个设计原则：**针对接口编程，而不是具体实现** ，这一点我感受颇深，因为工作中就有遇到这种情况：需要增加一个新的处理类（和之前类似，但是又有点不同），而原本的方法参数是**接受一个接口**，所以只需要将该处理类实现该接口，就可以将新的处理类传进方法。

这就是针对接口编程，定义变量的时候尽量使用接口（父类），这样以后修改程序的时候可以更加方便和有弹性。我们可以根据鸭子的不同种类定制鸭子的行为，毕竟行为我们都独立出来了。

```java
public class Duck{
	QuackBehavior quackBehavior; //这里是接口，我们可以定义成不同的子类
    FlyBehavior flyBehavior;
    
    public void performQuack(){
        quackBehavior.quack();
    }
    
}
```

看了上面的简单例子，有人觉得我们在Duck中会将接口实现成具体的类来定义行为，这不是违反了 针对接口编程的原则吗？ 但是我们实际运用中总是要有具体的行为吧，不可能全是接口。而且可以使用多态改变行为，所以这样做是没啥问题的。

<img src="https://github.com/krystalics/MyPostPicture/blob/master/_posts/Architecture/img/1.jpg?raw=true">

看着上图，第三个设计原则出来了：**多用组合，少用继承** 。

这就是**策略模式**，把可能会改变的的行为需求独立出来，通过组合达到目的。正式的定义是：**策略模式定义了算法族（对应FlyBehavior和QuackBehavior），分别封装起来让他们可以相互转换（算法族各自继承一个父类），此模式让算法的变化可以独立于使用算法的客户（鸭子）** 



#### 观察者模式

**定义了对象之间的一对多依赖，这样一来，当一个对象改变状态的时候，它的所有依赖者都会收到通知并自动更新。**实际上就是将观察者对象放进主题中，当状态改变，调用观察者对象的方法。

基于我之前对观察者模式的认识，对书中的例子进行设计

<img src="https://github.com/krystalics/MyPostPicture/blob/master/_posts/Architecture/img/2.png?raw=true">

```java
public class WeatherData implements Subject{
    private ArrayList<Observer> observers; //观察者队列
    //...
    public WeatherData(){
        observers=new ArrayList<>();
    }
    
    public void registerObserver(Observer o){
        observers.add(o);
    }
    
    public void removeObserver(Observer o){
        int i=observers.indexOf(o);
        if(i>=0){
            observer.remove(i);
        }
    }
    
    //通知所有观察者即调用观察者的update方法
    public void notifyObservers(){
        for(int i=0;i<observers.size();i++){
            Observer observer=observers.get(i);
            observer.update(); 
        }
    }
}
```

然后在程序入口将所有Observer都添加进WeatherData的观察者队列中(registerObserver)。当特定事件发生可以remove或者notify。



#### 装饰器模式

书中是一个奶茶店的例子，一杯奶茶可以有很多配料，奶茶本身也有多种，原本的设计是给每一种组合都来一个新类，新类和原来的会继承统一的父类Beverage(饮料类)。比如一个DarkRoast(深度烘焙)+Whip(奶泡)会组成一个新类 DarkRoastWithWhip ，一直类比下去。

很显然，在奶茶类型和配料增多的情况下，排列组合得到的数量陡增。我们这种设计面临着**类爆炸**的危险，一般的初学者也很少做出这种设计，感觉就是为了讲这个例子所以故意写了个傻瓜设计。

**进阶设计**是将每种配料都放进父类Beverage中，下面是奶茶类型作为子类，通过改变配料来调出不同的奶茶。

<img src="https://github.com/krystalics/MyPostPicture/blob/master/_posts/Architecture/img/2.jpg?raw=true">

这样只需要5个类就可以完成需求。这个设计的缺陷是：**当配料的价格改变需要更改Beverage中的代码，一旦出现新的配料也是要增加新的方法并改变Beverage的cost的方法**。这就牵扯出一个新的**设计原则**：**类应该对扩展开放，对修改关闭。**

意思很明显就是尽量不要修改原本的代码，但是又要能够满足新的需求。这就很难办了呀，相当于想要新东西还不想给太多钱，这不是强人所难吗？ 首先我们要明确一点：**并不是所有地方都要满足开闭原则的，通常只会在最有可能改变的地方应用它**。因为开闭原则会增加新的类，增加整个系统的复杂度。

装饰者模式的正式定义：**动态地将责任附加到对象上。若要扩展功能，装饰者提供了比继承更有弹性的替代方*案。** 特点是 

- 装饰器和被装饰对象继承同一个父类
- 可以用多个装饰器包装一个对象，例子中就是一杯奶茶多个配料
- 装饰者可以在所委托装饰者的行为之前，之后加上自己的行为，达到特定目的。这里的例子就是加配料

<img src="https://github.com/krystalics/MyPostPicture/blob/master/_posts/Architecture/img/3.jpg?raw=true">

在严格的装饰模式中 父类应该是一个接口，但这里采用的是抽象类其实是一样的，只是为了兼容之前例子中的代码。

```java
public abstract class Beverage{ 
    String description="Unknow Beverage"; //奶茶的描述
    
    public String getDescription(){
        return description;
    }
    
    public abstract double cost(); //奶茶的价钱
}

public abstract class CondimentDecorator extends Beverage{
    public abstract String getDescription();
}
```

```java
public class Espresso extends Beverage{
    public Espresso(){
        description="Espresso";
    }
    public double cost(){
        return 1.99; //单纯该类型奶茶的价格，没有包含配料
    }
}
//其他类型就不写了，都是类似的
```

上面已经将父类和基础的组件类都写好了，就差具体的装饰类了。下面的Mocha包装了一个 Beverage父类，表示一个奶茶类型加上摩卡的描述和价格

```java
public class Mocha extends CondimentDecorator{
    Beverage beverage; 
    public Mocha(Beverage beverage){
        this.beverage=beverage;
    }
    
    public String getDescription(){
        return beverage.getDescription()+", Mocha";
    }
    
    public double cost(){
        return 0.20+beverage.cost(); //摩卡加上奶茶类型的价格
    }
}
```

这样装饰器模式基本搞定了，我们来测试一下吧。

```java
public class Test{
    public static void main(String[]args){
        Beverage beverage=new Espresso(); //来一杯 Espresso
        System.out.println(beverage.getDescription()+" $"+beverage.cost());
        
        //下面利用多态开始包装b2
        Beverage b2=new DarkRoast(); 
        b2=new Mocha(b2); //用摩卡包装一下b2,意味着现在是一杯有摩卡的奶茶了
        b2=new Mocha(b2); //再包装一遍，就是有两倍摩卡的奶茶了
        b2=new Whip(b2); //就是一个有雪泡加上两倍摩卡的奶茶了
        System.out.println(b2.getDescription()+" $"+b2.cost());
        
    }
}
```



是不是很简单呢，当然了装饰器模式的缺陷也是有的**，比如上面一个例子，需要创建多个小对象来完成装饰**。 而且**如果针对某个具体的组件有特别的变化 比如上述例子中的Espresso打折，这个需求需要具体到某个组件类，使用装饰器模式还是需要修改原本Espresso的代码**，打折可能不够特别，因为其他奶茶也可以打折，这里只是举个例子，实际上打折可以在Beverage中加一个打折的属性，然后在入口处设置折扣。

加入只有Espresso会打折，其他组件不需要，这种只针对具体类的需求就要重新考虑装饰器模式是否合适了。

**Java I/O就是采用了装饰器模式。**FilterInputStream就是各个具体装饰器的父类，比如 BufferedInputStream，DataInputStream等。这些都是可以包装继承InputStream的基础组件类的装饰器。和我们的例子中是一致的，那我们就拓展一下Java I/O的装饰器吧

```java
public class LowerCaseInputStream extends FilterInputStream{
    //FilterInputStream中的 InputStream是 volatile的，这里只需要将外界赋的InputStream传递给父类就可以
    public LowerCaseInputStream(InputStream in){
        super(in);
    }
    
    public int read() throws IOException(){
        int c=super.read();
        return (c==-1?c:Character.toLowerCase((char) c)); 
    }
    
    public int read(byte[] b,int offset,int len) throws IOException{
        int result=super.read(b,offset,len);
        for(int i=offset;i<offset+result;i++){
            b[i]=(byte)Character.toLowerCase((char)b[i]);
        }
        return result;
    }
}
```

上面简单的对InputStream做了一个包装，使得读到的数据变成了小写字母

```java
public class Test{
    public static void main(String[]args) throws IOException{
    	int c;
       
        InputStream in=new FileInputStream("test.txt");
        in=new BufferedInputStream(in);
        in=new LowerCaseInputStream(in);
        
        while((c=in.read())>=0){
            System.out.println((char) c);
        }
        
        in.close();
    }
}
```



#### 工厂模式

 













