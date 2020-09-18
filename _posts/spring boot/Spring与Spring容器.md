<a name="spring">Spring与Spring容器</a>
---

 下面一段参考： https://www.cnblogs.com/chenbenbuyi/p/7470834.html

在Spring出来之前,也就是没有依赖注入DI之前Java应用程序之间的类与类要实现互相功能协作是比较直观的，如果A类需要依赖B，就在A类中创建一个B类的对象。这就等于A类需要负责B类对象整个生命周期的管理。在系统不复杂时例如只有4，5个类时这并不会有不好的影响，但是更大一点的系统里面类与类之间的关系错综复杂，依赖关系也是。这种系统的耦合度非常高，复杂度以及修改难度很大。

从某种意义上来说：Spring就是负责帮我们创建对象，搞定依赖关系的第三方组件。当某个类需要依赖另外的对象时，Spring会负责它的创建并交给它使用，通过依赖注入的方式。而Spring是如何注入的呢？就好像苹果并不在本部生产，而是通过第三方代工，比如富士康工厂。而Spring中生产对象的地方不叫工厂，而是叫做容器。容器的概念在java中有个很有名的例子。Tomcat就是一个正在运行的Servlet的web容器。在java中，Bean也是对象的意思，具体的原理还在下面。

SpringBoot 通过注解的方式来声明该类的对象 管理将由IoC容器负责，需要使用时还是通过注解使用。例子：

```java
public interface Super{
     
}

@Service  //将其交给 IoC容器管理
public class A implements Super{
    
}

@Service 
public class B implements Super{
    
}

@Autowired
privated Super a;  // IoC容器自动注入A或B的 Bean， 这个bean中只能使用Super中的方法
@Autowired 
privated A a; // 这个也是自动注入的 Bean ，但是里面的方法是A类中的所有方法，
// 这两种应用看哪种适合吧
```



Spring 使用 BeanFactory的子类(功能更强大) 统一管理那些Bean：

```java
public interface BeanFactory {

	/*
	 用于取消引用实例，并将其与FactoryBean创建的bean区分开来。
	 */
	String FACTORY_BEAN_PREFIX = "&";


	/**
	 返回指定bean的实例，该实例可以是共享的或独立的。
	 此方法允许Spring BeanFactory用作Singleton或Prototype设计模式的替代。
	 在Singleton bean的情况下，调用者可以保留对返回对象的引用。
	 将别名转换回相应的规范bean名称。
     将询问父工厂是否在此工厂实例中找不到bean。
	 */
	Object getBean(String name) throws BeansException;

	/*
		和上面的那个方法行为相同，但如果bean不是所需类型，则通过抛出BeanNotOfRequiredTypeException来提供类型安全性的度量。
	 */
	Object getBean(String name, Object... args) throws BeansException;

	/*
	返回唯一匹配给定对象类型的bean实例（如果有）。
	此方法进入ListableBeanFactory 按类型查找区域，但也可以根据给定类型的名称转换为传统的按名称查找。对于跨bean集的更广泛的检索操作，
	 */
	<T> T getBean(Class<T> requiredType) throws BeansException;

	
	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

	
	<T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);

	
	<T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);

	
	boolean containsBean(String name);

	/*
	此API将确认getBean（java.lang.String）是否返回独立实例 - 这意味着配置了原型范围的bean。需要注意的重要一点是，此方法返回false并不能清楚地表明单个对象。它表示非独立实例，也可能对应于其他范围。我们需要使用isSingleton（java.lang.String）操作来显式检查共享单例实例。
	*/
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	
	@Nullable
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	//返回给定bean名称的别名（如果有）。
	String[] getAliases(String name);

}

```



[关于ApplicationContext的东西](https://juejin.im/post/5a4d92d8f265da4311209f40)

[ApplicationContext](http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/context/ApplicationContext.html)是Spring应用程序中的中央接口，用于向应用程序提供配置信息 。主程序入口都是它的实现类，它继承了 BeanFactory ，功能更加强大。

而且`BeanFactory`是延迟加载某个Bean，如果Bean的某一个属性没有注入，`BeanFacotry`加载后，直至第一次使用调用getBean方法才会抛出异常；而`ApplicationContext`则在初始化自身时检验，这样有利于检查所依赖属性是否注入；所以通常情况下我们选择使用ApplicationContext。

而我们说的 IoC容器其实就是 ApplicationContext，

1. `ApplicationContext `提供更多企业级的服务，例如解析配置文本信息等这也是应用上下文实例对象最常见的应用场景
2. `AnnotationConfigApplicationContext: `从一个或多个基于java的配置类上加载上下文定义，使用于java注解的方式
3. `ClassPathXMLApplicationContext: `从类路径下的一个或多个基于xml配置文件中加载上下文定义，使用于xml配置方式
4. `FileSystemXmlApplicationContext: `从文件系统下的一个或多个xml文件上加载上下文定义，也就是说系统盘符中加载xml配置文件
5. `AnnotationConfigWebApplicationContext:` 专门为web应用准备的，适用于注解方式
6. `XmlWebApplicationContext: `从web应用下的一个或多个xml配置文件加载上下文定义，适用于xml配置方式。



而那些jar包之间复杂的依赖关系是怎么被注册到ApplicationContext中的呢？这就要谈到SpringBoot的自动配置了，在之前的版本中spring是采用的xml配置，也就是说需要我们另外写一个xml文档来说明该类需要IoC容器的控制。而这样的配置很麻烦，所以spring boot才横空出世，简化配置！

 参考自： [详解Spring Boot自动配置机制](http://p.primeton.com/articles/5a3777d74be8e66990001635)

Spring Boot 自动配置的目标是通过jar包的依赖，自动配置应用程序。`SpringFactoriesLoader`（抽象类）会通过`loadFactories/loadFactoryNames`加载每个jar包中 `META-INF/spring.factories`文件，(如果该jar包中真的有的话)。加载`spring.factories`中`EnableAutoConfiguration`下配置的所有的配置源，并注册到 Spring 的 `ApplicationContext `上去，了解了这个机制后，我们就可以按照自己的需求定制自动配置。

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName(); //获得加载进来的工厂类名
    try {
        //获得 spring.factories 的位置
        Enumeration<URL> urls = (classLoader != null ? classLoader.getResources(FACTORIES_RESOURCE_LOCATION) : 
                lassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
        
        List<String> result = new ArrayList<String>();
        while (urls.hasMoreElements()) { //遍历该urls数组，通过其中的URL来加载那些类的配置
            URL url = urls.nextElement();
            Properties properties = PropertiesLoaderUtils.loadProperties(new UrlResource(url));
            String factoryClassNames = properties.getProperty(factoryClassName);
            result.addAll(Arrays.asList(StringUtils.commaDelimitedListToStringArray(factoryClassNames)));
        }
        return result;
    }
    catch (IOException ex) {
        throw new IllegalArgumentException("Unable to load [" + factoryClass.getName() +
                "] factories from location [" + FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}

```



