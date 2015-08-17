# go env

命令```go env```用于打印Go语言的环境信息。其中的一些信息我们在之前已经多次提及，但是却没有进行详细的说明。在本小节，我们会对这些信息进行深入介绍。我们先来看一看```go env```命令情况下都会打印出哪些Go语言通用环境信息。

_表0-25 ```go env```命令可打印出的Go语言通用环境信息_
<table class="table table-bordered table-striped table-condensed">
   <tr>
    <th width=100px>
	  名称
	</th>
	<th width=220px>
	  说明
	</th>   
  </tr>
  <tr>
    <td>
	  CGO_ENABLED
	</td>
    <td>
	  指明cgo工具是否可用的标识。
	</td>
  </tr>
  <tr>
    <td>
	  GOARCH
	</td>
    <td>
	  程序构建环境的目标计算架构。
	</td>
  </tr>
  <tr>
    <td>
	  GOBIN
	</td>
    <td>
	  存放可执行文件的目录的绝对路径。
	</td>
  </tr>
  <tr>
    <td>
	  GOCHAR
	</td>
    <td>
	  程序构建环境的目标计算架构的单字符标识。
	</td>
  </tr>
  <tr>
    <td>
	  GOEXE
	</td>
    <td>
	  可执行文件的后缀。
	</td>
  </tr>
  <tr>
    <td>
	  GOHOSTARCH
	</td>
    <td>
	  程序运行环境的目标计算架构。
	</td>
  </tr>
  <tr>
    <td>
	  GOOS
	</td>
    <td>
	  程序构建环境的目标操作系统。
	</td>
  </tr>
  <tr>
    <td>
	  GOHOSTOS
	</td>
    <td>
	  程序运行环境的目标操作系统。
	</td>
  </tr>
  <tr>
    <td>
	  GOPATH
	</td>
    <td>
	  工作区目录的绝对路径。
	</td>
  </tr>
  <tr>
    <td>
	  GORACE
	</td>
    <td>
	  用于数据竞争检测的相关选项。
	</td>
  </tr>
  <tr>
    <td>
	  GOROOT
	</td>
    <td>
	  Go语言的安装目录的绝对路径。
	</td>
  </tr>
  <tr>
    <td>
	  GOTOOLDIR
	</td>
    <td>
	  Go工具目录的绝对路径。
	</td>
  </tr>
</table>

下面我们对这些环境信息进行逐一说明。

**CGO_ENABLED**

通过上一小节的介绍，相信读者对cgo工具已经很熟悉了。我们提到过，标准go命令可以自动的使用cgo工具对导入了代码包C的代码包和源码文件进行处理。这里所说的“自动”并不是绝对的。因为当环境变量CGO_ENABLED被设置为0时，标准go命令就不能处理导入了代码包C的代码包和源码文件了。请看下面的示例：

	hc@ubt:~/golang/goc2p/src/basic/cgo$ export CGO_ENABLED=0
	hc@ubt:~/golang/goc2p/src/basic/cgo$ go build -x
	WORK=/tmp/go-build775234613

我们临时把环境变量CGO_ENABLED的值设置为0，然后执行```go build```命令并加入了标记```-x```。标记```-x```会让命令程序将运行期间所有实际执行的命令都打印到标准输出。但是，在执行命令之后没有任何命令被打印出来。这说明对代码包```basic/cgo```的构建操作并没有被执行。这是因为，构建这个代码包需要用到cgo工具，但cgo工具已经被禁用了。下面，我们再来运行调用了代码包```basic/cgo```中函数的命令源码文件cgo_demo.go。也就是说，命令源码文件cgo_demo.go间接的导入了代码包```C```。还记得吗？这个命令源码文件被存放在goc2p项目的代码包```basic/cgo```中。示例如下：

	hc@ubt:~/golang/goc2p/src/basic/cgo$ export CGO_ENABLED=0
	hc@ubt:~/golang/goc2p/src/basic/cgo$ go run -work cgo_demo.go
	WORK=/tmp/go-build856581210
	# command-line-arguments
	./cgo_demo.go:4: can't find import: "basic/cgo/lib"

在上面的示例中，我们在执行```go run```命令时加入了两个标记——```-a```和```-work```。标记```-a```会使命令程序强行重新构建所有的代码包（包括涉及到的标准库），即使它们已经是最新的了。标记```-work```会使命令程序将临时工作目录的绝对路径打印到标准输出。命令程序输出的错误信息显示，命令程序没有找到代码包```basic/cgo```。其原因是由于代码包```basic/cgo```无法被构建。所以，命令程序在临时工作目录和工作区中都找不到代码包basic/cgo对应的归档文件cgo.a。如果我们使用命令```ll /tmp/go-build856581210```查看临时工作目录，也找不到名为basic的目录。

不过，如果我们在环境变量CGO_ENABLED的值为1的情况下生成代码包```basic/cgo```对应的归档文件cgo.a，那么无论我们之后怎样改变环境变量CGO_ENABLED的值也都可以正确的运行命令源码文件cgo_demo.go。即使我们在执行```go run```命令时加入标记```-a```也是如此。因为命令程序依然可以在工作区中找到之前在我们执行```go install```命令时生成的归档文件cgo.a。示例如下：

	hc@ubt:~/golang/goc2p/src/basic/cgo$ export CGO_ENABLED=1
	hc@ubt:~/golang/goc2p/src/basic/cgo$ go install ../basic/cgo
	hc@ubt:~/golang/goc2p/src/basic/cgo$ export CGO_ENABLED=0
	hc@ubt:~/golang/goc2p/src/basic/cgo$ go run -a -work cgo_demo.go
	WORK=/tmp/go-build130612063
	The square root of 2.330000 is 1.526434.
	ABC
	CFunction1() is called.
	GoFunction1() is called.

由此可知，只要我们事先成功安装了引用了代码包C的代码包，即生成了对应的代码包归档文件，即使cgo工具在之后被禁用，也不会影响到其它Go语言代码对该代码包的使用。当然，命令程序首先会到临时工作目录中寻找需要的代码包归档文件。

关于cgo工具还有一点需要特别注意，即：当存在交叉编译的情况时，cgo工具一定是不可用的。在标准go命令的上下文环境中，交叉编译意味着程序构建环境的目标计算架构的标识与程序运行环境的目标计算架构的标识不同，或者程序构建环境的目标操作系统的标识与程序运行环境的目标操作系统的标识不同。在这里，我们可以粗略认为交叉编译就是在当前的计算架构和操作系统下编译和构建Go语言代码并生成针对于其他计算架构或/和操作系统的编译结果文件和可执行文件。

**GOARCH**

GOARCH的值的含义是程序构建环境的目标计算架构的标识，也就是程序在构建或安装时所对应的计算架构的名称。在默认情况下，它会与程序运行环境的目标计算架构一致。即它的值会与GOHOSTARCH的值是相同。但如果我们显式的设置了环境变量GOARCH，则它的值就会是这个环境变量的值。

**GOBIN**

GOBIN的值为存放可执行文件的目录的绝对路径。它的值来自环境变量GOBIN。在我们使用```go tool install```命令安装命令源码文件时生成的可执行文件会存放于这个目录中。

**GOCHAR**

GOCHAR的值是程序构建环境的目标计算架构的单字符标识。它的值会根据GOARCH的值来设置。当GOARCH的值为386时，GOCHAR的值就是8。当GOARCH的值为amd64时GOCHAR的值就是6。而GOCHAR的值5与GOARCH的值arm相对应。

GOCHAR主要有两个用途，列举如下：

1. Go语言官方的平台相关的工具的名称会以它的值为前缀。的名称会以GOCHAR的值为前缀。比如，在amd64计算架构下，用于编译Go语言代码的编译器的名称是6g，链接器的名称是6l。用于编译C语言代码的编译器的名称是6c。而用于编译汇编语言代码的编译器的名称为6a。

2. Go语言的官方编译器生成的结果文件会以GOCHAR的值作为扩展名。Go语言的官方编译器6g在对命令源码文件编译之后会把结果文件_go_.6存放到临时工作目录的相应位置中。

**GOEXE**

GOEXE的值会被作为可执行文件的后缀。它的值与GOOS的值存在一定关系，即只有GOOS的值为“windows”时GOEXE的值才会是“.exe”，否则其值就为空字符串“”。这与在各个操作系统下的可执行文件的默认后缀是一致的。

**GOHOSTARCH**

GOHOSTARCH的值的含义是程序运行环境的目标计算架构的标识，也就是程序在运行时所在的计算机系统的计算架构的名称。在通常情况下，它的值不需要被显式的设置。因为用来安装Go语言的二进制分发文件和MSI（Microsoft软件安装）软件包文件都是平台相关的。所以，对于不同计算架构的Go语言环境来说，它都会是一个常量。

**GOHOSTOS**

GOHOSTOS的值的含义是程序运行环境的目标操作系统的标识，也即程序在运行时所在的计算机系统的操作系统的名称。与GOHOSTARCH类似，它的值在不同的操作系统下是固定不变的，同样不需要显式的设置。

**GOPATH**

这个环境信息我们在之前已经提到过很多次。它的值指明了Go语言工作区目录的绝对路径。我们需要显式的设置环境变量GOPATH。如果有多个工作区，那么多个工作区的绝对路径之间需要用分隔符分隔。在windows操作系统下，这个分隔符为“;”。在其它操作系统下，这个分隔符为“:”。注意，GOPATH的值不能与GOROOT的值相同。

**GORACE**

GORACE的值包含了用于数据竞争检测的相关选项。数据竞争是在并发程序中最常见和最难调试的一类bug。数据竞争会发生在多个Goroutine争相访问相同的变量且至少有一个Goroutine中的程序在对这个变量进行写操作的情况下。

数据竞争检测需要被显式的开启。还记得标记```-race```吗？我们可以通过在执行一些标准go命令时加入这个标记来开启数据竞争检测。在这种情况下，GORACE的值就会被使用到了。支持```-race```标记的标准go命令包括：```go test```命令、```go run```命令、```go build```命令和```go install```命令。

GORACE的值形如“option1=val1 option2=val2”，即：选项名称与选项值之间以等号“=”分隔，多个选项之间以空格“ ”分隔。数据竞争检测的选项包括log_path、exitcode、strip_path_prefix和history_size。为了设置GORACE的值，我们需要设置环境变量GORACE。或者，我们也可以在执行go命令时临时设置它，像这样：

	hc@ubt:~/golang/goc2p/src/cnet/ctcp$ GORACE="log_path=/home/hc/golang/goc2p /race/report strip_path_prefix=home/hc/golang/goc2p/" go test -race

关于数据竞争检测的更多细节我们将会在本书的第三部分予以说明。

**GOROOT**

GOROOT会是我们在安装Go语言时第一个碰到Go语言环境变量。它的值指明了Go语言的安装目录的绝对路径。但是，只有在非默认情况下我们才需要显式的设置环境变量GOROOT。这里所说的默认情况是指：在Windows操作系统下我们把Go语言安装到c:\Go目录下，或者在其它操作系统下我们把Go语言安装到/usr/local/go目录下。另外，当我们不是通过二进制分发包来安装Go语言的时候，也不需要设置环境变量GOROOT的值。比如，在Windows操作系统下，我们可以使用MSI软件包文件来安装Go语言。

**GOTOOLDIR**

GOTOOLDIR的值指明了Go工具目录的绝对路径。根据GOROOT、GOHOSTOS和GOHOSTARCH来设置。其值为$GOROOT/pkg/tool/$GOOS_$GOARCH。关于这个目录，我们在之前也提到过多次。

除了上面介绍的这些通用的Go语言环境信息，还两个针对于非Plan 9操作系统的环境信息。它们是CC和GOGCCFLAGS。环境信息CC的值是操作系统默认的C语言编译器的命令名称。环境信息GOGCCFLAGS的值则是Go语言在使用操作系统的默认C语言编译器对C语言代码进行编译时加入的参数。

如果我们要有针对性的查看上述的一个或多个环境信息，可以在```go env```命令的后面加入它们的名字并执行之。在```go env```命令和环境信息名称之间需要用空格分隔，多个环境信息名称之间也需要用空格分隔。示例如下：

	hc@ubt:~$ go env GOARCH GOCHAR
	386
	8

上例的```go env```命令的输出信息中，每一行对一个环境信息的值，且其顺序与我们输入的环境信息名称的顺序一致。比如，386为环境信息GOARCH，而8则是环境信息GOCHAR的值。

```go env```命令能够让我们对当前的Go语言环境进行简要的了解。通过它，我们也可以对当前安装的Go语言的环境设置进行简单的检查。 
