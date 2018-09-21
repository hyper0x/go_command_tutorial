# go get

```go
hc@ubt:~$ go get github.com/hyper-carrot/go_lib/logging
```

命令`go get`可以根据要求和实际情况从互联网上下载或更新指定的代码包及其依赖包，并对它们进行编译和安装。在上面这个示例中，我们从著名的代码托管站点Github上下载了一个项目（或称代码包），并安装到了环境变量GOPATH中包含的第一个工作区中。与此同时，我们也知道了这个代码包的导入路径就是github.com/hyper-carrot/go_lib/logging。

一般情况下，为了分离自己与第三方的代码，我们会设置两个或更多的工作区。我们现在有一个目录路径为/home/hc/golang/lib的工作区，并且它是环境变量GOPATH值中的第一个目录路径。注意，环境变量GOPATH中包含的路径不能与环境变量GOROOT的值重复。好了，如果我们使用`go get`命令下载和安装代码包，那么这些代码包都会被安装在上面这个工作区中。我们暂且把这个工作区叫做Lib工作区。在我们运行`go get github.com/hyper-carrot/go_lib/logging`之后，这个代码包就应该会被保存在Lib工作的src目录下，并且已经被安装妥当，如下所示：

```go
/home/hc/golang/lib:
	bin/
	pkg/
		linux_386/
			github.com/
				hyper-carrot/
					go_lib/
						logging.a  
	src/
		github.com/
			hyper-carrot/
				go_lib/
					logging/
	...
```

另一方面，如果我们想把一个项目上传到Github网站（或其他代码托管网站）上并被其他人使用的话，那么我们就应该把这个项目当做一个代码包来看待。其实我们在之前已经提到过原因，`go get`命令会将项目下的所有子目录和源码文件存放到第一个工作区的src目录下，而src目录下的所有子目录都会是某个代码包导入路径的一部分或者全部。也就是说，我们应该直接在项目目录下存放子代码包和源码文件，并且直接存放在项目目录下的源码文件所声明的包名应该与该项目名相同（除非它是命令源码文件）。这样做可以让其他人使用`go get`命令从Github站点上下载你的项目之后直接就能使用它。

实际上，像goc2p项目这样直接以项目根目录的路径作为工作区路径的做法是不被推荐的。之所以这样做主要是想让读者更容易的理解Go语言的工程结构和工作区概念，也可以让读者看到另一种项目结构。当然，如果你的项目使用了[gb](https://github.com/constabulary/gb)这样的工具那就是另外一回事了。这样的项目的根目录就应该被视为一个工作区（但是你不必把它加入到GOPATH环境变量中）。它应该由`git clone`下载到Go语言工作区之外的某处，而不是使用`go get`命令。

**远程导入路径分析**

实际上，`go get`命令所做的动作也被叫做代码包远程导入，而传递给该命令的作为代码包导入路径的那个参数又被叫做代码包远程导入路径。

`go get`命令不仅可以从像Github这样著名的代码托管站点上下载代码包，还可以从任何命令支持的代码版本控制系统（英文为Version Control System，简称为VCS）检出代码包。任何代码托管站点都是通过某个或某些代码版本控制系统来提供代码上传下载服务的。所以，更严格地讲，`go get`命令所做的是从代码版本控制系统的远程仓库中检出/更新代码包并对其进行编译和安装。

该命令所支持的VCS的信息如下表：

_表0-2 ```go get```命令支持的VCS_

名称        | 主命令   | 说明
---------- | ------- | -----
Mercurial  | hg      | Mercurial是一种轻量级分布式版本控制系统，采用Python语言实现，易于学习和使用，扩展性强。
Git        | git     | Git最开始是Linux Torvalds为了帮助管理 Linux 内核开发而开发的一个开源的分布式版本控制软件。但现在已被广泛使用。它是被用来进行有效、高速的各种规模项目的版本管理。
Subversion | svn     | Subversion是一个版本控制系统，也是第一个将分支概念和功能纳入到版本控制模型的系统。但相对于Git和Mercurial而言，它只算是传统版本控制系统的一员。
Bazaar    | bzr      | 	  Bazaar是一个开源的分布式版本控制系统。但相比而言，用它来作为VCS的项目并不多。

`go get `命令在检出代码包之前必须要知道代码包远程导入路径所对应的版本控制系统和远程仓库的URL。

如果该代码包在本地工作区中已经存在，则会直接通过分析其路径来确定这几项信息。`go get `命令支持的几个版本控制系统都有一个共同点，那就是会在检出的项目目录中存放一个元数据目录，名称为“.”前缀加其主命令名。例如，Git会在检出的项目目录中加入一个名为“.git”的子目录。所以，这样就很容易判定代码包所用的版本控制系统。另外，又由于代码包已经存在，我们只需通过代码版本控制系统的更新命令来更新代码包，因此也就不需要知道其远程仓库的URL了。对于已存在于本地工作区的代码包，除非要求强行更新代码包，否则`go get`命令不会进行重复下载。如果想要强行更新代码包，可以在执行`go get`命令时加入`-u`标记。这一标记会稍后介绍。

如果本地工作区中不存在该代码包，那么就只能通过对代码包远程导入路径进行分析来获取相关信息了。首先，`go get`命令会对代码包远程导入路径进行静态分析。为了使分析过程更加方便快捷，`go get`命令程序中已经预置了几个著名代码托管网站的信息。如下表：

_表0-3 预置的代码托管站点的信息_


名称                         | 主域名           | 支持的VCS                   | 代码包远程导入路径示例
--------------------------- | --------------- | -------------------------- | --------
Bitbucket                   | bitbucket.org   | Git, Mercurial             | bitbucket.org/user/project<br>bitbucket.org/user/project/sub/directory
GitHub                      | github.com      | Git                        | github.com/user/project<br>github.com/user/project/sub/directory
Google Code Project Hosting | code.google.com | Git, Mercurial, Subversion | code.google.com/p/project<br>code.google.com/p/project/sub/directory<br>code.google.com/p/project.subrepository<br>code.google.com/p/project.subrepository/sub/directory
Launchpad                   | launchpad.net   | Bazaar                     | launchpad.net/project<br>launchpad.net/project/series<br>launchpad.net/project/series/sub/directory<br>launchpad.net/~user/project/branch<br>launchpad.net/~user/project/branch/sub/directory
IBM DevOps Services         | hub.jazz.net    | Git                        | hub.jazz.net/git/user/project<br>hub.jazz.net/git/user/project/sub/directory

一般情况下，代码包远程导入路径中的第一个元素就是代码托管网站的主域名。在静态分析的时候，`go get`命令会将代码包远程导入路径与预置的代码托管站点的主域名进行匹配。如果匹配成功，则在对代码包远程导入路径的初步检查后返回正常的返回值或错误信息。如果匹配不成功，则会再对代码包远程导入路径进行动态分析。至于动态分析的过程，我就不在这里详细展开了。

如果对代码包远程导入路径的静态分析或/和动态分析成功并获取到对应的版本控制系统和远程仓库URL，那么`go get`命令就会进行代码包检出或更新的操作。随后，`go get`命令会在必要时以同样的方式检出或更新这个代码包的所有依赖包。

**自定义代码包远程导入路径**

如果你想把你编写的（被托管在不同的代码托管网站上的）代码包的远程导入路径统一起来，或者不希望让你的代码包中夹杂某个代码托管网站的域名，那么你可以选择自定义你的代码包远程导入路径。这种自定义的实现手段叫做“导入注释”。导入注释的写法示例如下：

```go
package analyzer // import "hypermind.cn/talon/analyzer"
```

代码包`analyzer`实际上属于我的一个网络爬虫项目。这个项目的代码被托管在了Github网站上。它的网址是：[https://github.com/hyper-carrot/talon](https://github.com/hyper-carrot/talon)。如果用标准的导入路径来下载`analyzer`代码包的话，命令应该这样写`go get github.com/hyper-carrot/talon/analyzer`。不过，如果我们像上面的示例那样在该代码包中的一个源码文件中加入导入注释的话，这样下载它就行不通了。我们来看一看这个导入注释。

导入注释的写法如同一条代码包导入语句。不同的是，它出现在了单行注释符`//`的右边，因此Go语言编译器会忽略掉它。另外，它必须出现在源码文件的第一行语句（也就是代码包声明语句）的右边。只有符合上述这两个位置条件的导入注释才是有效的。再来看其中的引号部分。被双引号包裹的应该是一个符合导入路径语法规则的字符串。其中，`hypermind.cn`是我自己的一个域名。实际上，这也是用来替换掉我想隐去的代码托管网站域名及部分路径（这里是`github.com/hyper-carrot`）的那部分。在`hypermind.cn`右边的依次是我的项目的名称以及要下载的那个代码包的相对路径。这些与其标准导入路径中的内容都是一致的。为了清晰起见，我们再来做下对比。

```go
github.com/hyper-carrot/talon/analyzer // 标准的导入路径
hypermind.cn           /talon/analyzer // 导入注释中的导入路径                   
```

你想用你自己的域名替换掉标准导入路径中的哪部分由你自己说了算。不过一般情况下，被替换的部分包括代码托管网站的域名以及你在那里的用户ID就可以了。这足以达到我们最开始说的那两个目的。

虽然我们在talon项目中的所有代码包中都加入了类似的导入注释，但是我们依然无法通过`go get hypermind.cn/talon/analyzer`命令来下载这个代码包。因为域名`hypermind.cn`所指向的网站并没有加入相应的处理逻辑。具体的实现步骤应该是这样的：

1. 编写一个可处理HTTP请求的程序。这里无所谓用什么编程语言去实现。当然，我推荐你用Go语言去做。

2. 将这个处理程序与`hypermind.cn/talon`这个路径关联在一起，并总是在作为响应的HTML文档的头中写入下面这行内容：

  ```html
  <meta name="go-import" content="hypermind.cn/talon git https://github.com/hyper-carrot/talon">
  ```
  hypermind.cn/talon/analyzer熟悉HTML的读者都应该知道，这行内容会被视为HTML文档的元数据。它实际上`go get`命令的文档中要求的写法。它的模式是这样的：

```html
<meta name="go-import" content="import-prefix vcs repo-root">
```

实际上，`content`属性中的`import-prefix`的位置上应该填入我们自定义的远程代码包导入路径的前缀。这个前缀应该与我们的处理程序关联的那个路径相一致。而`vcs`显然应该代表与版本控制系统有关的标识。还记得表0-2中的主命令列吗？这里的填入内容就应该该列中的某一项。在这里，由于talon项目使用的是Git，所以这里应该填入`git`。至于`repo-root`，它应该是与该处理程序关联的路径对应的Github网站的URL。在这里，这个路径是`hypermind.cn/talon`，那么这个URL就应该是`https://github.com/hyper-carrot/talon`。后者也是talon项目的实际网址。

好了，在我们做好上述处理程序之后，`go get hypermind.cn/talon/analyzer`命令的执行结果就会是正确的。`analyzer`代码包及其依赖包中的代码会被下载到GOPATH环境变量中的第一个工作区目录的src子目录中，然后被编译并安装。

注意，具体的代码包源码存放路径会是/home/hc/golang/lib/src/hypermind.cn/talon/analyzer。也就是说，存放路径（包括代码包源码文件以及相应的归档文件的存放路径）会遵循导入注释中的路径（这里是`hypermind.cn/talon/analyzer`），而不是原始的导入路径（这里是`github.com/hyper-carrot/talon/analyzer`）。另外，我们只需在talon项目的每个代码包中的某一个源码文件中加入导入注释，但这些导入注释中的路径都必须是一致的。在这之后，我们就只能使用`hypermind.cn/talon/`作为talon项目中的代码包的导入路径前缀了。一个反例如下：

```go
hc@ubt:~$ go get github.com/hyper-carrot/talon/analyzer
package github.com/hyper-carrot/talon/analyzer: code in directory /home/hc/golang/lib/src/github.com/hyper-carrot/talon/analyzer expects import "hypermind.cn/talon/analyzer"
```

与自定义的代码包远程导入路径有关的内容我们就介绍到这里。从中我们也可以看出，Go语言为了让使用者的项目与代码托管网站隔离所作出的努力。只要你有自己的网站和一个不错的域名，这就很容易搞定并且非常值得。这会在你的代码包的使用者面前强化你的品牌，而不是某个代码托管网站的。当然，使你的代码包导入路径整齐划一是最直接的好处。

OK，言归正传，我下面继续关注`go get`这个命令本身。

**命令特有标记**

`go get`命令可以接受所有可用于`go build`命令和`go install`命令的标记。这是因为`go get`命令的内部步骤中完全包含了编译和安装这两个动作。另外，`go get`命令还有一些特有的标记，如下表所示：

_表0-4 ```go get```命令的特有标记说明_

标记名称    | 标记描述 
--------- | ------- 
-d        | 让命令程序只执行下载动作，而不执行安装动作。
-f        | 仅在使用`-u`标记时才有效。该标记会让命令程序忽略掉对已下载代码包的导入路径的检查。如果下载并安装的代码包所属的项目是你从别人那里Fork过来的，那么这样做就尤为重要了。
-fix      | 让命令程序在下载代码包后先执行修正动作，而后再进行编译和安装。
-insecure | 允许命令程序使用非安全的scheme（如HTTP）去下载指定的代码包。如果你用的代码仓库（如公司内部的Gitlab）没有HTTPS支持，可以添加此标记。请在确定安全的情况下使用它。
-t        | 让命令程序同时下载并安装指定的代码包中的测试源码文件中依赖的代码包。
-u        | 让命令利用网络来更新已有代码包及其依赖包。默认情况下，该命令只会从网络上下载本地不存在的代码包，而不会更新已有的代码包。

为了更好的理解这几个特有标记，我们先清除Lib工作区的src目录和pkg目录中的所有子目录和文件。现在我们使用带有`-d`标记的`go get`命令来下载同样的代码包：

```go
hc@ubt:~$ go get -d github.com/hyper-carrot/go_lib/logging
```

现在，让我们再来看一下Lib工作区的目录结构：

```go
/home/hc/golang/lib:
	bin/
	pkg/
	src/
		github.com/
		hyper-carrot/
		go_lib/
			logging/
	...
```

我们可以看到，`go get`命令只将代码包下载到了Lib工作区的src目录，而没有进行后续的编译和安装动作。这个加入`-d`标记的结果。

再来看`-fix`标记。我们知道，绝大多数计算机编程语言在进行升级和演进过程中，不可能保证100%的向后兼容（Backward Compatibility）。在计算机世界中，向后兼容是指在一个程序或者代码库在更新到较新的版本后，用旧的版本程序创建的软件和系统仍能被正常操作或使用，或在旧版本的代码库的基础上编写的程序仍能正常编译运行的能力。Go语言的开发者们已想到了这点，并提供了官方的代码升级工具——`fix`。`fix`工具可以修复因Go语言规范变更而造成的语法级别的错误。关于`fix`工具，我们将放在本节的稍后位置予以说明。

假设我们本机安装的Go语言版本是1.5，但我们的程序需要用到一个很早之前用Go语言的0.9版本开发的代码包。那么我们在使用`go get`命令的时候可以加入`-fix`标记。这个标记的作用是在检出代码包之后，先对该代码包中不符合Go语言1.5版本的语言规范的语法进行修正，然后再下载它的依赖包，最后再对它们进行编译和安装。

标记`-u`的意图和执行的动作都比较简单。我们在执行`go get`命令时加入`-u`标记就意味着，如果在本地工作区中已存在相关的代码包，那么就是用对应的代码版本控制系统的更新命令更新它，并进行编译和安装。这相当于强行更新指定的代码包及其依赖包。我们来看如下示例：

```go
hc@ubt:~$ go get -v github.com/hyper-carrot/go_lib/logging 
```

因为我们在之前已经检出并安装了这个代码包，所以我们执行上面这条命令后什么也没发生。还记得加入标记`-v`标记意味着会打印出被构建的代码包的名字吗？现在我们使用标记`-u`来强行更新代码包：	

```go
hc@ubt:~$ go get -v -u  github.com/hyper-carrot/go_lib/logging
github.com/hyper-carrot/go_lib (download)
```	
	
其中，“(download)”后缀意味着命令从远程仓库检出或更新了该行显示的代码包。如果我们要查看附带`-u`的`go get`命令到底做了些什么，还可以加上一个`-x`标记，以打印出用到的命令。读者可以自己试用一下它。

**智能的下载**

命令`go get`还有一个很值得称道的功能。在使用它检出或更新代码包之后，它会寻找与本地已安装Go语言的版本号相对应的标签（tag）或分支（branch）。比如，本机安装Go语言的版本是1.x，那么`go get`命令会在该代码包的远程仓库中寻找名为“go1”的标签或者分支。如果找到指定的标签或者分支，则将本地代码包的版本切换到此标签或者分支。如果没有找到指定的标签或者分支，则将本地代码包的版本切换到主干的最新版本。

前面我们说在执行`go get`命令时也可以加入`-x`标记，这样可以看到`go get`命令执行过程中所使用的所有命令。不知道读者是否已经自己尝试了。下面我们还是以代码包`github.com/hyper-carrot/go_lib`为例，并且通过之前示例中的命令的执行此代码包已经被检出到本地。这时我们再次更新这个代码包：

```go
hc@ubt:~$ go get -v -u -x github.com/hyper-carrot/go_lib
github.com/hyper-carrot/go_lib (download)
cd /home/hc/golang/lib/src/github.com/hyper-carrot/go_lib
git fetch
cd /home/hc/golang/lib/src/github.com/hyper-carrot/go_lib
git show-ref
cd /home/hc/golang/lib/src/github.com/hyper-carrot/go_lib
git checkout origin/master
WORK=/tmp/go-build034263530
```
	
在上述示例中，`go get`命令通过`git fetch`命令将所有远程分支更新到本地，而后有用`git show-ref`命令列出本地和远程仓库中记录的代码包的所有分支和标签。最后，当确定没有名为“go1”的标签或者分支后，`go get`命令使用`git checkout origin/master`命令将代码包的版本切换到主干的最新版本。下面，我们在本地增加一个名为“go1”的标签，看看`go get`命令的执行过程又会发生什么改变：

```go
hc@ubt:~$ cd ~/golang/lib/src/github.com/hyper-carrot/go_lib
hc@ubt:~/golang/lib/src/github.com/hyper-carrot/go_lib$ git tag go1
hc@ubt:~$ go get -v -u -x github.com/hyper-carrot/go_lib
github.com/hyper-carrot/go_lib (download)
cd /home/hc/golang/lib/src/github.com/hyper-carrot/go_lib
git fetch
cd /home/hc/golang/lib/src/github.com/hyper-carrot/go_lib
git show-ref
cd /home/hc/golang/lib/src/github.com/hyper-carrot/go_lib
git show-ref tags/go1 origin/go1
cd /home/hc/golang/lib/src/github.com/hyper-carrot/go_lib
git checkout tags/go1
WORK=/tmp/go-build636338114
```
	
将这两个示例进行对比，我们会很容易发现它们之间的区别。第二个示例的命令执行过程中使用`git show-ref`查看所有分支和标签，当发现有匹配的信息又通过`git show-ref tags/go1 origin/go1`命令进行精确查找，在确认无误后将本地代码包的版本切换到标签“go1”之上。

命令`go get`的这一功能是非常有用的。我们的代码在直接或间接依赖某些同时针对多个Go语言版本开发的代码包时，可以自动的检出其正确的版本。也可以说，`go get`命令内置了一定的代码包多版本依赖管理的功能。 

到这里，我向大家介绍了`go get`命令的使用方式。`go get`命令与之前介绍的两个命令一样，是我们编写Go语言程序、构建Go语言项目时必不可少的辅助工具。
