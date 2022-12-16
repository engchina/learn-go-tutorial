# Pac Man(吃豆游戏)

使用Golang编写的"Pac Man"克隆版(有表情符号!)

![screenshot](./screenshot.jpg)

## 简介

欢迎来到Pac Go！ 这个项目是一个向人们介绍[Go编程语言](https://golang.org)的教程。

### 为什么要做一个新的教程？

我们有很多很好的教程，但是关于这个教程的整个想法是在学习Go的时候做一些不同的、有趣的东西。是的，我们每天都需要了解API和CRUD，但在处理新事物时，为什么不做一个游戏呢？

Go是一种以使编程重新变得有趣而闻名的语言，还有什么比编写一个游戏更有趣的呢？有兴趣吗？很好，让我们继续吧!

我们将在终端机上编写一个"Pac Man"的克隆游戏。在编写游戏的过程中，你会接触到一些有趣的要求，这些要求为探索语言的许多特性提供了非常好的基础，包括输入、输出、函数、指针、并发和一点数学。

你还会学到更多关于终端和其神奇的转义序列的知识。

### 许可证

见 [LICENSE](LICENSE)。

### 联系原作者

如果你有任何问题，请通过[daniela.petruzalek@gmail.com](mailto:daniela.petruzalek@gmail.com)与原作者联系。

## 开始学习

### 前提条件

建议具备以下条件：
- 对编程语言的工作原理有基本的了解，因为我们不会涉及基础知识
- 基本的终端知识（知道如何使用命令行应用程序）

当然，如果你不具备上述条件，但又有好奇心，想尝试一下，请随意。

### 兼容性警告！！！

本教程已在**Linux**和**Mac OS X**环境下测试。对于Windows环境，你可能_需要安装一个终端模拟器，如[Git BASH](https://gitforwindows.org/)。

请注意，由于这段代码依赖于终端来渲染游戏，因此在不同的配置下会产生不同的结果。

如果你有问题，请随时提出来，以便我们能找到一个合适的解决方案，同时提供你的操作系统和终端的名称和版本。

**注意：**目前已知的问题是，VS Code上的终端窗口不能正确渲染游戏。


### 设置

为了开始，请确保你的系统中已经安装了Go。

如果你没有，请按照[golang.org](https://golang.org)中的说明进行操作。

### 如何使用本教程

在每一个步骤中，包括第0步（这个步骤），我们都会在README.md文件中描述任务，然后是做这个任务的代码以及对其工作原理的解释。

除了这个步骤之外，每个步骤都位于其独立的文件夹中。为任何给定的步骤寻找文件夹stepXX。

我们将编辑一个名为`main.go`的文件。本教程中的所有代码都将放在这个文件中。一个合适的程序通常会有多个源代码文件，但为了简单起见，我们将这个程序限制在一个源代码中。

每个步骤的README.md将解释其意图，并显示继续进行所需的修改。你应该在你自己的`main.go`文件中进行这些修改。

你也可以把第00步中的`main.go`作为一个起点，在逐步推进的过程中逐步修改它。

如果你遇到问题了，每个步骤都有自己的`main.go`文件，并且已经应用了对该步骤的修改。这也意味着，如果你想快速进入某个步骤，你可以从上一个步骤的`main.go`开始。

## Step 00：你好（游戏）世界

我们首先要做的是为一个游戏程序打下基础，使其看起来像一个骨架。

选择一个目录作为你的工作目录（例如，在你的主文件夹下的`tutorial`），然后创建一个名为`main.go`的文件，内容如下。

注：或者你可以直接克隆这个资源库，并在其根目录下编辑`main.go`文件。

```go
package main

import "fmt"

func main() {
    // initialize game

    // load resources

    // game loop
    for {
        // update screen

        // process input

        // process movement

        // process collisions

        // check game over

        // Temp: break infinite loop
        fmt.Println("Hello, Pac Go!")
        break

        // repeat
    }
}
```

### 运行你的第一个Go程序

现在我们有了一个`main.go`文件（`.go`是Go源代码的文件扩展名），你可以使用命令行工具`go`来运行它。只要输入

```sh
$ go run main.go
Hello, Pac Go!
```

这就是我们在Go中运行单个文件的方法。你也可以用`go build`命令把它编译成一个可执行的程序。如果你运行`go build`，它将把当前目录下的文件编译成一个二进制文件，并标明目录的名称。然后你可以把它作为一个普通的程序来运行，比如说：

```sh
$ go build
$ ./pacgo
Hello, Pac Go!
```

在本教程中，我们只使用一个代码文件（`main.go`），所以你可以使用`go build`并运行命令，或者只使用`go run main.go`，因为它都会自动完成。

### 理解该程序

现在让我们仔细看看这个程序是做什么的。

第一行是"包"的名称。每个有效的Go程序都必须有一个这样的名字。另外，如果你想制作一个**可运行的**程序，你必须有一个叫做`main'的包和一个`main'函数，它将是程序的入口。

我们正在创建一个可执行的程序，所以我们在文件的顶部使用`package main`。

接下来是`import`语句。你使用这些语句来使其他包中的代码能够被你的程序访问。

最后是 "main "函数。你用关键字`func`在Go中定义函数，后面是它的名字，它的参数在一对小括号之间，后面是返回值，最后是函数体，它包含在一对大括号`{}`中。比如说：

```go
func main() {
    // I'm a function body
}
```

这是一个叫做`main'的函数，它不接受任何参数，也不返回任何东西。这是Go语言中main函数的正确定义，与其他语言中的定义不同，main函数可以接受命令行参数和/或返回一个整数值。

我们有不同的方法来处理Go中的命令行参数和返回值，我们将在[Step 08](step08/README.md)中看到。

在游戏主函数中，我们有一些注释（`//`后面的任何文字都是注释），这些注释是作为实际游戏代码的占位符。我们将用这些来有序地驱动每一个修改。

游戏中最重要的概念是所谓的游戏循环。这基本上是一个无限的循环，所有的游戏机制都在这里被处理。

Go中的循环是用关键字"for"来定义的。`for`循环可以有一个初始化器，一个条件和一个后处理步骤，但所有这些都是可选的。如果你省略了这些，你就会有一个永远不会结束的循环。

```go
for {
    // I never end
}
```

我们可以用`break`语句退出一个无限循环。我们在示例代码中使用它，在用`fmt`包中的`Println`函数打印 "Hello, Pac Go!"之后，结束无限循环（为简洁起见，省略注释）。

```go
func main() {
    for {
        fmt.Println("Hello, Pac Go!")
        break
    }
}
```

当然，在这种情况下，带有非条件中断的无限循环是没有意义的，但在接下来的步骤中会有意义的！

恭喜你，第00步已经完成了！

[带我到第01步！](step01/README.md)
