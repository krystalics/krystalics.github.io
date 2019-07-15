[使用hsdis与jitwatch查看JIT后的汇编](https://www.jianshu.com/p/78f71c033fae)

事先说明，我并不懂汇编。会记录这文章是因为工作中碰到了需要做性能分析乃至优化的情况，正所谓 JProfile（查看项目整体运行情况，调用链和热点方法以及内存使用情况），JMH（基准测试Benchmark，精确到方法级的性能分析，一般用作对比测试：对比两个版本方法的性能差异），JitWatch—hsdis（JIT之后，因为一般人看不懂汇编，所以它的作用一般会是查看哪些方法被inline） ——Java性能分析的三剑客（自己瞎编的）



使用JitWatch需要hsdis，hsdis官网有提供编译方法但是为了方便起见，这里也提供两个会用到的文件。

[hsdis-amd64.dll](https://github.com/krystalics/krystalics.github.io/tree/master/_posts/java/jitwatch/hsdis-amd64.dll)       [hsdis-amd64.dylib](https://github.com/krystalics/krystalics.github.io/tree/master/_posts/java/jitwatch/hsdis-amd64.dylib)

将下载的 `hsdis-amd64.dll` 放入你自己的jdk目录中，路径如下`jdk_xxx\jre\bin\server`或者`jdk_xxx\jre\bin\client` 

然后在命令行打出 `java -XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly -version` 如果输出 `Java HotSpot(TM) 64-Bit Server VM warning: PrintAssembly is enabled` 则表示安装成功

如何使用它呢？

运行程序之前需要配置参数，在IDEA中edit Configurations—>VM options中加入如下参数：

>-XX:+UnlockDiagnosticVMOptions
>-XX:+TraceClassLoading
>-XX:+LogCompilation
>-XX:LogFile=D:\jitwatch\logs\ByteIOV2.log 
>-XX:+PrintAssembly
>-XX:+TraceClassLoading

上面的参数中 LogFile 是指生成的log文件路径。用命令行运行的话

```
javac Test.java
java -server -XX:+UnlockDiagnosticVMOptions -XX:+TraceClassLoading  -XX:+PrintAssembly -XX:+LogCompilation -XX:LogFile=live.log Test
```

但是用javac编译过java文件的同学们都清楚，classpath需要一致很多时候你会得到 Could not fould Class Test..之类的信息。所以推荐使用IDE，eclipse也是一样的就不写了。

好了运行程序就可以输出log文件了，关键是这个log文件又臭又长

>[Constants]
>
>{method} {0x00000000147d9508} &apos;getSecurityManager&apos; &apos;()Ljava/lang/SecurityManager;&apos; in &apos;java/lang/System';
>
>  0x0000000002c537e0: mov    %eax,-0x6000(%rsp)
>  0x0000000002c537e7: push   %rbp
>  0x0000000002c537e8: sub    $0x30,%rsp
>  0x0000000002c537ec: movabs $0x149f6960,%rax   ;   {metadata(method data for {method} {0x00000000147d9508} &apos;getSecurityManager&apos; &apos;()Ljava/lang/SecurityManager;&apos; in &apos;java/lang/System&apos;)}
>  0x0000000002c537f6: mov    0xdc(%rax),%esi
>  0x0000000002c537fc: add    $0x8,%esi
>  0x0000000002c537ff: mov    %esi,0xdc(%rax)
>  0x0000000002c53805: movabs $0x147d9500,%rax   ;   {metadata({method} {0x00000000147d9508} &apos;getSecurityManager&apos; &apos;()Ljava/lang/SecurityManager;&apos; in &apos;java/lang/System&apos;)

上面的只是一个小片段。

所以我们需要JitWatch来帮助我们理解这东西。

[JitWatch源码](https://github.com/AdoptOpenJDK/jitwatch)  进入它的github主页，clone下来之后

```
git clone https://github.com/AdoptOpenJDK/jitwatch.git
cd jitwatch
mvn clean install -Dmaven.test.skip=true
```

然后等待它运行完毕之后，打开jitwatch的目录，如果是windows就运行launchUI.bat，对应的Linux为launchUI.sh 。

之后就会有UI出来了

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/49.png?raw=true">

点击上面的openlog，寻找我们之前运行程序生成的log文件，并通过config来配置代码的源码路径和class的路径，以及jdk路径。

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/50.png?raw=true">

点击start开始运行。运行完毕之后，左边会出来经过编译的类和包，选择其中一个再点击菜单栏中的TriView就可以看到运行情况了

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/51.png?raw=true">

如果该方法没有被编译进JIT，那么右侧就会显示的not jit compiled。如果是inline，那么右侧就是汇编代码。



还有一个很重要的东西，要知道JIT编译器方法的优化有很多种，其中最重要的就是

- 将bytecode编译成本地代码native：默认配置是方法执行超过10000次，jvm会将其编译成native。相关配置在-XX:CompileThreshold=10000 中。On Stack Replacement（OSR）：如果循环体执行的次数非常多，那么该循环体可能也会被编译为本地代码
- 分支预测（Branch Predication）：降低分支条件判断结果的随机性，使得CPU指令流水线缓存命中率提升
- 方法内联（inlining，对性能提升很大）：方法内联可以减少方法调用，从而减少方法栈的创建。毕竟可以减少进栈和出栈的消耗。
  - -XX:MaxInlineSize：配置能被内联方法的最大字节数，默认值为35Byte，一般不改它（各个大牛都这样说）。像是一般pojo类的getter和setter不需要频繁调用，但是它们的字节码一般非常短，这种方法在执行后是会被内联的。
  - -XX:FreqInlineSize：调用非常频繁的方法能被内联的最大字节数，默认值为325Byte（据说和平台有关，64位时325Byte）

查看方法如下图：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/img/53.png?raw=true">

