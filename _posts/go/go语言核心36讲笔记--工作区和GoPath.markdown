今天，公司实习任务（其实是我主动上报想学go），查了查网上的教程，发现还是买一个专栏省心，起码质量有保证，花了我99元呢，一定要看完。如果看完笔记，想要购买教程的同学们可以点击链接，扫描二维码，可以省30元哟

https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/tuijian.jpg

---

#### 工作区和GOPATH

文章中提到在Go1.5版本实现了自举(用Go语言编写程序来实现Go语言本身)，我从来没有听过这个概念，除了C生万物之外竟然还有自举这种东西，特意搜了一下什么叫自举： [编译器的自举原理](https://www.zhihu.com/question/28513473)

例子：如果你想创造一门语言X，并且用X语言来写X编译器，步骤如下

- 用C++把编译器A写出来，留下测试用例
- 用X语言把编译器B写出来，用A.exe来编译B，修改直到所有测试用例都通过了为止
- B.exe来编译B，得到B2.exe，修改直到B2.exe所有测试用例都通过，
- 当你觉得一切都ok了，就用A.exe吧B编译一遍，得到B.exe。然后A的代码和A.exe都给删掉，之后就不断用B.exe来编译下一个版本的B。

至于之后的安装和IDE下载就不记录了，有点编程经验的同学们都可以自己解决吧。

进入正题：

- GOROOT：Go语言安装目录的路径，比如都是C:/go/bin 
- GOPATH：工作区目录的路径。像是Eclipse的workspace，就是告知系统这里是Go的项目区间
- GOBIN：Go程序生成的可执行文件的路径，还会生成可执行文件有点像C++呀。。

那么问题来了，为什么要设置GOPATH和GOBIN这两个环境？关于这个问题有个通用的回答：设置工作区是因为Go语言项目在其生命周期内的所有操作（编码，依赖管理，构建，测试，安装等）基本都是围绕着GOPATH和工作区来进行的，像是NodeJs会专门在目录下创建node_module来管理依赖，python也有自己的一套体系，而GOPATH和工作区的概念就是Go的一套体系。

我们需要了解的是：1.Go语言源码的组织方式是怎么样的（这个可以理解为它的整个项目结构是怎么样的）；2。是否了解源码安装后的结果（像安装完python之后可以运行python的命令行）3。是否理解构建和安装Go程序的过程

首先，我们需要了解Go语言中的几种源码文件，如下图：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/3.png?raw=true">

文章中给出了答案：

##### -1：Go语言源码组织方式

​	以包为单位，像是python中的各种包其实就是对应的目录和子目录。其中.go文件就是存放代码的文件，和Java一样需要声明该文件处于什么包中（如：package com.example.gopro.test;）。

​	像Java中可以对包名重构一样，目录名是不会变的，这时以目录名为准（因为go是以目录来查找包的）。导入和python类似：很奇怪，go的文件路径命名以github的路径命名，

```go
import "github.com/labstack/echo";
```

​	在工作区的导入路径是相对路径，不像java是全路径。一般我们的项目都会在GOPATH包含的目录中。这部分内容与其他编程语言基本一致，没有什么需要特别关注的。

##### -2：了解源码安装后的结果以及最终Go项目的结构

go的源代码在安装之后会产生归档文件(archive) .a为拓展名的文件（这是一个静态链接库文件，存储多个源码文件内容的集合），会被自动放在工作区的pkg子目录；而可执行文件.exe会放在工作区的bin子目录。导入命令如下：

```sh
go install github.com/labstack/echo
```

对应的归档文件的相对目录  github.com/labstack ，文件名为 echo.a 。又因为go是跨平台的，各个系统的文件不一致，所以在pkg与github.com/labstack 之间还会有一层平台的目录，所以总的结构是这样的：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/1.png?raw=true">

一个更加细节具体的图 （图片来源：https://golang.org/doc/code.html）

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/2.png?raw=true">

源码文件一般放置在pkg下的同名目录中，或者直接就是bin目录中。

可以很明显的看出来，go install 并不是像 npm install这样的包管理方式，它是本地目录的install，并不是指获取github上的代码进行install。

##### -3：理解构建和安装Go程序的过程

build和install的过程是类似的，构建命令式 go build 安装命令式 go install 。构建和安装代码包的时候都会执行编译，打包等操作。并且它们都会被保存到某个临时目录中。

build的过程：

- 如果build库源码文件，那么产生的文件只会被保存到临时目录中，它的意义是检查和验证。
- 如果build的是命令源码文件，那么产生的文件会被传进源码文件所在的目录中。

在运行`go build`命令的时候，默认不会编译目标代码包所依赖的那些代码包。如果被依赖的代码包的归档文件不存在或者源码发生了变化，还是会被编译。需要其他功能时候加个辅助参数

```sh
go build -a #强制编译
go build -i #不但强制编译依赖的代码包，还安装他们的归档文件
go build -x #查看go build 命令具体执行了哪些操作，
go build -n # 只查看执行操作而不执行它们
go build -v #可以看到go build 命令编译的代码包的名称
```

关于被依赖的代码包的归档文件不存在，就是.a文件不存在，build的时候会编译该代码包。如果.a文件存在，如果像C语言的方式，hello.a这个链接库存在，编译运行的时候不会去接触源文件。但是go中会检查源文件是否发生了变化，我们可以运用上面的参数来看看 build的具体流程：

```sh
D:\awesomeProject>go build -x -v main
# 首先它指定了个临时的工作空间
WORK=C:\Users\12857\AppData\Local\Temp\go-build354097426
# 然后它准备编译 main 目录下的包
main
# 创建一个b001文件夹和里面的引入配置文件
mkdir -p $WORK\b001\
cat >$WORK\b001\importcfg << 'EOF' # internal 到EOF文件结束
# import config  引入配置，就是需要引入的依赖包，这里是各个源文件的归档文件
packagefile flag=E:\Go\pkg\windows_amd64\flag.a
packagefile fmt=E:\Go\pkg\windows_amd64\fmt.a
packagefile os=E:\Go\pkg\windows_amd64\os.a
packagefile stringutil=D:\awesomeProject\pkg\windows_amd64\stringutil.a
packagefile runtime=E:\Go\pkg\windows_amd64\runtime.a
EOF
# 进入我们要编译的目录，然后调用 go安装路径下的编译器 compile.exe 
cd D:\awesomeProject\src\main
"E:\\Go\\pkg\\tool\\windows_amd64\\compile.exe" -o "C:\\Users\\12857\\AppData\\Local\\Temp\\go-build354097426\\b001\\_pkg_.a" 
#...太多了，省略
# 生成 导入的link文件
cat >$WORK\b001\importcfg.link << 'EOF' # internal
packagefile main=$WORK\b001\_pkg_.a
packagefile flag=E:\Go\pkg\windows_amd64\flag.a
packagefile fmt=E:\Go\pkg\windows_amd64\fmt.a
packagefile os=E:\Go\pkg\windows_amd64\os.a
packagefile stringutil=D:\awesomeProject\pkg\windows_amd64\stringutil.a
packagefile runtime=E:\Go\pkg\windows_amd64\runtime.a
packagefile errors=E:\Go\pkg\windows_amd64\errors.a
packagefile io=E:\Go\pkg\windows_amd64\io.a
packagefile reflect=E:\Go\pkg\windows_amd64\reflect.a
packagefile sort=E:\Go\pkg\windows_amd64\sort.a
packagefile strconv=E:\Go\pkg\windows_amd64\strconv.a
#...还是太多了，省略
EOF
# 创建exe文件夹
mkdir -p $WORK\b001\exe\
# 进入当前目录，即执行go build命令的目录。执行以下操作
cd .
"E:\\Go\\pkg\\tool\\windows_amd64\\link.exe" -o "C:\\Users\\12857\\AppData\\Local\\Temp\\go-build354097426\\b001\\exe\\a.out.exe" -importcfg "C:\\Users\\12857\\App
Data\\Local\\Temp\\go-build354097426\\b001\\importcfg.link" -buildmode=exe -buildid=40kiYswH4PCMhaXMar_8/elbKqvKBgG3T6KwpMNrh/iKC7CdCM2AbhDX371myo/40kiYswH4PCMhaXM
ar_8 -extld=gcc "C:\\Users\\12857\\AppData\\Local\\Temp\\go-build354097426\\b001\\_pkg_.a"
"E:\\Go\\pkg\\tool\\windows_amd64\\buildid.exe" -w "C:\\Users\\12857\\AppData\\Local\\Temp\\go-build354097426\\b001\\exe\\a.out.exe" # internal
# 最后复制可执行文件到 main.exe中，删除之前创建的临时文件夹
cp $WORK\b001\exe\a.out.exe main.exe
rm -r $WORK\b001\
```

install会先执行build，然候会进行链接(Link) 并把结果文件传进指定目录（这点和C的编译过程很像）。和上面相对应的，

- 如果insatll的是库源码文件，产生的结果文件会被传进它所在的工作区的pkg目录中。
- 如果是命令源码文件，结果文件会被传进工作区的bin目录中，或者环境变量GOBIN指向的目录中。

如果迫不及待的想要运行第一个helloworld程序，请参考这里 [how to write go code](<https://golang.org/doc/code.html>) 

---

继续extend，`go get` 是类似于pip，npm一样的工具，它会自动从一些主流公用的代码仓库中（比如Github）下载目标代码包，并把它们安装到GOPATH包含的第一个工作区的相应目录。如果还设置了GOBIN，那么仅包含命令源文件的代码包会被安装到GOBIN指定的目录，和很多命令一样，会有辅助的参数来增加功能：

- -u：下载并安装代码包，不论工作区是否存在
- -d：只下载代码包，不安装
- -fix：在下载代码包之后先运行一个用于根据当前Go语言版本修正代码的工具，然后再安装代码包
- -t：同时下载测试所需的代码包
- -insecure：允许通过非安全的网络协议下载和安装代码包。例如HTTP

官方提供的`go get`比较基础，没有依赖管理的功能。github上有很多这样的工具 glide，gb以及官方出品的dep 、vgo等，它们内部也是使用的go get。

关于代码包远程导入路径自定义的内容我并没有完全理解，所以这里就暂时不写上去了。文章还给出了两题思考题：

1.Go语言在多个工作区中查找依赖包的时候是以怎样的顺序进行的？

按照我的理解，是就近原则。而且工作区之间可以相互连通吗？？看来我还是没有完全读懂这篇文章

查了一下文章，说是在查找依赖包的时候是按照工作区的先后顺序来查找的。

2.如果多个工作区都存在导入路径完全相同的代码包会产生冲突吗？

不冲突，因为按照工作区顺序查找找到之后就不找了，自然不会知道后面还有相同的包



##### 好了文章大致结束了，我们来实战演练一番吧：使用Goland或者其他IDE都可

在设置了GOPATH之后，我们在对应目录下创建src,bin,pkg，如下图

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/4.png?raw=true">

然后运行`go run hello.go`就可以了

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/5.png?raw=true">

想要将其编译打包成可执行文件，在对应目录运行`go install main`就可以了，像我的项目中是这样的`D:\awesomeProject\src>go install main` ，就可以在 bin目录看到main.exe了。

`D:\awesomeProject\src>go build main`则是在和main目录同级下生产main.exe

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/6.png?raw=true">



如果需要加入其它包中的文件，可能比较麻烦。

按照教程，假如需要加一个stringutil包，其中reverse.go如下：

```go
package stringutil
// 注意，方法开头要大写
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

不能像java一样直接引入 stringutil包，而是要先编译  reverse.go 文件。`go build stringutil` 

这一步不会产生可见的结果文件，它的结果保存在本地的build缓存中。再修改原来的hello.go

```go
package main

import (
	"fmt"
	"github.com/user/stringutil"
)

func main() {
	fmt.Println(stringutil.Reverse("hello world"))
}
//输出结果如下；
// dlrow olleh
```

目录结构如下：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/7.png?raw=true">

如果使用`go install stringutil` ，会生成一个.a文件，目录结构如下：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/10.png?raw=true">

嗯，如果还不满足，可以再写个测试，go自带一个轻量级的测试库testing。在stringutil中加入一个测试文件reverse_test.go

```go
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```

然后运行go test;

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/8.png?raw=true">

这个案例看上去已经perfect了，但是如果还想试一试go get：实际上如果在其他包中直接import该包，go get会自动将它fetch，build，and install。如下

```go
import "github.com/golang/example/hello"
```

使用命令行如下

```go
go get github.com/golang/example/hello
```

如果本地的工作区中没有该库，go get会将它放在GOPATH的第一个工作区的特定目录中，如果该库已经存在，则go get会跳过fetch阶段直接进入install

其实下载的并不只有hello这个包，还包括example中其他的包，但是它只将hello包给install了

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/9.png?raw=true">

如果使用命令的是这样的

```go
go get github.com/golang/example
```

命令行会报出 `package github.com/example : No go file ...`之类的错误，它的意思是说example目录下没有go文件，但其实包已经引入了，只是没有上一步那样编译hello包而已。



在网上浏览文章的时候，还看到一些很有意思的其他问题：我们知道go是按照目录名来编译的，如果该目录下有两个包，情况会怎么样。假如结构如下： [原文链接](https://blog.csdn.net/CMbug/article/details/49339341)

>stringutil
>
>--- reverse.go   (package stringutil_2)
>
>--- reverse_test.go   (package stringutil)

像GoLand这样的IDE会直接报错：同一个目录下存在多个包 ，

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/11.png?raw=true">

如果强行install会出现如下情况：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/12.png?raw=true">

我们可以通过查看install命令的帮助来看看install需要的东西：

<img src="https://github.com/krystalics/krystalics.github.io/blob/master/_posts/go/img/13.png?raw=true">

其中 Install compiles and installs  the packages named by the import paths。大致意思就是包名和导入路径一致，而因为导入路径是按照目录来编译的，所以包名要与目录名一致。如果目录中只有一个.go文件，包名和目录名可以不一致。import的时候是目录名，然后使用的时候是包名

```go
package stringutil_2  //包名与目录名不一致
import "stringutil"  // 导入时使用目录名
stringutil_2.Reverse() //使用时恢复 包名
```





go get下载的包如果都在第一个工作区，那么其他工作区中怎么会有相同包路径的包，是复制过去的吗？（这里想到了一个答复，相同包路径可能是由于我们人在不同工作区手动创建了相同的路径）









