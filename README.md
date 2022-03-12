# GO 命令教程

## 作者介绍

**郝林**，从事互联网软件研发和管理工作已有15年，在银行、电信、社交网络、互联网金融、电子商务、大数据等领域都工作过。我对Go语言和Julia语言都情有独钟，并且目前正在独立从事编程教育研究、专业内容输出、在线社群运营等工作。我制作和发布过一些很受欢迎的免费教程、技术图书和付费专栏，其中就包括本教程。另外还有（按时间排序）：慕课网的免费教程[《Go语言第一课》](https://www.imooc.com/learn/345)、人邮图灵的原创技术图书[《Go并发编程实战》](https://www.ituring.com.cn/book/1950)、极客时间的付费专栏[《Go语言核心36讲》](https://time.geekbang.org/column/intro/112)等。

## 本教程的由来

这份Go命令教程原先是我撰写的图书[《Go并发编程实战》](http://www.ituring.com.cn/book/1525)中的一部分。这部分内容与并发编程的关系不大，故被砍掉。但是它是有价值的，也算是我对Go语言官方提供的标准命令的一个学习笔记。所以，我觉得应该把它做成免费资源分享给大家。经出版社的认可，我将这份教程放在这里供广大Go语言爱好者阅读。

**信息更新**：
- 2017-04-01：[《Go并发编程实战》第2版](http://www.ituring.com.cn/book/1950)已经出版了。

## 相关源

本教程在Github上的地址在[这里](https://github.com/hyper0x/go_command_tutorial)。如果你喜欢本教程，请不吝Star。如果你想贡献一份力量，欢迎提交Pull Request（提示：请小步pr，避免大刀阔斧）。

针对Go 1.3和1.4版本的教程已被放入分支[go1.3](https://github.com/hyper0x/go_command_tutorial/tree/go1.3)。而主分支此后一段时间内会致力于针对Go 1.5进行更新。

本教程中会提及一个名为`goc2p`的项目。该项目实际上是《Go并发编程实战》随书附带的示例项目。这本书中讲到的所有源码都在`goc2p`项目中。我已在《Go并发编程实战》出版之时将[`goc2p`](https://github.com/hyper0x/goc2p)项目放出。另外，《Go并发编程实战》第2版的示例项目在[这里](https://github.com/gopcp/example.v2)，可供参考。

## 关于协议

我希望这个项目中的内容永远是免费的。也就是说，任何组织和个人不应该出于商业目的而摘取其中的内容。更详细的条款请阅读本项目中的[LICENSE](https://github.com/hyper0x/go_command_tutorial/blob/master/LICENSE)文件。

## 版本信息
书中内容及演示代码基于以下版本：

| 系统      | 版本信息
|---------|------
|Golang   |1.3(will be abandoned) + 1.5(in progress)
